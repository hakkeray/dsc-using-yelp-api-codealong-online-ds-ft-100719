
# Using the Yelp API - Codealong

## Introduction

Now that we've discussed HTTP requests and OAuth, it's time to practice applying those skills to a production level API. In this codealong, we'll take you through the process of signing up for an OAuth token and then using that to make requests to the Yelp API!

## Objectives

You will be able to:

* Make requests using OAuth
* Use the JSON module to load and parse JSON documents
* Convert JSON to a pandas dataframe

## Generating Access Tokens

As discussed, in order to use many APIs, one needs to use OAuth which requires an access token. As such, our first step will be to generate this login information so that we can start making some requests.  

With that, let's go grab an access token from an API site and make some API calls!
Point your browser over to this [yelp page](https://www.yelp.com/developers/v3/manage_app) and start creating an app in order to obtain and API access token:

![](./images/yelp_app.png)


You can either sign in to an existing Yelp account or create a new one if needed.

On the page you see above, simply fill out some sample information such as "Flatiron Edu API Example" for the app name, or whatever floats your boat. Afterward, you should be presented with an API key that you can use to make requests!

With that, let's set up our authentication tokens so that we can start making some API calls!

### Should I publicly share my passwords on Github?

When using an API that requires an API key and password you should **NEVER** hardcode theses values into your main file. When you upload your project onto github it is completely public and vulnerable to attack. Assume that if you put sensitive information publicly on the internet it will be found and abused. 

To this end, how can we easily access our API key without opening ourselves up to vulnerabilities?

There are many ways to store sensitive information but we will go with this method. 

#### Move to your home (root) directory:

```
cd ~
```

#### Now make the `.secret/` directory:

```
mkdir .secret
```

This will create a new folder in your home directory where you can store files for any of the API information you have. 

Can you find the file you just made in your terminal? 
NOTE: dot files won't show up with just `ls` you must use the show all command as well `ls -a`


#### Move into the newly created `.secret/` folder and create a file using vscode or any text editor to store your yelp API login info.

```
cd .secret/
code yelp_api.json
```

In this file, let's create a dictionary of values representing the client id and API key that looks something like this:

`{"api_key": "input api key here!"}`

NOTE: Double quotes are important! You'll copy and paste the `api_key` value that yelp grants you after you create your app.

Ok, so now we have a file in our .secret folder on our home directory. Safe and sound (mostly) from anyone trying to steal our info off github.

#### Finally, let's get our client id and API key into our jupyter notebook.

If we remember that our file is just a regular JSON file, open the file and pull out the appropriate information from the `~/.secret/yelp_api.json` file. 



```python
import json

def get_keys(path):
    with open(path) as f:
        return json.load(f)
```

> **Note**: Change the file path below to be your root directory. 
If you're not sure what your username is, check it with `pwd`  
For example, my current working directory is ```/Users/matthew.mitchell/Documents/dsc-using-yelp-api-codealong```  
So the line below would become:
```keys = get_keys("/Users/matthew.mitchell/.secret/yelp_api.json")```


```python
keys = get_keys("/Users/hakkeray/datascience/mod2/sec12/dsc-using-yelp-api-codealong-online-ds-ft-100719/.secret/yelp_api.json")

api_key = keys['api_key']

#While you may wish to print out these API keys to check that they imported properly,
#be sure to clear the output before uploading to Github. 
#Again, you don't want your keys stolen!!!
```

## An Example Request with OAuth <a id="oauth_request"></a>
https://www.yelp.com/developers/documentation/v3/get_started

In the next lesson, we'll further dissect how to read and translate online documentation like the link here. For now, let's simply look at an example request and dissect it into its constituent parts:


```python
import requests
term = 'Mexican'
location = 'Astoria NY'
SEARCH_LIMIT = 10

url = 'https://api.yelp.com/v3/businesses/search'

headers = {
        'Authorization': 'Bearer {}'.format(api_key),
    }

url_params = {
                'term': term.replace(' ', '+'),
                'location': location.replace(' ', '+'),
                'limit': SEARCH_LIMIT
            }
response = requests.get(url, headers=headers, params=url_params)
print(response)
print(type(response.text))
print(response.text[:1000])
```

    <Response [200]>
    <class 'str'>
    {"businesses": [{"id": "jeWIYbgBho9vBDhc5S1xvg", "alias": "chanos-cantina-astoria", "name": "Chano's Cantina", "image_url": "https://s3-media1.fl.yelpcdn.com/bphoto/B34FXjfQrAxMkWUpb3Pv5A/o.jpg", "is_closed": false, "url": "https://www.yelp.com/biz/chanos-cantina-astoria?adjust_creative=12IMkq-xxD-jcDHiyMb-IQ&utm_campaign=yelp_api_v3&utm_medium=api_v3_business_search&utm_source=12IMkq-xxD-jcDHiyMb-IQ", "review_count": 163, "categories": [{"alias": "cocktailbars", "title": "Cocktail Bars"}, {"alias": "newmexican", "title": "New Mexican Cuisine"}], "rating": 4.0, "coordinates": {"latitude": 40.756621, "longitude": -73.929336}, "transactions": ["pickup", "restaurant_reservation", "delivery"], "price": "$$", "location": {"address1": "35-55 31st", "address2": "", "address3": "", "city": "Astoria", "zip_code": "11106", "country": "US", "state": "NY", "display_address": ["35-55 31st", "Astoria, NY 11106"]}, "phone": "+19178327261", "display_phone": "(917) 832-7261", "distance": 1290.427487513


## Breaking Down the Request

As you can see, there are three main parts to our request.  
  
They are:
* The URL
* The header
* The parameters
  
The URL is fairly straightforward and is simply the base URL as described in the documentation (again more details in the upcoming lesson).

The header is a dictionary of key-value pairs. In this case, we are using a fairly standard header used by many APIs. It has a strict form where 'Authorization' is the key and 'Bearer YourApiKey' is the value.

The parameters are the filters that we wish to pass into the query. These will be embedded into the URL when the request is made to the API. Similar to the header, they form key-value pairs. Valid key parameters by which to structure your queries are described in the API documentation which we'll look at further shortly. A final important note, however, is the need to replace spaces with "+". This is standard to many requests as URLs cannot contain spaces. (Note that the header itself isn't directly embedded into the URL itself and as such, the space between 'Bearer' and YourApiKey is valid.)


## The Response

As before, our response object has both a status code, as well as the data itself. With that, let's start with a little data exploration!


```python
response.json().keys()
```




    dict_keys(['businesses', 'total', 'region'])



Now let's go a bit further and start to preview what's stored in each of the values for these keys.


```python
for key in response.json().keys():
    print(key)
    value = response.json()[key] #Use standard dictionary formatting
    print(type(value)) #What type is it?
    print('\n\n') #Separate out data
```

    businesses
    <class 'list'>
    
    
    
    total
    <class 'int'>
    
    
    
    region
    <class 'dict'>
    
    
    


Let's continue to preview these further to get a little better acquainted.


```python
response.json()['businesses'][:2]
```




    [{'id': 'jeWIYbgBho9vBDhc5S1xvg',
      'alias': 'chanos-cantina-astoria',
      'name': "Chano's Cantina",
      'image_url': 'https://s3-media1.fl.yelpcdn.com/bphoto/B34FXjfQrAxMkWUpb3Pv5A/o.jpg',
      'is_closed': False,
      'url': 'https://www.yelp.com/biz/chanos-cantina-astoria?adjust_creative=12IMkq-xxD-jcDHiyMb-IQ&utm_campaign=yelp_api_v3&utm_medium=api_v3_business_search&utm_source=12IMkq-xxD-jcDHiyMb-IQ',
      'review_count': 163,
      'categories': [{'alias': 'cocktailbars', 'title': 'Cocktail Bars'},
       {'alias': 'newmexican', 'title': 'New Mexican Cuisine'}],
      'rating': 4.0,
      'coordinates': {'latitude': 40.756621, 'longitude': -73.929336},
      'transactions': ['pickup', 'restaurant_reservation', 'delivery'],
      'price': '$$',
      'location': {'address1': '35-55 31st',
       'address2': '',
       'address3': '',
       'city': 'Astoria',
       'zip_code': '11106',
       'country': 'US',
       'state': 'NY',
       'display_address': ['35-55 31st', 'Astoria, NY 11106']},
      'phone': '+19178327261',
      'display_phone': '(917) 832-7261',
      'distance': 1290.4274875130448},
     {'id': '4cVajR3KnJh7cK9u_TiBAQ',
      'alias': 'orale-mex-food-astoria',
      'name': 'Orale Mex Food',
      'image_url': 'https://s3-media2.fl.yelpcdn.com/bphoto/qqQwbjaIqkjj_OKB56Q8Fw/o.jpg',
      'is_closed': False,
      'url': 'https://www.yelp.com/biz/orale-mex-food-astoria?adjust_creative=12IMkq-xxD-jcDHiyMb-IQ&utm_campaign=yelp_api_v3&utm_medium=api_v3_business_search&utm_source=12IMkq-xxD-jcDHiyMb-IQ',
      'review_count': 2,
      'categories': [{'alias': 'mexican', 'title': 'Mexican'}],
      'rating': 4.5,
      'coordinates': {'latitude': 40.770261, 'longitude': -73.918361},
      'transactions': ['pickup', 'delivery'],
      'location': {'address1': '30-19 Astoria Blvd',
       'address2': '',
       'address3': '',
       'city': 'Astoria',
       'zip_code': '11102',
       'country': 'US',
       'state': 'NY',
       'display_address': ['30-19 Astoria Blvd', 'Astoria, NY 11102']},
      'phone': '+19173961753',
      'display_phone': '(917) 396-1753',
      'distance': 494.8529458975233}]




```python
response.json()['total']
```




    675




```python
response.json()['region']
```




    {'center': {'longitude': -73.92219543457031, 'latitude': 40.76688875374591}}



As you can see, we're primarily interested in the 'businesses' entry. 

Let's go ahead and create a dataframe from that.


```python
import pandas as pd

df = pd.DataFrame.from_dict(response.json()['businesses'])
print(len(df)) #Print how many rows
print(df.columns) #Print column names
df.head() #Previews the first five rows. 
#You could also write df.head(10) to preview 10 rows or df.tail() to see the bottom
```

    10
    Index(['id', 'alias', 'name', 'image_url', 'is_closed', 'url', 'review_count',
           'categories', 'rating', 'coordinates', 'transactions', 'price',
           'location', 'phone', 'display_phone', 'distance'],
          dtype='object')





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>alias</th>
      <th>name</th>
      <th>image_url</th>
      <th>is_closed</th>
      <th>url</th>
      <th>review_count</th>
      <th>categories</th>
      <th>rating</th>
      <th>coordinates</th>
      <th>transactions</th>
      <th>price</th>
      <th>location</th>
      <th>phone</th>
      <th>display_phone</th>
      <th>distance</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>jeWIYbgBho9vBDhc5S1xvg</td>
      <td>chanos-cantina-astoria</td>
      <td>Chano's Cantina</td>
      <td>https://s3-media1.fl.yelpcdn.com/bphoto/B34FXj...</td>
      <td>False</td>
      <td>https://www.yelp.com/biz/chanos-cantina-astori...</td>
      <td>163</td>
      <td>[{'alias': 'cocktailbars', 'title': 'Cocktail ...</td>
      <td>4.0</td>
      <td>{'latitude': 40.756621, 'longitude': -73.929336}</td>
      <td>[pickup, restaurant_reservation, delivery]</td>
      <td>$$</td>
      <td>{'address1': '35-55 31st', 'address2': '', 'ad...</td>
      <td>+19178327261</td>
      <td>(917) 832-7261</td>
      <td>1290.427488</td>
    </tr>
    <tr>
      <td>1</td>
      <td>4cVajR3KnJh7cK9u_TiBAQ</td>
      <td>orale-mex-food-astoria</td>
      <td>Orale Mex Food</td>
      <td>https://s3-media2.fl.yelpcdn.com/bphoto/qqQwbj...</td>
      <td>False</td>
      <td>https://www.yelp.com/biz/orale-mex-food-astori...</td>
      <td>2</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}]</td>
      <td>4.5</td>
      <td>{'latitude': 40.770261, 'longitude': -73.918361}</td>
      <td>[pickup, delivery]</td>
      <td>NaN</td>
      <td>{'address1': '30-19 Astoria Blvd', 'address2':...</td>
      <td>+19173961753</td>
      <td>(917) 396-1753</td>
      <td>494.852946</td>
    </tr>
    <tr>
      <td>2</td>
      <td>6AJwsgXr7YwsqneGVAdgzw</td>
      <td>las-catrinas-mexican-bar-and-eatery-astoria</td>
      <td>Las Catrinas Mexican Bar &amp; Eatery</td>
      <td>https://s3-media3.fl.yelpcdn.com/bphoto/3uevye...</td>
      <td>False</td>
      <td>https://www.yelp.com/biz/las-catrinas-mexican-...</td>
      <td>303</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}, {'a...</td>
      <td>4.0</td>
      <td>{'latitude': 40.7614214682633, 'longitude': -7...</td>
      <td>[pickup, delivery]</td>
      <td>$$</td>
      <td>{'address1': '32-02 Broadway', 'address2': '',...</td>
      <td>+19177450969</td>
      <td>(917) 745-0969</td>
      <td>642.525771</td>
    </tr>
    <tr>
      <td>3</td>
      <td>AUyKmFjpaVLwc3awfUnqgQ</td>
      <td>chela-and-garnacha-astoria</td>
      <td>Chela &amp; Garnacha</td>
      <td>https://s3-media1.fl.yelpcdn.com/bphoto/ChVbA1...</td>
      <td>False</td>
      <td>https://www.yelp.com/biz/chela-and-garnacha-as...</td>
      <td>363</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}, {'a...</td>
      <td>4.5</td>
      <td>{'latitude': 40.7557171543477, 'longitude': -7...</td>
      <td>[pickup, delivery]</td>
      <td>$$</td>
      <td>{'address1': '33-09 36th Ave', 'address2': '',...</td>
      <td>+19178326876</td>
      <td>(917) 832-6876</td>
      <td>1318.326547</td>
    </tr>
    <tr>
      <td>4</td>
      <td>jzVv_21473lAMYXIhVbuTA</td>
      <td>de-mole-astoria-astoria</td>
      <td>De Mole Astoria</td>
      <td>https://s3-media1.fl.yelpcdn.com/bphoto/zZTTfd...</td>
      <td>False</td>
      <td>https://www.yelp.com/biz/de-mole-astoria-astor...</td>
      <td>359</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}]</td>
      <td>4.0</td>
      <td>{'latitude': 40.7625999, 'longitude': -73.9129...</td>
      <td>[pickup, delivery]</td>
      <td>$$</td>
      <td>{'address1': '4220 30th Ave', 'address2': '', ...</td>
      <td>+17187771655</td>
      <td>(718) 777-1655</td>
      <td>918.092772</td>
    </tr>
  </tbody>
</table>
</div>



## Summary <a id="sum"></a>

Congratulations! We've covered a lot here! We took some of your previous knowledge with HTTP requests and OAuth in order to leverage an enterprise API! Then we made some requests to retrieve information that came back as a JSON format. We then transformed this data into a dataframe using the Pandas package. In the next lab, we'll break down how to read API documentation!
