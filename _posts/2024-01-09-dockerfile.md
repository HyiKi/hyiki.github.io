---
title: Dockerfile 指令中文手册
date:  2024-01-09 20:30:00 +0800
category: Original
tags: [docker]
excerpt: 自动构建镜像必读的 Dockerfile 指令中文手册
---

* content
{:toc}

# Dockerfile 中文手册

Docker可以通过读取Dockerfile中的指令自动构建镜像。 Dockerfile是一个文本文件，包含用户可以在命令行上调用的所有命令，以组装镜像。

## Linux 镜像推荐

要在Dockerfile中拉取最小的Linux镜像，你可以使用"FROM"指令，并指定所需的Linux镜像作为基础。以下是一个示例：

```dockerfile
FROM alpine:latest
```

我们使用的是Alpine Linux镜像，它以其小巧和精简的特性而闻名。`Alpine` 操作系统是一个面向安全的轻型 `Linux` 发行版。它不同于通常 `Linux` 发行版，`Alpine` 采用了 `musl libc` 和 `busybox` 以减小系统的体积和运行时资源消耗，但功能上比 `busybox` 又完善的多，因此得到开源社区越来越多的青睐。

## Dockerfile 初见

1. 创建一个空文件夹

   ```shell
   mkdir mydocker
   ```

2. 自定义dockerfile

   * 编辑dockerfile

   ```shell
   cd mydocker
   vim dockerfile
   ```

   * 保存dockerfile

   ```dockerfile
   FROM alpine
   ENTRYPOINT ["echo", "Hello"]
   CMD ["world"]
   ```

3. 生成镜像 images

    ```shell
    docker build -t myimage .
    ```

4. 运行镜像

    ```shell
    # docker run --rm myimage
    Hello world
    ```

   ```shell
   # docker run --rm myimage Hyiki
   Hello Hyiki
   ```

## Dockerfile 指令

### FROM 基础镜像

`FROM`命令是Dockerfile中的一个必需命令，用于指定基础镜像。它的语法如下：

```
FROM <基础镜像>
```

`FROM`命令指定了构建新镜像的基础，它会从指定的基础镜像开始构建新的镜像。基础镜像是一个已经存在的镜像，它包含了操作系统和其他预安装的软件。

例如，下面是一个使用`FROM`命令指定基础镜像的示例：

```dockerfile
FROM ubuntu:latest
```

这个命令指定了基础镜像为`ubuntu:latest`，表示将以最新版本的Ubuntu镜像作为基础构建新的镜像。

在Dockerfile中，`FROM`命令通常是作为第一个命令出现的，因为它定义了构建过程的起点。接下来的命令将基于这个基础镜像进行操作，例如安装软件、复制文件等。

需要注意的是，`FROM`命令中的基础镜像可以是官方的Docker镜像，也可以是自定义的镜像。通常情况下，选择一个合适的基础镜像对于构建高效、可靠的镜像非常重要。

### RUN 执行命令

`RUN`命令是Dockerfile中的一个常用命令，用于在镜像构建过程中执行命令。它的语法如下：

```
RUN <命令>
```

`RUN`命令可以执行各种命令，包括安装软件包、运行脚本、创建文件夹等。每个`RUN`命令都会在镜像中创建一个新的中间层，并在其中执行指定的命令。

以下是一些使用`RUN`命令的示例：

1. 安装软件包：

   ```dockerfile
   RUN apt-get update && apt-get install -y package
   ```

   这个命令使用`apt-get`包管理器更新软件源并安装名为`package`的软件包。

2. 运行脚本：

   ```dockerfile
   RUN chmod +x script.sh && ./script.sh
   ```

   这个命令首先赋予`script.sh`脚本执行权限，然后在镜像中运行该脚本。

3. 创建文件夹：

   ```dockerfile
   RUN mkdir /app
   ```

   这个命令在镜像中创建一个名为`/app`的文件夹。

`RUN`命令可以多次使用，每个命令都会在镜像的新中间层中执行。但是请注意，每个`RUN`命令都会创建一个新的镜像层，因此过多的`RUN`命令可能导致镜像层的增加，增加镜像的大小。

在编写`RUN`命令时，可以使用反斜杠`\`将命令分为多行。例如：

```dockerfile
RUN apt-get update && \
    apt-get install -y package && \
    apt-get clean
```

这样可以使Dockerfile更易于阅读和维护。

#### 清理APT包

`apt-get purge -y --auto-remove`和`apt-get clean`是两个用于清理APT包管理系统的命令，它们在Docker镜像构建过程中有不同的用途。

1. `apt-get purge -y --auto-remove`命令用于完全删除指定的软件包及其相关依赖项。它的选项含义如下：
   * `-y`：自动回答"yes"，无需手动确认删除操作。
   * `--auto-remove`：自动移除不再需要的依赖项。

   此命令适用于在构建镜像时需要删除不再需要的软件包和其依赖项的情况。它会彻底清理掉指定软件包及其相关的文件和配置。

2. `apt-get clean`命令用于清理APT缓存，即删除已下载的软件包文件。它的作用是释放磁盘空间，但不会删除软件包及其依赖项。这个命令没有选项，只需直接运行即可。

   这个命令适用于在构建镜像时需要减小镜像大小，删除已下载的软件包文件以节省磁盘空间的情况。

在Docker镜像构建过程中，通常在安装软件包后，使用`apt-get purge -y --auto-remove`命令删除不再需要的软件包及其依赖项，然后使用`apt-get clean`命令清理APT缓存，以减小镜像的大小。

需要注意的是，这些命令应该在同一个`RUN`命令中连续执行，以避免在不同的镜像层中产生多余的文件。例如：

```dockerfile
RUN apt-get update && \
    apt-get install -y package && \
    apt-get purge -y --auto-remove && \
    apt-get clean
```

这样可以确保在同一个镜像层中执行清理操作，避免增加镜像的层数和大小。

### CMD 容器启动命令

`CMD`命令是Dockerfile中的一个关键命令，用于设置容器启动时要执行的默认命令。它定义了容器镜像被实例化为容器时要运行的默认命令或可执行文件。

`CMD`命令有两种不同的形式：shell 形式和 exec 形式。

1. Shell 形式：

   ```dockerfile
   CMD <命令>
   ```

   在 Shell 形式中，`<命令>`可以是任何可在容器内部执行的命令，例如启动应用程序的命令或执行脚本的命令。这个命令将在容器启动时作为默认命令执行。如果在 Dockerfile 中有多个 CMD 命令，只有最后一个 CMD 命令会生效。

   示例：

   ```dockerfile
   CMD python app.py
   ```

   上述命令将在容器启动时默认执行 `python app.py` 命令。

2. Exec 形式：

   ```dockerfile
   CMD ["可执行文件", "参数1", "参数2", ...]
   ```

   在 Exec 形式中，命令和参数被作为 JSON 数组提供。这种形式更常用，它避免了使用 shell 解释器，直接执行指定的可执行文件。

   示例：

   ```dockerfile
   CMD ["python", "app.py"]
   ```

   上述命令将在容器启动时直接执行 `python app.py` 命令。

`CMD`命令只能在 Dockerfile 中出现一次。如果有多个`CMD`命令，只有最后一个会生效。如果在构建容器时指定了要运行的命令，它将覆盖`CMD`命令。

需要注意的是，`CMD`命令不会立即执行，而是在容器启动时才执行。它提供了容器的默认行为，但可以通过在`docker run`命令中附加参数来覆盖默认行为。例如，可以使用`docker run`命令的`--entrypoint`参数来指定在容器启动时要执行的命令，覆盖`CMD`命令。

### LABEL

`LABEL`命令是Dockerfile中的一个指令，用于为镜像添加元数据标签。这些标签提供了关于镜像的描述性信息，如版本号、维护者、描述、发布日期等。标签可以用于组织和管理镜像，也可以在运行容器时提供额外的信息。

`LABEL`命令的语法如下：

```dockerfile
LABEL <key>=<value> <key>=<value> ...
```

其中，`<key>`是标签的键，`<value>`是标签的值。可以指定多个键值对，每个键值对之间用空格分隔。

示例：

```dockerfile
LABEL version="1.0"
LABEL maintainer="John Doe <john@example.com>"
LABEL description="This is a sample Docker image"
```

上述示例为镜像添加了三个标签：`version`、`maintainer`和`description`。这些标签可以提供有关镜像的信息，帮助用户了解镜像的特性和用途。

要查看镜像的标签，可以使用`docker inspect`命令。例如，要查看名为 `myimage` 的镜像的所有标签，可以运行以下命令：

```shell
docker inspect --format='{{json .Config.Labels}}' myimage
```

标签的命名约定是自由的，但建议使用合理的命名规范，以便更好地组织和管理镜像。常见的标签键包括`version`、`maintainer`、`description`、`author`、`vendor`等，但您可以根据需要自定义标签。

使用`LABEL`命令添加元数据标签可以提高镜像的可读性和可维护性，有助于团队协作和镜像管理。

### ENV 设置环境变量

`ENV`命令是Dockerfile中的一个指令，用于设置环境变量。环境变量是在容器运行时可访问的键值对，可以用于配置容器内的应用程序、指定路径、设置默认值等。

`ENV`命令的语法如下：

```dockerfile
ENV <key>=<value>
```

其中，`<key>`是环境变量的名称，`<value>`是环境变量的值。可以指定多个环境变量，每个环境变量之间用空格分隔。

示例：

```dockerfile
ENV APP_VERSION=1.0
ENV DB_HOST=localhost DB_PORT=5432
```

上述示例定义了两个环境变量：`APP_VERSION`和`DB_HOST`。这些环境变量可以在容器内的应用程序中使用，例如通过 `$APP_VERSION` 和 `$DB_HOST` 引用它们。

使用环境变量可以使容器更加灵活和可配置。您可以在容器构建过程中设置环境变量，也可以在容器运行时通过`docker run`命令的`-e`参数指定环境变量的值。

另外，还可以在`ENV`命令中引用先前定义的环境变量，例如：

```dockerfile
ENV BASE_PATH=/app
ENV LOG_PATH=$BASE_PATH/logs
```

在上述示例中，`LOG_PATH`的值将为`/app/logs`，因为它引用了先前定义的`BASE_PATH`环境变量。

除了使用`ENV`命令设置环境变量，还可以在`RUN`命令中使用`export`命令来设置临时环境变量。但是，使用`ENV`命令更常见和推荐，因为它可以在镜像中持久化环境变量，并且可以更清晰地定义和管理。

总结来说，`ENV`命令用于在Dockerfile中设置环境变量，使容器内的应用程序能够使用这些变量进行配置和自定义。

### ARG 构建参数

`ARG`命令是Dockerfile中的一个指令，用于定义构建参数（build-time variables）。

`ARG`命令的语法如下：

```dockerfile
ARG <name>[=<default value>]
```

其中，`<name>`是构建参数的名称，可以在Dockerfile中使用。`<default value>`是可选的默认值，用于在构建过程中提供参数的默认值。如果没有指定默认值，那么构建过程中必须通过`--build-arg`选项来显式地提供参数值。

示例：

```dockerfile
ARG VERSION=latest
```

上述示例定义了一个名为`VERSION`的构建参数，并设置其默认值为`latest`。

在Dockerfile中，可以使用`${<name>}`语法来引用构建参数的值。

示例：

```dockerfile
FROM ubuntu:${VERSION}
```

上述示例使用了`${VERSION}`来引用构建参数`VERSION`的值。

构建镜像时，可以通过`--build-arg`选项来覆盖构建参数的默认值。

示例：

```bash
docker build --build-arg VERSION=1.0 .
```

上述示例在构建过程中使用了`--build-arg`选项来设置构建参数`VERSION`的值为`1.0`，覆盖了默认值`latest`。

使用构建参数可以使得Dockerfile更加灵活，可以根据不同的构建环境或需求来定制镜像。

总结来说，`ARG`命令用于定义构建参数，可以在Dockerfile中使用。它可以设置默认值，并在构建过程中通过`--build-arg`选项来覆盖默认值。构建参数使得Dockerfile更加灵活和可定制。

### EXPOSE 暴露端口

`EXPOSE`命令是Dockerfile中的一个指令，用于声明容器在运行时将监听的端口号。它并不实际打开或映射端口，而只是向用户和开发人员提供了关于容器的网络服务的信息。

`EXPOSE`命令的语法如下：

```dockerfile
EXPOSE <port> [<port>/<protocol>...]
```

其中，`<port>`是容器内部要监听的端口号。您可以指定一个或多个端口，每个端口之间用空格分隔。还可以指定协议（如TCP或UDP），以便更具体地定义端口的类型。

示例：

```dockerfile
EXPOSE 80
EXPOSE 443/tcp
EXPOSE 8080/tcp 8080/udp
```

上述示例分别声明了容器将监听的端口号：80、443（使用TCP协议）以及8080（使用TCP和UDP协议）。

`EXPOSE`命令的主要作用是提供给用户和开发人员有关容器的网络服务的信息。它并不会自动在主机和容器之间创建端口映射。要将容器的端口映射到主机上的端口，需要在运行容器时使用`docker run`命令的`-p`或`-P`参数。

例如，要将容器的80端口映射到主机上的8080端口，可以运行以下命令：

```shell
docker run -p 8080:80 myimage
```

在上述命令中，`-p 8080:80` 表示将容器的80端口映射到主机上的8080端口。

总结来说，`EXPOSE`命令用于声明容器在运行时将监听的端口号，提供给用户和开发人员有关容器网络服务的信息。要在主机和容器之间创建端口映射，需要在运行容器时使用`-p`参数。

### ADD 更高级的复制文件

`ADD`命令是Dockerfile中的一个指令，用于将本地文件或远程URL添加到容器中。它可以将文件、目录、压缩文件等复制到容器的文件系统中。

`ADD`命令的语法如下：

```dockerfile
ADD <src> <dest>
```

其中，`<src>`是要添加到容器中的源文件或目录的路径，可以是相对路径或绝对路径。`<dest>`是目标路径，表示文件或目录在容器中的位置。

示例：

```dockerfile
ADD app.jar /app/
ADD data/ /data/
```

上述示例将本地的`app.jar`文件复制到容器的`/app/`目录下，并将本地的`data/`目录复制到容器的根目录下的`/data/`目录。

`ADD`命令还支持自动解压缩压缩文件（如.tar、.tar.gz、.tar.bz2、.tar.xz、.zip等），并将其内容复制到容器中。

示例：

```dockerfile
ADD archive.tar.gz /app/
```

上述示例将本地的`archive.tar.gz`文件解压缩，并将解压后的内容复制到容器的`/app/`目录下。

需要注意的是，当使用`ADD`命令复制远程URL时，Docker将尝试下载并添加该URL指向的文件。这可能会导致构建过程变慢或构建失败，因此建议在构建过程中尽量避免使用远程URL。

与`ADD`命令类似的是`COPY`命令，它也可以将本地文件或目录复制到容器中。与`ADD`命令不同的是，`COPY`命令只能复制本地文件或目录，而不能复制远程URL，也不会自动解压缩压缩文件。

总结来说，`ADD`命令用于将本地文件或目录添加到Docker容器中，可以复制文件、目录以及自动解压缩压缩文件。在构建过程中，要谨慎使用远程URL，以免影响构建速度或稳定性。如果只需要复制本地文件或目录，可以考虑使用`COPY`命令。

### COPY 复制文件

`COPY`命令是Dockerfile中的一个指令，用于将本地文件或目录复制到容器中。与`ADD`命令相比，`COPY`命令更简单，只能复制本地文件或目录，不支持自动解压缩压缩文件或复制远程URL。

`COPY`命令的语法如下：

```dockerfile
COPY <src> <dest>
```

其中，`<src>`是要复制到容器中的源文件或目录的路径，可以是相对路径或绝对路径。`<dest>`是目标路径，表示文件或目录在容器中的位置。

示例：

```dockerfile
COPY app.jar /app/
COPY data/ /data/
```

上述示例将本地的`app.jar`文件复制到容器的`/app/`目录下，并将本地的`data/`目录复制到容器的根目录下的`/data/`目录。

需要注意的是，与`ADD`命令不同，`COPY`命令不会自动解压缩压缩文件。如果需要复制压缩文件并解压缩，可以先使用`ADD`命令将压缩文件复制到容器中，然后使用适当的工具（如`tar`或`unzip`）进行解压缩。

与`ADD`命令类似，`COPY`命令也可以使用相对路径或绝对路径。相对路径是相对于Dockerfile所在的目录，而绝对路径是从根目录开始的完整路径。

总结来说，`COPY`命令用于将本地文件或目录复制到Docker容器中，不支持自动解压缩压缩文件或复制远程URL。如果需要这些功能，可以考虑使用`ADD`命令。在构建过程中，使用适当的路径来指定源文件或目录的位置。

### ENTRYPOINT 入口点

`ENTRYPOINT`命令是Dockerfile中的一个指令，用于配置容器启动时要执行的命令。它指定了容器的主要可执行文件或入口点。

`ENTRYPOINT`命令的语法有两种形式：

1. 使用数组形式：

   ```dockerfile
   ENTRYPOINT ["executable", "param1", "param2"]
   ```

   其中，`executable`是要执行的可执行文件或命令，`param1`、`param2`等是传递给可执行文件的参数。

   示例：

   ```dockerfile
   ENTRYPOINT ["java", "-jar", "app.jar"]
   ```

   上述示例指定了在容器启动时执行`java -jar app.jar`命令。

2. 使用Shell形式：

   ```dockerfile
   ENTRYPOINT command param1 param2
   ```

   其中，`command`是要执行的命令，`param1`、`param2`等是传递给命令的参数。

   示例：

   ```dockerfile
   ENTRYPOINT java -jar app.jar
   ```

   上述示例与前面的示例相同，指定了在容器启动时执行`java -jar app.jar`命令。

需要注意的是，`ENTRYPOINT`命令与`CMD`命令一起使用时，`CMD`命令的参数会作为`ENTRYPOINT`命令的参数进行传递。如果在运行容器时指定了额外的参数，它们将覆盖`CMD`命令和`ENTRYPOINT`命令中的参数。

示例：

```dockerfile
ENTRYPOINT ["echo", "Hello"]
CMD ["world"]
```

在运行容器时，如果不指定额外的参数，容器将输出`Hello world`。如果指定了额外的参数，如`docker run myimage Goodbye`，容器将输出`Hello Goodbye`。

总结来说，`ENTRYPOINT`命令用于配置容器启动时要执行的命令。它可以使用数组形式或Shell形式指定可执行文件、命令和参数。与`CMD`命令一起使用时，`CMD`命令的参数会作为`ENTRYPOINT`命令的参数传递。

### VOLUME 定义匿名卷

`VOLUME`命令是Dockerfile中的一个指令，用于在容器中创建一个或多个挂载点（volumes）。挂载点是用于持久化存储数据的目录或文件。

`VOLUME`命令的语法如下：

```dockerfile
VOLUME <path>
```

其中，`<path>`表示要创建的挂载点的路径。可以指定一个或多个路径，用空格分隔。

示例：

```dockerfile
VOLUME /data
VOLUME /config /logs
```

上述示例创建了两个挂载点，分别是`/data`和`/config /logs`。在容器运行时，这些挂载点可以用于持久化存储数据，使得容器中的数据在容器重启或删除后仍然可用。

需要注意的是，`VOLUME`命令并不会在镜像中创建实际的目录或文件，它只是声明了容器中的挂载点。在运行容器时，可以使用`-v`选项将本地目录或文件与挂载点进行绑定，实现数据的持久化。

示例：

```bash
docker run -v /host/data:/data myimage
```

上述示例将主机上的`/host/data`目录与容器中的`/data`挂载点进行绑定，实现数据的共享和持久化。

总结来说，`VOLUME`命令用于在Docker容器中创建挂载点，用于持久化存储数据。在运行容器时，可以使用`-v`选项将本地目录或文件与挂载点进行绑定，实现数据的共享和持久化。从而保证了容器存储层的无状态化。

### USER

`USER`命令是Dockerfile中的一个指令，用于指定在容器中运行命令和操作的用户或用户组。

`USER`命令的语法如下：

```dockerfile
USER <user>[:<group>]
```

其中，`<user>`表示要切换到的用户名或用户ID，`<group>`表示要切换到的用户组名或用户组ID。用户和用户组可以是容器内已存在的用户和用户组，也可以是在Dockerfile中通过其他命令（例如`RUN`命令）创建的用户和用户组。

示例：

```dockerfile
USER myuser
```

上述示例将容器中的用户切换为`myuser`。

```dockerfile
USER myuser:mygroup
```

上述示例将容器中的用户切换为`myuser`，并将其所属的用户组设置为`mygroup`。

需要注意的是，`USER`命令在Dockerfile中是持久的，即在后续的命令中，除非使用`USER`命令再次切换用户，否则都会使用指定的用户或用户组来执行命令和操作。

示例：

```dockerfile
USER myuser
RUN echo "This command is executed as myuser"
RUN whoami
```

在上述示例中，第一个`RUN`命令和第二个`RUN`命令都将以`myuser`身份执行。

通过使用`USER`命令，可以在容器中以非特权用户的身份运行应用程序，以增加容器的安全性。

总结来说，`USER`命令用于指定在容器中运行命令和操作的用户或用户组。它可以切换到已存在的用户和用户组，也可以切换到通过其他命令在Dockerfile中创建的用户和用户组。

### WORKDIR

`WORKDIR`命令是Dockerfile中的一个指令，用于设置容器中的工作目录（工作路径）。

`WORKDIR`命令的语法如下：

```dockerfile
WORKDIR <directory>
```

其中，`<directory>`表示要设置的工作目录。可以是相对路径或绝对路径。如果目录不存在，`WORKDIR`命令会自动创建该目录。

示例：

```dockerfile
WORKDIR /app
```

上述示例将容器中的工作目录设置为`/app`。

`WORKDIR`命令在Dockerfile中是持久的，即在后续的命令中，除非使用另一个`WORKDIR`命令来更改工作目录，否则都会在指定的工作目录下执行命令和操作。

示例：

```dockerfile
WORKDIR /app
RUN ls
RUN pwd
```

在上述示例中，`ls`命令和`pwd`命令都会在`/app`目录下执行。

使用`WORKDIR`命令可以简化后续命令的路径操作，使得在Dockerfile中的命令更加清晰和易读。

总结来说，`WORKDIR`命令用于设置容器中的工作目录，即在后续的命令中执行命令和操作的基准路径。它可以简化路径操作，并提高Dockerfile的可读性。
