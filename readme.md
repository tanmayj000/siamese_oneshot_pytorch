# Siamese Network for One-shot Learning

This is the unofficial pytorch implementation of the [original paper](https://www.cs.cmu.edu/~rsalakhu/papers/oneshot1.pdf), trained and tested on the [Omniglot dataset](https://github.com/brendenlake/omniglot). In this implementation, I achieved **90%** precision, 2% less than the officially reported precision in the paper.

## One-Shot Learning

Currently most deep learning models need generally thousands of labeled samples per class. Data acquisition for most tasks is very expensive. The possibility to have models that could learn from one or a few samples is a lot more interesting than having the need of acquiring and labeling thousands of samples. One could argue that a young child can learn a lot of concepts without needing a large number of examples.  This is where one-shot learning appears: the task of classifying with only having access of one example of each possible class in each test task. This ability of learning from little data is very interesting and could be used in many machine learning problems. 

Despite this paper is focused on images, this concept can be applied to many fields. To fully understand the problem we should describe what is considered an example of an one-shot task. Given a test sample, X, an one-shot task would aim to classify this test image into one of C categories. For this support set of samples with a representing N unique categories (N-way one shot task) is given to the model in order to decide what is the class of the test images. Notice that none of the samples used in this one-shot task have been seen by the model (the categories are different in training and testing). 


## Omniglot Dataset

<p align="center">
  <img src="https://user-images.githubusercontent.com/10371630/36079867-c94b19fe-0f7f-11e8-9ef8-6f017d214d43.png" alt="Omniglot Dataset"/>
</p>

The Omniglot dataset consists in 50 different alphabets, 30 used in a background set and 20 used in a evaluation set. Each alphabet has a number of characters from 14 to 55 different characters drawn by 20 different subjects, resulting in 20 105x105 images for each character. The background set should be used in training for hyper parameter tuning and feature learning, leaving the final results to the remaining 20 alphabets, never seen before by the models trained in the background set. Despite that this paper uses 40 background alphabets and 10 evaluation alphabets. 

This dataset is considered as sort of a MNIST transpose, where the number of possible classes is considerably higher than the number of training samples, making it suitable to one-shot tasks. 

The authors use 20-way one-shot task for evaluating the performance in the evaluation set. For each alphabet it is performed 40 different one-shot tasks, completing a total of 400 tasks for the 10 evaluation alphabets. An example of one one-shot task in this dataset can be seen in the following figure: 

<p align="center">
  <img src="https://user-images.githubusercontent.com/10371630/36079892-1df60568-0f80-11e8-8297-a7c6beec4491.png" alt="One-Shot Task"/>
</p>

Let's dive into the methodology proposed by Koch_et al._ to solve this one-shot task problem.

## Methodology

To solve this methodology, the authors propose the use of a Deep Convolutional Siamese Networks.  Siamese Nets were introduced by Bromley and Yan LeCun in the 90s for a verification problem. 
Siamese nets  are two twin networks that accept distinct inputs but are joined in by a energy function that calculates a distance metric between the outputs of the two nets. 
The weights of both networks are tied, allowing them to compute the same function. 
In this paper the weighed L1 distance between twin feature vectors is used as energy function, combined with a sigmoid activations. 

This architecture seems to be designed for verification tasks, and this is exactly how the authors approach the problem. 

In the paper a convolutional neural net was used. 3 Blocks of Cov-RELU-Max Pooling are used followed by a Conv-RELU connected to a fully-connected layer with a sigmoid function. This layer produces the feature vectors that will be fused by the L1 weighed distance layer. The output is fed to a final layer that outputs a value between 1 and 0 (same class or different class).  To assess the best architecture, Bayesian hyper-parameter tuning was performed. The best architecture is depicted in the following image:

<p align="center">
  <img src="https://user-images.githubusercontent.com/10371630/36121224-71403aa0-103d-11e8-81c6-6caae24a835c.png" alt="best_architecture"/>
</p>

L2-Regularization is used in each layer, and as an optimizer it is used Stochastic Gradient Descent with momentum. As previously mentioned, Bayesian hyperparameter optimization was used to find the best parameters for the following topics:
- Layer-wise Learning Rates (search from 0.0001 to 0.1) 
- Layer-wise Momentum (search from 0 to 1)
- Layer-wise L2-regularization penalty (from 0 to 0.1)
- Filter Size from 3x3 to 20x20
- Filter numbers from 16 to 256 (using multipliers of 16)
- Number of units in the fully connected layer from 128 to 4096 (using multipliers of 16)

For training some details were used:
- The learning rate is defined layer-wise and it is decayed by 1% each epoch.
- In every layer the momentum is fixed at 0.5 and it is increased linearly each epoch until reaching a value mu.
- 40 alphabets were used in training and validation and 10 for evaluation
- The problem is considered a verification task since the train consists in classifying pairs in same or different character. - After that in evaluation phase, the test image is paired with each one of the support set characters. The pair with higher probability output is considered the class for the test image. 
- Data Augmentation was used with affine distortions (rotations, translations, shear and zoom)

## Requirements
After activating a new virtual environment, install requirements for this implementation.

```shell
pip install -r requirements.txt
```

## Setup and Run
- Download dataset
```
git clone https://github.com/brendenlake/omniglot.git
cd omniglot/python
unzip images_evaluation.zip
unzip images_background.zip
cd ../..
```
- Begin training using: 
```shell
python3 train.py --train_path omniglot/python/images_background \
                 --test_path  omniglot/python/images_evaluation \
                 --gpu_ids 0 \
                 --model_path models
```


## Flags:
```shell
--cuda #If you have a GPU 
--train_path #Path to training dataset
--test_path #Path to test dataset
--way #Number of classes
--times #Number of samples trained after test accuracy is evaluated
--batch_size 
--lr #Learning Rate
--show_every #Show results after every x iterations
--save_every #Save weights after every x iterations
--max_iter #Number of iterations before training stops
--model_path #Path to store weights
--gpu_ids 
```

## Results
Loss value is sampled after every 200 batches
![img](https://github.com/fangpin/siamese-network/blob/master/loss.png)
There are a few changes from the original implementation proposed in the paper:

- Learning rate changed

- ADAM optimizer instead of SGD with momentum

## References

-  [Original Paper](https://www.cs.cmu.edu/~rsalakhu/papers/oneshot1.pdf): Koch, Gregory, Richard Zemel, and Ruslan Salakhutdinov. "Siamese neural networks for one-shot image recognition." ICML Deep Learning Workshop. Vol. 2. 2015.

-  [Siamese-Networks-for-One-Shot-Learning by tensorfreitas](https://www.cs.cmu.edu/~rsalakhu/papers/oneshot1.pdf)
