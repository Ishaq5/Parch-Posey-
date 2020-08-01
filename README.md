# Parch-Posey-<p><img height="10px" src="" align="left" hspace="0px" vspace="0px"></p> 
     
<h1>Exploratory Analysis on Porch & Posey</h1>


## Objective 

For this project, our focus is on Porch & Posey CO. The company works on statonaries specially papers, posters and prints. 
The data is stored in databases and in different tables. Some cleaning was done on the data before loading to this notebook. 
Analyzing the data will be done using pandas. 


**Throuhg this analysis, we attempt to answer some question we have. Following along this analysis is not sequentional thus there is no defined order.Starting from aggregated total sales and total orders.**

## A- Sales and orders
#### columns:
- company name.
- company website.
- primary contact person in the company.
- Total amount spent by the company.
- Total number of orders made by the company. 

because we are intrested in over all sales and orders per company
for their life value with Porch & Posey, the data set is aggregated across all orders. 


```python
# importing libraries
import psycopg2
from sqlalchemy import create_engine
import pandas as pd
import numpy as np 
import matplotlib.pyplot as plt 
import seaborn as sns
%matplotlib inline
%load_ext sql
```


```python
# display tables settings
pd.set_option('display.max_rows', 1000)
pd.set_option('display.max_columns', 100)
```


```python
# initiating connection to data_base
engine = create_engine('postgresql://postgres:***@localhost/parch_and_posey')
```


```python
# passing connection to %sql to query with sql magic methode in python
# no need to reinitiate connection 
%sql postgresql://postgres:***@localhost/parch_and_posey
```


```python
%%sql 
SELECT a.name, a.website, a.primary_poc, SUM(o.total_amt_usd) total_spent, SUM(o.total) total_quantity
FROM accounts a 
JOIN orders o on o.account_id=a.id
GROUP BY 1,2,3 ORDER BY 1,2
LIMIT 3
```

     * postgresql://postgres:***@localhost/parch_and_posey
    3 rows affected.
    




<table>
    <tr>
        <th>name</th>
        <th>website</th>
        <th>primary_poc</th>
        <th>total_spent</th>
        <th>total_quantity</th>
    </tr>
    <tr>
        <td>3M</td>
        <td>www.3m.com</td>
        <td>Orville Leavell</td>
        <td>127945.10</td>
        <td>19747</td>
    </tr>
    <tr>
        <td>Abbott Laboratories</td>
        <td>www.abbott.com</td>
        <td>Seymour Olmedo</td>
        <td>96819.92</td>
        <td>14707</td>
    </tr>
    <tr>
        <td>AbbVie</td>
        <td>www.abbvie.com</td>
        <td>Agnus Jenkin</td>
        <td>11243.63</td>
        <td>1725</td>
    </tr>
</table>



Since the query above obtained results needed for further analysis in pandas, 
now we are going to load the query result to a dataframe. 


```python
# dataframe object holding the result from the query
df_sales = pd.read_sql("""SELECT a.name company_name, a.website, a.primary_poc, SUM(o.total_amt_usd) total_spent, SUM(o.total) total_quantity
FROM accounts a 
JOIN orders o on o.account_id=a.id
GROUP BY 1,2,3 ORDER BY 1,2""", engine, index_col='company_name') # reseting index as company name for easier navigation
```


```python
# first five rows to check result
df_sales.head(5)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>website</th>
      <th>primary_poc</th>
      <th>total_spent</th>
      <th>total_quantity</th>
    </tr>
    <tr>
      <th>company_name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3M</th>
      <td>www.3m.com</td>
      <td>Orville Leavell</td>
      <td>127945.10</td>
      <td>19747</td>
    </tr>
    <tr>
      <th>Abbott Laboratories</th>
      <td>www.abbott.com</td>
      <td>Seymour Olmedo</td>
      <td>96819.92</td>
      <td>14707</td>
    </tr>
    <tr>
      <th>AbbVie</th>
      <td>www.abbvie.com</td>
      <td>Agnus Jenkin</td>
      <td>11243.63</td>
      <td>1725</td>
    </tr>
    <tr>
      <th>ADP</th>
      <td>www.adp.com</td>
      <td>September Jacquet</td>
      <td>163579.18</td>
      <td>28629</td>
    </tr>
    <tr>
      <th>Advance Auto Parts</th>
      <td>www.advanceautoparts.com</td>
      <td>Maxwell Vandergriff</td>
      <td>6962.07</td>
      <td>1286</td>
    </tr>
  </tbody>
</table>
</div>



### 1- Basic data set information
Next, we explore basic information about df_sales columns, null values, data type in each column 
and memory size used to create data frame. 


```python
# basic info 
df_sales.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Index: 350 entries, 3M to Yum Brands
    Data columns (total 4 columns):
     #   Column          Non-Null Count  Dtype  
    ---  ------          --------------  -----  
     0   website         350 non-null    object 
     1   primary_poc     350 non-null    object 
     2   total_spent     350 non-null    float64
     3   total_quantity  350 non-null    int64  
    dtypes: float64(1), int64(1), object(2)
    memory usage: 9.6+ KB
    

So fare, dataframe obtaine matches what we expected as number of entries matches number of companies 350. Also data types are expected for the columns and there are no null values in the data set. 

### 2- Basic stats
Now we attempt to see basic statistics about the data set
(mean, min, max, inner-quartiles) for total_spent and total_quantity columns, 
(frequancy, count, unique) for website, primary_poc columns. 


```python
# basic stat figures
df_sales.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>total_spent</th>
      <th>total_quantity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>350.000000</td>
      <td>350.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>66118.605229</td>
      <td>10502.185714</td>
    </tr>
    <tr>
      <th>std</th>
      <td>71299.263574</td>
      <td>10871.982596</td>
    </tr>
    <tr>
      <th>min</th>
      <td>390.250000</td>
      <td>50.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>15900.975000</td>
      <td>2540.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>38337.055000</td>
      <td>6161.500000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>95964.185000</td>
      <td>14895.500000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>382873.300000</td>
      <td>56410.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
# boxplot to visualize 'total_spent' column distribution
sns.boxplot(x=df_sales["total_spent"])
```




    <matplotlib.axes._subplots.AxesSubplot at 0x93e2670>




![png](output_17_1.png)



```python
# boxplot to visualize 'total_quantity' column distribution
sns.boxplot(x=df_sales["total_quantity"])
```




    <matplotlib.axes._subplots.AxesSubplot at 0x96caa70>




![png](output_18_1.png)


#### 2.1- Results for numerical values:
**a- total_spent:**
- mean = 66118.61USD
- std  = 71299.26USD
- max  = 382873.30USD
- min  = 390.25USD

**b- total_quantity**
- mean = 10502.20
- std  = 10871.98
- max  = 56410.00
- min  = 50.00

The boxplot above clearly shows that both features (total_spent, total_quantity)
are not symmetric in fact they are both skewed right. 


```python
# stat figures for categorial attributes
df_sales.describe(include=object)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>website</th>
      <th>primary_poc</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>350</td>
      <td>350</td>
    </tr>
    <tr>
      <th>unique</th>
      <td>350</td>
      <td>329</td>
    </tr>
    <tr>
      <th>top</th>
      <td>www.lear.com</td>
      <td>Jodee Lupo</td>
    </tr>
    <tr>
      <th>freq</th>
      <td>1</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>




```python
# duplicate entries in 'primary_poc' column
df_sales['primary_poc'].value_counts().head()
```




    Jodee Lupo          2
    Serafina Banda      2
    Khadijah Riemann    2
    Esta Engelhardt     2
    Jamel Mosqueda      2
    Name: primary_poc, dtype: int64




```python
# display duplicate entries 
# and calculating number of duplicates
contacts = df_sales['primary_poc'].value_counts() # pd.series of entries counts
max_contacts = contacts[contacts==2] # boolean to index count pd.series (contacts)

for x in max_contacts.index.tolist(): # for loop to display duplicate entries
    print(x, end=' // ')
    
print('\n') # prints new line
print('number of contact person assigned to more than one company:', len(max_contacts.index.tolist())) # calculate duplicate entries
print('total unique contact people in data set:' ,350 - len(max_contacts.index.tolist())) # calculates number of unique entries
```

    Jodee Lupo // Serafina Banda // Khadijah Riemann // Esta Engelhardt // Jamel Mosqueda // Buffy Azure // Savanna Gayman // Tamara Tuma // Chantell Drescher // Parker Hoggan // Angeles Crusoe // Kym Hagerman // Deanne Hertlein // Paige Bartos // Tuan Trainer // Nannie Brinkman // Dominique Favela // Jewell Likes // Terrilyn Kesler // Sung Shields // Anabel Haskell // 
    
    number of contact person assigned to more than one company: 21
    total unique contact people in data set: 329
    

#### 2.2- Results for categorials:

**A- website**
- unique = 350 records or entries
- freq = 1

**B- primary_poc**
- unique = 329
- freq = 2 

**For website column, the frequancy is one which means every compant has unique website as expected.
However, for primary_poc column, 329 unique entries suggests that there are repeating values for
more than one company. In other words, a person is assigned to more than one company. This might have 
different reason like contact person moved to a new company yet porch & posey didn't update company information
or might mean the contact person is a third party.**

**We used the above snippest to detemine contacting person assigned to more than one company.*

### 3- Analyzing top 25% and bottom 25% companies 

#### 3.1- Top 25% spending companies


```python
# finding top 25% by sorting descending order 'total_spent' 
top_companies = df_sales.loc[:, ['total_spent', 'total_quantity']].sort_values(by='total_spent', ascending=False).head(88)
top_companies.reset_index(inplace=True)
top_companies.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>company_name</th>
      <th>total_spent</th>
      <th>total_quantity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>EOG Resources</td>
      <td>382873.30</td>
      <td>56410</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Mosaic</td>
      <td>345618.59</td>
      <td>49246</td>
    </tr>
    <tr>
      <th>2</th>
      <td>IBM</td>
      <td>326819.48</td>
      <td>47506</td>
    </tr>
    <tr>
      <th>3</th>
      <td>General Dynamics</td>
      <td>300694.79</td>
      <td>43730</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Republic Services</td>
      <td>293861.14</td>
      <td>40833</td>
    </tr>
  </tbody>
</table>
</div>




```python
# calculating average amount spent per order in descending order
top_companies['avg_spent_per_order'] = top_companies['total_spent']/top_companies['total_quantity']
top_companies.sort_values(by='avg_spent_per_order', ascending=False).head(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>company_name</th>
      <th>total_spent</th>
      <th>total_quantity</th>
      <th>avg_spent_per_order</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>11</th>
      <td>Pacific Life</td>
      <td>255319.18</td>
      <td>33156</td>
      <td>7.700542</td>
    </tr>
    <tr>
      <th>77</th>
      <td>Fidelity National Financial</td>
      <td>110027.29</td>
      <td>14806</td>
      <td>7.431264</td>
    </tr>
    <tr>
      <th>28</th>
      <td>CHS</td>
      <td>167271.80</td>
      <td>22841</td>
      <td>7.323313</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Republic Services</td>
      <td>293861.14</td>
      <td>40833</td>
      <td>7.196658</td>
    </tr>
    <tr>
      <th>53</th>
      <td>National Oilwell Varco</td>
      <td>131318.64</td>
      <td>18427</td>
      <td>7.126425</td>
    </tr>
    <tr>
      <th>31</th>
      <td>Paccar</td>
      <td>161294.62</td>
      <td>22752</td>
      <td>7.089250</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Mosaic</td>
      <td>345618.59</td>
      <td>49246</td>
      <td>7.018206</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Goodyear Tire &amp; Rubber</td>
      <td>185463.76</td>
      <td>26597</td>
      <td>6.973108</td>
    </tr>
    <tr>
      <th>2</th>
      <td>IBM</td>
      <td>326819.48</td>
      <td>47506</td>
      <td>6.879541</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Wells Fargo</td>
      <td>200174.66</td>
      <td>29101</td>
      <td>6.878618</td>
    </tr>
  </tbody>
</table>
</div>




```python
# total amount spent by all companies
# total amount ordered by companies
all_companies_sales = df_sales['total_spent'].sum()
all_companies_orders = df_sales['total_quantity'].sum()
```


```python
# percentage of top 25% companies of total sales and orders
top25_spent_sum = top_companies['total_spent'].sum()
top25_quantity_sum = top_companies['total_quantity'].sum()

percent_top_spent = (top25_spent_sum/all_companies_sales)*100
percent_top_quantity = (top25_quantity_sum/all_companies_orders)*100

template = 'The top 25% of companies in terms of sales account for nearly {0:.2f}% of all sales.'
template1 = 'The top 25% of companies in terms of orders account for nearly {0:.2f}% of all orders.'

print(template.format(percent_top_spent), end='\n\n')
print(template1.format(percent_top_quantity))
```

    The top 25% of companies in terms of sales account for nearly 64.17% of all sales.
    
    The top 25% of companies in terms of orders account for nearly 63.41% of all orders.
    

#### 3.2- Lowest 25% spending and ordering companies


```python
# finding bottom 25% by sorting descending order 'total_spent' 
low_companies = df_sales.loc[:,['total_spent', 'total_quantity']].sort_values(by='total_spent', ascending=False).tail(87)
low_companies.reset_index(inplace=True)
low_companies.tail()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>company_name</th>
      <th>total_spent</th>
      <th>total_quantity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>82</th>
      <td>Bed Bath &amp; Beyond</td>
      <td>1069.64</td>
      <td>168</td>
    </tr>
    <tr>
      <th>83</th>
      <td>Deere</td>
      <td>1036.57</td>
      <td>170</td>
    </tr>
    <tr>
      <th>84</th>
      <td>Level 3 Communications</td>
      <td>881.73</td>
      <td>156</td>
    </tr>
    <tr>
      <th>85</th>
      <td>Delta Air Lines</td>
      <td>859.64</td>
      <td>164</td>
    </tr>
    <tr>
      <th>86</th>
      <td>Nike</td>
      <td>390.25</td>
      <td>50</td>
    </tr>
  </tbody>
</table>
</div>




```python
# calculating average amount spent per order in descending order
low_companies['avg_spent_per_order'] = low_companies['total_spent']/low_companies['total_quantity']
low_companies.sort_values(by='avg_spent_per_order', ascending=False).head(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>company_name</th>
      <th>total_spent</th>
      <th>total_quantity</th>
      <th>avg_spent_per_order</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>86</th>
      <td>Nike</td>
      <td>390.25</td>
      <td>50</td>
      <td>7.805000</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Starbucks</td>
      <td>14875.19</td>
      <td>2007</td>
      <td>7.411654</td>
    </tr>
    <tr>
      <th>41</th>
      <td>CBS</td>
      <td>8648.07</td>
      <td>1182</td>
      <td>7.316472</td>
    </tr>
    <tr>
      <th>36</th>
      <td>Newmont Mining</td>
      <td>9618.81</td>
      <td>1380</td>
      <td>6.970152</td>
    </tr>
    <tr>
      <th>43</th>
      <td>W.W. Grainger</td>
      <td>8384.61</td>
      <td>1203</td>
      <td>6.969751</td>
    </tr>
    <tr>
      <th>42</th>
      <td>Dow Chemical</td>
      <td>8431.81</td>
      <td>1221</td>
      <td>6.905659</td>
    </tr>
    <tr>
      <th>45</th>
      <td>Danaher</td>
      <td>8133.47</td>
      <td>1179</td>
      <td>6.898617</td>
    </tr>
    <tr>
      <th>44</th>
      <td>Publix Super Markets</td>
      <td>8364.40</td>
      <td>1214</td>
      <td>6.889951</td>
    </tr>
    <tr>
      <th>50</th>
      <td>C.H. Robinson Worldwide</td>
      <td>7460.32</td>
      <td>1088</td>
      <td>6.856912</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Tesoro</td>
      <td>14482.60</td>
      <td>2117</td>
      <td>6.841096</td>
    </tr>
  </tbody>
</table>
</div>




```python
# percentage of bottom 25% companies of total sales and orders
bottom25_spent_sum = low_companies['total_spent'].sum()
bottom25_quantity_sum = low_companies['total_quantity'].sum()

percent_low_spent = (bottom25_spent_sum/all_companies_sales)*100
percent_low_quantity = (bottom25_quantity_sum/all_companies_orders)*100

template = 'The bottom 25% of companies in terms of sales account for nearly {0:.2f}% of all sales.'
template1 = 'The bottom 25% of companies in terms of orders account for nearly {0:.2f}% of all orders.'

print(template.format(percent_low_spent), end='\n\n')
print(template1.format(percent_low_quantity))
```

    The bottom 25% of companies in terms of sales account for nearly 2.97% of all sales.
    
    The bottom 25% of companies in terms of orders account for nearly 3.08% of all orders.
    

#### 3.3- Findings:


```python
# pie plot demonstrating top 25% of companies
# share of total sales (total spent) 
remain_rest_sum_spent = (df_sales['total_spent'].sum())-(top25_spent_sum+bottom25_spent_sum)

labels = ['top 25%', 'bottom 25%', 'other 50%']
sizes = [top25_spent_sum,bottom25_spent_sum,remain_rest_sum_spent]
explode = (0.1, 0, 0)  

fig1, ax1 = plt.subplots()
ax1.pie(sizes,explode=explode,labels=labels, autopct='%1.1f%%', startangle=90)
ax1.axis('equal')

plt.title('top 25% companies percentages of sales')
plt.show()
```


![png](output_35_0.png)



```python
# pie plot demonstrating bottom 25% of companies
# share of total sales (total spent) 
labels = ['top 25%', 'bottom 25%', 'other 50%']
sizes = [top25_spent_sum,bottom25_spent_sum,remain_rest_sum_spent]
explode = (0, 0.3, 0)  

fig1, ax1 = plt.subplots()
ax1.pie(sizes,explode=explode,labels=labels,autopct='%1.1f%%', startangle=-90)
ax1.axis('equal')  
plt.title('bottom 25% companies percentages of sales')
plt.show()
```


![png](output_36_0.png)



```python
# calculating the average of top and bottom 25% sales
# and their percentage of all companies sales average
total_spent_mean = df_sales['total_spent'].mean()

top_companies_mean = top_companies['total_spent'].mean()
low_comanies_mean = low_companies['total_spent'].mean()

percent_above_avg_spent = (top_companies_mean/total_spent_mean)*100
percent_below_avg_spent = (low_comanies_mean/total_spent_mean)*100

top_temp = '*{0:.1f} average of (total amount) spent by the top 25% companies is {1:.1f}% more than the mean {2:.1f} of the all companies'
bottom_temp = '*{0:.1f} average of (total amount) spent by the bottom 25% companies is {1:.1f}% less than the mean {2:.1f}  of the all companies'

print(top_temp.format(top_companies_mean,percent_above_avg_spent,total_spent_mean))
print(bottom_temp.format(low_comanies_mean,percent_below_avg_spent,total_spent_mean))
```

    *168753.1 average of (total amount) spent by the top 25% companies is 255.2% more than the mean 66118.6 of the all companies
    *7901.9 average of (total amount) spent by the bottom 25% companies is 12.0% less than the mean 66118.6  of the all companies
    


```python
# bar plot to demonestrate above findings
size = [total_spent_mean, top_companies_mean, low_comanies_mean]
labels = ('all companies', ' top 25% ', 'bottom 25% ')
bins = np.arange(len(labels))

plt.bar(bins, size, color=(['blue', 'green', 'red']))
plt.xticks(bins, labels)
plt.xlabel('company category')
plt.ylabel('mean values')
plt.title('Average amount spent by companies')

plt.show()
```


![png](output_38_0.png)


*So far, we have examinned both top and bottom 25% of all companies in the data set. 
We can draw a conclusion that the top 25% of companies are doing exceptionally well as they are way far from the average sales for all other companies being 2.5 times better. 
On the other hand, bottom 25% are 11% less than average of all companies showing a significant drop in sales.
Meaning that average sales of all companies out performed bottom 25% companies by almost 9 times.*

## Conclusion

In this first part of the series of analyzing Porch & Posey database tables, We started with sales and orders to attempte 
to have a general understanding of number of companies, contact people and sales in data set. We did not find any missing data however, there are duplicate values in (primary_poc) which we expect to be unique for each company. We suggest an investegation as for the reasons and actions to be taken so the Porch & Posey won't loss companies with duplicate contact person as clients.

In terms of sales, we concluded that top 25% of companies are way above average of all 350 companies in sales. We looked further if there is a baise causing that large variance by single or couple companies but we have found more than 10 comopanies had close sales figures. 
However for bottom 25% of companies, their average is suggesting further analysis as poor sales performance comoparning to top companies is alarming. Further analysis may cover areas and attributes that might lead to poor performance.

In the next part of this series, we will take a closer look to indivisual orders for each product across different time farmes as well as the marketing attributes like channel and time of contact and their impact on order generation.
