# Unsupervised Stock Market Features Construction using Generative Adversarial Networks(GAN)
Deep Learning constructs feature using only raw data. The leaned representation of the data outperforms expert features for many modalities including Radio Frequency ([Convolutional Radio Modulation Recognition Networks](https://arxiv.org/pdf/1602.04105.pdf)), computer vision ([Convolutional Deep Belief Networks for Scalable Unsupervised Learning of Hierarchical Representations](https://www.cs.princeton.edu/~rajeshr/papers/icml09-ConvolutionalDeepBeliefNetworks.pdf)) and audio classification ([Unsupervised feature learning for audio classification using convolutional deep belief networks](http://www.robotics.stanford.edu/~ang/papers/nips09-AudioConvolutionalDBN.pdf)). 
# GAN 
In the case of Convolutional Neural Networks (CNN), the data representation is learned in a supervised fashion with respect to a task such as classification. GANs lean features in an unsupervised fashion. The competitive learning process for GANs results in more of the possible feature space being explored. This reduces that potential for features being overfitted to the training data. Since the features are constructed in an unsupervised process classification algorithms trained on the features will generalize on a smaller amount of data. In fact, GANs promote generalization beyond the training data. 
![gan.png]({{site.baseurl}}/media/gan.png)
The Generator is trained to generate data that looks like historical price data of the target stocks over a distribution. The Discriminator is trained to tell the difference between the data from the Generator and the real data. The loss from the Discriminator (how the Discriminator has learned to tell if a sample in real or fake) is used to train the Generator to defeat the Discriminator. The competition between the Generator and the Discriminator forces the Discriminator to distinguish random from real variability while the Generator learns to map a distribution into the sample space.    
This project explores Bidirectional Generative Adversarial Networks(BiGANs) based on the paper [Adversarial Feature Learning](https://arxiv.org/pdf/1605.09782.pdf). The primary difference in the BiGAN the Discriminator learn to determine the joint probability P(X, Z) = real/fake. Where X is the sample and Z is the generating distribution. This, in turn, means that the Generator learns to encode a real sample into its generating distribution.  ![BiGAN.png]({{site.baseurl}}/media/BiGAN.png)

*Figuer from Adversarial Feature Learning*


This project makes a modification to the BiGAN. Rather than learning to encode a real sample into the generating distribution. The model learns to encode the features learned by the discriminator into the generating distribution. For historical stock data, this architecture outperformed the BiGAN architecture. 
# Approach 

**Data**
Historical prices of stocks are likely not very predictive of the future price of the stock, but it is free data. 

**Training**
The GAN is trained on 96 stocks off the Nasdaq. Each stock is normalized using a 20-day rolling window (data-mean)/(max-min). The last 2 years (504 days) of trading are held out as a test set. Time series of 20 day periods are constructed and used as input to the GAN. Once the GAN is finished training, the leaned encoding for the Discriminator features to the generation distribution is used as the new representation of the data. The features are not guaranteed to be predictive of the direction of the stock market, but for other modalities, they have been shown to work well. Random Forests is trained to classify whether the stock will gain 10% over the next 10 trading days. This creates an unbalanced training set so the majority class is undersampled before training the Random Forest. 

**Results**
First, let's visualize the features leaned by the BiGAN. TSNE will be used to reduce the dimensions too 2D.
![tsne.png]({{site.baseurl}}/media/tsne.png)
The red dots are negative samples and green are positive samples. There appears to be some structure to the features leaned by the BiGAN, but not related to the target. More experimentation is needed to see what the structure is following, but I suspect the each curve is a specific stock. 

Since the classes are unbalanced, due to not many stocks gaining 10% in 10 days, accuracy is a poor metric. If we always predicted that stocks would not go up then the accuracy would be above 90%. So instead of accuracy, we will use Area Under the Curve (AUC). Check out this video to learn more about [AUC](http://www.dataschool.io/roc-curves-and-auc-explained/). An AUC of 1 would be a perfect model while an AUC of 0.5 means that the model performs the same as randomly picking a label. We can visualize the performance of the classifier using a ROC curve. 
![ReceiverOperatingCharacteristic.png]({{site.baseurl}}/media/ReceiverOperatingCharacteristic.png)

This show that the classifier is only a little better than random. Another way of visualizing the performance of the classification algorithm is a confusion matrix. 
![Full_Confusion_Matrix.png]({{site.baseurl}}/media/Full_Confusion_Matrix.png)
The top row shows that the classifier correctly classified 29007 samples as true negatives while misclassifying 16780 same as false positives. The bottom row shows that the classifier misclassified 900 samples as false negatives and correctly classified 900 examples as true positives. The perfect confusion matrix would only have values on the diagonal.

There are a very large number of false positives. If the false positives make some money just not 10% this could still be a valid trading strategy. So let's look at the distribution of the percent returns from the false positives.
![Distribution_of_False_Positives.png]({{site.baseurl}}/media/Distribution_of_False_Positives.png)

The distribution of the returns on the false positives does not look promising. They appeared skewed towards negative returns. 

![Distribution_of_Positive_Predictions.png]({{site.baseurl}}/media/Distribution_of_Positive_Predictions.png)

If we look at the overall distribution of the returns of all positive predictions, it still apperss skewed toward negatiev returns. The mean return for all posstive predition end up being -0.18 a small negitive return.
