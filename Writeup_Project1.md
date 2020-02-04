# **Finding Lane Lines on the Road** 

## By: Bahnisikha Dutta

### This writeup explains the lane finding project in details: delineating the steps in the pipeline, potential shortcomings of the pipeline and techniques that can further improve the pipeline. 

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

[image2]: ./test_images_output/gray/solidWhiteCurve.jpg "Grayscale"

[image3]: ./test_images_output/edge/solidWhiteCurve.jpg "Edge"

[image4]: ./test_images_output/edge_roi/solidWhiteCurve.jpg "Grayscale"

[image5]: ./test_images_output/solidWhiteCurve.jpg "Grayscale"

[image6]: ./test_images_output/concat/solidWhiteCurve.jpg "Grayscale"
---

### Reflection

### 1. Pipeline description.

My pipeline consisted of 6 steps: 
First, I converted the images to grayscale, then I applied a gaussian blur filter with kernel size 3. After that,I applied a Canny edge detection algorithm with the threshold ranges set between 100 and 200 and generated an edge image which is essentially a black canvas with the edges marked by white pixels.

While driving on a road, a lot of things can get classified as edges in addition to the lane lines, for eg: curbsides, car bumper, etc. For a successful lane following algorithm which eventually will be the goal for a self driving car, we are only concerned with the road lanes; in this project, more specifically the lanes right in front of us. So, in order to demarcate only those two lanes, I applied a polygon mask on the image which specifies the region of interest and finds out lane lines which falls within this region.

After I extract the lines only in the ROI, I overlay those on the original unprocessed image and test it on a batch of test images provided in the \test_images directory. I then, save the resultant output in test_images_output folder. Here is a step by step evolution of the images following each step above.

![alt text][image2]

![alt text][image3]

![alt text][image4]

![alt text][image5]

### 2. Modify Pipeline for concatanating lane lines.

There are several ways one can concatanate all the lanes lines to draw a single left and a right lane. I created concatanate_hough_lines function to do the same. Essentially, what this fuction does is that it pools the start and end points of the hough lines in our ROI as belonging to either left or right lane by sorting through whether the points are on the left or right of centerline of the image. Please note that this can also be done by sorting positive and negative slope as the left lane will have a positive slope whereas the right lane will have a negative slope. After the sorting is done, I calculate the slope and intercept of the left and right pooled coordinates using the polyfit function. I then, extrapolate the slope and intercept to give me the starting and ending x coordinates for both the lanes where I enforce the minimum and maximum y coordinates to be the minimum and maximum of my ROI y coordinates. I then, draw the concatanated line by feeding these starting and ending coordinates to opencv2.line. Here is an example image with concatanated lane lines. 

![alt text][image6]

### 2. Potential shortcomings with current pipeline

This pipeline is a very rudimentary technique for attaining a successful lane finding process. Most of the parameter was tuned manually and hence is highly sensitive to bias caused by the availability of the test images. Specifically, ascertaining the ROI is challenging as the actual ROI can be considered as a dynamically changing function which can change with road turns, car heading, etc.

Fixed edge detection and hough transform thresholds will not work for varying imaging conditions like different illumination. If there are cars right in front of you and if they are in the same color range as the lane lines, the pipeline in its current state will struggle to remove the noise lines caused by the car and this will effect the concatanation function.

Some of these problems are evident in the challenge exercise.


### 3. Suggest possible improvements to your pipeline

There are several ways one can improve this. One major improvement would be to color segment the images and post process for edge detection, extrapolation, etc on the colors of interest (in this case yellow and white). Please note that this won't entirely eliminate the environmental noise as there is always a finite possibility of there being a yellow or white car (say). 

The parameters of the system should not be manually tuned, but instead should be learnt through some training algorithms. 

Sharpening the images when the lanes are blurred can help as well. In a nutshell, there are several things one can try. But, some sort of machine learning training to make the algorithm smarter seems crucial to get it close to road readiness. 


### 3. Challenge exercise

The challenge videos had the following important differences from the other videos that warranted a change in the pipeline
1. The size of the image is larger; thus area of interest is different
2. The bonnet obscures the view and leads to false edges that can skew the extrapolation
3. The pavement changes at one point and generates false edges
4. The yellow left line is blurry when the pavement changes.

I made the following changes to attempt tracking the lane in this new video
1. Adjusted the ROI to take care of the larger image and the bonnet
2. Specified a range of slopes in my extrapolation function to eradicate the effect of erroneous edges caused by the change in road surface.

However, this still needs more work as it fails post 5s when the pavement changes. 