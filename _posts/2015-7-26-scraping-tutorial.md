---
layout: post
title: How to scrape a website that requires login with Python
comments: true
---

I've recently had to perform some web scraping from a site that required login.
It wasn't very straight forward as I expected so I've decided to write a tutorial for it.

For this tutorial we will scrape a list of projects from our bitbucket account.

The code from this tutorial can be found on my [Github](https://github.com/kazuar/login_scraper_example).

We will perform the following steps:

1. Extract the details that we need for the login
2. Perform login to the site
3. Scrape the required data

For this tutorial, I've used the following packages (can be found in the [requirements.txt](https://github.com/kazuar/login_scraper_example/blob/master/requirements.txt)):
{% highlight bash %}
requests
lxml
{% endhighlight %}

# Step 1: Study the website
## Open the login page
Go to the following page "[bitbucket.org/account/signin](https://bitbucket.org/account/signin/?next=/)" .
You will see the following page (perform logout in case you're already logged in) 
![scrape_login_page]({{ site.baseurl }}/images/scrape_login_page.png)

## Check the details that we need to extract in order to login

In this section we will build a dictionary that will hold our details for performing login:

1. Right click on the "Username or email" field and select "inspect element". We will use the value of the "name" attribue for this input which is "username". "username" will be the key and the our user name / email will be the value (on other sites this might be "email", "user_name", "login", etc.).
![scrape_username]({{ site.baseurl }}/images/scrape_username.png)
![login_source_html]({{ site.baseurl }}/images/login_source_html.png)
2. Right click on the "Password" field and select "inspect element". In the script we will need to use the value of the "name" attribue for this input which is "password". "password" will be the key in the dictionary and our password will be the value (on other sites this might be "user_password", "login_password", "pwd", etc.).
![scrape_password]({{ site.baseurl }}/images/scrape_password.png)
![password_source_html]({{ site.baseurl }}/images/password_source_html.png)
3. In the page source, search for a hidden input tag called "csrfmiddlewaretoken". "csrfmiddlewaretoken" will be the key and value will be the hidden input value (on other sites this might be a hidden input with the name "csrf_token", "authentication_token", etc.). For example "Vy00PE3Ga6aISwKBrPn72SFml00IcUV8".
![scrape_csrf]({{ site.baseurl }}/images/scrape_csrf.png)
![csrf_token_source_html]({{ site.baseurl }}/images/csrf_token_source_html.png)

We will end up with a dict that will look like this:
{% highlight python %}
payload = {
	"username": "<USER NAME>", 
	"password": "<PASSWORD>", 
	"csrfmiddlewaretoken": "Vy00PE3Ga6aISwKBrPn72SFml00IcUV8"
}
{% endhighlight %}

Keep in mind that this is the specific case for this site. While this login form is simple, other sites might require us to check the request log of the browser and find the relevant keys and values that we should use for the login step.

## Perform login to the site

First, we would like to create our [session object](http://docs.python-requests.org/en/latest/user/advanced/). This object will allow us to persist the login session across all our requests.

{% highlight python %}
session_requests = requests.session()
{% endhighlight %}

Second, we would like to extract the csrf token from the web page, this token is used during login.
For this example we are using lxml and xpath, we could have used regular expression or any other method that will extract this data.

{% highlight python %}
login_url = "https://bitbucket.org/account/signin/?next=/"
result = session_requests.get(login_url)

tree = html.fromstring(result.text)
authenticity_token = list(set(tree.xpath("//input[@name='csrfmiddlewaretoken']/@value")))[0]
{% endhighlight %}

Next, we would like to perform the login phase.
In this phase, we send a POST request to the login url. 
We use the payload that we created in the previous step as the data.
We also use a header for the request and add a referer key to it for the same url.

{% highlight python %}
result = session_requests.post(
	login_url, 
	data = payload, 
	headers = dict(referer=login_url)
)
{% endhighlight %}

Now, that we were able to successfully login, we will perform the actual scraping from [bitbucket dashboard page](https://bitbucket.org/dashboard/overview)

{% highlight python %}
url = 'https://bitbucket.org/dashboard/overview'
result = session_requests.get(
	url, 
	headers = dict(referer = url)
)
{% endhighlight %}

In order to test this, let's scrape the list of projects from the bibucket dashboard page.
Again, we will use xpath to find the target elements, clean up the text from new-lines and spaces and print out the results. If everything went OK, the output should be the list of buckets / project that are in your bitbucket account.

{% highlight python %}
tree = html.fromstring(result.content)
bucket_elems = tree.findall(".//span[@class='repo-name']/")
bucket_names = [bucket.text_content.replace("\n", "").strip() for bucket in bucket_elems]

print bucket_names
{% endhighlight %}

That's it.

Full code sample can be found on [Github](https://github.com/kazuar/login_scraper_example).

