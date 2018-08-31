---
layout: post
title: How to Scrape Web using Selenium and Beautiful Soup
description: Develop a recipe book for reading recipes without distubance of advertisements. Also devleop features for retrieving the recipes by using search through ingredients you have as well as recipe name.
---

In this tutorial, we will learn on how to scrap web using selenium and beautiful soup. I am going to use these tools to collect recipes from various food websites and store them in a structured format in a database. The 
two tasks involved in collecting the recipes are:
- Get all the recipe urls from the website using selenium
- Convert the html information of a recipe webpage into a structed json using beautiful soup.

For our task, I picked the [NDTV food](https://food.ndtv.com/recipes) and [Vahrehvah](https://www.vahrehvah.com/) as a source for extracting recipes.

## Selenium
[Selenim Webdriver](https://www.seleniumhq.org/projects/webdriver/) automates web browsers. The important use case of it is for autmating web applications for the testing purposes. It can also be used for web scraping. In our case, I used it for extracting all the urls corresponding to the recipes. 
<!-- In the following lines, I explain how to use selenium for extracting information in a web page.  -->

### Installation
I used selenium python bindings for using selenium web dirver. Through this python API, we can access all the functionalities of selenium web dirvers like Firefox, IE, Chrome, etc. We can use the following command for installing the selenium python API.

```bash
$ pip install selenium
```

Selenium python API requires a web driver to interface with your choosen browser. The corresponding web drivers can be downloaded from the following links. And also make sure it is in your PATH, e.g. `/usr/bin` or `/usr/local/bin`. For more information regarding installation, please refer to the [link](https://selenium-python.readthedocs.io/installation.html).


| Web browser   |     Web driver link     | 
|---------------|:-----------------------:|
| Chrome        | [chromedriver](https://sites.google.com/a/chromium.org/chromedriver/downloads)|
| Firefox       | [geckodriver](https://github.com/mozilla/geckodriver/releases)|
| Safari | [safaridriver](https://webkit.org/blog/6900/webdriver-support-in-safari-10/)|

I used chromedriver to automate the google chrome web browser. The following block of code opens the website in seperate window.

```python
from selenium import webdriver
from selenium.common.exceptions import NoSuchElementException,
                                       WebDriverException
driver = webdriver.Chrome(executable_path='/usr/local/bin/chromedriver')
driver.get("https://food.ndtv.com/recipes")
```

### Traversing the Sitemap of website

The [website that we want to scrape](https://food.ndtv.com/recipes) looks like this:

<p align="center">
<img src="/assets/Images/web_scraping/tab_contents.png" alt="tab_contents">
</p>

We need to collect all the group of the recipes like categories, cusine, festivals, occasion, member recipes, chefs, restaurant as shown in the above image. To do this, we will select the tab element and extract the text in it. 
We can find the id of the the tab and its attributes by inspect the source.
In our case, id is `insidetab`. We can extract the tab contents and their hyper links using the following lines.


```python
def get_link_by_text(text):
    """Find link in the page with given text"""
    element = driver.find_element_by_link_text(text.strip())
    return element.get_attribute("href")

def get_list_by_id(id_name):
    """Get list of text in element id_name"""
    try:
        element = driver.find_element_by_id(id_name)
        element_list = element.text.split('\n')
    except (NoSuchElementException, WebDriverException) as e:
        element_list = []
    return element_list

category_links = {x: get_link_by_text(x)
                  for x in get_list_by_id('insidetab')}
```

We need to follow each of these collected links and construct a link hierachy for the second level.

```python
sub_category_links = {}

for category, url in category_links.items():
    driver.get(url)  # open url in chrome

    sub_category_list = get_list_by_id('recipeListing')
    sub_category_links[category] = {x: get_link_by_text(x) 
                                    for x in sub_category_list}
```

When you load the leaf of the above `sub_category_links` dictionary, you will encounter the following pages with 'Show More' button as shown in the below image. Selenium shines at tasks like this where we can actually click the button using `element.click()` method.

<p align="center">
<img src="/assets/Images/web_scraping/show_more.png" alt="show_more">
</p>

For the click automation, we will use the below block of code.

```python
all_recipe_links = {}
def get_list_on_show_more(show_text='Show More',
                            x_path="//ul[@style='margin: 0;']"):
    """Loop till show_more doesn't have anything to load."""
    while True:
        try:
            driver.find_element_by_link_text(show_text).click()

            # Wait till the container of the recipes gets loaded 
            # after load more is clicked.
            time.sleep(5)

            # 
            containers = driver.find_elements_by_xpath(x_path)
            if len(containers) == 0:
                continue
            if 'No Record Found' in containers[-1].text:
                break
        except (NoSuchElementException, WebDriverException) as e:
            print(e)
            break
    result = []
    # get all the recipe elements of the page
    all_elements = driver.find_elements_by_class_name("main_image ")
    for element in all_elements:
        if len(element.text) > 0:
            result.append(element.text)
    return result

for category, urls in sub_category_links.items():
    all_recipe_links[category] = {}
    if category == 'CHEFS':
        # we want to ignore chefs.
        continue
    elif category == 'MEMBER RECIPES':
        # there's no additional tree traversal for member recipes
        all_recipe_links[category]['dummy_sub_category'] = urls
    else:
        bar = progressbar.ProgressBar()
        for sub_category, url in bar(urls.items()):
            driver.get(url)
            show_text = 'Show More'
            x_path = 
            all_recipe_list = get_list_on_show_more(driver, show_text, x_path)
            all_recipe_links[category][sub_category] = {
                x: get_link_by_text(x)
                for x in all_recipe_list
            }
            
```



## Beautiful Soup
[Beautiful Soup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/) is a python library for extracting required information by parsing HTML files. 

```python
from bs4 import BeautifulSoup
import requests

url = '<url>'
html_doc = requests.get(url).content
soup = BeautifulSoup(html_doc, 'html.parser')
```
I designed the output to be a fixed structured format. The following keys and information is extracted from each html page.
```js
{
    'source_url': url,
    'image_url': image,
    'title': title,
    'source': 'NDTV Food',
    'instructions': instructions,
    'ingredients': ingredients,
    'video_url': None,
    'cook_time': details.get('Cook Time', None),
    'prep_time': details.get('Prep Time', None),
    'difficulty': details.get('Difficulty Level', None),
    'servings': details.get('Recipe Servings', None),
    'other_details': other_details
}
``` 
Inspect the source page and get the class name for recipe container. In our case the recipe container class name is `recp-det-cont`. For the above keys, get the class name for the required information by inspecting the source. Use the below code for extracting above information.
```python
recipe_container = soup.find("div", {"class": "recp-det-cont"})
# Image url information
try:
    image = recipe_container.find(
            'div', {'class': 'art_imgwrap'}).picture.img['src']
except AttributeError:
    image = recipe_container.find(
            'div', {'class': 'food_gall'}).picture.img['src']
# title information
title = recipe_container.find('h1').get_text().strip()
    
description = recipe_container.find('p', {'class': 'recipe_description'}).get_text()

# details
recipe_details = recipe_container.find('div', {"class": "recipe-details"})
details = {}
for item in recipe_details.ul.children:
    key, value = item.get_text().split(':')
    details[key.strip()] = value.strip()

# ingredients
recipe_ingredients = recipe_container.find('div', {"class": "ingredients"})
ingredients = [x.get_text().strip()
               for x in recipe_ingredients.find_all('li')]

# tags
try:
    key_ingredients = recipe_container.find(
                        'div', {'class': 'keyword_tag'}).get_text()
    key_ingredients = [
                        x.strip() for x in
                        key_ingredients.replace('Key Ingredients: ', '').split(',')
                      ]
except AttributeError:
    key_ingredients = []

tags = [
        x.get_text().strip() for x in
        recipe_container.find('div', {'class': 'recptags'}).find_all('a')
       ]

# instructions
recipe_method = recipe_container.find('div', {"class": "method"})
instructions = []
for x in recipe_method.find_all('li'):
    try:
        instructions.append(x.span.get_text().strip())
    except AttributeError:
        instructions.append(x.get_text().strip())

#other_details
other_details = {
    'details': details,
    'description': description,
    'tags': tags,
    'key_ingredients': key_ingredients
}
```
