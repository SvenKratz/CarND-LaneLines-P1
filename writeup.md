# **Finding Lane Lines on the Road**

## Assignment Writeup



---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"
[image2]: ./writeup_images/pipeline_example.png "Visualization of pipeline output"
[image3]: ./writeup_images/drawing_function_diagram.png "Flowchart of updated drawing function"

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

** Image Processing Pipeline **

My final image processing pipeline (P1.ipynb, cell #24) for the most part follows the series of steps developed in the video lecture. Finding the right parameters for Gaussian blur, Canny edge detection and Hough transform resulted in significant trial & error experimentation. The pipeline performs the following steps:

1. convert input image to grayscale
2. apply Gaussian blur, with kernel size 5
3. apply Canny edge detection, with thresholds set to (75,150)
4. mask the image to a region of interest to the area most likely containing the lane lines
5. apply hough line detection with the following parameters set on the function call:
```
def hough_lines(img,
                rho = 3,
                theta = np.pi /180.0,
                threshold = 6,
                min_line_len = 65,
                max_line_gap = 25,
                improved_line_drawing = False):
```
6. draw lane lines using improved lane drawing function

The following image shows a visualization of the processing pipeline steps:

![alt text][image2]

**Improved Lane Drawing Function:**

The default lane-marking functions doesn't always produce a visually pleasing output. This is because since non-continuous lane markings are detected as disjointed sets of lines by the Hough transform. Sometimes spurious lines, that are not part of the lane markings are also detected.

I therefore implemented an improved lane drawing function, ```draw_lines_improved```, which aims at improving the lane marking detection visualization, by drawing a clear left and right line from the bottom of the screen to the top of the region of interest and smoothing the resultant output in order to reduce jitter in the visualization. The resulting smoothed lines could then also be used by other software modules to actually control the self-driving car.  

The improved lane drawing function performs the following steps:

1. Divide line segments found by Hough transform into left and right sets (`left_lines` and `right_lines`). This is accomplished by looking at the sign of the calculated slope for each line segment. The function `slope`implements slope finding, with the formula given in the template.
2. We extend the line segments in `left_lines` and `right_lines` to extend to the bottom of the screen and to the top of our region of interest, in order to make the visualization more consistent.
The function `get_extended_lines`implements line extension. We calculate slope and intercept of the line, and then solve the line equation for `y=0` and`y=top`in order to find the x position of the line on the bottom of the screen and top of the region of interest. The formula used for intercept is derived as follows `y = ax + b --> b = y - ax`. The formula used for obtaining the x coordinate when extending the line to an arbitrary y value is derived as follows `y = ax + b --> x = (y - b) / a`
3. In order to reduce jitter we perform averaging on the top and bottom x coordinates of the extended line segments, and then use this average value for further processing and, ultimately, to draw a visualization of the lane lines. The function `get_line_averages` performs this operation on a set of lines.
4. To further reduce jitter, we apply an averaging filter between video frames that smooths the final line output for `left_lines`and `right_lines`, respectively. The helper class `Filter` implements filtering by managing a FIFO queue and returning an average of the values when necessary. The filters have a queue length of 10, which provides adequate smoothing with minimal drift. Also, if no extended line segments were found in step (2), we retrieve the last value from the filters.
5. The filtered line segments are drawn on the input image via `cv2.line`. The left lane line is drawn in red and the right lane line is drawn in green.

The following diagram shows a graphical depiction of the update drawing function:

![alt text][image3]

### 2. Identify potential shortcomings with your current pipeline


I noticed these issues with my current approach:

1. The resulting lines can be "jiggly" if all the parameters for the image processing pipeline are not set exactly right.
2. I'm not sure how the approach will work with another other cars partially obscuring the lanes in front
3. It is unclear how the approach would work with narrower or wider lane markings.
4. The approach may not work at night or perhaps may work better, as lane markers are highlighted by the car's headlights
5. Hough transform can find several similar line segments, which may lie on the same line if connected. Performance could be optimized if the line drawing functions filtered out or combined line segments that lie on the same line.


### 3. Suggest possible improvements to your pipeline

I improved upon (1) by adding filtering over multiple frames to smooth out the detected lines. However, the image processing may need to be revisited to improve performance. Surprisingly, though, my pipeline worked very well on the "yellow line" example.

The image processing pipeline could, for instance, be improved by applying contrast-enhancing methods (e.g., gray-value normalization) early on to obtain better Canny Edge detection. Morphological operations such as dilation could be used to enhance short, or broken line segments to improve line detection.

As for issues (2)  and (3), it seems that the fixed region of interest seems to be a weak point in the algorithm. A method needs to be used to roughly determine the bounding boxes of this region dynamically. An approach could be to initialize the ROI in a fixed way and then dynamically adapt it based on the last line segments that were found within the region. E.g., if the line segments are found primarily inside, the ROI slowly contracts, if fewer line segments are found the ROI expands. This ROI issue is also one of the causes that the "challenge" video is not running optimally with my pipeline.

For issue 4, I would need additional video material with night-driving footage. One challenge here would be, that, depending on the dip of the low-beam headlights the ROI that contains the lane markings may be smaller than during the daytime, so the image processing pipeline would have to compensate for this.
