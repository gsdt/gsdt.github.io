---
title: SQL Column Truncation Vulnerabilities 
description: Lỗ hổng SQL Column Truncation
type: ctf-tutorial
language: Vietnamese
layout: post
tags: sql, web-security
author: giaosudauto 
---

SQL Column Truncation là một lỗ hổng liên quan tới việc các hệ quản trị cơ sở dữ liệu như MySQL cắt ngắn input đưa vào khi mà độ dài của input lớn hơn độ dài đã định nghĩa. Khi thao tác INSERT vào bảng, các lập trình viên phải kiểm tra lại input có hợp lệ không mới thực hiện thao tác INSERT. Vì nếu không sẽ gây ra nhiều rủi ro về bảo mật.

Ta lấy ví dụ như này:
Cột username có độ dài là 15 kí tự, nếu người dùng nhập hơn 15 kí tự thì chỉ có 15 kí tự đầu tiên được đưa vào bảng! 

Mặc dù hệ quản trị cơ sở dữ liệu có đưa ra một vài Cảnh báo, tuy nhiên đa phần cảnh báo này đều được bỏ qua. Trong MySQL chẳng hạn, người ta có thể cài đặt sql_mode thành STRICT_ALL_TABLE để chuyển các Cảnh báo thành thông báo Lỗi. Tuy nhiên phần lớn người ta để ở chế độ mặc định nếu việc kiểm tra lại tính hợp lệ của input là cần thiết.

**Kịch bản tấn công**:
Giải sử username có độ dài tối đa là 8 kí tự. Tài khoản quyền cao nhất là `username = 'admin'`
Đoạn code để kiểm tra username và password như sau:
```
$userdata = null;
if (isPasswordCorrect($username, $password)) {
   $userdata = getUserDataByLogin($username);
   ...
}
```
1. Hacker tạo một tài khoản có tên là `'admin              x'`. Sau khi thực hiện thao tác INSERT, chỉ có `'admin   '` là được đưa vào bảng.
2. Khi đăng nhập, câu lệnh sau sẽ được chạy:
`SELECT * FROM user WHERE username='admin   ' AND password='****'`
Vì hacker vừa tạo tài khoản này nên câu lệnh đó chạy thành công.
Sau khi check xong, câu query sau sẽ chạy:
`SELECT * FROM users WHERE username = 'admin   '`
kết quả là hacker sẽ lấy được quyền admin.

**Luyện tập**
1. https://ringzer0team.com/challenges/171


 
