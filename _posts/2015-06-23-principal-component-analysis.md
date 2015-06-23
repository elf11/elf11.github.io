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

This article will be one of a few articles done during my internship with the M-Lab organization link: http://measurementlab.net/ in the Outreachy Summer 2015 program. During this time I will work with different data from broadbandmap.gov link: http://broadbandmap.gov/ and census.gov `link: http://census.gov/` and probably other sites to try and establish a correlation between the internet broadband connection penetration rate in a community (or how many people have high-speed internet) and the characteristics of that community. Basically, what I want to do is to characterize the communities with internet connection and those without and see why the former are attractive to the Internet Service Providers and the latter are not. (Maybe some socio-economic factors are influencing the availablity of internet connection: like income, education, age, race).

What is Principal Component Analysis (PCA)? A technique that transforms a number of possibly correlated variables into a smaller number of variables called principal components. It is most commonly used as a first stepin trying to analyse large data sets. (Other applications include de-noissing signals and data compression.) 

PCA uses projection (a vector space transformation) to reduce the dimensionality of large data sets. The original data set can be therefore interpreted in just a few variables (the principal components). I am using it on the broadbandmap.gov data set to see if I can spot any trends, patterns and outliers in the data.

We are going to examine 3 data sets, all of them collected from broadbandmap.gov site. The data sets offer information about broad-band internet connection (the first data set) in the counties from the New England Region states (there are 67 counties in the 6 New England States - Connecticut, Maine, Massachusetts, New Hampshire, Rhode Island, and Vermont.), the demographics data for the same counties, and the third data set will be a combination of the first data sets, with information for each county that will comprise of broadband internet and demographics (basically we join the two data sets on the county id).

The broadband internet connection data set has around 118 dimensions, for obvious reason it would be impossible to see any trends in this data set. The data set can be seen `link: https://github.com/elf11/Outreachy-Mlab/blob/master/code/broad_sum.csv`, it has information about upload and download speed, wireless and wireline internet connection (and specific speeds for those as well as the percentage of population that has such a connection).

To make sense of the data and see if there are any trends that are not obvious by looking at the data we could use a series of bivariate plots (scatter diagrams) and analyse these to determine any relationship between variables. But, typically the number of such plots is O(n^2), where n is the number of variables. Clearly, this is not feasible. But, we can use PCA to perform such an analysis simultaneously.

In the broadband example we have 118 dimensional data (dimensions) for 67 counties (observations). If there is any correlation between the observations (the counties) it can be observed in the 118 dimensional space by the correlated points being clustered close together (but we are not able to visualise such a space, so we are not able to see the clustering directly). 

First task of PCA is to identify a new set of orthogonal coordinate axes through the data. This is achieved by finding the direction of maximal variance through the coordinates in the 118 dimensional space. It is equivalent to obtaining the (least-squares) line of best fit through the plotted data. We call this new axis the first principal component of the data. After this we can orthogonal projection to map the coordinates on this axis. This is the first principal component.

<div style="align: center;"><img src="/images/Broadband_PC1_Analysis.png" alt="Figure1"></div>

This type of diagram is known as a score plot. We can see already some clusters forming, in the sense that counties on the far right are forming a cluster (insert_counties_ids), then there are the central group of counties as a bulk, then another 4 counties that represent interest (insert_counties_ids), two smaller clusters, and another 2 counties that stand out (insert_counties_ids) at the opposite end of the axis. 

From the PCA we add another axis - the second principal component, which is orthogonal to the first PC, and is the next best direction for approximating the original data (finds the direction of second largest variance in the data). We project our coordinates down onto this plane and we find the second figure.

<div style="align: center;"><img src="/images/Broadband_PCA_Analysis.png" alt="Figure1"></div>


