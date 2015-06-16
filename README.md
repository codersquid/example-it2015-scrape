# example-it2015-scrape
example of scrape with new steve scraper. For PyCon Italia 2015.

Everyone has different conference sites and ways of posting schedules.
There is no common scraping code that will work from conference to conference,
or year to year. I end up writing one-off throw-away code 

In conjection with getting the youtube data from steve, I also wrote junky
one-off throw-away code to scrape their conference site to get
descriptions and speaker information. (I didn't find any way to call an
api to get a list of all the talks in machine readable form).

```python
import requests
from bs4 import BeautifulSoup

language_codes = {
    'IT': "Italian",
    'EN': "English",
}

def get_urls(fname):
    with open(fname) as fh:
        rawdata = fh.read()
        return [r.split(' ')[0] for r in rawdata.split('\n') if r]

def get_metadata(url):
    d = {}
    r = requests.get(url)
    soup = BeautifulSoup(r.content)
    d['url'] = url
    d['title'] = soup.h1.text
    speakers = soup.select(".talk-speakers")[0].find_all("a")
    d['speakers'] = [s.text for s in speakers]
    d['description'] = soup.select(".cms")[0].text
    language_code = soup.select(".details")[0].find_all("dd")[1].text
    d['language'] = language_codes.get(language_code, '')
    return d


def all_metadata(fname):
    urls = get_urls(fname)
    metadata = []
    for u in urls:
        metadata.append(get_metadata(u))
    return metadata
```

Once I got the scraped data in to a file, I went ahead and edited things by
hand in a split vim window, jumping between the conference file and the
steve files doing searches to find the right matches.

Depending on a conference, I might not have to do that if their av
provider has uploaded things to youtube using the same titles. At
some point I'll probably try out difflib to give me some score to attempt
to find matches. Haven't done it yet.
