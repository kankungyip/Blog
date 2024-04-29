今天准备自行编译一个 MicroPython 的固件，并带上 ST7789 的显示驱动。之前用纯 MicroPython 编写的驱动速度太慢，在网上找的用 C 语言写的驱动又必须通过固件编译才能用，找了很久没有找到现成的固件下载，只好自己来编译一个。根据别人编译 MicroPython 的经验，装了一个虚拟机（[UTM](https://getutm.app/)），并且安装好了 Linux（Debain 12 x86_64），一切就绪，准备开始吧。

## 设置和准备编译环境

```bash
sudo apt update
sudo apt install git wget flex bison gperf python3 python3-pip python3-venv cmake ninja-build ccache libffi-dev libssl-dev dfu-util libusb-1.0-0
```

### 准备 ESP-IDF

根据 ESP-IDF 官方的[教程](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/get-started/linux-macos-setup.html)进行安装，这里只是搬运了一下。

```bash
mkdir -p ~/esp
cd ~/esp
git clone https://github.com/espressif/esp-idf.git
```

!> 因为 MicroPython 并不一定支持最新版本的 ESP-IDF，所以要切换到正确的版本，需要根据最新 MicroPython 版本所[支持](https://github.com/micropython/micropython/tree/v1.22-release/ports/esp32#setting-up-esp-idf-and-the-build-environment)的版本来选择。

```bash
cd ~/esp/esp-idf
git checkout v5.1.2
git submodule update --init --recursive
```

接下来设置开发 ESP32-S3 所需的编译器、调试器等工具。在这个过程中下载 Github 发布版本中附带的一些工具，国内访问 GitHub 较为缓慢，所以设置一个环境变量，从 Espressif 的下载服务器下载资源。

```bash
cd ~/esp/esp-idf
export IDF_GITHUB_ASSETS="dl.espressif.com/github_assets"
./install.sh esp32s3
```

!> 如果是需要为其他目标芯片开发，可以在 `esp32s3` 后面一次性指定多个芯片，例如 `esp32s3,esp32c3`。

最后，设置好环境变量就可以使用了，将下面的内容添加到 `.bashrc` 文件中，

```
# get esp-idf
alias get_idf='. $HOME/esp/esp-idf/export.sh'
```

更新 `.bashrc` 文件后，还需要重启终端窗口或运行下面的命令来刷新配置文件，并完成 ESP-IDF 的环境变量设置。

```bash
source ~/.bashrc
get_idf
```

当出现 `Done! You can now compile ESP-IDF projects.` 准备 ESP-IDF 的步骤就全部完成了。

### 准备 MicroPython

建立 `~/mpy` 文件夹，存放所有 MicroPython 相关的源文件。

```bash
mkdir -p ~/mpy
cd ~/mpy
git clone https://github.com/micropython/micropython.git
```

切换到最新的发行版本。

```bash
cd ~/mpy/micropython
git checkout v1.22-release
```

完成后首先编译 `mpy-cross` 工具，这是为了构建 MicroPython 的交叉编译器，只需要编译一次，之后编译固件不在需要重复，除非版本有更新。

```bash
cd ~/mpy/micropython
make -C mpy-cross
```

最后还需要初始化子模块，同上只需要编译一次。

```bash
cd ~/mpy/micropython/ports/esp32
make submodules
```

### 编译测试

测试编译一个 **ESP32-S3** 的固件，一切顺利的话会在 `~/mpy/micropython/ports/esp32` 路径下会出现一个 `build-ESP32_GENERIC_S3` 文件夹，文件夹内涵一个 `firmware.bin` 文件，这就是适用于 **ESP32-S3** 开发板的 MicroPython 固件。

```bash
cd ~/mpy/micropython/ports/esp32
make BOARD=ESP32_GENERIC_S3
```

!> 如果想编译其他开发板的固件，只需要将 `BOARD=` 后面设置成对应的开发板名字，这些开发板名字可以在 `~/mpy/micropython/ports/esp32/boards` 路径下找到（文件夹的名字），也可以在这个路径下自定义一块属于自己的开发板。

## 编译自制固件

### 准备 ST7789 驱动

将 [ST7789 驱动](https://github.com/russhughes/st7789_mpy) 克隆到 `~/mpy/st7789_mpy` 路径下。

```bash
cd ~/mpy
git clone https://github.com/russhughes/st7789_mpy.git
```

!> 如果要修改，可以对驱动进行修改，例如增加显示分辨率、增加更多绘图或写中文的功能等。

### 加入驱动编译固件

```bash
cd ~/mpy/micropython/ports/esp32
make BOARD=ESP32_GENERIC_S3 USER_C_MODULES=~/mpy/st7789_mpy/st7789/micropython.cmake
```

现在就得到了一个具有 ST7789 驱动的 ESP32-S3 的固件了。

### 支持 octal-SPIRAM

```bash
cd ~/mpy/micropython/ports/esp32
make BOARD=ESP32_GENERIC_S3 BOARD_VARIANT=SPIRAM_OCT USER_C_MODULES=~/mpy/st7789_mpy/st7789/micropython.cmake
```

### 编译好的固件下载

- [ESP32-S3 with ST7789 v1.22.2](https://pan.quark.cn/s/e68515391396)
- [ESP32-S3 octal-SPIRAM with ST7789 v1.22.2](https://pan.quark.cn/s/e68515391396)
