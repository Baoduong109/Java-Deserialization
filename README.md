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

Tuy nhiên, khi server tiến hành hiển thị giá trị ```user``` dưới dạng string ở dòng 51 đã kích hoạt magic method ```toString()```.

Khi ```toString()``` được kích hoạt, chương trình sẽ chạy lệnh OS command ```whoami```.

<img width="880" alt="Ảnh chụp Màn hình 2024-10-29 lúc 22 19 51" src="https://github.com/user-attachments/assets/e44c6c51-206c-4a02-9193-6d48e3a539f4">

Ý tưởng để khai thác level này là ta sẽ lợi dụng class ```Admin``` để tạo ra 1 deserialize data có chức năng chạy được OS command mà ta mong muốn. Ta đã được biết khi server hiển thị giá trị ```user``` dưới dạng string thì magic method ```toString()``` được kích hoat.

Để tạo payload, ta sẽ sử dụng ```deserialize-exploit-tool```. Copy class ```Admin``` từ level 1.

<img width="312" alt="Ảnh chụp Màn hình 2024-10-29 lúc 22 22 47" src="https://github.com/user-attachments/assets/8eab6e5c-5258-4f88-9cef-e8b9d8746397">

Sau đó thay đổi giá trị ```whoami``` trong class ```Admin``` thành lệnh ta muốn.

<img width="884" alt="Ảnh chụp Màn hình 2024-10-29 lúc 22 24 29" src="https://github.com/user-attachments/assets/c38d6c0d-e4a6-4edb-a386-94997e160cd6">

Vào class ```HelloServlet```, Khai báo ```User user = new Admin();```

<img width="1312" alt="Ảnh chụp Màn hình 2024-10-29 lúc 23 09 58" src="https://github.com/user-attachments/assets/9f569c15-e5c0-4cb6-8ee9-98c9d80443fb">

Sau đó rebuild chương trình và truy cập vào browser để nhận serialize data.

<img width="798" alt="Ảnh chụp Màn hình 2024-10-29 lúc 22 26 17" src="https://github.com/user-attachments/assets/ad881786-849c-4f54-b22d-c3e4099b722e">

Mỗi lần ta thay đổi code là ta cần phải rebuild lại chương trình nên ta cần ghi chú vào phần serialize data để biết được data đã được thay đổi.

<img width="1437" alt="Ảnh chụp Màn hình 2024-10-29 lúc 22 27 37" src="https://github.com/user-attachments/assets/17dbf360-7da6-4cc7-943b-8b8e7bf4f9dc">

Sử dụng cookie này gửi để server bằng cách gán vào giá trị user trong cookie của levle 1.

<img width="1439" alt="Ảnh chụp Màn hình 2024-10-29 lúc 22 28 03" src="https://github.com/user-attachments/assets/2fefcdf8-3d5b-4a26-8b86-95378275e094">
<img width="1440" alt="Ảnh chụp Màn hình 2024-10-29 lúc 22 28 14" src="https://github.com/user-attachments/assets/5807696e-0007-448c-8b65-f64c8a34421e">

Sau đó reload lại trang để xem kết quả.

<img width="1241" alt="Ảnh chụp Màn hình 2024-10-29 lúc 22 45 54" src="https://github.com/user-attachments/assets/0a565fc6-f9fa-4277-8956-73e3c853c569">

Server sẽ bảo ta rằng "Please don't hack me". Ta cần kiểm tra xem chuyện gì đã xảy ra. Tại sao server không trả về kết quả như mong muốn. 

Truy cập vào log của docker xem đã có chuyện gì.

<img width="1423" alt="Ảnh chụp Màn hình 2024-10-29 lúc 22 46 28" src="https://github.com/user-attachments/assets/e1012173-3290-4673-827c-e0b924fbb22b">

Log hiển thị là class cục bộ không tương thích, VersionUID không trùng khớp. Lúc này ta cần phải kiểm tra class ```User``` của level 1 và exploit-tool.

<img width="1440" alt="Ảnh chụp Màn hình 2024-10-29 lúc 22 47 05" src="https://github.com/user-attachments/assets/a32c9ad0-0d43-4526-a932-7a69f9a5c2c9">

Vâng, có 1 sự khác biệt nhỏ. Ở exploit-tool thiếu ```getName``` làm cho VersionUID không trùng khớp. Bây giờ ta chỉ cần copy class ```User``` của level 1 sang exploit-tool và rebuild lại chươn trình.

<img width="1440" alt="Ảnh chụp Màn hình 2024-10-29 lúc 22 48 47" src="https://github.com/user-attachments/assets/fb504de3-74af-458d-bdab-f9b41dfca57d">

Mình đã đánh giấu serialize data này là 1.2. Copy và gán lại cho cookie user.

<img width="963" alt="Ảnh chụp Màn hình 2024-10-29 lúc 22 49 03" src="https://github.com/user-attachments/assets/f6b2f626-5905-4005-bde8-a50fe7774f14">

Thành công chạy lệnh OS command mong muốn. => RCE thành công.

## Level 2:

