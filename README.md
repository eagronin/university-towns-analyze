# Analysis

This section discusses the analysis ... The analysis is based on the data collected as described in [the data acquisition section](...) and processed in [the data cleaning section](...)

for data collection see [link](...).  for data cleaning and preparation for the final dataset see [link](...)

First create new data showing the decline or growth of housing prices between the recession start and 
the recession bottom. Then run a ttest comparing the university town values to the non-university towns values, 
return whether the alternative hypothesis (that the two groups are different) is true or not as well as 
the p-value of the confidence. 

Return the tuple (different, p, better) where different=True if the t-test is True at a p<0.01 
(we reject the null hypothesis), or different=False if otherwise (we cannot reject the null hypothesis). 
The value for better should be either "university town" or "non-university town" depending on which has 
a lower mean price ratio (which is equivilent to a reduced market loss).

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
a = run_ttest()
print(a)
```
