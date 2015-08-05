---
layout: post
title: How to detect if the oven is on with OpenCV and Python
comments: true
---

"Did I leave the <span style="color:red">oven</span> on?"

![did_i_leave_the_oven_on]({{ site.baseurl }}/images/did_i_leave_the_oven_on.jpg)

This question will sometimes pop up at the most inconvenient times.

Sometimes when you just left your house.<br/>
Sometimes on your way to work.<br/>
Sometimes on a plane while you're on your way for a long vacation...<br/>

There are different solutions for this problem:

1. The [rubber band method](http://www.apartmenttherapy.com/did-i-leave-the-oven-on-try-th-131769)
2. Saying / singing it out loud (like [Samuel L. Jackson](https://www.youtube.com/watch?v=vPN4XjrwD78))
3. Labeling your appliances or even making a checklist that includes the <span style="color:red">oven</span> before leaving the house to a long vacation.

Or, we can do something better...

In this tutorial we'll try the technical approach to the problem.

Full code can be found on [Github](https://github.com/kazuar/opencv_light_detection).

# Problem definition
In our case, we need to decide on the signal that we'll use in order to determine if the <span style="color:red">oven</span> is on / off.
In my kitchen, that signal is a red light on top of the text "<span style="color:red">OVEN</span> ON".

When the red light is on, the <span style="color:red">oven</span> is on:

![oven_on]({{ site.baseurl }}/images/oven_on.jpg)

When the red light is off, the <span style="color:red">over</span> is off:

![oven_off]({{ site.baseurl }}/images/oven_off.jpg)

# Prerequisites
Make sure that on your machine you have the following:

1. OpenCV 3.0
2. Python 2.7
3. Numpy 1.9

## Installing OpenCV 3.0 + Python 2.7
If you don't have OpenCV installed on your machine, start by following [Adrian Rosebrock's excellent tutorial](http://www.pyimagesearch.com/2015/06/15/install-opencv-3-0-and-python-2-7-on-osx/) on installing OpenCV 3.0 and Python 2.7+ on OS X.
[I've added my own notes about the installation process](http://kazuar.github.io/opecv-python2/), in case you run into some issues in compiling OpenCV 3.0 on OS X.

# Process
Based on the fact you were successfull in installing OpenCV on your environment, we will start the process of analyzing our data in order to determine if the <span style="color:red">oven</span> is ON / OFF.

## Load the required packages

1. argparse - for argument handling.
2. numpy - a highly optimized library for numerical operations. OpenCV uses numpy for its array structures.
3. cv2 - OpenCV for processing images.

{% highlight python %}
import argparse
import numpy as np
import cv2
{% endhighlight %}

## Loading the image
{% highlight python %}
image = cv2.imread(image_path)
{% endhighlight %}

## Reducing noise in the image
We will want to smooth the input image in order to reduce the noise in the image. This will make it easier to detect objects in the image. For medianBlur we will use aperture size of 3, higher value will mean that the image will be more blurry:
{% highlight python %}
blur_image = cv2.medianBlur(image, 3)
{% endhighlight %}

## Convert the image colors to HSV

HSV - Hue, Saturation and Value (brightness). HSV will allow us to extract a colored object since it is easier to represent a color in HSV than in BGR.

Converting the image to HSV will allow us to identify a color in the image using the hue value (a single value instead of three).

This is how it's done:
{% highlight python %}
hsv_image = cv2.cvtColor(blur_image, cv2.COLOR_BGR2HSV)
{% endhighlight %}

The code will result in the following image:

![hsv_on]({{ site.baseurl }}/images/hsv_on.jpg)

## Detecting color in the image

In order to decide on the color that we want to detect, we can look at the histogram of the color values in the image of the <span style="color:red">oven</span> light.

![image_color_hist]({{ site.baseurl }}/images/image_color_hist.png)
![light_mask]({{ site.baseurl }}/images/light_mask.png)

We can see that the color red is dominant in the image. There are two peaks of red color - one in the low range and one in the high range. These color values are translated to hue in the range of 0 to 10 and 160 to 180 (for the color red).

For each hue range we will create a mask on the HSV image and remove everything that isn't in the selected range of the required color.

{% highlight python %}
def create_hue_mask(image, lower_color, upper_color):
    lower = np.array(lower_color, np.uint8)
    upper = np.array(upper_color, np.uint8)
 
    # Create a mask from the colors
    mask = cv2.inRange(image, lower, upper)
    output_image = cv2.bitwise_and(image, image, mask = mask)
    return output_image

# Get lower red hue
lower_red_hue = create_hue_mask(hsv_image, [0, 100, 100], [10, 255, 255])

# Get higher red hue
higher_red_hue = create_hue_mask(hsv_image, [160, 100, 100], [179, 255, 255])
{% endhighlight %}

With the following results:

![lower_hue]({{ site.baseurl }}/images/lower_hue.png)
![higher_hue]({{ site.baseurl }}/images/higher_hue.png)

Next step is to merge these images together in order to catch all of the red hues:
{% highlight python %}
full_image = cv2.addWeighted(lower_red_hue, 1.0, higher_red_hue, 1.0, 0.0)
{% endhighlight %}

Which will have the following result:

![full_hue]({{ site.baseurl }}/images/full_hue.jpg)

## Find circles in the image
Now we have a picture with only the red hues in it, and we will want to identify whether the light is on (there is a circle of red hues) or the light is off (there is no circle).
We need to find the circles in the new image, but first we will need to convert the image to grayscale (since the input for HoughCircles is a grayscale image).

For detecting the circles in the image we will use the following parameters:

1. The image converted to grayscale.
2. HOUGH_GRADIENT is the circle detection method (currently the only one).
3. The inverse ratio of resolution. In this case, 1.2 will be used.
4. The minDist will be 100 - we want to make sure that we don't accidentaly detect false positive circles.

{% highlight python %}
# Convert image to grayscale
image_gray = cv2.cvtColor(full_image, cv2.COLOR_BGR2GRAY)

# Find circles in the image
circles = cv2.HoughCircles(image_gray, cv2.HOUGH_GRADIENT, 1.2, 100)
{% endhighlight %}

# Results
At this point it will be enough to check if there are any circles.
If there are, it means that we have a signal that at least one of the <span style="color:red">oven</span> lights is on.<br/>
If we couldn't find any circles it means that none of the lights are on and the <span style="color:red">oven</span> is off.

To prove that, we can draw the circles on the original image with the following code:
{% highlight python %}
# Draw the circles on the original image
circles = np.round(circles[0, :]).astype("int")
for (center_x, center_y, radius) in circles:
    cv2.circle(image, (center_x, center_y), radius, (0, 255, 0), 4)
{% endhighlight %}

The result will be:

![original_image_with_circles]({{ site.baseurl }}/images/original_image_with_circles.jpg)

# Next steps
There are many things that we can do from here, some examples are:

1. Detect the specific light that is on and by that understanding the exact status of the <span style="color:red">oven</span>.
2. Create a service that will allow us to check the status of the <span style="color:red">oven</span> remotely.
3. Add this functionality to raspberry pi so we can have a small device that will alert us if the <span style="color:red">oven</span> is off or on.

Full code sample can be found on [Github](https://github.com/kazuar/opencv_light_detection).
