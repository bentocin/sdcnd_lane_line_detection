**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./test_images_output/region_solidWhiteRight.jpg "Cut out Region of Interest"
[image2]: ./test_images_output/white_yellow_solidWhiteRight.jpg "White and Yellow Color Filtered"
[image3]: ./test_images_output/gray_solidWhiteRight.jpg "Grayscale Conversion"
[image4]: ./test_images_output/blur_solidWhiteRight.jpg "Gaussian Blur"
[image5]: ./test_images_output/canny_solidWhiteRight.jpg "Canny Edge Detection"
[image6]: ./test_images_output/hough_solidWhiteRight.jpg "Hough Transform"
[image7]: ./test_images_output/avg_line_solidWhiteRight.jpg "Averaged Lines"
[image8]: ./test_images_output/solidWhiteRight.jpg "Annotated Lane Lines"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 8 steps that in combination filter out the lane lines in the images. The following section will give a short description of the step along with an image of the result of this step.

1) Cutting out the region of interest
This step cuts out a region of the image fed in. After the provided material was reviewed it was clear that the images are very similar in what different regions of the images show. For the lane detection, the part showing the street is the one of interest. Therefore, parameters were chosen that cut out the are of interest.
![alt text][image1] 

2) Filtering white and yellow colors
The lane lines are colored in yellow and white. Filtering for these colors only will help the edge detection later to not consider edges of shadows that show up on the street. To filter for white and yellow colors in the image, it is converted to the HLS (hue, lightness, saturation) color space and the filters are parameterized accordingly.
![alt text][image2]

3) Converting to grascale 
As a preparation for the Canny edge detection the image has to be converted to grayscale. Because the previous steps act like masks stacked upon eachother, this step just converts white and yellow pixels that were in the region of interest into grayscale. 
![alt text][image3]

4) Applying Gaussian blur 
The image is blurred with a 3x3 kernel in this example. This step also prepares the image for the Canny edge detection and helps to suppress noise in the image that would otherwise lead to a poor result of the edge detection. Although there is blurring already implemented in OpenCV's Canny edge detector, this step provides more control over the blurring than the standard implementation does.
![alt text][image4]

5) Canny edge detection 
The Canny edge detecton algorithm is used from the OpenCV library. The algorithm considers horizontal and vertical gradients of pixels in a grayscale image. In a next step it categorizes the direction and the strength of the edges. To only have edges of the width of one pixel, the Canny algorithm suppresses non-maximum values of the identified edges. In the last step the algorithm checks whether the intensity of an edge fulfills the threshold criteria set by parameters.
![alt text][image5]

6) Hough transform 
The Hough transform applied in this example converts a line parameterized by slope m and intercept b into a representation in Hough. In Hough space a line is characterized by the intersection of sine curves that are described by rho and theta, the perpendicular distance to the origing and the angle of the line away from horizontal in the traditional representation. The Hough space is split up into cells and a threshold passed as parameter to the function determines how many intersections in one cell need to exist to be considered as a line candidate. 
![alt text][image6]

7) Averaging/extrapolating the lines 
This step takes the detected lines of the Hough transform as input. According to their slope it determines whether a line belongs to the markings of the left or the right side. After that, it weights the lines based on their length and takes the average for each side.
![alt text][image7]

8) Overlapping the image and its annotation.
In the final step the original image is overlapped by the averaged lines.
![alt text][image8]


In order to draw a single line on the left and right lanes, some additional functions were added. The get_average_lines() function takes an array of line endpoints as input and as a first step calculates their slope, intercept, and length. Then in a next step it it sorts the lines by its slope into right or left lane markings. Right markings have a negative slope as the pixel indeces of the image are increasing from the op to the bottom. The function finishes with calculating an average line of the left and right markings and returns the slope and the intercept of those. Another function has been added called draw_average_line(). It takes the iamge and the Hough lines as input. It first calls the get_average_lines() function and then uses the returned slope and intercept to calculate two endpoints defining each of the two lines. These endpoints are then handed over to the draw_lines() function that will actually draw the lines.


### 2. Identify potential shortcomings with your current pipeline

One potential shortcoming is that for now the algorithm is not very flexible. This means for example that lane markings of different colors than white and yellow would be filtered out and not be considered at all. The region of interest implementation is static and assumes that all lanes have a similar width, the horizon stays at the same position of the image, the camera has a clear view of the front, lane markings are not occluded, and the street has a high contrast color to the markings. 


### 3. Suggest possible improvements to your pipeline

Instead of filtering for colors it might be a better idea to filter out horizontal lines in the region of interest so that shadows are not taken into account. To better deal with curves or uphill/downhill roads the region of interest could be dynamically adapted according to other sensor information of the car, e.g. the steering angle or the pitch of the IMU.
