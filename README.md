# Power estimation in slum areas using satellite images
According to statistics, slums use most power per unit area after industrial plants. Some studies have shown that more than half of the power distributed in Mumbai is being used up in slum areas. <br/>
We propose a model that separates out slum areas in satellite images that can help prdeict the amount of power needed by an area. <br/>
Our project uses Mumbai as an example city for the purpose.


<p align="center">
  <img src="https://github.com/cbsudux/Mumbai-slum-segmentation/blob/master/assets/images/combined-intro.png" >
</p>

## Mumbai Slums

Mumbai is one of the most populous and wealthiest cities in India. However, it is also home to some of the world’s biggest slums -- **Dharavi, Mankhurd-Govandi belt, Kurla-Ghatkopar belt, Dindoshi and The Bhandup-Mulund slums**. The number of slum-dwellers in Mumbai is estimated to be around 9 million, up from 6 million in 2001 that is, 62% of of Mumbai live in informal slums.

![dharavi-govandi](/assets/images/dh-govandi.png)

![kurla](/assets/images/kurla.jpg)

## What did we do?

Any intitative on slum rehabitiation and improvement relies heavily on **slum mapping** and **monitoring**. When we spoke to the relevant authorities, we found out that they mapped slums manually (human annotators), which takes a substantial amount of time. We realised we could automate this and used a deep learning approach to **segment and map individual slums from satellite imagery**. In addition, we also wrote code to **perform change detection and monitor slum change over time**. Slum change detection is an important task and analysing increase/decrease of a slum can provide valuable insights into the power usage of the area.

## How did we go about it?

We curated a **dataset** containing 3-band (RGB) satellite imagery with 65 cm per pixel resolution
collected from Google Earth. Each image has a pixel size of 1280x720. The satellite imagery covers most of
Mumbai and we include images from 2002 to 2018, to analyze slum change. We used 513 images for training, and 97 images for testing. (Unfortunately, we cannot redistribute the dataset, due to Google policy.)

For **slum segmentation and mapping**, we trained a Mask R-CNN on our custom dataset. Check our [github readme](https://github.com/cbsudux/Mumbai-slum-segmentation/tree/master/slums) for our training and testing approaches, and our [paper](https://arxiv.org/abs/1811.07896) for more details.  

![kurla result](/assets/images/kurla-result_2.png)

For **slum change detection**, we took a pair of satellite images, representing the same location at different points of time. We predicted masks for both these images and then subtracted the masks to obtain a percentage icrease/decrease. The following images (below) show a change of +35.25% between 2018 (top row) and 2005 (bottom row) of the same slum.    

![change result](/assets/images/change.png)

# Training Details.

This file contains details about training and inference for slum segmentation using Mask RCNN.



## Installation
Use this Google Drive link to download the pre-trained weights: 
* Download `mask_rcnn_slum_600_00128.h5` and save it in root directory.
* Link : https://drive.google.com/file/d/1IIMZLrdCZXY_dA540Ve9lSJplYHLnTY4/view?usp=sharing
* Use `sudo pip3 install -r requirements.txt` to install relevant dependencies.
* Then `cd slums` to get into the project directory.

## Dataset
Dataset of satellite images can be created using Google Earth's desktop application. For our project, we used 720X1280 images at 1000m and 100m views, from various Mumbai slums. Google's policy states that we cannot redistribute the dataset. 

Also, we recommend using VGG Image Annotator tool for annotating the segmentation masks as the code is written for that format. The tool gives the annotations in the form of a JSON file, that should be placed inside the dataset folder as follows:
```
dataset/
        train/
            all training images
            train.json
        val/
            all val images
            val.json
```

Here are few links to help you to curate your own dataset: <br>
https://productforums.google.com/forum/#!msg/maps/8KjNgwbBzwc/4kNMfXB6CAAJ <br>
https://support.google.com/earth/answer/148146?hl=en <br>
http://www.robots.ox.ac.uk/~vgg/software/via/ <br>
## Training the model

Train a new model starting from pre-trained COCO weights
```
python3 slum.py train --dataset=/path/to/slum/dataset --weights=coco
```

Resume training a model that you had trained earlier
```
python3 slum.py train --dataset=/path/to/slum/dataset --weights=last
```

Train a new model starting from ImageNet weights
```
python3 slum.py train --dataset=/path/to/slum/dataset --weights=imagenet
```

* The training details are specified inside slum.py code.
* The model will save every checkpoint in root/logs folder.
* The logs folders are timestamped according to start time and also have tensorboard visualizations.


## Inference or testing the model
Testing mode, where a segmentation mask is applied on the detected instances. Make sure to place the images inside ```test_images``` folder.

```bash
python3 testing.py  --weights=/path/to/mask_rcnn/mask_rcnn_slum.h5 
```
This will save the detections (if any) for all the images in `test_images` and save it in `test_outputs`.

Apply splash effect on a video. Requires OpenCV 3.2+:
Segments out instances and applies masks on a video.
```bash
python3 slum.py splash --weights=/path/to/mask_rcnn/mask_rcnn_slum.h5 --video=<file name or URL>
```
## Change Detection
For detecting percentage change in masks, place the two images in ```change_det/ ``` folder and run:

```bash
python3 change_detection.py  --weights=/path/to/mask_rcnn/mask_rcnn_slum.h5 
```
