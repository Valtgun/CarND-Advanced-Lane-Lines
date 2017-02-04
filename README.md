**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to the center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/undistorted_calibration1.jpg "Undistorted Calibration Image"
[image2]: ./test_images/test2.jpg "Original"
[image3]: ./output_images/02undistorted_test.jpg "Undistorted"
[image4]: ./output_images/03sobel_magnitude_test.jpg "Sobel magnitude mask"
[image5]: ./output_images/04hls_s_test.jpg "HLS S mask"
[image6]: ./output_images/05white_yuv_test.jpg "White YUV mask"
[image7]: ./output_images/06combined_mask_test.jpg "Combined mask"

[image10]: ./output_images/09warped_mask_test.jpg "Perspective warp mask"


[image14]: ./output_images/14mid_lines_test.jpg "Initial detection at max Y"
[image15]: ./output_images/15points_test.jpg "Detected points"
[image16]: ./output_images/16polylines_test.jpg "Polyline fit to the ponts"

[imagep1]: ./output_images/p0persp_transf_orig_straight1.jpg "Straight Lines Original transformation rectangle"
[imagep2]: ./output_images/p1persp_transf_warped_straight1.jpg "Straight Lines Warped transformation rectangle"
[imagep3]: ./output_images/p0persp_transf_orig_test.jpg "Original transformation rectangle"
[imagep4]: ./output_images/p1persp_transf_warped_test.jpg "Warped transformation rectangle"

[image20]: ./output_images/20plotted_test.jpg "Results plotted on warped"
[image21]: ./output_images/21plotted_unwarp_test.jpg "Results un-warped"
[image22]: ./output_images/22result_test.jpg "Results overlaid on undistorted image"
[image23]: ./output_images/23result_texttest.jpg "Final result"

###Further there are each of the points in rubric described:
###Camera Calibration
A function that performs camera calibration is defined in notebook cell #2.
It tries to read the saved values from pickle file calibration.pkl
If file loading fails with an exception then it recalculates the calibration matrix and saves it to the pickle file. It was implemented to reduce processing times.

Calibration calculation initializes the grid for 10x7 grid point detection.
Object points are fixed points for all images representing the x,y grid on a plane, where z coordinate = 0.
Image points are read from each of the images in camera_cal folder and coordinates are found using cv2 function findChessboardCorners() using the grayscale image.
If the image points are found then they are added to the imgpoints array and also Object points are added to objpoints array.

This successfully processes 17 out of 20 images, the ones with a different number of points are rejected in this implementation.

Objpoints and Imgpoints arrays are processed by cv2 function calibrateCamera() that returns distortion matrices including the mtx and dist, that are saved to the pickle file and used in the pipeline.

Example of undistorted calibration image:
![alt text][image1]


###Pipeline (single images)
Note: Processing pipeline is performed by function process_frame(image, debug, name), that as input takes one RGB image.
It is possible to pass additional parameter debug, that has a default value of 'None'.
There can be 2 additional values set:
'Save' - it will output the pipeline steps to 'output_images' folder with the prefix filename passed in name parameter
'Debug1' or 'Debug2' - it will output the result as 1920x1080 image that will consist of multiple lower resolution frames, that each will show one of the pipeline steps.

####1. Provide an example of a distortion-corrected image.
Each image (or frame) is passed to function undistort(), that takes input parameters of image and distortion matrices:mtx and dist.
mtx and dist are calculated once in the beginning from the camera calibration step.
undistort() function uses cv2.undistort().

test_images/test1.jpg is processed by undistort and saved in output_images/undistorted_test1.jpg:
Orignal
![alt text][image2]
Undistorted
![alt text][image3]

####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

After the undistorted image is obtained then the various binary masks are calculated.
In the submission pipelines following 3 masks are used:
Sobel gradient magnitude (np.sqrt(sobelx* *2 + sobely* *2))
S channel from HLS (yellow and white)
White from YUV colorspace - in addition to above

Code for the image thresholding functions is in notebook cell 3.

During testing there were multiple additional binary masks calculated, e.g.:
Sobel in one direction X or Y
Grayscale - initial, used to test pipeline failover to full scan when there are many bad detections
Channels from RGB
Etc., however, they did not provide significant improvements. Also, the more different calculation are done, the individual frame processing performance becomes, which could be a significant factor if the processing should be performed real time.

It was also tested in perspective warping should be performed before or after thresholding, however, especially in a case of Sobel, it turned out that it is better to threshold original image and only then perform perspective warping on the binary image.
If perspective warping was performed before Sobel, then there were many artifacts introduced that did not allow good quality Sobel result.


Additional point is that various parameters can be tuned at this step in order to get the result better and what works in one video, might not transfer very well to other videos, especially the hardest challenge where there is a different environment.

Here are the images:
Sobel magnitude mask
![alt text][image4]
HLS S mask
![alt text][image5]
White YUV mask
![alt text][image6]
Combined mask
![alt text][image7]


####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

Code for the perspective transformation functions is in notebook cell 4.
There are two functions defined, one for calculating transformation and inverse transformation matrix and the second function for actually applying perspective warp by passing image and transformation matrix.

The transformation matrix is calculated by hardcoded values, finding out correct values turned out more complex than initially estimated.
It is not as simple as to take straight road segment and identify the points. It should be adjusted how long along a vertical (Y) axis the points are chosen and how much to offset lines horizontally from borders in order to get good enough results and still remain with consistent transformation on curved roads.

In the final solution values from Udacity classroom is used, as they did very well in order to avoid big transformation errors on curved road segments.

The difficulties are there because, even if we take points on an image with straight road, then there are additional variables that impact accuracy:
a) Car might not be centered within lane
b) Camera might not be centered in car
c) Camera angle (axis) might not be accurately aligned with car axis
d) Car is not driving parallel to the straight road lanes
In real life, it would be much better to calibrate camera by physically marking (measuring) the points, e.g. in parking spot, so that car and camera is centered and aligned with the measured axis. That way the small errors could be eliminated.

Here are the points that are used for the transformation drawn on the undistorted original and warped sample for the image of Straight lines.
![alt text][imagep1]
![alt text][imagep2]

And for the Test image:
![alt text][imagep3]
![alt text][imagep4]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

As an input the binary mask that is obtained by using the perspective transformation warping function above:
![alt text][image10]

There are two distinct functions for detection, one is for full frame and other is for partial frame based on previously fit polynomial.
Which detection is run is determined by state class.

####4.1 Full detection:
Code is within cells 5&6.

Full detection is run on initial frame or if it was not successfully detected in 10 sequential frames.
Full detection is always added to the detected line array.

Full detection is done by initially splitting frame to two at the middle point of the image. Then for each half, the maximum histogram points are detected, by checking histogram and reverse histogram indexes (described below) for frame between vertical point 480 and maximum. This results in peak point over full frame.
![alt text][image14]
This peak is used to narrow down search also for full frame detection and it checks 80 pixels around that peak in order to determine the line points. After each vertical loop, it adjusts the peak position to detected one previously, that allows to quickly recognize main lines and also follow then if they are curved.

####4.2 Partial detection:
Code is within cells 7&8.

Partial detection is done when there are previously fit polynomial line.
Partial detection is similar to the full detection, but it checks area only 40 pixels to both sides from the previous polynomial line.
Within that frame it checks histogram and reverse histogram (described below) and outputs detected points.

Successful partial detection is when both lines have at least some points detected (in order to fit polynomial) and also if detected polynomial passes test to determine if it was not too far off from the previously detected line average.

Polynomial function that is used is: ax^2+bx+c
Delta check is done based on b value of the polynomial function because it determines the main curvature of the line. If the difference between averaged polynomial fit over previous 7 frames and latest detected line is over the threshold then the line is rejected and marked as not detected.


####4.3 Both detection functions:
######Function for maximum point identification
Initially function "signal.find_peaks_cwt" was used, however, its performance was very slow, therefore some other functions was tested. In the end, final decision was to use histogram function and numpy argmax, that returns index of first maximum appearance.

#####Histogram and reverse histogram
Using histogram and np.argmax, run into cases when the lines shifted to the left (lower) indexes. This was because argmax returns first index and there might be case when there are multiple values at the maximum value.
For example, in the array:
0,0,1,2,5,5,5,5,2,1
Argmax would output index 4 (as it is first element with maximum value of 5)
In order to get more accurate result, the histogram was reversed and argmax was taken from the reverse histogram, in this case, it would be a value of 2, which leads to index 9-2=7.
The maximum is then averaged rounded down 4+7=11/2=index position of 5.

That creates special case when histogram contains just 0 values that middle value is returned, but this was covered by performing additional check and return NaN value if this is the case.

#####Vertical frame size
The height of 20 pixels is checked each time, this gave reasonable detections (lower false positives from occasional points) but kept the number of points high for the polynomial fit functions.

####4.4 Polynomial fit:
From the detected points:
![alt text][image15]

Pipeline passes detected points to the numpy function np.polyfit(), that calculates 2nd order polynomial.
![alt text][image16]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code for this is in Notebook cell 1, class Lane and function add().

It follows the class instructions of calculating pixel to meter conversion.
I take the averaged values over the last multiple successful frames and values are calculated from this averaged polyfit.
Curvature that is output to video is average between left and right lanes, in order to have fewer output parameters.
In addition to that, if curvature is too large (>4000m), then text "Curvature: straight" is displayed instead, as this is more meaningful value at such large curvatures.

Off center position is calculated by difference (in meters) between middle pixel between left and right polyfit position at maximum Y coordinate and image center position.
The output is provided in Centimeters without decimal places.


####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The code for drawing results is in function process_frame() under comment "Drawing results on road image".

Here are the images, after drawing results on warped image:
![alt text][image20]
After reverse perspective transform:
![alt text][image21]
And overlaid on original unwarped image:
![alt text][image22]

And there is the final image with curvature and off-center position:
![alt text][image23]

Note: Single image always runs in full detection mode as there is no previous information for polyline fit. 

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).
Here's a link to the final video:
[Resulting video](https://youtu.be/Pxtwv8-MIaA)

And two debugging videos:
First, for undistortion and masking:
[Debug video 1](https://youtu.be/VSyayLu-HUs)

Second, for detection and shows perspective warping:
[Debug video 2](https://youtu.be/zNKgCI8_7Ng)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?
#####Challenges
As mentioned above, it was not straightforward to detect the points for perspective transformation. If there would be access to physical car, then additional measurements or images could be taken and more accurate points can be determined.

#####Performance considerations
Some of the functions that could be used (e.g. signal peak detection or more complex transformations) are quite computationally heavy therefore led to longer processing times. While it might not be an issue in this learning phase, I did avoid very slow performing solutions, because those would not perform in real time. This is also a question how many masks and what type of masks to use because although there could be implemented many masks and their combinations which would give better quality result, it might be too slow for real-time applications.
A better approach would be to combine more inputs, for example, multiple camera images and then run slimmer pipeline on their combined result.
For example, if we check cameras on new Mercedes EClass, there are stereo windshield camera and front camera, combining and transforming those 3 images, probably could get better results and avoid issues with windshield glare, etc.

#####Personal skill challenge
For this slightly more complex project, I understood that additional Python skills would be beneficial, especially how to better organize code. Of course, it is possible to make the working code, but it is a long way to get it clear and well organized.

#####Findings
It was quite easy to run the initial detections even with grayscale image, that I used to get the pipeline done and make better recovery. Adding additional masks improved detection on harder environment, however, there is still long way to improve on the hardest challenge.
In order to get better masking results, it would be necessary to test which masks work better on the undistorted images and which ones work well better on the images after perspective transformation.

#####Approach for complex videos
There could be multiple additional approaches implemented for more complex videos, for example:
-Improving detection by offsetting left and right side of image, so that initial points (near the car) matches. This could improve pixel magnitude and work better where one line is dashed. As the curvature should be equal on the birds-eye-view, then they should overlap. 
-Detection of lines by interval. In hard to detect locations and environment, if we know the distance between the lines, then with quite high probability if only one line is detected then the second one can be calculated. However, this can not be primary means of detection as there might be line merges or other nonstandart line widths.

#####When it would fail
It would fail on more complex environments, e.g. hardest challenge with varying lights and shadows. Also it would not work where there are not so obvious line markings or there are specific road markings. For example where the lines are merging, some signs on the road, potentially also pedestrian crossings, etc.
