
Matthew Lee 999078525



# Assignment 4

In this assignment, you'll combine the assignment 3 data set with nutrition data from the [USDA Food Composition Databases](https://ndb.nal.usda.gov/ndb/search/list). The CSV file `fresh.csv` contains the fresh fruits and vegetables data you extracted in assignment 3.

The USDA Food Composition Databases have a [documented](https://ndb.nal.usda.gov/ndb/doc/index) web API that returns data in JSON format . You need a key in order to use the API. Only 1000 requests are allowed per hour, so it would be a good idea to use [caching][requests_cache].

[Sign up for an API key here](https://api.data.gov/signup/). The key will work with any Data.gov API. You may need the key again later in the quarter, so make sure you save it.

These modules may be useful:

* [requests](http://docs.python-requests.org/en/master/user/quickstart/)
* [requests_cache][]
* [urlparse](https://docs.python.org/2/library/urlparse.html)
* [pandas](http://pandas.pydata.org/pandas-docs/stable/)

[requests_cache]: https://pypi.python.org/pypi/requests-cache


```python
from urllib2 import Request, urlopen
from urlparse import urlparse, urlunparse
import requests, requests_cache
import pandas as pd
import seaborn as sns
import json
import re
from matplotlib import pyplot as plt
plt.style.use('ggplot')
%matplotlib inline
requests_cache.install_cache('demo_cache')
key="yCtT9prMnfCoZQwUP4MfYqPb37wTvalz3zrRnPvQ"
#make sure to get rid of key after
```

__Exercise 1.1.__ Read the [search request documentation](https://ndb.nal.usda.gov/ndb/doc/apilist/API-SEARCH.md), then write a function called `ndb_search()` that makes a search request. The function should accept the search term as an argument. The function should return the search result items as a list (for 0 items, return an empty list).

Note that the search url is: `https://api.nal.usda.gov/ndb/search`

As an example, a search for `"quail eggs"` should return this list:

```python
[{u'ds': u'BL',
  u'group': u'Branded Food Products Database',
  u'name': u'CHAOKOH, QUAIL EGG IN BRINE, UPC: 044738074186',
  u'ndbno': u'45094707',
  u'offset': 0},
 {u'ds': u'BL',
  u'group': u'Branded Food Products Database',
  u'name': u'L&W, QUAIL EGGS, UPC: 024072000256',
  u'ndbno': u'45094890',
  u'offset': 1},
 {u'ds': u'BL',
  u'group': u'Branded Food Products Database',
  u'name': u'BUDDHA, QUAIL EGGS IN BRINE, UPC: 761934535098',
  u'ndbno': u'45099560',
  u'offset': 2},
 {u'ds': u'BL',
  u'group': u'Branded Food Products Database',
  u'name': u'GRAN SABANA, QUAIL EGGS, UPC: 819140010103',
  u'ndbno': u'45169279',
  u'offset': 3},
 {u'ds': u'BL',
  u'group': u'Branded Food Products Database',
  u'name': u"D'ARTAGNAN, QUAIL EGGS, UPC: 736622102630",
  u'ndbno': u'45178254',
  u'offset': 4},
 {u'ds': u'SR',
  u'group': u'Dairy and Egg Products',
  u'name': u'Egg, quail, whole, fresh, raw',
  u'ndbno': u'01140',
  u'offset': 5}]
```

As usual, make sure you document and test your function.


```python
def ndb_search(search):
    url='https://api.nal.usda.gov/ndb/search'
    response=requests.get(url, params = {
        "q": search,
        "api_key": key,
        "format": "json",
    })
    
    
    #print response.url
    response.raise_for_status()
    test=response.json()
    return test['list']['item']
    #return test['list']['item']

ndb_search("quail eggs")
```




    [{u'ds': u'BL',
      u'group': u'Branded Food Products Database',
      u'name': u'CHAOKOH, QUAIL EGG IN BRINE, UPC: 044738074186',
      u'ndbno': u'45094707',
      u'offset': 0},
     {u'ds': u'BL',
      u'group': u'Branded Food Products Database',
      u'name': u'L&W, QUAIL EGGS, UPC: 024072000256',
      u'ndbno': u'45094890',
      u'offset': 1},
     {u'ds': u'BL',
      u'group': u'Branded Food Products Database',
      u'name': u'BUDDHA, QUAIL EGGS IN BRINE, UPC: 761934535098',
      u'ndbno': u'45099560',
      u'offset': 2},
     {u'ds': u'BL',
      u'group': u'Branded Food Products Database',
      u'name': u'GRAN SABANA, QUAIL EGGS, UPC: 819140010103',
      u'ndbno': u'45169279',
      u'offset': 3},
     {u'ds': u'BL',
      u'group': u'Branded Food Products Database',
      u'name': u"D'ARTAGNAN, QUAIL EGGS, UPC: 736622102630",
      u'ndbno': u'45178254',
      u'offset': 4},
     {u'ds': u'SR',
      u'group': u'Dairy and Egg Products',
      u'name': u'Egg, quail, whole, fresh, raw',
      u'ndbno': u'01140',
      u'offset': 5}]



__Exercise 1.2.__ Use your search function to get NDB numbers for the foods in the `fresh.csv` file. It's okay if you don't get an NDB number for every food, but try to come up with a strategy that gets most of them. Discuss your strategy in a short paragraph.

Hints:

* The foods are all raw and unbranded.
* You can test search terms with the [online search page](https://ndb.nal.usda.gov/ndb/search/list).
* You can convert the output of `ndb_search()` to a data frame with `pd.DataFrame()`.
* The string methods for [Python](https://docs.python.org/2/library/stdtypes.html#string-methods) and [Pandas](http://pandas.pydata.org/pandas-docs/stable/text.html#method-summary) are useful here. It's okay if you use _simple_ regular expressions in the Pandas methods, although this exercise can be solved without them.
* You can merge data frames that have a column in common with `pd.merge()`.

__The strategy I used below incorporated modifying the ndb_search() function that I originally created. And then I created a new function ndbnum() which I used to generate the output of the 'ndbnumber' from within the JSON extract. Within ndbnum(), I made it so that the entry was split by the '_' underscore and then the string 'raw' was added to the end. This way the ndb_search() would generate results that were more accurate. This proved to be the easiest method and involved the least amount of hardcode, aside from kiwi. Next was that I tested this out by running a for loop and returning it all within a pandas DataFrame as datalist. While not all of the entries returned are correct, the results were very accurate. There was trouble with kiwi, cabbage, and cucumber. And then finally appended this to the fresh.csv that we were given for this assignment and made a new row called 'ndbnumber'.__


```python
def ndb_search(search,ds):
    url='https://api.nal.usda.gov/ndb/search'
    response=requests.get(url, params = {
        "q": search,
        "api_key": key,
        "format": "json",
        "ds": ds
    })
   
            
    response.raise_for_status()
    test=response.json()
    return test['list']['item']


ndb_search("cherries raw", "Standard Reference")



# if 'search' fails because of list it is because it cannot read it. msut get rid of the '_' or keep only first word
# ex. collard_green must be collard, but green_bean works. and acorn_squash should be 'acorn squash' not just 'acorn'
```




    [{u'ds': u'SR',
      u'group': u'Fruits and Fruit Juices',
      u'name': u'Pitanga, (surinam-cherry), raw',
      u'ndbno': u'09276',
      u'offset': 0},
     {u'ds': u'SR',
      u'group': u'Fruits and Fruit Juices',
      u'name': u'Acerola, (west indian cherry), raw',
      u'ndbno': u'09001',
      u'offset': 1},
     {u'ds': u'SR',
      u'group': u'Fruits and Fruit Juices',
      u'name': u'Cherries, sweet, raw',
      u'ndbno': u'09070',
      u'offset': 2},
     {u'ds': u'SR',
      u'group': u'Fruits and Fruit Juices',
      u'name': u'Cherries, sour, red, raw',
      u'ndbno': u'09063',
      u'offset': 3}]




```python
fresh=pd.read_csv("fresh.csv")
fresh.food[0]
def ndbnum(search_term):
    
    search_term=search_term.partition('_')[0]+' '+search_term.partition('_')[2] + ' raw'
    #search_term=search_term+' ,raw'    
    if search_term=='kiwi  raw':
        search_term='kiwi'
    data=ndb_search(search_term,"Standard Reference")
    for entry in fresh.iterrows():
        df= {'Food name' : data[0]["name"],'ndbnumber' : data[0]['ndbno']}       
        df=pd.DataFrame(df,index=[0])
        #df2.append(df,ignore_index=True) ## dataframe aappend
    return df
ndbnum('watermelon')
#output, must be able to modify search term

```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Food name</th>
      <th>ndbnumber</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Watermelon, raw</td>
      <td>09326</td>
    </tr>
  </tbody>
</table>
</div>




```python
## Test example
datalist=pd.DataFrame()
#range below goes to 32 for w/e reason
for i in range(0,49):
    str=fresh.food[i]
    #str=str.partition('_')[0]
    hi=ndbnum(str)
    hi=pd.DataFrame(hi)
    datalist=datalist.append(hi,ignore_index=True)
datalist.head()  
#datalist
#datalist.drop_duplicates()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Food name</th>
      <th>ndbnumber</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Watermelon, raw</td>
      <td>09326</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Melons, cantaloupe, raw</td>
      <td>09181</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Tangerine juice, raw</td>
      <td>09221</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Strawberries, raw</td>
      <td>09316</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Plums, raw</td>
      <td>09279</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Final 1.2
ndbColumn=pd.DataFrame()
for i, row in fresh.iterrows():
    str1=fresh.food[i]
    hi=ndbnum(str1)
    hi=pd.DataFrame(hi)
    ndbColumn=ndbColumn.append(hi,ignore_index=True)
ndbColumn.drop_duplicates()
fresh['ndbnumber']=pd.Series(ndbColumn.ndbnumber)
```


```python
fresh
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>form</th>
      <th>price_per_lb</th>
      <th>yield</th>
      <th>lb_per_cup</th>
      <th>price_per_cup</th>
      <th>food</th>
      <th>type</th>
      <th>ndbnumber</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Fresh1</td>
      <td>0.333412</td>
      <td>0.520000</td>
      <td>0.330693</td>
      <td>0.212033</td>
      <td>watermelon</td>
      <td>fruit</td>
      <td>09326</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Fresh1</td>
      <td>0.535874</td>
      <td>0.510000</td>
      <td>0.374786</td>
      <td>0.393800</td>
      <td>cantaloupe</td>
      <td>fruit</td>
      <td>09181</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Fresh1</td>
      <td>1.377962</td>
      <td>0.740000</td>
      <td>0.407855</td>
      <td>0.759471</td>
      <td>tangerines</td>
      <td>fruit</td>
      <td>09221</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Fresh1</td>
      <td>2.358808</td>
      <td>0.940000</td>
      <td>0.319670</td>
      <td>0.802171</td>
      <td>strawberries</td>
      <td>fruit</td>
      <td>09316</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Fresh1</td>
      <td>1.827416</td>
      <td>0.940000</td>
      <td>0.363763</td>
      <td>0.707176</td>
      <td>plums</td>
      <td>fruit</td>
      <td>09279</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Fresh1</td>
      <td>1.035173</td>
      <td>0.730000</td>
      <td>0.407855</td>
      <td>0.578357</td>
      <td>oranges</td>
      <td>fruit</td>
      <td>09201</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Fresh1</td>
      <td>6.975811</td>
      <td>0.960000</td>
      <td>0.319670</td>
      <td>2.322874</td>
      <td>raspberries</td>
      <td>fruit</td>
      <td>09302</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Fresh1</td>
      <td>2.173590</td>
      <td>0.560000</td>
      <td>0.341717</td>
      <td>1.326342</td>
      <td>pomegranate</td>
      <td>fruit</td>
      <td>09286</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Fresh1</td>
      <td>0.627662</td>
      <td>0.510000</td>
      <td>0.363763</td>
      <td>0.447686</td>
      <td>pineapple</td>
      <td>fruit</td>
      <td>09266</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Fresh1</td>
      <td>3.040072</td>
      <td>0.930000</td>
      <td>0.363763</td>
      <td>1.189102</td>
      <td>apricots</td>
      <td>fruit</td>
      <td>09021</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Fresh1</td>
      <td>0.796656</td>
      <td>0.460000</td>
      <td>0.374786</td>
      <td>0.649077</td>
      <td>honeydew</td>
      <td>fruit</td>
      <td>09184</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Fresh1</td>
      <td>1.298012</td>
      <td>0.620000</td>
      <td>0.308647</td>
      <td>0.646174</td>
      <td>papaya</td>
      <td>fruit</td>
      <td>09226</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Fresh1</td>
      <td>2.044683</td>
      <td>0.760000</td>
      <td>0.385809</td>
      <td>1.037970</td>
      <td>kiwi</td>
      <td>fruit</td>
      <td>14161</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Fresh1</td>
      <td>3.592990</td>
      <td>0.920000</td>
      <td>0.341717</td>
      <td>1.334548</td>
      <td>cherries</td>
      <td>fruit</td>
      <td>09276</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Fresh1</td>
      <td>0.566983</td>
      <td>0.640000</td>
      <td>0.330693</td>
      <td>0.292965</td>
      <td>bananas</td>
      <td>fruit</td>
      <td>09040</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Fresh1</td>
      <td>1.567515</td>
      <td>0.900000</td>
      <td>0.242508</td>
      <td>0.422373</td>
      <td>apples</td>
      <td>fruit</td>
      <td>09003</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Fresh1</td>
      <td>1.591187</td>
      <td>0.960000</td>
      <td>0.341717</td>
      <td>0.566390</td>
      <td>peaches</td>
      <td>fruit</td>
      <td>09236</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Fresh1</td>
      <td>1.761148</td>
      <td>0.910000</td>
      <td>0.319670</td>
      <td>0.618667</td>
      <td>nectarines</td>
      <td>fruit</td>
      <td>09191</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Fresh1</td>
      <td>1.461575</td>
      <td>0.900000</td>
      <td>0.363763</td>
      <td>0.590740</td>
      <td>pears</td>
      <td>fruit</td>
      <td>09252</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Fresh1</td>
      <td>0.897802</td>
      <td>0.490000</td>
      <td>0.462971</td>
      <td>0.848278</td>
      <td>grapefruit</td>
      <td>fruit</td>
      <td>09117</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Fresh1</td>
      <td>5.774708</td>
      <td>0.960000</td>
      <td>0.319670</td>
      <td>1.922919</td>
      <td>blackberries</td>
      <td>fruit</td>
      <td>09042</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Fresh1</td>
      <td>2.093827</td>
      <td>0.960000</td>
      <td>0.330693</td>
      <td>0.721266</td>
      <td>grapes</td>
      <td>fruit</td>
      <td>09129</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Fresh1</td>
      <td>4.734622</td>
      <td>0.950000</td>
      <td>0.319670</td>
      <td>1.593177</td>
      <td>blueberries</td>
      <td>fruit</td>
      <td>09050</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Fresh1</td>
      <td>1.377563</td>
      <td>0.710000</td>
      <td>0.363763</td>
      <td>0.705783</td>
      <td>mangoes</td>
      <td>fruit</td>
      <td>09176</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Fresh1</td>
      <td>3.213494</td>
      <td>0.493835</td>
      <td>0.396832</td>
      <td>2.582272</td>
      <td>asparagus</td>
      <td>vegetables</td>
      <td>11011</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Fresh, consumed with peel1</td>
      <td>1.295931</td>
      <td>0.970000</td>
      <td>0.264555</td>
      <td>0.353448</td>
      <td>cucumbers</td>
      <td>vegetables</td>
      <td>11206</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Fresh, peeled1</td>
      <td>1.295931</td>
      <td>0.730000</td>
      <td>0.264555</td>
      <td>0.469650</td>
      <td>cucumbers</td>
      <td>vegetables</td>
      <td>11206</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Fresh1</td>
      <td>1.213039</td>
      <td>0.950000</td>
      <td>0.242508</td>
      <td>0.309655</td>
      <td>lettuce_iceberg</td>
      <td>vegetables</td>
      <td>11252</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Fresh1</td>
      <td>1.038107</td>
      <td>0.900000</td>
      <td>0.352740</td>
      <td>0.406868</td>
      <td>onions</td>
      <td>vegetables</td>
      <td>11282</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Fresh1</td>
      <td>2.471749</td>
      <td>0.750000</td>
      <td>0.319670</td>
      <td>1.053526</td>
      <td>turnip_greens</td>
      <td>vegetables</td>
      <td>11568</td>
    </tr>
    <tr>
      <th>30</th>
      <td>Fresh1</td>
      <td>2.569235</td>
      <td>0.840000</td>
      <td>0.308647</td>
      <td>0.944032</td>
      <td>mustard_greens</td>
      <td>vegetables</td>
      <td>11270</td>
    </tr>
    <tr>
      <th>31</th>
      <td>Fresh1</td>
      <td>0.564320</td>
      <td>0.811301</td>
      <td>0.264555</td>
      <td>0.184017</td>
      <td>potatoes</td>
      <td>vegetables</td>
      <td>11362</td>
    </tr>
    <tr>
      <th>32</th>
      <td>Fresh1</td>
      <td>2.630838</td>
      <td>1.160000</td>
      <td>0.286601</td>
      <td>0.650001</td>
      <td>collard_greens</td>
      <td>vegetables</td>
      <td>11161</td>
    </tr>
    <tr>
      <th>33</th>
      <td>Fresh1</td>
      <td>2.139972</td>
      <td>0.846575</td>
      <td>0.275578</td>
      <td>0.696606</td>
      <td>green_beans</td>
      <td>vegetables</td>
      <td>11052</td>
    </tr>
    <tr>
      <th>34</th>
      <td>Fresh1</td>
      <td>1.172248</td>
      <td>0.458554</td>
      <td>0.451948</td>
      <td>1.155360</td>
      <td>acorn_squash</td>
      <td>vegetables</td>
      <td>11482</td>
    </tr>
    <tr>
      <th>35</th>
      <td>Fresh1</td>
      <td>2.277940</td>
      <td>0.820000</td>
      <td>0.264555</td>
      <td>0.734926</td>
      <td>red_peppers</td>
      <td>vegetables</td>
      <td>11821</td>
    </tr>
    <tr>
      <th>36</th>
      <td>Fresh green cabbage1</td>
      <td>0.579208</td>
      <td>0.778797</td>
      <td>0.330693</td>
      <td>0.245944</td>
      <td>cabbage</td>
      <td>vegetables</td>
      <td>11503</td>
    </tr>
    <tr>
      <th>37</th>
      <td>Fresh red cabbage1</td>
      <td>1.056450</td>
      <td>0.779107</td>
      <td>0.330693</td>
      <td>0.448412</td>
      <td>cabbage</td>
      <td>vegetables</td>
      <td>11503</td>
    </tr>
    <tr>
      <th>38</th>
      <td>Fresh1</td>
      <td>0.918897</td>
      <td>0.811301</td>
      <td>0.440925</td>
      <td>0.499400</td>
      <td>sweet_potatoes</td>
      <td>vegetables</td>
      <td>11507</td>
    </tr>
    <tr>
      <th>39</th>
      <td>Fresh1</td>
      <td>1.639477</td>
      <td>0.769500</td>
      <td>0.396832</td>
      <td>0.845480</td>
      <td>summer_squash</td>
      <td>vegetables</td>
      <td>11475</td>
    </tr>
    <tr>
      <th>40</th>
      <td>Fresh1</td>
      <td>1.311629</td>
      <td>0.900000</td>
      <td>0.275578</td>
      <td>0.401618</td>
      <td>radish</td>
      <td>vegetables</td>
      <td>11429</td>
    </tr>
    <tr>
      <th>41</th>
      <td>Fresh1</td>
      <td>1.244737</td>
      <td>0.714000</td>
      <td>0.451948</td>
      <td>0.787893</td>
      <td>butternut_squash</td>
      <td>vegetables</td>
      <td>11485</td>
    </tr>
    <tr>
      <th>42</th>
      <td>Fresh1</td>
      <td>2.235874</td>
      <td>0.740753</td>
      <td>0.319670</td>
      <td>0.964886</td>
      <td>avocados</td>
      <td>vegetables</td>
      <td>09038</td>
    </tr>
    <tr>
      <th>43</th>
      <td>Fresh1</td>
      <td>2.807302</td>
      <td>1.050000</td>
      <td>0.286601</td>
      <td>0.766262</td>
      <td>kale</td>
      <td>vegetables</td>
      <td>11233</td>
    </tr>
    <tr>
      <th>44</th>
      <td>Fresh1</td>
      <td>2.213050</td>
      <td>0.375309</td>
      <td>0.385809</td>
      <td>2.274967</td>
      <td>artichoke</td>
      <td>vegetables</td>
      <td>11226</td>
    </tr>
    <tr>
      <th>45</th>
      <td>Fresh1</td>
      <td>3.213552</td>
      <td>0.769474</td>
      <td>0.352740</td>
      <td>1.473146</td>
      <td>okra</td>
      <td>vegetables</td>
      <td>11278</td>
    </tr>
    <tr>
      <th>46</th>
      <td>Fresh1</td>
      <td>1.410363</td>
      <td>0.820000</td>
      <td>0.264555</td>
      <td>0.455022</td>
      <td>green_peppers</td>
      <td>vegetables</td>
      <td>11333</td>
    </tr>
    <tr>
      <th>47</th>
      <td>Fresh1</td>
      <td>2.763553</td>
      <td>1.060000</td>
      <td>0.341717</td>
      <td>0.890898</td>
      <td>brussels_sprouts</td>
      <td>vegetables</td>
      <td>11098</td>
    </tr>
    <tr>
      <th>48</th>
      <td>Fresh1</td>
      <td>2.690623</td>
      <td>0.540000</td>
      <td>0.363763</td>
      <td>1.812497</td>
      <td>corn_sweet</td>
      <td>vegetables</td>
      <td>11900</td>
    </tr>
  </tbody>
</table>
</div>



__Exercise 1.3.__ Read the [food reports V2 documentation](https://ndb.nal.usda.gov/ndb/doc/apilist/API-FOOD-REPORTV2.md), then write a function called `ndb_report()` that requests a _basic_ food report. The function should accept the NDB number as an argument and return the list of nutrients for the food.

Note that the report url is: `https://api.nal.usda.gov/ndb/V2/reports`

For example, for `"09279"` (raw plums) the first element of the returned list should be:

```python
{u'group': u'Proximates',
 u'measures': [{u'eqv': 165.0,
   u'eunit': u'g',
   u'label': u'cup, sliced',
   u'qty': 1.0,
   u'value': u'143.93'},
  {u'eqv': 66.0,
   u'eunit': u'g',
   u'label': u'fruit (2-1/8" dia)',
   u'qty': 1.0,
   u'value': u'57.57'},
  {u'eqv': 151.0,
   u'eunit': u'g',
   u'label': u'NLEA serving',
   u'qty': 1.0,
   u'value': u'131.72'}],
 u'name': u'Water',
 u'nutrient_id': u'255',
 u'unit': u'g',
 u'value': u'87.23'}
```

Be sure to document and test your function.


```python
def ndb_report(NDB):
    url='https://api.nal.usda.gov/ndb/V2/reports'
    response=requests.get(url, params = {
        "ndbno": NDB,
        "api_key": key,
        "format": "json",  
    })
    
    response.raise_for_status()
    test=response.json()
    return test['foods'][0]['food']['nutrients']
#ndb_report("09279")
ndb_report("09279")[0]

```




    {u'group': u'Proximates',
     u'measures': [{u'eqv': 165.0,
       u'eunit': u'g',
       u'label': u'cup, sliced',
       u'qty': 1.0,
       u'value': u'143.93'},
      {u'eqv': 66.0,
       u'eunit': u'g',
       u'label': u'fruit (2-1/8" dia)',
       u'qty': 1.0,
       u'value': u'57.57'},
      {u'eqv': 151.0,
       u'eunit': u'g',
       u'label': u'NLEA serving',
       u'qty': 1.0,
       u'value': u'131.72'}],
     u'name': u'Water',
     u'nutrient_id': u'255',
     u'unit': u'g',
     u'value': u'87.23'}




```python
ndb_report(fresh.ndbnumber[0])[8]
```




    {u'group': u'Minerals',
     u'measures': [{u'eqv': 154.0,
       u'eunit': u'g',
       u'label': u'cup, balls',
       u'qty': 1.0,
       u'value': u'0.37'},
      {u'eqv': 152.0,
       u'eunit': u'g',
       u'label': u'cup, diced',
       u'qty': 1.0,
       u'value': u'0.36'},
      {u'eqv': 4518.0,
       u'eunit': u'g',
       u'label': u'melon (15" long x 7-1/2" dia)',
       u'qty': 1.0,
       u'value': u'10.84'},
      {u'eqv': 286.0,
       u'eunit': u'g',
       u'label': u'wedge (approx 1/16 of melon)',
       u'qty': 1.0,
       u'value': u'0.69'},
      {u'eqv': 122.0,
       u'eunit': u'g',
       u'label': u'watermelon balls',
       u'qty': 10.0,
       u'value': u'0.29'},
      {u'eqv': 280.0,
       u'eunit': u'g',
       u'label': u'NLEA serving',
       u'qty': 1.0,
       u'value': u'0.67'}],
     u'name': u'Iron, Fe',
     u'nutrient_id': u'303',
     u'unit': u'mg',
     u'value': u'0.24'}




```python
# Add calories and units for all food type to 'fresh' data frame
calories_info=pd.DataFrame()
for i, row in fresh.iterrows():
    measure=ndb_report(fresh.ndbnumber[i])
    calories_df = {'Calories_kCal' : measure[1]['value']}       
    calories_df=pd.DataFrame(calories_df,index=[0])
    calories_info=calories_info.append(calories_df,ignore_index=True)
calories_info
fresh['Calories_kCal']=pd.Series(calories_info.Calories_kCal)
#fresh
```


```python
# Add protein and units for all food type to 'fresh' data frame
protein_info=pd.DataFrame()
for i, row in fresh.iterrows():
    measure=ndb_report(fresh.ndbnumber[i])
    protein_df = {'Protein_g' : measure[2]['value'], 'unit':measure[2]['unit']}       
    protein_df=pd.DataFrame(protein_df,index=[0])
    protein_info=protein_info.append(protein_df,ignore_index=True)
protein_info
fresh['Protein_g']=pd.Series(protein_info.Protein_g)
#fresh
```

__Exercise 1.4.__ Which foods provide the best combination of price, yield, and nutrition? You can use kilocalories as a measure of "nutrition" here, but more a detailed analysis is better. Use plots to support your analysis.


```python
# Convert the 'object' type to float64 or int64 so it can be plotted
fresh.head()
fresh.dtypes
fresh = fresh.convert_objects(convert_numeric=True)
#.addlegend()
#.set_axis_labels()
```

    /Users/matthewlee/anaconda2/lib/python2.7/site-packages/ipykernel/__main__.py:4: FutureWarning: convert_objects is deprecated.  Use the data-type specific converters pd.to_datetime, pd.to_timedelta and pd.to_numeric.



```python
sns.jointplot(x='Calories_kCal',y='price_per_lb',data=fresh,color='g')
plt.show()
```


![png](output_19_0.png)



```python
plot=sns.FacetGrid(data=fresh,col="type")
plot = (plot.map(plt.scatter, "Protein_g", "price_per_lb",))
plt.show()
```


![png](output_20_0.png)



```python
#barplot works for discrete vs continuous
sns.barplot(x="type",y="yield",data=fresh)
plt.show()
```


![png](output_21_0.png)



```python
sns.regplot(x="yield",y="price_per_lb",data=fresh)
plt.show()
```


![png](output_22_0.png)



```python
sns.regplot(x="Calories_kCal",y="price_per_lb",data=fresh,color='g')
plt.show()
```


![png](output_23_0.png)


<p> From our plots we can see that calories for all the foods have a slight positive correlation with price per pound. The reason we chose to analyze price per pound rather than price per cup was because this is the unit that is commonly used in grocery stores in the United States.  </p> <p> When we analyze protein per food item, the fruits and vegetables were split up when comparing their prices. We can see that price per pound for fruit has a wider range of prices. And for the vegetables, the protein content does not have much indication on how the vegetable is priced.</p>
