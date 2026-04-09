# RK3588 部署全栈实战：基于 Qt6.1+ 与 YOLOv5 的高性能嵌入式视觉部署与工业模拟控制系统

###### 简介：

RK3588 部署全栈实战：基于 RK3588 与 Qt6 的嵌入式 YOLOv5 目标检测与工业级协同控制系统。

RK3588 Full-Stack Deployment Practical Guide: Embedded YOLOv5 Object Detection and Industrial-Level Collaborative Control System Based on RK3588 and Qt6

本项目旨在探索 RK3588 NPU 在工业自动化场景下的性能极限。通过 C++ 与 Qt6 驱动高帧率 UI，实现 YOLOv5 模型的高效率量化推理，并模拟视觉引导下的机械臂（HID 信号）精确控制链路。

This project aims to explore the performance limits of RK3588 NPU in industrial automation scenarios. By using C++and Qt6 to drive a high frame rate UI, the YOLOv5 model achieves efficient quantitative inference and simulates the precise control link of a robotic arm (HID signal) under visual guidance.

###### 如果遇到网络问题，请找到科学上网的渠道并开启梯子



## 一、前期环境搭建

### (一)Linux环境：为何选用WSL2而不是VMware

#### 1. 极高的时间与资源效率

##### (1)传统虚拟机（VMware）

 启动慢，需要预先分配固定的内存（比如死扣 4GB 或 8GB 给它），而且网络配置（NAT 或桥接）时不时会因为 Windows 更新或网络环境变化而断连。

##### (2)**WSL2：** 

几乎是秒级启动。它按需动态占用系统内存，你在 Linux 里跑大任务它就多占，跑完就释放回给 Windows。对于需要频繁在 Windows UI 和 Linux 编译环境之间切换的视觉算法开发来说，体验极其流畅。

#### 2. 生态工具的底层级无缝集成

在工业自动化或底层软件开发团队中，开发者通常需要在 Windows 下处理文档、使用特定的上位机软件或 CAD 工具，但同时又必须在 Linux 下编译 C++ 代码。

##### (1)VS Code 与 PyCharm

 微软对 WSL2 的支持是亲儿子级别的。在 VS Code 中，你只需要点击左下角的“连接到 WSL”，它就会自动在底层的 Ubuntu 中配置好 C++ 补全、CMake 甚至 GDB 调试。你不需要像在 VMware 中那样去配置复杂的 SSH 密钥和 IP 地址。

##### (2)文件互通

 在 WSL2 中，Windows C 盘直接挂载在 `/mnt/c`。你可以直接在 Windows 的资源管理器里输入 `\\wsl$` 瞬间访问 Linux 里的模型和代码，这在频繁拷贝 `.rknn` 模型文件时尤其高效。

#### 3. 为未来的技术栈扩展（如 Docker）铺路

现代企业在部署边缘 AI 应用时，越来越流行使用容器化技术（Docker）。而在 Windows 平台上运行的 Docker Desktop，目前官方强制推荐并默认使用 WSL2 作为其底层引擎。掌握了 WSL2，未来学习容器化部署会非常平滑，这也是一个非常加分的亮点。

### (二)WSL2以及Linux环境安装

#### 1.WSL2安装

##### (1)以管理员身份运行 PowerShell

①右键点击 Windows 桌面的“开始”按钮（或者按 `Win + X` 快捷键）。

②在弹出的菜单中，选择 **“Windows PowerShell (管理员)”** 或 **“Windows 终端 (管理员)”**。

③如果系统弹出用户账户控制提示框，点击“是”允许运行。

##### (2)执行安装命令

在弹出的命令行窗口中，输入以下命令并按回车：

```powershell
wsl --install -d Ubuntu
```

说明：这个命令会自动为你匹配并安装默认的最新 Ubuntu LTS（长期支持）版本（通常就是 22.04 或 24.04）。无论具体是哪个版本，对于 RK3588 的 rknn-toolkit2 工具链和后续的 C++ 交叉编译来说，都是完全兼容的业界标准环境。

##### (3)后续操作

当你按下回车后，只要看到屏幕上开始出现类似 **“正在下载 (Downloading...)”** 或 **“正在安装 (Installing...)”** 的进度提示，就说明问题已经解决了。

等待进度条走完后，如果系统提示需要重启，请**重启电脑**。重启后，系统通常会自动弹出一个黑色的终端窗口（如果没有自动弹出，请在“开始”菜单里找到“Ubuntu”并打开）

##### (4)本地下载并且导入

如果出现了“无法与服务器建立连接    0.0%  ”这样的网络问题，那我们需要本地下载Linux并且导入，具体步骤如下
①首先，需要访问 Ubuntu 官方 WSL 下载页面 并下载适用于 WSL 的 Ubuntu 映像。
直接下载链接：https://ubuntu.com/desktop/wsl 
请选择Intel or AMD 64-bit architecture分支，我们用来开发的 Windows 10 电脑，其 CPU 绝大概率是 Intel 或 AMD 的（这属于 x86_64 架构）。底层的 WSL 子系统必须和你的物理 CPU 架构相匹配。下面那个 ARM 64-bit 是给特定的 ARM 架构电脑（比如某些高通芯片的 Windows 笔记本，或者 Mac 的 M系列芯片）准备的。 我们费这么大劲装 Ubuntu，核心目的之一是为了跑 `rknn-toolkit2` 把 YOLOv5 模型转成 `.rknn` 格式。瑞芯微官方的模型转换工具，强制要求必须在 x86_64 架构的 Linux 环境下运行。

下载完成后会得到一个压缩包（ubuntu-24.04.4-wsl-amd64.gz），解压后把文件重命名为ubuntu-24.04.4-wsl-amd64.wsl，然后双机即可安装（或者使用指令安装，这里给出知乎原帖链接https://zhuanlan.zhihu.com/p/1984302163720167624）

#### 2.Linux环境配置

##### (1)初始化系统并设置账号密码

安装完成后，应该会在黑色的窗口里出现如下内容：

```powershell
正在安装: D:\ubuntu-24.04.4-wsl-amd64.wsl
已成功安装分发。可以通过 “wsl.exe -d Ubuntu-24.04” 启动它]
正在启动 Ubuntu-24.04...
Provisioning the new WSL instance Ubuntu-24.04
This might take a while...
Create a default Unix user account: 

```

解压完成后，系统会提示你创建专属账号：

- `Create a default Unix user account`:输入一个全英文字母的用户名，按回车。
- `New password:` 输入你的密码。**（新手避坑提示：在 Linux 终端输入密码时，屏幕上不会显示任何星号或字符，光标也不会动。这是正常的安全机制，你闭着眼睛盲打完密码按回车即可）**。
- `Retype new password:` 再次盲打输入密码确认。

成功后会显示：

```powershell
passwd: password updated successfully
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

yourname@DESKTOP-AOK4792:/mnt/d$
```

##### (2)更新系统底层库（非常重要）

为了确保后续安装 Python 和交叉编译工具时不报错，我们需要把刚装好的系统更新到最新状态。在绿色的提示符后面，复制粘贴（在终端里通常是**点击鼠标右键**粘贴）以下命令并回车：

```powershell
sudo apt update && sudo apt upgrade -y
```

### (三)安装Python环境

#### 1.安装 Miniconda 创建 Python 沙盒

##### (1)回到你的 Linux 主目录

```bash
cd ~
```

执行后，提示符末尾应该会变成一个波浪号 `~`，代表你回主目录了。

#### 2.下载 Miniconda 安装包

```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
```

这会从服务器拉取一个大概几十 MB 的安装脚本，等待进度条到 100%。

#### 3.运行安装程序

```bash
bash Miniconda3-latest-Linux-x86_64.sh
```

运行上一条命令后，程序会问你几次问题，按以下方式操作：

- **按 `Enter`（回车）键**查看许可协议。
- 一直按**空格键**翻页，直到最底部出现 `Do you accept the license terms? [yes|no]`。
- 输入 `yes` 并回车。
- 接着它会问你安装路径（默认是 `/home/skypuffer/miniconda3`），**直接按回车确认**。
- 最后，也是最重要的一步！它会问你 `You can undo this by running ... Do you wish the installer to initialize Miniconda3 by running conda init? [yes|no]`。**务必输入 `yes` 并回车**。

#### 4.激活 Conda

安装完成后，为了让 Conda 命令立刻生效，执行以下命令:

```bash
source ~/.bashrc
```

### (四)配置瑞芯微的模型转换工具链 (`rknn-toolkit2`)

#### 1.创建 RKNN 专属的 Python 虚拟环境

为了不把环境弄乱，我们专门为这个工具链开辟一个 Python 3.10 的独立房间。

##### (1)输入以下命令创建环境,接受 Conda 服务条款（`-y` 表示一路自动同意）：

```bash
conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/main
```

```bash
conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/r
```

```bash
conda create -n rknn python=3.10 -y
```

##### (2)创建完成后，激活进入这个专属环境：

```bash
conda activate rknn
```

成功后，你命令行最左边的 `(base)` 会变成 `(rknn)`，这说明你已经进入了正确的房间。

##### (3)下载瑞芯微官方工具链代码

Ubuntu 系统自带了 `git` 工具，我们直接用它把瑞芯微的 GitHub 仓库克隆到你的 Linux 系统里。

执行以下命令拉取代码（这个仓库大约几百 MB，包含了工具、示例和文档）：

```bash
git clone https://github.com/airockchip/rknn-toolkit2.git
```

###### 以下为下载速度特别慢的备用办法：

如果你跟我一样，几十 KB/s 的速度拉取几百兆的代码库确实让人抓狂。直接手动下载再导入是最高效的办法！

首先，在你的终端里按下 **`Ctrl + C`**，强制中断那个慢得发昏的 `git clone` 进程。

接下来，我们用最简单的“Windows 复制粘贴法”把代码放进 Linux 里。请按以下步骤操作：

①在你的 Windows 浏览器中打开这个链接：https://github.com/airockchip/rknn-toolkit2

②点击页面右侧绿色的 **`<> Code`** 按钮，在下拉菜单中选择 **Download ZIP**。如果你有梯子，下载会很快；如果没有，用迅雷等下载工具拉取这个 ZIP 文件通常也会比命令行快很多

③下载完成后，在 Windows 里将这个 ZIP 文件**解压**。你会得到一个名为 `rknn-toolkit2-master` 的文件夹。

④打开任意一个 Windows 的**文件资源管理器**（就是你平时看 C 盘 D 盘的那个窗口），在窗口最上方的**地址栏**中，清空里面的内容，输入以下路径并回车：

```bash
\\wsl$
```

如果进不去，可以尝试输入 `\\wsl.localhost`

⑤你会看到一个名为 **`Ubuntu-24.04`** 的网络驱动器，双击进入，接着依次进入 **`home`** -> **`yourname`** 文件夹，这就是你在 Linux 里的主目录。

⑥把刚才在 Windows 里解压出来的 **`rknn-toolkit2-master`** 文件夹，直接**复制并粘贴（或拖拽）**到这个 `skypuffer` 文件夹里，为了方便后续敲代码，建议你在 Windows 里把刚拖进去的这个文件夹**重命名**为 **`rknn-toolkit2`**（就是把后面的 `-master` 删掉）。

##### (4)安装运行所需的底层依赖库（使用清华源）

①进入安装包所在的目录

```bash
cd ~/rknn-toolkit2/rknn-toolkit2/packages
```

```bash
cd x86_64
```

②安装运行所需的底层依赖库

```bash
pip install -r requirements_cp310*.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```

这步会下载很多科学计算库比如 numpy, torch 等，稍微等进度条跑完。

③安装 RKNN 核心工具包

```bash
pip install rknn_toolkit2*cp310*.whl
```

④安装完成后，验证一下是否大功告成

```bash
python -c "from rknn.api import RKNN; print('RKNN 导入成功！环境配置完美！')"
```

只要看到那句中文打印出来，模型转换环境就彻底搞定了！接下来，就把你现有的 YOLOv5 模型塞进去转成 RK3588 认识的格式。

### (五)模型转换（INT8量化）

###### 请提前准备好一个yolov5的onnx格式的模型，推荐4类别以下，大小640x640以下的比较好哦

#### 1.建立转换工作区

①在你 Windows 的某个盘里（比如桌面或者 D 盘的项目文件夹），新建一个文件夹，命名为 `rknn_convert_workspace`。

②把你要转换的 YOLOv5 模型（假设叫 `yolov5s.onnx`）复制到这个文件夹里。

#### 2.准备“量化校准数据集”（关键!)

要让模型在 RK3588 的 NPU 上跑得极快，就必须做 **INT8 量化**。量化的过程需要“看”几张实际的图片，来校准神经元数据的分布范围。

①在 `rknn_convert_workspace` 文件夹里，新建一个子文件夹，命名为 `dataset`。

②从你以前训练 YOLOv5 的数据集中，挑 **20~50 张**具有代表性的图片（必须是你的工业检测场景图），放进 `dataset` 文件夹。

③在 `rknn_convert_workspace` 文件夹下，新建一个文本文件 `dataset.txt`。里面写上这几十张图片的相对路径，每行一个。 比如 `dataset.txt` 里的内容看起来像这样：

```yaml
dataset/img_000.jpg
dataset/img_001.jpg
dataset/img_002.jpg
```

**不要有中文大括号（）或者其他容易引起识别错误的符号！！！！！！**

#### 3.编写核心转换脚本

在 `rknn_convert_workspace` 文件夹里，新建一个文本文档，重命名为 `convert.py`（注意把 `.txt` 后缀删掉），然后用记事本或 VS Code 打开它，把下面这段转换代码复制进去：

```python
import os
from rknn.api import RKNN

# --- 配置参数 ---
ONNX_MODEL = 'yolov5s.onnx'       # 你的 ONNX 模型文件名
RKNN_MODEL = 'yolov5s.rknn'       # 输出的 RKNN 模型文件名
DATASET = './dataset.txt'         # 刚才建的量化校准集路径
QUANTIZE_ON = True                # 开启量化

def main():
    # 1. 创建 RKNN 对象
    rknn = RKNN(verbose=True)

    # 2. 配置模型输入参数
    # YOLOv5 默认输入 RGB 图像，像素值在 0~255。
    # NPU 需要将其归一化到 0~1，所以 mean=0, std=255。
    print('--> 正在配置 RKNN 参数...')
    rknn.config(mean_values=[[0, 0, 0]], 
                std_values=[[255, 255, 255]], 
                target_platform='rk3588')

    # 3. 加载 ONNX 模型
    print('--> 正在加载 ONNX 模型...')
    ret = rknn.load_onnx(model=ONNX_MODEL)
    if ret != 0:
        print('加载 ONNX 模型失败！')
        exit(ret)

    # 4. 构建 RKNN 模型 (这里进行量化)
    print('--> 正在构建并量化 RKNN 模型...')
    ret = rknn.build(do_quantization=QUANTIZE_ON, dataset=DATASET)
    if ret != 0:
        print('构建 RKNN 模型失败！')
        exit(ret)

    # 5. 导出保存为 .rknn 文件
    print('--> 正在导出 RKNN 模型...')
    ret = rknn.export_rknn(RKNN_MODEL)
    if ret != 0:
        print('导出 RKNN 模型失败！')
        exit(ret)

    print('🎉 恭喜！模型转换与量化成功！文件已保存为:', RKNN_MODEL)

    # 释放资源
    rknn.release()

if __name__ == '__main__':
    main()
```

做完这三步后，你的文件夹里应该有：`yolov5s.onnx`、`dataset`文件夹、`dataset.txt`、`convert.py`。

#### 4.开始转换

虽然 WSL2 可以通过特定路径（如 `/mnt/d/`）直接访问你的 D 盘，但 **WSL2 跨文件系统（从 Linux 跨界去读 Windows 硬盘）的 I/O 速度非常慢**。 我们接下来的量化过程需要频繁读取图片，如果放在 Windows 盘里，转换脚本可能会卡住或者运行得很慢；如果把它们放在 Linux 原生的文件系统里，速度会飞起来。

①打开你的 D 盘 `D:\Project\`，鼠标右键复制 `rknn_convert_workspace` 这个文件夹。

②在 Windows 的文件资源管理器地址栏中，输入用过的： `\\wsl$\Ubuntu-24.04\home\yourname` *(按回车进入)* 在这个你的 Linux 主目录里，**直接粘贴**。

③回到那个带有 `(rknn) yourname@DESKTOP-AOK4792:~$` 的黑框框里。需要输入：

```bash
cd ~/rknn_convert_workspace
```

④直接运行：

```bash
python convert.py
```

#### 5.可能的报错解决

##### ①ModuleNotFoundError: No module named 'pkg_resources'

报错信息 `ModuleNotFoundError: No module named 'pkg_resources'` 表明你当前的 Python 虚拟环境中缺少了一个非常基础的 Python 打包工具模块：`setuptools`。这是一个非常典型的“版本太新导致的坑”。你刚才的操作完全正确，系统也明确提示了 `setuptools` 已经安装，并且版本是极新的 `82.0.1`。

报错的根本原因在于：Python 的 `setuptools` 包在最近的重大更新（70.0 之后的版本）中，正式弃用并结构性地移除了老旧的 `pkg_resources` 模块。而瑞芯微的 `rknn-toolkit2` 底层代码由于发布较早，代码里硬编码了对这个老模块的依赖，所以系统直接抛出了找不到该模块的错误。

**解决办法极其简单：把 `setuptools` 降级到一个包含该模块的经典稳定版本即可。**

```bash
pip install setuptools==68.2.2 -i https://pypi.tuna.tsinghua.edu.cn/simple
```

##### ②AttributeError: module 'onnx' has no attribute 'mapping'

这和刚才的 `setuptools` 是同一种情况：你当前环境里的 `onnx` 库版本**太新了**。在最新版的 `onnx` 库中，官方删除了一个叫 `mapping` 的旧模块。但是，瑞芯微的 `rknn-toolkit2` 在解析 ONNX 模型时，还在执着地去调用这个旧模块，结果当然就是找不到而崩溃了。

我们需要把 `onnx` 库退回到一个兼容的经典版本（比如 `1.13.1` 或 `1.12.0`）。

**降级安装 onnx（继续使用清华源加速）：** 请在终端中直接复制并运行以下命令：

```bash
pip install onnx==1.13.1 -i https://pypi.tuna.tsinghua.edu.cn/simple
```



## 二、嵌入式 C++ 交叉编译与板端部署

### (一)RK3588硬件调试

#### 1、给你的 RK3588 插上电源（确认硬件无损坏即可）

在我们在电脑上配置复杂的 C++ 交叉编译工具链之前，我们需要确保你的“靶机”（RK3588）已经做好了迎接代码的准备。

**这需要你进行以下物理操作：**

①给你的 RK3588 插上电源。

②插上你准备好的 mini 屏幕（HDMI 或 MIPI 接口）。

③插上鼠标键盘，并连上网线（或者连上 Wi-Fi）。

#### 2、刷入Linux系统22.04

①在浏览器搜索并下载 **balenaEtcher**（这是一个免费开源的经典工具）

下载链接：https://www.pcsoft.com.cn/soft/30222152.html

②下载官方 Ubuntu 系统镜像

注意：请根据你购买的系统去官方下载对应的Linux版本，这里就不贴出下载链接了，我的是LubanCat 5

③下载**`Ubuntu22.04`** 并且带有 **`XFCE`** 或 **`Desktop`** 字样的文件（这代表带图形桌面的系统，能让你的屏幕亮起来），把它下载到你的电脑上。如果下载下来是压缩包（比如 `.img.gz` 或 `.zip`），**不用解压**，Etcher 可以直接生吞压缩包！

④使用balenaEtcher或者其他烧录工具，对TF卡进行烧录（自备TF卡和读卡器，TF卡推荐64G以上）

