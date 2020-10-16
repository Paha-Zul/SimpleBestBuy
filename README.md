# Simple BestBuy API for PHP

This is a simple high-level PHP client for the [Best Buy developer API](https://developer.bestbuy.com/).
You will need an API key from BestBuy in order to access their API. [More information here.](https://bestbuyapis.github.io/api-documentation/#get-a-key)

# Table of Contents
1. [Quickstart](#quickstart)
2. [Introduction](#introduction)
4. [How to Install](#how-to-install)
    1. [Dependencies](#dependencies)
    2. [From github](#from-github)
    3. [API Key](#api-key)
5. [Usage](#usage)
    1. [ProductOptions](#productoptions)
    2. [APIOptions](#apioptions)
    3. [APIQueryBuilder]($apiquerybuilder)
    4. [BestBuyAPI](#bestbuyapi)
6. [Shortcomings](#shortcomings)
7. [Roadmap](#roadmap)



# Quickstart 

```PHP
$options = new APIOptions(); # ProductOptions to pass into APIQueryBuilder for building a URL
$options->restrictions = ProductOptions::salePrice()."<=29.99";

# The API object
$api = new BestBuyAPI();

# Fetch (up to) the first 100 results from the api
$results = $api->fetch(APIQueryBuilder::products($options));

# Fetch all results from the API
$results = $api->fetchAll(APIQueryBuilder::products($options), 1.0, 5);

# Checking for errors
if($results->error != "")
  handleError();
  
# Using the products returns from the API call
foreach($results->products as $product)
  doSomething();

# Getting a value from the raw json data from the API call
$totalPageSize = $results->rawData->totalPages;
```

# Introduction
This is a small package I made for a personal project and very much a work in progress. I found it messy and hard
to remember the options for the products of BestBuy's API. Therefore, I made a workflow to help make consistent API calls
and a ProductOptions class that contains all of the options I could find. Hopefully in the future it will become more organized
and I will implement the other missing features like categories, recommendations, etc.


# How to install

### Dependencies
Uses the [spatie/enum](https://github.com/spatie/enum) package for strongly typed enums in PHP. You can either open a terminal
in your root directory and run 
```composer require spatie/enum:^3.1``` 

or alternatively you can open the ``composer.json`` file and include it there like so:
```
"require": {
        "spatie/enum": "^3.1"
    }
```
Then you must open a terminal and run ``composer install`` to install the package.
  

### From github
There are 2 ways to pull the release from github. The first is through cloning:
  Open a terminal in the directoy you wish to clone into and run
  ```git clone --depth 1 --branch 0.1-alpha https://github.com/Paha-Zul/SimpleBestBuy```

Alternatively, you can use the release page to download the zip and extract it into your desired directly
once the files are in place.


### API key
You will need an API key from BestBuy in order to access their API. [More information here.](https://bestbuyapis.github.io/api-documentation/#get-a-key)

Once you have an API key for BestBuy, open ``api.json`` and insert the key. This will be inserted in the query during the URl building
by the APIQueryBuilder class. 

Make sure to add ``api.json`` to .gitignore so that it is not exposed if you decide to upload the project
to github/gitlab/gitbucket or any other hosting site.


# Usage
The general workflow of this package is that values from ProductOption are loaded into a APIOptions container which are
used in the APIQueryBuilder to generate a URL that is loaded into the BestBuyApi. So to sum it up:
ProductOptions > APIOptions > APIQueryBuilder > BestBuyAPI->fetch/fetchAll.

Each class and usage is described below.

### ProductOptions
The ProductOptions class contains around 400 static properties (accessed like ProductOptions::sku()) that
correspond to the available properties from the BestBuyAPI. These are used to show parameters
for the returned products or to refine searches. Quick usage:

```PHP
$optionsToShow = [ProductOptions::sku(), ProductOptions::name(), ProductOptions::startDate()];
$refineSearch = ProductOptions::salePrice."<=29.99";
```

### APIOptions
The APIOptions class holds data to be passed in to the APIQueryBuilder to generate a URL to pass into the
BestBuyAPI class. The use of ProductOptions to help build this data is recommended. Quick usage:

```PHP
$options = new APIOptions();
$options->restrictions = ProductOptions::salePrice."<=29.99";
$options->optionsToShow = [ProductOptions::sku(), ProductOptions::name(), ProductOptions::startDate()]
```

### APIQueryBuilder
The APIQueryBuilder is responsible for building a URL from the APIOptions. It will return a string URL that can be passed
in to the BestBuyAPI methods. Quick usage:
```PHP
$options = new APIOptions();
$options->restrictions = ProductOptions::salePrice."<=29.99";
$options->optionsToShow = [ProductOptions::sku(), ProductOptions::name(), ProductOptions::startDate()]

# string URL built for calling from the products API
$url = APIQueryBuilder::products($options);
```

### BestBuyAPI
Contains the methods ``fetch($url)`` and ``fetchAll($url, $delay, $numAttempts)`` to get data from the API.

The ``fetch($url)`` method will return an object containing ``string $error``, ``array $products``, and ``object rawData``.

  - *The ``$error`` is simply a string with the error message. Check ``$results->error === ""`` to validate no error.*
  - *The ``$products`` object is an array of up to 100 returned products from the API call. Refer to the [BestBuy Products API](https://bestbuyapis.github.io/api-documentation/#products-api)
  documentation for further information.*
  - *The ``$rawData`` is the json object returned from the API call. This can be used to get information like total pages from the response.
  Refer to [the response format](https://bestbuyapis.github.io/api-documentation/#response-format) for more information.*

The ``fetchAll(string $url, float $delayBetweenCalls, int $numAttempts)`` will attempt to fetch all results using the cursor mark from each call.
The $delayBetweenCalls parameter sleeps the function between each API call. Use this to prevent reaching the call per second limit.
This differs from the regular ``fetch()`` call in the following ways:

  - *The ``$error`` will only contain the most recent error during the call.*
  - *The ``$products`` will contain as many products as the ``fetchAll()`` call gathered.*
  - *The ``$rawData`` object will only contain the data from the most last call before finishing.*

The $url should be created from the APIQueryBuilder which will handle building and inclusion of the API key. A custom
url can be used if desired. Quick usage:

```PHP
$timeBetweenCalls = 1.0;
$numAttempts = 5; 

$url = APIQueryBuilder::products($options);

$api = new BestBuyAPI();

$results = $api->fetch($url);
# or
$results = $api->fetchAll($url, $timeBetweenCalls, $numAttempts);

# Checking for errors
if($results->error != "")
  handleError();
  
# Using the products returns from the API call. Up to 100 when using fetch()
foreach($results->products as $product)
  doSomething();

# Getting a value from the raw json data from the API call
$totalPageSize = $results->rawData->totalPages;

```

# Shortcomings
Currently only the products API is supported. If other parts of the API are needed, a hand built URL can be passed into the BestBuyAPI functions
to retrieve data.

# Roadmap

- Implement availability
- Implement categories
- Implement open box products
- Implement product recommendations
- Implement products reviews
- Implement stores