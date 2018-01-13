
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
[image1]: ./output_images/Sample-Car-NotCar.png
[image2]: ./output_images/Hog-Features-Car-NotCar2.png
[image3]: ./output_images/Detection-at-different-scales.png
[image4]: ./output_images/test1out.png
[image5]: ./output_images/test2out.png
[image6]: ./output_images/test3out.png
[image7]: ./output_images/test4out.png
[image8]: ./output_images/test5out.png
[image9]: ./output_images/test6out.png
[image10]: ./output_images/vframe010out.png
[image11]: ./output_images/vframe011out.png
[image12]: ./output_images/vframe012out.png
[image13]: ./output_images/vframe013out.png
[image14]: ./output_images/vframe014out.png
[image15]: ./output_images/vframe015out.png
[video1]: ./project_video_output.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. The Writeup / README that includes all the rubric points and my attempt at addressing each of them is [Here](https://github.com/gvogety/CarND-Vehicle-Detection/blob/master/README.md).

You're also reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.


HOG features for the training images are extracted as part of extract_features()-->single_img_features()-->extract_hog_features(). All 3 channels are extracted for training purposes.
The code for this step is contained in the first code cell of the IPython notebook (or in lines # through # of the file called `some_file.py`).  

First, from the project training set,  all the `vehicle` and `non-vehicle` images are read.  Here is an example of a few of the `vehicle` and `non-vehicle` images:

![alt text][image1]

HOG features for the training images are extracted as part of extract_features()-->single_img_features()-->extract_hog_features(). All 3 channels are extracted for training purposes.

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the YCrCb color space (all 3 channels) and HOG parameters of `orientations=9`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:


![alt text][image2]

#### 2. Explain how you settled on your final choice of HOG parameters.

After experimenting with various color spaces, I selected YCrCb which seemed better with respect to the HOG features, especially in channel 0 and 1. I experimented with orientations 9,11 and 13, pixels_per_cell of 8, 16, 32.  YCrCb, Orient 9 and 8 pixels per cell  gave the most compact and distinguishable parameters to eliminate/minimize false +ves.

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

Linear SVM with defaults was used after experimenting with Rbf kernel and various C values (1-10). Even though Rbf kernel was consistently better in scoring higher on the test set, Rbf was taking lot longer to train and the resulting Video performance w.r.t false +ves and -ves was not qualitatively better. Code cell #7 has the training logic.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?


Sliding window search is implemented using find_cars() function from the lessons. One cell was used for each step. The core frame processing function frame_proc() calls sliding window at different scales. For cars farther on the horizon, smaller scales are used and the search area was restricted to a smaller y-range. As the cars are searched at closer to the camera, the scale is increased. Code cell#8 shows detection of cars in the same frame but at different scales. As shown, at higher scales, cars farther are not reliably detected. 

![alt text][image3]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on 3 scales (1.5, 2 and 3) using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a better result.  

Here are some example test images:

(include a set of images)
![alt text][image4]
![alt text][image5]
![alt text][image6]
![alt text][image7]
![alt text][image8]
![alt text][image9]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video_output.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

In order to minimize/eliminate false +ves, I used the heat map approach from the lessons. All code is implemented in frame_proc() routine. I used last N frames to look for cars. Each frame is looked at 3 or 4 different scales and bounding boxes are found at each scale. 
I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

Here's an example result showing the heatmap from a series of frames of video, and the bounding boxes then overlaid on them:

### Here are a few frames with final bounding boxes overlaid and their corresponding heatmaps:

![alt text][image10]
![alt text][image11]
![alt text][image12]
![alt text][image13]
![alt text][image14]
![alt text][image15]


---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

It was a challenge to reduce the false +ves. Most of the false +ves are in the shadows, guard rails or lanes. It is possible these are under-represented in the training dataset. If 'predict' is more reliable, it is possible to eliminate false +ves at the prediction time itself, allowing to go more aggressive on thresholding on the heatmap. I.e we can use higher threshholds and not miss real cars. Otherwise, having higher thresholds may miss to detect cars that are properly detected.

More intelligent scale values are another area where detection can be enhanced and reduce false +ves. For example, depending on the area being looked at, scale factor can be automatically adjusted and limit scan to one pass. 

