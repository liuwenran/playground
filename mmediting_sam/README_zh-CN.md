# MMEditing-SAM

<div align=center>
<img src="https://user-images.githubusercontent.com/12782558/232700025-a7bfe119-9eb5-46d2-b57c-ba7dc8c40d83.png"/>
</div>

这个文件夹下包含了将MMEditing和SAM一起使用的有趣玩法

## 📄 目录

- [🛠️ 安装](#安装)
- [⬇️ 下载](#下载)
- [🚀 玩法](#玩法)

## 安装

首先创建一个conda环境，然后把MMEditing和SAM安装到里面。

```shell
# create env and install torch
conda create -n mmedit-sam python=3.8 -y
conda activate mmedit-sam
pip install torch==1.12.1+cu113 torchvision==0.13.1+cu113 --extra-index-url https://download.pytorch.org/whl/cu113

# install mmediting
pip install openmim
mim install mmengine "mmcv>=2.0.0"
git clone -b dev-1.x https://github.com/open-mmlab/mmediting.git
pip install -e ./mmediting

# install sam
pip install git+https://github.com/facebookresearch/segment-anything.git

# you may need ffmpeg to get frames or make video
sudo apt install ffmpeg
```

## 下载

下载SAM的模型。

```shell
mkdir -p checkpoints/sam
wget -O checkpoints/sam/sam_vit_h_4b8939.pth https://dl.fbaipublicfiles.com/segment_anything/sam_vit_h_4b8939.pth

```

## 玩法

### 结合SAM一起玩controlnet动画

***使用方法***

找一个视频拆出视频帧。

```shell
mkdir -p inputs/demo_video
ffmpeg -i your_video.mp4 inputs/demo_video/%04d.jpg
```

运行脚本。

```shell
python play_controlnet_animation_sam.py
```

用输出的视频帧组合成视频。

```shell
ffmpeg -r 10 -i results/final_frames/%04d.jpg -b:v 30M -vf fps=10 results/final_frames.mp4
```

***输出样例***

下面是我们一个视频的输入输出示例。试一下你自己的视频吧！

<div align="center">
  <video src="https://user-images.githubusercontent.com/12782558/232666513-a735fadb-b92b-4807-ba32-8a38b1514622.mp4" width=1024/>
</div>

***方法解析***

我们通过下面的步骤得到最终的视频：

1. 将输入视频拆成帧

2. 通过MMEditing的前向接口调用controlnet animation模型对每帧视频进行修改，使其变为AI动画

3. 使用MMEditing内的stable diffusion生成一张和动画内容语意贴合的背景图片

4. 用SAM预测动画中人物的mask

5. 将动画中的背景替换为我们生成的背景图片
