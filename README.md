# History

## 24.03.27
- Repository fork
- Create docker environment
```bash
docker run -v /your/code/dir/PyTorch-Spiking-YOLOv3/:/code -v /data/dir/:/data -it --gpus=all --ipc=host --name yolov3_snn --shm-size=32g pytorch/pytorch:2.1.2-cuda12.1-cudnn8-devel
```
- yolov3-tiny-ours.cfg -> **yolov3-tiny-mp2conv-mp1none-lk2relu-up2tconv.cfg**
- Additional requirements (at docker env.)
```bash
apt-get update
apt-get upgrade -y
apt-get install -y git libgl1-mesa-glx libglib2.0-0
pip install tensorboard matplotlib apex pycocotools opencv-python scikit-image
pip install numpy==1.23.0
```
- Using COCO2017 dataset
> - Download link: [train](http://images.cocodataset.org/zips/train2017.zip), [validation](http://images.cocodataset.org/zips/val2017.zip), [test](http://images.cocodataset.org/zips/test2017.zip) and [labels](https://github.com/WongKinYiu/yolov7/releases/download/v0.1/coco2017labels-segments.zip)
- Update data cfg loader: load data from .yaml ([data/coco.yaml](data/coco.yaml))
- [train.py](/train.py) debugging
- Bug when loading labels

## 24.04.02
- w/ COCO2017 have to be debugged more
- Training, validation w/ COCO2014 code dubugging
> - Download link: [train](http://images.cocodataset.org/zips/train2014.zip), [validation](http://images.cocodataset.org/zips/val2014.zip), [test](http://images.cocodataset.org/zips/test2014.zip) and [train/val annotations](http://images.cocodataset.org/annotations/annotations_trainval2014.zip), [test annotations](http://images.cocodataset.org/annotations/image_info_test2014.zip)
---

# PyTorch-Spiking-YOLOv3
A PyTorch implementation of Spiking-YOLOv3, based on the PyTorch implementation of YOLOv3([ultralytics/yolov3](https://github.com/ultralytics/yolov3)), with support for Spiking-YOLOv3-Tiny at present. The whole Spiking-YOLOv3 will be supported soon.

## Introduction
For spiking implementation, some operators in YOLOv3-Tiny have been converted equivalently. Please refer to yolov3-tiny-ours(\*).cfg in /cfg for details.
### Conversion of some operators
+ 'maxpool(stride=2)'->'convolutional(stride=2)'
+ 'maxpool(stride=1)'->'none'
+ 'upsample'->'transposed_convolutional'
+ 'leaky_relu'->'relu'
+ 'batch_normalization'->'fuse_conv_and_bn'

## Usage
Please refer to [ultralytics/yolov3](https://github.com/ultralytics/yolov3) for the basic usage for training, evaluation and inference. The main advantage of PyTorch-Spiking-YOLOv3 is the transformation from ANN to SNN.
### Train
```
$ python3 train.py --batch-size 32 --cfg cfg/yolov3-tiny-ours.cfg --data data/coco.data --weights ''
```
### Test
```
$ python3 test.py --cfg cfg/yolov3-tiny-ours.cfg --data data/coco.data --weights weights/best.pt --batch-size 32 --img-size 640
```
### Detect
```
$ python3 detect.py --cfg cfg/yolov3-tiny-ours.cfg --weights weights/best.pt --img-size 640
```
### Transform
```
$ python3 ann_to_snn.py --cfg cfg/yolov3-tiny-ours.cfg --data data/coco.data --weights weights/best.pt --timesteps 128
```
For higher accuracy(mAP), you can try to adjust some hyperparameters.

*Trick: the larger timesteps, the higher accuracy.*

## Results
Here we show the results(mAP) of PASCAL VOC & COCO which are commonly used in object detection，and two custom datasets UAV & UAVCUT.
|  dataset  |  yolov3  |  yolov3-tiny  |  yolov3-tiny-ours  |  yolov3-tiny-ours-snn  |
|  ----  |  ----  |  ----  |  ----  |  ----  |
|  UAVCUT  |  98.90%  |  99.10%  |  **98.80%**  |  **98.60%**  |
|  UAV  |  99.50%  |  99.40%  |  **99.10%**  |  **98.20%**  |
|  VOC07+12  |  77.00%  |  52.30%  |  **55.50%**  |  **55.56%**  |
|  COCO2014  |  56.50%  |  33.30%  |  **38.70%**  |  **29.50%**  |

From the results, we can conclude that: 
1) for simple custom datasets like UAV & UAVCUT, the accuracy of converting some operators is nearly equivalent to the original YOLOv3-Tiny; 
2) for complex common datasets like PASCAL VOC & COCO, the accuracy of converting some operators is even better than the original YOLOv3-Tiny; 
3) for most datasets, our method of transformation from ANN to SNN can be nearly lossless;
4) for rather complex dataset like COCO, our method of transformation from ANN to SNN causes a certain loss of accuracy(which will been improved later).

UAVCUT

![avatar](/assets/uavcut.png)

UAV

![avatar](/assets/uav.png)

PASCAL VOC

![avatar](/assets/voc.jpg)

COCO

![avatar](/assets/coco.jpg)

## References
### Articles
+ [Theory and Tools for the Conversion of Analog to Spiking Convolutional Neural Networks](https://arxiv.org/abs/1612.04052)
+ [Spiking-YOLO: Spiking Neural Network for Energy-Efficient Object Detection](https://arxiv.org/abs/1903.06530)
### GitHub
+ [NeuromorphicProcessorProject/snn_toolbox](https://github.com/NeuromorphicProcessorProject/snn_toolbox)
+ [hahnyuan/ANN2SNN](http://git.wildz.cn/hahnyuan/ANN2SNN)
