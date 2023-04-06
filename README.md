# Youtube-Analytics

This article is about utilizing the YouTube Data API to obtain data from the platform and analyze it. The author will explain what the API is and how to configure it, followed by creating a Python script to gather data from YouTube channels. Finally, they will examine the collected data and answer questions about YouTube.

The latest version of the YouTube API is DATA API v3. It offers better control over applications, improved performance, and stability. To start, developers need to set up their development environment by installing Python 3 and the Google APIs Client Library for Python. The latter allows interaction with YouTube's Data API. Users can install this library by entering a command in the terminal.


`pip install --upgrade google-api-python-client`

Tip: if you get error of module not found try different environments and pip3 along with google documentations to upgrade lib.

Now that our development environment is ready, we can begin coding. The initial step is to acquire a developer key from Google, as it is necessary to use YouTube's Data API. To obtain a developer key, you must create a new project in the Google Developers Console and activate the YouTube Data API v3 for that particular project. After receiving the developer key, we can initiate the Google APIs Client Library for Python using the following method:

```python
from googleapiclient.discovery import build 
DEVELOPER_KEY = 'YOUR_DEVELOPER_KEY
```
