# Yolov5_10class的demo开发日志

## 1.demo环境配置（win10/ubuntu18.04）
```
依照demo的reqiurements:
# base ----------------------------------------
matplotlib>=3.2.2
numpy>=1.18.5
opencv-python>=4.1.2
Pillow
PyYAML>=5.3.1
scipy>=1.4.1
#torch>=1.7.0
torchvision>=0.8.1
tqdm>=4.41.0
requests
# plotting ------------------------------------
seaborn>=0.11.0
pandas
thop  # FLOPS computation
pycocotools>=2.0  # COCO mAP
protobuf
```
安装过程中出现的问题汇总：
* 在 pytorch 官网上选取配置对应自身配置选取操作平台、编程语言、显卡配置再进行下载（可选取历史版本）；
* 安装 torch、torchvision、torchaudio后，import torch出错：
  ```
    # 出现错误提示：
    Error loading “D:\Coding\python\lib\site-packages\torch\lib\asmjit.dll“
    # 从 https://aka.ms/vs/16/release/vc_redist.x64.exe 下载并安装即可；
  ```
* 安装 pycocotools 报错：
    ![](https://github.com/minieyeqi/md/raw/main/images/demo_3.PNG)
  ```
  # 1.1. 从 https://github.com/pdollar/coco.git 这个网址下载源码（直接把压缩包下下来），解压到本地（按理来说哪儿都可以，但是既然能遇到这种问题，说明还是懂中文路径不友好的，所以放到英文路径下)，我个人是放在Python文件夹下；
  # 1.2 若是ubuntu18.04,直接编译即可;
  # 1.3 若是win10:进入cocoapi-master/PythonAPI文件夹，在此处打开Powershell窗口（shift+鼠标右键，就能看到了），运行命令：
    python setup.py build_ext --inplace
    python setup.py build_ext install
  ```
* 在 run sil_yolov5_10class.py： 
   * 'No module named 'yaml'：
        ![](https://github.com/minieyeqi/md/raw/main/images/demo_1.PNG)
        ```
        # 报错位置在位置在/utils/plots.py
        # 解决方法：
        pip install pyyaml
        ```
* 在 run demo_for_test_video.py：
  *  'No module named 'torch2txt'：
        ![](https://github.com/minieyeqi/md/raw/main/images/demo_2.PNG)
        ```
        # torch2trt作用是直接转torch模型为tensorrt
        ```
## demo程序结构
```

```
