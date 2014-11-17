# Candidate Classifier

## Overview

This system collects and classifies candidate social media sites.

It is divided into two main steps:

1. Gathering potential sites using the Google Custom Search API and candidate information 

2. Classifying the results as candidate web presence or not

### Setup
Install the requirememts file. 

The scripts rely on access to the BIP Production Dropbox folder. 
That folder holds the flat files which contains all of the candidate data by state.
There will also be an all_states file, used mostly for splitting files at the end. This is simply all the candidate data combined into one file. 

You'll need to touch the following files in each of the social media directories for the scripts to run happily:
[ test this and include]


### Step 1) Using the Google Search API

The get_X_links.py trio of scripts all serve similar purposes. They import a csv of candidate information (name, electoral district, known web presence, if it exists, etc) and hit the API with couple of variations on search terms. The results of the search are cleaned and combined. If there is a known url already in the flat file, the script will attempt to find the relationship of the result to the known url (Parent, Child, Equal, or None). The results are written to a variety of csvs, with each result linked to the candidates unique identifier (UID). 

The file that really matters is fullXsearch_results_combined.csv, where X is fb, twitter, or web.

There is a wonky thing here, where if there is not a known url in the flat file, you must fill in the empty column with 'gernensamples.com'. Note: You don't have to do this with Twitter links, because twitter.com/ is prepended to all the columns, and things will run. 

There are a limited number of API hits per day, so note that you will have to run these scripts over the course of days. With our key, you can do approximately one platform per day for a full candidate data set. Plan ahead! 
Hits reset at 12:00 am PT.


### Step 2) Classifying Search Results

The classify_X_combined.py scripts actually take the results and classify the search results as candidate campaign sites or not using page text. The set of candidates that have known urls forms the training and test set. Since this relys on page text, note that Twitter classification will be especially unreliable. 

Inputs will be the fullXseach__results_combined.csv file and your trainig and test files. However, before running, split your file into ~20 smaller files using splitfile.py. For now, the number of files the script expects is hardcoded in, so make sure that number matches. 

When you run this with the --full argument, you will be running it on live data. Without --full, it will only run on training/test data.

Results will go to X_classifier_outs.py in Dropbox.

## Methodology 

(designed and described by https://github.com/natgaertner/)

The classifiers used were from the scikit learn python library (http://scikit-learn.org/0.11/index.html). Their usage was  based on example code here: http://scikit-learn.org/0.11/auto_examples/document_classification_20newsgroups.html . Two significant modifications were made to this code, other than obviously pulling data from this search text collection rather than their sample database:

A custom feature identifier was used that could identify some custom multi-word features and some features that would be specific to each entry (for example if the candidate's name appeared in the text, if the candidate's office name appeared in the text.) The scikit learn default was simply using single word features. It has an n-gram analyzer as well, but it wasn't immediately clear where that could be used for multi-word features.

The custom features were added to the single word default analyzer, so all single word features (minus stop words) were included as well. The addition of custom feature detection was probably the most significant improvement achieved over just running the classifiers as-is.

Second, the results of the classifiers were combined into a bayesian probability estimate for the class of the page.The input was used as both the discrete classification returned by each classifier and the score returned for each class by each classifier. This almost certainly led to some inaccuracy in the conditional independence assumptions for input variables, but it did give a really nice (artificially) sharp logistic shaped function for making class choices.

The Bayesian formula used was:
P(class | classifications, score) = P(class)*prod(P(classification|class) for each classifier)*P(score|class)/sum(P(class)*prod(P(classification|class) for each classifier)*P(score|class) for each class)


There's a lot of room for cleaning and refactoring, but getting it running was prioritzed over elegance. Code is definitely a little quirky. Maybe one day I can fix it.