## Advanced Lane Finding

![sdc-1-1024x427](https://user-images.githubusercontent.com/25509152/33837471-770be3a2-de9d-11e7-97cc-50c6224bfcc3.png)

---
### Table of Contents
---

- Overview
- steps of project
- steps detail
- pipeline
- Test on videos
- concolusion
- end

---
## Overview :point_left:
---

## Advanced Lane Detection Project which includes advanced image processing to detect lanes irrespective of the road texture, brightness, contrast, curves etc. Used Image warping and sliding window approach to find and plot the lane lines. Also determined the real curvature of the lane and vehicle position with respect to center.

---
![](gif.gif)
---
## steps of project :
---
### Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.

### Apply a distortion correction to raw images. Use color transforms, gradients, etc., to create a thresholded binary image

### Apply a perspective transform to rectify binary image ("birds-eye view").

### Detect lane pixels and fit to find the lane boundary.

### Determine the curvature of the lane and vehicle position with respect to center.

### Warp the detected lane boundaries back onto the original image.

### Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle

### applay this steps on video stream.


---
## Camera Calibration
---

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The first step in the pipeline is to undistort the camera. Some images of a 9x6 chessboard are given and are distorted. Our task is to find the Chessboard corners an plot them. For this, after loading the images we calibrate the camera. Open CV functions like findChessboardCorners(), drawChessboardCorners() and calibrateCamera() help us do this.

---
### image distortion

#### These images are Distortion Corrected :

- code 

```
objp = np.zeros((6*9,3), np.float32)
objp[:,:2] = np.mgrid[0:9,0:6].T.reshape(-1,2)
ret, corners = cv2.findChessboardCorners(gray, (9,6),None)

```
```
def cal_undistort(img, objpoints, imgpoints):
    # Use cv2.calibrateCamera() and cv2.undistort()
    ret, mtx, dist, rvecs, tvecs = cv2.calibrateCamera(objpoints, imgpoints, img.shape[1::-1], None, None)
    undist = cv2.undistort(img, mtx, dist, None, mtx)
    print( dist)
    return undist
```

![output_1](https://user-images.githubusercontent.com/34116562/49129716-d0936f80-f2f6-11e8-9165-fbbdbc7e96ec.png)
![output_2](https://user-images.githubusercontent.com/34116562/49129717-d1c49c80-f2f6-11e8-993a-03642ccce32a.png)
![output_3](https://user-images.githubusercontent.com/34116562/49129719-d2f5c980-f2f6-11e8-8875-a3a3445d4d00.png)
![output_4](https://user-images.githubusercontent.com/34116562/49129720-d4bf8d00-f2f6-11e8-9ed2-e2eb9256bbdf.png)
![output_5](https://user-images.githubusercontent.com/34116562/49129735-e4d76c80-f2f6-11e8-9a3e-af0ace294a08.png)
![output_6](https://user-images.githubusercontent.com/34116562/49129738-e6089980-f2f6-11e8-98a4-804e9e632a4a.png)

---

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image. Provide an example of a binary image result.

Detecting edges around trees or cars is okay because these lines can be mostly filtered out by applying a mask to the image and essentially cropping out the area outside of the lane lines. It's most important that we reliably detect different colors of lane lines under varying degrees of daylight and shadow. So, that our self driving car does not become blind in extreme daylight hours or under the shadow of a tree.
 
I performed gradient threshold and color threshold individually and then created a binary combination of these two images to map out where either the color or gradient thresholds were met called the combined_binary in the code.

![output_1](https://user-images.githubusercontent.com/34116562/49129596-68448e00-f2f6-11e8-8d0d-bfa31bbc1dbe.png)
![output_1](https://user-images.githubusercontent.com/34116562/49129605-6c70ab80-f2f6-11e8-9678-c058d70bd1a3.png)
![output_2](https://user-images.githubusercontent.com/34116562/49129599-6a0e5180-f2f6-11e8-9539-b4d630e90a84.png)
![output_2](https://user-images.githubusercontent.com/34116562/49129607-6e3a6f00-f2f6-11e8-926a-bc7c87323ce2.png)
---
```
thresh_min = 20
thresh_max = 100
s_thresh_min = 170
s_thresh_max = 255

```
---

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

Perspective Transform is the Bird's eye view for Lane images. We want to look at the lanes from the top and have a clear picture about their curves. Implementing Perspective Transform was the most interesting one for me. I used values of src and dst as shown below: 

src = np.float32([[585, 460],[203, 720],[1127, 720],[695, 460]])

dst = np.float32([[320,0],[320,720],[960, 720],[960, 0]])

Also, made a function warper(img, src, dst) which takes in the BInary Warped Image and return the perspective transform using cv2.getPerspectiveTransform(src, dst) and cv2.warpPerspective(img, M, img_size, flags=cv2.INTER_NEAREST). The results are shown below:


![output_1](https://user-images.githubusercontent.com/34116562/49129781-1d774600-f2f7-11e8-80df-2d0d6b0a3950.png)
![output_2](https://user-images.githubusercontent.com/34116562/49129782-1f410980-f2f7-11e8-894a-94fdf6f519d8.png)
![output_3](https://user-images.githubusercontent.com/34116562/49129786-20723680-f2f7-11e8-9268-273b2526af11.png)
![output_4](https://user-images.githubusercontent.com/34116562/49129788-223bfa00-f2f7-11e8-9aed-0555271d53ed.png)
![output_5](https://user-images.githubusercontent.com/34116562/49129790-2405bd80-f2f7-11e8-90c8-61f1f9490cab.png)
![output_6](https://user-images.githubusercontent.com/34116562/49129795-2700ae00-f2f7-11e8-8987-eda7e5cc51a2.png)

---
```
def warper(img, src, dst):

    # Compute and apply perpective transform
    img_size = (img.shape[1], img.shape[0])
    M = cv2.getPerspectiveTransform(src, dst)
    warped = cv2.warpPerspective(img, M, img_size, flags=cv2.INTER_NEAREST)  # keep same size as input image

    return warped

```
---

### Finding the lines - Sliding Window and fitting a polynomial

![output_1](https://user-images.githubusercontent.com/34116562/49129915-9bd3e800-f2f7-11e8-9d7b-7ee1480b480c.png)
![output_2](https://user-images.githubusercontent.com/34116562/49129917-9c6c7e80-f2f7-11e8-8b37-4791cb2e60e3.png)
![output_3](https://user-images.githubusercontent.com/34116562/49129920-9e364200-f2f7-11e8-9852-5e35607d7c07.png)
![output_4](https://user-images.githubusercontent.com/34116562/49129924-9f676f00-f2f7-11e8-81fb-aa529361acfc.png)


### Finding the lines - Search From Prior and fitting a polynomial

![output_1](https://user-images.githubusercontent.com/34116562/49129956-b60dc600-f2f7-11e8-82e2-c8e48d9e3d3f.png)
![output_2](https://user-images.githubusercontent.com/34116562/49129958-b73ef300-f2f7-11e8-9898-bada3f974564.png)
![output_3](https://user-images.githubusercontent.com/34116562/49129959-b8702000-f2f7-11e8-9477-95f0ef30d523.png)
![output_4](https://user-images.githubusercontent.com/34116562/49129961-b9a14d00-f2f7-11e8-8d33-315251f554fb.png)
![output_5](https://user-images.githubusercontent.com/34116562/49129965-bb6b1080-f2f7-11e8-9d74-91581421f7f2.png)
![output_6](https://user-images.githubusercontent.com/34116562/49129969-bdcd6a80-f2f7-11e8-8428-96e62595f872.png)
---
#### 4. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

After implementing all the steps, it's time to create the pipeline for one image. Created a function process_image() as the main pipeline function. Also, I put the Radius of Curvature and Center Offset on the final image using cv2.putText() function. The result is shown below: 

##### Lane Area Drawn without Information:

![output-without-info](https://user-images.githubusercontent.com/34116562/49130028-fec57f00-f2f7-11e8-8293-34e09bc07880.png)

##### Lane Area Drawn with Radius of Curvature and Central Offset:

![output-with-radius-and-offset](https://user-images.githubusercontent.com/34116562/49130030-008f4280-f2f8-11e8-8ed1-8a1aad84de5a.png)

---
## pipeline for video stream

```
white_output = 'test_videos_output/project_video_output.mp4'
clip1 = VideoFileClip("project_video.mp4")
white_clip = clip1.fl_image(process_image) #NOTE: this function expects color images!!
%time white_clip.write_videofile(white_output, audio=False)

```
---
### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

i face problem when i try to upload folder on colab but stack over flow help me in this i applay alot of technique which i have learned from this nanodegree like how to make calibration to my camera and how to undistortion images which come from camera 
using math equation which i have learned in lesson and applay prespective transform to image to correct it using src and dest array 
and applay how to cover line and calculate curvature of road by calculate width of road 3.7 meter and length camera 30 meter 
and finally applay pipeline on the video stream 

---
```
def process_image(image):
    # TODO: put your pipeline here
    initial_image = np.copy(image)

    undistorted = cal_undistort(initial_image, objpoints, imgpoints)

    #gray = cv2.cvtColor(undistorted, cv2.COLOR_RGB2GRAY)
    hls = cv2.cvtColor(undistorted, cv2.COLOR_RGB2HLS)

    s_channel = hls[:,:,2]

    gray = cv2.cvtColor(undistorted, cv2.COLOR_RGB2GRAY)

    sobelx = cv2.Sobel(gray, cv2.CV_64F, 1, 0) # Take the derivative in x

    abs_sobelx = np.absolute(sobelx) # Absolute x derivative to accentuate lines away from horizontal

    scaled_sobel = np.uint8(255*abs_sobelx/np.max(abs_sobelx))

    sxbinary = np.zeros_like(scaled_sobel)

    sxbinary[(scaled_sobel >= thresh_min) & (scaled_sobel <= thresh_max)] = 1

    s_binary = np.zeros_like(s_channel)

    s_binary[(s_channel >= s_thresh_min) & (s_channel <= s_thresh_max)] = 1

    color_binary = np.dstack(( np.zeros_like(sxbinary), sxbinary, s_binary)) * 255

    combined_binary = np.zeros_like(sxbinary)

    combined_binary[(s_binary == 1) | (sxbinary == 1)] = 1

    warped_image = warper(combined_binary, src, dst)

    #result = fit_polynomial(warped_image)
    #result = search_around_poly(warped_image)

    curvature_string, offset = radius_and_offset(warped_image)

    margin = 100

    nonzero = warped_image.nonzero()

    nonzeroy = np.array(nonzero[0])

    nonzerox = np.array(nonzero[1])

    left_lane_inds = ((nonzerox > (left_fit[0]*(nonzeroy**2) + left_fit[1]*nonzeroy + 
                                   
                    left_fit[2] - margin)) & (nonzerox < (left_fit[0]*(nonzeroy**2) + 
                                                          
                    left_fit[1]*nonzeroy + left_fit[2] + margin)))
    
    right_lane_inds = ((nonzerox > (right_fit[0]*(nonzeroy**2) + right_fit[1]*nonzeroy + 
                                    
                    right_fit[2] - margin)) & (nonzerox < (right_fit[0]*(nonzeroy**2) + 
                                                           
                    right_fit[1]*nonzeroy + right_fit[2] + margin)))
    
    leftx = nonzerox[left_lane_inds]

    lefty = nonzeroy[left_lane_inds] 

    rightx = nonzerox[right_lane_inds]

    righty = nonzeroy[right_lane_inds]

    left_fitx, right_fitx, ploty = fit_poly(warped_image.shape, leftx, lefty, rightx, righty)

    warp_zero = np.zeros_like(warped_image).astype(np.uint8)

    color_warp = np.dstack((warp_zero, warp_zero, warp_zero))

    pts_left = np.array([np.transpose(np.vstack([left_fitx, ploty]))])

    pts_right = np.array([np.flipud(np.transpose(np.vstack([right_fitx, ploty])))])

    pts = np.hstack((pts_left, pts_right))

    cv2.fillPoly(color_warp, np.int_([pts]), (0,255, 0))

    newwarp = cv2.warpPerspective(color_warp, M_inverse, img_size) 

    result_final = cv2.addWeighted(undistorted, 1, newwarp, 0.3, 0)

    cv2.putText(result_final,curvature_string , (125, 90), cv2.FONT_HERSHEY_SIMPLEX, 1.7, (255,255,255), thickness=4)

    cv2.putText(result_final, offset, (125, 150), cv2.FONT_HERSHEY_SIMPLEX, 1.7, (255,255,255), thickness=4)

    return result_final

result_image = process_image(image123)

plt.imshow(result_image, cmap='gray')
```
---
End :raising_hand:
