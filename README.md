# Vehicle trajectory prediction for driver assistance systems

This project focuses on vehicle trajectory prediction using the monocular image of the KITTI dataset.

### Prediction 1s → 2s

![](doc/scene_10_20.gif)

### Prediction 2s → 1s
![](doc/scene_20_10.gif)



## Motivation
In addition to vehicle movement prediction, this work also deals with the issue
creating a suitable representation of the surrounding environment. Its goal is to act only using monocular recording, which, compared to the use of expensive sensors, requires only one camera.
Based on this, I propose a model consisting of two parts. First part
it is responsible for creating a suitable representation of the environment, and the second part processes this representation and creates a prediction.




## Setup

The system was developed on Ubuntu 18.04 64-bit using Python 3.8.

To create the conda environment run

`conda create --name <env_name> --file requirements.txt --python=3.8`

Activate the environment by

`conda activate <env_name>`

1. Install [ORB_SLAM2](https://github.com/raulmur/ORB_SLAM2)
2. For ORB_SLAM2 to work in a Python environment, it is needed to install [ORB_SLAM2-PythonBindings](https://github.com/jskinn/ORB_SLAM2-PythonBindings)

A graphics card with a CUDA computing platform is required to run the system.

### Models for preprocessing

The trained models used are available at the link https://drive.google.com/drive/folders/1yCVw2ORS1v4qe3tx9Lv5e1eYRI6nP1oQ?usp=sharing.

These models need to be placed following:

mono+stereo_640x192: `depth_estimation/monodepth2/models/`

panoptic_fpn_R_101_3x.pkl: `semantic_segmentation/`

test.pth: `object_detection_3d/GUPNet/code/checkpoints/`

### Predikčné modely

The trained prediction models are available at the link https://drive.google.com/file/d/1FrGpkrq2iCrAwgR5_tMNF7jD7llFWl2E/view?usp=sharing. The contents of the `.zip` file must be extracted into  
`prediction/` folder.

## Dataset preparation

The system works with the monocular recording of the KITTI dataset, specifically with the scenes from the kiti_raw and kitti_tracking parts. Before using the program, it is necessary to organize the file structure of the dataset according to the following form.

```
├── kitti_raw
|   ├── 2011_09_26
|   |   |   ...
|   │   └── 2011_09_26_drive_0009_sync
|   │       └── image_02
|   │           └── data
|   ├── 2011_09_28
|   |   |   ...
|   │   └── 2011_09_28_drive_0002_sync
|   │       └── image_02
|   │           └── data
|   ├── 2011_09_29
|   |   |   ...
|   │   └── 2011_09_29_drive_0071_sync
|   │       └── image_02
|   │           └── data
|   ├── 2011_09_30
|   |   |   ...
|   │   └── 2011_09_30_drive_0034_sync
|   │       └── image_02
|   │           └── data
|   └── 2011_10_03
|       |   ...
|       └── 2011_10_03_drive_0047_sync
|           └── image_02
|               └── data
|
└── kitti_tracking
    └── data_tracking_image_2
        ├── testing
        │   └── image_02
        │       |   ...
        │       └── 0017
        └── training
            └── image_02
                |   ...
                └── 0015
```

The data processed into the form used by the prediction model is available at the link  https://drive.google.com/drive/folders/1Nj-Wa8nCbfe2yfTS3imTGUKzEZJ7v7Yw?usp=sharing.

## Image preprocessing

In the `config/*.yaml` configuration files, before starting the image processing scripts, the directory with the KITTI dataset and the target directory where the processed data should be saved must be set.

The original size image is processed using the command

`python generate_dataset.py config/kitti_big.yaml`

Downsampled image is processed by the command

`python generate_dataset.py config/kitti_small.yaml`

## Training of prediction models

The configurations of the trained prediction models are located in the `prediction/config` directory. `.yaml` files intended for training on data obtained from the original size image are located in the `big_input` folder. `.yaml` files from the `small_input` folder can be used to configure training models from reduced images. In the individual configuration files, a folder with pre-processed data must be set before the actual training.



Before running the scripts itself, it is necessary to move to the `prediction/` folder. For each configuration, the parts of the model must be trained in this order

1. auto-encoder: `python train_autoencoder.py <config>`
2. memory-controller: `python train_memory_controller.py <config>`
3. iterative refinement module: `python train_itrefmodule.py <config>`

Using the `train_all.sh` script it is possible to train models for each configuration.

## Testing
This system is inadequate compared to the best methods and makes an error that is unacceptable in the industry. Also, the image processing time does not achieve a reliability that approaches real-time performance. 

Before actually running the scripts for testing, it is necessary to move to the `prediction/` folder.
Models can be tested using the command

`python test_configs.py <config>`

Using the script `test_all.sh' it is possible to test the models created by each of the configurations.


## Visualization


The visualization of the projected predictions into the image can be started using a script

`python run.py <prediction_config> <preprocessing_config> <data_path>`



## Reference

Depth Estimation - [monodepth2](https://github.com/nianticlabs/monodepth2)

Semantic Segmentation - [detectron2](https://github.com/facebookresearch/detectron2)

3D Vehicle Detection - [GUPNet](https://github.com/SuperMHP/GUPNet)

Camera Localization - [ORB_SLAM2](https://github.com/raulmur/ORB_SLAM2)

Python bindings for SLAM -[ORB_SLAM2-PythonBindings](https://github.com/jskinn/ORB_SLAM2-PythonBindings)

Trajectory Prediction - [MANTRA-CVPR20](https://github.com/Marchetz/MANTRA-CVPR20)
