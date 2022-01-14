# **Finding Lane Lines on the Road** 

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 7 steps. These steps are described in detail in the sections below:

  ### 1. Color Selection and Thresholding
  The first step in this section was to identify and select the colours to be used in thresholding. Clearly, the key colours for lane detection are yellow and white. A new helper function titled ```thresholding()``` was declared for this task. Using HSV for the yellow thresholding and RGB for the white thresholding, upper and lower boundaries were set from which appropriate masks were created. Using the ```bitwise_and()``` function, a yellow image and white image were both created from masking the original image using their respective colour ranges. These can be seen in the two images below (Yellow above, White below):
  
  ![yellow](https://user-images.githubusercontent.com/64507102/149540184-f38bab8e-31cd-400a-9ed8-99bf9d778f2f.png)
  ![white](https://user-images.githubusercontent.com/64507102/149540200-e0bdf82c-7db5-4a99-bc35-d95c16f37d72.png)
  
  These two images were then merged such that they could be grayscaled.

  ### 2. Grayscale
  The next step was to grayscale the merged image containing the yellow and white components of the original image. This is to satisfy the required parameter for the gassian blur being a grayscaled photo which is then fed to the canny edge detector later. A photo of the grayscaled image can be seen below:
  
  ![grayscaled](https://user-images.githubusercontent.com/64507102/149541103-f489c0b9-fd8c-4835-ab1c-b01190c065e7.png)
  
  ### 3. Gaussian Blur
  The grayscaled image was then put through a gaussian blur function to smooth out some of the lines with a selected kernel size of 15 from trial and error revealing the best result.
  
  ### 4. Canny Edge Detection
  The smoothed grayscale image was then passed to the canny edge detector that highlighted the sharp edges of the objects it detected. This can be seen in the following image:
  
  ![canny](https://user-images.githubusercontent.com/64507102/149541647-cbcf073c-d124-4ebc-8a1a-08970fce2d64.png)
  
  ### 5. Region of Interest Selection
  Using the image from step 4, a more specific region of interest was selected, removing the objects in the sky that could potentially lead to distractions for the algorithm from the lanes. This refined image can be seen below:
  
  ![roi](https://user-images.githubusercontent.com/64507102/149541786-415a4193-3418-4318-b949-a7b3f5c5d55d.png)

  ### 6. Hough Line Detection (Line segment detection)
  This step was crucial in identifying the actual lines to draw. The ```hough_lines``` function was left mostly untouched, however the ```draw_lines``` function called within it was modified heavily.
  
  In order to identify the best left and right lines to plot for the hough lines, a rolling average for the left and right slope is calculated continuously based on the present line coordinates (x1, x2, y1, y2) available at any given frame. Using these lines and their coordinates, the slope and hough intersect (b value) are constantly calculated. With these values, I am able to distinguish whether a line is contributing to the left or right side based off the slope (a negative slope value indicates it is a left lane, whereas a positive slope value indicates a right lane - this is because the figure plots are inverted in the Y-axis of each image, making the slopes backwards). In addition to using slopes to distinguish between the sides, the average intersect is used with the slope to move the lines from the Hough space back to the Image space. Given a set of coordinates mapping to a singular line allows me to plot the most accurate and point-awarded line possible at any given frame.
  
  A depiction of the hough lines of a single frame can be seen in the image below:
  
  ![hough](https://user-images.githubusercontent.com/64507102/149543037-97440cb7-8cc7-4482-ab8e-ff2dedc9501a.png)

  ### 7. Line Fitting (Projecting the Hough Lines)
  The last step is project the hough lines onto the original image using the ```weighted_img``` helper function. This final depiction of lane line tracking can be seen below:
  
  ![1](https://user-images.githubusercontent.com/64507102/149543507-6527679c-7ad2-46e1-9daf-e3da68ce18cb.png)

### 2. Identify potential shortcomings with your current pipeline

One shortcoming I identified while examining the videos is the shakiness of the lines. I noticed this mostly when there was slight paint problems on the road making it harder to detect, or when a slight curve changed the trajectory of the line. This was especially evident during the challenge video where the dotted white line on the curve was having plenty of trouble identifying where to place the line.

Another slight shortcoming is that the white threshold was not as great at identifying its lines as the yellow threshold using HSV. This could be attributed to it being easily confused with certain gray patches or portions on the road itself, or because RGB isn't as accurate for thresholding as HSV or other alternatives.

Possible improvements for these shortcomings are discussed in the following section.

### 3. Suggest possible improvements to your pipeline

Firstly, a possible improvement for the curve and shakiness issue could be avoiding the use of one primary averaged line per frame. Rather, with more data and smaller lines to work with, rolling averages every few pixels could be calculated determining the position and slope of the highest point-awarded lines at any part of the identified lane. This would allow for smoother curves to be detected and also overlayed on the original frames.

Secondly, another improvement for the white threshold detection issue could be examining specific colours per pixel to identify the best upper and lower thresholds for the white mask. At the moment, the values I used were taken directly from class notes which are an approximation for the best value for the colour white with RGB thresholding; however, it is not exactly a perfect application for this scenario on the road. A colour picker could provide more detail.
