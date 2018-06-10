# **Finding Lane Lines on the Road** 

## Huy Nguyen

---

**Finding Lane Lines on the Road**

[//]: # (Image References)
[image_gray]: ./report/gray.jpg "Grayscale"
[image_canny]: ./report/canny.jpg "Canny Transform"
[image_roi]: ./report/roi.jpg "Region of interest"
[image_hough]: ./report/hough.jpg "Hough line transform"
[image_final]: ./report/final.jpg "Final line with initial image"


---\

### Reflection

### 1. The pipeline.

My pipeline consisted of 5 steps.


First, I converted the images to grayscale:
![alt text][image_gray]
Second, I applied Gaussian Noise kernel (with kernel size 5) to smooth out the edges.
Then, I transformed the images with Canny transformation to detect the edges. For Canny transformation,
I set the low threshold to 75 and high threshold to 75, this seems to give me the best detection of the
lane line:
![alt text][image_canny]
Then, I applied a image mask to only keep a region that I am intersted in.
I used a quadrilateral to crop out area near the intersection point at the top of the
lane lines where there can be other cars/objects that could add noise to my detection.
![alt text][image_roi]
Finally, I used Hough line transform to detect lines. The following image show the two lane lines detected via Hough line transforms:
![alt text][image_hough]
This is the combination of the detected line and the initial image:
![alt text][image_final]

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by partitioning the
lines into two sets based on the slope: a left lane set for negative slope lines and a right set for positive slope lines.
After the partition, I output 2 lines which are the average of all lines in each of the set.
I got an okay lane line detection. However, the detected lines are not very stable, there are a lot of jumping/sudden changes in the result video.
This is mosty due to noise from the line detections.

To mitigate this issue, I added a few more modification to stablize the detections:

* I only kept slope in ranges [-0.9, -0.5] (for left lane set) and [0.5, 0.9] (for right lane set). This helps filter out all
horizontal lines.

* I added a weight for each line which is equal to square of the line length, so that longer line will have more contribution
toward the final (average) line.

As a result, the videos are much more stable.

### 2. Identify potential shortcomings with your current pipeline

One potential shortcoming would be what would happen when the lane lines slope are out of the ranges that I hard-coded (i.e., [-0.9, -0.5] and [0.5, 0.9]).
This usually happens for curve lane when the slope can be smaller.

Another short coming could be due to the region of interest. For now, I set the region to be cut off at y=305. This value should depend on the position of the mounted camera.
For different position of the camera, we should pick different cut-off point.


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to instead of setting a fixed range to filter out the slope we want, we should use something like k-means to detect the slope for left and right lane lines, or some outlier detection algorithm to remove slope that are significantly different from most other slopes.

Another potential improvement could be to keep track of the lines detected in previous frames and combine that with the line detected in this frame (i.e., the final line should be a weighted
average of previous line and current line). This should help stablize the detected line.
