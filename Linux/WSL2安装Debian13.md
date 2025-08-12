## 先安装Debian12
在Windows应用商店中搜索下载Debian，安装的是Debian12

在powershell中设置wsl2为默认
```shell
wsl —set-default-version 2
```
在powershell中，进入debian

```shell
wsl -d Debian
```

## 修改镜像
修改`/etc/apt/sources.list` 文件为以下内容

```shell
deb http://deb.debian.org/debian trixie main
deb http://deb.debian.org/debian trixie-updates main
deb http://security.debian.org/debian-security trixie-security main
deb http://ftp.debian.org/debian trixie-backports main
```

## 升级到Debian13

```shell
sudo apt update && sudo apt upgrade -y
sudo apt full-upgrade -y
```

安装系统软件

```shell
sudo apt install -y build-essential curl wget libcurl4-openssl-dev libhdf5-dev libgdal-dev 
```
## 安装R语言
```shell
sudo apt update 
sudo apt install r-base r-base-dev
```

## 安装micromamba
```shell
"${SHELL}" <(curl -L micro.mamba.pm/install.sh)
source ~/.bashrc
```
更新micromamba
```shell
micromamba self-update
```
修改镜像
`~/.condarc` 为以下内容
```yaml
channel_priority: strict
always_yes: true
remote_connect_timeout_secs: 120
remote_read_timeout_secs: 120
remote_max_retries: 5
channels:
  - defaults
show_channel_urls: true
default_channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/bioconda
```
