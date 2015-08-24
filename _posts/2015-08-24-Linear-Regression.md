---
layout: post
title: Linear Regression in the context of m-lab data
description: "linear regression used to outline a model of correlation between population/median income and internet speeds/availability in USA (the Outreachy - m-lab project)"
<!-- modified: 2015-08-18 -->
tags: [outreachy, m-lab, k-means]
image:
  feature: upload_median ~ medianIncome.png
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

Linear regression predictics a quantitative response using a single feature=predictor/input variable and has the following form: y=β0+β1x
Where:
y is the response
x is the feature
β0 is the intercept
β1 is the coefficient for x

Together, β0 and β1 are called the model coefficients. To create our model, we must "learn" the values of those coefficients. 

### Intuition

To learn the coefficients means to estimate them. In general they are being estimated using the least squared method. The least squared method finds the line that minimizes the sum of squared residuals - or "the sum of squared errors".

Looking at Figure 1, we interpret it as follows: the black dots are the values for x and y (observed values), the blue line is the least squared error line and the red lines are the residuals/errors. The residuals are the distance between the observed value and the minimized line. 

<figure style="align:center">
	<a href="/images/lin_reg.png"><img src="/images/lin_reg.png" alt=""></a>
	<figcaption style="align: center;">Figure 1 </figcaption>
</figure>

A good question is how do our coefficients β0 and β1 relate to this line, β0 is the intercept, or the value of y when x=0; and β1 is the slope of the line.

## Method explained

Let's take a look at the data, ask some questions about it and then answer those questions using linear regression.

The data set we used for this analysis consists of New England characteristics for internet speed and availability, collected with piecewise tool from m-lab and demographics data from the broadbandmap.gov site. We used the population and median income for each of the counties in New England from the census. 

What are the features we are interested in?

- MedianRTT : the median round time trip in ms for each county in New England
- download_median: the median download speed in mb/s for each county in New England
- upload_median: the median upload speed in mb/s for each county in New England

What are we interested in? How those features correlate with the population and the median income for each of those counties.

We can visualize the relationship between those features using scatter plots. In Figure 2 below we have the population plotted against the MedianRTT, median_upload and median_download, same with medianIncome.

<figure class="third">
	<a href="/images/Relationship_feature_MedianRTT.png"><img src="/images/Relationship_feature_MedianRTT.png" alt=""></a>
	<a href="/images/Relationship_feature_download.png"><img src="/images/Relationship_feature_download.png" alt=""></a>	
	<a href="/images/Relationship_feature_upload.png"><img src="/images/Relationship_feature_upload.png" alt=""></a>
	<figcaption style="align: center;">Figure 2 </figcaption>
</figure>

Questions about the data:

1. Is there a relationship between the socio-economic features and the internet features?
2. How strong is that relationship?
3. Do population and median income influence the RTT and internet speeds?
4. What is the effect of each of those (population and median income) on the internet characteristics?

We used the following function to implement linear regression using the statsmodels library.

{% highlight python %}
def simple_linear_reg(x_str, y_str, form):
    print 'features\n',form
    # create a fitted model
    lm = smf.ols(formula=form, data=data).fit()

    print 'the coefficients\n',lm.params
    
    # create a DataFrame with the minimum and maximum values of the feature
    X_new = pd.DataFrame({x_str: [data[x_str].min(), data[x_str].max()]})
    print 'minimum and maximum values of the feature\n',X_new.head()
    
    # make predictions for those x values and store them
    preds = lm.predict(X_new)
    print 'predictions for minimum and maximum values of the feature\n',preds
    
    # first, plot the observed data
    data.plot(kind='scatter', x=x_str, y=y_str)
    
    # then, plot the least squares line
    plt.plot(X_new, preds, c='red', linewidth=2)
    title = form+'.png'
    plt.savefig(title, dpi=125)
    plt.close()
    
    # print the confidence intervals for the model coefficients
    print 'confidence intervals\n',lm.conf_int()
    
    # print the p-values for the model coefficients
    print 'p-values for model coefficients\n',lm.pvalues
    
    # print the R-squared value for the model
    print 'R-squared value for the model\n',lm.rsquared
{% endhighlight %}

We are going to analyse each coefficient for all of our models and see what they are and what they represent, but before that in the code above we made the prediction for the smallest and largest observed values of x and after that we used the predicted values to plot the least squared line. Those least squared lines can be seen in Figure 3 for MedianRTT, Figure 4 for median download and Figure 5 for median upload. 

<figure class="half">
	<a href="/images/MedianRTT ~ population.png"><img src="/images/MedianRTT ~ population.png" alt=""></a>
	<a href="/images/MedianRTT ~ medianIncome.png"><img src="/images/MedianRTT ~ medianIncome.png" alt=""></a>
	<figcaption style="align: center;">Figure 3, linear regression for median RTT </figcaption>
</figure>


MedianRTT ~ population
the coefficients
Intercept     66.845054
population    -0.000002

confidence intervals
                    0          1
Intercept   61.954426  71.735681
population  -0.000015   0.000011

p-values for model coefficients
Intercept     2.530838e-37
population    7.690526e-01

R-squared value for the model
0.00133564516481



MedianRTT ~ medianIncome
the coefficients
Intercept       94.902806
medianIncome    -0.000479

confidence intervals
                      0           1
Intercept     79.297948  110.507664
medianIncome  -0.000735   -0.000224

p-values for model coefficients
Intercept       2.235541e-18
medianIncome    3.845225e-04

R-squared value for the model
0.177502986466



Interpreting model coefficients: for example for MedianRTT ~ population model, the population coefficient (β1) means that "unit" decrease in population is associated with 0.000002 "unit" decrease in median RTT. For the MedianRTT ~ medianIncome model, the medianIncome coefficient (β1) means that a "unit" decrease in median income is associated with 0.000479 decrease in median RTT. Or more clearly here, +4.79$ to the median income decreses the median RTT with 1ms.

<figure class="half">
	<a href="/images/download_median ~ population.png"><img src="/images/download_median ~ population.png" alt=""></a>
	<a href="/images/download_median ~ medianIncome.png"><img src="/images/download_median ~ medianIncome.png" alt=""></a>
	<figcaption style="align: center;">Figure 4, linear regression for median download </figcaption>
</figure>


download_median ~ population
the coefficients
Intercept     5.850996
population    0.000006

confidence intervals
                   0         1
Intercept   4.744083  6.957908
population  0.000003  0.000009

p-values for model coefficients
Intercept     9.885625e-16
population    1.015543e-04

R-squared value for the model
0.2087835594


download_median ~ medianIncome
the coefficients
Intercept      -2.234928
medianIncome    0.000159

confidence intervals
                     0         1
Intercept    -5.890936  1.421080
medianIncome  0.000099  0.000219

p-values for model coefficients
Intercept       0.226551
medianIncome    0.000001

R-squared value for the model
0.301756007441


Here we can interpret the download_median ~ population as follows: the population coefficient (β1) means that "unit" increase in population is associated with 0.000006 "unit" increase in median download speed. Or, 6 units increase in population ads 1M units increase in download speed.

<figure class="half">
	<a href="/images/upload_median ~ population.png"><img src="/images/upload_median ~ population.png" alt=""></a>
	<a href="/images/upload_median ~ medianIncome.png"><img src="/images/upload_median ~ medianIncome.png" alt=""></a>
	<figcaption style="align: center;">Figure 3, linear regression for median upload </figcaption>
</figure>


upload_median ~ population

the coefficients

Intercept     1.212062

population    0.000002

confidence intervals
                   0         1

Intercept   0.727203  1.696922

population  0.000001  0.000004


p-values for model coefficients

Intercept     0.000005

population    0.000960


R-squared value for the model

0.155504840302



upload_median ~ medianIncome

the coefficients

Intercept      -3.044039

medianIncome    0.000080

confidence intervals

                     0         1

Intercept    -4.450465 -1.637614

medianIncome  0.000057  0.000103


p-values for model coefficients

Intercept       5.405864e-05

medianIncome    2.267954e-09


R-squared value for the model

0.425192237591


Here we can interpret the download_median ~ population as follows: the population coefficient (β1) means that "unit" increase in population is associated with 0.000002 "unit" increase in median download speed. Or, 2 units increase in population ads 1M units increase in download speed.

The linear regression model is a high bias/low variance model. This means that if we sample repeatedly, the line will stay roughly in the same place (low variance), but the average of those models will not show the true relationship (high bias). 

### Hypothesis testing and p-values

Using the model created, we tested some conventional hypothesis regarding the model coefficients:

1. _null hypothesis_: there is no relationship between the population/median income feature and median rtt, upload and download (so β1 equals 0)
2. _alternative hypothesis_: there is a relationship between the population/median income feature and median rtt, upload and download (so β1 is not equal to 0)

The hypothesis testing is strongly related to confidence intervals, in statistics a confidence interval is a interval estimate of a population parameter. Usually it is calculated from observations, samples are observed and it is different from sample to sample. Statsmodels calculates 95% confidence intervals for our model coefficients, which are interpreted as follows: if the population from which this sample was drawn was sampled 100 times, approximately 95 of those confidence intervals would contain the "true" coefficient. So, how do we relate this to the hypothesis testing? We reject the null hypothesis and believe the alternative if the _95% confidence interval does not include zero_. The p-value (in the data above) is the probability that the coeficient is actually 0. 

So, if the 95% confidence interval includes 0, then the p-value for that coeficient will be greater than 0.05. The only instance where this is the case is for the MedianRTT ~ population, where the p-value for population in relation to median rtt is greater than 0.05, so probably there is no relationship between the population and the median RTT. But, we can believe looking at the other values that there is a relationship between median rtt and median income, and download/upload speed and median income/population.

## Conclusion

Using simple linear regression we analysed the m-lab data set and the correlation between the socio-economic  features and internet characteristics. After the analysis, we can believe that there is a correlation between those characteristics and the socio-economic features influence the internet speed characteristics of the counties in New England.
