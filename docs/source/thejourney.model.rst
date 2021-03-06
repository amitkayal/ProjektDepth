1. The Model
============

GitHub Link : `<https://github.com/satyajitghana/ProjektDepth/blob/master/notebooks/10_DepthModel_ModelImprovisation.ipynb>`_
Colab Link  : `<https://colab.research.google.com/github/satyajitghana/ProjektDepth/blob/master/notebooks/10_DepthModel_ModelImprovisation.ipynb>`_

Given the task to do monocular depth estimation and also segmentation, we researched
upon a few possibilities

The obvious answer was to use a Encoder-Decoder architecture, since the reference model
we found were these

1. `<https://github.com/OniroAI/MonoDepth-PyTorch/>`_
2. `<https://github.com/wolverinn/Depth-Estimation-PyTorch>`_
3. `<https://heartbeat.fritz.ai/research-guide-for-depth-estimation-with-deep-learning-1a02a439b834>`_

All of them using a variation of Encoder-Decoder architecture

Various Papers were surveyed to find that UNet with Residual connections work best, also multiple encoder-decoder networks can be combined together
to form a new network.

1. Stacked U-Nets: A No-Frills Approach to Natural Image Segmentation: `<https://arxiv.org/pdf/1804.10343.pdf>`_
: This inspired me to stack the decoder part of my network to give two outputs

2. Depth Estimation and Semantic Segmentation from a Single RGB Image Using a Hybrid Convolutional Neural Network: `<https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6514714/>`_
: Various loss functions were inspired from this

3. Road Extraction by Deep Residual U-Net `<https://arxiv.org/abs/1711.10684>`_
: I referred this to create the ResUNet

4. Transfer Learning for Brain Tumor Segmentation `<https://arxiv.org/abs/1912.12452>`_
: Brain Tumor Segmentation done using UNet

5. Image Super-Resolution Using Deep Convolutional Networks `<https://arxiv.org/pdf/1501.00092>`_
: For the Decoder Network to prevent checkerboard issues, PixelShuffle was used

So we went with that combined with my knowledge that residual networks have proved to be really
good, and for segmentation a UNet kind of architecture is good, finally we created a custom Residual-UNet
architecture, with Single Encoder - Double Decoder and Residual Connections between them, the model diagram will
make it more clear to understand how these residual connections between the decoders and the encoder play out.

The model was made over 3 iterations, in which various configurations were tried, but the model was too densly connected
since we had used `concatenation` to add the residual connections, which created a huge backprop size, that couldn't fit into
memory, hence we converted the `concat` ops to `add` ops, various model param counts were also tried.

The input to our model will be `fg_bg` and `bg` image and output will be `depth_fg_bg` and `fg_bg_mask`

input: :math:`(3, 196, 196), (3, 196, 196)`

output: :math:`(1, 196, 196), (1, 196, 196)`

Model Architecture
##################

.. figure:: assets/ResUNet-V3-transparent.png
    :align: center
    :figclass: align-center

    ResUNet-V3

Total Params: 15,855,712

Summary of the Model
####################

.. code-block:: none

    ----------------------------------------------------------------
            Layer (type)               Output Shape         Param #
    ================================================================
                Conv2d-1         [-1, 32, 192, 192]           4,800
                Conv2d-2         [-1, 64, 192, 192]           2,048
        BatchNorm2d-3         [-1, 64, 192, 192]             128
        BatchNorm2d-4         [-1, 32, 192, 192]              64
                ReLU-5         [-1, 32, 192, 192]               0
                Conv2d-6         [-1, 64, 192, 192]          18,432
        BatchNorm2d-7         [-1, 64, 192, 192]             128
                ReLU-8         [-1, 64, 192, 192]               0
                Conv2d-9         [-1, 64, 192, 192]          36,864
        ResDoubleConv-10         [-1, 64, 192, 192]               0
            MaxPool2d-11           [-1, 64, 96, 96]               0
        ResDownBlock-12  [[-1, 64, 96, 96], [-1, 64, 192, 192]]               0
            Conv2d-13          [-1, 128, 96, 96]           8,192
        BatchNorm2d-14          [-1, 128, 96, 96]             256
        BatchNorm2d-15           [-1, 64, 96, 96]             128
                ReLU-16           [-1, 64, 96, 96]               0
            Conv2d-17          [-1, 128, 96, 96]          73,728
        BatchNorm2d-18          [-1, 128, 96, 96]             256
                ReLU-19          [-1, 128, 96, 96]               0
            Conv2d-20          [-1, 128, 96, 96]         147,456
        ResDoubleConv-21          [-1, 128, 96, 96]               0
            MaxPool2d-22          [-1, 128, 48, 48]               0
        ResDownBlock-23  [[-1, 128, 48, 48], [-1, 128, 96, 96]]               0
            Conv2d-24          [-1, 256, 48, 48]          32,768
        BatchNorm2d-25          [-1, 256, 48, 48]             512
        BatchNorm2d-26          [-1, 128, 48, 48]             256
                ReLU-27          [-1, 128, 48, 48]               0
            Conv2d-28          [-1, 256, 48, 48]         294,912
        BatchNorm2d-29          [-1, 256, 48, 48]             512
                ReLU-30          [-1, 256, 48, 48]               0
            Conv2d-31          [-1, 256, 48, 48]         589,824
        ResDoubleConv-32          [-1, 256, 48, 48]               0
            MaxPool2d-33          [-1, 256, 24, 24]               0
        ResDownBlock-34  [[-1, 256, 24, 24], [-1, 256, 48, 48]]               0
            Conv2d-35          [-1, 512, 24, 24]         131,072
        BatchNorm2d-36          [-1, 512, 24, 24]           1,024
        BatchNorm2d-37          [-1, 256, 24, 24]             512
                ReLU-38          [-1, 256, 24, 24]               0
            Conv2d-39          [-1, 512, 24, 24]       1,179,648
        BatchNorm2d-40          [-1, 512, 24, 24]           1,024
                ReLU-41          [-1, 512, 24, 24]               0
            Conv2d-42          [-1, 512, 24, 24]       2,359,296
        ResDoubleConv-43          [-1, 512, 24, 24]               0
            MaxPool2d-44          [-1, 512, 12, 12]               0
        ResDownBlock-45  [[-1, 512, 12, 12], [-1, 512, 24, 24]]               0
        BatchNorm2d-46          [-1, 512, 12, 12]           1,024
                ReLU-47          [-1, 512, 12, 12]               0
            Conv2d-48          [-1, 512, 12, 12]       2,359,296
        BatchNorm2d-49          [-1, 512, 12, 12]           1,024
                ReLU-50          [-1, 512, 12, 12]               0
            Conv2d-51          [-1, 512, 12, 12]       2,359,296
        ResDoubleConv-52          [-1, 512, 12, 12]               0
            Conv2d-53         [-1, 2048, 12, 12]       1,048,576
        PixelShuffle-54          [-1, 512, 24, 24]               0
            Conv2d-55          [-1, 512, 24, 24]         262,144
            Conv2d-56          [-1, 256, 24, 24]         131,072
        BatchNorm2d-57          [-1, 256, 24, 24]             512
        BatchNorm2d-58          [-1, 512, 24, 24]           1,024
                ReLU-59          [-1, 512, 24, 24]               0
            Conv2d-60          [-1, 256, 24, 24]       1,179,648
        BatchNorm2d-61          [-1, 256, 24, 24]             512
                ReLU-62          [-1, 256, 24, 24]               0
            Conv2d-63          [-1, 256, 24, 24]         589,824
        ResDoubleConv-64          [-1, 256, 24, 24]               0
        ResUpBlock-65          [-1, 256, 24, 24]               0
            Conv2d-66         [-1, 1024, 24, 24]         262,144
        PixelShuffle-67          [-1, 256, 48, 48]               0
            Conv2d-68          [-1, 256, 48, 48]          65,536
            Conv2d-69          [-1, 128, 48, 48]          32,768
        BatchNorm2d-70          [-1, 128, 48, 48]             256
        BatchNorm2d-71          [-1, 256, 48, 48]             512
                ReLU-72          [-1, 256, 48, 48]               0
            Conv2d-73          [-1, 128, 48, 48]         294,912
        BatchNorm2d-74          [-1, 128, 48, 48]             256
                ReLU-75          [-1, 128, 48, 48]               0
            Conv2d-76          [-1, 128, 48, 48]         147,456
        ResDoubleConv-77          [-1, 128, 48, 48]               0
        ResUpBlock-78          [-1, 128, 48, 48]               0
            Conv2d-79          [-1, 512, 48, 48]          65,536
        PixelShuffle-80          [-1, 128, 96, 96]               0
            Conv2d-81          [-1, 128, 96, 96]          16,384
            Conv2d-82           [-1, 64, 96, 96]           8,192
        BatchNorm2d-83           [-1, 64, 96, 96]             128
        BatchNorm2d-84          [-1, 128, 96, 96]             256
                ReLU-85          [-1, 128, 96, 96]               0
            Conv2d-86           [-1, 64, 96, 96]          73,728
        BatchNorm2d-87           [-1, 64, 96, 96]             128
                ReLU-88           [-1, 64, 96, 96]               0
            Conv2d-89           [-1, 64, 96, 96]          36,864
        ResDoubleConv-90           [-1, 64, 96, 96]               0
        ResUpBlock-91           [-1, 64, 96, 96]               0
            Conv2d-92          [-1, 256, 96, 96]          16,384
        PixelShuffle-93         [-1, 64, 192, 192]               0
            Conv2d-94         [-1, 64, 192, 192]           4,096
            Conv2d-95         [-1, 16, 192, 192]           1,024
        BatchNorm2d-96         [-1, 16, 192, 192]              32
        BatchNorm2d-97         [-1, 64, 192, 192]             128
                ReLU-98         [-1, 64, 192, 192]               0
            Conv2d-99         [-1, 16, 192, 192]           9,216
        BatchNorm2d-100         [-1, 16, 192, 192]              32
                ReLU-101         [-1, 16, 192, 192]               0
            Conv2d-102         [-1, 16, 192, 192]           2,304
    ResDoubleConv-103         [-1, 16, 192, 192]               0
        ResUpBlock-104         [-1, 16, 192, 192]               0
            Conv2d-105          [-1, 1, 192, 192]              16
            Conv2d-106         [-1, 2048, 12, 12]       1,048,576
        PixelShuffle-107          [-1, 512, 24, 24]               0
            Conv2d-108          [-1, 512, 24, 24]         262,144
            Conv2d-109          [-1, 512, 24, 24]         131,072
            Conv2d-110           [-1, 64, 24, 24]          32,768
        BatchNorm2d-111           [-1, 64, 24, 24]             128
        BatchNorm2d-112          [-1, 512, 24, 24]           1,024
                ReLU-113          [-1, 512, 24, 24]               0
            Conv2d-114           [-1, 64, 24, 24]         294,912
        BatchNorm2d-115           [-1, 64, 24, 24]             128
                ReLU-116           [-1, 64, 24, 24]               0
            Conv2d-117           [-1, 64, 24, 24]          36,864
    ResDoubleConv-118           [-1, 64, 24, 24]               0
        ResUpBlock-119           [-1, 64, 24, 24]               0
            Conv2d-120          [-1, 256, 24, 24]          16,384
        PixelShuffle-121           [-1, 64, 48, 48]               0
            Conv2d-122           [-1, 64, 48, 48]          16,384
            Conv2d-123           [-1, 64, 48, 48]           8,192
            Conv2d-124           [-1, 64, 48, 48]           4,096
        BatchNorm2d-125           [-1, 64, 48, 48]             128
        BatchNorm2d-126           [-1, 64, 48, 48]             128
                ReLU-127           [-1, 64, 48, 48]               0
            Conv2d-128           [-1, 64, 48, 48]          36,864
        BatchNorm2d-129           [-1, 64, 48, 48]             128
                ReLU-130           [-1, 64, 48, 48]               0
            Conv2d-131           [-1, 64, 48, 48]          36,864
    ResDoubleConv-132           [-1, 64, 48, 48]               0
        ResUpBlock-133           [-1, 64, 48, 48]               0
            Conv2d-134          [-1, 256, 48, 48]          16,384
        PixelShuffle-135           [-1, 64, 96, 96]               0
            Conv2d-136           [-1, 64, 96, 96]           8,192
            Conv2d-137           [-1, 64, 96, 96]           4,096
            Conv2d-138           [-1, 32, 96, 96]           2,048
        BatchNorm2d-139           [-1, 32, 96, 96]              64
        BatchNorm2d-140           [-1, 64, 96, 96]             128
                ReLU-141           [-1, 64, 96, 96]               0
            Conv2d-142           [-1, 32, 96, 96]          18,432
        BatchNorm2d-143           [-1, 32, 96, 96]              64
                ReLU-144           [-1, 32, 96, 96]               0
            Conv2d-145           [-1, 32, 96, 96]           9,216
    ResDoubleConv-146           [-1, 32, 96, 96]               0
        ResUpBlock-147           [-1, 32, 96, 96]               0
            Conv2d-148          [-1, 128, 96, 96]           4,096
        PixelShuffle-149         [-1, 32, 192, 192]               0
            Conv2d-150         [-1, 32, 192, 192]           2,048
            Conv2d-151         [-1, 32, 192, 192]             512
            Conv2d-152         [-1, 16, 192, 192]             512
        BatchNorm2d-153         [-1, 16, 192, 192]              32
        BatchNorm2d-154         [-1, 32, 192, 192]              64
                ReLU-155         [-1, 32, 192, 192]               0
            Conv2d-156         [-1, 16, 192, 192]           4,608
        BatchNorm2d-157         [-1, 16, 192, 192]              32
                ReLU-158         [-1, 16, 192, 192]               0
            Conv2d-159         [-1, 16, 192, 192]           2,304
    ResDoubleConv-160         [-1, 16, 192, 192]               0
        ResUpBlock-161         [-1, 16, 192, 192]               0
            Conv2d-162          [-1, 1, 192, 192]              16
    ================================================================
    Total params: 15,855,712
    Trainable params: 15,855,712
    Non-trainable params: 0
    ----------------------------------------------------------------
    Input size (MB): 0.84
    Forward/backward pass size (MB): 14099753.81
    Params size (MB): 60.48
    Estimated Total Size (MB): 14099815.14
    ----------------------------------------------------------------


Another model that we made but wasn't used, was a ResUNeXt model, which was inspired from ResNeXt and UNet
this model will be very useful for multiclass segmentation output, but for our needs it wasn't required since we
had atmost 100 different classes to segment, which should be pretty easy.

anyways this was the summary of ResUNeXt

Refer to :doc:`model` for more details

Total Params: 16,409,856

.. code-block:: none

    ----------------------------------------------------------------
            Layer (type)               Output Shape         Param #
    ================================================================
                Conv2d-1         [-1, 32, 192, 192]           4,800
                Conv2d-2         [-1, 64, 192, 192]           2,048
        BatchNorm2d-3         [-1, 64, 192, 192]             128
        BatchNorm2d-4         [-1, 32, 192, 192]              64
                ReLU-5         [-1, 32, 192, 192]               0
                Conv2d-6        [-1, 320, 192, 192]          10,240
        BatchNorm2d-7        [-1, 320, 192, 192]             640
                ReLU-8        [-1, 320, 192, 192]               0
                Conv2d-9        [-1, 320, 192, 192]          28,800
        BatchNorm2d-10        [-1, 320, 192, 192]             640
                ReLU-11        [-1, 320, 192, 192]               0
            Conv2d-12         [-1, 64, 192, 192]          20,480
        ResDoubleConv-13         [-1, 64, 192, 192]               0
            MaxPool2d-14           [-1, 64, 96, 96]               0
        ResDownBlock-15  [[-1, 64, 96, 96], [-1, 64, 192, 192]]               0
            Conv2d-16          [-1, 128, 96, 96]           8,192
        BatchNorm2d-17          [-1, 128, 96, 96]             256
        BatchNorm2d-18           [-1, 64, 96, 96]             128
                ReLU-19           [-1, 64, 96, 96]               0
            Conv2d-20          [-1, 672, 96, 96]          43,008
        BatchNorm2d-21          [-1, 672, 96, 96]           1,344
                ReLU-22          [-1, 672, 96, 96]               0
            Conv2d-23          [-1, 672, 96, 96]         127,008
        BatchNorm2d-24          [-1, 672, 96, 96]           1,344
                ReLU-25          [-1, 672, 96, 96]               0
            Conv2d-26          [-1, 128, 96, 96]          86,016
        ResDoubleConv-27          [-1, 128, 96, 96]               0
            MaxPool2d-28          [-1, 128, 48, 48]               0
        ResDownBlock-29  [[-1, 128, 48, 48], [-1, 128, 96, 96]]               0
            Conv2d-30          [-1, 256, 48, 48]          32,768
        BatchNorm2d-31          [-1, 256, 48, 48]             512
        BatchNorm2d-32          [-1, 128, 48, 48]             256
                ReLU-33          [-1, 128, 48, 48]               0
            Conv2d-34         [-1, 1344, 48, 48]         172,032
        BatchNorm2d-35         [-1, 1344, 48, 48]           2,688
                ReLU-36         [-1, 1344, 48, 48]               0
            Conv2d-37         [-1, 1344, 48, 48]         508,032
        BatchNorm2d-38         [-1, 1344, 48, 48]           2,688
                ReLU-39         [-1, 1344, 48, 48]               0
            Conv2d-40          [-1, 256, 48, 48]         344,064
        ResDoubleConv-41          [-1, 256, 48, 48]               0
            MaxPool2d-42          [-1, 256, 24, 24]               0
        ResDownBlock-43  [[-1, 256, 24, 24], [-1, 256, 48, 48]]               0
            Conv2d-44          [-1, 512, 24, 24]         131,072
        BatchNorm2d-45          [-1, 512, 24, 24]           1,024
        BatchNorm2d-46          [-1, 256, 24, 24]             512
                ReLU-47          [-1, 256, 24, 24]               0
            Conv2d-48         [-1, 2720, 24, 24]         696,320
        BatchNorm2d-49         [-1, 2720, 24, 24]           5,440
                ReLU-50         [-1, 2720, 24, 24]               0
            Conv2d-51         [-1, 2720, 24, 24]       2,080,800
        BatchNorm2d-52         [-1, 2720, 24, 24]           5,440
                ReLU-53         [-1, 2720, 24, 24]               0
            Conv2d-54          [-1, 512, 24, 24]       1,392,640
        ResDoubleConv-55          [-1, 512, 24, 24]               0
            MaxPool2d-56          [-1, 512, 12, 12]               0
        ResDownBlock-57  [[-1, 512, 12, 12], [-1, 512, 24, 24]]               0
        BatchNorm2d-58          [-1, 512, 12, 12]           1,024
                ReLU-59          [-1, 512, 12, 12]               0
            Conv2d-60         [-1, 2720, 12, 12]       1,392,640
        BatchNorm2d-61         [-1, 2720, 12, 12]           5,440
                ReLU-62         [-1, 2720, 12, 12]               0
            Conv2d-63         [-1, 2720, 12, 12]       2,080,800
        BatchNorm2d-64         [-1, 2720, 12, 12]           5,440
                ReLU-65         [-1, 2720, 12, 12]               0
            Conv2d-66          [-1, 512, 12, 12]       1,392,640
        ResDoubleConv-67          [-1, 512, 12, 12]               0
            Conv2d-68         [-1, 2048, 12, 12]       1,048,576
        PixelShuffle-69          [-1, 512, 24, 24]               0
            Conv2d-70          [-1, 512, 24, 24]         262,144
            Conv2d-71          [-1, 256, 24, 24]         131,072
        BatchNorm2d-72          [-1, 256, 24, 24]             512
        BatchNorm2d-73          [-1, 512, 24, 24]           1,024
                ReLU-74          [-1, 512, 24, 24]               0
            Conv2d-75         [-1, 1344, 24, 24]         688,128
        BatchNorm2d-76         [-1, 1344, 24, 24]           2,688
                ReLU-77         [-1, 1344, 24, 24]               0
            Conv2d-78         [-1, 1344, 24, 24]         508,032
        BatchNorm2d-79         [-1, 1344, 24, 24]           2,688
                ReLU-80         [-1, 1344, 24, 24]               0
            Conv2d-81          [-1, 256, 24, 24]         344,064
        ResDoubleConv-82          [-1, 256, 24, 24]               0
        ResUpBlock-83          [-1, 256, 24, 24]               0
            Conv2d-84         [-1, 1024, 24, 24]         262,144
        PixelShuffle-85          [-1, 256, 48, 48]               0
            Conv2d-86          [-1, 256, 48, 48]          65,536
            Conv2d-87          [-1, 128, 48, 48]          32,768
        BatchNorm2d-88          [-1, 128, 48, 48]             256
        BatchNorm2d-89          [-1, 256, 48, 48]             512
                ReLU-90          [-1, 256, 48, 48]               0
            Conv2d-91          [-1, 672, 48, 48]         172,032
        BatchNorm2d-92          [-1, 672, 48, 48]           1,344
                ReLU-93          [-1, 672, 48, 48]               0
            Conv2d-94          [-1, 672, 48, 48]         127,008
        BatchNorm2d-95          [-1, 672, 48, 48]           1,344
                ReLU-96          [-1, 672, 48, 48]               0
            Conv2d-97          [-1, 128, 48, 48]          86,016
        ResDoubleConv-98          [-1, 128, 48, 48]               0
        ResUpBlock-99          [-1, 128, 48, 48]               0
            Conv2d-100          [-1, 512, 48, 48]          65,536
        PixelShuffle-101          [-1, 128, 96, 96]               0
            Conv2d-102          [-1, 128, 96, 96]          16,384
            Conv2d-103           [-1, 64, 96, 96]           8,192
        BatchNorm2d-104           [-1, 64, 96, 96]             128
        BatchNorm2d-105          [-1, 128, 96, 96]             256
                ReLU-106          [-1, 128, 96, 96]               0
            Conv2d-107          [-1, 320, 96, 96]          40,960
        BatchNorm2d-108          [-1, 320, 96, 96]             640
                ReLU-109          [-1, 320, 96, 96]               0
            Conv2d-110          [-1, 320, 96, 96]          28,800
        BatchNorm2d-111          [-1, 320, 96, 96]             640
                ReLU-112          [-1, 320, 96, 96]               0
            Conv2d-113           [-1, 64, 96, 96]          20,480
    ResDoubleConv-114           [-1, 64, 96, 96]               0
        ResUpBlock-115           [-1, 64, 96, 96]               0
            Conv2d-116          [-1, 256, 96, 96]          16,384
        PixelShuffle-117         [-1, 64, 192, 192]               0
            Conv2d-118         [-1, 64, 192, 192]           4,096
            Conv2d-119         [-1, 16, 192, 192]           1,024
        BatchNorm2d-120         [-1, 16, 192, 192]              32
        BatchNorm2d-121         [-1, 64, 192, 192]             128
                ReLU-122         [-1, 64, 192, 192]               0
            Conv2d-123         [-1, 64, 192, 192]           4,096
        BatchNorm2d-124         [-1, 64, 192, 192]             128
                ReLU-125         [-1, 64, 192, 192]               0
            Conv2d-126         [-1, 64, 192, 192]           1,152
        BatchNorm2d-127         [-1, 64, 192, 192]             128
                ReLU-128         [-1, 64, 192, 192]               0
            Conv2d-129         [-1, 16, 192, 192]           1,024
    ResDoubleConv-130         [-1, 16, 192, 192]               0
        ResUpBlock-131         [-1, 16, 192, 192]               0
            Conv2d-132          [-1, 1, 192, 192]              16
            Conv2d-133         [-1, 2048, 12, 12]       1,048,576
        PixelShuffle-134          [-1, 512, 24, 24]               0
            Conv2d-135          [-1, 512, 24, 24]         262,144
            Conv2d-136          [-1, 512, 24, 24]         131,072
            Conv2d-137           [-1, 64, 24, 24]          32,768
        BatchNorm2d-138           [-1, 64, 24, 24]             128
        BatchNorm2d-139          [-1, 512, 24, 24]           1,024
                ReLU-140          [-1, 512, 24, 24]               0
            Conv2d-141          [-1, 320, 24, 24]         163,840
        BatchNorm2d-142          [-1, 320, 24, 24]             640
                ReLU-143          [-1, 320, 24, 24]               0
            Conv2d-144          [-1, 320, 24, 24]          28,800
        BatchNorm2d-145          [-1, 320, 24, 24]             640
                ReLU-146          [-1, 320, 24, 24]               0
            Conv2d-147           [-1, 64, 24, 24]          20,480
    ResDoubleConv-148           [-1, 64, 24, 24]               0
        ResUpBlock-149           [-1, 64, 24, 24]               0
            Conv2d-150          [-1, 256, 24, 24]          16,384
        PixelShuffle-151           [-1, 64, 48, 48]               0
            Conv2d-152           [-1, 64, 48, 48]          16,384
            Conv2d-153           [-1, 64, 48, 48]           8,192
            Conv2d-154           [-1, 64, 48, 48]           4,096
        BatchNorm2d-155           [-1, 64, 48, 48]             128
        BatchNorm2d-156           [-1, 64, 48, 48]             128
                ReLU-157           [-1, 64, 48, 48]               0
            Conv2d-158          [-1, 320, 48, 48]          20,480
        BatchNorm2d-159          [-1, 320, 48, 48]             640
                ReLU-160          [-1, 320, 48, 48]               0
            Conv2d-161          [-1, 320, 48, 48]          28,800
        BatchNorm2d-162          [-1, 320, 48, 48]             640
                ReLU-163          [-1, 320, 48, 48]               0
            Conv2d-164           [-1, 64, 48, 48]          20,480
    ResDoubleConv-165           [-1, 64, 48, 48]               0
        ResUpBlock-166           [-1, 64, 48, 48]               0
            Conv2d-167          [-1, 256, 48, 48]          16,384
        PixelShuffle-168           [-1, 64, 96, 96]               0
            Conv2d-169           [-1, 64, 96, 96]           8,192
            Conv2d-170           [-1, 64, 96, 96]           4,096
            Conv2d-171           [-1, 32, 96, 96]           2,048
        BatchNorm2d-172           [-1, 32, 96, 96]              64
        BatchNorm2d-173           [-1, 64, 96, 96]             128
                ReLU-174           [-1, 64, 96, 96]               0
            Conv2d-175          [-1, 160, 96, 96]          10,240
        BatchNorm2d-176          [-1, 160, 96, 96]             320
                ReLU-177          [-1, 160, 96, 96]               0
            Conv2d-178          [-1, 160, 96, 96]           7,200
        BatchNorm2d-179          [-1, 160, 96, 96]             320
                ReLU-180          [-1, 160, 96, 96]               0
            Conv2d-181           [-1, 32, 96, 96]           5,120
    ResDoubleConv-182           [-1, 32, 96, 96]               0
        ResUpBlock-183           [-1, 32, 96, 96]               0
            Conv2d-184          [-1, 128, 96, 96]           4,096
        PixelShuffle-185         [-1, 32, 192, 192]               0
            Conv2d-186         [-1, 32, 192, 192]           2,048
            Conv2d-187         [-1, 32, 192, 192]             512
            Conv2d-188         [-1, 16, 192, 192]             512
        BatchNorm2d-189         [-1, 16, 192, 192]              32
        BatchNorm2d-190         [-1, 32, 192, 192]              64
                ReLU-191         [-1, 32, 192, 192]               0
            Conv2d-192         [-1, 64, 192, 192]           2,048
        BatchNorm2d-193         [-1, 64, 192, 192]             128
                ReLU-194         [-1, 64, 192, 192]               0
            Conv2d-195         [-1, 64, 192, 192]           1,152
        BatchNorm2d-196         [-1, 64, 192, 192]             128
                ReLU-197         [-1, 64, 192, 192]               0
            Conv2d-198         [-1, 16, 192, 192]           1,024
    ResDoubleConv-199         [-1, 16, 192, 192]               0
        ResUpBlock-200         [-1, 16, 192, 192]               0
            Conv2d-201          [-1, 1, 192, 192]              16
    ================================================================
    Total params: 16,409,856
    Trainable params: 16,409,856
    Non-trainable params: 0
    ----------------------------------------------------------------
    Input size (MB): 0.84
    Forward/backward pass size (MB): 14098296.45
    Params size (MB): 62.60
    Estimated Total Size (MB): 14098359.89
    ----------------------------------------------------------------
