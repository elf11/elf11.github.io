---
layout: post
title: Analysing internet speeds trends using k-means
description: "k-means analysis done for different data sets to highlight internet speeds/availability in USA (the Outreachy - M-lab project)"
<!-- modified: 2015-08-18 -->
tags: [outreachy, m-lab, k-means]
image:
  feature: MedianRTT85.png
  credit: elf11

comments: true
share: true
---

This article is the second one in the M-lab Outreachy series and it describes the process we followed in obtaining a cluster visualization and the following analysis of the internet speeds and characteristics for USA using K-means. If you remember from the previous article the purpose of this project is to be able to state some facts about the internet connection speed/availability in different parts of the United States and how those speeds correlate with socio-economic factors. The next step we decided on was to do a cluster analysis and see if any communities are clustering together based on their internet speed (upload and download speed) and RTT (round time trip) and then observe if any of those cluster share some socio-economic traits.

# Method

The analytic process began with identifying a data set to use. We had the option to use the same data set used for the Principal Component Analysis study `link: http://broadbandmap.gov/` (after we cleaned it up a little bit), or we could use the M-lab data collected using Piecewise `link: https://github.com/opentechinstitute/piecewise`. We decided to use both of those data sets, with more interest in the M-lab data set since it is more accurate than the broadbandmap.gov one, but we should use the broadbandmap data set for census information and as a verify against kind of data set.

Another step in the analytic process was deciding what was the best way to select the regions for analysis. Considering we are using the data at the county level and that United States has a relatively big population and a lot of census data, to analyse the whole country at once was not a real option. So we decided to continue with the New England region analysis. But after that it came the question "how do we check that the model is right?". If we observe some trends in the New England region how do we know they are not just a fluke and they can be found in other regions too. We had two options: analyse another entire region of the USA or try and find similar counties with those that we initially observed that sport special traits. Both of those approaches have drawbacks, but we went with analysing another entire region of the USA, specifically the South Atlantic region `link: https://en.wikipedia.org/wiki/South_Atlantic_States` because it will hopefully not introduce a the analyser's bias, and it makes sense not to hand pick counties all over the USA to analyse and try to find the similarities with New England counties. Another reason why we chose the South Atlantic region was that it was most similar with the New England region, it wouldn't make sense to select a Midwestern region for example, mostly because we might introduce a higher cultural bias. Of course this might not have been the best option available and there might be other biases introduced.

Having selected a set of counties to analyse, we then went and clean up the broadband data set, keeping only some of the more important features from it, and combining some other features in a single one. We also collected the data from M-lab and bigQuery using Piecewise. We fitted a normal curve to the data to see if it is normally distributed or not, most of the broadband data was not normally distributed, this might be due the fact that the entire data set is not really accurate or it might be due to inconsistencies regarding the internet availability for different counties. Next, we used the previously obtained Principal Component Analysis results in a preliminary k-means analysis, we used the lower dimensionality data obtained to represent the m-lab and broadband data sets in 2D graphs and we dermined the optimum k for each data set. We then cluster all the counties from a region in clusters and visualized those clusters demographics properties as well as internet speed and rtt properties. Regarding the demographics we paid special attention to the most signifiant variables that PCA suggested added the most variance to the data: population (we actually used the population density - calculated by divinding the population to the land area), median income and households. Another variable that was interesting in our analysis was the percentage of the population with an income greater than $100k.

# New England region - Results, Code and Discussion

## k-means clustering - how it works

k-means is a method to analyse data sets using unsupervised learning `link: https://en.wikipedia.org/wiki/Unsupervised_learning`. For example we have two data sets - the broadbandmap.gov data set and the M-lab data sets with some observed examples and each of those examples has a set of features, but has no labels. On such a data set we can find groups of data which are similar to one another - those are the clusters. 

Formally, k-means is a method of vector quantization `link: https://en.wikipedia.org/wiki/Unsupervised_learning`, it partition the data space into Voronoi cells and separates the samples into n groups of equal variance. To separate the space it uses the Euclidian distance as metric. So for example we have the previously used broadbandmap.gov data set and we observed by using PCA (Principal Component Analysis) that the features that add the most variance to the data set are: population, median income and households. k-means separates the initial data space into cluster where the variance of those features is not as great, the variance inside of one of those clusters should be minimal.

k-means clustering uses iterative refinement in 3 basic steps: 
* Choose k
* Iterate over
** assignment
** update

The steps are repeated until convergence has been reached. We choose to use k-means mostly because it has some obvious advantages, like it can be used for exploratory analysis easily (what we are doing), scales well, it is efficient and will always converge.

### Intuition

Figure 1 shows k-means with a 2 dimensional feature vector, each point has two dimensions and x and a y. In our work we used a data set with a lot more features, in fact the broadbandmap.gov data set still has around 50 features after we cleaned it up. So we can only visualize the data in 2 or 3 dimensions, but to get a better intuition the figure uses a 2 dimensional vector that is clustered, the training examples are shown as dots and the centroids are shown as crosses. (a) is the original data set, (b) represents the random initial cluster centroids and images (c-f) illustrates two running of the k-means algorithm. In each iteration, each training example is assigned to the closest cluster centroid, then the cluster centroid is moved to the mean of the points assigned to it. This is the method we used in our analysis too.
<figure>
	<a href="http://stanford.edu/~cpiech/cs221/img/kmeansViz.png"><img src="http://stanford.edu/~cpiech/cs221/img/kmeansViz.png" alt=""></a>
	<figcaption><a href="http://stanford.edu/~cpiech/cs221/img/kmeansViz.png" title="Figure 1">Figure 1</a>.</figcaption>
</figure>
