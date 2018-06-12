# Observations

* Of the drug treatments, Capomulin and Ramicane show the most promise. Both decreased tumor volume, had markedly better survival rates, and had relatively less aggressive growth of metastatic sites compared to the other treatements.

* Propriva has a markedly worse survival rate than the other drugs, despite not being noticeably worse in tumor growth or growth of metastatic sites.

* The value of the metastatic sites count data point is worth exploring. Of the data points collected, it's the only one in which there's a clear differentiation between the 10 treatments. For the other data points, the treatments either track the placebo or break off showing some promise.

A few things I would work on with additional time:

* One of the mice (g989) in the dataset was treated with multiple drugs. I would create deduplication logic which removes it.

* Create some logic which always places placebo at the bottom.

* Size up the plots and make them easier to read and to track the data points.

```python
#import dependencies

import pandas as pd
import matplotlib.pyplot as plt
import numpy as np
import seaborn as sns
```


```python
#store raw data, print info() for overview of available data

trial_data = pd.read_csv('raw_data/clinicaltrial_data.csv')
drug_data = pd.read_csv('raw_data/mouse_drug_data.csv')

print(trial_data.info())

print(drug_data.info())

# Merge the two together to put drug data in the clinical trial data

trial_data = trial_data.merge(drug_data, on='Mouse ID', how='left')

print(trial_data.info())

#extra entries (1906 instead or original 1893 because one of the mice is listed as being on multiple treatements)

tumor_volume_group = trial_data.groupby(['Drug', 'Timepoint'])['Tumor Volume (mm3)'].mean()
tumor_volume = tumor_volume_group.to_frame()

#confirming multiple mouse IDs
mouse_counts = drug_data['Mouse ID'].value_counts()

print(type(mouse_counts))

#drop the duplicates
#trial_data = trial_data[trial_data['Mouse ID'] != 'g989']

#trial_data.reset_index()

#print(trial_data.info())


```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 1893 entries, 0 to 1892
    Data columns (total 4 columns):
    Mouse ID              1893 non-null object
    Timepoint             1893 non-null int64
    Tumor Volume (mm3)    1893 non-null float64
    Metastatic Sites      1893 non-null int64
    dtypes: float64(1), int64(2), object(1)
    memory usage: 59.2+ KB
    None
    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 250 entries, 0 to 249
    Data columns (total 2 columns):
    Mouse ID    250 non-null object
    Drug        250 non-null object
    dtypes: object(2)
    memory usage: 4.0+ KB
    None
    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 1906 entries, 0 to 1905
    Data columns (total 5 columns):
    Mouse ID              1906 non-null object
    Timepoint             1906 non-null int64
    Tumor Volume (mm3)    1906 non-null float64
    Metastatic Sites      1906 non-null int64
    Drug                  1906 non-null object
    dtypes: float64(1), int64(2), object(2)
    memory usage: 89.3+ KB
    None
    <class 'pandas.core.series.Series'>


# Scatter plot, tumor volume for each treatment


```python
#Creates a groupby object for mean of tumor sizes by drug and timepoint, then converts to DF
tumor_volume_group = trial_data.groupby(['Drug', 'Timepoint'])['Tumor Volume (mm3)'].mean()
tumor_volume = tumor_volume_group.to_frame()

#Re-arranging the format of the multi-index DF for insertion into a scatter plot
tumor_volume = tumor_volume.unstack().transpose().reset_index(level=0, drop=True)

#Create data for x-axis (time) and labels (drug names)
timepoint = tumor_volume.index
drugs = tumor_volume.columns

#standard error calculation for error bar
standard_error=tumor_volume.sem()

#marker array to cycle through in loop for data point differentiation
markers = ['o', 's', 'x', 'v']

#loop through each unique drug in drug name, and create scatter plot w/ error bars
for x in range(len(drugs)):

    volume_line=tumor_volume.iloc[:, x]

    marker_position = (x)%4
    
    plt.errorbar(timepoint, volume_line, yerr=standard_error, marker=markers[marker_position], linestyle='dashed')

#title
plt.title('Tumor Volume During Treatment')

#grid
plt.grid(linestyle='dashed')

#limits
plt.xlim(min(timepoint)-5, max(timepoint)+5)

#axes labels
plt.xlabel('Time (Days)')
plt.ylabel('Tumor Size (mm)')

#anchors legend to outside of the chart
plt.legend(bbox_to_anchor=(1.04,1), loc="upper left")

#show plot
plt.show()

#need to figure out how to resize for better visibility
```


![png](output_4_0.png)


# Scatter plot, number of metastatic sites for each treatment


```python
#Creates a groupby object for mean of tumor sizes by drug and timepoint, then converts to DF
metastatic_group = trial_data.groupby(['Drug', 'Timepoint'])['Metastatic Sites'].mean()
metastatic = metastatic_group.to_frame()

#Re-arranging the format of the multi-index DF for insertion into a scatter plot
metastatic = metastatic.unstack().transpose().reset_index(level=0, drop=True)

#Create data for x-axis (time) and labels (drug names)
timepoint = metastatic.index
drugs = metastatic.columns

#standard error calculation for error bar
standard_error=metastatic.sem()

#marker array to cycle through in loop for data point differentiation
markers = ['o', 's', 'x', 'v']

#loop through each unique drug in drug name, and create scatter plot w/ error bars
for x in range(len(drugs)):

    meta_line=metastatic.iloc[:, x]

    marker_position = (x)%4
    
    plt.errorbar(timepoint, meta_line, yerr=standard_error, marker=markers[marker_position], linestyle='dashed')
    
#title
plt.title('Metastatic Sites During Treatment')
    
#limits
plt.xlim(min(timepoint)-5, max(timepoint)+5)

#grids
plt.grid(linestyle='dashed')

#axes labels
plt.xlabel('Treatment Duration (Days)')
plt.ylabel('Number of Metastatic Sites')

#anchors legend to outside of the chart
plt.legend(bbox_to_anchor=(1.04,1), loc="upper left")

#show plot
plt.show()

#need to figure out how to resize for better visibility
```


![png](output_6_0.png)


# Scatter plot, survival rates for each treatment


```python
#Creates a groupby object for mean of tumor sizes by drug and timepoint, then converts to DF
survival_group = trial_data.groupby(['Drug', 'Timepoint'])['Mouse ID'].count()
survival = survival_group.to_frame()

#Re-arranging the format of the multi-index DF for insertion into a scatter plot
survival = survival.unstack().transpose().reset_index(level=0, drop=True)

#Create data for x-axis (time) and labels (drug names)
timepoint = survival.index
drugs = survival.columns

#standard error calculation for error bar
standard_error=survival.sem()

#marker array to cycle through in loop for data point differentiation
markers = ['o', 's', 'x', 'v']

#loop through each unique drug in drug name, and create scatter plot w/ error bars
for x in range(len(drugs)):

    surviving_mice=survival.iloc[:, x]
    
    initial_mice=survival.iloc[0, x]

    marker_position = (x)%4
    
    plt.errorbar(timepoint, surviving_mice/initial_mice * 100, yerr=standard_error, marker=markers[marker_position], linestyle='dashed')
    
#title
plt.title('Mice Survival During Treatment')

#grids
plt.grid(linestyle='dashed')

#limits
plt.xlim(min(timepoint)-5, max(timepoint)+5)

#axes labels
plt.xlabel('Treatment Duration (Days)')
plt.ylabel('Survival Rate (%)')

#anchors legend to outside of the chart
plt.legend(bbox_to_anchor=(1.04,1), loc="upper left")

#show plot
plt.show()

#need to figure out how to resize for better visibility
```


![png](output_8_0.png)


# Bar chart, tumor volume percent change for each treatment


```python
# Use the data from the first scatter chart on tumor decrease

#Blank list to track % change for each treatment
tumor_changes = []

#Blank list to track whether % change is up or down, and colors to assign
bar_colors =[]

for x in range(len(drugs)):

    initial_tumor_size=tumor_volume.iloc[0,x]
    final_tumor_size=tumor_volume.iloc[-1,x]
    tumor_change = (final_tumor_size - initial_tumor_size) / initial_tumor_size*100
    tumor_changes.append(tumor_change)
    
    if (tumor_change > 0):
        
        bar_colors.append('red')
        
    else:
        
        bar_colors.append('green')

plt.title('Tumor Volume Changes')        

plt.hlines(0,-1,len(tumor_changes))

plt.xticks(rotation='vertical')

plt.xlabel('Drug')

plt.ylabel('Percent Change in Tumor Volume')

plt.bar(drugs, tumor_changes, color=bar_colors)
```




    <Container object of 10 artists>




![png](output_10_1.png)

# UCBDataBootcampHW5
