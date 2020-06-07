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


![output_1](https://user-images.githubusercontent.com/34116562/49129716-d0936f80-f2f6-11e8-9165-fbbdbc7e96ec.png)
![output_2](https://user-images.githubusercontent.com/34116562/49129717-d1c49c80-f2f6-11e8-993a-03642ccce32a.png)
![output_3](https://user-images.githubusercontent.com/34116562/49129719-d2f5c980-f2f6-11e8-8875-a3a3445d4d00.png)
![output_4](https://user-images.githubusercontent.com/34116562/49129720-d4bf8d00-f2f6-11e8-9ed2-e2eb9256bbdf.png)
![output_5](https://user-images.githubusercontent.com/34116562/49129735-e4d76c80-f2f6-11e8-9a3e-af0ace294a08.png)
![output_6](https://user-images.githubusercontent.com/34116562/49129738-e6089980-f2f6-11e8-98a4-804e9e632a4a.png)

---
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
End :raising_hand:
