# Windows 上 Docker 与 Python 环境搭建指南

## 一、前提条件

1. 具有管理员权限。
2. 已在 BIOS/UEFI 中启用虚拟化（VT‑x/AMD‑V）。
3. 在 CMD 中验证虚拟化是否启用：

   * 方法一：运行 `systeminfo | findstr /i "Virtualization Enabled In Firmware"`，若显示 `Virtualization Enabled In Firmware: Yes` 即表示已启用。
   * 方法二：运行 `wmic cpu get VirtualizationFirmwareEnabled`，若显示 `TRUE` 即表示已启用。
   * 方法三（PowerShell）：运行：

     ```powershell
     Get-WmiObject -Class Win32_Processor | Select-Object Name,VirtualizationFirmwareEnabled
     ```

     若 `VirtualizationFirmwareEnabled` 为 `True`，则表示启用。
   * 如果系统已安装并运行 Hyper-V，`systeminfo` 可能不会显示上述行，而是在输出末尾看到：

     ```
     Virtualization-based security: Status: Running
     Hyper-V Requirements: A hypervisor has been detected. Features required for Hyper-V will not be displayed.
     ```

     这同样表示虚拟化功能已启用。

## 二、安装 Docker Desktop

1. 访问官网：[https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
2. 下载适用于 Windows 的安装包（Windows 10/11 Pro 或企业版，支持 Hyper-V）。
3. 双击安装包，按照提示完成安装，勾选“使用 WSL 2”选项。
4. 安装完成后，重启计算机。
5. 启动 Docker Desktop，确认 Docker 图标常驻系统托盘。

## 三、安装 Python（可选，宿主机环境）

1. 访问官网：[https://www.python.org/downloads/windows/](https://www.python.org/downloads/windows/)
2. 下载最新的 Python 3.x 安装程序（建议 x64）。
3. 运行安装程序，勾选“Add Python to PATH”，选择“Customize installation”，确保 pip 已勾选。
4. 安装完成后，在 PowerShell 中运行：

   ```powershell
   python --version
   pip --version
   ```

## 四、使用 Docker 构建 Python 环境

1. 创建项目目录：

   ```powershell
   mkdir D:\Projects\python-docker
   cd D:\Projects\python-docker
   ```
2. 在目录中创建 `Dockerfile`，示例内容：

   ```dockerfile
   FROM python:3.11-slim
   WORKDIR /app
   COPY requirements.txt .
   RUN pip install --no-cache-dir -r requirements.txt
   COPY . .
   CMD ["python", "main.py"]
   ```
3. （可选）创建 `requirements.txt`，填写所需依赖，如：

   ```text
   requests
   flask
   ```
4. 构建镜像：

   ```powershell
   docker build -t my-python-app .
   ```
5. 运行容器，并将当前目录挂载到容器 `/app`：

   ```powershell
   docker run -it --name python-container -v ${PWD}:/app my-python-app
   ```

## 五、使用 Docker Compose（可选）

1. 在项目目录创建 `docker-compose.yml`：

   ```yaml
   version: "3.8"
   services:
     app:
       build: .
       volumes:
         - .:/app
       command: python main.py
   ```
2. 启动服务：

   ```powershell
   docker-compose up --build
   ```

## 六、在 IDE 中调试

* 推荐使用 VS Code，安装 Docker 与 Python 插件。
* 打开项目文件夹，使用 “Remote - Containers” 功能进入容器开发。
