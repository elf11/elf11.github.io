---
layout: post
title: Linear Regression in the context of m-lab data
description: "linear regression used to outline a model of correlation between population/median income and internet speeds/availability in USA (the Outreachy - m-lab project)"
<!-- modified: 2015-08-18 -->
tags: [outreachy, m-lab, k-means]
image:
  feature: medianIncome ~ download_median.png
  credit: elf11

comments: true
share: true
---

Continuing the Outreachy article series, this article describes the outcome of applying linear regression to the m-lab data and the first steps in establishing a model of correlation between the socio-economic factors (population and median income) and the internet speed and availability for the New England region.

# Method

We builded on the previous work, meaning that we did linear regression analysis for the two most important socio-economic factors (population and median income) in correlation with MedianRTT, median upload speed and median download speed. We choose those features to analyse because both, the PCA analysis and the k-means analysis outlined the fact that those were the factors that could have been the most correlated. The analysis was done in python, using one of the linear regression libraries (statsmodel).

# New England region - Results, Code and Discussion

## linear regression - how it works

Why are we using linear regression? It is an easy to use method (not a lot of tuning required), runs fast, is highly interpretable and it can be used as basis for other methods (cross validation). 

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

## Conclusion

Analysing those data sets using k-means clustering offered us a way to visualize and explore the data sets as well as finding out trends in the data. As we said before, the data clusters in rural and urban communities, communities with a lot of population and those with less population, that get distinctive by the way they are served by ISPs. We observed that communities with a high number for population/households and with a great median income are better served by ISPs, with better upload and download speeds and lower RTT.
