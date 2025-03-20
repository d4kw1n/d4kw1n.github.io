---
title: 'vtables (Virtual Tables) in C++'
date: 2025-03-13 20:47:39
tags:
  - Knowledge
category:
  - Knowledge
  - C++
author:
  - d4kw1n
---

Trong thử thách bản thân để cố gắng clear challs nhiều nhất có thể trên pwnable.kr. Ở bài **uaf** mình đã stuck ở giai đoạn overwrite giá trị địa chỉ trên heap của chương trình C++ (Rõ ràng đã đè đúng địa chỉ mà nó không gọi được hàm (@_@)). Qua tìm hiểu thì mình biết được trong C++ có một thứ gọi là Virtual Tables (vtables). Cũng là vì tò mò, học hỏi và để giải được chall này nên mình ngồi nghiên cứu về nó trong thời gian rảnh rỗi lúc chưa thi giữa kỳ ^^_^^

> Bài này mình sẽ một phần phân tích thêm từ các bài viết mà mình tìm được.
> https://shaharmike.com/cpp/vtable-part1/

## 