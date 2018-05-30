# Analysis

## Overview 
This section tests the hypothesis whether university towns have their mean housing prices less effected by recessions. This is done by running a t-test to compare the ratio of the mean price of houses in university towns in the quarter of the recession bottom compared to the quarter before the recession starts. (price_ratio=recession_bottom/quarter_before_recession).

The previous step, which describes data cleaning and processing, is described [here](https://eagronin.github.io/university-towns-prepare/).

## Finding Recession Start, End and Bottom

In order to test the hypothesis whether university towns have their mean housing prices less effected by recessions, we first need to define what we mean by recession. 

A recession is defined as starting with two consecutive quarters of GDP decline, and ending with two consecutive quarters of GDP growth.  

A recession bottom is the quarter within a recession which had the lowest GDP.

We then proceed by finding the recession start, recession end and recession bottom in the data.  We will use these figures in 


This function returns the year and quarter of the recession start time as a string value in a format such as 2005Q3:

```
// 
def get_recession_start():
    gdp = gdp_lead_lag()
    gdp['Recession Start Dummy'] = 0
    gdp['Recession Start Dummy'][(gdp['Change in GDP'] < 0) & (gdp['Lead Change in GDP'] < 0)] = 1
    recession_start = gdp[gdp['Recession Start Dummy'] == 1]
    recession_start = recession_start.reset_index(drop = True)
    recession_start = recession_start['Quarter'].iloc[0]
    return recession_start
```

The results shows that the recession started in 2008Q3.

This function returns the year and quarter of the recession end time as a string value in a format such as 2005Q3:

```
def get_recession_end():
    gdp = gdp_lead_lag()
    recession_start = get_recession_start()    
    gdp['Recession Start Dummy'] = np.nan
    gdp['Recession Start Dummy'][gdp['Quarter'] == recession_start] = 1
    gdp['Recession Start Dummy'] = gdp['Recession Start Dummy'].ffill()
    gdp['Recession End Dummy'] = np.nan
    gdp['Recession End Dummy'][(gdp['Change in GDP'] > 0) & (gdp['Lagged Change in GDP'] > 0)] = 1
    recession_end = gdp[(gdp['Recession Start Dummy'] == 1) & (gdp['Recession End Dummy'] == 1)]
    recession_end = recession_end.reset_index(drop = True)
    recession_end = recession_end['Quarter'].iloc[0]
    return recession_end
```

The results show that the recession ended in 2009Q4.

This function returns the year and quarter of the recession bottom time as a string value in a format such as 2005Q3:

```
def get_recession_bottom():
    # Load data
    gdp = load_gdp_data()    # Get recession start and end and delete data outside of the recession period
    start = get_recession_start()
    end = get_recession_end()
    gdp['Start'] = np.nan
    gdp.Start[gdp.Quarter == start] = 1
    gdp.Start = gdp.Start.ffill()
    gdp['End'] = np.nan
    gdp.End[gdp.Quarter == end] = 1
    gdp.End = gdp.End.bfill()
    recession = gdp[(gdp.Start == 1) & (gdp.End == 1)]
    recession = recession[recession.GDP == recession.GDP.min()]
    recession = recession.reset_index(drop = True)
    recession = recession.Quarter.iloc[0]
    return recession
```

The results show that the recession bottom was reached in 2009Q2.

## The t-test

First, we create new data showing the decline or growth of housing prices between the recession start and 
the recession bottom. Then we run a ttest comparing the university town values to the non-university towns values, 
which returns whether the null hypothesis (that the two groups are the same) is rejected as well as 
the p-value of the confidence. 

The following function returns the tuple (different, p, better) where different=True if the t-test is True at a p<0.01 
(we reject the null hypothesis), or different=False if otherwise (we cannot reject the null hypothesis). 
The value for better is either "university town" or "non-university town" depending on which has 
a higher mean price ratio (which is equivilent to a reduced market loss).

```
def run_ttest():    
    # Create university towns dummy
    u_towns = get_list_of_university_towns()
    u_towns = u_towns[['State', 'RegionName']]
    u_towns['U_Town'] = 1
    u_towns = u_towns.set_index(['State', 'RegionName'])
    
    # Merge university towns with housing data by state code and region name
    x = convert_housing_data_to_quarters()
    x = x.merge(u_towns, how = 'left', left_index = True, right_index = True)
    x.U_Town[x.U_Town.isnull()] = 0
    
    # Analyze trends in housing values
    start = get_recession_start()
    bottom = get_recession_bottom()
    y = x[[start, bottom, 'U_Town']]
    y['Change'] = y['2009Q2'] / y['2008Q3']
    ut = y[y['U_Town'] == 1]['Change'].dropna()
    not_ut = y[y['U_Town'] == 0]['Change'].dropna()
    t = ttest_ind(ut, not_ut)
    p = t[1]
    different = False
    if p < 0.01:
        different = True
    better = 'university town'
    if ut.mean() < not_ut.mean():
        better = 'non-university town'
    print(ut.mean())
    print(not_ut.mean())
    answer = (different, p, better)
    return answer
```

## Results

The resulting tuple is (True, 0.0031, 'university town'), which means that the null hypotheses (the two groups are the same) is rejected with the p-value of 0.0031.  It further suggests that university towns experienced smaller housing price declines during the recession of 2008-2009 compared to non-university towns.
