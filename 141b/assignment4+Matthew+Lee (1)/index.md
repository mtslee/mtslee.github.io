


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

__Exercise 1.2.__ Use your search function to get NDB numbers for the foods in the `fresh.csv` file. It's okay if you don't get an NDB number for every food, but try to come up with a strategy that gets most of them. Discuss your strategy in a short paragraph.

Hints:

* The foods are all raw and unbranded.
* You can test search terms with the [online search page](https://ndb.nal.usda.gov/ndb/search/list).
* You can convert the output of `ndb_search()` to a data frame with `pd.DataFrame()`.
* The string methods for [Python](https://docs.python.org/2/library/stdtypes.html#string-methods) and [Pandas](http://pandas.pydata.org/pandas-docs/stable/text.html#method-summary) are useful here. It's okay if you use _simple_ regular expressions in the Pandas methods, although this exercise can be solved without them.
* You can merge data frames that have a column in common with `pd.merge()`.

<p>The strategy I used below incorporated modifying the ndb_search() function that I originally created. And then I created a new function ndbnum() which I used to generate the output of the 'ndbnumber' from within the JSON extract. Within ndbnum(), I made it so that the entry was split by the '_' underscore and then the string 'raw' was added to the end. This way the ndb_search() would generate results that were more accurate. This proved to be the easiest method and involved the least amount of hardcode, aside from kiwi. Next was that I tested this out by running a for loop and returning it all within a pandas DataFrame as datalist. While not all of the entries returned are correct, the results were very accurate. There was trouble with kiwi, cabbage, and cucumber. And then finally appended this to the fresh.csv that we were given for this assignment and made a new row called 'ndbnumber'. </p>


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

```


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
#ndbnum('watermelon')
```


```python
datalist=pd.DataFrame()
for i in range(0,49):
    str=fresh.food[i]
    #str=str.partition('_')[0]
    hi=ndbnum(str)
    hi=pd.DataFrame(hi)
    datalist=datalist.append(hi,ignore_index=True)
datalist.head()  
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
fresh.head()
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
      <td>0.52</td>
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
      <td>0.51</td>
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
      <td>0.74</td>
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
      <td>0.94</td>
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
      <td>0.94</td>
      <td>0.363763</td>
      <td>0.707176</td>
      <td>plums</td>
      <td>fruit</td>
      <td>09279</td>
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
ndb_report("09279")[0]

```


```python
ndb_report(fresh.ndbnumber[0])[8]
```


```python
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


![png](output_18_0.png)



```python
plot=sns.FacetGrid(data=fresh,col="type")
plot = (plot.map(plt.scatter, "Protein_g", "price_per_lb",))
plt.show()
```


![png](output_19_0.png)



```python
#barplot works for discrete vs continuous
sns.barplot(x="type",y="yield",data=fresh)
plt.show()
```


![png](output_20_0.png)



```python
sns.regplot(x="yield",y="price_per_lb",data=fresh)
plt.show()
```


![png](output_21_0.png)



```python
sns.regplot(x="Calories_kCal",y="price_per_lb",data=fresh,color='g')
plt.show()
```


![png](output_22_0.png)


<p> From our plots we can see that calories for all the foods have a slight positive correlation with price per pound. The reason we chose to analyze price per pound rather than price per cup was because this is the unit that is commonly used in grocery stores in the United States.  </p> <p> When we analyze protein per food item, the fruits and vegetables were split up when comparing their prices. We can see that price per pound for fruit has a wider range of prices. And for the vegetables, the protein content does not have much indication on how the vegetable is priced.</p>
