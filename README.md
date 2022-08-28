# Docker

### 1. What is docker?

```
Giúp quá trình triển khai phần mềm của chúng ta đơn giản hơn
Giúp đóng gói sản phẩm có 2 tác dụng:
+ Không lo bị lộ mã nguồn
+ Khách hàng chạy ứng dụng thông qua 1 hộp chứa

```

### 2. Getting started

```
Command: `docker run -d -p 80:80 docker/getting-started`

+ -d -> Chạy vùng chứa ở chế độ tách rời (trong nền)
+ -p 80:80 -> Ánh xạ cổng 80 của máy chủ tới cổng 80 trong container. Để truy cập hướng dẫn, hãy mở trình duyệt web và điều hướng đến http://localhost:80. Nếu bạn đã có một dịch vụ đang lắng nghe trên cổng 80 trên máy chủ của mình, bạn có thể chỉ định một cổng khác.
+ docker/getting-started -> Chỉ định hình ảnh để sử dụng.

! Mẹo
Command: `docker run -dp 80:80 docker/getting-started`

Conclusion: Kéo 1 image (project) từ docker hub về và nó đóng gói lại bên trong docker desktop => gọi nó là 1 container
```

### 3. What is a container?

```
Hoạt động độc lập với máy tính 
Có thể chạy trên máy cục bộ, máy ảo hoặc triển khai lên đám mây.
```

### 4. What is a container image?

```
Là thành phần cốt lõi của docker, nhờ có image thì docker mới biết làm sao để chạy ứng dụng, build ra một hộp đen.
```

```
Tạo ra 1 tệp cấu hình với tên Dockerfile:

FROM node:12-alpine
RUN apk add --no-cache python2 g++ make
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
EXPOSE 3000

Explain:
- FROM: Là câu lệnh chuẩn bị môi trường (kéo về container đóng gói)
- WORKDIR: Tạo ra đường dẫn cho hộp chứa đó
- COPY: Sao lưu source vào thư mục app
- yarn install: Cài đặt các deps => node_modules
- --production: Tối ưu hóa việc các pakages được cài đặt trong node_modules
- CMD (command): Hướng dẫn cho docker làm sao có thể chạy được image này
```

### 5. Build docker image

```
Command: `docker build -t getting-started .`

+ Lệnh này đã sử dụng Dockerfile để xây dựng một hình ảnh vùng chứa mới.
+ xây dựng hình ảnh vùng chứa bằng lệnh `docker build` 
+ -t là cờ gắn thẻ hình ảnh
```

### 6. Docker ps / stop / rm

```
- ps => process status: trạng thái của tiến trình

Command: `docker ps` && `docker ps -a` (xem container)

Command: `docker image ls` (xem ảnh có trong ứng dụng)

Command: `docker stop [containerID]`
```

### 7. Share the apps

```
- Tag: Định danh cái version phần mềm sử dụng

Command: `docker tag image:tag [containerID]/name_repo

Command: `docker push [containerID]/name_repo`
```

### 8. Persist the DB

```
- Container volumes: Để lưu trữ data trong 1 container thì tách riêng cục lưu trữ ra ngoài container 

- Khởi tạo một volume: Tạo dữ liệu ra bên ngoài container của docker 
Command: `docker volume create todo-db` 

- Khởi tạo vùng chứa 
Command: `docker run -dp 3000:3000 -v todo-db:/etc/todos [tên ảnh]:tag`
```

### 9. Dive into the volume

```
Command: `docker volume inspect todo-db`
```

### 10. Use bind mounts

```
Problems: `Sửa đổi mã nguồn thì docker thay đổi ngay lập tức mà không cần build lại`
```

```
Syntax: docker run -dp 3000:3000`
     -w /app -v "$(pwd):/app" `
     node:12-alpine `
     sh -c "yarn install && yarn run dev"
```

```
Command: `docker run -dp 3000:3000 `
     -w /app -v "$(pwd):/app" `
     node:12-alpine `
     sh -c "yarn install && yarn run dev"

-dp 3000:3000- giống như trước. Chạy ở chế độ tách rời (nền) và tạo ánh xạ cổng
-w /app- đặt “thư mục làm việc” hoặc thư mục hiện tại mà lệnh sẽ chạy từ đó
-v "$(pwd):/app"- bind gắn kết thư mục hiện tại từ máy chủ lưu trữ trong vùng chứa vào /appthư mục
node:12-alpine- hình ảnh để sử dụng. Lưu ý rằng đây là hình ảnh cơ sở cho ứng dụng của chúng tôi từ Dockerfile
sh -c "yarn install && yarn run dev"- lệnh. Chúng tôi đang bắt đầu một trình bao bằng cách sử dụng sh(alpine không có bash) và chạy yarn installđể cài đặt tất cả các phụ thuộc và sau đó chạy yarn run dev. Nếu chúng ta nhìn vào package.json, chúng ta sẽ thấy rằng devtập lệnh đang bắt đầu nodemon 
```

```
Purpose: Test ứng dụng tại local 
```

### 11. Multi-container apps

```
Purpose: `Mỗi container nên làm một việc và làm tốt điều đó`
```

```
Container networking: `Nếu hai vùng chứa trên cùng một mạng, chúng có thể nói chuyện với nhau. Nếu không, họ không thể.`
```

```
Step start MySQL:

1. Create the network

`docker network create todo-app`

2. Start a MySQL container and attach it to the network

`docker run -d `
     --network todo-app --network-alias mysql `
     -v todo-mysql-data:/var/lib/mysql `
     -e MYSQL_ROOT_PASSWORD=secret `
     -e MYSQL_DATABASE=todos `
     mysql:5.7`

3. Verify database connection 

`
    docker exec -it <mysql-container-id> mysql -u root -p
    mysql> SHOW DATABASES;
    mysql> exit
`
```

### 12. Connect to MySQL

```
1. Start a new container using the nicolaka/netshoot image. Make sure to connect it to the same network.

`docker run -it --network todo-app nicolaka/netshoot`

Note: Images netshoot => Cho biết địa chỉ IP 

2. Look up the IP address for the hostname mysql.

`dig mysql`
```

### 13. Run app with MYSQL

```
Command: `docker run -dp 3000:3000 `
   -w /app -v "$(pwd):/app" `
   --network todo-app `
   -e MYSQL_HOST=mysql `
   -e MYSQL_USER=root `
   -e MYSQL_PASSWORD=secret `
   -e MYSQL_DB=todos `
   node:12-alpine `
   sh -c "yarn install && yarn run dev"`

Command: `docker exec -it <mysql-container-id> mysql -p todos`
```

### 14. Docker compose

```
1. Install Docker compose

2. Define the app service

version: "3.7"

services:
app:
     image: node:12-alpine
     command: sh -c "yarn install && yarn run dev"
     ports:
     - 3000:3000
     working_dir: /app
     volumes:
     - ./:/app
     environment:
     MYSQL_HOST: mysql
     MYSQL_USER: root
     MYSQL_PASSWORD: secret
     MYSQL_DB: todos

3. Define the MySQL service

...
  mysql:
    image: mysql:5.7
    volumes:
      - todo-mysql-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:

4. Run the application stack

Command: `docker-compose up -d`
```