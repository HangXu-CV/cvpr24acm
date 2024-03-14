<div align="center">

# <b>Rethinking Boundary Discontinuity Problem for Oriented Object Detection

[Hang Xu](https://hangxu-cv.github.io/)<sup>1*</sup>, [Xinyuan Liu](https://scholar.google.com/citations?user=eXwizz8AAAAJ&hl=zh-CN)<sup>2*</sup>, [Haonan Xu](https://scholar.google.com/citations?user=Up_a2VAAAAAJ&hl=zh-CN)<sup>2</sup>, [Yike Ma](http://www.ict.cas.cn/sourcedb_ict_cas/cn/jssrck/201511/t20151119_4470413.html)<sup>2</sup>, [Zunjie Zhu](http://iipl.net.cn/index/team_details/id/129.aspx)<sup>1</sup>, [Chenggang Yan](https://cgyan-iipl.github.io/)<sup>1</sup>, [Feng Dai](http://www.ict.cas.cn/sourcedb_ict_cas/cn/jssrck/201404/t20140422_4096774.html)<sup>2†</sup>

<p><sup>1</sup>Hangzhou Dianzi University &nbsp;&nbsp;<sup>2</sup>Institute of Computing Technology, Chinese Academy of Sciences
<br><sup>*</sup>Equal contribution &nbsp;&nbsp;<sup>&dagger;</sup>Corresponding author<p>

### [Projectpage]() · [Paper](https://arxiv.org/abs/2305.10061) · [Video]()

</div>

## :fire: News
- **[2/27/2024]** This paper is accepted by CVPR 2024.

## Introduction
We propose GPS-Gaussian, a generalizable pixel-wise 3D Gaussian representation for synthesizing novel views of any unseen characters instantly without any fine-tuning or optimization.

## Installation
### Requirements
* Linux
* Python 3.7+
* Pytorch 1.7.0 or higher
* mmcv
* CUDA 11.0
* GCC 7.5.0

####  INSTALL
1. Create a conda virtual environment and activate it

```
conda create -n GF python=3.7 -y
conda activate cvpr24acm
```

2. Install PyTorch and torchvision following the [official instructions](https://pytorch.org/), e.g.

`conda install pytorch==1.7.1 torchvision==0.8.2 torchaudio==0.7.2 cudatoolkit=11.0 -c pytorch`

3. Install [mmcv](https://github.com/open-mmlab/mmcv) for DCNv2, e.g.

```
#pip install mmcv-full -f https://download.openmmlab.com/mmcv/dist/{cu_version}/{torch_version}/index.html
pip install mmcv-full -f https://download.openmmlab.com/mmcv/dist/cu110/torch1.7.1/index.html
```

4. Install DOTA_devkit

```
sudo apt-get install swig
cd DOTA_devkit
swig -c++ -python polyiou.i
python setup.py build_ext --inplace
```

### Dataset Preparation
For DOTA datasets, please refer [DOTA_devkit](https://github.com/CAPTAIN-WHU/DOTA_devkit) to crop the original images into patches. e.g. 1024×1024 pixels with overlap 256 px.

Please organize the datasets in the following format. Note that the test set of DOTA does not provide annotations, so you can place the corresponding empty files in the test_split/labelTxt path.

As described in the paper, we use a relatively large image resolution during the test, please crop the test image into a 4000×4000 px with overlap 2000 px.
 
```
cvpr24acm
├── DOTA_devkit
│   ├── datasets
│   │   ├── DOTA
│   │   │   ├── trainvalsplit-1024-256
│   │   │   │   ├── images
│   │   │   │   ├── labelTxt
│   │   │   ├── trainvalsplit-multiscale
│   │   │   │   ├── images
│   │   │   │   ├── labelTxt
│   │   │   │── test4000
│   │   │   │   ├── images
│   │   │   │   ├── labelTxt
│   │   │── HRSC2016
│   │   │   │── train
│   │   │   │   ├── images
│   │   │   │   ├── labelTxt
│   │   │   │── test
│   │   │   │   ├── images
│   │   │   │   ├── labelTxt
│   │   │── UCAS_AOD
│   │   │   │── train
│   │   │   │   ├── images
│   │   │   │   ├── labelTxt
│   │   │   │── test
│   │   │   │   ├── images
│   │   │   │   ├── labelTxt
```

for each annotation file (.txt), each line represent an object following:

x1, y1, x2, y2, x3, y3, x4, y4, class, difficult

```
e.g.:
2753 2408 2861 2385 2888 2468 2805 2502 plane 0
3445 3391 3484 3409 3478 3422 3437 3402 large-vehicle 0
3185 4158 3195 4161 3175 4204 3164 4199 large-vehicle 0
2870 4250 2916 4268 2912 4283 2866 4263 large-vehicle 0
630 1674 628 1666 640 1654 644 1666 small-vehicle 0
636 1713 633 1706 646 1698 650 1706 small-vehicle 0
717 76 726 78 722 95 714 90 small-vehicle 0
737 82 744 84 739 101 731 98 small-vehicle 0
...
```

### Training
For example, using 2 GPUs to training a ResNet50 model:
```
#DOTA
CUDA_VISIBLE_DEVICES=0,1 python train_dota.py --datadir ./DOTA_devkit/datasets/DOTA/trainvalsplit-1024-256 --model 50 
#HRSC2016
CUDA_VISIBLE_DEVICES=0,1 python train_hrsc.py --datadir ./DOTA_devkit/datasets/HRSC2016/train --model 50 --input_size 640
#UCAS-AOD
CUDA_VISIBLE_DEVICES=0,1 python train_ucas.py --datadir ./DOTA_devkit/datasets/UCAS_AOD/train --model 50 --input_size 640
```
`--input_size ` is the long side resolution during training,  which must be divisible by 32.

### Testing

For DOTA dataset:
```
# Single scale testing
python evaluate.py --operation DOTA_test \
    --model 50 --weight_path ./checkpoint/r50-scale=[0.5.1.0].pth \
    --test_size 4000 --output_id 0
```
The results files will appear at "./result/test0/test0_final", which can be subsequently sent to the [DOTA server](http://bed4rs.net:8001/login/) to obtain the evaluation results.
```
# Multiscale scale testing
python evaluate.py --operation DOTA_MS_test \
    --model 50 --weight_path ./checkpoint/r50-scale=[0.5.1.0].pth \
    --test_image_dir ./DOTA_devkit/datasets/DOTA/test4000/images
```
The results files will appear at "./result/test/test_final"

For HRSC2016 or UCAS-AOD dataset:
```
# Single scale testing
python evaluate.py --operation HRSC_test \
    --model 50 --weight_path ${WEIGHT_PATH} \
    --hrsc_test_size 640 
```
```
# Multiscale scale testing
python evaluate.py --operation HRSC_MS_test \
    --model 50 --weight_path ${WEIGHT_PATH} 
```
## Citation

If you find this code useful for your research, please consider citing:
```
@article{zheng2023gpsgaussian,
  title={GPS-Gaussian: Generalizable Pixel-wise 3D Gaussian Splatting for Real-time Human Novel View Synthesis},
  author={Zheng, Shunyuan and Zhou, Boyao and Shao, Ruizhi and Liu, Boning and Zhang, Shengping and Nie, Liqiang and Liu, Yebin},
  journal={arXiv},
  year={2023}
```
