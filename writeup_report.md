## Writeup Report for P5

---

**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./write_up_imgs/data_explore.png
[image2]: ./write_up_imgs/original.png
[image3]: ./write_up_imgs/hog.png
[image4]: ./write_up_imgs/scale11.png
[image5]: ./write_up_imgs/scale12.png
[image6]: ./write_up_imgs/scale15.png
[image7]: ./write_up_imgs/scale2.png
[image8]: ./write_up_imgs/scale25.png
[image9]: ./write_up_imgs/sliding.png
[image10]: ./write_up_imgs/heat_map.png
[image11]: ./write_up_imgs/test_imgs.png
[video1]: ./project_video_out.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

I first load data in code cell [1] of the IPython notebook and explore some data using code cell [2]. They look like:

![alt text][image1]


The feature extraction function are in code cell [4]. 

I only use color hist and HOG features. Spatial features are not used since I think there are some redundant and my cause overfitting. The code for extracting features is in code cell [7] of IPython notebook.  


I explored color spaces using `YUV` with 32 hist bins. The `skimage.hog()` parameters for HOG feature extraction are (`orientations = 10`, `pixels_per_cell = 16`, and `cells_per_block = 2`). I attached two example figures about HOG feature extraction. One for `vehicle` and one for `non-vehicle`.

Original images:
![alt text][image2]

HOG images:
![alt text][image3] 


#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters, i.e., (`orientations = 8`, `pixels_per_cell = 8`, and `cells_per_block = 2`), (`orientations = 11`, `pixels_per_cell = 16`, and `cells_per_block = 2`), (`orientations = 9`, `pixels_per_cell = 16`, and `cells_per_block = 2`), and (`orientations = 10`, `pixels_per_cell = 16`, and `cells_per_block = 2`). Finally according to extracting time, the number of features, and classifier prediction accuracy, I choose (`orientations = 10`, `pixels_per_cell = 16`, and `cells_per_block = 2`) for a moderate number of features and extracting time, as well as a relative higher prediction accuracy (`98.79%`).

The entire parameters for feature extraction are the following:

|Parameter|Value|
|:--------|----:|
|Color Space|YUV|
|HOG Orient|10|
|HOG Pixels per cell|16|
|HOG Cell per block|2|
|HOG Channels|All|
|Histogram bins|32|


#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using LinearSVC and due to I use both color hist feature and HOG feature, I used a scaler to nomalize the extracted features. I split my data with ratio 0.2, which results in around 14200 training data and 3500 test data. To avoid the bias caused by data order, I shuffled the whole data. 

The classifier information is:
|Name|Type|
|:---|---:|
|Classifier|LinearSVC|
|Scaler|StandardScaler|

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I combined different scale sliding windows and searched in different region. Actually, I noticed that the vehicles appeared in area with y from 350 to 450 are usually small and the vehicles appeared in area with y from 400 to 600 are usually large. In this case, some small scale sliding windows were used in middle area of the image and large sliding windows wee used in the whole bottom half of image. There are some different scale sliding windows' results:

scale = 1.1
![alt text][image4]

scale = 1.2
![alt text][image5]

scale = 1.5
![alt text][image6]

scale = 2.0
![alt text][image7]

scale = 2.5
![alt text][image8]

Finally, I combined all different scale sliding windows and the result for one of the test images is

![alt text][image9]

To reduce the false positive, I used heat map to filter out those detected non-vehicles. Through many times experiments, I finally choose heat map threshold with 2, which provided good performance for all 6 test images. The heat map illustration is shown:

Heat map for test image 6
![alt text][image10]


#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Final results for all 6 test images are:

![alt text][image11]

---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video_out.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I have applied a heatmap and threshold in the pipeline to minimize false positives. Furthre more, I used the function label() from scipy.ndimage.measurements to deal with multiple bounding boxes and find the vehicles. To filter false positives, the image heatmap map was combined over 20 consecutive frames. The implementation could be found in code cell [22] and [23].

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.

Actually, at the first a few times experiments, I only have a classifier prediction accuracy around `96%`, which I thought should be fine enough. However, when I tested on different test images, I found that there were too many false positive results. Therefore, I tuned the parameters and finally achieved `98%` prediction accuracy to have a acceptable result. To further improve the performance, I would like to try some more advanced classifier to achieve a higher prediction accuracy.

I also encountered some problems with choosing a proper heat map threshold, sliding window searching area, and scales. I think it would be better to have an adaptive method to choose the heat map threshold, this is because for some simple images, i.e., test image 3, there are only a few detected sliding windows, you may want to use a small threshold. But for some other images, i.e., test image 5, many detected sliding windows appeared and a larger threshold may render better result. For better sliding window searching area and scales, it is better to explore more images and find some properties like: far-end vehicle usually has a small size in the image may only require a small scale sliding window and near-end vehicle appears a larger size that a larger scale and searching area may be needed.

Since I have tried so many different parameters during the project, I think the end-to-end deep learning methods may reduce some tuning works and achieve more robust results. I would like to implement some deep learning methods for this project in the future.

