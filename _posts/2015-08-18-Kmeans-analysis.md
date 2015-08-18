---
layout: post
title: Analysing internet speeds trends using k-means
description: "k-means analysis done for different data sets to highlight internet speeds/availability in USA (the Outreachy - m-lab project)"
<!-- modified: 2015-08-18 -->
tags: [outreachy, m-lab, k-means]
image:
  feature: SA_BBand_1kmeans.png
  credit: elf11

comments: true
share: true
---

This article is the second one in the m-lab Outreachy series and it describes the process we followed in obtaining a cluster visualization and the following analysis of the internet speeds and characteristics for USA using K-means. If you remember from the previous article the purpose of this project is to be able to state some facts about the internet connection speed/availability in different parts of the United States and how those speeds correlate with socio-economic factors. The next step we decided on was to do a cluster analysis and see if any communities are clustering together based on their internet speed (upload and download speed) and RTT (round time trip) and then observe if any of those cluster share some socio-economic traits.

# Method

The analytic process began with identifying a data set to use. We had the option to use the same data set used for the Principal Component Analysis study `link: http://broadbandmap.gov/` (after we cleaned it up a little bit), or we could use the m-lab data collected using Piecewise `link: https://github.com/opentechinstitute/piecewise`. We decided to use both of those data sets, with more interest in the m-lab data set since it is more accurate than the broadbandmap.gov one, but we should use the broadbandmap data set for census information and as a verify against kind of data set.

Another step in the analytic process was deciding what was the best way to select the regions for analysis. Considering we are using the data at the county level and that United States has a relatively big population and a lot of census data, to analyse the whole country at once was not a real option. So we decided to continue with the New England region analysis. But after that it came the question "how do we check that the model is right?". If we observe some trends in the New England region how do we know they are not just a fluke and they can be found in other regions too. We had two options: analyse another entire region of the USA or try and find similar counties with those that we initially observed that sport special traits. Both of those approaches have drawbacks, but we went with analysing another entire region of the USA, specifically the South Atlantic region `link: https://en.wikipedia.org/wiki/South_Atlantic_States` because it will hopefully not introduce a the analyser's bias, and it makes sense not to hand pick counties all over the USA to analyse and try to find the similarities with New England counties. Another reason why we chose the South Atlantic region was that it was most similar with the New England region, it wouldn't make sense to select a Midwestern region for example, mostly because we might introduce a higher cultural bias. Of course this might not have been the best option available and there might be other biases introduced.

Having selected a set of counties to analyse, we then went and clean up the broadband data set, keeping only some of the more important features from it, and combining some other features in a single one. We also collected the data from m-lab and bigQuery using Piecewise. We fitted a normal curve to the data to see if it is normally distributed or not, most of the broadband data was not normally distributed, this might be due the fact that the entire data set is not really accurate or it might be due to inconsistencies regarding the internet availability for different counties. Next, we used the previously obtained Principal Component Analysis results in a preliminary k-means analysis, we used the lower dimensionality data obtained to represent the m-lab and broadband data sets in 2D graphs and we dermined the optimum k for each data set. We then cluster all the counties from a region in clusters and visualized those clusters demographics properties as well as internet speed and rtt properties. Regarding the demographics we paid special attention to the most signifiant variables that PCA suggested added the most variance to the data: population (we actually used the population density - calculated by divinding the population to the land area), median income and households. Another variable that was interesting in our analysis was the percentage of the population with an income greater than $100k.

# New England region - Results, Code and Discussion

## k-means clustering - how it works

k-means is a method to analyse data sets using unsupervised learning `link: https://en.wikipedia.org/wiki/Unsupervised_learning`. For example we have two data sets - the broadbandmap.gov data set and the m-lab data sets with some observed examples and each of those examples has a set of features, but has no labels. On such a data set we can find groups of data which are similar to one another - those are the clusters. 

Formally, k-means is a method of vector quantization `link: https://en.wikipedia.org/wiki/Unsupervised_learning`, it partition the data space into Voronoi cells and separates the samples into n groups of equal variance. To separate the space it uses the Euclidian distance as metric. So for example we have the previously used broadbandmap.gov data set and we observed by using PCA (Principal Component Analysis) that the features that add the most variance to the data set are: population, median income and households. k-means separates the initial data space into cluster where the variance of those features is not as great, the variance inside of one of those clusters should be minimal.

k-means clustering uses iterative refinement in 3 basic steps: 
* Choose k
* Iterate over: (a) assignment (b) update

The steps are repeated until convergence has been reached. We choose to use k-means mostly because it has some obvious advantages, like it can be used for exploratory analysis easily (what we are doing), scales well, it is efficient and will always converge.

### Intuition

Figure 1 shows k-means with a 2 dimensional feature vector, each point has two dimensions and x and a y. In our work we used a data set with a lot more features, in fact the broadbandmap.gov data set still has around 50 features after we cleaned it up. So we can only visualize the data in 2 or 3 dimensions, but to get a better intuition the figure uses a 2 dimensional vector that is clustered, the training examples are shown as dots and the centroids are shown as crosses. (a) is the original data set, (b) represents the random initial cluster centroids and images (c-f) illustrates two running of the k-means algorithm. In each iteration, each training example is assigned to the closest cluster centroid, then the cluster centroid is moved to the mean of the points assigned to it. This is the method we used in our analysis too.
<figure>
	<a href="http://stanford.edu/~cpiech/cs221/img/kmeansViz.png"><img src="http://stanford.edu/~cpiech/cs221/img/kmeansViz.png" alt=""></a>
	<figcaption><a href="http://stanford.edu/~cpiech/cs221/img/kmeansViz.png" title="Figure 1, credit stanford.edu">Figure 1, credit stanford.edu</a>.</figcaption>
</figure>

## Method explained

As said before, we are going to compare the results obtained by the analysing the m-lab data set with the broadbandmap.gov data set and see if the two show the same trends for the New England region, and after that we are going to analyse the South Atlantic region and see if we can extrapolate the study. 

For the broadbandmap.gov data we initially tried to see if the data is normally distributed. We calculated the mean of the data for all the individual features we decided to keep and after that we fit a normal function to the data and see how the two actually differ. Results for all the 50 features we decided to keep for this data set can be found in the folder `link: https://github.com/elf11/Outreachy-Mlab/tree/master/k-means/normal_test`. The before normalization files can be interpreted like this: the y axis unit is number of samples within the bin intervals in the x axis, and the after normalization files like this: the y axis unit is frequency of the bin values as a percentage over all the samples. An example of the kind of graphs we obtained for the broadbandmap.gov data can be seen in Figure 2, where we see the feature of "NumberofWirelineProviderEqual2" being fit to a normal curve. 

<figure style="align: center;">
	<a href="/images/afternormalization_numberOfWirelineProvidersEquals2.png"><img src="/images/afternormalization_numberOfWirelineProvidersEquals2.png" alt=""></a>
	<figcaption><a href="/images/afternormalization_numberOfWirelineProvidersEquals2.png" title="Figure 2, Number of Wireline Provider Equal 2, for New England region">Figure 2, Number of Wireline Provider Equal 2, for New England region</a>.</figcaption>
</figure>

{% highlight python %}
def normal_func(broad_df, names, ending):
    # trying out a normality test, see how the data is distributed in the
    # broadbandmap.gov data set. Actually, I am fitting a normal curve to the data
    # and see how it differs (from the graphs is pretty clear that the data is not normaly distributed)
    for name in names:
        array = broad_df[name].values
        print name,' normality=',st.normaltest(array)
        fig_name = 'normal_test/' + ending + '_afternormalization_' + name  +'.png'
       
        fig, ax = plt.subplots()

        # The required parameters
        num_steps = 10
        max_percentage = 0.1
        num_bins = 40
        
        # Calculating the maximum value on the y axis and the yticks
        max_val = max_percentage * len(array)
        step_size = max_val / num_steps
        yticks = [ x * step_size for x in range(0, num_steps+1) ]
        ax.set_yticks( yticks )
        plt.ylim(0, max_val)
        
        # Running the histogram method
        n, bins, patches = plt.hist(array, num_bins)
        mu = np.mean(array)
        sigma = np.std(array)
        plt.plot(bins, mlab.normpdf(bins, mu, sigma))
        
        # Before normalisation: the y axis unit is number of samples within the bin intervals in the x axis        
        # After normalisation: the y axis unit is frequency of the bin values as a percentage over all the samples
        # To plot correct percentages in the y axis     
        to_percentage = lambda y, pos: str(round( ( y / float(len(array)) ) * 100.0, 2)) + '%'
        plt.gca().yaxis.set_major_formatter(FuncFormatter(to_percentage))
        plt.title('Normal curve fit to the data');
        plt.savefig(fig_name, dpi=125)
        plt.show()
        plt.close()
{% endhighlight %}

After that we had to decide on the best size for k (the number of clusters), so we run k-means over the data sets using scikit-learn in Python. Since intuition fails in high dimensions to choose the k we used the results from PCA and applied k-means over those results in order to obtain a 2-D visualisation of the clusters and to obtain a curve that describes the percentage of variance explained by each value of k between 1 and 14. As it can be seen in the code below, we read the values from the data frame, then we use a train_test_split function to split the size of our data set in two, we used 30% of the data for training, and the rest for testing (in practice the training set is somewhere between 20-40 %). After that we are applying PCA on the training set (we decided previously that the dimensionality of the data can be reduce to two without losing too much of the data variance), and on the data resulted from the PCA we apply k-means and plot the clusters in 2D.

To determine the best value for k, we fit the k-means model for each k in [1,14], calculated the Euclidean distance from each point to each centroid (cluster center), calculated the within cluster sum of squares (the distance between all the points in a cluster and the centroid), then the total sum of squares and the between clusters sum of squares. The variance explained by each k is function of the between clusters sum of squares.

{% highlight python %}
def kmeans_func(broad_df, K, fileName):
    pc_toarray = broad_df.values
    hpc_fit, hpc_fit1 = train_test_split(pc_toarray, train_size=.3)
    print broad_df.head()
    
    hpc = PCA(n_components=2).fit_transform(hpc_fit)
    print hpc
    k_means = KMeans(n_clusters=K)
    k_means.fit(hpc)
    
    x_min, x_max = hpc[:, 0].min() - 8, hpc[:, 0].max() + 8
    y_min, y_max = hpc[:, 1].min() - 5, hpc[:, 1].max() + 5
    print x_min, x_max
    print y_min, y_max
    xx, yy = np.meshgrid(np.arange(x_min, x_max, .02), np.arange(y_min, y_max, .02))
    
    Z = k_means.predict(np.c_[xx.ravel(), yy.ravel()])
    Z = Z.reshape(xx.shape)
    
    plt.figure(1)
    plt.clf()
    plt.imshow(Z, interpolation='nearest',
              extent=(xx.min(), xx.max(), yy.min(), yy.max()),
              cmap=plt.cm.Paired,
              aspect='auto', origin='lower')
    
    plt.plot(hpc[:, 0], hpc[:, 1], 'k.', markersize=4)
    centroids = k_means.cluster_centers_
    inert = k_means.inertia_
    plt.scatter(centroids[:, 0], centroids[:, 1],
               marker='x', s=169, linewidths=3,
               color='b', zorder=8)
    plt.xlim(x_min, x_max)
    plt.ylim(y_min, y_max)
    plt.xticks(())
    plt.yticks(())
    fileName = fileName + 'kmeans.png'
    plt.savefig(fileName, dpi=125)
    
    
    
    # Determine your k range
    k_range = range(1,14)
    
    # Fit the kmeans model for each n_clusters = k
    k_means_var = [KMeans(n_clusters=k).fit(hpc) for k in k_range]
    
    # Pull out the cluster centers for each model
    centroids = [X.cluster_centers_ for X in k_means_var]
    
    # Calculate the Euclidean distance from 
    # each point to each cluster center
    k_euclid = [cdist(hpc, cent, 'euclidean') for cent in centroids]
    dist = [np.min(ke,axis=1) for ke in k_euclid]
    
    # Total within-cluster sum of squares
    wcss = [sum(d**2) for d in dist]
    
    # The total sum of squares
    tss = sum(pdist(hpc)**2)/hpc.shape[0]
    
    # The between-cluster sum of squares
    bss = tss - wcss
    
    # elbow curve
    fig = plt.figure()
    ax = fig.add_subplot(111)
    ax.plot(k_range, bss/tss*100, 'b*-')
    ax.set_ylim((0,100))
    plt.grid(True)
    plt.xlabel('n_clusters')
    plt.ylabel('Percentage of variance explained')
    plt.title('Variance Explained vs. k')
    fileName = fileName + 'variance_explained.png'
    plt.savefig(fileName, dpi=125)
{% endhighlight %}

The plotted clusters in 2D and the variance plots can be observed in Figure3. Images (a-b) are representative for broadbandmap.gov data and images (c-d) are representative for m-lab data set. As it can be seen in the variance graphs, most of the variance can be explained with k=6 for both sets of data, so next we are going to use that value for assigning each county in the New England region to a cluster.

<figure class="half">
	<a href="/images/BBand_83kmeans.png"><img src="/images/BBand_83kmeans.png" alt=""></a>
	<a href="/images/BBand_83kmeans.pngvariance_explained.png"><img src="/images/BBand_83kmeans.pngvariance_explained.png" alt=""></a>
	<a href="/images/Combined_83kmeans.png"><img src="/images/Combined_83kmeans.png" alt=""></a>
	<a href="/images/Combined_83kmeans.pngvariance_explained.png"><img src="/images/Combined_83kmeans.pngvariance_explained.png" alt=""></a>
	<figcaption style="align: center;">Figure 3. (a) 2D k-means clustering for broadbandmap.gov data (b) Variance explained for broadbandmap.gov data (c) 2D k-means clustering for m-lab data (d) Variance explained for m-lab data (from left to right, top to bottom)</figcaption>
</figure>

We assigned each of the counties in New England to a cluster for each of the data sets. After that we used the clusters to plot the demographics for the counties in each of those clusters and see if there are any obvious trends. Results for the broadbandmap.gov site can be observed below. The code for assigning each of the counties to a cluster is below:

{% highlight python %}
def clustering(broad_df, target_names, K, fileName):
    #initialize and carry out clustering
    km = KMeans(n_clusters = K)
    print 'clustering ', broad_df
    km.fit(broad_df)
    
    print 'HERE'
    
    #find center of clusters
    centers = km.cluster_centers_
    print 'len(centers[0]) ', len(centers[0])
    centers[centers<0] = 0 #the minimization function may find very small negative numbers, we threshold them to 0
    centers = centers.round(2)
    centers_num = len(centers[0])
    file1 = fileName + 'centers.txt'
    f = open(file1, 'w')
    f.write('\n--------Centers of the four different clusters--------\n')
    i = 0
    county_str = 'County\t' 
    while i < K:
        j = i+1
        val = ' Cent' + str(j)
        county_str = county_str + val
        i = i + 1
    county_str = county_str + '\n'
    f.write(county_str)
    for i in range(centers_num):
        j = 0
        line = '' + target_names[i]
        while j < K:
            line = line + '\t' + str(centers[j,i])
            j = j + 1
        line = line + '\n'
        f.write(line)
    f.close()
    #find which cluster each county is in
    prediction = km.predict(broad_df)
    file2 = fileName + 'prediction.txt'
    f = open(file2, 'w')
    #f.write('--------Which cluster each county is in--------\n')
    f.write('{:<5},{}'.format('County','Cluster\n'))
    print 'len(prediction) ',len(prediction)
    for i in range(len(prediction)):
        f.write('{:<5},{}'.format(target_names[i],prediction[i]+1))
        f.write('\n')
    f.close()
{% endhighlight %}

<figure class="third">
	<a href="/images/BBand_83_cluster_group_0.png"><img src="/images/BBand_83_cluster_group_0.png" alt=""></a>
	<a href="/images/BBand_83_cluster_group_1.png"><img src="/images/BBand_83_cluster_group_1.png" alt=""></a>
	<a href="/images/BBand_83_cluster_group_2.png"><img src="/images/BBand_83_cluster_group_2.png" alt=""></a>
</figure>
<figure class="third">
	<a href="/images/BBand_83_cluster_group_3.png"><img src="/images/BBand_83_cluster_group_3.png" alt=""></a>
	<a href="/images/BBand_83_cluster_group_4.png"><img src="/images/BBand_83_cluster_group_4.png" alt=""></a>
	<a href="/images/BBand_83_cluster_group_5.png"><img src="/images/BBand_83_cluster_group_5.png" alt=""></a>
	<figcaption style="align: center;">Figure 4. Broadbandmap.gov data clustered and demographics characteristics plotted</figcaption>
</figure>

As it can be seen in the images the data was clustered by the internet specifications (internet speed, number of providers, type of connection etc.), but the clusters also present some particular demographics characteristics. We can observe that the clusters formed have in common mostly 3 characteristis: population, households and median income. This might mean that the separation of the internet services might be done by rural-urban counties. Rurals counties are served differently than urban counties by ISPs. To highlight this difference more we can look at the following 3 graphs (Figure 5), that create a median value over each cluster for the 3 variables.

<figure class="third">
	<a href="/images/Median_Population_bb_83.png"><img src="/images/Median_Population_bb_83.png" alt=""></a>
	<a href="/images/Median_Households_bb_83.png"><img src="/images/Median_Households_bb_83.png" alt=""></a>
	<a href="/images/Median_MedianIncome_bb_83.png"><img src="/images/Median_MedianIncome_bb_83.png" alt=""></a>
	<figcaption style="align: center;">Figure 5. Broadbandmap.gov data mean demographics characteristics for each cluster</figcaption>
</figure>

To test if the hypothesis that rural and urban counties are served differently by ISPs we ran the same test on the m-lab data set.

<figure class="third">
	<a href="/images/Combined_83_cluster_group_0.png"><img src="/images/Combined_83_cluster_group_0.png" alt=""></a>
	<a href="/images/Combined_83_cluster_group_1.png"><img src="/images/Combined_83_cluster_group_1.png" alt=""></a>
	<a href="/images/Combined_83_cluster_group_2.png"><img src="/images/Combined_83_cluster_group_2.png" alt=""></a>
</figure>
<figure class="third">
	<a href="/images/Combined_83_cluster_group_3.png"><img src="/images/Combined_83_cluster_group_3.png" alt=""></a>
	<a href="/images/Combined_83_cluster_group_4.png"><img src="/images/Combined_83_cluster_group_4.png" alt=""></a>
	<a href="/images/Combined_83_cluster_group_5.png"><img src="/images/Combined_83_cluster_group_5.png" alt=""></a>
	<figcaption style="align: center;">Figure 6. m-lab data clustered and demographics characteristics plotted</figcaption>
</figure>

As it can be observed the same separation by the population, households and median income is visible with the m-lab data set too. (Note: just the internet data is from m-lab, the demographics data is still the one available on the broadbandmap.gov site that is provided by census.gov)

In Figure 7 we can observe the same graphic as Figure 5 just with clustering done with m-lab data. We also used the mean value for each cluster in this situation.

<figure class="third">
	<a href="/images/Median_Population_comb_83.png"><img src="/images/Median_Population_comb_83.png" alt=""></a>
	<a href="/images/Median_Households_comb_83.png"><img src="/images/Median_Households_comb_83.png" alt=""></a>
	<a href="/images/Median_MedianIncome_comb_83.png"><img src="/images/Median_MedianIncome_comb_83.png" alt=""></a>
	<figcaption style="align: center;">Figure 7. m-lab data mean demographics characteristics for each cluster</figcaption>
</figure>

To further highlight the difference in service between the rural and urban communities, as well as between communities with a higher median income and those with a lower median income, we plotted the mean values for RTT, upload and download (in Mb/s) for each of those clusters.

<figure class="third">
	<a href="/images/Average_Median83.png"><img src="/images/Average_Median83.png" alt=""></a>
	<a href="/images/Average_Upload83.png"><img src="/images/Average_Upload83.png" alt=""></a>
	<a href="/images/Average_Download83.png"><img src="/images/Average_Download83.png" alt=""></a>
	<figcaption style="align: center;">Figure 8. m-lab data mean values for RTT, upload and download</figcaption>
</figure>

It can be observed how the values for the clusters with a high population and number of households (Cluster 2, 3, 6) have a better upload and download speed and a lower RTT.

The code for reproducing the graphs and collecting data from Piecewise can be found `link: https://github.com/elf11/Outreachy-Mlab/tree/master/k-means/code`.

# South Atlantic region - Results, Code and Discussion

We have extended the analysis on data from the South Atlantic region. We applied the same method in analysing this data set as previously explained and following are the graphs for the clusters obtained.

<figure class="half">
	<a href="/images/SA_BBand_1kmeans.png"><img src="/images/SA_BBand_1kmeans.png" alt=""></a>
	<a href="/images/SA_BBand_1kmeans.pngvariance_explained.png"><img src="/images/SA_BBand_1kmeans.pngvariance_explained.png" alt=""></a>
	<a href="/images/SA_Combined_1kmeans.png"><img src="/images/SA_Combined_1kmeans.png" alt=""></a>
	<a href="/images/SA_Combined_1kmeans.pngvariance_explained.png"><img src="/images/SA_Combined_1kmeans.pngvariance_explained.png" alt=""></a>
	<figcaption style="align: center;">Figure 9. (a) 2D k-means clustering for broadbandmap.gov data (b) Variance explained for broadbandmap.gov data (c) 2D k-means clustering for m-lab data (d) Variance explained for m-lab data</figcaption>
</figure>

We can observe that this time the number of clusters is 9, so k=9 to be able to cover all the variance in the data with k-means. We are going to further present the demographics data just for the clusters formed by applying k-means on the m-lab data set, since that is the most relevant one. This data set is a lot bigger than the previous one, the New England data set, due to the fact that there are ~500 counties in the South Atlantic region, compared to only 67 in New England. Due to this fact the reading of the graphs might be a little hard.

<figure class="third">
	<a href="/images/Density_Combined_1_cluster_group_0.png"><img src="/images/Density_Combined_1_cluster_group_0.png" alt=""></a>
	<a href="/images/Density_Combined_1_cluster_group_1.png"><img src="/images/Density_Combined_1_cluster_group_1.png" alt=""></a>
	<a href="/images/Density_Combined_1_cluster_group_2.png"><img src="/images/Density_Combined_1_cluster_group_2.png" alt=""></a>
</figure>
<figure class="third">
	<a href="/images/Density_Combined_1_cluster_group_3.png"><img src="/images/Density_Combined_1_cluster_group_3.png" alt=""></a>
	<a href="/images/Density_Combined_1_cluster_group_4.png"><img src="/images/Density_Combined_1_cluster_group_4.png" alt=""></a>
	<a href="/images/Density_Combined_1_cluster_group_5.png"><img src="/images/Density_Combined_1_cluster_group_5.png" alt=""></a>
<figure class="third">
	<a href="/images/Density_Combined_1_cluster_group_6.png"><img src="/images/Density_Combined_1_cluster_group_6.png" alt=""></a>
	<a href="/images/Density_Combined_1_cluster_group_7.png"><img src="/images/Density_Combined_1_cluster_group_7.png" alt=""></a>
	<a href="/images/Density_Combined_1_cluster_group_8.png"><img src="/images/Density_Combined_1_cluster_group_8.png" alt=""></a>
	<figcaption style="align: center;">Figure 10. m-lab data clustered and demographics characteristics plotted</figcaption>
</figure>

Due to the fact that we can not spot the trends so well in all those graphs, we plotted graphs with the median values for each cluster for population, number of households and median income.

<figure class="third">
	<a href="/images/Median_Population_comb_1.png"><img src="/images/Median_Population_comb_1.png" alt=""></a>
	<a href="/images/Median_Households_comb_1.png"><img src="/images/Median_Households_comb_1.png" alt=""></a>
	<a href="/images/Median_MedianIncome_comb_1.png"><img src="/images/Median_MedianIncome_comb_1.png" alt=""></a>
	<figcaption style="align: center;">Figure 11. m-lab data mean demographics characteristics for each cluster</figcaption>
</figure>


