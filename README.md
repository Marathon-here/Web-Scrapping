# YouTube Data Scraping, Preprocessing and Analysis using Python

## 📌 Project Overview

YouTube is one of the largest video-sharing platforms, hosting millions of videos across different categories and audiences. Analyzing YouTube data helps uncover valuable insights about **content performance, viewer engagement, and emerging trends**.

This project demonstrates how to:

- Scrape YouTube video data using Python
- Preprocess and clean the dataset
- Perform text preprocessing on video titles
- Visualize insights using data visualization techniques

The project extracts data from a YouTube channel and analyzes it to identify patterns in video performance.

---

# 📂 Project Workflow

The project consists of four main stages:

1. **Web Scraping**
2. **Data Preprocessing**
3. **Text Preprocessing**
4. **Data Visualization**

---

# 1️⃣ Web Scraping

## What is Web Scraping?

Web scraping is the process of automatically extracting data from websites. In this project, we scrape video information such as:

- Video Title
- View Count
- Video Duration

We use the following Python libraries:

- `requests` – to send HTTP requests
- `BeautifulSoup` – to parse HTML content
- `json` – to handle JSON data
- `xlsxwriter` – to store the scraped data in Excel

The data is scraped from the following YouTube channel:

https://www.youtube.com/c/GeeksforGeeksVideos/videos

---

## Installing Required Libraries

```bash
pip install requests beautifulsoup4 xlsxwriter
```

---

## Importing Required Libraries

```python
import requests
from bs4 import BeautifulSoup
import json
import xlsxwriter
```

---

## Sending HTTP Request

```python
url = "https://www.youtube.com/c/GeeksforGeeksVideos/videos"

headers = {
    "User-Agent": "Mozilla/5.0"
}

response = requests.get(url, headers=headers)
html = response.text
```

---

## Parsing HTML with BeautifulSoup

```python
soup = BeautifulSoup(html, "html.parser")
```

---

## Extracting YouTube JSON Data

YouTube dynamically loads video information using JavaScript. The required data is stored inside a JSON object called **ytInitialData**.

```python
script_tag = soup.find("script", text=lambda t: t and "var ytInitialData = " in t)

json_text = script_tag.string.strip()[len("var ytInitialData = "):-1]

data = json.loads(json_text)
```

---

## Navigating Video Metadata

```python
videos_data = data['contents']['twoColumnBrowseResultsRenderer']['tabs'][1]\
['tabRenderer']['content']['richGridRenderer']['contents']
```

---

## Extracting Titles, Views and Durations

```python
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
```

---

## Saving Data to Excel

```python
workbook = xlsxwriter.Workbook("youtube_videos.xlsx")
sheet = workbook.add_worksheet()

sheet.write(0,0,"Title")
sheet.write(0,1,"Views")
sheet.write(0,2,"Duration")

for i in range(len(titles)):
    sheet.write(i+1,0,titles[i])
    sheet.write(i+1,1,views[i])
    sheet.write(i+1,2,durations[i])

workbook.close()

print("Scraped latest 30 videos successfully! Saved as youtube_videos.xlsx")
```

---

# 2️⃣ Data Preprocessing

Scraped data often contains inconsistencies and textual values. Data preprocessing is required to clean and standardize the dataset.

We use **Pandas** for data manipulation.

```python
import pandas as pd

data = pd.read_excel("youtube_videos.xlsx")
data.head()
```

---

## Cleaning the Views Column

Steps performed:

- Remove the word **"views"**
- Remove commas
- Convert **K values** into numeric format
- Convert strings into numbers

```python
data['Views'] = data['Views'].str.replace(" views","").str.strip()
```

Example conversion:

| Original | Converted |
|--------|--------|
| 12K views | 12000 |
| 1,500 views | 1500 |

---

## Cleaning Duration Column

Duration values are converted into **seconds**.

```python
data['Duration'] = data['Duration'].str.replace("\n","")
```

Example:

| Duration | Seconds |
|--------|--------|
| 10:30 | 630 |
| 01:02:10 | 3730 |

---

## Categorizing Video Duration

Videos are categorized into three groups:

| Duration | Category |
|--------|--------|
| < 15 minutes | Mini Videos |
| 15 minutes – 1 hour | Long Videos |
| > 1 hour | Very Long Videos |

---

# 3️⃣ Text Preprocessing

Video titles contain important textual information. To analyze them properly we apply text preprocessing.

Steps include:

- Lowercasing
- Removing URLs
- Removing special characters
- Tokenization
- Stopword removal
- Stemming

Libraries used:

- `nltk`
- `re`
- `tqdm`

```python
from nltk.corpus import stopwords
from nltk.tokenize import word_tokenize
from nltk.stem.porter import PorterStemmer
```

Example:

**Original Title**

```
Learn Python Programming in 10 Minutes!
```

**Processed Title**

```
learn python program minut
```

---

# 4️⃣ Data Visualization

Data visualization helps transform processed data into graphical insights.

Libraries used:

- `matplotlib`
- `seaborn`
- `wordcloud`

---

## Word Cloud of Video Titles

A word cloud shows the most frequent words appearing in video titles.

```python
from wordcloud import WordCloud
import matplotlib.pyplot as plt
```

This helps identify **trending topics in the channel content**.

---

## Top 3 Most Viewed Videos

A bar chart shows the videos with the highest view counts.

```python
top_videos = data.sort_values(by='Views',ascending=False).head(3)
```

This helps identify **popular content on the channel**.

---

## Video Count by Duration Category

A count plot shows how many videos fall into each duration category.

Categories:

- Mini Videos
- Long Videos
- Very Long Videos

This helps understand the **content strategy of the channel**.

---

# 🛠 Technologies Used

| Technology | Purpose |
|---|---|
Python | Programming Language |
Requests | HTTP Requests |
BeautifulSoup | HTML Parsing |
Pandas | Data Processing |
NLTK | Text Processing |
Matplotlib | Data Visualization |
Seaborn | Statistical Visualization |
WordCloud | Word Visualization |
XlsxWriter | Excel File Creation |

---

# 📊 Project Output

The project generates:

**Dataset**

```
youtube_videos.xlsx
```

**Visualizations**

- Word Cloud of Video Titles
- Top 3 Most Viewed Videos
- Video Distribution by Duration

These outputs help analyze **content trends and viewer engagement**.

---

# 🚀 Future Improvements

Possible enhancements include:

- YouTube **comments sentiment analysis**
- Using **YouTube Data API**
- Building a **dashboard using Power BI or Tableau**
- Predicting video popularity using **Machine Learning**

---

# 📌 Conclusion

This project demonstrates a complete pipeline for **YouTube data scraping, preprocessing, and analysis using Python**.

By combining **web scraping, data cleaning, text processing, and visualization**, meaningful insights can be extracted from YouTube content data.

This approach can help analyze **content trends, audience engagement, and video performance**.
