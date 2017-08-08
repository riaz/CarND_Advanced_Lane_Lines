## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/calibration.jpg "Undistorted Image"
[image2]: ./output_images/lane_undistorted.jpg "Undistorted Lane Image"
[image3]: ./output_images/thresholded.jpg "Thresh"
[image4]: ./output_images/perspective_transform.png "Perspective Warped Image"
[image5]: ./output_images/lane_fit.jpg "Fit Visual Image"
[image6]: ./output_images/advanced_lane_detect.jpg "Final Output Image"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in `./Advanced_Lane_Detection.ipynb`  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  We have (9x6) chessboard images for the calibration step.

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Similar to how we have undistorted the image of a chessboard which we have originally used for calibration, we will undistort the test images provided with the project.

Here we make an assumption that, a single camera was used to take the pics of the chess-board from different directions in object space and also for capturing the lane image as we see in the `test_images` folder.

The [5]th cell in the IPython notebook has the code , where I undistort every test_image. Here is one such image.

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

In order to create a thresholded binary image, we use a combination of image gradients, magitude and direction of gradiants and Image Saturation channel. We derive the binary thresholded image of each of these thansforms and combine them to generate a single thresholded binary image (Check: `combined_thresh` function in the notebook.)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is part of the function `perspective_transform` in the IPython notebook).  The function takes as inputs an undistorted image (`img`), as well as source (`src`) and destination (`dst`) points. And returns the perspective warped image as well as the inverse tranformation matrix, which we use during the visualization step.

I chose the hardcode the source and destination points in the following manner:

```python
src_points = np.float32(
        [[200, 720],
        [1100, 720],
        [600, 450],
        [680, 450]])
    
    dst_points = np.float32(
        [[350, 720],
        [950, 720],
        [350, 0],
        [950, 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 200, 720      | 350, 720      | 
| 1100, 720     | 950, 720      |
| 600, 450      | 350, 0        |
| 680, 450      | 950, 0        |

I verified that my perspective transform was working as expected by drawing the `src_points` and `dst_points` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

After the perspective transform step, we get a binary warped image , and then we apply the lane_fittng pipeline functions to detect the lane in an image.

We have implemented two classes namely: Line and Lane

The Line class is used to create instances of left and right line
while the Lane class is a helper function that enables us to write the advanced lane detection pipeline in very few steps.

Following are the methods in the Line and Lane Classes:

Line Class:
       
       1. get_fit  - this is used to get the best_fit
       2. add_fit  - this is used to add a new fit and calculate the new best_fit, the added fit is saved in current_fit
    
Lane Class:

        1. fit_lane  - this function is based on the lessons and we a sliding window method of 9 windows, to fit a lane.
        2. fit_visual - based on the lane that was fit using fit_lane, we used these results to fit a ploynomial
        3. measure_curvature - this is used to measure the curvature of the road 
        4. calculate_offset - this is used to calculate by how much the car has drifted from the center of the lane
        5. visualize  - this is the final step of the pipeline, which is the advanced lane detection

Following is the output of the fit_visual step which is part of the pipeline, but the output is computed separate from the per-frame processing pipeline (see: process_image function in the notebook)

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

As described earlier, we use the functions measure_curvature and calculate_offset which are part of the Lane class.

 1. `measure_curvature`
       
       In this function, we first define conversion from pixel shape to meters by using the US lane-width standards.
       We then extract, non-zero left and right lane positions
       Followed by using the Numpy `polyfit`, function to fit a polygon to the detected lane lines
       We then calculate the radii of the left lane curve and right lane curve and return
       
  2. `calculate_offset`
  
       In this function, we first calculate the vehicle center offset in pixel space, based on the `left_fit` and `right_fit`
       data which are saved as part of the lane detector. We then use the conversion ratio `3.7/700` to convert pixel offset to   meters.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The `visualizer` function of the Lane Class has the code to detect lanes that incorporate curvature and calculate vehicle offset.
We also annotate these values per frame as part of the pipeline.

Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here is the link to the output of the project [link to my video result](./output_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The calibration and perspective transform part was relatively easy, but I spent considerable amount of time working on the curve fitting algorithm and how to make it work efficently with changing frames, calulating averages etc.

The pipeline is working reasonably well, even when there is a change in road color, shadows , but the pipeline may fail in narrow roads and if the lane lines curve very soon, especially in the harder challenge.
