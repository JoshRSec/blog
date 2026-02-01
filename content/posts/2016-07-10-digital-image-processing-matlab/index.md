---
title: "Digital Image Processing - Matlab"
date: 2016-07-10
summary: "A MATLAB pipeline that denoises lung X‑ray images (median and Gaussian filtering), segments the lung region via thresholding, morphological opening, and connected components, then detects and counts circular lesions—including partial ones—using the circular Hough transform (imfindcircles). Detected circles are overlaid with green boundaries, and the total count is printed to the Command Window."
description: "A MATLAB pipeline that denoises lung X‑ray images (median and Gaussian filtering), segments the lung region via thresholding, morphological opening, and connected components, then detects and counts circular lesions—including partial ones—using the circular Hough transform (imfindcircles). Detected circles are overlaid with green boundaries, and the total count is printed to the Command Window."
slug: "digital-image-processing-matlab"
aliases:
  - "/digital-image-processing-matlab"
categories:
  - "Coursework"
  - "Image Processing"
tags:
  - "Image"
  - "Image Processing"
  - "Joshua"
  - "Joshua Robbins"
  - "MathWorks"
  - "MATLAB"
  - "Proccessing"
  - "Robbins"
---

Write a program in MATLAB that perform the following tasks for all the images:
1. Identify the type of noise in the image and seek a way to remove the noise.
The program at this stage should generate a de-noised image. (Note that not all images have the same type of noise)
2. Segment the simulated lung area from the image.
The program should generate a binary image showing the simulated lung area on a black background (e.g. Figure 1 shows an example of the segmented lung region)
3. Count and display in the Command Window the number of circles inside the lung region. For some images, half circles also appear at the edge of the lung region. Your program should count those as well. Here, your program should display a figure with detected circles marked (e.g. you may mark it with a small cross or a bounding box).
Challenge task - for additional credit, make your program also perform the following:
4. Segment detected circles from the lung region and draw their boundaries (use a green line).
# Task 1

The noise on each image was initially detected by sight, images 1 through to 6 appeared to have contained salt-and-pepper noise, so I tried applying a median filter onto the images to reduce the noise.
Firstly I implemented Matlab’s function “rgb2gray” (MathWorks, 2015), which converts an RGB image to an grayscale intensity image, by removing hue and saturation and keeping the images luminance information. The image was converted to grayscale as the image was mainly black and white with an exception to the red-cross which was not wanted and had to be later removed.

After converting the image to grayscale the image was cropped using Matlab’s “imcrop” (MathWorks, 2015), the image was cropped to a fixed rectangle which kept the lung region across all images and removed unwanted data in the image.

By converting the image to grayscale the amount of matrices storing information was reduced to 1 instead of 3 (Red, Green, Blue) improving efficiency as many of the functions implemented loop through the image matrix. Once an image is converted to grayscale its colour information will be lost, although this is not important for this application as the only colour on the images was the unwanted red-cross.

After the image is converted to grayscale it is passed through a median filter, the median filter works by taking the values of the input image corresponding to a desired sub-window or mask, I found a 3x3 mask worked best at producing a more clear filter for reducing noise, the values are then sorted and the value in the middle is then applied, for a 3x3 mask the middle value would be the 5th largest value as there are 9 values in the mask at any time.

The median filter removed most of the noise on the images except image ‘4.jpg’ which still had noise, so an Gaussian filter was applied on top of the median filter. The Gaussian filter smoothed the image, and removed the remaining noise. The Gaussian filter works by using a kernel to represent the Gaussian shape. σ determines the width of the filter which in turn determines the amount of smoothing on the image, I found the optimal value for me was 1.4, with a grid of 5x5.

![Matlab - (a) Image 4.jpg before median and Gaussian filter (b) after median and Gaussian filter](img/image1.png)

&nbsp;
# Task 2

To segment the lungs region from the image after removing the noise, the image had to be converted to a binary image, this was achieved by comparing each value in the image to a threshold, depending on the images’ value it a value of 1 or 0 is assigned to a new matrix the size of the inputted image, which after all the values have been compared and assigned a 1 or a 0 is the new outputted image.

From adjusting the threshold with the different images I concluded that the threshold of 89 worked best for me, so anything less than 89 it is assigned a 1 making it white, if the value being compare is above 89 the value is assigned 0 making it black.

It is at this stage Opening (Erosion followed by Dilation) is performed on the image. Opening is applied to smooth foreground objects by breaking narrow links and remove thin protrusions, this will help later on in the program when identifying circles. Opening was used instead of Closing (Dilation followed by Erosion) as the images foreground was white, Closing smooths with respect to the background (black in this case) and as I want to make the circles more defined Opening was implemented.

To remove the background Connected Components was used to identify each component in the image and label is respectively, once the background was identified using its label it was assigned the value of 0 making it black. The lungs were then identified, labelled and assigned the value of 1 making them white. By adding the labels of the lungs onto the background an image is produced highlighting only the lungs and removing the white background.

Connected Components works by identifying sets of values (1, 0) by checking if values of the same are connected, if sets of values meet the criteria they are labelled as a group and can be manipulated accordingly.

By applying connected components some of the unwanted background features are also removed as seen in Figure 2, below:

![Matlab - (c) image ‘4.jpg’ before connected components is applied (d) image ‘4.jpg’ after connected components is applied](img/image2.png)
# Task 3/4

To achieve task 3 and task 4 I implemented Matlab’s function “imreadcircles” (Mathworks, 2015), “imreadcircles” use a Circle Hough transform.

The Circle Hough Transform aims the find circular patterns with a given radius within an image, the Circle Hough transform does its detections by a technique which filters the image for matching circles. The Circle Hough Transform is useful as it can operate with good results on images with noise, it can detect circles which are partially obstructed (such as circles on the wall of the lung) as well as working well with low contrast objects (Ioannou, et al., 1999). As imreadcircles uses a Circle Hough Transform it  makes it ideal for this task of finding tumours in the lung and tumours which are partially obscured on the lung wall.

The parameters which are passed through the “imreadcircles” in the program are identified and described as follows:

Rmin, Rmax; Rmin is the minimum radius for the circle in which to detect, while Rmax is the maximum radius for the circle to detect.

ObjectPolarity: defines the polarity (bright or dark) or the circular objects, bright identifies circles which are brighter than the background and darker identifies circles which are darker than the background.

Sensitivity: defines the sensitivity of the Circle Hough Transform accumulator array, the higher the value the more sensitive the Circle Hough Transform accumulator array.

EdgeThreshold: sets the gradient threshold for declaring edge pixels in an image.

To count and display the number of circles detected, the output variable “radiiBright “ of “imshowcircles” is stored and then displayed into the console.

To display a green circle around the identified circles the function “viscircles” (MathWorks, 2015), is called and using the outputted variables (“centersBright”, “radiiBright”) from “imshowcircles” they are plotted on top of the segmented lung region.

![Matlab - (e) shows '4.jpg' with detected circles highlighted (f) shows number of circles highlighted displayed in Matlab console](img/image3.png)

&nbsp;
# References

Ioannou, D., Huda, W. & Laine, A., 1999. Circle recognition through a 2D Hough Transform and radius histogramming. *Image and Vision Computing, *Volume 17, pp. 15-28.

MathWorks, 2015. *Convert RGB image or colormap to grayscale. *[Online]
Available at:  https://uk.mathworks.com/help/matlab/ref/rgb2gray.html
[Accessed 26 Novemeber 2015].

MathWorks, 2015. *Create circle. *[Online]
Available at: [https://uk.mathworks.com/help/images/ref/viscircles.html](https://uk.mathworks.com/help/images/ref/viscircles.html)
[Accessed 26 November 2015].

MathWorks, 2015. *Crop image. *[Online]
Available at: [http://uk.mathworks.com/help/images/ref/imcrop.html](http://uk.mathworks.com/help/images/ref/imcrop.html)
[Accessed 26 November 2015].

Mathworks, 2015. *Find circles using circular Hough transform. *[Online]
Available at: [http://uk.mathworks.com/help/images/ref/imfindcircles.html](http://uk.mathworks.com/help/images/ref/imfindcircles.html)
[Accessed 26 November 2015].
