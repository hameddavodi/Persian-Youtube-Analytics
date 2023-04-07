
video_df.drop(columns=['video_id','title','description','tags','favouriteCount','duration','definition','caption','tagsCount','likeRatio','commentRatio','titleLength'],inplace=True)

video_df.columns


regression Anaysis
The regression model I have developed includes 'publishedAt', 'likeCount', 'commentCount', 'pushblishDayName', and 'durationSecs' as independent variables, while the dependent variable is 'viewCount'. The purpose of this model is to predict the number of views a given video on a platform such as YouTube might receive based on these inputs. 'publishedAt' refers to the date and time the video was uploaded, 'likeCount' tracks the number of likes received by the video, 'commentCount' records the number of comments posted, 'pushblishDayName' indicates the day of week the video was published, and 'durationSecs' measures the length of the video in seconds. By analyzing how these factors relate to 'viewCount', we can gain insights into what makes a video more or less popular and use this information to improve our content strategy.

This is a simple cross-sectional regression. Cross-sectional data refers to a type of data that is collected from a sample of individuals at a specific point in time, rather than over a period of time. In other words, it provides a snapshot of a particular population or group at a given moment. For example, if we collect data on 'viewCount', 'publishedAt', 'likeCount', 'commentCount', 'pushblishDayName', and 'durationSecs' for a set of YouTube videos that were published in the month of March 2023, then this would be considered cross-sectional data. This type of data can be useful for studying trends or patterns within a certain population at a single point in time, but it does not allow for the analysis of changes or developments over time.


x=video_df['publishedAt', 'likeCount', 'commentCount', 'pushblishDayName', 'durationSecs']

y = video_df['viewCount']

model = sm.OLS(y, sm.add_constant(X))
results = model.fit()

print(results.summary())

