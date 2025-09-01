
## 安装linux系统必要依赖
```shell
sudo apt update
sudo apt install build-essential cmake libfftw3-dev libopenmpi-dev openmpi-bin \
    libx11-dev libgl1-mesa-dev libglu1-mesa-dev libftgl-dev \
    libxext-dev libxcursor-dev libxinerama-dev libxi-dev
```

## 安装NVIDIA驱动（根据显卡型号选择合适版本）
```shell
sudo apt install nvidia-driver-525
```


## 下载gromacs
```shell
wget https://ftp.gromacs.org/gromacs/gromacs-2025.2.tar.gz
md5sum gromacs-2025.2.tar.gz
tar -xf gromacs-2025.2.tar.gz
```

## 安装gromacs
```shell
cd gromacs-2025.2
mkdir build && cd build

# 查看DCUDA_ARCHITECTURES为8.9，所以下边cmake写为89
nvidia-smi --query-gpu=compute_cap --format=csv,noheader

cmake .. -DGMX_BUILD_OWN_FFTW=ON -DREGRESSIONTEST_DOWNLOAD=ON \
-DGMX_GPU=CUDA -DGMX_MPI=OFF -DGMX_OPENMP=ON \
-DCMAKE_INSTALL_PREFIX=/usr/local/gromacs \
-DCUDA_ARCHITECTURES=89 \
-DGMX_BUILD_MANUAL=OFF \
-DGMX_BUILD_TUTORIAL=OFF \
-DGMX_BUILD_DOC=OFF

# 编译并安装

make -j $(nproc)
sudo make install

```

测试安装成功
```shell
# 加载GROMACS环境
source /usr/local/gromacs/bin/GMXRC
gmx --version
```

安装acpype用于小分子准备
```shell
mamba create -n acpype -c conda-forge acpype python=3.9
mamba activate acpype
```


## 无root 无GPU

安装acpype用于小分子准备
```shell
mamba create -n gromacs -c conda-forge acpype python=3.9
mamba activate gromacs
```

## 下载gromacs
```shell
wget https://ftp.gromacs.org/gromacs/gromacs-2025.3.tar.gz
md5sum gromacs-2025.3.tar.gz
tar -xf gromacs-2025.3.tar.gz
```
