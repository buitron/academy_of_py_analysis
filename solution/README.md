
# PyCity Schools Analysis
* The top three schools in the district ranked by '% Overall Passing' are schools that are grouped into the small school size category. Cabrera High School is ranked \#1 with only 1858 students
* 8 of the top 10 % Overall Passing schools are Charter schools. Of those top 10 the bottom 2 are District schools.
* Schools that spend less than \$616 per student, which is less than half of the average spending rate per child, are able to acheive an average passing rate of over %99.


```python
from os import path
import numpy as np
import pandas as pd
import warnings
warnings.filterwarnings('ignore')
```


```python
csv_file01 = path.join('..', 'raw_data', 'schools_complete.csv')
csv_file02 = path.join('..', 'raw_data', 'students_complete.csv')
```


```python
schools_df = pd.read_csv(csv_file01)
students_df = pd.read_csv(csv_file02)
```

### Prepping DFs


```python
# add new boolean columns to students_df that show student passing ability
pass_read_score = [1 if i >= 70 else 0 for i in students_df['reading_score']]
pass_math_score = [1 if i >= 70 else 0 for i in students_df['math_score']]

total_student_score = students_df['reading_score'] + students_df['math_score']
pass_overall_score = [1 if i >= 140 else 0 for i in total_student_score]

students_df['read_pass'] = pass_read_score
students_df['math_pass'] = pass_math_score
students_df['overall_pass'] = pass_overall_score

# rename schools_df column label
schools_df = schools_df.rename(columns={'name':'school'})

```

### Formatting Functions


```python
def series_format(adjustment, mult_by, *series):
    ser_formatted = [(series[i] * mult_by).apply(lambda x: adjustment.format(x))
                 for i in range(len(series))]
    return ser_formatted

def value_format(adjustment, mult_by, *value):
    val_formatted = [adjustment.format((value[i] * mult_by)) for i in range(len(value))]
    return val_formatted
```

## District Summary


```python
total_schools = len(schools_df['school'].unique())
total_students = schools_df['size'].sum()
total_budget = schools_df['budget'].sum()
avg_math_score = students_df['math_score'].mean()
avg_read_score = students_df['reading_score'].mean()

perc_pass_math = students_df['math_pass'].sum() / total_students
perc_pass_read = students_df['read_pass'].sum() / total_students
perc_pass_overall = students_df['overall_pass'].sum() / total_students

# format values
total_students_frmt = "{:,}".format(total_students)
total_budget_frmt = "${:,}".format(total_budget)
ds_period_formatted = value_format("{:.4f}", 1, avg_math_score, avg_read_score)
ds_percent_formatted = value_format("{:.2f}%", 100, perc_pass_math, perc_pass_read, perc_pass_overall)


district_summary_df = pd.DataFrame({
    'Total Schools': total_schools,
    'Total Students': total_students_frmt,
    'Total Budget': total_budget_frmt,
    'Average Math Score': ds_period_formatted[0],
    'Average Reading Score': ds_period_formatted[1],
    '% Passing Math': ds_percent_formatted[0],
    '% Passing Reading': ds_percent_formatted[1],
    '% Overall Passing Rate': ds_percent_formatted[2]
}, index=['summary values'], columns=[
    'Total Schools',
    'Total Students',
    'Total Budget',
    'Average Math Score',
    'Average Reading Score',
    '% Passing Math',
    '% Passing Reading',
    '% Overall Passing Rate'
])

district_summary_df
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Total Schools</th>
      <th>Total Students</th>
      <th>Total Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>summary values</th>
      <td>15</td>
      <td>39,170</td>
      <td>$24,649,428</td>
      <td>78.9854</td>
      <td>81.8778</td>
      <td>74.98%</td>
      <td>85.81%</td>
      <td>89.39%</td>
    </tr>
  </tbody>
</table>
</div>




```python
students_gb_sm_df = students_df.groupby(by='school').sum().reset_index()

school_summary_df = pd.merge(schools_df,students_gb_sm_df, how='outer', on='school')
school_summary_df = school_summary_df.rename(columns={
    'school': 'School',
    'type': 'School Type',
    'size': 'Total Students',
    'budget': 'Total School Budget',
    'reading_score': 'Average Reading Score',
    'math_score': 'Average Math Score',
    'read_pass': '% Passing Reading',
    'math_pass': '% Passing Math',
    'overall_pass': '% Overall Passing Rate'
})

school_summary_df['Per Student Budget'] = school_summary_df['Total School Budget'] / school_summary_df['Total Students']
school_summary_df['Average Math Score'] = school_summary_df['Average Math Score'] / school_summary_df['Total Students']
school_summary_df['Average Reading Score'] = school_summary_df['Average Reading Score'] / school_summary_df['Total Students']
school_summary_df['% Passing Reading'] = school_summary_df['% Passing Reading'] / school_summary_df['Total Students']
school_summary_df['% Passing Math'] = school_summary_df['% Passing Math'] / school_summary_df['Total Students']
school_summary_df['% Overall Passing Rate'] = school_summary_df['% Overall Passing Rate'] / school_summary_df['Total Students']
```

## School Summary


```python
# transfer of ownership in order to clean school_summary_df
clean_ss_df = school_summary_df.copy()

# format columns
ss_money_period_formatted = series_format("${:,.2f}", 1, clean_ss_df['Total School Budget'], clean_ss_df['Per Student Budget'])
ss_period_formatted = series_format("{:.4f}", 1, clean_ss_df['Average Math Score'], clean_ss_df['Average Reading Score'])
ss_percent_formatted = series_format("{:.2f}%", 100, clean_ss_df['% Passing Math'], clean_ss_df['% Passing Reading'], clean_ss_df['% Overall Passing Rate'])

ss_df = pd.DataFrame({
    'School': clean_ss_df['School'],
    'School Type': clean_ss_df['School Type'],
    'Total Students': clean_ss_df['Total Students'].map("{:,}".format,),
    'Total School Budget': ss_money_period_formatted[0],
    'Per Student Budget': ss_money_period_formatted[1],
    'Average Math Score': ss_period_formatted[0],
    'Average Reading Score': ss_period_formatted[1],
    '% Passing Math': ss_percent_formatted[0],
    '% Passing Reading': ss_percent_formatted[1],
    '% Overall Passing Rate': ss_percent_formatted[2]
}, columns=[
    'School','School Type','Total Students','Total School Budget',
    'Per Student Budget','Average Math Score','Average Reading Score',
    '% Passing Math','% Passing Reading','% Overall Passing Rate'
]).set_index('School')
del ss_df.index.name

ss_df
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2,917</td>
      <td>$1,910,635.00</td>
      <td>$655.00</td>
      <td>76.6294</td>
      <td>81.1827</td>
      <td>65.68%</td>
      <td>81.32%</td>
      <td>84.98%</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2,949</td>
      <td>$1,884,411.00</td>
      <td>$639.00</td>
      <td>76.7118</td>
      <td>81.1580</td>
      <td>65.99%</td>
      <td>80.74%</td>
      <td>84.67%</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>Charter</td>
      <td>1,761</td>
      <td>$1,056,600.00</td>
      <td>$600.00</td>
      <td>83.3595</td>
      <td>83.7257</td>
      <td>93.87%</td>
      <td>95.85%</td>
      <td>99.38%</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>District</td>
      <td>4,635</td>
      <td>$3,022,020.00</td>
      <td>$652.00</td>
      <td>77.2898</td>
      <td>80.9344</td>
      <td>66.75%</td>
      <td>80.86%</td>
      <td>84.88%</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>1,468</td>
      <td>$917,500.00</td>
      <td>$625.00</td>
      <td>83.3515</td>
      <td>83.8168</td>
      <td>93.39%</td>
      <td>97.14%</td>
      <td>99.46%</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>2,283</td>
      <td>$1,319,574.00</td>
      <td>$578.00</td>
      <td>83.2742</td>
      <td>83.9895</td>
      <td>93.87%</td>
      <td>96.54%</td>
      <td>99.26%</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>1,858</td>
      <td>$1,081,356.00</td>
      <td>$582.00</td>
      <td>83.0619</td>
      <td>83.9758</td>
      <td>94.13%</td>
      <td>97.04%</td>
      <td>99.57%</td>
    </tr>
    <tr>
      <th>Bailey High School</th>
      <td>District</td>
      <td>4,976</td>
      <td>$3,124,928.00</td>
      <td>$628.00</td>
      <td>77.0484</td>
      <td>81.0340</td>
      <td>66.68%</td>
      <td>81.93%</td>
      <td>85.19%</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>Charter</td>
      <td>427</td>
      <td>$248,087.00</td>
      <td>$581.00</td>
      <td>83.8033</td>
      <td>83.8150</td>
      <td>92.51%</td>
      <td>96.25%</td>
      <td>98.59%</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>962</td>
      <td>$585,858.00</td>
      <td>$609.00</td>
      <td>83.8399</td>
      <td>84.0447</td>
      <td>94.59%</td>
      <td>95.95%</td>
      <td>99.17%</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>Charter</td>
      <td>1,800</td>
      <td>$1,049,400.00</td>
      <td>$583.00</td>
      <td>83.6822</td>
      <td>83.9550</td>
      <td>93.33%</td>
      <td>96.61%</td>
      <td>99.22%</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>3,999</td>
      <td>$2,547,363.00</td>
      <td>$637.00</td>
      <td>76.8427</td>
      <td>80.7447</td>
      <td>66.37%</td>
      <td>80.22%</td>
      <td>84.75%</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>4,761</td>
      <td>$3,094,650.00</td>
      <td>$650.00</td>
      <td>77.0725</td>
      <td>80.9664</td>
      <td>66.06%</td>
      <td>81.22%</td>
      <td>84.98%</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>District</td>
      <td>2,739</td>
      <td>$1,763,916.00</td>
      <td>$644.00</td>
      <td>77.1026</td>
      <td>80.7463</td>
      <td>68.31%</td>
      <td>79.30%</td>
      <td>84.78%</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>Charter</td>
      <td>1,635</td>
      <td>$1,043,130.00</td>
      <td>$638.00</td>
      <td>83.4183</td>
      <td>83.8489</td>
      <td>93.27%</td>
      <td>97.31%</td>
      <td>99.08%</td>
    </tr>
  </tbody>
</table>
</div>



## Top Performing Schools (By Passing Rate)


```python
ss_df.sort_values(by='% Overall Passing Rate', ascending=False).head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>1,858</td>
      <td>$1,081,356.00</td>
      <td>$582.00</td>
      <td>83.0619</td>
      <td>83.9758</td>
      <td>94.13%</td>
      <td>97.04%</td>
      <td>99.57%</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>1,468</td>
      <td>$917,500.00</td>
      <td>$625.00</td>
      <td>83.3515</td>
      <td>83.8168</td>
      <td>93.39%</td>
      <td>97.14%</td>
      <td>99.46%</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>Charter</td>
      <td>1,761</td>
      <td>$1,056,600.00</td>
      <td>$600.00</td>
      <td>83.3595</td>
      <td>83.7257</td>
      <td>93.87%</td>
      <td>95.85%</td>
      <td>99.38%</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>2,283</td>
      <td>$1,319,574.00</td>
      <td>$578.00</td>
      <td>83.2742</td>
      <td>83.9895</td>
      <td>93.87%</td>
      <td>96.54%</td>
      <td>99.26%</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>Charter</td>
      <td>1,800</td>
      <td>$1,049,400.00</td>
      <td>$583.00</td>
      <td>83.6822</td>
      <td>83.9550</td>
      <td>93.33%</td>
      <td>96.61%</td>
      <td>99.22%</td>
    </tr>
  </tbody>
</table>
</div>



## Bottom Performing Schools (By Passing Rate)


```python
ss_df.sort_values(by='% Overall Passing Rate', ascending=True).head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2,949</td>
      <td>$1,884,411.00</td>
      <td>$639.00</td>
      <td>76.7118</td>
      <td>81.1580</td>
      <td>65.99%</td>
      <td>80.74%</td>
      <td>84.67%</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>3,999</td>
      <td>$2,547,363.00</td>
      <td>$637.00</td>
      <td>76.8427</td>
      <td>80.7447</td>
      <td>66.37%</td>
      <td>80.22%</td>
      <td>84.75%</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>District</td>
      <td>2,739</td>
      <td>$1,763,916.00</td>
      <td>$644.00</td>
      <td>77.1026</td>
      <td>80.7463</td>
      <td>68.31%</td>
      <td>79.30%</td>
      <td>84.78%</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>District</td>
      <td>4,635</td>
      <td>$3,022,020.00</td>
      <td>$652.00</td>
      <td>77.2898</td>
      <td>80.9344</td>
      <td>66.75%</td>
      <td>80.86%</td>
      <td>84.88%</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2,917</td>
      <td>$1,910,635.00</td>
      <td>$655.00</td>
      <td>76.6294</td>
      <td>81.1827</td>
      <td>65.68%</td>
      <td>81.32%</td>
      <td>84.98%</td>
    </tr>
  </tbody>
</table>
</div>



## Math Scores by Grade


```python
school_math_scores_df = students_df.groupby(by=['school','grade'])[['math_score']].mean()
school_math_scores_df = school_math_scores_df.pivot_table(index='school', columns='grade', values='math_score')
school_math_scores_df = school_math_scores_df.rename_axis(None).rename_axis(None, axis=1)

# format values
grade_level = ['9th','10th','11th','12th']
for i in range(len(grade_level)):
    school_math_scores_df[grade_level[i]] = school_math_scores_df[grade_level[i]].map("{:.4f}".format)

school_math_scores_df[['9th', '10th', '11th', '12th']]
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>9th</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>77.0837</td>
      <td>76.9968</td>
      <td>77.5156</td>
      <td>76.4922</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.0947</td>
      <td>83.1545</td>
      <td>82.7656</td>
      <td>83.2775</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>76.4030</td>
      <td>76.5400</td>
      <td>76.8843</td>
      <td>77.1514</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>77.3613</td>
      <td>77.6723</td>
      <td>76.9181</td>
      <td>76.1800</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>82.0440</td>
      <td>84.2291</td>
      <td>83.8421</td>
      <td>83.3562</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>77.4385</td>
      <td>77.3374</td>
      <td>77.1360</td>
      <td>77.1866</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.7874</td>
      <td>83.4298</td>
      <td>85.0000</td>
      <td>82.8554</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>77.0273</td>
      <td>75.9087</td>
      <td>76.4466</td>
      <td>77.2256</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>77.1879</td>
      <td>76.6911</td>
      <td>77.4917</td>
      <td>76.8632</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.6255</td>
      <td>83.3720</td>
      <td>84.3281</td>
      <td>84.1215</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>76.8600</td>
      <td>76.6125</td>
      <td>76.3956</td>
      <td>77.6907</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>83.4208</td>
      <td>82.9174</td>
      <td>83.3835</td>
      <td>83.7790</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.5900</td>
      <td>83.0879</td>
      <td>83.4988</td>
      <td>83.4970</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.0856</td>
      <td>83.7244</td>
      <td>83.1953</td>
      <td>83.0358</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.2647</td>
      <td>84.0103</td>
      <td>83.8368</td>
      <td>83.6450</td>
    </tr>
  </tbody>
</table>
</div>



## Reading Scores by Grade


```python
school_read_scores_df = students_df.groupby(by=['school','grade'])[['reading_score']].mean()
school_read_scores_df = school_read_scores_df.pivot_table(index='school', columns='grade', values='reading_score')
school_read_scores_df = school_read_scores_df.rename_axis(None).rename_axis(None, axis=1)

# format values
grade_level = ['9th','10th','11th','12th']
for i in range(len(grade_level)):
    school_read_scores_df[grade_level[i]] = school_read_scores_df[grade_level[i]].map("{:.4f}".format)

school_read_scores_df[['9th', '10th', '11th', '12th']]
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>9th</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>81.3032</td>
      <td>80.9072</td>
      <td>80.9456</td>
      <td>80.9125</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.6761</td>
      <td>84.2532</td>
      <td>83.7884</td>
      <td>84.2880</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>81.1986</td>
      <td>81.4089</td>
      <td>80.6403</td>
      <td>81.3849</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>80.6327</td>
      <td>81.2627</td>
      <td>80.4036</td>
      <td>80.6623</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>83.3692</td>
      <td>83.7069</td>
      <td>84.2881</td>
      <td>84.0137</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>80.8669</td>
      <td>80.6601</td>
      <td>81.3961</td>
      <td>80.8571</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.6772</td>
      <td>83.3246</td>
      <td>83.8155</td>
      <td>84.6988</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>81.2903</td>
      <td>81.5124</td>
      <td>81.4175</td>
      <td>80.3060</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>81.2607</td>
      <td>80.7734</td>
      <td>80.6160</td>
      <td>81.2276</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.8073</td>
      <td>83.6120</td>
      <td>84.3359</td>
      <td>84.5912</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>80.9931</td>
      <td>80.6298</td>
      <td>80.8648</td>
      <td>80.3764</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>84.1226</td>
      <td>83.4420</td>
      <td>84.3738</td>
      <td>82.7817</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.7289</td>
      <td>84.2542</td>
      <td>83.5855</td>
      <td>83.8314</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.9398</td>
      <td>84.0215</td>
      <td>83.7646</td>
      <td>84.3177</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.8333</td>
      <td>83.8128</td>
      <td>84.1563</td>
      <td>84.0732</td>
    </tr>
  </tbody>
</table>
</div>



## Scores by School Spending

### dynamic bin and label generator for per student spending ranges


```python
# locate the per student spending max and mins
spend_min = school_summary_df['Per Student Budget'].min()
spend_max = school_summary_df['Per Student Budget'].max()

# dynamic bin generator
bins = list(np.linspace(spend_min - 1,spend_max + 1,num=(spend_max-spend_min)/14, dtype=int))

# dynamic label generator
labels = ["<${}".format(bins[1])]
for k in range(len(bins)):
    labels.append("${0}-{1}".format(bins[k-1], bins[k]))
del labels[1:3]
```


```python
school_summary_df['Spending Ranges (Per Student)'] = pd.cut(school_summary_df['Per Student Budget'], bins=bins, labels=labels)
school_spending_ranges_df = school_summary_df[[
    'Spending Ranges (Per Student)','Average Math Score','Average Reading Score',
    '% Passing Math','% Passing Reading','% Overall Passing Rate']].copy()

# grouby Spending Ranges (Per Student)
school_spend_gp_mn_df = school_spending_ranges_df.groupby(by='Spending Ranges (Per Student)').mean()

# format columns

school_spend_gp_mn_df['% Passing Math'] = series_format("{:.2f}%", 100, school_spend_gp_mn_df['% Passing Math'])[0]
school_spend_gp_mn_df['% Passing Reading'] = series_format("{:.2f}%", 100, school_spend_gp_mn_df['% Passing Reading'])[0]
school_spend_gp_mn_df['% Overall Passing Rate'] = series_format("{:.2f}%", 100, school_spend_gp_mn_df['% Overall Passing Rate'])[0]

school_spend_gp_mn_df
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>Spending Ranges (Per Student)</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>&lt;$596</th>
      <td>83.455399</td>
      <td>83.933814</td>
      <td>93.46%</td>
      <td>96.61%</td>
      <td>99.16%</td>
    </tr>
    <tr>
      <th>$596-616</th>
      <td>83.599686</td>
      <td>83.885211</td>
      <td>94.23%</td>
      <td>95.90%</td>
      <td>99.27%</td>
    </tr>
    <tr>
      <th>$616-636</th>
      <td>80.199966</td>
      <td>82.425360</td>
      <td>80.04%</td>
      <td>89.54%</td>
      <td>92.32%</td>
    </tr>
    <tr>
      <th>$636-656</th>
      <td>77.866721</td>
      <td>81.368774</td>
      <td>70.35%</td>
      <td>83.00%</td>
      <td>86.87%</td>
    </tr>
  </tbody>
</table>
</div>



## Scores by School Size

### dynamic bin and label generator for school size


```python
# locate the per student schooling max and mins
school_min = school_summary_df['Total Students'].min()
school_max = school_summary_df['Total Students'].max()

# dynamic bin generator
bins = list(np.linspace(school_min - 1,school_max + 1,num=(school_max-school_min)/1000, dtype=int))

# dynamic label generator
labels = ["(<{})".format(bins[1])]
for k in range(len(bins)):
    labels.append("({0}-{1})".format(bins[k-1], bins[k]))
del labels[1:3]

labels[0] = "Small {}".format(labels[0])
labels[1] = "Medium {}".format(labels[1])
labels[2] = "Large {}".format(labels[2])
```


```python
school_summary_df['School Size'] = pd.cut(school_summary_df['Total Students'], bins=bins, labels=labels)
school_size_ranges_df = school_summary_df[[
    'School Size','Average Math Score','Average Reading Score',
    '% Passing Math','% Passing Reading','% Overall Passing Rate']].copy()

# grouby School Size (Per Student)
school_size_gp_mn_df = school_size_ranges_df.groupby('School Size').mean()

# format columns
school_size_gp_mn_df['% Passing Math'] = series_format("{:.2f}%", 100, school_size_gp_mn_df['% Passing Math'])[0]
school_size_gp_mn_df['% Passing Reading'] = series_format("{:.2f}%", 100, school_size_gp_mn_df['% Passing Reading'])[0]
school_size_gp_mn_df['% Overall Passing Rate'] = series_format("{:.2f}%", 100, school_size_gp_mn_df['% Overall Passing Rate'])[0]

school_size_gp_mn_df
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School Size</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Small (&lt;1943)</th>
      <td>83.502373</td>
      <td>83.883125</td>
      <td>93.59%</td>
      <td>96.59%</td>
      <td>99.21%</td>
    </tr>
    <tr>
      <th>Medium (1943-3460)</th>
      <td>78.429493</td>
      <td>81.769122</td>
      <td>73.46%</td>
      <td>84.47%</td>
      <td>88.42%</td>
    </tr>
    <tr>
      <th>Large (3460-4977)</th>
      <td>77.063340</td>
      <td>80.919864</td>
      <td>66.46%</td>
      <td>81.06%</td>
      <td>84.95%</td>
    </tr>
  </tbody>
</table>
</div>



## Scores by School Type


```python
school_type_gp_mn_df = school_summary_df.groupby(by='School Type').mean()

school_type_df = school_type_gp_mn_df[[
    'Average Math Score','Average Reading Score','% Passing Math',
    '% Passing Reading','% Overall Passing Rate']].copy()

# format columns
school_type_df['% Passing Math'] = series_format("{:.2f}%", 100, school_type_df['% Passing Math'])[0]
school_type_df['% Passing Reading'] = series_format("{:.2f}%", 100, school_type_df['% Passing Reading'])[0]
school_type_df['% Overall Passing Rate'] = series_format("{:.2f}%", 100, school_type_df['% Overall Passing Rate'])[0]

school_type_df
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School Type</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Charter</th>
      <td>83.473852</td>
      <td>83.896421</td>
      <td>93.62%</td>
      <td>96.59%</td>
      <td>99.22%</td>
    </tr>
    <tr>
      <th>District</th>
      <td>76.956733</td>
      <td>80.966636</td>
      <td>66.55%</td>
      <td>80.80%</td>
      <td>84.89%</td>
    </tr>
  </tbody>
</table>
</div>


## Merry Christmas!!
<img src="../images/harry_potter_mean_girls_christmas.gif"/>