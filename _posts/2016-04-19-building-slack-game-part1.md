---
layout: post
title: Writing a Slack game with Python - part 1
comments: true
---

![slack_bot]({{ site.baseurl }}/images/slack_bot/slack_bot.jpeg)

In this series of posts I'm going to describe how to build a slack bot that will play a simple game.

In this first part we will create our slack bot and use it to send and receive simple messages.

There are multiple packages out there that we can use for our task such as [python-rtmbot](https://github.com/slackhq/python-rtmbot) or [slackbot](https://github.com/lins05/slackbot).
However, for this tutorial I wanted to use the basic [Slack Real Time Messaging API](https://github.com/slackhq/python-slackclient).

The code from this tutorial can be found on my [Github](https://github.com/kazuar/slack_bot_tutorial).

# Step 1: Create a slack account

If you don't have one already, create one at [slack.com](http://slack.com) and start using it.
... or go read something else :)

# Step 2: Regsiter a slack bot

1. Go to [https://slack.com/apps/build](https://slack.com/apps/build) and click on **"Make a Custom Integration"** under **"Something just for my team"**
![]({{ site.baseurl }}/images/slack_bot/screen1.png)
2. In the new [page](https://wat-team.slack.com/apps/build/custom-integration), click on **"Bots"**
![]({{ site.baseurl }}/images/slack_bot/screen2.png)
3. Choose your new slack bot name (i.e. *slack_bot*) and press **"Add bot integration"**
![]({{ site.baseurl }}/images/slack_bot/screen3.png)
4. In the new page, copy the API token of the new slack bot as we will need it in our script.
![]({{ site.baseurl }}/images/slack_bot/screen4.png)

# Step 2: Create the environment

1. Create a folder for the project

    ```bash
    mkdir slack_bot
    cd slack_bot
    ```

2. Create a virtual environment for our project

    ```bash
    mkvirtualenv slack_bot
    ```

3. Install the following requirements

    ```bash
    pip install slackclient
    ```

4. Create a new file for our script, let's call it **"slack_bot.py"**

# Step 3: Send a simple message with the slack bot

In the new file, we'll start by importing the slackclient package:

```python
from slackclient import SlackClient
```

Create the slack bot instance with the token we created earlier:

```python
sc = SlackClient(BOT_TOKEN)
```

Once we initialize the instance we'll need to perform the connect action:

```python
sc.rtm_connect()
```

Now we can send a simple message to the #general channel:

```python
sc.rtm_send_message("general", "I'm ALIVE!!!")
```

# Step 4: Interact with the slack bot

We'll make the slack bot respond to every message on our **#general** channel.

Every time someone writes something, the slack bot will repond with the message **"@*username* wrote something..."**

Annoying, I know...

We'll perform this by running an infinite loop and making our slack bot go over all the messages that go through our slack channel.
The slack bot will read the message using the **rtm_read** method which returns a list of messages.
The script will iterate through them and send the annoying message accordingly.
Between these, the bot will sleep for half a second every cycle of the loop.

```python
import time

while True:
    for slack_message in sc.rtm_read():
        message = slack_message.get("text")
        user = slack_message.get("user")
        if not message or not user:
            continue
        sc.rtm_send_message("general", "<@{}> wrote something...".format(user))
    # Sleep for half a second
    time.sleep(0.5)
```

The full script would look like this:

```python

import time
from slackclient import SlackClient

BOT_TOKEN = "<SLACK_BOT_TOKEN>"
CHANNEL_NAME = "general"

def main():
    # Create the slackclient instance
    sc = SlackClient(BOT_TOKEN)

    # Connect to slack
    if sc.rtm_connect():
        # Send first message
        sc.rtm_send_message(CHANNEL_NAME, "I'm ALIVE!!!")

        while True:
            # Read latest messages
            for slack_message in sc.rtm_read():
                message = slack_message.get("text")
                user = slack_message.get("user")
                if not message or not user:
                    continue
                sc.rtm_send_message(CHANNEL_NAME, "<@{}> wrote something...".format(user))
            # Sleep for half a second
            time.sleep(0.5)

if __name__ == '__main__':
    main()
```

The code from this tutorial can be found on my [Github](https://github.com/kazuar/slack_bot_tutorial).

# Conclusions

In this tutorial we learned how to create a slack bot and perform a simple action.

Next time, we'll create a simple game using this slack bot.
