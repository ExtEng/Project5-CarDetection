# Vehicle Detection
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)


In this project, My requirements was to write a software pipeline to detect vehicles in a video.  

[//]: # (Image References)

[image1]: ./output_images/CarImage.jpg "Sample Images of the Car Training Dataset"
[image2]: ./output_images/NonCarImage.jpg "Sample Images of the Non-Car Training Dataset"
[image3]: ./output_images/CarHOG_YUV.jpg "Histogram of Gradients for sample car images"
[image4]: ./output_images/NonCarHOG_YUV.jpg "Histogram of Gradients for sample car images"
[image5]: ./output_images/SlidingWin.jpg "Sliding & Scaled Search Windows"
[image6]: ./output_images/HotWin.jpg "Windows with Car Detections"
[image7]: ./output_images/Heatmap.jpg "Heat Map of all Car Detections"
[image8]: ./output_images/Heatmap_thres.jpg "Heat Map of all Car Detections after Threshold"
[image9]: ./output_images/draw_img.jpg "Final Image of Vehicle Detection"

The Project
---

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

## Step 1: Load & Explore the Data

The Vehicle and Non-Vehicle images were extracted from the provided libraries. I plotted out the images from both datasets (as seen below) to explore the data. From the extracted dataset, the total number of car images was 8792 and the number of non-car images was 8968.

![alt text][image1]
![alt text][image2]

## Step 2: Perform HOG Feature Extraction
In order to grasp which parameters provided the HOG features with the most contrast between vehicles and nonvehicle, I ran the extraction on multiple types of color spaces, RBG, HSV,etc. After a few trials, I found that YUV provided a decent visual contrast in HOG features. These trials extended to training the classifier and getting the testing accuracy as a measure.

See Below of an example of HOG features for Grayscale (2nd Coolum) and YUV (Colours 3-5).

![alt text][image3]
![alt text][image4]

## Step 3: Extract Features and Classify
The general Methodology follows the same flow as that presented in the course:
* Extract HOG features (Parameters were tuned)
  * ColorSpace: RGB, HSV, LUV, YUV & YCrCb.
  * Orientations: 6 & 9
  * Pixels per cell 8, 12 & 16
  * Spatial Size : 24, 32, & 48
  * Cells per block: 2, 3, & 4
  * Histogram bins 24, 32, & 48
* Scale the data and run a classifier
  * LinearSVC, DecisionTree, & MLPClassifier

After testing all the variations of these parameters, the Classifier testing's accuracy showed that, a MLP classifier running of HOG features (with parameters HSV, 9 orientations, 8 pixels per cell, 12 Spatial Size, 48 Histogram bins) was the best choice at 99.3 % accuracy. However, during implementation and testing on the test images taken from the project video, the Linear SVC (with parameters HSV, 9 orientations, 8 pixels per cell) proved to have the best balance between generalization (performed better on images not seen before) and least false positives. It should be noted the Decision Tree classifier generally had the lowest accuracy between the classifiers.

I also output the classifier's results and the image in question as a sanity check. I computed the classifier's results and image for 20 random images for both car and noncar images

## Step 4: Windowed Search & Vehicle Detection

For the Window search, I tried multiple routines and parameters to optimize processing time and identification. For the Parameters, I trial and errored search window size from (48x48 to 256X256), percent overall (50% to 95 %), and total search bounds from static size to 16 pixel decrement per iteration. The routine which I found yielded the most "hot windows" was a routine were I had 8000 search windows, of varying pixel size from 32x32 to 256x256 (in 16 px increments) and the total search bounds was y =(380 to 650) x =(full range). However, this also had multiple false positives and took approximately 13 hours to finish computing the project video. Video is labelled Output_project_video_scanwindow_48_no_resize.mp4. However, this is not that can be practically done in real time. From this maximum "capture" point I lowered my search windows until I achieved a balance in computing and "detection". 

The final implementation used scaling factor approach. Starting from max search window size 256x256, y search bounds (between 380 to 650) both where decremented by 16 pixels, i.e. 2nd rounds search window size was 240*240 and y bounds were between 380 to 644. with 80% percent overlay. The derivation and labelling of the "positive car" results code and methodology follows code presented in class.

In regards to training time, generally the Linear SVC took between 17 to 23 seconds to train (when considering only changing the colorspace). The best time recorded for the Linear SVC was 3.07 seconds to train, for the parameters HSV, 9 Orientation, 16 pixels per cell, 2 cells per block, 12 spatial size & 32 Histogram bins, with a testing accuracy of 98.4%. 

The Desicion tree classifier took on average 20 times longer to train, varying from 175 to 265 secs, with accuracy never going above 89 %. Due to training time required and testing accuracy, I chose not to use this classifier for the pipeline testing.

The MLP classifier was by far the fastest to train (t ~ 3 to 8 secs) and achieved the highest accuracy of 99.3%. However, I did not select this for the final implementation, because it seemed to overfit the training data, and have alot fewer car positive matches on new car data.

The Final routine can be seen below:
* Specify search windows
![alt text][image5]
* Find "hot" Windows where classifier predicts 1 i.e. car 
![alt text][image6]
* Draw Heat map from all the hot windows 
![alt text][image7]
* Apply a threshold to areas where less than or equal to 1 hot window  
![alt text][image8]
* Apply a label to the original image that corresponds to the box or boxes that correspond to the threshold heatmap.
![alt text][image9]

## Step 5: Windowed Search & Vehicle Detection - Video Pipeline

To make the pipeline a little more robust, I created a history class, to store 1. sliding search windows and, 2.the heat maps from the last 20 frames. Using the history class I was able to conduct a rolling average of all the previous car detections. This elimated a majority of the jitter and  false positives. The processing and video output is imbedded at the end of my Project Jupiter notebook.

The output video for the final submission is labelled:
Output_project_video_scanwindow_64_16_y_resize.mp4


## Discussions

While time was of the essence and the algorithm produced decent results, I still feel like more work is required to make this pipeline robust. 
* I think including more types of training data of cars, will help generalize and improve results. The data also need to be augmented with different road vehicles, i.e. motorbikes, trucks (trailer, fire, and tow), ambulances etc., because currently the pipeline will fail if any of these vehicles are seen.
* While this implementation of the routine performed "relatively" fast I still believe this pipeline requires a greater increase in speed to be utilized on the road, without too much measurement delay. 
* While images can provide us a cost-effective sensor to detect vehicles far away, I believe this pipeline should be augment with data from a range finder or a radar system to provide more reliable data. 

