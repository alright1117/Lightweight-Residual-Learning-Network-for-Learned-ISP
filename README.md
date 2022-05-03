# Deep Learning for Smartphone ISP 

## Overview 

[**[Challenge Website]**](https://competitions.codalab.org/competitions/28054) [**[Workshop Website]**](http://ai-benchmark.com/workshops/mai/2021/)

This repository provides the implementation of our model, for the [***Learned Smartphone ISP*** Challenge](https://competitions.codalab.org/competitions/28054) in [*Mobile AI (MAI) Workshop @ CVPR 2021*](http://ai-benchmark.com/workshops/mai/2021/). We proposed a very compact CNN model with a local fusion block. This block consists of two parts: multiple stacked adaptive weight residual units, termed as RRDB (residual in residual dense block), and a local residual fusion unit (LRFU). The RRDB module can improve the information flow and gradients, while the LRFU module can effectively fuse multi-level residual information in the local fusion block. The model is trained to convert **RAW Bayer data** obtained directly from mobile camera sensor into photos captured with a professional *Fujifilm DSLR* camera, thus replacing the entire hand-crafted ISP camera pipeline. The provided pre-trained model can be used to generate full-resolution **12MP photos** from RAW image files captured using the *Sony IMX586* camera sensor. We received 7th place in this challenge.

### Contents:
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Dataset and model preparation](#dataset-and-model-preparation)
- [Learned ISP Pipeline](#learned-isp-pipeline)
- [Training](#training)
  - [Start training](#start-training)
  - [Resume training](#resume-training)
- [Test/Inference](#testinference)
  - [Use the model](#use-the-self-obtained-model)
- [Folder structure (default)](#folder-structure-default)
- [Reference](#reference)

---
### Prerequisites

- Python: numpy, scipy, imageio and pillow packages
- [TensorFlow 1.15.0](https://www.tensorflow.org/install/) + [CUDA cuDNN](https://developer.nvidia.com/cudnn)
- GPU for training (e.g., Nvidia GeForce GTX 1080)

[[back]](#contents)
<br/>

---
### Dataset and model preparation

- Download training data and extract it into `raw_images/train/` folder.    
- Download validation data and extract it into `raw_images/val/` folder.
- Download testing data and extract it into `raw_images/test/` folder.    
  
- [Optional] Download pre-trained [VGG-19 model](https://polybox.ethz.ch/index.php/s/7z5bHNg5r5a0g7k) <sup>[Mirror](https://drive.google.com/file/d/0BwOLOmqkYj-jMGRwaUR2UjhSNDQ/view?usp=sharing)</sup> and put it into `vgg_pretrained/` folder.  

[[back]](#contents)
<br/>

---
### Learned ISP Pipeline

1. deBayer pre-processing (in [`load_dataset.py`](load_dataset.py)):
    * *Input*: RAW data [`H x W x 1`]
    * *Output*: deBayer RAW data [`(H/2) x (W/2) x 4`]  
2.  model (in [`model.py`](model.py)):
    * *Input*: deBayer RAW data [`(H/2) x (W/2) x 4`]
    * *Output*: RGB image [`H x W x 3`]

[[back]](#contents)
</br>

---
### Training

#### Start training
To train the model, use the following command:

```bash
python train_model.py
```

Optional parameters (and default values):

>```dataset_dir```: **```raw_images/```** &nbsp; - &nbsp; path to the folder with the dataset <br/>
>```model_dir```: **```models/```** &nbsp; - &nbsp; path to the folder with the model to be restored or saved <br/>
>```vgg_dir```: **```vgg_pretrained/imagenet-vgg-verydeep-19.mat```** &nbsp; - &nbsp; path to the pre-trained VGG-19 network <br/>
>```dslr_dir```: **```fujifilm/```** &nbsp; - &nbsp; path to the folder with the RGB data <br/>
>```phone_dir```: **```mediatek_raw/```** &nbsp; - &nbsp; path to the folder with the Raw data <br/>
>```arch```: **```model```** &nbsp; - &nbsp; architecture name <br/>
>```num_maps_base```: **```16```** &nbsp; - &nbsp; base channel number (e.g. 8, 16, 32, etc.) <br/>
>```restore_iter```: **```None```** &nbsp; - &nbsp; iteration to restore <br/>
>```patch_w```: **```256```** &nbsp; - &nbsp; width of the training images <br/>
>```patch_h```: **```256```** &nbsp; - &nbsp; height of the training images <br/>
>```batch_size```: **```32```** &nbsp; - &nbsp; batch size [small values can lead to unstable training] <br/>
>```train_size```: **```5000```** &nbsp; - &nbsp; the number of training patches randomly loaded each 1000 iterations <br/>
>```learning_rate```: **```5e-5```** &nbsp; - &nbsp; learning rate <br/>
>```eval_step```: **```1000```** &nbsp; - &nbsp; each ```eval_step``` iterations the accuracy is computed and the model is saved <br/>
>```num_train_iters```: **```100000```** &nbsp; - &nbsp; the number of training iterations <br/>

</br>

Below we provide an example command used for training the mode.

```bash
CUDA_VISIBLE_DEVICES=0 python train_model.py \
  model_dir=models/punet_MAI/ arch=punet num_maps_base=16 \
  patch_w=256 patch_h=256 batch_size=32 \
  eval_step=1000 num_train_iters=100000
```

After training, the following files will be produced under `model_dir`:
>```checkpoint```                                 &nbsp; - &nbsp; contain all the checkpoint names <br/>
>```logs_[restore_iter]-[num_train_iters].txt```  &nbsp; - &nbsp; training log (including loss, PSNR, etc.) <br/>
>```[arch]_iteration_[iter].ckpt.data```          &nbsp; - &nbsp; part of checkpoint data for the model `[arch]_iteration_[iter]` <br/>
>```[arch]_iteration_[iter].ckpt.index```          &nbsp; - &nbsp; part of checkpoint data for the model `[arch]_iteration_[iter]` <br/>

#### Resume training
To resume training from `restore_iter`, use the command like follows:

```bash
CUDA_VISIBLE_DEVICES=0 python train_model.py \
  model_dir=models/punet_MAI/ arch=punet num_maps_base=16 \
  patch_w=256 patch_h=256 batch_size=32 \
  eval_step=1000 num_train_iters=110000 restore_iter=100000 
```

[[back]](#contents)
<br/>

---
### Test/Inference
`test_model.py` runs a model on testing images with the height=`img_h` and width=`img_w`. Here we use `img_h=1088` and `img_w=1920` as the example. If `save=True`, the protobuf (frozen graph) that corresponds to the testing image resolution will also be produced.

<br/>

#### Use the model

To produce output images and protobuf using the self-trained model, use the following command:

```bash
python test_model.py
```

Optional parameters (and default values):

>```dataset_dir```: **```raw_images/```** &nbsp; - &nbsp; path to the folder with the dataset <br/>
>```test_dir```: **```fujifilm_full_resolution/```** &nbsp; - &nbsp; path to the folder with the test data <br/>
>```model_dir```: **```models/```** &nbsp; - &nbsp; path to the folder with the models to be restored/loaded <br/>
>```result_dir```: **```results/```** &nbsp; - &nbsp; path to the folder with the produced outputs from the loaded model <br/>
>```arch```: **```model```** &nbsp; - &nbsp; architecture name <br/>
>```num_maps_base```: **```16```** &nbsp; - &nbsp; base channel number (e.g. 8, 16, 32, etc.) <br/>
>```orig```: **```True```**, **```False```** &nbsp; - &nbsp; use the pre-trained model or not <br/>
>```restore_iter```: **```None```** &nbsp; - &nbsp; iteration to restore (when not specified with self-train model, the last saved model will be loaded)<br/>
>```img_h```: **```1088```** &nbsp; - &nbsp; width of the testing images <br/>
>```img_w```: **```1920```** &nbsp; - &nbsp; height of the testing images <br/>
>```use_gpu```: **```True```**,**```False```** &nbsp; - &nbsp; run the model on GPU or CPU <br/>
>```save```: **```True```** &nbsp; - &nbsp; save the loaded check point and protobuf (frozed graph) again <br/>
>```test_image```: **```True```** &nbsp; - &nbsp; run the loaded model on the test images. Can set as **```False```** if you only want to save models. <br/>

</br>

* *Example 1*: For evaluating PSNR for validation data (resolution: *256x256*) and generating processed images:
```bash
CUDA_VISIBLE_DEVICES=0 python test_model.py \
  test_dir=mediatek_raw/ model_dir=models/punet_MAI/ result_dir=results/ \
  arch=punet num_maps_base=16 orig=False restore_iter=98000 \
  img_h=256 img_w=256 use_gpu=True save=True test_image=True
```
* *Example 2*: For evaluating the inference latency (resolution: *1088x1920*) without generating any output images:
```bash
CUDA_VISIBLE_DEVICES=0 python test_model.py \
  model_dir=models/punet_MAI/ \
  arch=punet num_maps_base=16 orig=False restore_iter=98000 \
  img_h=1088 img_w=1920 use_gpu=True save=True test_image=False
```

[[back]](#contents)
</br>

---
### Folder structure

>```models/```             &nbsp; - &nbsp; logs and models that are saved during the training process <br/>
>```raw_images/```         &nbsp; - &nbsp; the folder with the dataset <br/>
>```results/```            &nbsp; - &nbsp; visual results for the produced images <br/>
>```vgg-pretrained/```     &nbsp; - &nbsp; the folder with the pre-trained VGG-19 network <br/>

>```load_dataset.py```     &nbsp; - &nbsp; python script that loads training data <br/>
>```model.py```            &nbsp; - &nbsp; PUNET implementation (TensorFlow) <br/>
>```train_model.py```      &nbsp; - &nbsp; implementation of the training procedure <br/>
>```test_model.py```       &nbsp; - &nbsp; applying the trained model to testing images <br/>
>```utils.py```            &nbsp; - &nbsp; auxiliary functions <br/>
>```vgg.py```              &nbsp; - &nbsp; loading the pre-trained vgg-19 network <br/>
>```ckpt2pb.py```          &nbsp; - &nbsp; convert checkpoint to protobuf (frozen graph) <br/>
>```pb2tflite.sh```        &nbsp; - &nbsp; bash script that converts protobuf to tflite <br/>

[[back]](#contents)
<br/>

---

### Convert checkpoint to pb

`test_model.py` can produce protobuf automatically if `save=True`. 
If you want to directly convert the checkpoint model (including `.meta`, `.data`, and `.index`) to protobuf, use `ckpt2pb.py` to do so.
The main arguments (and default values) are as follows:

>```--in_path```: **```models/punet_MAI/punet_iteration_100000.ckpt```**  &nbsp; - &nbsp; input checkpoint file (including `.meta`, `.data`, and `.index`) <br/>
>```--out_path```: **```models/punet_MAI/punet_iteration_100000.pb```**   &nbsp; - &nbsp; output protobuf file <br/>
>```--out_nodes```: **```output_l0```**                                   &nbsp; - &nbsp; output node name <br/>

Notes:
1. As mentioned earlier, the **output node name** needs to be specified. There are two ways to check the output node name:
    * check the graph in [Tensorboard](https://www.tensorflow.org/tensorboard).
    * directly specify the node name in the source code (e.g. use `tf.identity`).
2. `.meta` is necessary to convert a checkpoint to protobuf since it contains the important model information (e.g. architecture, input size, etc.).

Below we provide an example command:

```bash
python ckpt2pb.py \
  --in_path models/punet_MAI/punet_iteration_100000.ckpt \
  --out_path models/punet_MAI/punet_iteration_100000.pb \
  --out_nodes output_l0
```

[[back]](#contents)
</br>

---
### Convert pb to tflite

The last step is converting the frozen graph to TFLite so that the evaluation server can evaluate the performance on MediaTek devices. Please use the official Tensorflow function `tflite_convert`. The main arguments (and default values) are as follows:

>```graph_def_file```: **```models/original/punet_pretrained.pb```**  &nbsp; - &nbsp; input protobuf file <br/>
>```output_file```: **```models/original/punet_pretrained.tflite```** &nbsp; - &nbsp; output tflite file <br/>
>```input_shape```: **```1,544,960,4```**                           &nbsp; - &nbsp; the network input, which is after debayering/demosaicing. If the raw image shape is `(img_h, img_w, 1)`, `input_shape` should be `(img_h/2, img_w/2, 4)`. <br/>
>```input_arrays```: **```Placeholder```**                            &nbsp; - &nbsp; input node name (can be found in [Tensorboard](https://www.tensorflow.org/tensorboard), or specified in source codes) <br/>
>```output_arrays```: **```output_l0```**                             &nbsp; - &nbsp; output node name (can be found in [Tensorboard](https://www.tensorflow.org/tensorboard), or specified in source codes) <br/>

An example command:

```bash
tflite_convert \
  --graph_def_file=models/punet_MAI/punet_iteration_100000.pb \
  --output_file=models/punet_MAI/punet_iteration_100000.tflite \
  --input_shape=1,544,960,4 \
  --input_arrays=Placeholder \
  --output_arrays=output_l0
```

[[back]](#contents)
</br>

---

### Reference
This code borrows heavily from [MediaTek-NeuroPilot/mai21-learned-smartphone-isp](https://github.com/MediaTek-NeuroPilot/mai21-learned-smartphone-isp). The model architecture is inspired by this paper "Residual dense network for image super-resolution" Zhang, Yulun, et al. in CVPR 2018.


[[back]](#contents)
<br/>

---

