# docker-vuejs-demo

## Project setup

```
npm install
```

### Compiles and hot-reloads for development

```
npm run serve
```

### Compiles and minifies for production

```
npm run build
```

### Run your tests

```
npm run test
```

### Lints and fixes files

```
npm run lint
```

### Customize configuration

See [Configuration Reference](https://cli.vuejs.org/config/).

## docker vuejs demo

> 建一个简单的 VueJS Web 应用程序，如何将 VueJS Web 应用程序部署到 Docker，如何创建 Docker 映像并从映像启动容器，如何管理 Docker 容器，以及如何使用 NGINX 在生产中部署 VueJS 应用程序等

- 前提
  - mac 电脑
  - 安装了 docker nodejs npm vue-cli

### 1 vue-cli 初始化新的 vue 项目

## `vue create docker-vuejs-demo` 完成后 本地运行项目：`cd docker-vuejs-demo/ && npm run serve` , 浏览器里面查看效果： `http://localhost:8080/` 可以选择性改一下里面的 Helloworld.vue 后再看效果

### 2 编写 Dockefile

- docker-vuejs-demo 根目录下创建`Dockerfile` 里面内容为：

```

# Choose the Image which has Node installed already
FROM node:lts-alpine

# install simple http server for serving static content
RUN npm install -g http-server

# make the 'app' folder the current working directory
WORKDIR /app

# copy both 'package.json' and 'package-lock.json' (if available)
COPY package*.json ./

# install project dependencies
RUN npm install

# copy project files and folders to the current working directory (i.e. 'app' folder)
COPY . .

# build app for production with minification
RUN npm run build

EXPOSE 8080
CMD [ "http-server", "dist" ]
```

### 3 根据书写好的 Dockerfile build 一个 vuejs 的 docker 镜像， 推荐 用户名/项目名, 项目根目录下执行

`docker build -t alex/docker-vuejs .` 里面 .代表 docker 去当前目录下查找 Dockerfile 找到了就根据里面的内容进行 build.

- 如何知道成功了？ 执行完上面的命令后会很友好的输出当前进行的进度`Step 1/9 : FROM node:lts-alpine`，中间 build 失败，重新运行上面这条命令， 会在原来基础上继续 build, 最后看到`Successfully built b7aff97e3c0a Successfully tagged alex/docker-vuejs:latest`

- `docker images` 查看已经 build 好的 image

## 4 根据 build 好的 vue 项目的镜像， 启动容器

`docker run -it -p 8081:8080 -d --name docker-vuejs alex/docker-vuejs`

- 解释：

```
-it  –  This flag sets the container in Interactive mode and allocate a Dedicated TTY id for later SSHing.

-d –  This flag sets the container to run in the background.

-p 8081:8080 – Port Forwarding Between Host and the Container. Right to the colon is a container and Left to the colon is Host. 8081 is the Host and 8080 is the container Port.

–name – Once started docker daemon assigns a random string name to the container. But i recommend defining a name can be handy way to add meaning to a container.

docker-vuejs – Name of the container we are starting ( Replacement of Container ID).

alex/docker-vuejs – 将要创建容器的image的名字.
```

## 5 验证 vuejs 镜像是否在容器中启动

- `docker ps`可以看到， 浏览器里面打开， 为什么是 8081？ 因为上面命令里面指定了
  `http://localhost:8081` 正确看到效果，成功。

### 还可以做什么？用 nginx 启动

- 类似的步骤: 拉取 nginx 镜像， 编写 Dockerfile build 镜像 docker 运行镜像， 浏览器查看效果

1. 创建 DockerFile

```
# build stage
FROM node:lts-alpine as build-stage
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# production stage
FROM nginx:stable-alpine as production-stage
COPY --from=build-stage /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

2. 用 Dockerfile build 一个 vue-nginx 镜像
   `docker build -t alex/docker-vuejs-nginx .`
   发现 上次以及安装了 nodejs 这次 build 的时候 就很快了
3. 根据刚才 build 好的 vue-nginx 镜像 起一个容器

- 可能会 build 失败，重新执行一次命令就好了，失败的原因可能是我之前安装过一次 ，node_modules 里面有内容了，
- 执行这个命令`docker run -it -p 8080:80 -d --name docker-vuejs-nginx alex/docker-vuejs-nginx`

4. 验证`http://localhost:8080` 完成
5. `docker ps` 查看正在运行的容器
6. 停止一个容器 `docker ps 看CONTAINER ID`然后 `docker stop + 要停止的 CONTAINER ID`
7. 再次`docker ps` 查看正在运行的容器 ，看是否停止成功

## 参考资料

- vue-cli 官网
- nodejs 官网
- docker 官网
- [docker-vuejs](https://www.middlewareinventory.com/blog/docker-vuejs/)
