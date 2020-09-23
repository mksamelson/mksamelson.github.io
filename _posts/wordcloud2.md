title:  "Insight through Words:  Political Messaging" - Part II
date: 2020-06-15
tags: [Data Science]
excerpt:  "Word use can highlight conversation themes."
---
This post highlights the code used to generate the word clouds and insights in Part I of this post.

# Code #

The code used to generate word clouds employs basic webscraping and NLP techniques.

## Webscraping a Transcript ##

Political press conference transcripts are readily available on the web with minimal google searching.  

Rev.com (https://www.rev.com) hosts a large number of press conference transcriptions.

Once a particular individual and press conference is identified, the process of acquiring the content is straightforward.  The site address and individual's name are fed into the scraper and the code extracts the content associated with that individual for further processing.

Scraping is easily accomplished using the Selenium, BeautifulSoup, and regular expression Python packages.

```Python

from selenium import webdriver
from bs4 import BeautifulSoup
import re

url = 'https://www.rev.com/blog/transcripts/donald-trump-coronavirus-press-conference-transcript-april-17'

driver = webdriver.Chrome(executable_path=r'chromedriver.exe')

driver.get(url)

soup = BeautifulSoup(driver.page_source,"lxml")

time.sleep(3)

driver.close()

transcript = soup.find('div', {'class':'fl-callout-text'})

master_string = ''
speaker_list = transcript.find_all('p')
for speaker in speaker_list:
    if re.search('Mike Pence:', speaker.text):
        speaker_words = re.sub(r"Mike Pence: \s*\(.*\)\s*", "", speaker.text)
        speaker_words = re.sub(r"\s*\[.*\]\s*", "", speaker_words)
        master_string += speaker_words

```

The master_string is simply raw content identified as spoken by our person of interest.  Exploring Vice President Pence's comments at the April 17, 2020 presidential press conference, the beginning of the master_string looks like this:

```python

Thank you Mr. President, and good afternoon, all. Today, as the president just reflected, it remains a challenging time in the life of our nation. But because of the extraordinary efforts of the American people, because of the strong partnership the federal government has forged with states across ......

```

Note that the master_string content includes punctuation, capitalization, and various verb tenses.  There are also a number of contracted words (e.g., "it's") and potentially digits.  The string must be cleaned to perform any type of meaningful analysis.

Capitalization is easily removed from the master_string with one line of code:

```python
master_string = master_string.lower()
```

Punctuation and numbers are not much more difficult to remove.  A boolean to remove digits is set to true and a regular expression including or excluding digits is initialized.  A straightforward regular expression search and replace is then performed:

```python
remove_digits = True
pattern = r'[^a-zA-Z0-9\s]' if not remove_digits else r'[^a-zA-Z\s]'
master_string = re.sub(pattern, '', master_string)

```

Removing contractions is easy - if a list of them is handy.  We reference a list of contractions and their corresponding uncontracted form in a dictionary:

```Python

CONTRACTION_MAP = {
"ain't": "is not",
"aren't": "are not",
"can't": "cannot"
}

```

Then we employ a somewhat ugly function through which we pass the master_string and replace contractions with their expanded form:

```python

def expand_contractions(text, contraction_mapping=CONTRACTION_MAP):
    contractions_pattern = re.compile('({})'.format('|'.join(contraction_mapping.keys())),
                                      flags=re.IGNORECASE | re.DOTALL)

    def expand_match(contraction):
        match = contraction.group(0)
        first_char = match[0]
        expanded_contraction = contraction_mapping.get(match) \
            if contraction_mapping.get(match) \
            else contraction_mapping.get(match.lower())
        expanded_contraction = first_char + expanded_contraction[1:]
        return expanded_contraction

    expanded_text = contractions_pattern.sub(expand_match, text)
    expanded_text = re.sub("'", "", expanded_text)
    return expanded_text

```
