# **Finding Lane Lines on the Road** 

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

---


### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

Pipeline Steps:
* Convert to grayscale
* Gaussian Blur
* Canny Edge Detection
* Apply mask for region of interest
* Find lines using hough lines function
* Add those detected lines to the original image

Initially, I modified the parameters for the hough lines function in order to get the lines drawn on the lane lines without any extraneous lines for all the example images. 
These values worked well for the given images:
```
rho = 2 # distance resolution in pixels of the hogh grid
theta = np.pi/180 # angular resolution in radians of the hough grid
threshold = 120 #minimum number of votes ( intersections in hough grid cell)
min_line_len = 10 #minimum number of pixels making up a line
max_line_gap = 50 # maximum gap in pixels between connectable lines
```
After drawing lines on the video, I discovered that those parameters were too strict to draw all the required lines.  Relaxing the parameters gave me all the lines I was looking for, but would also result in drawing unwanted lines for neighboring lanes and edges between the road and grass.
Modified hough lines parameters: 
```
rho = 1 # distance resolution in pixels of the hogh grid
theta = np.pi/180 # angular resolution in radians of the hough grid
threshold = 40 #minimum number of votes ( intersections in hough grid cell)
min_line_len = 20 #minimum number of pixels making up a line
max_line_gap = 150 # maximum gap in pixels between connectable lines
```
To solve the issue, I also filtered the lines on the slope given that the current lane to be between 0.55 and 0.85 for the left lane lines and -0.55 and -0.85 for the right lane lines.  

In order to draw a single line on the left and right lanes, I modified the draw_lines() function.  Lane lines were separated into right and left lists by the slope of the line using the mathematical formula for slope.
```
def slope(line):
    for x1,y1,x2,y2 in line:
        return ((y2-y1)/(x2-x1))
```

Lines were extrapolated to the bottom of the image and top of the region of interest using the point slope equation.  `x1_result = (y_value_top_limit - y2) / slope(line) + x2`. 
The point values for the left side lines and right side lines were then averaged to form a single line for the left lane and a single line for the right lane.  
```
np.average(extrapolated_lines_left, axis=0)
```


### 2. Identify potential shortcomings with your current pipeline

The pipeline relies on the lane lines being in a certain region of interest.  If the lane of interest is not in a similar position for future data, the resulting lines could be innacurate. 

The pipeline expects images of a certain resolution and has extrapolation points and regions of interest defined by pixels. Curves could pose a problem as the lane line extrapolations are linear.  Shadows or cars in the region of interest would present additional edges that would be detected. 



### 3. Suggest possible improvements to your pipeline

A possible improvement would be to create additional points for each detected line and then do a polynomial regression through those lines.  A second possible improvement would be to not use pixel values in fuctions, but to derive desired areas of interest and extrapolation end point based on each image size.  A third possible improvement would be to use deep learning for edge detection  
