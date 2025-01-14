---
layout: post
comments: true
title: Object Detection and Classification of Cars by Make using MMDetection
author: Ben Klingher, Erik Ren
date: 2023-01-29
---

<!--more-->

# Object Detection and Classification of Cars by Make using MMDetection

## Abstract

The detection and classification of vehicle make and model in images is a challenging task with various practical applications, including parking administration, dealership stock management, surveillance systems, and civil security. In this project, we used the MMDetection library to build models for detecting and classifying the make of cars trained on annotated images from the Stanford Cars dataset. Because this dataset dates from 2013, it displayed issues generalizing to more modern cars. Other challenges arose in the approach to finetuning the different detector models and with the large number of classes we were attempting to classify. To address these challenges, we explored various methods, including limiting the number of classes to just the make of a car, using pre-trained (COCO dataset) object detectors and finetuning them to detect car makes, and, finally, creating a custom Resnet model (pretrained on Imagenet) to apply on top of an existing pretrained COCO detector. Our experiments revealed that finetuning pretrained detector models alone did not deliver satisfactory results, and we achieved better performance by limiting the number of classes and adding a secondary classification model, after detecting the bounding box for the cars with a model better trained to that application.

## Presentation Video

Our presentation is here: https://youtu.be/ndCu8zopa9M

## Introduction

We leveraged the MMDetection library to train a model to detect and classify the make of cars using the Stanford Cars dataset with the goal of applying it to real-life photos from the UCLA neighborhood. Although we were able to develop a successful model to detect and classify the makes of cars, this task presented significant challenges including the relatively large number of classes (combinations of make model and year) and the imbalanced class distribution, as well as the deep subtlety needed to identify specific car models.

The Stanford Cars dataset consists of 16,185 images each annotated with bounding boxes labeled into one of 196 classes, which indicate the make, model, and year for the car. While the dataset is roughly balanced in the specific classes of make, model and year, it is particularly unbalanced with respect to just the make of the vehicle, with some makes having a substantially smaller number of samples than others. It is also very spotty on the coverage of vehicles encountered in public. For instance, it does not have any Subaru cars in the dataset. Another significant challenge was the outdated nature of the Stanford Cars dataset. The dataset was last updated in 2013, making it difficult to accurately detect the make of newer cars in current day photos/videos. Moreover, the dataset's images were taken in a controlled environment, often in dealerships, making it challenging to generalize the trained model to real-world scenarios.

Many of the models we employed in this project were pretrained, so it is useful to discuss the contents of those datasets. The COCO dataset is an object detection dataset which includes annotations for a wide variety of recognition tasks, including object detection and classification. It categorizes objects into 80 different categories that range from a wide variety of different objects, from vehicles to foods. The Resnet model we ultimately trained for classifying the make of cars is pretrained on Imagenet, which is a familiar dataset.

Early on in this project, we discovered that classifying make/model/year of cars is a difficult task. Even many humans struggle with this. We encountered a new issue when we realized the very different types of classification that the pretrained models performed versus the classification necessary to differentiate specific vehicle models. The object detection models that we employed in the MMDetection library were trained primarily on the COCO dataset. In that dataset, object classes vary from “cars” to things like “cows”, “spoon”, “banana”, etc. The types of features that a convolutional neural network would extract to perform that kind of classification are very different from those necessary to identify even the make, let alone the model and year, of a car. In the latter case, much finer details of the car itself are necessary to make those kinds of distinctions.

First we attempted to use the pre-trained detection models and fine-tuning techniques, but found that pre-trained models alone did not deliver satisfactory results due both to the dataset's large class distribution and the nature of the features found in the pretrained model. Consequently, we explored several methods to handle these challenges including limiting the total number of classes to just the make of a car and creating a custom Resnet model to further improve overall performance. We also explored several other car datasets like the Car Connection dataset in order to include more recent car models that were not present in the Stanford Cars dataset.

To address these challenges, we iterated on different approaches to the problem, from fine tuning existing object detection methods, to training an independent classification model. We provide a detailed account of our approach to addressing the challenges of car make classification and describe our model selection and training process. We also discuss our efforts to handle the issues with the low performing model and how we re-organized the data and our model architecture to greatly improve our results. Furthermore, we present the results of our experiments, including an evaluation of the effectiveness of our approach in accurately detecting the make of cars in real-life photos from the UCLA neighborhood.

## Method

Our initial approach was to use a pre-trained object detector in MMDetection that had been trained on the COCO dataset and fine-tune it to the Stanford Cars dataset. In addition to the task of accurately generating the bounding boxes for the cars, we aimed to identify the make, model, and year of the cars depicted in each image. 

The Stanford Cars dataset includes bbox annotations, so we ran a long series of training cycles using its annotations to see if we could correctly detect and label the make/model/year of these images. We tried several different models pretrained on the COCO dataset. Our initial experiment was with one version of the Faster R-CNN model. We also tried using a version of Mask R-CNN with the mask head turned off. Ultimately, we had the best results using the small version of YOLOX provided by MMDetection.

Training was complicated by the size of the dataset and the limited GPU memory. After an initial period of testing using a mini dataset, we then ran eight hour periods of training for each of these models. In all of our initial trials, these resulted in extremely poor results. All the mAP values for validation were near zero and running inference on test, or even training, images resulted in no bounding box found at all, or only boxes with very low confidence scores.

Eventually we decided that the 196 classes of make/model/year were too many categories to accurately train a fine tuned model. So, we decided to move to training only to classify by the make of the vehicle. This approach reduced the number of classes to 49, thereby improving the model's performance slightly. Using YOLOX as our initial pretrained model, we trained for over 20 hours on the Stanford dataset, and did begin to see some reasonable results. However, the model still struggled to return results for many test images. A particular problem was the bounding boxes output by the model. Even if the classification of the make was correct the bounding box often failed to correctly identify the care in the image.

This led us to a new consideration. These models pretrained on the COCO dataset are already particularly good at drawing bounding boxes around cars in a wide variety of images. Therefore, the logical approach would be to use the pretrained detection model to generate the bounding boxes which we would classify with a secondary classifier, trained independently.

So, we decided to train a custom Resnet model to classify the make of the cars. We tried various approaches to this classification model. We began by training a Resnet50 on the classification task. We tried both Resnets pretrained on Imagenet and fresh Resnet models. We found that those pretrained on ImageNet reach satisfactory performance far more quickly than those without the pretraining. The Resnet50 was able to achieve a test accuracy of about 60% in classifying the make of vehicles, which was too low for our goals. We then trained a Resnet10 pretrained on ImageNet and we were able to reach a test accuracy of 75% for classifying the make of the vehicles.

Despite the promising results of our custom Resnet model, its performance on newer cars in real-life photos was not consistent. This issue arose due to the limitations of the Stanford Cars dataset, which contained only images up until the year 2013, leaving out a substantial portion of cars that are commonly found on roads today. To overcome this challenge, we attempted to expand our dataset size by incorporating additional data from the Car Connect dataset, which includes more recent images of cars. One issue with this dataset was the type of car images included; many of the images were of the interior of the car which is not useful for our task. This required us to filter out the images in the dataset by using YOLOv3 to filter out the interiors of cars. By doing so, we were able to collect a few thousand more images of newer cars, but it was not significant enough to improve our performance. Due to the issues of merging these two datasets and the failure to deliver increased performance, our final Resnet model was trained only on the Stanford Cars dataset.

We also experimented with different thresholds for rejecting the labels. If the YOLOX model did not have at least 70% confidence that the object was a car then it was not labeled. Because of the number of classes in our model, we often had lower confidence levels in the classification stage, so we accepted labels with a confidence of at least 30% in our examples shown in this report. Even this low confidence limit has been shown to be useful, as will be demonstrated below.

In the end, our best performing architecture used a YOLOX model pretrained on the COCO dataset to draw the bounding box around the vehicles, which is then passed to the Resnet classifier to identify the make of the vehicle in the bounding box. 

## Results

Our top performing architecture yielded promising results with a validation score of around 75% for our custom Resnet model on the Stanford Cars dataset. The average precision at IoU=0.5 for the final model was around 0.5 based on annotations in the validation set. The fairly low average precision was mainly due to the large number of incorrect labels. Because our classification based architecture returns a label for each bounding box as long as it achieves a minimum of 30%, we are skewed to produce more false negative labeling. 

Here we will show and compare several results from the fine-tuned YOLOX architecture, as well as the top-performing model stacked with the Resnet classifier.

The following two images show how the output of the COCO-trained YOLOX model predicts more than three cars in the image, but our threshold removes several that cannot be adequately classfied in the final image

![Audi](../assets/images/team49/audi_yolo.png)
*Many cars predicted in image*

![Audi](../assets/images/team49/audi_resnet.png)
*Notice that several of the car predictions have been removed while the rest are correctly classfieid*

Below the images show how the finetuned model does worse than the Resnet backed model on generating the correct boxes.

![Ford Finetuned](../assets/images/team49/ford_finetuned_bad_box_good_label.png)
*Finetuned model here gives the correct label, but a poor bounding box*

![Ford Resnet](../assets/images/team49/ford_resnet_good_box_bad_label.png)
*Resnet model here gives the wrong label, but a good bounding box (as the result of YOLO trained on COCO dataset)*

Below we look at another example of poor bounding boxes generated by the finetuned model.

![Ford Finetuned](../assets/images/team49/ford_finetuned_bad_box.png)
*Finetuned model gives a poor bounding box*

![Ford Resnet](../assets/images/team49/ford_resnet_good_box.png)
*Resnet model here gives a good bounding box. It also correctly labels the second car*

Below we see how in many cases, the Resnet-backed model was better at identifying multiple cars in the image.

![Jeep Finetuned](../assets/images/team49/finetuned_jeep.png)
*Finetuned model gives a poor bounding box*

![Jeep Resnet](../assets/images/team49/resnet_jeep.png)
*Resnet model here gives a good bounding box. It also correctly labels the other cars*

Now, we'll try images from wild. The following images we recorded ourselves.

First, we see how the model correct labels one car, while providing no label to one that is not in the dataset. The Stanford Cars dataset does not include any Prius, so it correctly gives it no label.

![Prius](../assets/images/team49/threshold_prius.png)
*Correct label for the Honda, none for the Prius*

Though performing fairly well on the test data, we experienced challenges with the accuracy of our model on real-life photos, particularly with newer cars. This can be attributed to the outdated nature of the Stanford Cars dataset, which only included cars up until the year 2013, leaving out a significant number of cars commonly seen on roads today.

In the following video you can see the failure of the model to label the cars driving on Wilshire Blvd.

![Wilshire](../assets/images/team49/wilshire3.gif)
*Most of these labels are incorrect. Also, the labels are very unstabel in this type of complex environment with modern cars and those outside the dataset*

Last but not least is my car in my parking garage, correctly labeled.

![Mine](../assets/images/team49/my_hyundai_garage.png)
*My car, correct label*

## Discussion

This project provided a lot of insight into the methods of developing strong object detection tools suited to complex tasks. Our original instinct was that pretrained object detection models would generalize well to our classfication task, but we quickly realized that is not the case. We then realized that the best technique was to employ the pretrained model for the task to which it is best suited and then train a secondary model to solve the secondary problem.

More concretely, the MMDetection models pretrained on COCO are very good at differentiating a car from a banana, but the set of image features useful for drawing that distinction are very different from those necessary to differentiate specific cars. If you think about how a human performs those tasks, they would look at very different attributes. Identifying a specific car type often require reading text or looking at a logo, or possibly a strong understanding of the specific shapes of many different cars. But to identify vastly different objects like in the COCO dataset, one only need consider much broader overall details. In short, classifying cars is a subtle problem.

Because classifying cars is so different, the logical solution was to separate the car classification from the problem of drawning the bounding boxes. There is another good argument for this separation. The models pretrained on COCO are already extremely good at generating bounding boxes for cars. Finetuning the models to also classify the cars could never compete with its ability to identify cars as objects. Therefore, combining the COCO pretraining with a separate classifier proved to be the optimal approach.

The finetuning method was also particularly bad at drawing bounding boxes, and always had a near zero mAP score because of this. That was another strong argument for using the pretrained YOLOX model for identifying cars before feeding the results to a second classfier.

There were however some advantages to the finetuning method. It was more efficient when implemented, and sometimes correctly labeled cars in cases where our custom trained Resnet was incorrect, as it had a higher threshold to return a label. Also, one goal of an object detection model should be to combine all these tasks into as small an architecture as possible, as YOLO condensed the multiple stages of Faster R-CNN into a single stage. So, in the future, it is possible that longer training with better data would lead to good results for this task without the second classify we trained. Nonetheless, we found the training of such a model problematic given the data we had on hand. Finding the best approach for the dataset you have served us well.

One major insight we found is that it is much harder than expected to classify cars. Even humans struggle at it. It requires quite detailed understanding and insight into the image data in order to draw the fine distinctions between makes of cars, let alone models and years. We learned about the great limitations a dataset can impose, and the difficulty of generating better data on which to train. Clearly, the data problem is the greatest issue in the way these models are developed. Working with a particular kind of image, mostly from dealerships, that was ten years old made the problem very hard to solve in the general case. The process of data collection and augementing is critical.

Lastly, this project highlighted the value of transfer learning. In both of our stacked models we used pretraining to produce our top-performing method. The COCO pretraining made the YOLOX model particularly good at identifying cars, and the ImageNet pretraining made the training of the ResNet classifier far more efficient. Image data is a field where existing feature information can be particularly fruitful, as long as not misapplied to the wrong kind of problem.

In the future, there are many ways we would want to improve our model. We would focus on finding a more general dataset for cars as well as generating more augmentations that would help in training. We particularly want to train on more data from the wild, to apply it to real world data. We would also want to experiement with improving the finetuning of the object detection model and give more time to training that, so we can improve overall efficiency. Another goal would be to speed up the inference process to the point where it can handle real time data. Overall, there are many interesting avenues for future pursuit.

## References

Vehicle Attribute Recognition by Appearance: Computer Vision Methods for Vehicle Type, Make and Model Classification
https://link.springer.com/article/10.1007/s11265-020-01567-6

Real-Time Vehicle Make and Model Recognition with the Residual SqueezeNet Architecture
https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6427723/

Object Detection With Deep Learning: A Review
https://ieeexplore.ieee.org/abstract/document/8627998?casa_token=O2bJ9bs8fF8AAAAA:UitiBGhZBSdgAheBAPj9ZnGgW64oKa-bXSNibIaTk1oZAtDMGboHxcPq32fdaQTgN02tz0iZKA

YOLOX: Exceeding YOLO Series in 2021
https://arxiv.org/abs/2107.08430

ResNet strikes back: An improved training procedure in timm
https://arxiv.org/abs/2110.00476

Deep Learning Based Vehicle Make-Model Classification
https://arxiv.org/abs/1809.00953

Unsupervised Feature Learning Toward a Real-time Vehicle Make and Model Recognition
https://arxiv.org/abs/1806.03028

