##General Information

All data pulls use the hashtags #wimbledon and #federer, as retweets were previously discarded in assignment 2. 

##1 - Storing Tasks
*1-1*

Program collects tweets and writes them to files in 5000 tweet chunks. Using the AppAuthHandler instead of the OAuthHandler bumps the rate limit for the Search function of the REST API up to 450/15 mins in batches of 100 tweets. I do not implement using tweepy's cursor class here due to a known memory leak.

*1-2*

Data was gathered from the Steaming API using the program from Assignment 2 during the Wimbledon finals. Roughly 378k tweets were acquired between 7/11/15 and 7/13/15. Tweets were then uploaded to S3, and redownloaded to be put in a mongo collection.

##2 - Retrieving and Analyzing
*2-1*

Top retweeters were determined by aggregating retweets based on the original tweet ID, then sorting using mongodb. The list of original tweeters users was created by iterating down the list of most retweeted tweets until 40 unique users were found with fewer than 300k followers. This threshold was chosen for practical reasons, such that scraping the list of followers from twitter would take at most one hour at 5000 followers * (15 requests / 15 mins)*  (4 * 15 mins/ hr). I felt kind of dirty doing this, so I tossed in another 10 users for good measure. This code segment outputs the list of top tweeters, their location if one is found, and then the number of times they were retweeted within the data collection window.

*2-2*

Tweets were pulled from db_restT using an aggregate function based on user ID. The text for that user was collected into a single document and passed through nltk to look for real words. Then the unique words were counted and the total words, and all that good information was dumped back into MongoDB. The returned set of documents exceeds 16MB, which requires passing the allowDiskUse option to the aggregate function. This step requires MongoDB 2.6+ since lower versions don't have it. The actual nltk step should be easily paralellizable, but it's still pretty fast and I didn't care much for the headache.  
A single user was neglected from the analysis ('Bosch DIY & Garden') due to not having tweeted during the collection period of db_restT, but being present exclusively as a retweet in the Streaming API collection phase. Thus they had no words, and therefore no lexical diversity.


*2-3*

Still feeling vaguely guilty for filtering down to users with <300k followers, I ran this on all 40 users for a total of roughly 3M followers at each timepoint (6M total). For each user, I put the week-separated followers into two separate sqlite databases in memory. To find the difference between followers I performed a left sided join across week 0 and week 1, and looked to see which users were not paired on the right. 
The dropped users are placed in a json object and flushed to disk as 2-3.dump

**format**
{user.id:[unfollower1,unfollower2,unfollower3, ...]}


*2-4*

not done

##3 - Storing and Retrieving
Since I'm turning everything in using an iPython notebook, it seemed kind of silly to add command line arguments. Instead, toggle the mode of the program using the "mode" variable. It would probably be more efficient to just use the builtin mongobackup and mongorestore functions from the command line.
Backups are stored in S3:

https://s3.amazonaws.com/mids-w205-mak-a3-db-backups/db_tweets
https://s3.amazonaws.com/mids-w205-mak-a3-db-backups/db_restT
