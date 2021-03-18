# Search Engine Optimisation

## Executive Summary

The goal of this challenge was to code an optimised search engine for recipes achieving bespoke results (/i.e. supporting the user's food preferences) with minimal wall time. A data set is provided. The search engine is to be pretty basic, returning all recipes that contain all of the provided keywords. However, the user can choose from a number of orderings depending on their food preferences, which the search engine should support.

## Specification

The system provides a function ``search``, with the following specification:
```
def search(query, ordering = 'normal', count = 10):
  ...
```

It `print`s out the results of the search, subject to the following rules:
1. It selects from the set of all recipes that contain __all__ of the words in the query (the positions/order of the words in the recipe are to be ignored).
2. It orders them based on the provided ordering (a string, meaning defined below).
3. It `print`s the top `count` matches only, preserving the order from best to worst. The search engine `print`s just their title, one per line.


### Data set

A file, `recipes.json` is provided, containing 17K recipes. It can be parsed into a Python data structure using the [`json`](https://docs.python.org/3/library/json.html) module. It is a list, where each recipe is a dictionary containing various keys:
* `title` : Name of recipe; you can assume these are unique
* `categories` : A list of tags assigned to the recipe
* `ingredients` : What is in it, as a list
* `directions` : List of steps to make the recipe
* `rating` : A rating, out of 5, of how good it is
* `calories` : How many calories it has
* `protein` : How much protein is in it
* `fat` : How much fat is in it

Note that the data set was obtained via web scrapping and hence is noisy - every key except for `title` is missing from at least one recipe. The code will need to cope with this.

The data set comes from https://www.kaggle.com/hugodarwood/epirecipes/version/2 though it has undergone some preprocessing/cleaning, i.e. removing of duplicates and really 'dodgy' entries.



### Search

The search should check the following parts of the recipe (see data set description below):
* `title`
* `categories`
* `ingredients`
* `directions`

For instance, given the query "banana cheese" you would expect "Banana Layer Cake with Cream Cheese Frosting" in the results. Note that case is ignored ("banana" matches "Banana") and the words __do not__ have to be next to one another, in the same order as the search query or even in the same part of the recipe ("cheese" could appear in the title and "banana" in the ingredients). However, all words in the search query __must__ appear somewhere.



### Tokenisation

In order to match words, tokenisation is carried out as follows:
1. All punctuation and digits are converted into spaces. For punctuation I use the set in [`string.punctuation`](https://docs.python.org/3/library/string.html#string.punctuation), for digits [`string.digits`](https://docs.python.org/3/library/string.html#string.digits).
2. [`split()`](https://docs.python.org/3/library/stdtypes.html#str.split) to extract individual tokens.
3. Ignore any token that is less than $3$ characters long.
4. Make tokens lowercase.

When matching words for search it's only a match if the entire token is matched (There are many scenarios where this simple approach will fail, but it's good enough for this exercise). When carrying out a search the code should ignore terms in the search string that fail the above requirements.



### Ordering

There are three ordering modes to select from / that the search engine will offer, each indicated by passing a string to the `search` function:
* `normal` - Based simply on the number of times the search terms appear in the recipe. A score is calculated and the order is highest to lowest. The score sums the following terms (repeated words are counted multiple times, i.e. "cheese cheese cheese" is $3$ matches to "cheese"):
    * $8 \times$ Number of times a query word appears in the title
    * $4 \times$ Number of times a query word appears in the categories
    * $2 \times$ Number of times a query word appears in the ingredients
    * $1 \times$ Number of times a query word appears in the directions
    * The `rating` of the recipe (if not available assume $0$).

* `simple` - Tries to minimise the complexity of the recipe, for someone who is in a rush. Orders to minimise the number of ingredients multiplied by the numbers of steps in the directions.

* `healthy` - Order from lowest to highest by this cost function:
$$\frac{|\texttt{calories} - 510n|}{510} + 2\frac{|\texttt{protein} - 18n|}{18} + 4\frac{|\texttt{fat} - 150n|}{150}$$
Where $n \in \mathbb{N}^+$ is selected to minimise the cost ($n$ is a positive integer and $n=0$ is not allowed). This can be understood in terms of the numbers $510$, $18$ and $150$ being a third of the recommended daily intake (three meals per day) for an average person, and $n$ being the number of whole meals the person gets out of cooking/making the recipe. So this tries to select recipes that neatly divide into a set of meals that are the right amount to consume for a healthy, balanced diet.

To clarify the use of the ordering string, to get something healthy that contains cheese one might call `search('cheese', 'healthy')`. 

In the case of a recipe that is missing a key in its dictionary the rules are different for each search mode:
* `normal` - Consider a missing entry to be zero matches, but the item is still eligible for inclusion in the results, assuming it can match the search with a different entry.
* `simple` - If a recipe is missing either `ingredients` or `directions` it is dropped from such a search result. Because the data is messy if either of these lists is of length $1$ it should be assumed that the list extraction has failed and the recipe is to also be dropped from the search results.
* `healthy` - If any of `calories`, `protein` or `fat` is missing the recipe should be dropped from the result.

(Finally, goal is to achieve a search that it's faster than $0.1$ seconds on average)
