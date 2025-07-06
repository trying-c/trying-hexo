---
title: 在 Ubuntu 系统上使用 Docker 部署 MinIO 对象存储服务
tags:
  - 华为云
  - 安装教程
  - Docker
  - MinIO
abbrlink: 56438
date: 2025-07-06 16:44:33
---

好的，我们接着上一步，继续配置更多的国内 Docker 镜像加速源，并部署一个公网可以访问的 Minio 对象存储服务。

### 第四部分：配置更多国内 Docker 镜像加速源

除了华为云自带的镜像加速器，还可以添加多个备用加速源，以提高镜像拉取的稳定性和速度。当一个源不稳定时，Docker 会自动尝试下一个。

常用的国内镜像加速源包括：

- **阿里云加速器** (需要登录阿里云容器镜像服务获取专属地址)
- **网易云加速器**: `http://hub-mirror.c.163.com`
- **Docker 中国官方镜像**: `https://registry.docker-cn.com`
- **中科大镜像**: `https://docker.mirrors.ustc.edu.cn`

**操作步骤：**

1.  **编辑 Docker 配置文件**：
    打开或创建 `/etc/docker/daemon.json` 文件。如果上一步已经创建了该文件，现在我们来编辑它。

    ```bash
    sudo vi /etc/docker/daemon.json
    ```

2.  **添加多个镜像源**：
    将文件内容修改为以下格式，将想用的加速器地址都加进去。我们把上次配置的华为云地址和这次新增的几个地址都放进去。

    ```json
    {
      "registry-mirrors": [
        "https://<your-unique-code>.mirror.swr.myhuaweicloud.com",
        "http://hub-mirror.c.163.com",
        "https://registry.docker-cn.com",
        "https://docker.mirrors.ustc.edu.cn"
      ]
    }
    ```

    **注意**：请务必将 `<your-unique-code>.mirror.swr.myhuaweicloud.com` 替换为自己的华为云专属加速地址。多个地址之间用英文逗号 `,` 分隔。

3.  **保存文件并重启 Docker**：
    保存并退出 `vi` 编辑器 (按 `ESC`，然后输入 `:wq` 并回车)。之后，重新加载配置并重启 Docker 服务。

    ```bash
    sudo systemctl daemon-reload
    sudo systemctl restart docker
    ```

现在，的 Docker 已经配置了多个国内镜像源，拉取镜像的速度和稳定性会更有保障。

---

### 第五部分：使用 Docker 部署 MinIO 对象存储

MinIO 是一个开源的高性能对象存储服务，兼容 Amazon S3 接口。使用 Docker 部署非常方便。

**操作步骤：**

1.  **创建数据存储目录**：
    为了让 MinIO 的数据能够持久化存储（即使容器被删除，数据也不会丢失），我们需要在宿主机上创建一个目录，用于挂载到容器内部。

    ```bash
    # 在当前用户的主目录下创建一个名为 minio-data 的文件夹
    mkdir ~/minio-data
    ```

2.  **执行 Docker 命令部署 MinIO**：
    接下来，使用 `docker run` 命令来启动一个 MinIO 容器。

    ```bash
    docker run -d \
       -p 9000:9000 \
       -p 9090:9090 \
       --name minio \
       -v ~/minio-data:/data \
       -e "MINIO_ROOT_USER=admin" \
       -e "MINIO_ROOT_PASSWORD=password123" \
       quay.io/minio/minio server /data --console-address ":9090"
    ```

    >上述命令部署的是较新的版本，官方删除了相当多一部分的功能和权限模块。推荐用以下命令部署指定版本的MinIO，这个版本为最新的没有删除管理模块的版本。

    ```bash
    docker run -d \
        --name minio \
        -p 9000:9000 \
        -p 9001:9001 \
        -v ~/minio-data:/data \
        -e MINIO_ROOT_USER=myadmin \
        -e MINIO_ROOT_PASSWORD=mypassword123 \
        minio/minio:RELEASE.2025-04-22T22-12-26Z \
        server /data --console-address ":9001"
    ```

    **命令参数详解**：

    - `-d`: 后台运行容器。
    - `-p 9000:9000`: 将容器的 `9000` 端口 (MinIO S3 API 端口) 映射到宿主机的 `9000` 端口。
    - `-p 9090:9090`: 将容器的 `9090` 端口 (MinIO Web 控制台端口) 映射到宿主机的 `9090` 端口。
    - `--name minio`: 给容器命名为 `minio`，方便管理。
    - `-v ~/minio-data:/data`: 将宿主机的 `~/minio-data` 目录挂载到容器内的 `/data` 目录。这是实现数据持久化的关键。
    - `-e "MINIO_ROOT_USER=admin"`: 设置 MinIO 的管理员用户名为 `admin`。**请替换为自己的用户名**。
    - `-e "MINIO_ROOT_PASSWORD=password123"`: 设置 MinIO 的管理员密码为 `password123`。**强烈建议替换为一个更复杂的强密码**。
    - `quay.io/minio/minio`: MinIO 的官方镜像地址。
    - `server /data --console-address ":9090"`: 启动 MinIO 服务，指定数据目录为 `/data`，并让控制台监听所有网络接口的 `9090` 端口。

3.  **检查 MinIO 容器状态**：
    查看容器是否成功启动。

    ```bash
    docker ps
    ```

    应该能看到一个名为 `minio` 的容器正在运行 (Up)。

---

### 第六部分：配置安全组，使公网可以访问 MinIO

现在，MinIO 已经在的 Flexus L 实例上运行了，但是公网还无法访问它，因为云服务器的安全组默认是关闭大部分端口的。我们需要放行刚刚映射的 `9000` 和 `9090` 端口。

**操作步骤：**

1.  **登录华为云控制台**。
2.  进入“**弹性云服务器 ECS**” 管理页面。
3.  在左侧导航栏中，找到并点击“**安全组**”。
4.  找到的 Flexus L 实例所绑定的那个安全组，点击它的名字进入配置页面。
5.  在安全组规则页面，选择“**入方向规则**”，然后点击右上角的“**添加规则**”按钮。
6.  **添加入方向规则 (放行 9090 端口 - Web 控制台)**：
    - **优先级**: 保持默认的 `1` 即可。
    - **策略**: 选择“**允许**”。
    - **协议端口**: 选择“**TCP**”，然后在后面的输入框中填入 `9090`。
    - **源地址**: 为了安全起见，可以只填写自己电脑的公网 IP 地址。如果想让任何地方都能访问，可以填写 `0.0.0.0/0`。**（注意：`0.0.0.0/0` 意味着对所有 IP 开放，请谨慎使用）**。
    - 点击“**确定**”。
7.  **添加入方向规则 (放行 9000 端口 - S3 API)**：
    - 重复上一步，再次点击“**添加规则**”。
    - **协议端口**: 选择“**TCP**”，填入 `9000`。
    - **源地址**: 同样，根据的需求填写自己的 IP 或 `0.0.0.0/0`。
    - 点击“**确定**”。

### 第七部分：访问的 MinIO 服务

一切就绪！现在可以在浏览器中访问的 MinIO Web 控制台了。

- **访问地址**: `http://<的Flexus L实例公网IP>:9090`
- **用户名**: `admin` (或自己设置的 `MINIO_ROOT_USER`)
- **密码**: `password123` (或自己设置的 `MINIO_ROOT_PASSWORD`)

登录成功后，就可以看到 MinIO 的管理界面，可以创建存储桶 (Bucket)、上传和管理文件了。的应用程序也可以通过 `http://<的Flexus L实例公网IP>:9000` 这个地址来使用 S3 API 与 MinIO 进行交互。

至此，已经成功地在华为云 Flexus L 实例上配置了 Docker 加速，并部署了一个公网可访问的 MinIO 对象存储服务。
