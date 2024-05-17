import tweepy
import requests
from bs4 import BeautifulSoup
import time

# X APIの認証情報を設定
bearer_token = '    send  '

# Tweepyの認証
client = tweepy.Client(bearer_token=bearer_token)

# PubMedからデータを取得する関数
def get_pubmed_data():
    url = "https://pubmed.ncbi.nlm.nih.gov/?term=vestibular rehabilitation"
    response = requests.get(url)
    soup = BeautifulSoup(response.text, 'html.parser')
    articles = soup.find_all('article', class_='full-docsum')
    summaries = []
    for article in articles[:5]:  # 上位5件の要約を取得
        title = article.find('a', class_='docsum-title').text.strip()
        article_url = "https://pubmed.ncbi.nlm.nih.gov" + article.find('a', class_='docsum-title')['href']
        summaries.append(f"Title: {title}\n{article_url}")
    return summaries

# 自動ツイートする関数
def auto_tweet():
    summaries = get_pubmed_data()
    for summary in summaries:
        try:
            client.create_tweet(text=summary)
        except tweepy.TweepyException as e:
            print(e)
        time.sleep(7200)  # 2時間待つ

# 実行
if __name__ == "__main__":
    while True:
        auto_tweet()
        time.sleep(86400)  # 24時間待つ
