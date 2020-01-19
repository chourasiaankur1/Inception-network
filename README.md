# Inception-network

The network architecture in this paper is quite different from VGGNet, ZFNet, and AlexNet. It contains 1×1 Convolution at the middle of the network. And global average pooling is used at the end of the network instead of using fully connected layers. These two techniques are from another paper “Network In Network” (NIN) . Another technique, called inception module, is to have different sizes/types of convolutions for the same input and stacking all the outputs.

1. The 1×1 Convolution
The 1×1 convolution is introduced by NIN . 1×1 convolution is used with ReLU. Thus, originally, NIN uses it for introducing more non-linearity to increase the representational power of the network since authors in NIN believe data is in non-linearity form. In GoogLeNet, 1×1 convolution is used as a dimension reduction module to reduce the computation. By reducing the computation bottleneck, depth and width can be increased.
I pick a simple example to illustrate this. Suppose we need to perform 5×5 convolution without the use of 1×1 convolution as below:

Without the Use of 1×1 Convolution
Number of operations = (14×14×48)×(5×5×480) = 112.9M
With the use of 1×1 convolution:

With the Use of 1×1 Convolution
Number of operations for 1×1 = (14×14×16)×(1×1×480) = 1.5M
Number of operations for 5×5 = (14×14×48)×(5×5×16) = 3.8M
Total number of operations = 1.5M + 3.8M = 5.3M
which is much much smaller than 112.9M !!!!!!!!!!!!!!!
Indeed, the above example is the calculation of 5×5 conv at inception (4a).
(We may think that, when dimension is reduced, actually we are working on the mapping from high dimension to low dimension in a non-linearity way. In contrast, for PCA, it performs linear dimension reduction.)
Thus, inception module can be built without increasing the number of operations largely compared the one without 1×1 convolution!
1×1 convolution can help to reduce model size which can also somehow help to reduce the overfitting problem!!
2. Inception Module
The inception module (naive version, without 1×1 convolution) is as below:

Inception Module (Without 1×1 Convolution)
Previously, such as AlexNet, and VGGNet, conv size is fixed for each layer.
Now, 1×1 conv, 3×3 conv, 5×5 conv, and 3×3 max pooling are done altogether for the previous input, and stack together again at output. When image’s coming in, different sizes of convolutions as well as max pooling are tried. Then different kinds of features are extracted.
After that, all feature maps at different paths are concatenated together as the input of the next module.
However, without the 1×1 convolution as above, we can imagine how large the number of operation is!

Inception Module (With 1×1 Convolution)
Thus, 1×1 convolution is inserted into the inception module for dimension reduction!
3. Global Average Pooling

Fully Connected Layer VS Global Average Pooling
Previously, fully connected (FC) layers are used at the end of network, such as in AlexNet. All inputs are connected to each output.
Number of weights (connections) above = 7×7×1024×1024 = 51.3M
In GoogLeNet, global average pooling is used nearly at the end of network by averaging each feature map from 7×7 to 1×1, as in the figure above.
Number of weights = 0
And authors found that a move from FC layers to average pooling improved the top-1 accuracy by about 0.6%.
This is the idea from NIN [6] which can be less prone to overfitting.
4. Overall Architecture
After knowing the basic units as described above, we can talk about the overall network architecture.

GoogLeNet Network (From Left to Right)
There are 22 layers in total!
It is already a very deep model compared with previous AlexNet, ZFNet and VGGNet. (But not so deep compared with ResNet invented afterwards.) And we can see that there are numerous inception modules connected together to go deeper. (There are some intermediate softmax branches at the middle, we will describe about them in the next section.)
Below is the details about the parameters if each layer. We actually can extend the example of 1×1 convolution to calculate the number of operations by ourselves. :)

Details about Parameters of Each Layer in GoogLeNet Network (From Top to Bottom)
5. Auxiliary Classifiers for Training
As we can see there are some intermediate softmax branches at the middle, they are used for training only. These branches are auxiliary classifiers which consist of:
5×5 Average Pooling (Stride 3)
1×1 Conv (128 filters)
1024 FC
1000 FC
Softmax
The loss is added to the total loss, with weight 0.3.
Authors claim it can be used for combating gradient vanishing problem, also providing regularization.
And it is NOT used in testing or inference time.
6. Testing Details
7 GoogLeNet are used for ensemble prediction. This is already a kind of boosting approach from LeNet, AlexNet, ZFNet and VGGNet.
Multi-scale testing is used just like VGGNet, with shorter dimension of 256, 288, 320, 352. (4 scales)
Multi-crop testing is used, same idea but a bit different from and more complicated than AlexNet.
First, for each scale, it takes left, center and right, or top, middle and bottom squares (3 squares). Then, for each square, 4 corners and center as well as the resized square (6 crops) are cropped as well as their corresponding flips (2 versions) are generated.
The total is 4 scales×3 squares×6 crops×2 versions=144 crops/image
Softmax probabilities are averaged over all crops.

Ablation Study
With 7 models + 144 crops, the top-5 error is 6.67%.
Compared with 1 model + 1 crop, there are large reduction from 10.07%.
From this, we can observe that, besides the network design, the other stuffs like ensemble methods, multi-scale and multi-crop approaches are also essential to reduce the error rate!!!
And these techniques actually are not totally new in this paper!

Comparison with State-of-the-art Approaches
Finally, GoogLeNet outperforms other previous deep learning networks, and won in ILSVRC 2014.
