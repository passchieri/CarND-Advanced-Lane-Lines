# Advanced lane finding
**Author: Igor Passchier**

**igor.passchier@tassinternational.com**

## Introduction

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

## Provided results
The following documents are provided as part of this project report:
* **Writeup**: This document
* **Example images**: Can be found in the [output](output) subdirectory
* **Result video**: Can also be found in the [output](output) subdirectory
* **Code**: The code is stored in the jupyter notebook [CarND-Advanced-Lanes%20v2.ipynb](CarND-Advanced-Lanes%20v2.ipynb). The structure of the notebook is in line with the Rubric points, and this writeup. Code blocks will be referred based on sections in the notebook. A [pdf](CarND-Advanced-Lanes%20v2.pdf) of this notebook is also provided.

[//]: # (Image References)

[calibration1]: ./output/calibration1.png "Calibration 1"
[calibration2]: ./output/calibration2.png "Calibration 2"

[filtergray]: ./output/filtergray.png "Gray filter"
[filterr]: ./output/filterr.png "Red filter"
[filterl]: ./output/filterl.png "Lightness filter"
[filtersobel]: ./output/filtersobel.png "Sobel x filter"
[filterfinal]: ./output/filterfinal.png "Final filter"

[perspective1]: ./output/perspective1.png "Perspective tranform"
[perspective2]: ./output/perspective2.png "Perspective tranform"

[fit1]: ./output/fit1.png "Curvature fit"
[fit2]: ./output/fit2.png "Curvature fit"
[final]: ./output/final.png "Final result"

[result1]: ./output/result1.png "Final result"
[result2]: ./output/result2.png "Final result"
[result3]: ./output/result3.png "Final result"






## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points


### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

Its this document.

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The camera calibration is implemented in the 2 code blocks in the *Compute the camera calibration matrix and 
distortion coefficients* section. The first section is to get a feeling on how 
<code>cv2.findChessboardCorners</code> is working. In the second section, all calibration images are processed to 
determine all chessboard corners. The actual camera calibration is done with <code>cv2.calibrateCamera</code>.
The results are stored in a global variable with the calibration matrix. Below one of the calibration
images before and after distortion correction is shown. 

![Calibration 1][calibration1]

** Figure 1: Calibration image before and after distortion correction **

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The correction calculated in the previous step, is applied to one of the test images.

![Calibration 2][calibration2]

** Figure 2: Test image before and after distortion correction **

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I have implemented an extensive set of filters, and tuned them all to detect the lane pixels as good as possible. 
The code is provided in the code block in the *Create a threshold binary image* section. For the tuning, I have 
also created an additional test image. This image is selected to have different shades of concrete, and also 
shadows of the trees. This extra image is shown later, together with the final filter.

The filters I have tuned, operate on the following layers:
* Red, Green, and Blue
* Gray
* Hue, Saturation, Lightness
* Sobel x, Sobel y, Sobel absolute, Sobel gradient

For these 4 sets of filters, a optimized combinations per set, to get the best result. Per filter per set, I 
optimized to make sure not to lose too many valid pixels, to the expense of have more invalid pixels selected. 
I ended up using the following filters per set:
* rgb: only a filter on red
* gray: kept as is
* hsl: An <code>and</code> of (wide) filters on h, s, and l
* sobel: only sobel x is kept

![Red Filter][filterr]

** Figure 3: Red filter **

![Gray Filter][filtergray]

** Figure 4: Gray filter **

![Lightness Filter][filterl]

** Figure 5: Lightness filter **

![Sobel x Filter][filtersobel]

** Figure 6: Sobel x filter **

As a second step, I combine these 4 resulting filters, where I take an <code>and</code> of rgb, gray, and hsl, 
and then an <code>or</code> with sobel. The first 3 filters all select the actual lane pixels directly, so taking 
an <code>and</code> requires the pixels to be identified as a lane pixel 3 times. The sobel filter actually 
filters the edges, and not the lane pixels itself. Therefore, if I would include this in the <code>and</code>, 
that would throw away a lot of the pixels of the lanes. By taking an <or>, I combine the dtection of the actual 
pixels, with the detection of the border of the lanes: Best of both worlds. Or, in code:

```python
    filter=((greybin>0) & (rgbbin>0) & (hslbin>0)) | (sobelbin>0)
```

![Final combined filter][filterfinal]

** Figure 6: Final, combined filter **

In a later stage, to speed up the calculation, I have commented out all the filters that are not actually used. 
These lines have been commented out by 2 hashes (##)

The final filter is implemented in the function <code>filter</code>, starting at line 39.

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The perspective transform is implemented in the section *Birds-eye view*. I have implemented 2 function. The first, starting at line 3, calculates the tranformation matrix and the inverse transformation matrix. I have used straitght_lines1.jpg image for the calibration. In this image, I have select 4 points on the lanes for the transformation: 
```
    [266,675],[1038,675],[655,433],[619,433]
```
and tranformed that to a destination rectangle, defined by
```
    [300,719]-[900,10]
```

The actual perspective transform is implemented in the function <code>warp</code>, starting at line 32. Below
are 2 examples of transformed images. The first is the image used to determine the perspective transform, 
including overlayed green lines indicating the area used for the transformation, and a binary picture of straight 
lines.

![Perspective transform][perspective1]

** Figure 7: Result of the perspective transform. The green line specifies the area used to determine the transform. **

![Perspective transform][perspective2]

** Figure 8: Straight line binary image perspective transform result. **

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code to detect the lane lines is provided in the section *Detect lane pixels and fit to find the lane 
boundary*. I have implemented 2 helper function, to calculate sliding window boxes (within limits of the shape), 
at line 1. The second helper function is to extract all relevant pixels within a window, starting at line 14. The 
actual algoritm is implemented in <code>detect_lines</code>, starting at line 25.

The rationale of the algoritm is as follows;
1. Initially, no line fits are available, and a histgram is made of the lower 1/3 of the image. The peaks on left and right half are used as starting point. Only 1/3 is used, because that gives better results than with 1/2, if the lanes are curved.
2. Then, 17 windows from bottom to top are created. The center is taken to be the average pixels of the previous window with sufficient pixels.
3. As an optimization, if sufficent pixels are found, the process is repeated with a smaller window, around the average of the points in the current window (in contrast, in step 2 the average of the previous window is used). This allows to use a smaller window, even when the lanes are stongly curved.
4. If a valid fit is already available from a previous step, then step 1, 2 and 3 are replaced by selecting the window based on the result of the fit of the previous step. In this case, the smaller window size is used, similar as in step 3. Also the number of windows is increased by a factor 10, to have effectively 1 large selection around the fit of the line.
5. All pixels that fall within the windows are collected, and stored in 2 sets of pixels, that later can be used to fit the line

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code for fitting is in section *Determine the curvature of the lane and vehicle position with respect to center*. First, I have defined a helper function (line 4, <code>fit_curve</code> that can fit a second order polynomial through a set of x and y points. The code expects points in pixels, but can be given scale factors to calculate the fit in units of m. By putting the scales to 1, the fit is done in pixel coordinates.

I am using a weight function to take into account that the points at the bottom of the image are more reliable than the points at the top. Furthermore, due to the perspective transform, the number of pixels detected at the top tend to increase somewhat, which would result in taking the top pixels (so furthest away) too much into account.

I have experimented with several weight functions, but at the end decided to sue the y pixel coordinate as weight. In practise, this means that the points closest by get a weight approx 2-3 times higher than the points furthest away.

I have used the width of a lane (3.7m) and length of a line segment (3 m) to calibrate the x and y conversion from pixels to meters by measuring those distances in one of the sample images. I found about 600 pixels wide, and 50 pixels long, so resultsing in a scale of 3.7 m/600pix and 3m/50pix. These numbers are the default in the fit_curve function for the scales.

The result can be seen in the image below

![fit][fit1]

In the next code block, I have implemented the function <code>curve_and_center</code> that can calculate the curvature for the left and right line at an arbitrary place, and also calculates the offset from the center. It works both in pixel coordinates, and in m coordinates. The center is just taken to be the middle of the left and right line.

In the actual pipeline, I use the lowest point of the warped images (closest by the car) to calculate the offset and curvature.

For the image above, the results are:

```
    curvatures (left and right, 1/pix): 608, 463
    off center (pix): 2
    curvatures (left and right 1/m): 246, 205
    off center (m): 0.02 m
```

The curvature in m seems rather strong, so probably the conversion 
from pixels to m is not completely accurate. I could have adjusted 
the yscale (the xscale is correct) until the fits are more realistic, 
but as that is rather arbitrary, I have not done that. A more 
reliable way is to use some dedicated calibration images, with a 
clear ruler in the image, so that it can be measured accurately 
(similar as was done to determine the lens corrections).



#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Warping back is implemented in the section *Warp back to original 
image*. The filter and pixel detection functions already return 
images identifying the relavant pixels. The fitting function overlays 
the pixels for the fit on top of these. In the method here, I 
calculate 20 points of the fit (line 16-20), and create a polygon on 
top of the overlay. This gives me an image with the detected pixels, 
selected pixels, fit lines and lane overay.

![fit][fit2]

I warp this image back (line 37), and overlay it with the original image. Finally, I print the (average) curvature and offset from center on the image. See result below.

![final][final]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

The pipeline code is presented in the *Complete pipeline* section. I have 2 functions. the first, init_pipeline, initializes all global variables, which can not be prevented due to the way the pipeline is used.
The second function, pipeline, is the actual pipeline. It takes the following steps:

1. image distortion correction (line 18-19)
1. filter image (line 20)
2. warp the filtered image (line 22)
3. detect the line pixels (24)
4. fit the lines (26-40). I do this both in pixels (for plotting) and 1/m (to determine the actual curvature
5. unwarp the result (41)
6. Store the results for plotting afterwards (44-56)
7. Determine whether the fit is reliable. If not, reset the pipeline to go back to the histogram based line pixel detections (50-52)

In the section *Process all test images* I have used the complete pipeline to see whether any of those had serious issues. In the second code block in this section, I made a function to plot histograms of the result of a full video processing. The resulting pictures are shown below:

![result1][result1]
![result2][result2]
![result3][result3]

It shows that the radius of curvature for the left and right lane are rather similar everywhere (as expected), the car stays within 25 cm from the center (sounds realistic) and that the fit to the line is never lost (reset=0 everywhere)

Here's a [link to my video result](output/result.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The most effort went into tuning the filters: how strict to make 
them, and combine them in an "AND" or "OR", etc. I played around with 
making the individual filters very strict ("never" have false 
positives), but then combine all of them with "OR", or make them 
loose ("never" have a false negative), and combine them with "AND". I 
anded up mixing the two approaches.

After I finished the full pipeline, the performance was very low (2 s 
per frame), which made tuning on the complete vidoes also very slow. 
This was caused by the plotting algorithm I used: I initially used 
PIL, and not CV2 for plotting. PIL has a bit more advanced image 
overlay, resulting in nicer pictures, but the price to pay in the 
performance was unacceptable. It was actually not so much PIL itself, 
but going back and forth between PIL images and np arrays, that are 
used in the rest of the code.

Finally, also the part of the algoritm that determines whether the 
fitted lines are reliable, is not so strong and rather simplistic. 
The performance metrics of the fit itself should be taken into 
account, and more strict rules could be applied to determine whether 
they are realistic. This is not necessary for the project video (the 
pipeline is strong enough to never make a mistake in this video), but 
with the challenge video it fails without being detected.

Something that also goes wrong with the challenge video, is the 
detection of the repair lines. These are picked up by the sobel 
filter. This might be prevented by first bluring the image, as the 
repairs are much smaller than the actual lines. The disadvantage 
would be that in the far distance sobel would also not be able to 
detect the lines anymore. It would take careful tuning the get this 
right.

