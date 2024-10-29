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

