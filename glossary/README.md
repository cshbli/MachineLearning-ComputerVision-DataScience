﻿# Machine Learning Glossary
 * [bag of words](./README.md#bag-of-words)
 * [clustering](./README.md#clustering)
 * [confusion matrix](./README.md#confusion-matrix)
 * [convex function](./README.md#convex-function)
 * [convex set](./README.md#convex-set)
 * [convolutional layer](./README.md#convolutional-layer)
 * [convolutional operation](./README.md#convolutional-operation)
 * [decision tree](./README.md#decision-tree)
 * [hashing](./README.md#hashing)
 * [intersection over union (IoU)](./README.md#intersection-over-union-iou)
 
## bag of words
A representation of the words in a phrase or passage, irrespective of order. For example, bag of words represents the following three phrases identically:

  * the dog jumps
  * jumps the dog
  * dog jumps the
  
Each word is mapped to an index in a sparse vector, where the vector has an index for every word in the vocabulary. For example, the phrase the dog jumps is mapped into a feature vector with non-zero values at the three indices corresponding to the words the, dog, and jumps. The non-zero value can be any of the following:

  * A 1 to indicate the presence of a word.
  * A count of the number of times a word appears in the bag. For example, if the phrase were <i>the maroon dog is a dog with maroon fur</i>, then both maroon and dog would be represented as 2, while the other words would be represented as 1.
  * Some other value, such as the logarithm of the count of the number of times a word appears in the bag. 
 
## clustering
Grouping related examples, particularly during unsupervised learning. Once all the examples are grouped, a human can optionally supply meaning to each cluster.

Many clustering algorithms exist. For example, the k-means algorithm clusters examples based on their proximity to a centroid, as in the following diagram:

<img src="kmeans_example_1.svg" width="400px">

A human researcher could then review the clusters and, for example, label cluster 1 as "dwarf trees" and cluster 2 as "full-size trees."

As another example, consider a clustering algorithm based on an example's distance from a center point, illustrated as follows:

<img src="RingCluster_example.svg" width="400px">

## confusion matrix
An NxN table that summarizes how successful a classification model's predictions were; that is, the correlation between the label and the model's classification. One axis of a confusion matrix is the label that the model predicted, and the other axis is the actual label. N represents the number of classes. In a binary classification problem, N=2. For example, here is a sample confusion matrix for a binary classification problem:

|  | Tumor(predicted) | Non-Tumor(predicted|
| --- | ---| --- | 
| Tumor (actual) |	18 |	1 |
|Non-Tumor (actual)	| 6	| 452 |

The preceding confusion matrix shows that of the 19 samples that actually had tumors, the model correctly classified 18 as having tumors (18 <b>true positives</b>), and incorrectly classified 1 as not having a tumor (1 <b>false negative</b>). Similarly, of 458 samples that actually did not have tumors, 452 were correctly classified (452 <b>true negatives</b>) and 6 were incorrectly classified (6 <b>false positives</b>).

The confusion matrix for a multi-class classification problem can help you determine mistake patterns. For example, a confusion matrix could reveal that a model trained to recognize handwritten digits tends to mistakenly predict 9 instead of 4, or 1 instead of 7.

Confusion matrices contain sufficient information to calculate a variety of performance metrics, including <b>precision</b> and <b>recall</b>.

## convex function
A function in which the region above the graph of the function is a convex set. The prototypical convex function is shaped something like the letter U. For example, the following are all convex functions:

<img src="convex_functions.png" title="Convex Functions">

A typical convex function is shaped like the letter 'U'.

By contrast, the following function is not convex. Notice how the region above the graph is not a convex set:

<img src="nonconvex_function.svg" title="Non-Convex Function">

A strictly convex function has exactly one local minimum point, which is also the global minimum point. The classic U-shaped functions are strictly convex functions. However, some convex functions (for example, straight lines) are not U-shaped.

A lot of the common loss functions, including the following, are convex functions:

 * L2 loss
 * Log Loss
 * L1 regularization
 * L2 regularization

Many variations of gradient descent are guaranteed to find a point close to the minimum of a strictly convex function. Similarly, many variations of stochastic gradient descent have a high probability (though, not a guarantee) of finding a point close to the minimum of a strictly convex function.

The sum of two convex functions (for example, L2 loss + L1 regularization) is a convex function.

Deep models are never convex functions. Remarkably, algorithms designed for convex optimization tend to find reasonably good solutions on deep networks anyway, even though those solutions are not guaranteed to be a global minimum.

## convex set
A subset of Euclidean space such that a line drawn between any two points in the subset remains completely within the subset. For instance, the following two shapes are convex sets:

<img src="convex_set.png" title="A rectangle and a semi-ellipse are both convex sets.">

By contrast, the following two shapes are not convex sets:

<img src="nonconvex_set.png" title="A pie-chart with a missing slice and a firework are both nonconvex sets.">

## convolutional layer
A layer of a deep neural network in which a convolutional filter passes along an input matrix. For example, consider the following 3x3 convolutional filter:

<img src="ConvolutionalFilter_example.svg" title="3x3 convolutional filter">

The following animation shows a convolutional layer consisting of 9 convolutional operations involving the 5x5 input matrix. Notice that each convolutional operation works on a different 3x3 slice of the input matrix. The resulting 3x3 matrix (on the right) consists of the results of the 9 convolutional operations:

<img src="AnimatedConvolution.gif">

## convolutional operation
The following two-step mathematical operation:

  * Element-wise multiplication of the convolutional filter and a slice of an input matrix. (The slice of the input matrix has the same rank and size as the convolutional filter.)
  * Summation of all the values in the resulting product matrix.
  
For example, consider the following 5x5 input matrix:

<img src="ConvolutionalLayerInputMatrix.svg">

Now imagine the following 2x2 convolutional filter:

<img src="ConvolutionalLayerFilter.svg">

Each convolutional operation involves a single 2x2 slice of the input matrix. For instance, suppose we use the 2x2 slice at the top-left of the input matrix. So, the convolution operation on this slice looks as follows:

<img src="ConvolutionalLayerOperation.svg">

A convolutional layer consists of a series of convolutional operations, each acting on a different slice of the input matrix.

## decision tree
A model represented as a sequence of branching statements. For example, the following over-simplified decision tree branches a few times to predict the price of a house (in thousands of USD). According to this decision tree, a house larger than 160 square meters, having more than three bedrooms, and built less than 10 years ago would have a predicted price of 510 thousand USD.

<img src="DecisionTree.svg" title="A tree three-levels deep whose branches predict house prices.">

Machine learning can generate deep decision trees.

## hashing
In machine learning, a mechanism for bucketing categorical data, particularly when the number of categories is large, but the number of categories actually appearing in the dataset is comparatively small.

For example, Earth is home to about 60,000 tree species. You could represent each of the 60,000 tree species in 60,000 separate categorical buckets. Alternatively, if only 200 of those tree species actually appear in a dataset, you could use hashing to divide tree species into perhaps 500 buckets.

A single bucket could contain multiple tree species. For example, hashing could place baobab and red maple—two genetically dissimilar species—into the same bucket. Regardless, hashing is still a good way to map large categorical sets into the desired number of buckets. Hashing turns a categorical feature having a large number of possible values into a much smaller number of values by grouping values in a deterministic way.

## intersection over union (IoU)
The intersection of two sets divided by their union. In machine-learning image-detection tasks, IoU is used to measure the accuracy of the model’s predicted bounding box with respect to the ground-truth bounding box. In this case, the IoU for the two boxes is the ratio between the overlapping area and the total area, and its value ranges from 0 (no overlap of predicted bounding box and ground-truth bounding box) to 1 (predicted bounding box and ground-truth bounding box have the exact same coordinates).

For example, in the image below:

  * The predicted bounding box (the coordinates delimiting where the model predicts the night table in the painting is located) is outlined in purple.
  
  * The ground-truth bounding box (the coordinates delimiting where the night table in the painting is actually located) is outlined in green.

<img src="iou_van_gogh_bounding_boxes.jpg">

Here, the intersection of the bounding boxes for prediction and ground truth (below left) is 1, and the union of the bounding boxes for prediction and ground truth (below right) is 7, so the IoU is 1/7.

<img src="iou_van_gogh_intersection.jpg">
<img src="iou_van_gogh_union.jpg">
