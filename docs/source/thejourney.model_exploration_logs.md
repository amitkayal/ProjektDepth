﻿# 7. Dataset and Model Exploration Log

## Ideas
- Train the model on `96x96` images then move on to larger `192x192` images, since i made the images dataset of `200x200` i'll have to resize them, dumb me, should have thought of taking the size in multiple of `32`, that's what the GPU 💖s.
- Encoder-Decoder Model ? might be overkill ? should try it though, but seems like i could get away without using ED model
- Go through the various loss functions: [https://pytorch.org/docs/stable/nn.html#loss-functions](https://pytorch.org/docs/stable/nn.html#loss-functions)
- First train the network  images (Encoder) so it learns to identify the objects and segment them, then teach the network to do depth estimation on large images as well as segmentation. - [X] Nope we'll train the entire model with smaller images first and then train with large images
- [https://github.com/fastai/fastai/blob/master/fastai/layers.py#L202](https://github.com/fastai/fastai/blob/master/fastai/layers.py#L202) read this to understand how to initialize the pixel shuffle
- Use [https://github.com/OniroAI/MonoDepth-PyTorch/blob/master/models_resnet.py](https://github.com/OniroAI/MonoDepth-PyTorch/blob/master/models_resnet.py) for reference 
- Use PreActivated ResNetV2, since its proven to be working better

There are so many mistakes in sir's code, why did he use interpolation ? why not use deconvolution ? it will not introduce the checkerboard issue. I figures i should use PixelShuffle, let's see how good it works


- Use Pixel Shuffel algorithm to increase resolution
- Maybe Use DepthWise Separation Convolution (MobileNet uses this)
- Use ResNeXt like architecture, let each kernel choose which object to segment
- Use IoU as a metric to tell how good my model is
- Try DiceLoss ?
- Try BCE Loss ?
- Try DICE  + BCE Loss
- Try Jaccard Loss ? (Hybrid Loss as mentioned in Stacked Unet)
- Try image gradient loss ? from here [https://github.com/wolverinn/Depth-Estimation-PyTorch](https://github.com/wolverinn/Depth-Estimation-PyTorch)
- Read this [https://heartbeat.fritz.ai/research-guide-for-depth-estimation-with-deep-learning-1a02a439b834](https://heartbeat.fritz.ai/research-guide-for-depth-estimation-with-deep-learning-1a02a439b834)
- Study [https://github.com/wolverinn/Depth-Estimation-PyTorch/blob/master/fyn_main.py](https://github.com/wolverinn/Depth-Estimation-PyTorch/blob/master/fyn_main.py)
- Sudy UNet
- Study ResNeXt
- Add a pre convolution

Higher Batch Size, Higher Learning Rate

Make Unet-Resnet Architecture, Then add Pixel Shuffling, Then use ResNeXt

Combine all these ideas to make the final model, All the best

BOOOO

## TODO

- [x] Make Unet-Resnet
- [ ] Try the different loss functions
- [x] Add Pixel Shuffling
- [x] Check if everything works with ResNet and smaller dataset
- [x] Add ResNeXt 
- [ ] Try DataAugmentation
- [ ] ~~Check if everything works with ResNeXt with smaller dataset~~
- [x] Create a Deeper Network
- [x] Reduce the Filters on the Segmentation Decoder
- [ ] Make the library
- [ ] Add save model checkpoint
- [ ] Add Tensorboard
- [ ] Train ! Train ! Train !

## Model

```
Encoding -> Bridge -> Decoder1
                   -> Decoder2
 ```

## Requirements

- 70-30 train-test split


## Notes

1.  All convolution operations are with 3*3 filters and with the SAME padding thus the size of the feature map remains the same on each level of contracting path and corresponding expanding path. With the same padding, the boundary information is preserved and it also allows for more convolutions to be added.
2.  Because the feature size remains the same on a single level, cropping of the feature map from the contracting path is not required in order to concatenate with the corresponding feature map of the expanding path. No cropping means no loss of information.
3.  Along with the long skip connection between every level of contracting and expanding paths, we have local skip connection between convolutions on each level. Skip connection helps in getting a smooth loss curve and also helps to avoid gradient disappearance and explosion.


## Colab Accounts Management

- Dataset Referenced Links : shadowleaf.deeplearning@gmail.com
- Actual Dataset: shadowleaf.contact@gmail.com
- Notebooks: shadowleaf.contact@gmail.com


## Further Improvements

- Try better quality images dataset, probably i compressed the jpeg too much

## Targets

- 11th May - Research about the possible models
- 12th May - Research about the shortlisted models
- 13th May - Create Models Conceptually, Research on Possible Loss Functions
- 14th May - Implement the Models and create the smaller dataset of 96x96
- 15th May - Try different loss functions and train with smaller dataset, ~~Create the library therefore with Tensorboard~~ 
- 16th May - ~~Train Train Train~~ Failed at running the model
- 17th May - ~~Train Train Train~~ Failed at running the model
- 18th May - The model works ! no memory leaks !
- 19th May - Tested the model works on TPU, reduced time by half, lower the model size ? create the library, fix the model
- 20th May - Create the library and train on`96x96`
