
# Scraper Google
Objectif: récupérer le positionnement des sites sur le moteur de recherche Google

### Import des packages


```python
import requests
import time
from bs4 import BeautifulSoup
from urllib.parse import urlparse
import json
import csv

USER_AGENT = {'User-Agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/61.0.3163.100 Safari/537.36'}

```

### Fonction qui récupère les résultats de Google


```python
def fetch_results(search_term, number_results, language_code):
    assert isinstance(search_term, str), 'Search term must be a string'
    assert isinstance(number_results, int), 'Number of results must be an integer'
    escaped_search_term = search_term.replace(' ', '+')
 
    google_url = 'https://www.google.com/search?q={}&num={}&hl={}'.format(escaped_search_term, number_results, language_code)
    response = requests.get(google_url, headers=USER_AGENT)
    response.raise_for_status()
 
    return search_term, response.text
```

### Fonction qui structure les résultats


```python
def parse_results(html, keyword):
    soup = BeautifulSoup(html, 'html.parser')

    found_results = []
    rank = 1
    result_block = soup.find_all('div', attrs={'class': 'g'})
    for result in result_block:

        link = result.find('a', href=True)
        title = result.find('h3')
        website = result.find('cite')
        description = result.find('span', attrs={'class': 'st'})
        if link and title:
            link = link['href']
            title = title.get_text()
            if website:
                website = urlparse(website.get_text()).netloc.split(' ')[0]                    
            if description:
                description = description.get_text()
            if link != '#':
                if website:
                    found_results.append({'keyword': keyword, 
                                          'rank': rank, 
                                          'title': title, 
                                          'description': description, 
                                          'website': website})
                    rank += 1
    return found_results
```

### Fonction qui exporte les résultats en csv


```python
def export_csv(filepath, data):
    outfile = open(filepath, "w", encoding="utf-8", newline='')
    keys = data[0].keys()
    dict_writer = csv.DictWriter(outfile, keys)
    dict_writer.writeheader()
    dict_writer.writerows(data)
    outfile.close()
    return "Fin de l'export"
```

### Fonction principale


```python
def scrape_google(search_term, number_results, language_code):
    try:
        keyword, html = fetch_results(search_term, number_results, language_code)
        results = parse_results(html, keyword)
        return results
    except AssertionError:
        raise Exception("Incorrect arguments parsed to function")
    except requests.HTTPError:
        raise Exception("You appear to have been blocked by Google")
    except requests.RequestException:
        raise Exception("Appears to be an issue with your connection")
```

### Définition des mots clés à scraper


```python
keywords = ['location voiture']
```

# Lancement du programme


```python
_FILENAME_ = "output.csv"

data = []
for keyword in keywords:
    try:
        results = scrape_google(keyword, 100, "fr")
        for result in results:
            data.append(result)
    except Exception as e:
        print(e)
    finally:
        time.sleep(10)
export_csv(_FILENAME_, data)

```
