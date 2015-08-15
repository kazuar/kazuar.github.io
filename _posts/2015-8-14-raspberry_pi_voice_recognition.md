---
layout: post
title: Controlling your TV with voice and Raspberry Pi
comments: true
---

![raspberry_pi]({{ site.baseurl }}/images/raspberry_pi/raspberry_pi.jpg)

What's cooler than controlling your TV with voice commands?<br/>
**A billion dollars!**

But for this post,<br/>
It's controlling your TV with voice commands through **Raspberry Pi!**<br/>
(and a billion dollars...)

So, down to business.

Full code can be found on [Github](https://github.com/kazuar/raspberrypi_voice_control).

# Hardware requirements:
1. TV set with enabled [HDMI-CEC](https://en.wikipedia.org/wiki/HDMI#CEC) / Anynet+ (Samsung) - You'll have to check your own TV set and make sure that you enable the HDMI-CEC control.
2. Raspberry Pi - I recommend the [CanaKit Raspberry Pi 2](http://www.amazon.com/gp/product/B00G1PNG54). For this tutorial, the Raspberry Pi should be connected to the TV set via HDMI.
3. USB microphone - I used [C-Media microphone USB](http://www.amazon.com/gp/product/B00IR8R7WQ) 

# Install usb microphone on Raspberry Pi

It seems that just connecting the USB microphone is not enough with the Raspberry Pi, you actually have to enable it on the Raspberry Pi OS in order to use it:

1) Edit **/etc/modprobe.d/alsa-base.conf** with your favorite editor 

1.1) Set the value **options snd-usb-audio index=-2** to be **options snd-usb-audio index=0**<br/>
1.2) On the next line add: **options snd_bcm2835 index=1**<br/>

2) Reboot your Raspberry Pi
{% highlight bash %}
sudo reboot
{% endhighlight %}

3) From the Raspberry Pi terminal, run **lsusb**.<br/> Make sure that you can see your device in the list (in my case "C-Media Electronics, Inc. CM108 Audio Controller"):
{% highlight bash %}
pi@raspberrypi ~ $ lsusb
Bus 001 Device 002: ID 0424:9514 Standard Microsystems Corp.
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 003: ID 0424:ec00 Standard Microsystems Corp.
Bus 001 Device 004: ID 148f:5370 Ralink Technology, Corp. RT5370 Wireless Adapter
Bus 001 Device 005: ID 0781:5575 SanDisk Corp.
Bus 001 Device 006: ID 0d8c:013c C-Media Electronics, Inc. CM108 Audio Controller
Bus 001 Device 007: ID 046d:c31c Logitech, Inc. Keyboard K120 for Business
{% endhighlight %}
4) Run "**cat /proc/asound/cards**" and verify that your USB mic is set on device 0:
{% highlight bash %}
pi@raspberrypi ~ $ cat /proc/asound/cards
 0 [Device         ]: USB-Audio - USB PnP Sound Device
                      C-Media Electronics Inc. USB PnP Sound Device at usb-3f980000.usb-1.4, full spe
 1 [ALSA           ]: bcm2835 - bcm2835 ALSA
                      bcm2835 ALSA
{% endhighlight %}

# Install requirements (on the Raspberry Pi)

## Install basics 

{% highlight bash %}
sudo apt-get install libcec-dev build-essential python-dev
{% endhighlight %}

## Install pyaudio

1) Clone the [pyaudio](http://people.csail.mit.edu/hubert/git/pyaudio.git) git repository

{% highlight bash %}
git clone http://people.csail.mit.edu/hubert/git/pyaudio.git
{% endhighlight %}

2) Go to the pyaudio folder and run the install command

{% highlight bash %}
cd pyaudio
sudo python setup.py install
{% endhighlight %}

## Install flac

[Flac](https://en.wikipedia.org/wiki/FLAC) is an audio compression format (like MP3).<br/>
If you don't have it, pyaudio will not be able to run.

{% highlight bash %}
sudo apt-get install flac
{% endhighlight %}

# Install required python modules

Last step before creating our script is to install a few python packages.
I usually like to create [virtual environments](http://docs.python-guide.org/en/latest/dev/virtualenvs/) for these projects.

In your virtual environment install the following packages:

1. [Speechrecognition](https://pypi.python.org/pypi/SpeechRecognition/) - Library for performing speech recognition with the Google Speech Recognition API.
2. [pyaudio](https://people.csail.mit.edu/hubert/pyaudio/) - provides Python bindings for PortAudio, the cross-platform audio I/O library
3. [python cec](https://pypi.python.org/pypi/cec/0.2.3) - Python bindings for libcec. This will be used to control the TV through HDMI.

{% highlight bash %}
pip install Speechrecognition
pip install --allow-external pyaudio --allow-unverified pyaudio pyaudio
pip install cec
{% endhighlight %}

# Creating the script

This is a very simple script, which performs the following steps:

1. Initialize the required components
2. Start running an infinite loop until there's a stop command:
	1. Record audio through Microphone
	2. Translate the audio to text
	3. Check if text contains a command
	4. Perform the required command

### Step 1: Initialize required components

We'll start by importing the required packages:
{% highlight python %}
import cec
import speech_recognition as sr
{% endhighlight %}

We need to define the commands that we want to use. We will want to start with the following commands:

1. Turn the TV on
2. Turn the TV off
3. Close program

{% highlight python %}
TURN_TV_ON = "turn tv on"
TURN_TV_OFF = "turn tv off"
CLOSE_PROGRAM = "close program"
{% endhighlight %}

Initialize CEC control
{% highlight python %}
cec.init()
{% endhighlight %}

Create speech recognition object
{% highlight python %}
r = sr.Recognizer()
{% endhighlight %}

### Step 2: Record and analyze audio

In an infinite loop, record audio through the microphone
{% highlight python %}
with sr.Microphone() as source:
    audio = r.listen(source)
{% endhighlight %}

Translate the audio to text
{% highlight python %}
command = r.recognize(audio)
{% endhighlight %}

Check if the command is to turn the TV **on**
{% highlight python %}
if TURN_TV_ON in command.lower():
	# Get tv device and turn it on
    tv = cec.Device(0)
    tv.power_on()
{% endhighlight %}

Check if the command is to turn the TV **off**
{% highlight python %}
if TURN_TV_OFF in command.lower():
    # Get tv device and turn it off
    tv = cec.Device(0)
    tv.standby()
{% endhighlight %}

Check if the command is to stop the script
{% highlight python %}
if CLOSE_PROGRAM in command.lower():
    # Stop program
    break
{% endhighlight %}

This is how the full code looks like:

{% highlight python %}
import cec
import speech_recognition as sr

TURN_TV_ON = "turn tv on"
TURN_TV_OFF = "turn tv off"
CLOSE_PROGRAM = "close program"

def main():
    # Create cec control
    cec.init()

    # Ceate speech recognizer object
    r = sr.Recognizer()

    # Create infinite loop
    while True:
        # Record sound
        with sr.Microphone() as source:
            print("Recording")
            audio = r.listen(source)

        try:
            # Try to recognize the audio
            command = r.recognize(audio)
            print("Detected speech:{0}".format(command))
            # Check the current command
            if TURN_TV_ON in command.lower():
                # Get tv device and turn it on
                tv = cec.Device(0)
                tv.power_on()
            elif TURN_TV_OFF in command.lower():
                # Get tv device and turn it off
                tv = cec.Device(0)
                tv.standby()
            elif CLOSE_PROGRAM in command.lower():
                # Stop program
                break
        except LookupError:
            # In case of an exception
            print("Could not translate audio")

if __name__ == '__main__':
    main()
{% endhighlight %}

If you run the script, you can say the commands and see if the script output will be able to translate you speech correctly and run the correct command.

That's it!

Full code can be found on [Github](https://github.com/kazuar/raspberrypi_voice_control).

So, what's cooler than controlling your TV with voice commands?<br/>
A billion dollars!

![billion]({{ site.baseurl }}/images/raspberry_pi/billion.jpg)
