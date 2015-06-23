---
layout: post
title: Principal Component Analysis on the broadbandmap.gov data
description: "PCA applied to broadbandmap.gov data in the Outreachy - M-lab project"
<!-- modified: 2015-06-23 -->
tags: [outreachy, m-lab, principal component analysis]
image:
  feature: Complete_PCA_Analysis.png
  credit: elf11

comments: true
share: true
---

This article will be one of a few articles done during my internship with the M-Lab organization `link: http://measurementlab.net/` in the Outreachy Summer 2015 program. During this time I will work with different data from broadbandmap.gov `link: http://broadbandmap.gov/` and census.gov `link: http://census.gov/` and probably other sites to try and establish a correlation between the internet broadband connection penetration rate in a community (or how many people have high-speed internet) and the characteristics of that community. Basically, what I want to do is to characterize the communities with internet connection and those without and see why the former are attractive to the Internet Service Providers and the latter are not. (Maybe some socio-economic factors are influencing the availablity of internet connection: like income, education, age, race).

What is Principal Component Analysis (PCA)? A technique that transforms a number of possibly correlated variables into a smaller number of variables called principal components. It is most commonly used as a first stepin trying to analyse large data sets. (Other applications include de-noissing signals and data compression.) 

PCA uses projection (a vector space transformation) to reduce the dimensionality of large data sets. The original data set can be therefore interpreted in just a few variables (the principal components). I am using it on the broadbandmap.gov data set to see if I can spot any trends, patterns and outliers in the data.
