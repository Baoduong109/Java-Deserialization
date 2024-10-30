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

Ở level 2, chương trình có thêm các class khác, ở đây ta sẽ chú ý đến class ```MyHTTPClient```, trong class có sử dụng chứng năng kiểm tra ```HTTPConnection``` bằng cách sử dụng OS command ```ping``` và ```curl```(dòng 16 và 28)

<img width="1330" alt="Ảnh chụp Màn hình 2024-10-29 lúc 23 11 19" src="https://github.com/user-attachments/assets/d2f3e61c-d3bd-4d0c-ac38-3a3133f69b4c">

Người dùng sẽ gửi cho server giá trị host thông qua serialize data, server sẽ tiến hành deserialize data để đọc và truyền giá trị này vào OS command.

Ý tưởng khai thác: Untrusted data rơi vào magic method ```readObject```, chương trình sẽ deserialize data và truyền giá trị host vào OS command -> Các yếu tố này kết hợp dễ dàng tấn công OS command injection thông qua biến host.

Copy file ```MyHTTPClient.java``` sang thư mục ```exploit-tool```, VS Code sẽ báo lỗi vì class ```MyHTTPClient``` được kế thừa từ ```HTTPConnection``` nên ta copy luôn cả file ```HTTPConnection.java```.

<img width="1407" alt="Ảnh chụp Màn hình 2024-10-29 lúc 23 12 18" src="https://github.com/user-attachments/assets/d4ebb1b5-a2f4-48a7-9555-2b33f503e656">

Sau đó ta tiến hành khởi tạo các tham số để tạo ra payload.

Tạo object ```MyHTTPClient client = new MyHTTPClient("xxxx; id");```. Sau đó in ra output của client để lấy được payload.

<img width="974" alt="Ảnh chụp Màn hình 2024-10-30 lúc 12 00 55" src="https://github.com/user-attachments/assets/b5622277-a78e-42ba-95d2-db93d748460a">

Tiếp theo là lấy payload gán vào cookie ở level 2.

<img width="1078" alt="Ảnh chụp Màn hình 2024-10-29 lúc 23 16 06" src="https://github.com/user-attachments/assets/9c8ee7ed-0165-4a75-9bda-beb59371e19e">

Server sẽ trả về cho ta là Please don't hack me. Lúc này ta cần debug xem đã xảy ra chuyện gì.

Vào ```MyHTTPClient.java```, đặt 1 breakpoint ở dòng 28 xem ```readObject()``` có được gọi đến và các giá trị ta tạo có được gán cho biến ```host``` hay không.

<img width="1232" alt="Ảnh chụp Màn hình 2024-10-29 lúc 23 16 38" src="https://github.com/user-attachments/assets/dfa8354b-3b1e-4cd7-b41e-f7e8153fec1c">

Sau đó rebuild lại level 2 và reload lại trang web. Sau khi reload thì chương trình sẽ dừng ngay ở vị trí ta đặt breakpoint.

<img width="1387" alt="Ảnh chụp Màn hình 2024-10-29 lúc 23 17 18" src="https://github.com/user-attachments/assets/6ce8ccf8-02ff-48ba-aeef-3742be4441b6">

Kiểm tra thì các giá trị ta đặt đã được gán vào biến host.

<img width="332" alt="Ảnh chụp Màn hình 2024-10-29 lúc 23 18 38" src="https://github.com/user-attachments/assets/a8fe99aa-052a-4489-bbcd-cd90655461c7">

Vậy là chương trình đã được gọi đến ```readObject()``` nhưng đã bị huỷ ngay sau đó. Vậy có nghĩa là ta sẽ sử dụng được các lệnh OS command. Ta sẽ sử dụng 1 cách khác là đưa những thông tin ta muốn ra bên ngoài server. Ở đây ta sẽ sử dụng webhook là nơi sẽ đón nhận các gói tin ta gửi đi.

<img width="1192" alt="Ảnh chụp Màn hình 2024-10-30 lúc 12 01 25" src="https://github.com/user-attachments/assets/d437c10f-8c63-4a54-99a5-87f2595043bd">

Lý do mình dùng wget thay vì curl tại vì server không có curl :))))

Rebuild ```exploit-tool``` và copy gán vào cookie level 2. Sau đó try cập vào webhook. Ta sẽ thấy được 1 request được gửi tới.

<img width="1440" alt="Ảnh chụp Màn hình 2024-10-29 lúc 23 21 28" src="https://github.com/user-attachments/assets/bdd1612c-2c08-4ee9-85d6-73a5f0a9a545">

Tương tự, thêm ```method``` và ```body-file``` cho wget. Rebuild và gán lại cho cookie.

<img width="1122" alt="Ảnh chụp Màn hình 2024-10-30 lúc 12 01 51" src="https://github.com/user-attachments/assets/682ff83e-f1ff-4ea8-82de-a856673335ad">

Ở webhook ta sẽ nhận được 1 request với method POST, các values trong đó sẽ chứa các tên thư mục root. => RCE thành công.

## Level 3:

Ở level 3 thì cũng sẽ tương tự level 2, nhưng lúc này method ```readObject()``` đã thay đổi thành ```connect```.

<img width="881" alt="Ảnh chụp Màn hình 2024-10-29 lúc 23 26 58" src="https://github.com/user-attachments/assets/8018f635-1bac-4b99-b9e2-09f53069a7bd">

Làm thế nào để có thể gọi đến ```connect```. Ta search xem có class nào gọi đến hàm này hay không.

<img width="328" alt="Ảnh chụp Màn hình 2024-10-29 lúc 23 27 18" src="https://github.com/user-attachments/assets/f95a42f9-fbc8-414a-8058-f1d429bb602c">

Ta thấy được có class ```TestConnection``` có magic method ```readObject()``` gọi đến hàm này.

<img width="966" alt="Ảnh chụp Màn hình 2024-10-29 lúc 23 27 26" src="https://github.com/user-attachments/assets/a8aea351-6d98-4200-a1cd-bb7e697ae0cc">

Ý tưởng để khai thác level này là ta sẽ lợi dụng ```MyHTTPClient``` có hàm ```connect()``` execute untrusted data. Tạo thêm object ```TestConnection``` vì trong đó có ```readObject()``` để gọi hàm ```connect()```

Để cho VersionUID không bị lỗi thì ta nên xoá các file của level trước và copy các file cần tạo payload sang.

Sau đó khởi tạo các object như bên dưới.

<img width="1371" alt="Ảnh chụp Màn hình 2024-10-29 lúc 23 30 52" src="https://github.com/user-attachments/assets/333073dc-934f-47ce-be1c-3d9262acd2ab">

Cookie ở dạng serialize như sau:

<img width="1437" alt="Ảnh chụp Màn hình 2024-10-29 lúc 23 31 13" src="https://github.com/user-attachments/assets/f094d53a-600f-48e1-98f0-c766d988289d">

Gán vào cookie user ở level 3, vào webhook là ta sẽ thấy được request từ server.

<img width="1440" alt="Ảnh chụp Màn hình 2024-10-29 lúc 23 33 51" src="https://github.com/user-attachments/assets/49b38680-f278-4bd9-aad9-5b75b0219727">

Tương tự như level 2, thêm ```method``` và ```body-file``` cho wget là thành công RCE.

## Level 4:

Ta chỉ thấy 2 file chính là ```HelloServlet.java``` và ```User.java```

Nếu như ở những các level trước có những class mà ta có thể lợi dụng để tấn công. Vậy level này thì sao? Ta chỉ có mỗi class ```User```.

Nhưng ngoài code của anh dev, thì trong chương trình này còn có code của thư viện. Liệu ta có thể tìm những class nguy hiểm hay gadget chain trong thư viện để thực hiện tấn công không.

Đầu tiên ta phải biết chương trình đang sử dụng thư viện gì. Thông tin này được thể hiện trong file ```pom.xml```.

<img width="978" alt="Ảnh chụp Màn hình 2024-10-29 lúc 23 36 39" src="https://github.com/user-attachments/assets/243fc59b-76af-4751-a7e1-e463c345d5f8">

Ta thấy chương trình đang sử dụng thư viện ```commons-collections version 3.1```.

Để tìm kiếm 1 gadget chain trong thư viện mất rất nhiều thời gian, nên ta sẽ sử dụng ysoerial để tạo ra gadget chain.

<img width="1436" alt="Ảnh chụp Màn hình 2024-10-29 lúc 23 37 17" src="https://github.com/user-attachments/assets/7dc45274-e2cc-4f1a-9426-e1ca5b9745fb">

Cú pháp để sử dụng ysoserial:

java -jar ysoserial.jar [payload] '[command]'

<img width="1195" alt="Ảnh chụp Màn hình 2024-10-29 lúc 23 39 54" src="https://github.com/user-attachments/assets/f4360dad-29f0-488f-923d-ebad84b1f16b">

Ta sẽ sử dụng ysoserial với payload là CommonsCollection5.

<img width="1243" alt="Ảnh chụp Màn hình 2024-10-29 lúc 23 41 23" src="https://github.com/user-attachments/assets/befc3605-b3f4-4831-94ac-303b553c3dae">

Và phải endcode base64 nữa nhé :))))

<img width="1247" alt="Ảnh chụp Màn hình 2024-10-29 lúc 23 41 50" src="https://github.com/user-attachments/assets/21b72614-cc11-4a1e-97a7-d42b2b03f763">

Cuối cùng ta gán giá trị này vào cookie user nữa là xong. Webhook sẽ nhận được request mà server gửi đến.

<img width="1440" alt="Ảnh chụp Màn hình 2024-10-29 lúc 23 42 19" src="https://github.com/user-attachments/assets/8a990266-938f-46b7-87dc-61880949b2a6">

=> Thành công RCE.
