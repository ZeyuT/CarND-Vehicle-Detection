# CarND-Vehicle-Detection # 
Self-Driving Car Engineer Nanodegree Program

---

## Introduction ##

This is my solution to the Vehicle Detection and Tracking Project of the Udacity Self-Driving Car Engineer Nanodegree Program. The goal for the project is to write a software pipeline to identify vehicles in a video from a front-facing camera on a car.

YCrCb color space, combined features of color histogram, binned color features and HOGs are used to build a feature vector in my project. The chosen classifier is Linear SVC, which is built by using sklearn tool. A sliding window method is used to search vehicles in each frame or test image.

[Here](./examples/test4.jpg) is an example of output images. All final output images can be seen in the folder "./output_images".

[Here](./test_videos_output/project_video_output.mp4) is the final output video.

[Here](./writeup.pdf) is a detailed writeup report.

## Implementation ##

[image1]: ./examples/rawdataset.JPG "data set example"
[image2]: ./examples/testing_result.png "testing result"
[image3]: ./examples/all_windows.jpg "all window"
[image4]: ./examples/all_windows_positive_detections.png "All windows positive detections"
[image5]: ./examples/heatmap.jpg "Heat map"

### Training Data ###

Labeled data set is given by Udacity. Here are links to the labeled data for [vehicle](https://s3.amazonaws.com/udacity-sdc/Vehicle_Tracking/vehicles.zip) and [non-vehicle](https://s3.amazonaws.com/udacity-sdc/Vehicle_Tracking/non-vehicles.zip) examples.  These example images come from a combination of the [GTI vehicle image database](http://www.gti.ssr.upm.es/data/Vehicle_database.html), the [KITTI vision benchmark suite](http://www.cvlibs.net/datasets/kitti/), and examples extracted from the project video itself.

Here is an example of image with cars and an example of image without cars.

![data set example][image1]

### Prepare Feature Vectors ###

#### Extract Features ####

Based on multiple trials, I choose to use YCrCb color space, and combined features of color histogram, binned color features and HOGs. Here are details of these features.

| Feature Names              | Values   |
|--------------------------- | ---------|
| Color space                | YCrCb    |
| Number of HOG orientations | 9        | 
| Channels of HOG            | All      |
| HOG pixels per cell        | 8        |
| HOG cells per block        | 2        |
| Spatial binning dimensions | 32 * 32  |
| Number of histogram bins   | 16       |

When tuning parameters, my principle is to find values of parameters so as to make the classifier works well on all testing images in folder ‘test_images’ while to minimize the computing time.
For each image, extracted features are stored in a feature vector.

#### Preprocessing on Feature Vectors ####

All feature vectors, along with their labels are randomly divided into training set and testing set.
After that, all feature vectors in training set are normalized to 0 mean and small variance, and feature vectors in testing set are scaled with the same scaler factor.

### Train a Classifier ###

I choose to use Linear SVC as the classifier. Because the differences between features vectors of images with cars and images without cars, and the data set looks not big.
I feed all shuffled feature vectors of training set in to SVC, and get fitted model. Then I test the model on testing set. Here is the testing result.

![testing_result][image2]

### Build Pipeline ###

#### Set Sliding Window Search ####

To set sliding window search, I define several functions in the first cell of my codes.
`slide_window()` function is used to generate lists of windows positions, based on input images and other settings about windows.
`draw_boxes()` function is used to draw bounding boxes on input images according to input windows positions
`single_img_features()` is used to define a function to extract features from a single image window.
`search_windows()` is to search all windows and get windows with positive positions. The parameters about extracting features are the same as above. The function is only used to visualize the process by generating images with all windows with positive detections. 

However, extracting HOG in each window require pretty much computing effect. To reduce computing effect, I define another function `find_cars()`. The function is used to get final image with hot windows on it. The function compute HOG only once in every input image. And each window just extract subarray from the HOG feature array.

By calling functions above several times with different window sizes, multiple scales of windows can be used to search on one image.
Here are all important parameters.

| Window sizes, in pixels                | 128, 96, 64 |
| Overlapping                            | 75%         | 
| Range of searching area on y direction | (400, 656)  |

Here are some examples of images during sliding windows. More images during the process can be found in folder "./test_images_output".

All sliding windows:

![All windows][image3]

All windows with positive detections:

![All windows_positive_detections][image4]

#### Heat Map and Thresholding ####

Based on hot windows, I add 1 heat to each hot window area on a blank image. Then a threshold is used to remove false positives. After that, a heat map, along with labels on each heat area, can be obtained. Finally, I draw boxes that can cover each heat area, which are considered to be positions of detected vehicles.
When dealing with videos, to filter out transient false positives, all heats are stored during the last 5 frame, and a threshold is used to the sum heat.
What is more, to filter out false positives with high heat values in small area, the minimum size of output boxes is limited 
Here are the thresholds. All thresholds are proved to be working well on test images and the given videos.

| Threshold on single image              | 1       |
| Threshold on videos                    | 7       | 
| Minimum size of boxes                  | 30 * 30 |

Here is an example of heat map. More images during the process can be found in folder "./test_images_output".

![Heat Map][image5]

Finally, according to these heat map images, the min/max x and y coordinates of positive pixels are used to build boundary boxes for those detected vehicles.
