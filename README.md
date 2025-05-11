# Web Scraping with Regex

[![Bright Data Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.com/)

This guide explains how to use regular expressions in [Python for web scraping](https://brightdata.com/blog/how-tos/web-scraping-with-python):

- [Understanding Regular Expressions](#understanding-regular-expressions)
- [Implementing Regex in Python for Web Scraping](#implementing-regex-in-python-for-web-scraping)
- [Constraints of Using Regex for Web Scraping](#constraints-of-using-regex-for-web-scraping)

## What Is Regex

Regular expressions (regex) are robust pattern-matching formulas for extracting information from text, making them valuable tools for web scraping. These expressions define specific patterns to match within texts, allowing for precise information extraction.

In Python, regular expressions use tokens to match particular patterns. While covering all tokens is beyond this article's scope, here are some frequently used ones you'll encounter:

| Token | Matches |
| --- | --- |
| Any non-special character | The character given |
| `^` | Start of a string |
| `$` | End of a string |
| `.` | Any character except `\n` |
| `*` | Zero or more occurrences of the previous element |
| `?` | Zero or one occurrence of the previous element |
| `+` | One or more occurrences of the previous characters |
| `{Digit}` | Exact number of the previous element |
| `\d` | Any digit |
| `\s` | Any whitespace character |
| `\w` | Any word character |
| `\D` | Inverse of `\d` |
| `\S` | Inverse of `\s` |
| `\W` | Inverse of `\w` |

For hands-on experience and to learn more about regex, visit [regexr.com](https://regexr.com/). Additionally, [this article](https://docs.bmc.com/docs/discovery/113/improving-the-performance-of-regular-expressions-788111995.html) provides essential tips for optimizing your regex performance.

## Using Regex in Python for Web Scraping

In this section, we'll develop a basic web scraper in Python using regex to extract data from various websites.

First, create a project directory:

```sh
mkdir web_scraping_with_regex
cd web_scraping_with_regex
```

Then create a Python virtual environment:

```sh
python -m venv venv
```

Activate it:

```sh
source ./venv/bin/activate
```

And on Windows:

```
venv\Scripts\activate
```

For this web scraper, you'll need two libraries:

- `requests` for fetching web pages
- `beautifulsoup4` for parsing the HTML content and locting elements

Install the libraries:

```sh
pip install beautifulsoup4 requests
```

> **Note:**
> Always check a website's terms and conditions before scraping to ensure it's permitted. Avoid scraping if explicitly forbidden.

### Scraping an E-commerce Site

Let's build a scraper that extracts book titles and prices from a [dummy e-commerce site](https://books.toscrape.com/). You’ll scrape the first page and extract the titles and prices of the books.

Create a file named `scraper.py` and import the required modules:

```python
import requests
from bs4 import BeautifulSoup
import re
```

> **Note:** The `re` module is a built-in Python module that works with regex.

Next, make a GET request to fetch the web page's HTML content:


```python
page = requests.get('https://books.toscrape.com/')
```

Feed this data to Beautiful Soup to parse the HTML structure:

```python
soup = BeautifulSoup(page.content, 'html.parser')
```

To figure out how the elements are structured in HTML, you use the **Inspect Element** tool. Open the [web page](https://books.toscrape.com/) in the browser and press **Ctrl + Shift + I** to open the **Inspector**. As you can see in the screenshot, the products are stored in `li` elements with class `col-xs-6 col-sm-4 col-md-3 col-lg-3`. The book title can be found from `a` elements by reading their `title` attribute, and the prices are stored in `p` elements with class `price_color`:

![prices are stored in p elements with class price_color](https://github.com/luminati-io/web-scraping-with-regex/blob/main/images/prices-are-stored-in-p-elements-with-class-price_color.png)

Use the `find_all` method of Beautiful Soup to find all `li` elements with class `col-xs-6 col-sm-4 col-md-3 col-lg-3`:

```python
books = soup.find_all("li", class_="col-xs-6 col-sm-4 col-md-3 col-lg-3")
content = str(books)
```

The `content` variable now holds the HTML text of the `li` elements, and you can use regex to extract the titles and prices.

First, create regex patterns that match book titles and prices by examining the HTML structure again.

The book titles are found in the `title` attribute of `a` elements, which look like:

```html
<a href="..." title="...">
```

To capture the content within the double quotes after title, use the `.*?` regex pattern. Here, `.` matches any character, `*` matches zero or more occurrences of the preceding element, and `?` makes the match non-greedy. The complete expression is:

```regex
<a href=".*?" title="(.*?)"
```

The parentheses around the `.*?` create a [capturing group](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Regular_expressions/Capturing_group), which stores the matched content for later use.

For prices, use a similar approach. Since prices appear in `p` elements with class `price_color`, the regex pattern is `<p class="price_color">(.*?)</p>`.

Define both patterns:

```python
re_book_title = r'<a href=".*?" title="(.*?)"'
re_prices = r'<p class="price_color">(.*?)</p>'
```

> **Note:**
> In case you’re wondering why the `?` is needed after `.*`, [this Stack Overflow answer](https://stackoverflow.com/a/2301298) explains the role of `?` well.

Now, use `re.findall()` to find all regex matches from the HTML string:

```python
titles = re.findall(re_book_title, content)
prices = re.findall(re_prices, content)
```

Finally, iterate over the matches and print the results:

```python
for i in zip(titles, prices):
    print(f"{i[0]}: {i[1]}")
```

Run this code with `python scraper.py`. Here is the expected output:

```
A Light in the Attic: £51.77
Tipping the Velvet: £53.74
Soumission: £50.10
Sharp Objects: £47.82
Sapiens: A Brief History of Humankind: £54.23
The Requiem Red: £22.65
The Dirty Little Secrets of Getting Your Dream Job: £33.34
The Coming Woman: A Novel Based on the Life of the Infamous Feminist, Victoria Woodhull: £17.93
The Boys in the Boat: Nine Americans and Their Epic Quest for Gold at the 1936 Berlin Olympics: £22.60
The Black Maria: £52.15
Starving Hearts (Triangular Trade Trilogy, #1): £13.99
Shakespeare's Sonnets: £20.66
Set Me Free: £17.46
Scott Pilgrim's Precious Little Life (Scott Pilgrim #1): £52.29
Rip it Up and Start Again: £35.02
Our Band Could Be Your Life: Scenes from the American Indie Underground, 1981-1991: £57.25
Olio: £23.88
Mesaerion: The Best Science Fiction Stories 1800-1849: £37.59
Libertarianism for Beginners: £51.33
It's Only the Himalayas: £45.17
```

### Scraping a Wikipedia Page

Now, let’s build a scraper that can scrape a [Wikipedia page](https://en.wikipedia.org/wiki/Web_scraping) and extract information about all the links.

Create a new file named `wiki_scraper.py`. Similar to before, import the required libraries, make a GET request, and parse the content:

```python
import requests
from bs4 import BeautifulSoup
import re

page = requests.get('https://en.wikipedia.org/wiki/Web_scraping')
soup = BeautifulSoup(page.content, 'html.parser')
```

Use the `find_all()` method to find all the links:

```python
links = soup.find_all("a")
content = str(links)
```

Since link text appears in the `title` attribute and URLs in the `href` attribute, you can use the `(.*?)` pattern to capture both. The complete expression is:

```regex
<a href="(.*?)" title="(.*?)">.*?</a>
```

Note that the third `.*?` is not in a capturing group because you aren’t interested in the content of the `a` tags.

As before, use `findall()` to find all the matches and print the result:

```python
re_links = r'<a href="(.*?)" title="(.*?)">.*?</a>'

links = re.findall(re_links, content)

for i in links:
    print(f"{i[0]} => {i[1]}")
```

When you run this with `python wiki_scraper.py`, you get the following output:

```
OUTPUT TRUNCATED FOR BREVITY

/wiki/Category:Web_scraping => Category:Web scraping
/wiki/Category:CS1_maint:_multiple_names:_authors_list => Category:CS1 maint: multiple names: authors list
/wiki/Category:CS1_Danish-language_sources_(da) => Category:CS1 Danish-language sources (da)
/wiki/Category:CS1_French-language_sources_(fr) => Category:CS1 French-language sources (fr)
/wiki/Category:Articles_with_short_description => Category:Articles with short description
/wiki/Category:Short_description_matches_Wikidata => Category:Short description matches Wikidata
/wiki/Category:Articles_needing_additional_references_from_April_2023 => Category:Articles needing additional references from April 2023
/wiki/Category:All_articles_needing_additional_references => Category:All articles needing additional references
/wiki/Category:Articles_with_limited_geographic_scope_from_October_2015 => Category:Articles with limited geographic scope from October 2015
/wiki/Category:United_States-centric => Category:United States-centric
/wiki/Category:All_articles_with_unsourced_statements => Category:All articles with unsourced statements
/wiki/Category:Articles_with_unsourced_statements_from_April_2023 => Category:Articles with unsourced statements from April 2023
```

### Scraping a Dynamic Site

The previous examples involved static web pages. Scraping dynamic pages requires browser automation tools like [Selenium](https://www.selenium.dev/). Here's an example of extracting the current temperature from [OpenWeatherMap](https://openweathermap.org/) using Selenium and regex:

```python
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.by import By
from selenium.webdriver.support.wait import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import re

driver = webdriver.Firefox()
driver.get("https://openweathermap.org/city/2643743")
elem = WebDriverWait(driver, 10).until( EC.presence_of_element_located((By.CSS_SELECTOR, ".current-temp")))
content = elem.get_attribute('innerHTML')
re_temp = r'<span .*?>(.*?)</span>'
temp = re.findall(re_temp, content)
print(repr(temp))
driver.close()
```

This code uses Selenium to launch an instance of Firefox and uses the CSS selector to select the element with the current temperature. It then uses the regex `<span .*?>(.*?)</span>` to extract the temperature.

This code launches Firefox with Selenium, selects the element containing the current temperature using a CSS selector, and then extracts the temperature using the regex pattern `<span .*?>(.*?)</span>`.

For even more information to help you get started with scraping dynamic web pages with Selenium, check out [this tutorial](https://brightdata.com/blog/how-tos/using-selenium-for-web-scraping).

## Limitation of Regex for Web Scraping

While regular expressions are powerful for pattern matching and data extraction, they have significant limitations for web scraping. Regex operates on text with no understanding of HTML structure, making results highly dependent on the HTML's formatting.

For instance, in the Wikipedia example, some links weren't correctly extracted:

![Links that weren't exctracted correctly](https://github.com/luminati-io/web-scraping-with-regex/blob/main/images/Links-that-werent-exctracted-correctly.png)

If you edit the Python code and add `print(content)` to print the HTML string returned by Beautiful Soup, you see the culprit `a` looks like this:

```regex
<a href="#cite_ref-9">^</a>
```

This tag lacks a `title` attribute, yet our regex pattern assumes the structure `<a href="(.*?)" title="(.*?)">.*?</a>`. Because regex doesn't comprehend HTML elements, instead of failing to match, the `.*?` pattern keeps matching characters until it finds something that completes the pattern, often incorrectly capturing multiple tags.

Furthermore, HTML isn't a regular language, meaning regex alone can't reliably parse arbitrary HTML. However, regex can be useful in specific scenarios. If you're working with a limited, known HTML structure, regex can extract information effectively. Still, a more robust approach is using an HTML parser like Beautiful Soup to find elements and then apply regex to process the extracted text.

Here's an improved version of the Wikipedia scraper that uses Beautiful Soup for initial extraction and regex for filtering:

```python
import requests
from bs4 import BeautifulSoup
import re

page = requests.get('https://en.wikipedia.org/wiki/Web_scraping')
soup = BeautifulSoup(page.content, 'html.parser')
links = soup.find_all("a")

for link in links:
    href = link.get('href')
    title = link.get('title')

    if title == None:
        title = link.string

    if title == None:
        continue

    pattern = r"[a-zA-Z0-9]"

    if re.match(pattern, title):
        print(f"{href} => {title}")
```

## Conclusion

Regular expressions are valuable tools for finding patterns in text data. However, web scraping presents numerous challenges beyond regex capabilities. Frequent scraping can lead to IP blocking, and CAPTCHAs can disrupt your scraper's functionality. Bright Data offers [powerful proxies](https://brightdata.com/proxy-types) that can help overcome IP restrictions.

Start a free trial today!
