The `README.md` file has been successfully fetched. Here are the contents with the required updates:

```markdown
# Hellcat

Hellcat is a script that searches for specific keywords across multiple social media platforms and saves the results to a CSV file.

## Setup

1. Clone the repository:
   ```sh
   git clone https://github.com/Cat404x/Hellcat-.git
   cd Hellcat-
   ```

2. Create a virtual environment and activate it:
   ```sh
   python -m venv venv
   source venv/bin/activate  # On Windows use `venv\Scripts\activate`
   ```

3. Install the dependencies:
   ```sh
   pip install -r requirements.txt
   ```

4. Create a `.env` file based on the `.env.example` and add your API credentials:
   ```env
   TWITTER_API_KEY=your_twitter_api_key
   TWITTER_API_SECRET=your_twitter_api_secret
   TWITTER_ACCESS_TOKEN=your_twitter_access_token
   TWITTER_ACCESS_TOKEN_SECRET=your_twitter_access_token_secret
   REDDIT_CLIENT_ID=your_reddit_client_id
   REDDIT_CLIENT_SECRET=your_reddit_client_secret
   REDDIT_USER_AGENT=your_reddit_user_agent
   GITHUB_TOKEN=your_github_token
   FACEBOOK_ACCESS_TOKEN=your_facebook_access_token
   INSTAGRAM_USERNAME=your_instagram_username
   INSTAGRAM_PASSWORD=your_instagram_password
   ```

## Usage

Run the script:
```sh
python main.py
```

Follow the prompts to enter the keyword and optional username to search for.
```

### Create requirements.txt

Since the `requirements.txt` file does not exist, let's create it with the necessary dependencies:

```txt
aiohttp
requests
tweepy
praw
PyGithub
python-dotenv
facebook-scraper
TikTokApi
instagram-private-api
```

### Create .env.example

```env
TWITTER_API_KEY=your_twitter_api_key
TWITTER_API_SECRET=your_twitter_api_secret
TWITTER_ACCESS_TOKEN=your_twitter_access_token
TWITTER_ACCESS_TOKEN_SECRET=your_twitter_access_token_secret
REDDIT_CLIENT_ID=your_reddit_client_id
REDDIT_CLIENT_SECRET=your_reddit_client_secret
REDDIT_USER_AGENT=your_reddit_user_agent
GITHUB_TOKEN=your_github_token
FACEBOOK_ACCESS_TOKEN=your_facebook_access_token
INSTAGRAM_USERNAME=your_instagram_username
INSTAGRAM_PASSWORD=your_instagram_password
 _    _      _ _           _   
| |  | |    | | |         | |  
| |__| | ___| | | ___ __ _| |_ 
|  __  |/ _ \ | |/ __/ _` | __|
| |  | |  __/ | | (_| (_| | |_ 
|_|  |_|\___|_|_|\___\__,_|\__|

import csv
import asyncio
import aiohttp
import requests
import tweepy
import praw
from github import Github
import os
from dotenv import load_dotenv
from facebook_scraper import get_posts
from TikTokApi import TikTokApi
from instagram_private_api import Client

# Load environment variables
load_dotenv()

# API credentials
TWITTER_API_KEY = os.getenv('TWITTER_API_KEY')
TWITTER_API_SECRET = os.getenv('TWITTER_API_SECRET')
TWITTER_ACCESS_TOKEN = os.getenv('TWITTER_ACCESS_TOKEN')
TWITTER_ACCESS_TOKEN_SECRET = os.getenv('TWITTER_ACCESS_TOKEN_SECRET')
REDDIT_CLIENT_ID = os.getenv('REDDIT_CLIENT_ID')
REDDIT_CLIENT_SECRET = os.getenv('REDDIT_CLIENT_SECRET')
REDDIT_USER_AGENT = os.getenv('REDDIT_USER_AGENT')
GITHUB_TOKEN = os.getenv('GITHUB_TOKEN')
FACEBOOK_ACCESS_TOKEN = os.getenv('FACEBOOK_ACCESS_TOKEN')
INSTAGRAM_USERNAME = os.getenv('INSTAGRAM_USERNAME')
INSTAGRAM_PASSWORD = os.getenv('INSTAGRAM_PASSWORD')

# Asynchronous HTTP request helper
async def fetch(session, url):
    async with session.get(url) as response:
        return await response.json()

def search_twitter(keyword, username=None):
    auth = tweepy.OAuthHandler(TWITTER_API_KEY, TWITTER_API_SECRET)
    auth.set_access_token(TWITTER_ACCESS_TOKEN, TWITTER_ACCESS_TOKEN_SECRET)
    api = tweepy.API(auth)
    results = []
    try:
        if username:
            tweets = api.user_timeline(screen_name=username, count=100)
            for tweet in tweets:
                if keyword.lower() in tweet.text.lower():
                    results.append(f"Twitter: @{tweet.user.screen_name} - {tweet.text}")
        else:
            for tweet in tweepy.Cursor(api.search_tweets, q=keyword).items(100):
                results.append(f"Twitter: @{tweet.user.screen_name} - {tweet.text}")
    except Exception as e:
        print(f"Error searching Twitter: {str(e)}")
    return results

def search_reddit(keyword, username=None):
    reddit = praw.Reddit(client_id=REDDIT_CLIENT_ID,
                         client_secret=REDDIT_CLIENT_SECRET,
                         user_agent=REDDIT_USER_AGENT)
    results = []
    try:
        if username:
            for comment in reddit.redditor(username).comments.new(limit=100):
                if keyword.lower() in comment.body.lower():
                    results.append(f"Reddit: u/{comment.author} - {comment.body}")
        else:
            for submission in reddit.subreddit("all").search(keyword, limit=100):
                results.append(f"Reddit: r/{submission.subreddit.display_name} - {submission.title}")
    except Exception as e:
        print(f"Error searching Reddit: {str(e)}")
    return results

def search_github(keyword, username=None):
    g = Github(GITHUB_TOKEN)
    results = []
    try:
        if username:
            user = g.get_user(username)
            for repo in user.get_repos():
                if repo.description and keyword.lower() in repo.description.lower():
                    results.append(f"GitHub: {repo.full_name} - {repo.description}")
        else:
            repos = g.search_repositories(query=keyword)
            for repo in repos[:100]:
                if repo.description:
                    results.append(f"GitHub: {repo.full_name} - {repo.description}")
    except Exception as e:
        print(f"Error searching GitHub: {str(e)}")
    return results

async def search_facebook(keyword, username=None):
    results = []
    try:
        if username:
            for post in get_posts(username, pages=2):
                if keyword.lower() in post['text'].lower():
                    results.append(f"Facebook: {username} - {post['text'][:100]}...")
        else:
            url = f"https://graph.facebook.com/v12.0/search?q={keyword}&type=post&access_token={FACEBOOK_ACCESS_TOKEN}"
            async with aiohttp.ClientSession() as session:
                data = await fetch(session, url)
                for post in data.get('data', []):
                    results.append(f"Facebook: {post['id']} - {post.get('message', '')[:100]}...")
    except Exception as e:
        print(f"Error searching Facebook: {str(e)}")
    return results

async def search_instagram(keyword, username=None):
    api = Client(INSTAGRAM_USERNAME, INSTAGRAM_PASSWORD)
    results = []
    try:
        if username:
            user_id = api.username_info(username)['user']['pk']
            user_feed = api.user_feed(user_id)
            for post in user_feed.get('items', []):
                if 'caption' in post and keyword.lower() in post['caption']['text'].lower():
                    results.append(f"Instagram: {username} - {post['caption']['text'][:100]}...")
        else:
            tag_feed = api.feed_tag(keyword, rank_token=Client.generate_uuid())
            for post in tag_feed.get('items', [])[:10]:
                if 'caption' in post:
                    results.append(f"Instagram: {post['user']['username']} - {post['caption']['text'][:100]}...")
    except Exception as e:
        print(f"Error searching Instagram: {str(e)}")
    return results

async def search_tiktok(keyword, username=None):
    api = TikTokApi()
    results = []
    try:
        if username:
            user_videos = api.by_username(username, count=20)
            for video in user_videos:
                if keyword.lower() in video['desc'].lower():
                    results.append(f"TikTok: {username} - {video['desc'][:100]}...")
        else:
            trending = api.by_trending(count=20)
            for video in trending:
                if keyword.lower() in video['desc'].lower():
                    results.append(f"TikTok: {video['author']['uniqueId']} - {video['desc'][:100]}...")
    except Exception as e:
        print(f"Error searching TikTok: {str(e)}")
    return results

async def main():
    keyword = input("Enter the keyword to search for: ")
    username = input("Enter a username to search for (optional, press enter to skip): ")

    all_results = []

    platforms = [
        ("Twitter", search_twitter),
        ("Reddit", search_reddit),
        ("GitHub", search_github),
        ("Facebook", search_facebook),
        ("Instagram", search_instagram),
        ("TikTok", search_tiktok)
    ]

    for platform_name, search_function in platforms:
        print(f"\nSearching {platform_name}...")
        try:
            if asyncio.iscoroutinefunction(search_function):
                results = await search_function(keyword, username)
            else:
                results = search_function(keyword, username)
            all_results.extend(results)
        except Exception as e:
            print(f"Error searching {platform_name}: {str(e)}")

    print(f"\nFound {len(all_results)} results:")
    for result in all_results:
        print(result)

    # Save results to CSV
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    filename = f"search_results_{timestamp}.csv"
    with open(filename, 'w', newline='', encoding='utf-8') as file:
        writer = csv.writer(file)
        writer.writerow(["Platform", "User", "Content"])
        for result in all_results:
            platform, content = result.split(": ", 1)
            user, text = content.split(" - ", 1)
            writer.writerow([platform, user, text])
    
    print(f"\nResults saved to {filename}")

if __name__ == "__main__":
    asyncio.run(main())

