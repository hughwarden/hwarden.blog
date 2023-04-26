---
title: "Classifying Oncogenes Based on PubMED Abstracts"
date: 2023-04-26T00:00:00+01:00
draft: false
description: "Training a classifier to recognise gain and loss of function mutations from abstracts of papers on PubMED."
slug: "mutation-nlp"
tags: ["Machine Learning", "Genetics", "R", "Python"]
series: []
series_order: 1
showAuthor: true
authors:
  - "Hugh Warden"
showAuthorsBadges : true 
---

I am currenlty a PhD student at the [Institute of Genetics and Cancer](https://www.ed.ac.uk/institute-genetics-cancer) in Edinburgh, which is never where I thought I would end up given that when I was 15 I was literally counting down the days until I never had to study biology again. Since then I went to college and university and I studied maths and when it came to choosing what  wanted to write my dissertation on  decided I wanted to do some machine learning and the supervisor I chose was a mathematical biologist. I wrote my dissertation on machine learning, applying it to a scRNA-seq data set and it opened my eyes to the world of genetics and all the cool maths they are using. 

I've now been working in genetics for about two years and I really enjoy it, but when I first started it was very overwhelming to be contantly bombarded with words that I did not know. I've got a lot better but it's still hard sometimes to keep up with presentations. Recently, I was reading a paper and I was finding it hard to keep up with all the biological terms and I wondered "would I be able to train a machine learning algorithm to read biology papers?". This idea stuck with me and I thought it might be a fun idea to try.

If you would like to see the code for this project you can find it here:

{{< github repo="hwarden162/mutation_nlp" >}}

## The Problem

From the outset I would like to make it clear that as well as only recently learning about biology I know next to nothing about machine learning on text, so please don't use this as a guide to learn from! The first thing I had to do was to find a problem to work on. Given my inexperience I thought I'd try and start with something (I thought would be) fairly simple. 

Cancer is very very complex indeed so I have decided to drastically oversimply it for this project. You can essentially think of cancer as when cells start to grow out of control. There are many causes of cancer but a major cause is when the DNA in a particular cell gets mutated in a certain way. DNA in your cells get mutated all the time, but if a certain part of your DNA (which we call a gene) gets mutated in a particular way then this can cause that cells to grow and divide uncontrollably. The beginning of this process is called oncogenesis and the mutations that cause this are called oncogenic mutations, specifically these mutations that cause the cell to grow and divide uncontrollably are called gain of function mutations.

However, your cells are clever and have lots of self regulatory mechanisms that are able to recognise that something is going wrong and cause the cell to essentially self destruct (this is called apoptosis). So a gain of function mutation on its own is not usually enough to cause cancer as your body will recognise this and stop it. Therefore, you need another (oncogenic) mutation to stop the cell from self destructing. These mutations are called loss of function mutations. You can then think of cancer as a cell having both a gain of function mutation and a loss of function mutation. There are many different genes that are gain of function and many that are loss of function and I thought for this project it might be cool to see if we could identify whether a gene is related with gain of function or loss of function mutations from the abstract of a paper.

## Data Collection

The first step of any machine learning project is collecting data. I was originally planning on trying to write my own web scraper to collect the information I wanted, when I started looking into it I found that PubMED has its own API that allows you to search for papers and download the abstracts. This was great as it meant I didn't have to write my own web scraper and I could just use the API. What was even better was that there is a python package called [PyMed](https://package.wiki/pymed) that allows you to easily interact with the PubMED API and is basically the only reason why I was able to do this project.

Having figured out how I would be able to access abstracts I now needed a list of gain of function (GOF) and loss of function (LOF) genes. This, again, was much much more difficult than I had assumed it would be (as is always the way) but I did manage to find such information on [CancerQuest](https://www.cancerquest.org/cancer-biology/cancer-genes) which I used to create lists of GOF and LOF genes that I put into a Python dictionary

``` python
search_genes = {
    "GOF": [
        'ABL1', 'AFF4', 'AKAP13', 'AKT2', 'ALK', 'AML1',
        'AXL', 'BCL-2', 'BCL-3', 'BCL-6', 'BCR', 'CAN',
        'CBFB', 'CCND1', 'CSF1R', 'DEK', 'E2A', 'EGFR', 
        'ERBB2', 'ERG', 'ETS1', 'EWSR1', 'FES', 'FGF3',
        'FGF4', 'FLI1', 'FOS', 'FUS', 'GLI1', 'GNAS', 
        'HER2', 'HRAS', 'IL3', 'JUN', 'K-SAM', 'KIT', 
        'KRAS', 'LCK', 'LMO1', 'LMO2', 'LYL1','MAS1', 
        'MCF2', 'MDM2', 'MLLT11', 'MOS', 'MTG8', 'MYB', 
        'MYC', 'MYCN', 'MYH11', 'NEU', 'NFKB2', 'NOTCH1', 
        'NPM', 'NRAS', 'NRG', 'NTRK1', 'NUP214', 'PAX-5', 
        'PBX1', 'PIM1', 'PML', 'RAF1', 'RARA', 'REL', 
        'RET', 'RHOM1', 'RHOM2', 'ROS1', 'RUNX1', 'SET', 
        'SIS', 'SKI', 'SRC', 'TAL1', 'TAL2', 'TCF3', 
        'TIAM1', 'TLX1', 'TSC2'
    ],
    "LOF": [
        'APC', 'BRCA1', 'BRCA2', 'CDKN2A', 'DCC', 
        'DPC4', 'MADR2', 'MEN1', 'NF1', 'NF2', 'PTEN', 
        'RB1', 'TP53', 'VHL', 'WRN', 'WT1'
    ]
}
```

Now we have to iterate over these lists to generate a list of abstracts for each gene. I found a great [StackOverflow article](https://stackoverflow.com/questions/57053378/query-pubmed-with-python-how-to-get-all-article-details-from-query-to-pandas-d) on how to do this and output the results as a pandas data frame. I then saved this data frame as a tsv file to be processed later.

``` python
from pymed import PubMed
import pandas as pd

pubmed = PubMed(tool="GeneSearcher", email="hugh.warden@outlook.com")

data = pd.DataFrame(columns=['pubmed_id','title','abstract','gene','gene_type','keywords','journal','conclusions','methods','results','copyrights','doi','publication_date','authors'])

for gene_type in search_genes.keys():
    for gene in search_genes[gene_type]:
        results = pubmed.query(gene, max_results=500)
        articleList = []
        articleInfo = []
        for article in results:
            articleDict = article.toDict()
            articleList.append(articleDict)
        for article in articleList:
            pubmedId = article['pubmed_id'].partition('\n')[0]
            articleInfo.append({u'pubmed_id':pubmedId,
                            u'title':article['title'],
                            u'abstract':article['abstract'],
                            u'gene':gene,
                            u'gene_type':gene_type,
                            #u'keywords':article['keywords'],
                            #u'journal':article['journal'],
                            #u'conclusions':article['conclusions'],
                            #u'methods':article['methods'],
                            #u'results': article['results'],
                            #u'copyrights':article['copyrights'],
                            #u'doi':article['doi'],
                            #u'publication_date':article['publication_date'], 
                            #'authors':article['authors']
                        })
        appendPD = pd.DataFrame.from_dict(articleInfo)
        data = data.append(appendPD)
        
data.to_csv(
    "data/01_data_collection/raw_data.csv", 
    sep="\t",
    index=False
)
```

You can see here that there are many other bits of information I could have collected from the API, but I decided to just collect the abstracts as I thought this would be enough to train a model, it also would require some more careful handling of the request in the cases of missing data. I also limited the number of results to 500 as I didn't want to be collecting data for too long. In an ideal world I would have collected all of the data and used it to perform meta-analysis of my work, investigating whether there are other factors affecting the model. For example, if a particular author writes about one gene and has a particular writing style then the model will associate that author's writing style with the gene and not the content of the article. I didn't have time to do this but it would have been interesting to see.

If you would like to follow along with the rest of this project without performing this step yourself, you can download the data set that I created here.

## Data Pre-Processing

Although we have collected our daata we still need to do some pre-processing before we can start training our model. The first thing we need to do is remove any duplicate abstracts. An article may appear more than once in our data if it was returned in the search results for more than one gene. However, there are two