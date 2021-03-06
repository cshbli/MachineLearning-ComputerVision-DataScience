﻿# Pix2Pix: Image-to-Image Translation with Conditional Adversarial Netwoks

[Pix2Pix](https://arxiv.org/pdf/1611.07004.pdf) network is basically a Conditional GANs (cGAN) that learn the mapping from an input image to output an image. 

Image-To-Image Translation is a process for translating one representation of an image into another representation.

[A jupyter notebook based on Keras](./pix2pix-network-Image-to-image-translation-using-conditional-gans.ipynb)

[Another implementation with jupyter notebook based on Keras running on maps dataset](./pix2pix-maps.ipynb)

* The Generator Network

  Generator network uses a <b>U-Net</b>-based architecture. U-Net’s architecture is similar to an <b>Auto-Encoder</b> network except for one difference. Both U-Net and Auto-Encoder network has two networks The <b>Encoder</b> and the <b>Decoder</b>.
  
  * U-Net Architecture Diagram
    <p align="center">
      <img src="unet_architecture_diagram.png" width="400px" title="U-Net Architecture">
    </p>
    
    * U-Net’s network has skip connections between Encoder layers and Decoder layers.
    
    * As shown in the picture the output of the first layer of Encoder is directly passed to the last layer of the Decoder and output of the second layer of Encoder is pass to the second last layer of the Decoder and so on.
    
    * Let’s consider if there are total N layers in U-Net’s(including middle layer), Then there will be a skip connection from the kth layer in the Encoder network to the (N-k+1)th layer in the Decoder network. where 1 ≤ k ≤ N/2.
   
   * Auto-Encoder Architecture Diagram
      <p align="center">
        <img src="auto_encoder_architecture.png" width="400px" title="Auto-Encoder Architecture">
      <p>
  
     * As shown in the picture Auto-Encoder doesn’t have skip connections between Encoder layers and Decoder layers.
     
   * The Generator Architecture
   
      The Generator network is made up of these two networks.
      
        * The Encoder network is a downsampler.
           * The Encoder network of the Generator network has seven convolutional blocks.
           * Each convolutional block has a convolutional layer, followed a LeakyRelu activation function.
           * Each convolutional block also has a batch normalization layer except the first convolutional layer.
           
        * The Decoder network is an upsampler.
          * The Decoder network of the Generator network has seven upsampling convolutional blocks.
          * Each upsampling convolutional block has an upsampling layer, followed by a convolutional layer, a batch normalization layer and a ReLU activation function.
        <p align="center">
          <img src="generator_architecture.png" width="800px" title="Generator Architecture">
        </p>      
        
* Discriminator Architecture

  Discriminator network uses of PatchGAN architecture. The PatchGAN network contains five convolutional blocks.
  
  The PatchGAN discriminator used in pix2pix is another unique component to this design. The PatchGAN / Markovian discriminator works by classifying individual (N x N) patches in the image as “real vs. fake”, opposed to classifying the entire image as “real vs. fake”. The authors reason that this enforces more constraints that encourage sharp high-frequency detail. Additionally, the PatchGAN has fewer parameters and runs faster than classifying the entire image. The image below depicts results experimenting with the size of N for the N x N patches to be classified:
  <p align="center">
    <img src="patch_gan_example.png" width="600px" title="PatchGan example">
  </p>
  <p align="center">
    <img src="discriminator_architecture.png" width="400px" title="Discriminator Architecture">
  </p>
      
* Pix2Pix GAN Architecture
  <p align="center">
    <img src="pix2pix_gan_architecture.png" width="200px" title="Pix2Pix GAN Architecture">
  </p>
  
* Pix2Pix Network Training

  A naive way to do Image-to-Image translation would be to discard the adversarial framework altogether. A source image would just be passed through a parametric function and the difference in the resulting image and the ground truth output would be used to update the weights of the network. <b>However, designing this loss function with standard distance measures such as L1 and L2 will fail to capture many of the important distinctive characteristics between these images</b>. However, the authors do find some value to the L1 loss function as a weighted sidekick to the adversarial loss function.

  Pix2Pix is a conditional GANs. The loss function for the conditional GANs can be written as below.
  <p align="center">
    <img src="conditional_gan_loss_function.png" width="600px" title="Loss function for the conditional GANs">
  </p>
  
   We have to minimize the loss between the reconstructed image and the original image. To make the images less blurry we can either use L1 or L2 regularization.
     * L1 regularization is the sum of the absolute error for each data point.
     * L2 regularization is the sum of the squared loss for each data point.
     * The L1 regularization loss function can be shown for a single image as bellow.     
    <p align="center">
      <img src="l1_regularization_loss_function.png" width="400px" title="L1 regularization loss function">
    </p>

   Where y is the original image and G(x, z) is the image generated by the Generator network. The L1 loss is calculated by the sum of all the absolute differences between all pixel values of the original image and all pixel values of the generated image.
   
   The final loss function for Pix2Pix is as given below.
   <p align="center">
      <img src="total_loss_function_of_pix2pix.png" width="600px" title="Total loss function of Pix2Pix">
   </p>
   
   In the experiments, the authors report that they found the most success with the <b>lambda parameter equal to 100</b>.
