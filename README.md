### Build
docker-compose up --build 

- Debug java web: `http://localhost:13337/debug-java-web-1.0-SNAPSHOT/`
- Level1: `http://localhost:13337/java-deserialize-lv1-1.0-SNAPSHOT/`
- Level2: `http://localhost:13337/java-deserialize-lv2-1.0-SNAPSHOT/`
- Level3: `http://localhost:13337/java-deserialize-lv3-1.0-SNAPSHOT/`
- Level4: `http://localhost:13337/java-deserialize-lv4-1.0-SNAPSHOT/`
- Deserialize exploit tool: `http://localhost:13337/deserialize-exploit-1.0-SNAPSHOT/`


Note: Cần tải đúng version jdk-8u131 trên máy thật để debug 


# Write-Up Java-Deserialization level 1-4: GOAL: RCE.

# Level 1:

Sau khi khởi động docker, truy cập theo đường link phía trên để truy cập vào level 1. Sau khi truy cập, ta sẽ nhận được giao diện sau.

<img width="1035" alt="Ảnh chụp Màn hình 2024-10-29 lúc 21 34 03" src="https://github.com/user-attachments/assets/65ea9f2b-f439-4bdb-8126-144b393a6e53">

Click vào ```Hell Servlet```

<img width="784" alt="Ảnh chụp Màn hình 2024-10-29 lúc 21 34 24" src="https://github.com/user-attachments/assets/b2188fd3-b07b-45f7-a44c-2b3aa696fbc8">

Không có gì đặc biệt nên ta bắt đầu phân tích source code xem bên trong có gì.

Level 1 gồm 3 class: ```User.java```, ```Admin.java```, ```HelloServlet.java```.

Class ```User``` có 1 thuộc tính là ```name``` và method ```getName``` để trả giá trị này về.

<img width="784" alt="Ảnh chụp Màn hình 2024-10-29 lúc 22 18 16" src="https://github.com/user-attachments/assets/c1e4780c-7cb5-4fa2-acb3-313773dd2804">

Tiếp theo là class ```Admin```.

<img width="880" alt="Ảnh chụp Màn hình 2024-10-29 lúc 22 19 51" src="https://github.com/user-attachments/assets/31871001-2b1d-46b3-bbb3-4bdba77fc85e">

Class này kế thừa class ```User``` và có thêm thuộc tính ```getNameCMD```.
Class ```Admin``` có sử dụng một magic method ```toString()``` sẽ trả về kết quả thực thi ```getNameCMD```.

Cuối cùng là class ```HelloServlet```.

<img width="941" alt="Ảnh chụp Màn hình 2024-10-30 lúc 01 02 44" src="https://github.com/user-attachments/assets/c58b6474-0658-486c-b6a2-a92acbc68957">

Clas này sẽ deserialize cookie để lấy giá trị user(dòng 43) và hiển thị ra browser cho người dùng (dòng 51).
Nếu ở phía browser không có cookie, server sẽ tạo 1 cookie mới và trả về cho browser (dòng 39).

Tuy nhiên, khi server tiến hành hiển thị giá trị user dưới dạng string ở dòng 51 đã kích hoạt magic method ```toString()```.

Khi ```toString()``` được kích hoạt, chương trình sẽ chạy lệnh OS command ```whoami```.

<img width="880" alt="Ảnh chụp Màn hình 2024-10-29 lúc 22 19 51" src="https://github.com/user-attachments/assets/e44c6c51-206c-4a02-9193-6d48e3a539f4">
