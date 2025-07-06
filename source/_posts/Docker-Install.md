---
title: 使用 Gemini 指导在 Ubuntu 系统上安装 Docker
tags:
  - 华为云
  - 安装教程
  - Docker
abbrlink: 53523
date: 2025-07-06 15:39:58
---
 

**核心思想**：在华为云 Flexus L 实例上安装 Docker，与在任何其他标准云服务器上安装 Docker 的过程基本一致。关键在于选择正确的操作系统，并通过 SSH 连接到实例后，执行相应的官方安装命令。

### 前提条件

1.  **已购买并运行一个华为云 Flexus L 实例**：
      * 在购买时，**操作系统 (OS)** 建议选择主流的 Linux 发行版，如 **CentOS 7.x/8.x** 或 **Ubuntu 18.04/20.04/22.04**。华为云的公共镜像市场通常会提供这些受支持的系统。
2.  **获取实例的弹性公网 IP (EIP)**：用于通过 SSH 从本地计算机连接到您的云服务器。
3.  **配置安全组规则**：
      * 确保您的安全组已开放 **22 端口**，以便进行 SSH 连接。
      * 如果您计划从外部访问 Docker 容器提供的服务（例如 Web 应用），请根据需要开放相应的端口（如 80, 443 等）。
4.  **拥有一个 SSH 客户端**：
      * Windows 用户可以使用 MobaXterm, Xshell, PuTTY 或 Windows Terminal。
      * macOS 和 Linux 用户可以直接使用系统自带的 `ssh` 终端命令。
5.  **拥有 root 用户权限或 sudo 权限**：安装软件需要管理员权限。

----- 

### 安装 Docker  

对于 Ubuntu 用户， 推荐使用 Docker 官方提供的便捷安装脚本。

**步骤 1：更新系统软件包**

首先，更新您的 `apt` 包索引和已安装的包。

```bash
sudo apt-get update
sudo apt-get upgrade -y
```

> 顺利完成。

**步骤 2：使用官方脚本安装 Docker**

和 CentOS 一样，使用官方的 `get-docker.sh` 脚本进行安装。

```bash
curl -fsSL https://get.docker.com -o get-docker.sh 
sudo sh get-docker.sh
``` 

>这一步遇到一个报错`curl: (35) Recv failure: Connection reset by peer`，只要后面多执行几遍就可以。两条命令最好分开执行。

**步骤 3：启动并验证 Docker**

在 Ubuntu 上，通过脚本安装后 Docker 服务通常会自动启动。您可以检查其状态。

```bash
# 检查 Docker 服务状态
sudo systemctl status docker
```

如果服务未运行，手动启动并设置开机自启：

```bash
sudo systemctl start docker
sudo systemctl enable docker
```
 
-----

### 第三部分：配置 Docker 镜像加速 (强烈推荐)

由于 Docker Hub 的服务器在国外，直接从官方仓库拉取镜像可能会很慢。强烈建议您配置国内的镜像加速器。华为云本身就提供了镜像加速服务。

1.  **登录华为云 SWR (容器镜像服务) 控制台**。
2.  在左侧导航栏中，选择 “镜像中心” -\> “镜像加速器”。
3.  控制台会为您提供一个专属的加速器地址，格式通常为 `https://<your-unique-code>.mirror.swr.myhuaweicloud.com`。

**配置步骤**：

1.  创建或修改 Docker 的配置文件 `/etc/docker/daemon.json`。

    ```bash
    sudo mkdir -p /etc/docker
    sudo tee /etc/docker/daemon.json <<-'EOF'
    {
      "registry-mirrors": ["https://<your-unique-code>.mirror.swr.myhuaweicloud.com"]
    }
    EOF
    ```

    **注意**：请务必将上面的 `<your-unique-code>` 替换为您在 SWR 控制台获取到的真实地址。

2.  **重启 Docker 服务**以使配置生效。

    ```bash
    sudo systemctl daemon-reload
    sudo systemctl restart docker
    ```

3.  现在，当您执行 `docker pull` 或 `docker run` 时，Docker 会优先尝试从华为云的镜像加速器拉取镜像，速度将得到极大提升。

### 总结

至此，您已经在华为云 Flexus L 实例上成功安装并配置好了 Docker。整个过程的核心是利用官方提供的一键化脚本，它能很好地处理不同 Linux 发行版的依赖和配置问题。配置镜像加速器是确保后续使用体验的关键一步。

现在，您可以开始构建自己的镜像、运行容器或部署各种容器化应用了。


