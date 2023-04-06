## Exploratory Data Analysing for Persian rap Youtubers

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

<img width="547" alt="Screenshot 2023-04-06 at 14 42 50" src="https://user-images.githubusercontent.com/109058050/230381872-4cc0c56b-03f8-44ea-9bab-d98c2f75be7e.png">

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

<img width="551" alt="Screenshot 2023-04-06 at 14 43 28" src="https://user-images.githubusercontent.com/109058050/230381995-74f0088a-f28d-4304-9828-efd4e2274413.png">


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
comments_data
> a table of "1297 rows × 2 columns"

video_data
> a table of "1297 rows × 13 columns"

and we have turned them into dataframes


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


I aim to enhance the data for future examinations by performing the following tasks:

   - Generating a new column that displays the published date of the video and also indicates the day of the week on which it was published. This information will be beneficial for future analysis.

   - Transforming the format of the video duration from a string representation to seconds.

   - Computing the number of tags for each video.

   - Calculating the ratio of comments and likes per 1000 view count.

   - Determining the length of the title in terms of the number of characters.


```python
# Create publish day (in the week) column
video_df['publishedAt'] =  video_df['publishedAt'].apply(lambda x: parser.parse(x)) 
video_df['pushblishDayName'] = video_df['publishedAt'].apply(lambda x: x.strftime("%A")) 
'''
0       Wednesday
1       Wednesday
2       Wednesday
3         Tuesday
4         Tuesday
          ...    
1292      Tuesday
1293       Monday
1294       Monday
1295       Monday
1296     Saturday
Name: pushblishDayName, Length: 1297, dtype: object
'''
# convert duration to seconds
video_df['durationSecs'] = video_df['duration'].apply(lambda x: isodate.parse_duration(x))
video_df['durationSecs'] = video_df['durationSecs'].astype('timedelta64[s]')

'''
0      0 days 00:11:42
1      0 days 00:00:55
2      0 days 00:18:34
3      0 days 00:00:36
4      0 days 00:29:50
             ...      
1292   0 days 00:04:24
1293   0 days 00:06:59
1294   0 days 00:15:23
1295   0 days 00:11:13
1296   0 days 00:11:19
Name: durationSecs, Length: 1297, dtype: timedelta64[s]
'''

# Add number of tags
video_df['tagsCount'] = video_df['tags'].apply(lambda x: 0 if x is None else len(x))

'''
0       34
1        0
2       25
3        0
4       11
        ..
1292    27
1293    30
1294    33
1295    31
1296    21
Name: tagsCount, Length: 1297, dtype: int64
'''

# Comments and likes per 1000 view ratio
video_df['likeRatio'] = video_df['likeCount']/ video_df['viewCount'] * 1000
video_df['commentRatio'] = video_df['commentCount']/ video_df['viewCount'] * 1000

'''
0        2645000
1       26008000
2       25822000
3       38789000
4       25664000
          ...   
1292    12017000
1293     8650000
1294    16215000
1295    13134000
1296    27702000
Name: viewCount, Length: 1297, dtype: int64
'''

# Title character length
video_df['titleLength'] = video_df['title'].apply(lambda x: len(x))

'''
0       38
1       31
2       44
3       26
4       65
        ..
1292    92
1293    66
1294    71
1295    68
1296    74
Name: titleLength, Length: 1297, dtype: int64
'''

```

## Does the number of likes and comments matter for a video to get more views?

```python
fig, ax =plt.subplots(1,2)
sns.scatterplot(data = video_df, x = "commentCount", y = "viewCount", ax=ax[0])
sns.scatterplot(data = video_df, x = "likeCount", y = "viewCount", ax=ax[1])
```

<img width="548" alt="Screenshot 2023-04-06 at 14 53 24" src="https://user-images.githubusercontent.com/109058050/230384272-4ae98a61-ffcd-4230-8b10-5d4b9c3b1371.png">

Now we will take a look at the correlation if we look at the comment ratio and like ratio instead of the absolute number.


```python
fig, ax =plt.subplots(1,2)
sns.scatterplot(data = video_df, x = "commentRatio", y = "viewCount", ax=ax[0])
sns.scatterplot(data = video_df, x = "likeRatio", y = "viewCount", ax=ax[1])
```

<img width="541" alt="Screenshot 2023-04-06 at 14 54 00" src="https://user-images.githubusercontent.com/109058050/230384411-54cc96af-c84d-423e-9b55-8e822dad179c.png">

## Does the video duration matter for views and interaction (likes/ comments)?

```python
sns.scatterplot(data = video_df, x = "durationSecs", y = "viewCount")
```

<img width="544" alt="Screenshot 2023-04-06 at 14 54 29" src="https://user-images.githubusercontent.com/109058050/230384505-244422df-84a9-46bd-8a6a-d44c0e3d06fa.png">


Now we plot the duration against comment count and like count. It can be seen that actually shorter videos tend to get more likes and comments than very long videos.

```python
fig, ax =plt.subplots(1,2)
sns.scatterplot(data = video_df, x = "durationSecs", y = "commentCount", ax=ax[0])
sns.scatterplot(data = video_df, x = "durationSecs", y = "likeCount", ax=ax[1])
```

<img width="550" alt="Screenshot 2023-04-06 at 14 54 48" src="https://user-images.githubusercontent.com/109058050/230384592-e1b59a7c-94e0-44d6-8b6f-d01f0335e69b.png">


## Does title length matter for views?

There is no clear relationship between title length and views as seen the scatterplot below, but most-viewed videos tend to have average title length of 30-70 characters.

```python
sns.scatterplot(data = video_df, x = "titleLength", y = "viewCount")
```

<img width="544" alt="Screenshot 2023-04-06 at 14 55 11" src="https://user-images.githubusercontent.com/109058050/230384688-b4c17f42-7a80-4eb8-8e11-8af0bb5da580.png">


References/ Resources used:

[1] Youtube API. Avaiable at https://developers.google.com/youtube/v3

[2] Converting video durations to time function. https://stackoverflow.com/questions/15596753/how-do-i-get-video-durations-with-youtube-api-version-3

[3] P. Covington, J. Adams, E. Sargin. The youtube video recommendation system. In Proceedings of the Fourth ACM Conference on Recommender Systems, RecSys '16, pages 191-198, New York, NY, USA, 2016. ACM.



