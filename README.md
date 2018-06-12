# **Finding Lane Lines on the Road** 

## Writeup

---

**Finding Lane Lines on the Road**

This project, we use computer vision techniques in order to detect lane lines on the road. 
This file explains the pipeline, and reflects on its different aspects and how they could be improved

[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"
[hsl_img]: ./img_for_writeup/hsl-hsv.png "HSL colors"
[color_masked1]: ./test_images_debug/color_mask_solidWhiteCurve.jpg "color masked image #1"
[blurred1]: ./test_images_debug/blurred_solidWhiteCurve.jpg "blurred image #1"
[RoI_masked1]: ./test_images_debug/RoI_masked_solidWhiteCurve.jpg "RoI masked image #1"
[Canny1]: ./test_images_debug/canny_solidWhiteCurve.jpg "Canny image #1"
[SegmentsOnImage1]: ./test_images_debug/segments_on_image_solidWhiteCurve.jpg "Segment on image #1"
[FinalOutput1]: ./test_images_out/solidWhiteCurve.jpg "Output image #1"


---

### Reflection

### 1. Description of the pipeline

My pipeline consisted of 5 steps: 

**A) Keep only yellow and white pixels **

I first perform a color filter based on HSL (Hue-Saturation-Lightness) representation, and mask the image to keep only white and yellow pixels. This representation has been chosen after [reading this article/code](https://medium.com/towards-data-science/finding-lane-lines-on-the-road-30cf016a1165) 
![alt text][hsl_img]

An example of results can be seen below:
![][color_masked1]

**B) Convert to grayscale and apply a Gaussian smoothing **

Then, I convert the image to grayscale, and apply a Gaussian blur to it. This has the effect of removing weak edges. See the result (on the same example) below:
![][blurred1]

**C) Apply Region-of-Interest mask **

Then, we remove all the pixels that are not within a Region-of-Interest, defined by 4 points. See the result (on the same example) below:
![][RoI_masked1]

**D) (Optional) Apply Canny edge detection **

Although I found it did not really improve the final results, we can then apply a Canny Edge detector. See the result (on the same example) below:
![][Canny1]

**E) Detect lines with Hough Transform **

At this stage, the lines are already pretty well isolated. We now can apply the Hough Transform algorithm in order to detect the road lane segments. We can now view the first stage of the pipeline, which is supposed to detect line segments:
![][SegmentsOnImage1]


**F) Extrapolate the final lines **

At this stage, we have several lines (usually more than 2), and we hope all of them belong to some segment of the left or right lane line. The goal now is to extract 2 more robust final lane lines, and extrapolate them so that they don't just cover the lane segments. In order to do that, I:
1. Separate lines in 2 groups (left/right) according to the sign of their slope
2. For each group, compute the median of the slope and intercept of all lines, to "fuse" all lines into a single one. I chose to use a median (instead of an average, for example) because it is more robust to outliers. At this stage, we already have good lines.
3. However, in order to further increase precision and robustness, I do a second pass on all the lines and remove the lines that have a slope too far from the median slope.
4. Finally, I perform an average on these "good" lines. This final trick helped me remove out several isolated case of failure

The final output of the pipeline (still on the same example is given below):
![][FinalOutput1]


### 2. Shortcomings of the current pipeline

I see 2 main shortcomings in the current pipeline:
1. There is no confidence score.
2. The lanes lines estimation is unstable in the video (which creates a very unpleasant jitter effect)
3. The pipeline is sometimes failing and gives completely wrong lines.


### 3. Possible improvements of the pipeline

** Confidence score ** 

From my experience, an algorithm is hardly usable in practice if it doesn't include some sort of confidence, and the algorithm currently doesn't include one. Here are for example a few sanity checks that would enable us to indicate how confident we are:
1. Compute the intersection point between the 2 lines, and check that it is on a "reasonable zone" (typically the center of the image). 
2. Compute the angle between the 2 lines and check that it corresponds to the angle you would expect (I would say between 40 and 80 degrees)


** Temporal processing ** 

Especially once a confidence score is available, it could make a lot of sense to add some time filtering in order to increase robustness. Typically, in order to solve shortcomings 2 and 3, I would add a Kalman Filter that uses the confidence scores mentioned above. Indeed, if at frame n, the left lane line is (a,b), then we expect the left line of n+1 to be very close to (a,b) too. Such filter should both remove the jitter effect, and enable to detect and handle cases of failure (when the confidence is too low, just take the last frame)

