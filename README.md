
# Vehicle Detection and Tracking Project

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[car-hog]: ./output_images/Car-HOGViz.png
[car-hog1]: ./output_images/Car-HOGViz-1.png
[notcar-hog]: ./output_images/NotCar-HOGViz.png
[notcar-hog1]: ./output_images/NotCar-HOGViz-1.png
[heatmap]: ./output_images/heatmap.png
[heatmap0a]: ./output_images/heatmap0a.png
[heatmap0b]: ./output_images/heatmap0b.png
[heatmap1]: ./output_images/heatmap1.png
[heatmap2]: ./output_images/heatmap2.png
[heatmap3]: ./output_images/heatmap3.png
[video-screenshot-test]: ./output_images/video-screenshot-test.png
[image1]: ./examples/car_not_car.png
[image2]: ./examples/HOG_example.jpg
[image3]: ./examples/sliding_windows.jpg
[image4]: ./examples/sliding_window.jpg
[image5]: ./examples/bboxes_and_heat.png
[image6]: ./examples/labels_map.png
[image7]: ./examples/output_bboxes.png
[video1]: ./project_output.mp4
[video2]: ./test_output.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](README.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The HOG Features were extracted using the `hog` method in `skimage.feature` package.

The code for this in the `1. Lesson Functions` section of the [Vehicle-Detection-and-Tracking.ipynb](Vehicle-Detection-and-Tracking.ipynb) file.

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=8`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:

**Cars**

![alt text][car-hog]
![alt text][car-hog1]

**Not-Cars**

![alt text][notcar-hog]
![alt text][notcar-hog1]


#### 2. Explain how you settled on your final choice of HOG parameters.

I tried HOG search with various combinations of parameters and found that `YCrCb` color space worked fine.I use all the three channels as the feature vectors.

I use 8x8 pixels for each cell and 2x2 cells in each block. This gives a good balance of being computation heavy and extracting good feature set.


#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

HOG Feature Extraction and Training Linear SVM Classifier is implemented in  `2. Histogram of Oriented Gradients (HOG) Feature Extraction on a Labeled Training Set of Images and Training Linear SVM Classifier` section of the [Vehicle-Detection-and-Tracking.ipynb](Vehicle-Detection-and-Tracking.ipynb) file.

I used total three feature vectors. One is spatial binning to get the raw color info, second is using a histogram of the color spectrum to get color info, and third is the HOG features to get the shape info. I concatenate all the features to give the feature vector.

The total length of the feature vector comes to 6108. It takes 4.71 seconds to train the SVC model with a test accuracy of 0.9901

I trained a linear SVM using `LinearSVC()` from the `sklearn.svm` package

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

Sliding Window Search is implemented in `3. Implement a Sliding-Window Technique and use the Trained Classifier to search for cars in images` section of the [Vehicle-Detection-and-Tracking.ipynb](Vehicle-Detection-and-Tracking.ipynb) file.

The `find_car` function combines HOG feature extraction with a sliding window search, the HOG features are extracted for the entire image (or a selected portion of it) and then these full-image features are subsampled according to the size of the window and then fed to the classifier. The function performs the classifier prediction on the HOG features for each window region and returns a list of rectangle objects corresponding to the windows that generated a positive ("car") prediction.

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  To Show that my pipeline is working , please refer to [https://github.com/vdt/CarND-Vehicle-Detection-P5/blob/master/README.md#2-describe-how-and-identify-where-in-your-code-you-implemented-some-kind-of-filter-for-false-positives-and-some-method-for-combining-overlapping-bounding-boxes] 


---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_output.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

Filter for false positives and a Method for combining overlapping bounding boxes is implemented in `5. Estimate a Bounding Box for Cars detected` section of the [Vehicle-Detection-and-Tracking.ipynb](Vehicle-Detection-and-Tracking.ipynb) file.

I took six test images in the test_images folder.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I had imported `from scipy.ndimage.measurements import label` at the beginning of the notebook and used the `label` function to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

Here's an example result showing the heatmap from six test images, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the image :

### Here are six images and their corresponding heatmaps:

![alt text][heatmap]
![alt text][heatmap0a]
![alt text][heatmap0b]
![alt text][heatmap1]
![alt text][heatmap2]
![alt text][heatmap3]

### Here the resulting bounding boxes are drawn onto the last frame in the series in the video:

![alt text][video-screenshot-test]


---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

1. The size of the bounding box does not flow smoothly between frames

2. When two cars are overtaking, it treats both as the same car. track individual positions more accurately over time.

3. The data processing pipeline is slow at 3 frames per second . We have to work on it to make it realtime.
