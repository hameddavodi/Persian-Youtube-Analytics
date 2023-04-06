## Exploratory Data Analysing for Persian Youtubers

Exploratory Data Analysis (EDA) is an essential stage in data analysis, which aids in comprehending the data and obtaining valuable insights from it. It involves summarizing the primary traits of a dataset through visual techniques such as histograms, scatter plots, box plots, etc., to identify patterns, trends, and correlations among variables.

In this project, I plan to explore the following:

  - Learn about Youtube API and how to obtain video data.
  - Does the number of likes and comments matter for a video to get more views?
  - Does the video duration matter for views and interaction (likes/ comments)?
  - Does title length matter for views?
  - How many tags do good performing videos have? What are the common tags among these videos?

Also pre-requisites libraries for this project are:

```python 


import pandas as pd
import numpy as np
from dateutil import parser
import isodate

# Data visualization libraries
import matplotlib.pyplot as plt
import matplotlib.ticker as ticker
import seaborn as sns
sns.set(style="darkgrid", color_codes=True)

# Google API
from googleapiclient.discovery import build

```
## Dataset Collection and API connection:

Step 1: create a project on Google Developers Console.
Step 2: Request an authorization credential (API key).
Step 3: Enable API.
Step 4: Collect channel IDs to start research on them. 


```python 

# Connecting to the DATA v3 API key
# Parameters: 
   # youtube: the build object from googleapiclient.discovery
   # channels_ids: list of channel IDs
    
api_key = 'AIzaSyB-4NIQtecQPbRX7TWKphThkb9_Brh2wL4' 


channel_ids = ['UCJk9mjbfuZ3k7oiWLlbKYFA', # Persian Rap Reaction
               'CB0qQXpkEjVK2_yd5w-Khgw', # RealPelmx
               'UCoOjmdECYvybtOOKtlwvsAw', # RadioActive Zone
               'UCV_bYCslgWyIZWH_Jita9rw', # MA2YAR TV
              ]

youtube = build('youtube', 'v3', developerKey=api_key)



```
Then to collect data from selected channels I can write following functions:

```python 

# Channel stats

def get_channel_stats(youtube, channel_ids):

    all_data = []
    request = youtube.channels().list(
                part='snippet,contentDetails,statistics',
                id=','.join(channel_ids))
    response = request.execute() 
    
    for i in range(len(response['items'])):
        data = dict(channelName = response['items'][i]['snippet']['title'],
                    subscribers = response['items'][i]['statistics']['subscriberCount'],
                    views = response['items'][i]['statistics']['viewCount'],
                    totalVideos = response['items'][i]['statistics']['videoCount'],
                    playlistId = response['items'][i]['contentDetails']['relatedPlaylists']['uploads'])
        all_data.append(data)
    
    return pd.DataFrame(all_data)
    
# Returns:
   # Dataframe containing the channel statistics for all channels in the provided list: title, subscriber count, view count, video count, upload playlist.

# Video IDs
def get_video_ids(youtube, playlist_id):
    
    request = youtube.playlistItems().list(
                part='contentDetails',
                playlistId = playlist_id,
                maxResults = 50)
    response = request.execute()
    
    video_ids = []
    
    for i in range(len(response['items'])):
        video_ids.append(response['items'][i]['contentDetails']['videoId'])
        
    next_page_token = response.get('nextPageToken')
    more_pages = True
    
    while more_pages:
        if next_page_token is None:
            more_pages = False
        else:
            request = youtube.playlistItems().list(
                        part='contentDetails',
                        playlistId = playlist_id,
                        maxResults = 50,
                        pageToken = next_page_token)
            response = request.execute()
    
            for i in range(len(response['items'])):
                video_ids.append(response['items'][i]['contentDetails']['videoId'])
            
            next_page_token = response.get('nextPageToken')
        
    return video_ids

# Vido Details

def get_video_details(youtube, video_ids):

        
    all_video_info = []
    
    for i in range(0, len(video_ids), 50):
        request = youtube.videos().list(
            part="snippet,contentDetails,statistics",
            id=','.join(video_ids[i:i+50])
        )
        response = request.execute() 

        for video in response['items']:
            stats_to_keep = {'snippet': ['channelTitle', 'title', 'description', 'tags', 'publishedAt'],
                             'statistics': ['viewCount', 'likeCount', 'favouriteCount', 'commentCount'],
                             'contentDetails': ['duration', 'definition', 'caption']
                            }
            video_info = {}
            video_info['video_id'] = video['id']

            for k in stats_to_keep.keys():
                for v in stats_to_keep[k]:
                    try:
                        video_info[v] = video[k][v]
                    except:
                        video_info[v] = None

            all_video_info.append(video_info)
            
    return pd.DataFrame(all_video_info)
# Comments
def get_comments_in_videos(youtube, video_ids):

    all_comments = []
    
    for video_id in video_ids:
        try:   
            request = youtube.commentThreads().list(
                part="snippet,replies",
                videoId=video_id
            )
            response = request.execute()
        
            comments_in_video = [comment['snippet']['topLevelComment']['snippet']['textOriginal'] for comment in response['items'][0:10]]
            comments_in_video_info = {'video_id': video_id, 'comments': comments_in_video}

            all_comments.append(comments_in_video_info)
            
        except: 
            # When error occurs - most likely because comments are disabled on a video
            print('Could not get comments for video ' + video_id)
        
    return pd.DataFrame(all_comments) 
```
To get channel statistics:

```python 

channel_data = get_channel_stats(youtube, channel_ids)
channel_data

```
This code produce the following outcome:
`
 	channelName 	subscribers 	views 	totalVideos 	playlistId
0 	Persian Rap Reaction 	41500 	18096597 	914 	UUJk9mjbfuZ3k7oiWLlbKYFA
1 	MA2YAR TV 	9030 	623573 	47 	UUV_bYCslgWyIZWH_Jita9rw
2 	RadioActive Zone 	131000 	58079956 	1297 	UUoOjmdECYvybtOOKtlwvsAw
`

Some Other type changes:

```python
# Convert count columns to numeric columns
numeric_cols = ['subscribers', 'views', 'totalVideos']
channel_data[numeric_cols] = channel_data[numeric_cols].apply(pd.to_numeric, errors='coerce')


sns.set(rc={'figure.figsize':(10,8)})
ax = sns.barplot(x='channelName', y='subscribers', data=channel_data.sort_values('subscribers', ascending=False))
ax.yaxis.set_major_formatter(ticker.FuncFormatter(lambda x, pos: '{:,.0f}'.format(x/1000) + 'K'))
plot = ax.set_xticklabels(ax.get_xticklabels(),rotation = 90)



ax = sns.barplot(x='channelName', y='views', data=channel_data.sort_values('views', ascending=False))
ax.yaxis.set_major_formatter(ticker.FuncFormatter(lambda x, pos: '{:,.0f}'.format(x/1000) + 'K'))
plot = ax.set_xticklabels(ax.get_xticklabels(),rotation = 90)
```
and creating dataframes:

```python
# Create a dataframe with video statistics and comments from all channels

video_df = pd.DataFrame()
comments_df = pd.DataFrame()

for c in channel_data['channelName'].unique():
    print("Getting video information from channel: " + c)
    playlist_id = channel_data.loc[channel_data['channelName']== c, 'playlistId'].iloc[0]
    video_ids = get_video_ids(youtube, playlist_id)
    
    # get video data
    video_data = get_video_details(youtube, video_ids)
    # get comment data
    comments_data = get_comments_in_videos(youtube, video_ids)

    # append video data together and comment data toghether
    video_df = pd.concat([video_data], ignore_index=True)
    comments_df = pd.concat([comments_data], ignore_index=True)
```
Note: In this step we could use `video_df = video_df.append(video_data, ignore_index=True)`, but in some cases I got an error and then I used concat to tackle the problem.

So to prevent further use of quotas of API key lets save data in csv files:

```python
# Write video data to CSV file for future references
video_df.to_csv('video_data_top10_channels.csv')
comments_df.to_csv('comments_data_top10_channels.csv')
```

In this step, we convert these count columns into integer.

```python
cols = ['viewCount', 'likeCount',  'commentCount']
video_df[cols] = video_df[cols].apply(pd.to_numeric, errors='coerce', axis=1)
```


I want to enrich the data for further analyses, for example:

  -  create published date column with another column showing the day in the week the video was published, which will be useful for later analysis.

  -  convert video duration to seconds instead of the current default string format

  -  calculate number of tags for each video

  -  calculate comments and likes per 1000 view ratio

  -  calculate title character length
```python
# Create publish day (in the week) column
video_df['publishedAt'] =  video_df['publishedAt'].apply(lambda x: parser.parse(x)) 
video_df['pushblishDayName'] = video_df['publishedAt'].apply(lambda x: x.strftime("%A")) 

# convert duration to seconds
video_df['durationSecs'] = video_df['duration'].apply(lambda x: isodate.parse_duration(x))
video_df['durationSecs'] = video_df['durationSecs'].astype('timedelta64[s]')

# Add number of tags
video_df['tagsCount'] = video_df['tags'].apply(lambda x: 0 if x is None else len(x))

# Comments and likes per 1000 view ratio
video_df['likeRatio'] = video_df['likeCount']/ video_df['viewCount'] * 1000
video_df['commentRatio'] = video_df['commentCount']/ video_df['viewCount'] * 1000

# Title character length
video_df['titleLength'] = video_df['title'].apply(lambda x: len(x))

```

## Does the number of likes and comments matter for a video to get more views?

```python
fig, ax =plt.subplots(1,2)
sns.scatterplot(data = video_df, x = "commentCount", y = "viewCount", ax=ax[0])
sns.scatterplot(data = video_df, x = "likeCount", y = "viewCount", ax=ax[1])
```


Now we will take a look at the correlation if we look at the comment ratio and like ratio instead of the absolute number.


```python
fig, ax =plt.subplots(1,2)
sns.scatterplot(data = video_df, x = "commentRatio", y = "viewCount", ax=ax[0])
sns.scatterplot(data = video_df, x = "likeRatio", y = "viewCount", ax=ax[1])
```

## Does the video duration matter for views and interaction (likes/ comments)?

```python
sns.scatterplot(data = video_df, x = "durationSecs", y = "viewCount")
```


Now we plot the duration against comment count and like count. It can be seen that actually shorter videos tend to get more likes and comments than very long videos.
```python
fig, ax =plt.subplots(1,2)
sns.scatterplot(data = video_df, x = "durationSecs", y = "commentCount", ax=ax[0])
sns.scatterplot(data = video_df, x = "durationSecs", y = "likeCount", ax=ax[1])
```


## Does title length matter for views?



There is no clear relationship between title length and views as seen the scatterplot below, but most-viewed videos tend to have average title length of 30-70 characters.

```python
sns.scatterplot(data = video_df, x = "titleLength", y = "viewCount")
```



References/ Resources used:

[1] Youtube API. Avaiable at https://developers.google.com/youtube/v3

[2] Converting video durations to time function. https://stackoverflow.com/questions/15596753/how-do-i-get-video-durations-with-youtube-api-version-3

[3] P. Covington, J. Adams, E. Sargin. The youtube video recommendation system. In Proceedings of the Fourth ACM Conference on Recommender Systems, RecSys '16, pages 191-198, New York, NY, USA, 2016. ACM.



