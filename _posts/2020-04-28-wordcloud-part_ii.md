---
title:  "Insight through Words:  Political Messaging - Part II"
date: 2020-04-28
tags: [Data Science]
excerpt:  "This post highlights the code used to generate the word clouds and insights in Part I."
---
This post highlights the code used to generate the word clouds and insights in Part I.  If you haven't read Part I, click [here](https://mksamelson.github.io/wordcloud/).

# Code #

The code used to generate word clouds employs basic webscraping and NLP techniques.

## Webscraping a Transcript ##

Political press conference transcripts are readily available on the web with minimal google searching.  

Rev.com (https://www.rev.com) hosts a large number of press conference transcriptions.

Once a particular individual and press conference is identified, the process of acquiring the content is straightforward.  The site address and individual's name are fed into the scraper and the code extracts the content associated with that individual for further processing.

Scraping is easily accomplished using the Selenium, BeautifulSoup, and regular expression Python packages.

```python

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

## Cleaning The Transcript ##

Note that the master_string content includes punctuation, capitalization, and various verb tenses.  There are also a number of contracted words (e.g., "it's") and potentially digits.  The string must be cleaned to perform any type of meaningful analysis.

### Capitalization ###

Capitalization is easily removed from the master_string with one line of code:

```python
master_string = master_string.lower()
```

### Punctuation and Numbers ###

Punctuation and numbers are not much more difficult to remove.  A boolean to remove digits is set to true and a regular expression including or excluding digits is initialized.  A straightforward regular expression search and replace is then performed:

```python
remove_digits = True
pattern = r'[^a-zA-Z0-9\s]' if not remove_digits else r'[^a-zA-Z\s]'
master_string = re.sub(pattern, '', master_string)

```

### Contracted Words ###

Removing contractions is easy - if a list of them is handy.  We reference a list of contractions and their corresponding uncontracted form in a dictionary:

```python

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

### Stop Words ###

'Stop Words' are common words in language that carry little meaning.  

Examples of common stop words in the English language include "but", "and", "to", "about", etc.

There is no definitive list of stop words

Pythons nltk library contains a list of recognized English stop words.  We can use the list to filter out stop words from our transcript.

To do this we tokenize the master_string (split it into individual words) and then reform the string excluding stop words in the ntlk English stop word list:

```python

from nltk.corpus import stopwords

stopword_list = stopwords.words('english')
tokens = nltk.word_tokenize(master_string)
tokens = [token.strip() for token in tokens]
tokenized_output = ' '.join([token for token in tokens if token not in stopword_list])

```

### Lemmatization ###

In linguistics, a lemma is the canonical or 'dictionary' form of a word.

In spoken and written language, some words can have multiple forms.  Think verbs.   The lemma "go" represents the word forms "go", "goes", "going", "went", and "gone".

Lemmatization involves determining the lemma of so-called 'inflected' forms given the context of the sentence.  Stemming, a related process, where different forms of a word are reduced to a common stem - a part of a word that never changes.

Accordingly, a stem is the part of a word that never changes even when inflected while a lemma is the base form of the word.

Fortunately for us, the ntlk library has some functions that do the heavy lifting.

In order to correctly lemmatize words we need to tag them with their part of speach (e.g., "NOUN","VERB", "ADJ", etc.).  We can tag each word by tolkenizing the tolkenized_output to this point and then using the pos tagger in nltk:

```python

word_list = nltk.word_tokenize(tokenized_output)
word_list = nltk.pos_tag(word_list)

```

Once words are tagged we can lemmatize.

```python

from nltk.corpus import wordnet

def get_wordnet_pos(treebank_tag):

    if treebank_tag.startswith('J'):
        return wordnet.ADJ
    elif treebank_tag.startswith('V'):
        return wordnet.VERB
    elif treebank_tag.startswith('N'):
        return wordnet.NOUN
    elif treebank_tag.startswith('R'):
        return wordnet.ADV
    else:
        return wordnet.NOUN

lemmatized_word_list = [lemmatizer.lemmatize(w[0],get_wordnet_pos(w[1])) for w in word_list]

```

### Word Count ###

Now we are ready to do a straightforward wordcount.  To do this we create a dictionary where the keys are the words and the values are the counts.  (Alternatively we could have used the Counter from the Python's collections library).

```python
freq = {}
for item in lemmatized_word_list:
    if (item in freq):
        freq[item] += 1
    else:
        freq[item] = 1
```

## Word Cloud Generation ##

Finally we're ready to generate the word cloud.

```python
from wordcloud import WordCloud

wordcloud = WordCloud(max_words=30,relative_scaling=1,mask=mask,normalize_plurals=False,background_color="white")
wordcloud.generate_from_frequencies(freq)
plt.imshow(wordcloud, interpolation='bilinear')
plt.suptitle("Pence Words")
plt.title("COVID Update April 17th")
plt.axis("off")
plt.show()

```
Which yields the finished product.

<img src="{{site.url}}{{ site.baseurl }}/images/wordcloud/Pence2Words417.png" alt="">

Full code can be found in my Github repository [here](https://github.com/mksamelson/Political_Wordcloud)
