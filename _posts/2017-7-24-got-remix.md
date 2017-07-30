---
layout: post
title: Sam cleaning the Citadel (GoT) for 10 hours with Python
tags:
- python
comments: true
---

<iframe width="420" height="315" src="http://www.youtube.com/embed/hI4Ed9ejEuU" frameborder="0" allowfullscreen></iframe>

As a fan of `Game of Thrones`, I couldn't wait for it to return for this 7th season and I really enjoyed that great scene of Sam cleaning the Citadel.

I enjoyed it so much that I wanted to see more of it... **much more of it.**

In this post we'll take the short video compilation of sam cleaning the Citadel,
we will split it to multiple sub clips and create a random video of sam cleaning the Citadel using a random mix of those sub clips.

Now, it would have taken me about 5 minutes to do all this manually by splitting the video and then combining all the parts using some video editing program.

But why would I waste 5 minutes when I can spend a whole weekend doing the same thing with Python?
I mean... am I'm right or am I right? **Right?**

Anyway... 

We will use Python to analyze the audio of the video, find the "silent" parts in the video and split those into frames.
Then, we will combine those frames in a random order and save it as a new video file.

> **Disclaimer:**  As this is one of the first times I've actually done this kind of signal processing and analysis, I'm pretty sure there are better and more optimized ways to do these kind of things. If you have a different suggestion, please share them in the comments.

Source code can be found on [github](https://github.com/kazuar/got_remix).

## Create a dev environment for our project

For this project, I used two main packages:

1. [Librosa](https://github.com/librosa/librosa) - Python library for audio and music
2. [MoviePy](https://github.com/Zulko/moviepy) - Python library for video

Start by creating a virtual / conda environment with the following packages:

{% highlight python %}
numpy
matplotlib
librosa
moviepy
jupyter
progressbar
{% endhighlight %}

We will use the `librosa` package to analyze our audio and find our frames in the audio.
After that we will use those frames to split the video and compine them randomly using the `moviepy` package.

## Processing the audio frames in the video

First we'll load the audio from our video file:

{% highlight python %}
video_file_path = "resources/short_got.mp4"
audio_data, sr = librosa.load(video_file_path)
{% endhighlight %}

```librosa.load``` will load the audio data from the file as a numpy array into ```audio_data``` and will set the sampling rate of the audio data in ```sr```, which is the number of samples per second in the audio.

Lets take a look at the audio.
In order to do that we can use ```librosa.display``` to plot the wave.

The code for that will use librosa and matplotlib packages:

{% highlight python %}
import librosa
import librosa.display
import matplotlib.pyplot as plt

video_file_path = "resources/short_got.mp4"
audio_data, sr = librosa.load(video_file_path)
librosa.display.waveplot(audio_data)
plt.show()
{% endhighlight %}

If we run the above code on our video file we will see the following wave plot:

![waveplot]({{ site.baseurl }}/images/got_remix/wave_plot.png)

Seems like this audio is a bit noisy.
We need to look at the "silent" parts of the audio and find a way to identify them.

Lets zoom into one of the parts in the audio and try to identify the "boundaries of silence":

![waveplot]({{ site.baseurl }}/images/got_remix/wave_plot_pre_zoom.png)

We get a better and clearer image of what the audio looks like in the "silent" parts:

![waveplot]({{ site.baseurl }}/images/got_remix/wave_plot_zoom.png)

It seems like we can declare that a location in the audio is silent if the wave is between `-0.02` to `0.02`.

Let take a look at the plot with those limits:

{% highlight python %}
import librosa
import librosa.display
import matplotlib.pyplot as plt

# Load audio from file
video_file_path = "resources/short_got.mp4"
audio_data, sr = librosa.load(video_file_path)

# Display wave plot
librosa.display.waveplot(audio_data)

# Add limits between -0.02 and 0.02
plt.plot([0.02] * len(audio_data))
plt.plot([-0.02] * len(audio_data))

# Show the plot
plt.show()
{% endhighlight %}

![waveplot]({{ site.baseurl }}/images/got_remix/wave_plot_limits.png)

Once we set our limits, we can analyze the audio and try to find "silent" frames.
We do that by setting a frame size and then iterating over the audio data, frame by frame, and checking if each frame has a value that is beyond our limits.

{% highlight python %}
import math
import librosa
import librosa.display
import matplotlib.pyplot as plt

# Load audio from file
video_file_path = "resources/sam_citadel.mp4"
audio_data, sr = librosa.load(video_file_path)

max_limit = 0.02
frame_size = 0.1
frame_len = frame_size * sr

# Calculate the number of frames in the audio file
num_of_frames = math.floor(len(audio_data) / frame_len)

silent_frames_indexes = []
for frame_num in range(int(num_of_frames)):
    # Get the start and end of the frame
    start = int(frame_num * frame_len)
    stop = int((frame_num + 1) * frame_len)

    # Get the absolute values in each frame
    abs_frame = map(abs, audio_data[start:stop])

    # Get the maximum value in the frame
    cur_max_val = max(abs_frame)

    # We've reached a "silent" frame if the maximum 
    # value in the frame below our limit
    if cur_max_val < max_limit:
        silent_frames_indexes.append(frame_num)
{% endhighlight %}

What we've done here is find the indexes of silent frames in the audio. 

In our case, each frame is the size of `0.01` so each frame's length is `0.01 * sample rate (22050)`.

For each frame, if the absolute maximum value of the frame is below our limit (`0.02`) we mark it as a "silent" frame and collect it in our `silent frames` list.

Once we collected all indexes of the "silent" frames, we can plot them on our audio wave:

![waveplot]({{ site.baseurl }}/images/got_remix/wave_plot_silent_frames.png)

It seems like there's a lot of sections that we signaled as "silent".

We can aggregate them and take another look at a cleaner plot.

{% highlight python %}
# Aggregate the frames into sections of silent frames
aggregated_frames = [silent_frames_indexes[0]]
for index, frame_num in enumerate(silent_frames_indexes[1:]):
    if frame_num - silent_frames_indexes[index] > FRAME_MIN_SIZE:
        aggregated_frames.append(frame_num)
{% endhighlight %}

![waveplot]({{ site.baseurl }}/images/got_remix/wave_plot_aggregated_silent_frames.png)

Now we can start splitting up the video into the parts that are between the "silent" frames markers.

{% highlight python %}
# Zip the frames together so we'll have a complete list of frames
frames = zip(
    [int(frame*frame_len) for frame in aggregated_frames], 
    [int(frame*frame_len) - 1 for frame in aggregated_frames][1:]
)

# Check if we calculated frames until the end of the file.
# If not, add another frame until the end of the file.
if frames[-1][1] < len(audio_data):
   frames.append((frames[-1][1] + 1, len(audio_data))) 
{% endhighlight %}

That's it, we have everything we need in order to create our new video of Sam in the Citadel.

![waveplot]({{ site.baseurl }}/images/got_remix/sam_in_the_citadel.gif)

## Creating the video mix

Once we have our audio frames, we can use them for creating our new video.

In this part we will convert the audio frames to the video frames, 
split the video to those frames and combine them randomly into a new video.

![hot_py]({{ site.baseurl }}/images/got_remix/hot_py.jpg)

First, lets load the movie using the ```moviepy``` module:

{% highlight python %}
from moviepy.video.io.VideoFileClip import VideoFileClip

video_clip = VideoFileClip(video_file_path)
{% endhighlight %}

Next, we'll go over all of our selected frames and create sub video clip of them:

{% highlight python %}
sub_clips = []
# For each frame, convert the start and end indexes of the frame
# to the location in time in the clip and subclip it
for frame_index, frame in enumerate(frames):
    start_frame, end_frame = frame

    # Convert start and end frame indexes to 
    sub_clip = video_clip.subclip(start_frame / float(sr), end_frame / float(sr))

    # Save each video clip
    sub_clip.write_videofile("resources/frame{0}.mp4".format(frame_index))
    sub_clips.append(sub_clip)
{% endhighlight %}

Now we can check a few examples:

<video width="480" height="320" controls="controls">
  <source src="/images/got_remix/frame10.mp4" type="video/mp4">
</video>

<video width="480" height="320" controls="controls">
  <source src="/images/got_remix/frame15.mp4" type="video/mp4">
</video>

<video width="480" height="320" controls="controls">
  <source src="/images/got_remix/frame9.mp4" type="video/mp4">
</video>

<video width="480" height="320" controls="controls">
  <source src="/images/got_remix/frame5.mp4" type="video/mp4">
</video>

Next step is to mix those sub clips that we created into a new video.
We can create a new video that is longer than the original video by 
combining the same sub clips again and again in a random order.

Lets create a new video clip that is 5 minutes 
(you can create longer clips but for this example, let's keep it short):

{% highlight python %}
# Create a sub clip from frame
def create_sub_clip(video_clip, frame, sr):
    start, stop = frame
    sub_clip = video_clip.subclip(start / float(sr), stop / float(sr))
    return sub_clip

# Save the start and end sub clips and put random clips between them
start_clip = create_sub_clip(video_clip, frames[0], sr)
end_clip = create_sub_clip(video_clip, frames[-1], sr)

max_duration = 300
video_duration = start_clip.duration + end_clip.duration

# This list will hold the sub clips for the final video
sub_clips = []

# This loop will run until our new video duration is over max duration
while True:
    # Select a random frame
    frame = random.choice(frames[1:-1])
    # Create a sub clip from the random frame
    sub_clip = create_sub_clip(video_clip, frame, sr)
    # Calculate the new video duration with this new sub clip
    video_duration += sub_clip.duration
    # If the new duration is longer than
    # the max duration, stop the loop
    if video_duration >= max_duration:
        break
    # Add the new random sub clip to our list of random sub clips
    sub_clips.append(sub_clip)

# Create a list of the random clips and the start and end clips
sub_clips = [start_clip] + sub_clips + [end_clip]

# Concatenate all sub clips into 
concat_clip = concatenate_videoclips(sub_clips, method="compose")

# Output video to file
concat_clip.write_videofile("resources/got_mix.mp4")
{% endhighlight %}

The result will be something like:

<iframe width="420" height="315" src="http://www.youtube.com/embed/hI4Ed9ejEuU" frameborder="0" allowfullscreen></iframe>

That's it, now we have a longer version of sam in the Citadel.

Source code can be found on [github](https://github.com/kazuar/got_remix).

## What's next?

You're probably wondering, what else we can do with this?

Well, I got only one things to say about that:

![OYSTER, CLAMS and COCKLES]({{ site.baseurl }}/images/got_remix/oysters_clams_and_cockles_meme.jpg)

10 hours of OYSTER, CLAMS and COCKLES!
