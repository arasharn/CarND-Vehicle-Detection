## Writeup Template
### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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
[image1]: ./Screenshot2018-09-1709.13.05.png
[image21]: ./Screenshot2018-09-1709.48.03.png
[image22]: ./Screenshot2018-09-1709.49.31.png
[image3]: ./Screenshot2018-09-1712.56.06.png
[image4]: ./Screenshot2018-09-1713.15.16.png
[image41]: ./Screenshot2018-09-1713.40.00.png
[image5]: ./Screenshot2018-09-1713.30.40.png
[image6]: ./Screenshot2018-09-1713.41.52.png
[image7]: ./examples/output_bboxes.png
[video1]: ./project_video_output.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the first code cell 8 through 14 of the IPython notebook 

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is 10 examples of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different on cell 10 and tried to fit the same linear SVM on a subset of data. Based on the results I chose the YCrCb color map with orientation of 8, with 8 pixel per cell 2 block per cell, in all HOG channels. This selected features were replaced in class `FeaturesParameters` in cell 9 and fit the classifier on that (Cell 11).

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=8`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)` for vehicle:


![alt text][image21]

And for non-vehicle here:

![alt text][image22]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried to use a subset of data to measure the performance of the classifier for most of the possible combination in cell 10. Then based on results, the different color channels with their HOG features were tested on Cells 13 and 14. Based on the results my final feature combination was as follow (This parameters can be found on cell 9 in FeaturesParameters class):

* Color space: YCrCb
* HOG orient: 8
* HOG pixel per cell: 8
* HOG cell per block: 2
* Spatial bin size: (16,16)
* Histogram bins: 32
* Classifier: Linear SVC

It should be noticed that data was standardized (mean zero with standard deviation of 1) by using `sklearn.StandarScaler()` in cell 7 (and also 5). I also spliced the data to training (80%) and test (20%) sets in cell 6 to validate the model that gives the accuracy of 99.16%  

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

Initially multiple windows were defined and then feature extraction was done for each of the windows in each image to see if any car is in them. This part can be found on cells 12 and 14. The hyper parameter over lap (`xy_overlap`) and scaler (`scaler `) were chosen try an error. Also, the rest of the parameters were chosen based on fitting different combination on cell 5.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

Initially multiple windows were defined and then feature extraction was done for each of the windows in each image to see if any car is in them. This part can be found on cells 16 through and 18. The hyper parameter over lap (`xy_overlap`) and scaler (`scaler `) were chosen try an error. Here is the results, using the test images:

![alt text][image3]

Later for combining the results of multiple boxes in an image, reduce false positives, a heat map was implemented with a threshold  and the function `label()` from `scipy.ndimage.measurements` was used to find where the cars we (cells 18 and 19). with step by step process:


And this images shows the final results:

![][image41]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  To make the HOG faster; a HOG sub-sampling was implemented as suggested on Udacity's lectures. The implementation of this method could be found on In cells 16 through 19. Image below show the initial results:

![alt text][image5]

Then the same threshold and heat map function were used to reduce the false positives:

![alt text][image4]

And the final result:

![alt text][image6]

---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

The positions of detected cars in each frame of the video were recorded.  From the positive detections a heatmap was created and then thresholded that map to identify vehicle positions.  Then the `scipy.ndimage.measurements.label()` was used to identify individual blobs in the heatmap, which each blobs considered as a vehicle. Next a bounding boxes was constructed to cover the area of each blob detected.  

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?


The major issue I was encountered with was that the algorithm kept identifying many false positives. I addressed this by tweaking the parameters, training on the full image set, by limiting the search region. Also, I defined a filter by using the heat map to reduce the false positives. It has been noticed that the car with dark colors  tend to make the algorithms less robust and also one way to improve the algorithm would be to spend some time smoothing the boxes by giving the bounding box sizes some history and averaging. Also, multiple scaling could be used to improve the results.

