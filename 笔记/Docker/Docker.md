# 基本概念
## 镜像
**Docker 镜像**（`Image`），就相当于是一个 `root` 文件系统。
## 分层存储
在构建镜像的时候，需要额外小心，每一层尽量只包含该层需要添加的东西，任何额外的东西应该在该层构建结束前清理掉。
## 容器
容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的命名空间.
容器存储层的生存周期和容器一样，容器消亡时，容器存储层也随之消亡。因此，任何保存于容器存储层的信息都会随容器删除而丢失。
## 仓库
仓库名经常以 _两段式路径_ 形式出现，比如 `jwilder/nginx-proxy`，前者往往意味着 Docker Registry 多用户环境下的用户名，后者则往往是对应的软件名。但这并非绝对，取决于所使用的具体 Docker Registry 的软件或服务。
# Docker的安装配置
## 卸载旧版本
```bash
for pkg in docker \
           docker-engine \
           docker.io \
           docker-doc \
           docker-compose \
           podman-docker \
           containerd \
           runc;
do
    sudo apt remove $pkg;
done
```
## 安装
```bash
sudo apt update

sudo apt install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg


# 官方源
# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://mirrors.aliyun.com/docker-ce/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null


# 官方源
# $echo \
#   "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
#   (lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update

sudo apt install docker-ce docker-ce-cli containerd.io
```
## 换源
[Docker换源](https://zhuanlan.zhihu.com/p/32004414428)
## 可选操作
[可选设置](https://docker.it-docs.cn/manuals/engine_install_linux-postinstall)

# 使用镜像
## 获取
```bash
docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
docker pull --help(这是查看选项用法的命令)
```
## 运行
```bash
docker run -it --rm ubuntu:18.04 bash(在操作容器中会细讲)
```
解释:
- `-it`：这是两个参数，一个是 `-i`：交互式操作，一个是 `-t` 终端。我们这里打算进入 `bash` 执行一些命令并查看返回结果，因此我们需要交互式终端。    
- `--rm`：这个参数是说容器退出后随之将其删除。默认情况下，为了排障需求，退出的容器并不会立即删除，除非手动 `docker rm`。我们这里只是随便执行个命令，看看结果，不需要排障和保留结果，因此使用 `--rm` 可以避免浪费空间。
- `ubuntu:18.04`：这是指用 `ubuntu:18.04` 镜像为基础来启动容器。
- `bash`：放在镜像名后的是 **命令**，这里我们希望有个交互式 Shell，因此用的是 `bash`。
## 列出镜像
```bash
docker image ls
```
### 镜像体积
```bash
docker system df (查看镜像、容器、数据卷所占用的空间。)
```
### 虚悬镜像
这个镜像原本是有镜像名和标签的，原来为 `mongo:3.2`，随着官方镜像维护，发布了新版本后，重新 `docker pull mongo:3.2` 时，`mongo:3.2` 这个镜像名被转移到了新下载的镜像身上，而旧的镜像上的这个名称则被取消，从而成为了 `<none>`。除了 `docker pull` 可能导致这种情况，`docker build` 也同样可以导致这种现象。由于新旧镜像同名，旧镜像名称被取消，从而出现仓库名、标签均为 `<none>` 的镜像。这类无标签镜像也被称为 **虚悬镜像(dangling image)**
#### 查看
```bash
docker image ls -f dangling=true
```
#### 删除
```bash
docker image prune
```
### 中间层镜像
```bash
docker image ls -a (查看所有包括中间层的镜像)
```
### 列出部分镜像
```bash
docker image ls 仓库名[:标签]
docker image ls -f since/before=仓库名:标签
```
### 以特定格式显示
```bash
docker image ls -q (按照ID列出镜像)
docker image ls --format "{{.ID}}: {{.Repository}}" 以"ID:仓库名"的形式列出镜像
docker image ls --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}" 以表格的形式(列分别是ID,仓库名,标签)列出镜像
```
## 删除本地镜像
```bash
docker image rm [选项] <镜像1> [<镜像2> ...]
```
`<镜像>` 可以是 `镜像短 ID`、`镜像长 ID`、`镜像名` 或者 `镜像摘要`
### Untagged Deleted
因此当我们使用上面命令删除镜像的时候，实际上是在要求删除某个标签的镜像。所以首先需要做的是将满足我们要求的所有镜像标签都取消，这就是我们看到的 `Untagged` 的信息。因为一个镜像可以对应多个标签，因此当我们删除了所指定的标签后，可能还有别的标签指向了这个镜像，如果是这种情况，那么 `Delete` 行为就不会发生。所以并非所有的 `docker image rm` 都会产生删除镜像的行为，有可能仅仅是取消了某个标签而已。
当该镜像所有的标签都被取消了，该镜像很可能会失去了存在的意义，因此会触发删除行为。
## docker commit
```bash
docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]
```
`--author` 是指定修改的作者，而 `--message` 则是记录本次修改的内容
### 慎用 `docker commit`

使用 `docker commit` 命令虽然可以比较直观的帮助理解镜像分层存储的概念，但是实际环境中并不会这样使用。

首先，如果仔细观察之前的 `docker diff webserver` 的结果，你会发现除了真正想要修改的 `/usr/share/nginx/html/index.html` 文件外，由于命令的执行，还有很多文件被改动或添加了。这还仅仅是最简单的操作，如果是安装软件包、编译构建，那会有大量的无关内容被添加进来，将会导致镜像极为臃肿。

此外，使用 `docker commit` 意味着所有对镜像的操作都是黑箱操作，生成的镜像也被称为 **黑箱镜像**，换句话说，就是除了制作镜像的人知道执行过什么命令、怎么生成的镜像，别人根本无从得知。而且，即使是这个制作镜像的人，过一段时间后也无法记清具体的操作。这种黑箱镜像的维护工作是非常痛苦的。

而且，回顾之前提及的镜像所使用的分层存储的概念，除当前层外，之前的每一层都是不会发生改变的，换句话说，任何修改的结果仅仅是在当前层进行标记、添加、修改，而不会改动上一层。如果使用 `docker commit` 制作镜像，以及后期修改的话，每一次修改都会让镜像更加臃肿一次，所删除的上一层的东西并不会丢失，会一直如影随形的跟着这个镜像，即使根本无法访问到。这会让镜像更加臃肿。
## Dockerfile定制镜像
```bash
# 步骤如下
mkdir myimage
cd mynginx
touch Dockerfile
```
Dockerfile内容
```bash
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```
### FROM
指定 **基础镜像**，因此一个 `Dockerfile` 中 `FROM` 是必备的指令，并且必须是第一条指令。
除了选择现有镜像为基础镜像外，Docker 还存在一个特殊的镜像，名为 `scratch`。这个镜像是虚拟的概念，并不实际存在，它表示一个空白的镜像。
### RUN
用来执行命令行命令。其格式有两种：
- _shell_ 格式：`RUN <命令>`，就像直接在命令行中输入的命令一样。
- _exec_ 格式：`RUN ["可执行文件", "参数1", "参数2"]`，这更像是函数调用中的格式。
每一个 `RUN` 的行为：新建立一层，在其上执行命令，执行结束后，`commit` 这一层的修改，构成新的镜像。
```bash
rm -rf /var/lib/apt/lists/*
apt-get purge -y --auto-remove $buildDeps (清理无关文件)
```
### COPY
格式：
- `COPY [--chown=<user>:<group>] <源路径>... <目标路径>`
- `COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]`
`--chown=<user>:<group>` 选项来改变文件的所属用户及所属组
### CMD
这是启动命令
- `shell` 格式：`CMD <命令>`
- `exec` 格式：`CMD ["可执行文件", "参数1", "参数2"...]`
正确的做法是直接执行 `nginx` 可执行文件，并且要求以前台形式运行。比如：
```
CMD ["nginx", "-g", "daemon off;"]
```
### ENTRYPOINT
格式同上CMD

```DOCKERFILE
FROM ubuntu:18.04
RUN apt-get update \
    && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*
ENTRYPOINT [ "curl", "-s", "http://myip.ipip.net" ]
```

```bash
docker run myip -i
```
存在 `ENTRYPOINT` 后，`CMD` 的内容将会作为参数传给 `ENTRYPOINT`，而这里 `-i` 就是新的 `CMD`，因此会作为参数传给 `curl`，从而达到了我们预期的效果。
### WORKDIR
因此如果需要改变以后各层的工作目录的位置，那么应该使用 `WORKDIR` 指令。
```
WORKDIR /app

RUN echo "hello" > world.txt
```
如果你的 `WORKDIR` 指令使用的相对路径，那么所切换的路径与之前的 `WORKDIR` 有关：
```
WORKDIR /a
WORKDIR b
WORKDIR c
```
`RUN pwd` 的工作目录为 `/a/b/c`。
###  ENV
设置环境变量
格式有两种：
- `ENV <key> <value>`
- `ENV <key1>=<value1> <key2>=<value2>...`
### EXPOSE
格式为 `EXPOSE <端口1> [<端口2>...]`。

`EXPOSE` 指令是声明容器运行时提供服务的端口，这只是一个声明，在容器运行时并不会因为这个声明应用就会开启这个端口的服务。在 Dockerfile 中写入这样的声明有两个好处，一个是帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射；另一个用处则是在运行时使用随机端口映射时，也就是 `docker run -P` 时，会自动随机映射 `EXPOSE` 的端口。

要将 `EXPOSE` 和在运行时使用 `-p <宿主端口>:<容器端口>` 区分开来。`-p`，是映射宿主端口和容器端口，换句话说，就是将容器的对应端口服务公开给外界访问，而 `EXPOSE` 仅仅是声明容器打算使用什么端口而已，并不会自动在宿主进行端口映射。
### VOLUME
格式为：
- `VOLUME ["<路径1>", "<路径2>"...]`
- `VOLUME <路径>`
在 `Dockerfile` 中，我们可以事先指定某些目录挂载为匿名卷，这样在运行时如果用户不指定挂载，其应用也可以正常运行，不会向容器存储层写入大量数据。
### 构建镜像
```bash
docker build [选项] <上下文路径/URL/->
```
-t指定了最终镜像的名称 
#### 上下文
**构建时可用的全部文件集合**
.dockerignore文件是用于剔除不需要作为上下文传递给 Docker 引擎的文件。
-f ../Dockerfile.php` 参数指定某个文件作为 `Dockerfile`。
#### URL
https://github.com/docker-library/hello-world.git#master:amd64/hello-world
指定分支为 master，构建目录为 /amd64/hello-world/
#### 用给定的 tar 压缩包构建
如果所给出的 URL 不是个 Git repo，而是个 `tar` 压缩包，那么 Docker 引擎会下载这个包，并自动解压缩，以其作为上下文，开始构建。
#### 从标准输入中读取Dockerfile或上下文压缩包进行构建
```bash
docker build - < Dockerfile
docker build - < context.tar.gz
```
如果发现标准输入的文件格式是 gzip、bzip2 以及 xz 的话，将会使其为上下文压缩包，直接将其展开，将里面视为上下文，并开始构建