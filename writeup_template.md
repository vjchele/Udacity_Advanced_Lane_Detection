##Writeup Template
###You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[image1]: ./camera_calibration_original_Image.png "Original"
[image2]: ./camera_calibration_undistorted_Image.png "undistorted"
[image3]: ./Original_Image.png "Original Straight Line Image"
[image4]: ./Undistorted_Image.png "Undistorted_Image"
[image5]: ./combined_binary.png "Binary lane detected image"
[image6]: ./output_binary_warped_Image.png "binary warped image"
[image7]: ./histogram.png "Histogram image"
[image8]: ./final_result.PNG "Final Result image"
[video1]: ./video_output.mp4 "Video"

---

###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the code cell 20 of the IPython notebook Advanced_LaneDetection.ipynb .  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 
The calibrated values mtx and dist are pickled into a file 'camera_calibration_result.p' for use later.
#### Original Calibration Image
![Original][image1] 
#### Calibrated Undistorted Image
![Undistorted][image2]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one. Here even though distortion correction has been applied it is not very obvious. 
#### Original Straight Line Image
![Original_Straight_Line][image3] 
#### Undistoted Straight Line Image
![Undistorted_Straight_line][image4]
####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of color and gradient thresholds to generate a binary image (thresholding steps done in notebook cell 24). I have applied sobel operator on R and S channels. Intially, I tried with just absolute sobel operator, but found that using S and R channels are very effective. I figured out that using these channels helped me identify the lanes precisely.  Here's an example of my output for this step. Binary activation occurs anywhere two of the three (sobelx, or R threshold or S channel threshold)
#### Binary Lane Detected Image
![Binary Lane Detected][image5]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `pipeline`, which appears in notebook cell 23.  The `pipeline()` function takes as inputs an image (`img`), as well as mtx (`mtx`) and distortion coefficients (`dst`) points.  I chose the hardcode the source and destination points in the following manner. I have used an offset of 300. I tried with other values but, found 300 to give me the desired output.

```
# Source points - defined area of lane line edges
    src = np.float32([[670,450],[1100,img_size[1]],[175,img_size[1]],[590,450]])

    
    offset = 300 # offset for dst points
    dst = np.float32([[img_size[0]-offset, 0],[img_size[0]-offset, img_size[1]],
                      [offset, img_size[1]],[offset, 0]])

```
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 670, 450      | 980, 0        | 
| 1100, 720      | 980, 0      |
| 175, 720     | 300, 720      |
| 590, 450      | 300, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.
#### Binary Warped Image
![Binary warped Image][image6]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Two line objects Left_line and Right_line were created to store left and right lane information (code cell 25). Code in cell 8, draw_lines_using_sliding_window was used to first detect lanes by generating histogram of bottom half of the image and identified the lanes where the pixels peak.

This histogram follows pretty close to what I expected from the last binary warped image above. Later I used the sliding windows, to calculate the lane-line pixels and fit a second order polynomial to each lane :
#### Histogram Image
![Histogram][image7]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The function draw_lines_using_sliding_window() (code cell 26), is used to calculate the lane curvature. The code pretty much follows the sample code that was given the class. The radius of curvature and position of the car from center are two important parameters that are calculated here.

For the lane curvature, I first have to convert the space from pixels to real world dimensions. Then, I calculate the polynomial fit in that space. 
For the car's position vs. center, I calculated where the lines began at the bottom of the picture . The lines of code starting from lines 148 - 172 in cell 26


####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Code in notebook cell 27 processes the image and outputs a video.  Here is an example of my result on a test image:
#### Final Result Image
![Final Result Image][image8]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./video_output.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

This project was very challenging for me as I do not have a background in computer vision. There was lot of trial and error involved, looking for solutions online, and study how others have solved the problem was crucial for me in this project.

Every step of the way, I found things that did not work. especially the thresholds I used, which channels to use, perspective transform and finally the curvature. After lots of hours of hard work, I think I have a solution that is reasonably good.

 
Potential improvements

As explained in the class lessons, we could use deep learning techniques to perhaps train a network to output a polynomial function directly for both left and right lanes. 
We would have to train the network by manually feeding a known set of polynomials and then let the network generate the right output.

Another potential improvement could be to use convolutions for the sliding windows and maximize the number of "hot" pixels in each window. A convolution is the summation of the product of two sepeate signals.

Conclusion

Overall, it was a very challenging project, but I definitely have a much better feel for computer vision and how to detect lane lines after spending extra time on it. I hope to eventually tackle the challenge videos as well, but for now I think I have a pretty solid pipeline!  

