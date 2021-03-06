﻿# Model Quantization
  * [Quantization Arithmetic](./quantization_arithmetic.md)     
  * [Quantization Techniques and Approaches](#quantization-techniques-and-approaches)
     * [Post-training quantization](#post-training-quantization)
       * [Hybrid operations](#hybrid-approaches)
       * [INT8 quantization](#int8-quantization)
     * [Quantization aware training](#quantization-aware-training)
       * [MNIST exmaple](./QAT/qat_mnist.md)
  * [Tensorflow Quantization](./tensorflow_quantization.md)     
  
Quantization for deep learning is the process of approximating a neural network that uses floating-point numbers, which by default are 32-bit, by a neural network of low bit width numbers. This results in a smaller model size and faster computation.

  
## INT8 Quantization  
* Some frameworks simply introduce Quantize and Dequantize layer which converts FP32 to INT8 and the reverse, when feeding to and fetching from Convolution/Fully Connected layer. In this case, the model itself and input/output are in FP32 format. Deep learning framework loads the model, rewrites network to insert Quantize and Dequantize layer, and converts weights to INT8 format.

 * Some other frameworks convert the network into INT8 format as a whole, online or offline. Thus, there is no format translation during inference. This method needs to support quantization per operator, for the data flowing between operators is INT8. For the not-yet-supported ones, it may fallback to Quantize/Dequantize scheme. 

![](./figs/mixed-fp32int8-pure-int8.svg)
*Figure: Mixed FP32/INT8 and Pure INT8 Inference. Red color is FP32, green color is INT8 or quantization*

## Quantization Techniques and Approaches

Post-training quantization via “hybrid operations”, which is quantizing the parameters of the model (i.e. weights), but allowing certain parts of the computation to take place in floating point. 

Post-training integer quantization. Integer quantization is a general technique that reduces the numerical precision of the weights and activations of models to reduce memory and improve latency.

### The Accuracy Problem
The method described in [Quantizing Floating-point](#quantizing-floating-point) section is pretty straightforward. In the early development of a framework (or engine or whatever else you call it), that trivial approach is applied to make INT8 able to run. However, the predication accuracy of such INT8 quantized network usually drops significantly.

What happened? Though the value range of FP32 weight is narrow, the value points are huge. Taking the scaling example, 2<sup>31</sup> around (yes, basically half of the representables) FP32 values in [−1,1] are mapped into 256 INT8 values. Now consider two important rules discussed in Quantization Arithmetic section:

* The value density improves as floating-point values approach zero. The nearer a value is to zero, the more accurate it can be.
* The uniform quantization approach maps dynamic value density of floating-point to fixed-point of which the value density is constant.

So in the naive quantization approach, the floating-point values that near zero are less accuratly represented in fixed-point than the ones are not when quantizing. Consequently, the predicate result of quantized network is far less accurate when compared with the original network. This problem is inevitable for uniform quantization.

Equation 4 shows that the value mapping precision is singificantly impacted by x<sub>scale</sub> which is derived from x<sup>min</sup><sub>float</sub> and x<sup>max</sup><sub>float</sub>. And, weight value distribution shows that the number of value points near x<sup>min</sup><sub>float</sub> and x<sup>max</sup><sub>float</sub> are often ignorable. So, maybe the min and max of floating-point value can be tweaked?

![](./figs/min-max-tweaking.jpg)
*Figure: Tweaking min/max when quantizing floating-point to fixed-point*

Tweaking min/max means chosing a value range such that values in the range are more accurately quantized while values out the range are not (mapped to min/max of the fixed-point). For example, when chosing x<sup>min</sup><sub>float</sub>=−0.9 and x<sup>max</sup><sub>float</sub>=0.8 from original value range [−1,1], values in [−0.9,0.8] are more accurately mapped into [0,255], while values in [−1,−0.9] and [0.8,1] are mapped to 0 and 255 respectively.

### Tweaking Approaches
The tweaking is yet another machine learning process which learns hyper parameter (min/max) of the quantization network with a target of good predicate accuracy. Different tweaking approaches have been proposed and can be categoried into Calibration (post-training quantization) and Quantization-aware Training according to when the tweaking happens.

TensorRT, MXNet and some other frameworks that are likely to be deployed in inference enviroment are equipped with calibration. Top half of the below Figure is the process of calibration which works with pre-trained network regardless of how it is trained. Calibration often combines the min/max searching and quantization into one step. After calibration, the network is quantized and can be deployed.

![Calibration and quantization-aware training process](./figs/calibration-and-quantization-aware-training.jpg)
*Figure: Calibration and quantization-aware training process*

As calibration choses a training independent approach, TensorFlow inovates quantization-aware training which includes four steps:

  1. Training models in floating-point with TensorFlow as usual.
  
  2. Training models with tf.contrib.quantize which rewrites network to insert Fake-Quant nodes and train min/max.
  
  3. Quantizing the network by TensorFlow Lite tools which reads the trained min/max of step 2.
  
  4. Deploying the quantized network with TensorFlow Lite.
  
Step 2 is the so-called quantization-aware training of which the forwarding is simulated INT8 and backwarding is FP32. Figure 12 illustrates the idea. Figure 12 left half is the quantized network which receives INT8 inputs and weights and generates INT8 output. Right half of Figure 12 is the rewrited network, where Fake-Quant nodes (in pink) quantize FP32 tensors into INT8 (FP32 actually, the original FP32 was Quantize and Dequantize to simulate the quantization arithmetic) on-the-fly during training. The network forwarding of Step 2 above simulates the INT8 inference arithmetic.

![](./figs/rewrite-network.jpg)
*Network node example of quantization-aware training*

## Summary

You may ask why quantization works (having a good enough predication accuracy) with regard to the information losing when converting FP32 to INT8? Well, there is no solid theory yet, but the intuition is that neural networks are over parameterized such that there is enough redundant information which can be safely reduced without significant accuracy drop. One evidence is that, for given quantization scheme, the accuracy gap between FP32 network and INT8 network is small for large networks, since the large networks are more over parameterized.

* Overview of schemes for model quantization: One can quantize weights post training (left) or quantize weights and activations post training (middle). It is also possible to perform quantization aware training for improved accuracy.   
    
   <img src="./figs/tensorflow_overview_of_schemes_for_model_quantization.png" width="600px" title="Overview of schemes for model quantization">
        
* Weight only quantization: per-channel quantization provides good accuracy, with asymmetric quantization providing close to floating point accuracy.
    
    <p align="center">
       <img src="./figs/weight_only_quantization.png" width="600px" title="weight only quantization">
    </p>
    
    *  Post training quantization of weights and activations: per-channel quantization of weights and per-layer quantization of activations works well for all the networks considered, with asymmetric quantization providing slightly better accuracies.
    
    <p align="center">
       <img src="./figs/post_training_quantization_of_weights_and_activation.png" width="600px" title="post training quantization of weights and activations">
    </p>
    
  * Continuous-discrete learning
    * During training there are effectively two networks : float-precision and binary-precision. The binary-precision is updated in the forward pass using the float-precision, and the float-precision is updated in the backward pass using the binary-precision. In this sense, the training is a type of alternating optimization.
    
    <p align="center">
       <img src="./figs/quantization_of_parameteres during training.png" width="400px" title="Quantization of parameters during training">
    </p>
    
     * Ternary parameter networks are an alternative to binary parameters that allow for higher accuracy, at cost of larger model size.
    <p align="center">
       <img src="./figs/ternary_quantization_for_gaussain_distributed_parameters.png" width="600px" title="Ternary quantization for Gaussian-distributed parameters">
    </p>
    
  * Quantized activations
 
    <p align="center">
       <img src="./figs/quantization_of_activation.png" width="400px" title="Quantization of activations, allowing binary calculations, with integer accumulations. The binary calculation can be convolutional layer or fully-connected layer">
    </p>
    
  * Multiple bit width activations
 
    <p align="center">
       <img src="./figs/multiple_bit_width_activations.png" width="400px" title="DoReFa-Net style 3-bit activation quantizer function">
    </p>
   
## References
* [Quantizing deep convolutional networks for efficient inference: A white paper](https://arxiv.org/pdf/1806.08342.pdf) by Google.
* [Quantization and Training of Neural Networks for Efficient Integer-Arithmetic-Only Inference](https://arxiv.org/pdf/1712.05877.pdf) by Benoit Jacob, Skirmantas Kligys, Bo Chen, Menglong Zhu, Matthew Tang, Andrew Howard, Hartwig Adam, and Dmitry Kalenichenko from Google.
* [Quantization Algorithms](https://nervanasystems.github.io/distiller/algo_quantization.html) Neural Network Distiller
* [Mixed-Precision Training of Deep Neural Networks](https://devblogs.nvidia.com/mixed-precision-training-deep-neural-networks/) by Nvidia
* [8-bit Inference with TensorRT](http://on-demand.gputechconf.com/gtc/2017/presentation/s7310-8-bit-inference-with-tensorrt.pdf) by Szymon Migacz from Nvidia
* [Fast INT8 Inference for Autonomous Vehicles with TensorRT 3](https://devblogs.nvidia.com/int8-inference-autonomous-vehicles-tensorrt/) by Joohoon Lee from Nvidia
* [Making Neural Nets Work with Low Precision](https://sahnimanas.github.io/post/quantization-in-tflite/) 
* [What I've learned about neural network quantization](https://petewarden.com/2017/06/22/what-ive-learned-about-neural-network-quantization/) by Pete Warden.
