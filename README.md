YouTube Data Scraping, Preprocessing and Analysis using Python
Introduction

YouTube is one of the largest video-sharing platforms, hosting millions of videos across various categories and audiences. Analyzing YouTube data can provide valuable insights into content performance, viewer engagement, and emerging trends.

In this project, we extract data from a real YouTube channel and perform data scraping, preprocessing, text processing, and visualization using Python. The goal is to identify patterns and present meaningful insights from the collected data.

1. Web Scraping
What is Web Scraping?

Web scraping is the process of automatically extracting data from websites. It allows us to collect useful information such as titles, views, durations, or comments for analysis.

In this project, we scrape video information from a YouTube channel using Python libraries:

Requests – for sending HTTP requests

BeautifulSoup – for parsing HTML content

JSON – for processing structured data

XlsxWriter – for saving data into Excel files

The data is collected from the following YouTube channel:

https://www.youtube.com/c/GeeksforGeeksVideos/videos
1.1 Installing Required Libraries

The required Python libraries are installed using pip.

!pip install requests beautifulsoup4 xlsxwriter --quiet
1.2 Importing Libraries

The following libraries are imported to perform web scraping and data handling.

import requests
from bs4 import BeautifulSoup
import json
import xlsxwriter
1.3 Sending HTTP Request to YouTube Page

An HTTP request is sent to the YouTube channel page.
A User-Agent header is added to simulate a real browser request and prevent blocking.

url = "https://www.youtube.com/c/GeeksforGeeksVideos/videos"
headers = {"User-Agent": "Mozilla/5.0"}

response = requests.get(url, headers=headers)
html = response.text
1.4 Creating the BeautifulSoup Object

The HTML response is parsed using BeautifulSoup, which helps navigate the webpage structure and extract required data.

soup = BeautifulSoup(html, "html.parser")
1.5 Extracting Embedded YouTube JSON Data

YouTube loads video information dynamically using JavaScript.
The data is stored in a JSON object called ytInitialData, which is extracted from the page source.

script_tag = soup.find("script", text=lambda t: t and "var ytInitialData = " in t)
json_text = script_tag.string.strip()[len("var ytInitialData = "):-1]
data = json.loads(json_text)
1.6 Navigating to Video Metadata

The JSON structure contains multiple nested objects.
We navigate through it to locate the video metadata section.

videos_data = data['contents']['twoColumnBrowseResultsRenderer']['tabs'][1]\
              ['tabRenderer']['content']['richGridRenderer']['contents']
1.7 Extracting Titles, Views, and Durations

Each video entry is processed in a loop to extract:

Video Title

View Count

Video Duration

Only the latest 30 videos are collected.

titles, views, durations = [], [], []
count = 0

for item in videos_data:
    try:
        video = item['richItemRenderer']['content']['videoRenderer']
        titles.append(video['title']['runs'][0]['text'])
        views.append(video.get('viewCountText', {}).get('simpleText', 'N/A'))
        durations.append(video.get('lengthText', {}).get('simpleText', 'N/A'))
        count += 1
        if count >= 30:
            break
    except:
        continue
1.8 Creating the Excel File

An Excel workbook is created to store the scraped data.

workbook = xlsxwriter.Workbook("youtube_videos.xlsx")
sheet = workbook.add_worksheet()

sheet.write(0, 0, "Title")
sheet.write(0, 1, "Views")
sheet.write(0, 2, "Duration")
1.9 Writing Data to Excel Sheet

The extracted data is written into the Excel sheet row by row.

for i in range(len(titles)):
    sheet.write(i+1, 0, titles[i])
    sheet.write(i+1, 1, views[i])
    sheet.write(i+1, 2, durations[i])
1.10 Saving the File

The Excel file is saved after writing the data.

workbook.close()
print("Scraped latest 30 videos successfully! Saved as youtube_videos.xlsx")
2. Data Preprocessing

Scraped data often contains text values, inconsistencies, and formatting issues. Data preprocessing cleans and standardizes the dataset for further analysis.

2.1 Importing the Data

The Excel file is loaded into a Pandas DataFrame.

import pandas as pd

data = pd.read_excel("youtube_videos.xlsx")
data.head()
2.2 Cleaning the Views Column

The Views column contains text values such as:

12K views
1,200 views

Steps performed:

Remove the word "views"

Remove commas

Convert K values into numeric values

Convert strings to numeric values

Handle missing values

data['Views'] = data['Views'].str.replace(" views", "", regex=False).str.strip()

cleaned_views = []

for i in data['Views']:
    if pd.isna(i):
        cleaned_views.append(None)
        continue

    i = str(i).replace(",", "")

    if i.endswith('K') or i.endswith('k'):
        i = i.replace('K', '').replace('k', '')
        try:
            cleaned_views.append(float(i) * 1000)
        except:
            cleaned_views.append(None)
    else:
        try:
            cleaned_views.append(float(i))
        except:
            cleaned_views.append(None)

data['Views'] = cleaned_views
2.3 Cleaning the Duration Column

The duration values are converted into total seconds.

Steps performed:

Remove newline characters

Convert time format into seconds

Handle invalid values like SHORTS or N/A

data['Duration'] = data['Duration'].str.replace("\n", "", regex=False)

def duration_to_seconds(duration_str):
    if pd.isna(duration_str) or duration_str in ['SHORTS', 'N/A']:
        return None

    parts = str(duration_str).split(':')

    if len(parts) == 3:
        h, m, s = map(int, parts)
        return h * 3600 + m * 60 + s

    elif len(parts) == 2:
        m, s = map(int, parts)
        return m * 60 + s

    return None

data['Duration'] = data['Duration'].apply(duration_to_seconds)
2.4 Categorizing Videos by Duration

Videos are categorized into groups based on duration.

Duration	Category
Less than 15 minutes	Mini-Videos
15 minutes – 1 hour	Long-Videos
More than 1 hour	Very-Long-Videos
for i in data.index:
    val = data.loc[i, 'Duration']

    if val is None:
        continue
    elif val < 900:
        data.loc[i, 'Duration'] = 'Mini-Videos'
    elif val < 3600:
        data.loc[i, 'Duration'] = 'Long-Videos'
    else:
        data.loc[i, 'Duration'] = 'Very-Long-Videos'
3. Text Preprocessing

Text preprocessing prepares textual data such as video titles for analysis.

Steps include:

Lowercasing text

Removing URLs

Removing special characters

Tokenization

Stopword removal

Stemming

3.1 Importing Libraries
import re
from tqdm import tqdm
import nltk

nltk.download('punkt')
nltk.download('stopwords')
nltk.download('punkt_tab')

from nltk.corpus import stopwords
from nltk.tokenize import word_tokenize
from nltk.stem.porter import PorterStemmer
3.2 Initializing Stopwords and Stemmer
stop_words = set(stopwords.words('english'))
stemmer = PorterStemmer()
3.3 Defining Text Preprocessing Function
def preprocess_text(text_data):

    preprocessed_text = []

    for sentence in tqdm(text_data):

        sentence = str(sentence).lower()

        sentence = re.sub(r'http\S+|www\S+|https\S+', '', sentence)

        sentence = re.sub(r'[^a-z\s]', '', sentence)

        tokens = word_tokenize(sentence)

        tokens = [stemmer.stem(word) for word in tokens if word not in stop_words]

        preprocessed_text.append(" ".join(tokens))

    return preprocessed_text
3.4 Applying Text Preprocessing
data['Title'] = preprocess_text(data['Title'].values)
4. Data Visualization

Data visualization helps represent data in graphical format to easily understand patterns and trends.

Libraries used:

Matplotlib

Seaborn

WordCloud

4.1 Word Cloud of Video Titles

A Word Cloud shows frequently used words in video titles.

from wordcloud import WordCloud
import matplotlib.pyplot as plt
import seaborn as sns

sns.set(style="whitegrid")

consolidated = ' '.join(word for word in data['Title'].astype(str))

wordCloud = WordCloud(
    width=1600,
    height=800,
    random_state=21,
    max_font_size=110,
    collocations=False,
    background_color='white'
)

plt.figure(figsize=(15,10))
plt.imshow(wordCloud.generate(consolidated), interpolation='bilinear')
plt.axis('off')
plt.title("WordCloud of Video Titles")
plt.show()
4.2 Top 3 Most Viewed Videos

A bar chart displays the top performing videos.

top_videos = data.sort_values(by='Views', ascending=False).head(3)

plt.figure(figsize=(12,6))

sns.barplot(
    x='Views',
    y='Title',
    data=top_videos,
    palette='coolwarm'
)

plt.title("Top 3 Most Viewed Videos")
plt.xlabel("Views")
plt.ylabel("Video Title")

plt.show()
4.3 Video Count by Duration Category

This chart shows the distribution of videos by duration type.

plt.figure(figsize=(8,6))

sns.countplot(
    x='Duration',
    data=data,
    palette='viridis',
    order=data['Duration'].value_counts().index
)

plt.title("Number of Videos by Duration Category")
plt.ylabel("Count")
plt.xlabel("Duration Category")

plt.show()
Conclusion

This project demonstrates how to perform YouTube data scraping, preprocessing, and visualization using Python.

Key outcomes of the project:

Extracted YouTube video data using web scraping

Cleaned and standardized raw data

Performed text preprocessing on video titles

Visualized insights using graphs and word clouds

This workflow helps in understanding content trends, audience engagement, and channel performance.
