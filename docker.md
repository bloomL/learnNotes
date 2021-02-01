1. **docker ps**	所有运行的容器

   **docker inspect <friendly-name|container-id>**   provides more details about a running container, such as IP address.

   **docker logs <friendly-name|container-id>**

2. Redis is running, but is surprised that she cannot access it. The reason is that each container is sandboxed（沙盒）. If a service needs to be accessible by a process not running in a container, then the port needs to be **exposed** via the Host.

   -p <host-port>:<container-port> option.

   docker run **-d --name** redisHostPort **-p 6379:6379** redis

3. The problem with running processes on a fixed port is that you can **only run one** instance. Jane would prefer to run multiple Redis instances and configure the application depending on which port Redis is running on.

   docker run -d --name redisDynamic **-p 6379 **redis

   ![](C:\Users\l'g\AppData\Roaming\Typora\typora-user-images\image-20210125142740890.png)

4. data stored keeps being removed when she deletes and re-creates a container. Jane needs the data to be persisted and reused when she recreates a container.(删除并重新创建容器时，存储的数据将不断被删除。 Jane在重新创建容器时需要保留并重用数据。)

   Containers are designed to be stateless(无状态). Binding directories (also known as volumes) is done using the option 

   **-v <host-dir>:<container-dir>**

   When a directory is mounted, the files which exist in that directory on the host can be accessed by the container and any data changed/written to the directory inside the container will be stored on the host.（挂载目录后，容器可以访问主机上该目录中的文件，并且更改/写入容器内目录的所有数据都将存储在主机上）

   docker run -d  --name redisMapped **-v /opt/docker/data/redis:/data redis**

5. you should define an environment variable for NODE_ENV when running in production(在生产环境中运行时，应为NODE_ENV定义环境变量)

   Using **-e** option, you can set the name and value as -e NODE_ENV=production

   docker run -d --name my-production-running-app **-e NODE_ENV=production** -p 3000:3000 my-nodejs-app

6. Dockerfile01

   ```dockerfile
   FROM nginx:alpine
   COPY . /user/share/nginx/html
   
   ## The first line defines(定义) our base image. 
   ##The second line copies the content of the current directory into a particular location inside the container.（将当前目录的内容复制到容器内的特定位置。）
   ```

   **docker build -t <build-directory>**

   ![](C:\Users\l'g\AppData\Roaming\Typora\typora-user-images\image-20210125150706707.png)

   docker build -t webserver-image:v1 **.**

##### Dockerfile（Docker映像是基于Dockerfile的内容构建的）

```dockerfile
FROM 基于哪个镜像实现

MAINTAINER 镜像创建者

ENV 声明环境变量
##ENV JAVA_OPTS="-Xms128m -Xmx256m -Djava.security.egd=file:/dev/./urandom"

RUN 执行的命令

ADD 添加宿主机文件到容器，有需要解压的文件会自动解压

COPY 添加宿主机文件到容器

WORKDIR 工作目录

EXPOSE 容器内应用可使用的端口

CMD 容器启动后所执行的程序，如果执行docker run 后面跟启动命令会被覆盖掉

ENTRYPOINT 与CMD一样，但docker run 不会覆盖，如果需要覆盖可使用参数-entrypoint来覆盖

VOLUME 将宿主机的目录挂载到容器
```

#### Building Container Images

1. To define a base image we use the instruction **FROM <image-name>:<tag>**

   ```dockerfile
   FROM nginx:1.11-alpine
   ```

2. **COPY <src> <dest>**

   ```dockerfile
   COPY index.html /usr/share/nginx/html/index.html
   ```

3. **EXPOSE <port> ** ports should be open and can be bound(绑定) to. You can define multiple ports on the single command, for example, *EXPOSE 80 433* or *EXPOSE 7000-8000*

   ```dockerfile
   EXPOSE 80
   ```

4. The **CMD** line in a Dockerfile defines the default command to run when a container is launched.(Dockerfile中的CMD行定义了启动容器时要运行的默认命令。)

   ```dockerfile
   CMD ["nginx", "-g", "daemon off;"]
   ```

   An alternative approach to CMD is ENTRYPOINT. While a CMD can be overridden when the container starts, a ENTRYPOINT defines a command which can have arguments passed to it when the container launches.

   (CMD的替代方法是**ENTRYPOINT**。可以在启动容器时覆盖CMD，但是ENTRYPOINT定义了一个命令，可以在启动容器时将参数传递给它。)

5. `docker build` to turn it into(转换) an image **docker build -t <build-directory>**

   ```docker
   docker build -t firstfile:v1 .
   ```

6. 还需要创建应用程序运行所在的基本目录，Using the **RUN <command>** we can execute commands as if they're running from a command shell, by using mkdir we can create the directories where the application will execute from（使用RUN <command>，我们可以像执行命令外壳一样执行命令，通过使用**mkdir**，我们可以创建应用程序将从中执行的目录）**mkdir -p** 创建多级目录

   ```dockerfile
   RUN mkdir -p /src/app
   ```

7. We can define a working directory using **WORKDIR <directory>** to ensure that all future commands are executed from the directory relative to our application.（我们可以使用WORKDIR <directory>定义一个工作目录，以确保所有将来的命令都从相对于我们的应用程序的目录中执行。）

   ```dockerfile
   WORKDIR /src/app
   ```

#### Data Containers

is to be a place to store/manage data.(存储/管理数据的地方。)

1. docker create -v /config --name dataContainer busybox

   **-v option** 定义其他容器将在哪里读取/保存数据

2. docker **cp** config.conf dataContainer:/config/

3. docker run **--volumes-from** dataContainer ubuntu ls /config

4. docker **export** dataContainer > dataContainer.tar

5. docker **import** dataContainer.tar

#### Docker Compose

1. Defining First Container

2. Defining Settings

   ```yaml
   web:
     build: .
     ports:
       - "3001"
     links:
       - redis
   ```

3. Defining Second Container

   ```yaml
   redis:
     image: redis:alpine
     volumes:
       - /var/redis/data:/data
   ```

4. docker-compose **up** -d

5. docker-compose **ps**

   docker-compose **logs**

6. docker-compose **scale** web=3

   **scale**指定服务，然后指定所需的实例数

7. docker-compose stop

8. docker-compose rm

##### 进入容器终端并且的保留为容器终端的输入形式(-it和bash的结合作用)

docker exec -it  CONTAINER_ID   bash

docker exec -it redis-server-new bash

##### 删除none镜像

docker images | grep 'none' | awk  '{print $3}'

docker rmi $(docker images | grep 'none' | awk  '{print $3}')

##### awk