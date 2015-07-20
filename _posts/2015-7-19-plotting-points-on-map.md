---
layout: post
title: Plotting geo coordinates on map
comments: true
---

In case you want to plot geo coordinates on a map:

In this example, we want to plot geo coordinates on a map of US states.

In order to do that we first need to install some dependencies. Most important are the following:
{% highlight bash %}
pip install matplotlib
pip install basemap --allow-external basemap --allow-unverified basemap
{% endhighlight %}

The solution is based on this question: [Stackoverflow question](http://stackoverflow.com/questions/7586384/color-states-with-pythons-matplotlib-basemap)

Keep in mind to have the following files in your script directory ([Can be found here](https://github.com/matplotlib/basemap/tree/master/examples)):

1. st99_d00.shx
2. st99_d00.shp
3. st99_d00.dbf

{% highlight python %}
import matplotlib.pyplot as plt
from matplotlib.patches import Polygon
from mpl_toolkits.basemap import Basemap

# create the map
map = Basemap(
	llcrnrlon = -119, llcrnrlat = 22, urcrnrlon = -64, 
	urcrnrlat = 49, projection = 'lcc', lat_1 = 33, 
	lat_2 = 45, lon_0 = -95)

# load the shape file with "states"
map.readshapefile('st99_d00', name='states', drawbounds=True)

# set a geo coordinate (for example, new york)
lat = 40.7127
lon = -74.0059
x,y = map(lat, lon)

# place it on the map
map.plot(x, y, 'ro', markersize=4)

plt.show()
{% endhighlight %}

The result should like:

![basemap]({{ site.baseurl }}/images/basemap.png)









