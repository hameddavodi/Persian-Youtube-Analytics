# Youtube-Analytics

In this repository, I will be discussing how to use the YouTube Data API to collect data from YouTube and then analyze that data to glean insights about YouTube as a platform. I will first go over what the YouTube Data API is and how to set it up. Then, I will write a Python script to collect data from YouTube channels. Finally, I will analyze the collected data to answer some questions about YouTube.

The DATA API v3 is the latest version of YouTube’s Application Programming Interface. This newer version gives developers more control over their applications and provides better performance and stability. 
First, we need to set up our development environment. We will be using Python 3 for this project. If you don’t have Python 3 installed on your system, you can download it from the official Python website. We also need to install the Google APIs Client Library for Python. This library will allow us to interact with YouTube’s Data API. To install this library, run the following command in your terminal:

`pip install --upgrade google-api-python-client`

Tip: if you get error of module not found try different environments and pip3 along with google documentations to upgrade lib.

With our development environment set up, we can now start writing some code. The first thing we need to do is get a developer key from Google. A developer key is required in order to useYouTube’s Data API. You can get a developer key by creating a new project in theGoogle Developers Console and then enabling the YouTube Data API v3 for that project. Once you have obtained a developer key, we can initialize the Google APIs Client Library for Python like so:
```python
from googleapiclient.discovery import build 
DEVELOPER_KEY = 'YOUR_DEVELOPER_KEY
```
