import snscrape
import pandas as pd
from pymongo import MongoClient
import streamlit as st

# Scrape data from Twitter using snscrape library
def scrape_data(keyword, date_range, tweet_limit):
    tweets = snscrape.modules.Twitter.TwitterSearchScraper(keyword, limit=tweet_limit).get_items()
    data = []
    for tweet in tweets:
        data.append({
            'date': tweet.time,
            'id': tweet.id,
            'url': tweet.url,
            'content': tweet.content,
            'user': tweet.user.name,
            'reply_count': tweet.replies,
            'retweet_count': tweet.retweets,
            'language': tweet.language,
            'source': tweet.source,
            'like_count': tweet.likes
        })
    # Create a dataframe with the scraped data
    df = pd.DataFrame(data)
    return df

# Store data in MongoDB
def store_data(df, keyword):
    client = MongoClient()
    db = client['twitter_scraped_data']
    collection_name = keyword + "_" + str(pd.Timestamp.now())
    db[collection_name].insert_many(df.to_dict('records'))

# Create a GUI using Streamlit
def main():
    st.title("Twitter Data Scraper")
    keyword = st.text_input("Enter keyword or hashtag to search for:")
    date_range = st.date_range_input("Select date range:")
    tweet_limit = st.number_input("Enter limit for number of tweets to scrape:")
    if st.button("Scrape"):
        df = scrape_data(keyword, date_range, tweet_limit)
        st.dataframe(df)
        if st.button("Store in MongoDB"):
            store_data(df, keyword)
            st.success("Data stored in MongoDB")
        if st.button("Download as CSV"):
            csv = df.to_csv()
            b64 = base64.b64encode(csv.encode()).decode()  # some strings <-> bytes conversions necessary here
            href = f'<a href="data:file/csv;base64,{b64}" download="{keyword}_tweets.csv">Download CSV File</a>'
            st.markdown(href, unsafe_allow_html=True)
        if st.button("Download as JSON"):
            json = df.to_json()
            b64 = base64.b64encode(json.encode()).decode()  # some strings <-> bytes conversions necessary here
            href = f'<a href="data:file/json;base64,{b64}" download="{keyword}_tweets.json">Download JSON File</a>'
            st.markdown(href, unsafe_allow_html=True)

if __name__ == '__main__':
    main()
