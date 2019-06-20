---
title: "Love Island Sentiment Analysis"
date: 2019-05-05
tags: [sentiment-analysis, twitter]
header:
  image:"../assets/images/love_island/loveisland_banner.png"
excerpt: "Using the witter API to see how we feel about Love Island"
toc_label: "Love Island Sentiment Analysis"
toc_icon: "umbrella-beach"  #  Font Awesome icon name (without fa prefix)

---

In this article we will use the twitter API to gauge sentiment about love island. As well as using the Twitter API we will also use [TextBlob](https://textblob.readthedocs.io/en/dev/quickstart.html) to parse our selected tweets and use the natural language processing tools in the package. Firstly lets import the packages we will be using including some visualisation libraries. 

```python
import tweepy
from textblob import TextBlob
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import datetime
```

# Twitter API

To make use of the twitter API you must register a developer account which can be linked to your personal account. To do so use this [link](https://apps.twitter.com/) and follow the instructions, the process shouldn't take too long. After you have successfully registered, we need to define our API Keys:

```python
# Define API keys
consumer_key = 'your-consumer_key'
consumer_secret = 'your-consumer_secret'
access_token = 'your-access_token'
access_token_secret = 'your-access_token_secret'
```

and authenticate our connection to the API:

```python
# Authenticate your application
auth = tweepy.OAuthHandler(consumer_key, consumer_secret)
auth.set_access_token(access_token, access_token_secret)
api = tweepy.API(auth)
```

Now we have successfully connected to the twitter API we are ready to collect some tweets. 

# Sentiment Analysis

Before we start asking twitter for some tweets, maybe now os a good time to talk about what sentiment analysis is. 

MENTION MORE LOVE ISLAND

## Sentiment

## Subjectivity



TextBlob is one implementation of this and is built on the Natural Language Toolkit [library](http://www.nltk.org) 

# TextBlob

The TextBlob documentation provides some useful code examples with many more elsewhere online. We are able to write a function to get the sentiment scores from a string. The input will be tweets for us.

```python
def get_sentiment_scores(search_string):
    '''Takes in twitter search term, outputs array in the format:
     [ [sentiment_score],[subjectivity_score] ]'''
    sent_for_term = []
    subj_for_term = []
    # Use tweepy to search the string, returns 100 score pairs
    public_tweets = api.search(q=search_string, count=100)

    for tweet in public_tweets:
        # TextBlob returns tuple
        analysis = TextBlob(tweet.text)
        # Store the sentiment and subjectivity scores
        sent_for_term.append(analysis.sentiment[0])
        subj_for_term.append(analysis.sentiment[1])

    score_collect = ([sent_for_term, subj_for_term])
    return (score_collect)
```

The search string is the term we're looking for, it would be the term you would type into the twitter search bar on the web client. The `public_tweets` variable holds 100 tweets and the corresponding meta data: very messy. So we use TextBlob (which has it's own twitter methods) to parse these messy tweets into what we want, the `tweet.text`! Now we have the text, we use the TextBlob sentiment method to get the sentiment and subjectivity of each tweet. 

## Plotting

To plot our results we may use the seaborn joint plot. This will allow us to plot the sentiment against the subjectivity. This function calls the previous get sentiment function to get the sentiment scores.

```python
    '''Takes search terms, passed to obtain scores, then prints a joint plot'''
    for item in search_terms:
        # Call to the function to query tweepy and store the scores
        scores = get_sentiment_scores(item)
        # Zip the sentiment and subjectivity ready to plot
        x_coords, y_coords = zip(np.array(scores))

        h = sns.jointplot(x_coords, y_coords, kind="kde", space=0, color="navy")
        h.ax_joint.set_xlabel('Sentiment Score')
        h.ax_joint.set_ylabel('Subjectivity Score')
        h.fig.suptitle(f'{now.strftime("%Y-%m-%d %H:%M")} Search item: {item}')
        plt.show()
```

```python
print_analysis("#boris")
```

​								 ![boris_sentiment](/Users/yusufsohoye/quotennial.github.io/assets/images/love_island/boris_sentiment.png)

Using our `print_analysis` function we are able to search the  top 100 tweets using the #boris, and the joint plot uncovers some interesting results. The bottom dark blue spot indicates a fairly neutral tweet, we can interpret this as fact based, maybe news reports and updates. As the subjectivity increases we can see the sentiment splits, these more opinion based tweets can give us an insight into how the twittersphere is feeling about this PM candidate. 

# Sentiment through time

By altering the `print_analysis` function to store the sentiment scores, we are able to track sentiment through time. the function now appends the sentiment scores to a csv file. 

```python
def append_analysis(*search_terms):
    '''Takes search terms, passed to obtain scores, then prints a joint plot'''
    for item in search_terms:
        # Call to the function to query tweepy and store the scores
        scores = get_sentiment_scores(item)
        append_list = scores[0]
        #append the info at the start
        append_list.insert(0,now.strftime("%Y-%m-%d %H:%M"))
        append_list.insert(0,item)
        with open('loveisland_sentiment.csv', 'a') as csvFile:
            writer = csv.writer(csvFile)
            writer.writerow(append_list)
    csvFile.close()
```

We can use a list of names in the villa currently in a `for` loop to store each score. 