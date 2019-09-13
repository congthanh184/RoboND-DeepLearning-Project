## Project: Follow Me

---

[//]: # (Image References)

[image_0]: ./misc/sim_screenshot.png
[overview]: ./misc/overview.png
[architecture]: ./misc/architecture.jpg
[train_curves]: ./misc/train_curves.png
[hero_result]: ./misc/hero_result.png
[passenger_result1]: ./misc/passenger_result1.png
[passenger_result2]: ./misc/passenger_result2.png
[20_90_15]: ./misc/20_90_15.png
[15_80_15]: ./misc/15_80_15.png

## [Rubric](https://review.udacity.com/#!/rubrics/1155/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

This project expores the ability of deep neural network in classifying and segmenting objects. In particular, a fully convolutional neural network has been trained and implemented to allow a drone to track and follow a `hero`. The simulating environment is taken place in a large area with buildings, trees, and another passengers.

![alt text][image_0] 

##### 1. Architecture

The segmentation system utilizes the fully convolutional network to assign label (background, person, hero) to each pixel of images captured from a camera installed in a drone. An overview of this network can be demonstrated in the picture below 

![alt text][architecture]

* ##### Fully Convolutional Network

A fully convolutional network combines two main blocks which are the Encoders and Decoders. Each encoder layer is a separable convolution which reduces the size of images and enriches the depth of the output. For the encoder block, transposed covolution layers are used to upsample the output. At the end of the network, a classifing layer is used to label the pixel.

* ##### Separable convolutions with 1x1 convolution

A separable convolution comprises a normal convolution following a 1x1 convolution. It reduces the number of parameters which not only improves calculation performance but also reduces the overfitting.

* ##### Batch Normaliation

Each separable convolution layer comes with a Batch Normalization. This additional way attempts to normalize the inputs to layers in a network which results in a higher learning rates and introducing a bit of regularization by adding a litte noise to the network. Input normalization has a similar effect as a Dropout or Skip Connection.

* ##### Fully connected layer vs 1x1 convolution

A fully connected layer is the final step to map all the result coming from convolution layers and reasoning it which is essential to classify the output. A 1x1 convolution, in addition, is a technique to manipulate the dimentionality with fewer parameters and, therefore, faster computation and reduce overfitting. A 1x1 convolution is mathematically equivalent to a fully connected layer, and therefore, can substitute fully connected layers in the network. Finally, an 1x1 convolution introduces new parameters and new non-linearity into the network so it can also improve the accuracy. 

* ##### Transposed Convolutions

Transposed convolution is a way of upsampling layers to higher dimentions or resolutions. In this project, Bilinear Upsampling or Bilinear Interpolation is implemented.

* ##### Skip Connection

Each decoder layer comprises a transposed convolution and a skip connection. In this project, Layer Concatenation technique is used to concatenate the upsampled layer and a layer with the more spatial information layer to retain the finer details.

#### 2. Encoder & Decoder Block

* ##### Encoder Block

The encoder block consists two separable convolution layers. Each separable convolution will reduces the size of the input images by half while enrich the the depth of the output. The process starts with a convolution filter with the same depth as the input, then followed by a 1x1 convolution. Intuitively, Encoder block downsamples input images (convolution filter) and looks for interesting features (1x1 convolution and `ReLU`).

* ##### Decoder Block

The decoder block comprises two transposed convolution layers. They will upsample the result from encoder block which will restore it to its original size. To enchance the finer details that might be lost during the process, skip connection technique is applied to concatenate the original source with the upsampled output.

#### 3. Hyper parameters

`Epoch` needs to be enough in order to let the accuracy to converged.
`Learning Rate` a smaller learning rate (from `0.01` to `0.005`) shows a higher accuracy with an unsignificantly slower convergence.
`Batch size` a sufficient batch size is crucial. Depend on the architure of the network, I found increasing batch size ( from `50` to `70`) overally improve the accuracy, however, larger size than that will not yield prominent diffrence and may exhaust the machine.

| Epoch | Learning Rate | Batch size | Samples | Score | Comment |
|-------|---------------|------------|---------|-------|---------|
| 10    | 0.02          |  75        | ~3000   | 0.26  | Initial run to test the network        |
| 15    | 0.02          |  75        | ~14000  | 0.44  | Decoder layers (128 and 64)        |
| 20    | 0.015         |  90        | ~14000  | 0.40  | Decoder layers (128 and 64)        |
| 15    | 0.015         |  80        | ~14000  | 0.44  | Decoder layers (128 and 64)        |
| 14    | 0.005         |  55        | ~14000  | 0.44  | Decoder layers (256 and 128)        |

A high `Learning Rate` and smaller `Batch size` can increase the convergence rate ( iteration `3` and `4`). The training curves are represented in the pictures below. The left image is iteration 3, and iteration 4 is on the right.

<img src="./misc/20_90_15.png" width="400"/> <img src="./misc/15_80_15.png" width="405"/>

Increase the complex of the network's architecture can also increase the convergence rate, however it also takes more time to train the system. For example, the system uses decoder layers with depth's config 256 and 128 respectively takes up to ~500s for each `epoch` while the smaller system takes ~300s.

#### 4. Results

The training curves is presented below. According to the graph, it is clear that both train loss and validation loss improve together which also indicated that the model is not overfitting.

![alt text][train_curves]

Comparing the prediction with ground truth labels and original images, it confirms that the current model can classify the hero well when she is close. However, the numbel of misclassified pixels increases when the size of hero reduces which can be origininated as a result of network's architecture.  

![alt text][hero_result]

In addition, model accuracy reduces significantly when passengers wear clothes similar to the background.

![alt text][passenger_result1]
![alt text][passenger_result2]

#### 5. Future Enhancements

More encoder/decoder layers can be added to increase tracking accuracy when the hero is far from the drone. 

This model needs to be re-trained with another objects such as dog, cat, car, ... (increase the number of classes) in order to let it work well in tracking different type of objects. This process will also require adding more samples.


