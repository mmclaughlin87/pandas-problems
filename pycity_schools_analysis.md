
# PyCity School Analysis
- Observed Trend 1: Charter Schools have outperformed Districts Schools, with higher average scores in math and reading, and higher passing rates.
- Observed Trend 2: Smaller schools have outperformed larger schools. Moving from large to medium to small schools, average scores and passing rates improve.
- Observed Trend 3: Surprisingly, schools with lower spending per student have outperformed schools with higher spending per student. Perhaps the schools with larger budgets are being given additional funding to overcome poor scores.


```python
import pandas as pd
import os

filepath_schools = os.path.join("raw_data","schools_complete.csv")
filepath_students = os.path.join("raw_data","students_complete.csv")

df_schools = pd.read_csv(filepath_schools)
df_students = pd.read_csv(filepath_students)
```

# District Summary


```python
# # Create a high level snapshot (in table form) of the district's key metrics

total_schools = len(df_schools)
total_students = df_schools["size"].sum()
total_budget = df_schools["budget"].sum()
avg_math = df_students["math_score"].mean()
avg_reading = df_students["reading_score"].mean()

students_passing_math = df_students.loc[df_students["math_score"] >= 70,:]
passing_math = len(students_passing_math)
pct_passing_math = (passing_math / total_students)*100

students_passing_reading = df_students.loc[df_students["reading_score"] >= 70,:]
passing_reading = len(students_passing_reading)
pct_passing_reading = (passing_reading / total_students)*100

passing_rate = (pct_passing_reading + pct_passing_math)/2

# Format items
total_budget = '${:,.0f}'.format(total_budget)
avg_math = '{0:.2f}'.format(avg_math)
avg_reading = '{0:.2f}'.format(avg_reading)
pct_passing_math = '{0:.2f}%'.format(pct_passing_math)
pct_passing_reading = '{0:.2f}%'.format(pct_passing_reading)
passing_rate = '{0:.2f}%'.format(passing_rate)

# Create Summary Table
district_summary = pd.DataFrame({
    "Total Schools":[total_schools],
    "Total Students":[total_students],
    "Total Budget":[total_budget],
    "Average Math Score":[avg_math],
    "Average Reading Score":[avg_reading],
    "% Passing Math":[pct_passing_math],
    "% Passing Reading":[pct_passing_reading],
    "Overall Passing Rate":[passing_rate]
})

# Reset column order
district_summary = district_summary[["Total Schools",
                                     "Total Students",
                                     "Total Budget",
                                     "Average Math Score",
                                     "Average Reading Score",
                                     "% Passing Math",
                                     "% Passing Reading",
                                     "Overall Passing Rate"]]

district_summary
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
      <th>Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15</td>
      <td>39170</td>
      <td>$24,649,428</td>
      <td>78.99</td>
      <td>81.88</td>
      <td>74.98%</td>
      <td>85.81%</td>
      <td>80.39%</td>
    </tr>
  </tbody>
</table>
</div>



# School Summary


```python
# Create an overview table that summarizes key metrics about each school

student_by_school = df_students.groupby("school")

df_schools["Per Student Budget"] = df_schools["budget"] / df_schools["size"]

school_performance = pd.DataFrame({}) 
school_performance["Average Math Score"] = student_by_school["math_score"].mean()
school_performance["Average Reading Score"] = student_by_school["reading_score"].mean()

math_passers_by_school = students_passing_math.groupby("school")
math_passers_by_school_counts = math_passers_by_school.count()
school_performance["Number Passing Math"] = math_passers_by_school_counts["name"]

reading_passers_by_school = students_passing_reading.groupby("school")
reading_passers_by_school_counts = reading_passers_by_school.count()
school_performance["Number Passing Reading"] = reading_passers_by_school_counts["name"]

school_performance = school_performance.reset_index()
school_performance = school_performance.rename(columns={"school":"name"})

summary_by_school = pd.merge(df_schools, school_performance, on="name")
summary_by_school["% Passing Math"] = (summary_by_school["Number Passing Math"] / summary_by_school["size"])*100
summary_by_school["% Passing Reading"] = (summary_by_school["Number Passing Reading"] / summary_by_school["size"])*100
summary_by_school["% Overall Passing Rate"] = (summary_by_school["% Passing Math"] + summary_by_school["% Passing Reading"])/2

# Create summary table
school_overview = pd.DataFrame({
    "School Name":summary_by_school["name"],
    "School Type":summary_by_school["type"],
    "Total Students":summary_by_school["size"],
    "Total Budget":summary_by_school["budget"],
    "Per Student Budget":summary_by_school["Per Student Budget"],
    "Average Math Score":summary_by_school["Average Math Score"],
    "Average Reading Score":summary_by_school["Average Reading Score"],
    "% Passing Math":summary_by_school["% Passing Math"],
    "% Passing Reading":summary_by_school["% Passing Reading"],
    "% Overall Passing Rate":summary_by_school["% Overall Passing Rate"]    
})
school_overview = school_overview.set_index("School Name")

# Format items
school_overview["% Passing Math"] = school_overview["% Passing Math"].map('{0:.2f}%'.format)
school_overview["% Passing Reading"] = school_overview["% Passing Reading"].map('{0:.2f}%'.format)
school_overview["Average Math Score"] = school_overview["Average Math Score"].map('{0:.2f}'.format)
school_overview["Average Reading Score"] = school_overview["Average Reading Score"].map('{0:.2f}'.format)
school_overview["% Overall Passing Rate"] = school_overview["% Overall Passing Rate"].map('{0:.2f}%'.format)
school_overview["Per Student Budget"] = school_overview["Per Student Budget"].map('${:,.2f}'.format)
school_overview["Total Budget"] = school_overview["Total Budget"].map('${:,.0f}'.format)

# Reset column order
school_overview= school_overview[["School Type",
                                  "Total Students",
                                  "Total Budget",
                                  "Per Student Budget",
                                  "Average Math Score",
                                  "Average Reading Score",
                                  "% Passing Math",
                                  "% Passing Reading",
                                  "% Overall Passing Rate"]]
school_overview
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
      <th>Total Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School Name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2917</td>
      <td>$1,910,635</td>
      <td>$655.00</td>
      <td>76.63</td>
      <td>81.18</td>
      <td>65.68%</td>
      <td>81.32%</td>
      <td>73.50%</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2949</td>
      <td>$1,884,411</td>
      <td>$639.00</td>
      <td>76.71</td>
      <td>81.16</td>
      <td>65.99%</td>
      <td>80.74%</td>
      <td>73.36%</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>Charter</td>
      <td>1761</td>
      <td>$1,056,600</td>
      <td>$600.00</td>
      <td>83.36</td>
      <td>83.73</td>
      <td>93.87%</td>
      <td>95.85%</td>
      <td>94.86%</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>District</td>
      <td>4635</td>
      <td>$3,022,020</td>
      <td>$652.00</td>
      <td>77.29</td>
      <td>80.93</td>
      <td>66.75%</td>
      <td>80.86%</td>
      <td>73.81%</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>1468</td>
      <td>$917,500</td>
      <td>$625.00</td>
      <td>83.35</td>
      <td>83.82</td>
      <td>93.39%</td>
      <td>97.14%</td>
      <td>95.27%</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>2283</td>
      <td>$1,319,574</td>
      <td>$578.00</td>
      <td>83.27</td>
      <td>83.99</td>
      <td>93.87%</td>
      <td>96.54%</td>
      <td>95.20%</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>1858</td>
      <td>$1,081,356</td>
      <td>$582.00</td>
      <td>83.06</td>
      <td>83.98</td>
      <td>94.13%</td>
      <td>97.04%</td>
      <td>95.59%</td>
    </tr>
    <tr>
      <th>Bailey High School</th>
      <td>District</td>
      <td>4976</td>
      <td>$3,124,928</td>
      <td>$628.00</td>
      <td>77.05</td>
      <td>81.03</td>
      <td>66.68%</td>
      <td>81.93%</td>
      <td>74.31%</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>Charter</td>
      <td>427</td>
      <td>$248,087</td>
      <td>$581.00</td>
      <td>83.80</td>
      <td>83.81</td>
      <td>92.51%</td>
      <td>96.25%</td>
      <td>94.38%</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>962</td>
      <td>$585,858</td>
      <td>$609.00</td>
      <td>83.84</td>
      <td>84.04</td>
      <td>94.59%</td>
      <td>95.95%</td>
      <td>95.27%</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>Charter</td>
      <td>1800</td>
      <td>$1,049,400</td>
      <td>$583.00</td>
      <td>83.68</td>
      <td>83.95</td>
      <td>93.33%</td>
      <td>96.61%</td>
      <td>94.97%</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>3999</td>
      <td>$2,547,363</td>
      <td>$637.00</td>
      <td>76.84</td>
      <td>80.74</td>
      <td>66.37%</td>
      <td>80.22%</td>
      <td>73.29%</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>4761</td>
      <td>$3,094,650</td>
      <td>$650.00</td>
      <td>77.07</td>
      <td>80.97</td>
      <td>66.06%</td>
      <td>81.22%</td>
      <td>73.64%</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>District</td>
      <td>2739</td>
      <td>$1,763,916</td>
      <td>$644.00</td>
      <td>77.10</td>
      <td>80.75</td>
      <td>68.31%</td>
      <td>79.30%</td>
      <td>73.80%</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>Charter</td>
      <td>1635</td>
      <td>$1,043,130</td>
      <td>$638.00</td>
      <td>83.42</td>
      <td>83.85</td>
      <td>93.27%</td>
      <td>97.31%</td>
      <td>95.29%</td>
    </tr>
  </tbody>
</table>
</div>



# Top Performing Schools (By Passing Rate)


```python
# Create a table that highlights the top 5 performing schools based on Overall Passing Rate
top_performers = school_overview.sort_values("% Overall Passing Rate", ascending=False)
top_performers = top_performers.head(5)
top_performers
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
      <th>Total Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School Name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>1858</td>
      <td>$1,081,356</td>
      <td>$582.00</td>
      <td>83.06</td>
      <td>83.98</td>
      <td>94.13%</td>
      <td>97.04%</td>
      <td>95.59%</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>Charter</td>
      <td>1635</td>
      <td>$1,043,130</td>
      <td>$638.00</td>
      <td>83.42</td>
      <td>83.85</td>
      <td>93.27%</td>
      <td>97.31%</td>
      <td>95.29%</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>1468</td>
      <td>$917,500</td>
      <td>$625.00</td>
      <td>83.35</td>
      <td>83.82</td>
      <td>93.39%</td>
      <td>97.14%</td>
      <td>95.27%</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>962</td>
      <td>$585,858</td>
      <td>$609.00</td>
      <td>83.84</td>
      <td>84.04</td>
      <td>94.59%</td>
      <td>95.95%</td>
      <td>95.27%</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>2283</td>
      <td>$1,319,574</td>
      <td>$578.00</td>
      <td>83.27</td>
      <td>83.99</td>
      <td>93.87%</td>
      <td>96.54%</td>
      <td>95.20%</td>
    </tr>
  </tbody>
</table>
</div>



# Bottom Performing Schools (By Passing Rate)


```python
# Create a table that highlights the bottom 5 performing schools based on Overall Passing Rate
bottom_performers = school_overview.sort_values("% Overall Passing Rate", ascending=True)
bottom_performers = bottom_performers.head(5)
bottom_performers
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
      <th>Total Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School Name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>3999</td>
      <td>$2,547,363</td>
      <td>$637.00</td>
      <td>76.84</td>
      <td>80.74</td>
      <td>66.37%</td>
      <td>80.22%</td>
      <td>73.29%</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2949</td>
      <td>$1,884,411</td>
      <td>$639.00</td>
      <td>76.71</td>
      <td>81.16</td>
      <td>65.99%</td>
      <td>80.74%</td>
      <td>73.36%</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2917</td>
      <td>$1,910,635</td>
      <td>$655.00</td>
      <td>76.63</td>
      <td>81.18</td>
      <td>65.68%</td>
      <td>81.32%</td>
      <td>73.50%</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>4761</td>
      <td>$3,094,650</td>
      <td>$650.00</td>
      <td>77.07</td>
      <td>80.97</td>
      <td>66.06%</td>
      <td>81.22%</td>
      <td>73.64%</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>District</td>
      <td>2739</td>
      <td>$1,763,916</td>
      <td>$644.00</td>
      <td>77.10</td>
      <td>80.75</td>
      <td>68.31%</td>
      <td>79.30%</td>
      <td>73.80%</td>
    </tr>
  </tbody>
</table>
</div>



# Math Scores By Grade


```python
# Create a table that lists the average Math Score for students of each grade level (9th, 10th, 11th, 12th) at each school.

grade_9 = df_students.loc[df_students["grade"] == "9th",:]
grade_10 = df_students.loc[df_students["grade"] == "10th",:]
grade_11 = df_students.loc[df_students["grade"] == "11th",:]
grade_12 = df_students.loc[df_students["grade"] == "12th",:]

math_by_grade = pd.DataFrame({}) 

grade_9_by_school = grade_9.groupby("school")
grade_9_math = grade_9_by_school["math_score"].mean()
math_by_grade["9th"] = grade_9_math

grade_10_by_school = grade_10.groupby("school")
grade_10_math = grade_10_by_school["math_score"].mean()
math_by_grade["10th"] = grade_10_math

grade_11_by_school = grade_11.groupby("school")
grade_11_math = grade_11_by_school["math_score"].mean()
math_by_grade["11th"] = grade_11_math

grade_12_by_school = grade_12.groupby("school")
grade_12_math = grade_12_by_school["math_score"].mean()
math_by_grade["12th"] = grade_12_math

math_by_grade
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
    <tr>
      <th>school</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>77.083676</td>
      <td>76.996772</td>
      <td>77.515588</td>
      <td>76.492218</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.094697</td>
      <td>83.154506</td>
      <td>82.765560</td>
      <td>83.277487</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>76.403037</td>
      <td>76.539974</td>
      <td>76.884344</td>
      <td>77.151369</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>77.361345</td>
      <td>77.672316</td>
      <td>76.918058</td>
      <td>76.179963</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>82.044010</td>
      <td>84.229064</td>
      <td>83.842105</td>
      <td>83.356164</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>77.438495</td>
      <td>77.337408</td>
      <td>77.136029</td>
      <td>77.186567</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.787402</td>
      <td>83.429825</td>
      <td>85.000000</td>
      <td>82.855422</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>77.027251</td>
      <td>75.908735</td>
      <td>76.446602</td>
      <td>77.225641</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>77.187857</td>
      <td>76.691117</td>
      <td>77.491653</td>
      <td>76.863248</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.625455</td>
      <td>83.372000</td>
      <td>84.328125</td>
      <td>84.121547</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>76.859966</td>
      <td>76.612500</td>
      <td>76.395626</td>
      <td>77.690748</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>83.420755</td>
      <td>82.917411</td>
      <td>83.383495</td>
      <td>83.778976</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.590022</td>
      <td>83.087886</td>
      <td>83.498795</td>
      <td>83.497041</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.085578</td>
      <td>83.724422</td>
      <td>83.195326</td>
      <td>83.035794</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.264706</td>
      <td>84.010288</td>
      <td>83.836782</td>
      <td>83.644986</td>
    </tr>
  </tbody>
</table>
</div>



# Reading Scores By Grade


```python
# Create a table that lists the average Reading Score for students of each grade level (9th, 10th, 11th, 12th) at each school

reading_by_grade = pd.DataFrame({}) 

grade_9_reading = grade_9_by_school["reading_score"].mean()
reading_by_grade["9th"] = grade_9_reading

grade_10_reading = grade_10_by_school["reading_score"].mean()
reading_by_grade["10th"] = grade_10_reading

grade_11_reading = grade_11_by_school["reading_score"].mean()
reading_by_grade["11th"] = grade_11_reading

grade_12_reading = grade_12_by_school["reading_score"].mean()
reading_by_grade["12th"] = grade_12_reading

reading_by_grade
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
    <tr>
      <th>school</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>81.303155</td>
      <td>80.907183</td>
      <td>80.945643</td>
      <td>80.912451</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.676136</td>
      <td>84.253219</td>
      <td>83.788382</td>
      <td>84.287958</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>81.198598</td>
      <td>81.408912</td>
      <td>80.640339</td>
      <td>81.384863</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>80.632653</td>
      <td>81.262712</td>
      <td>80.403642</td>
      <td>80.662338</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>83.369193</td>
      <td>83.706897</td>
      <td>84.288089</td>
      <td>84.013699</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>80.866860</td>
      <td>80.660147</td>
      <td>81.396140</td>
      <td>80.857143</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.677165</td>
      <td>83.324561</td>
      <td>83.815534</td>
      <td>84.698795</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>81.290284</td>
      <td>81.512386</td>
      <td>81.417476</td>
      <td>80.305983</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>81.260714</td>
      <td>80.773431</td>
      <td>80.616027</td>
      <td>81.227564</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.807273</td>
      <td>83.612000</td>
      <td>84.335938</td>
      <td>84.591160</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>80.993127</td>
      <td>80.629808</td>
      <td>80.864811</td>
      <td>80.376426</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>84.122642</td>
      <td>83.441964</td>
      <td>84.373786</td>
      <td>82.781671</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.728850</td>
      <td>84.254157</td>
      <td>83.585542</td>
      <td>83.831361</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.939778</td>
      <td>84.021452</td>
      <td>83.764608</td>
      <td>84.317673</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.833333</td>
      <td>83.812757</td>
      <td>84.156322</td>
      <td>84.073171</td>
    </tr>
  </tbody>
</table>
</div>



# Scores By School Spending


```python
# Create a table that breaks down school performances based on average Spending Ranges (Per Student)

summary_by_school["Per Student Budget"] = summary_by_school["Per Student Budget"].map(int)

# Set bin sizes and names
bins = [0,585,615,645,1000]
group_names = ["<$585","$585-615","$620-640",">$640"]
summary_by_school["Spending Range (Per Student)"] = pd.cut(summary_by_school["Per Student Budget"], bins, labels=group_names)

schools_by_spending = summary_by_school.groupby("Spending Range (Per Student)")
summary_schools_by_spending = pd.DataFrame({}) 

# Build summary table
summary_schools_by_spending["Average Math Score"] = schools_by_spending["Average Math Score"].mean()
summary_schools_by_spending["Average Reading Score"] = schools_by_spending["Average Reading Score"].mean()
summary_schools_by_spending["% Passing Math"] = schools_by_spending["% Passing Math"].mean()
summary_schools_by_spending["% Passing Reading"] = schools_by_spending["% Passing Reading"].mean()
summary_schools_by_spending["% Overall Passing Rate"] = schools_by_spending["% Overall Passing Rate"].mean()

# Format summary table
summary_schools_by_spending["% Passing Math"] = summary_schools_by_spending["% Passing Math"].map('{0:.2f}%'.format)
summary_schools_by_spending["% Passing Reading"] = summary_schools_by_spending["% Passing Reading"].map('{0:.2f}%'.format)
summary_schools_by_spending["Average Math Score"] = summary_schools_by_spending["Average Math Score"].map('{0:.2f}'.format)
summary_schools_by_spending["Average Reading Score"] = summary_schools_by_spending["Average Reading Score"].map('{0:.2f}'.format)
summary_schools_by_spending["% Overall Passing Rate"] = summary_schools_by_spending["% Overall Passing Rate"].map('{0:.2f}%'.format)
summary_schools_by_spending
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
      <th>Spending Range (Per Student)</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>&lt;$585</th>
      <td>83.46</td>
      <td>83.93</td>
      <td>93.46%</td>
      <td>96.61%</td>
      <td>95.04%</td>
    </tr>
    <tr>
      <th>$585-615</th>
      <td>83.60</td>
      <td>83.89</td>
      <td>94.23%</td>
      <td>95.90%</td>
      <td>95.07%</td>
    </tr>
    <tr>
      <th>$620-640</th>
      <td>79.08</td>
      <td>81.89</td>
      <td>75.67%</td>
      <td>86.11%</td>
      <td>80.89%</td>
    </tr>
    <tr>
      <th>&gt;$640</th>
      <td>77.00</td>
      <td>81.03</td>
      <td>66.16%</td>
      <td>81.13%</td>
      <td>73.65%</td>
    </tr>
  </tbody>
</table>
</div>



# Scores By School Size


```python
# Repeat the above breakdown, but this time group schools based on a reasonable approximation of school size

summary_by_school["size"] = summary_by_school["size"].map(int)

# Set bin sizes and names
bins = [0, 1750, 3500, 5000]
group_names = ["Small (<1750)","Medium (1750-3500)","Large (>3500)"]
summary_by_school["School Size"] = pd.cut(summary_by_school["size"], bins, labels=group_names)

schools_by_size = summary_by_school.groupby("School Size")
summary_schools_by_size = pd.DataFrame({}) 

# Build summary table
summary_schools_by_size["Average Math Score"] = schools_by_size["Average Math Score"].mean()
summary_schools_by_size["Average Reading Score"] = schools_by_size["Average Reading Score"].mean()
summary_schools_by_size["% Passing Math"] = schools_by_size["% Passing Math"].mean()
summary_schools_by_size["% Passing Reading"] = schools_by_size["% Passing Reading"].mean()
summary_schools_by_size["% Overall Passing Rate"] = schools_by_size["% Overall Passing Rate"].mean()

# Format summary table
summary_schools_by_size["% Passing Math"] = summary_schools_by_size["% Passing Math"].map('{0:.2f}%'.format)
summary_schools_by_size["% Passing Reading"] = summary_schools_by_size["% Passing Reading"].map('{0:.2f}%'.format)
summary_schools_by_size["Average Math Score"] = summary_schools_by_size["Average Math Score"].map('{0:.2f}'.format)
summary_schools_by_size["Average Reading Score"] = summary_schools_by_size["Average Reading Score"].map('{0:.2f}'.format)
summary_schools_by_size["% Overall Passing Rate"] = summary_schools_by_size["% Overall Passing Rate"].map('{0:.2f}%'.format)
summary_schools_by_size
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
      <th>Small (&lt;1750)</th>
      <td>83.60</td>
      <td>83.88</td>
      <td>93.44%</td>
      <td>96.66%</td>
      <td>95.05%</td>
    </tr>
    <tr>
      <th>Medium (1750-3500)</th>
      <td>80.55</td>
      <td>82.68</td>
      <td>82.17%</td>
      <td>89.63%</td>
      <td>85.90%</td>
    </tr>
    <tr>
      <th>Large (&gt;3500)</th>
      <td>77.06</td>
      <td>80.92</td>
      <td>66.46%</td>
      <td>81.06%</td>
      <td>73.76%</td>
    </tr>
  </tbody>
</table>
</div>



# Scores By School Type


```python
# Repeat the above breakdown, but this time group schools based on school type (Charter vs. District).

district_schools = summary_by_school.loc[summary_by_school["type"] == "District", :]
charter_schools = summary_by_school.loc[summary_by_school["type"] == "Charter", :]

# Build summary table
summary_schools_by_type = pd.DataFrame({
    "School Type":["District Schools","Charter Schools"],
    "Average Math Score":[district_schools["Average Math Score"].mean(),charter_schools["Average Math Score"].mean()],
    "Average Reading Score":[district_schools["Average Reading Score"].mean(),charter_schools["Average Reading Score"].mean()],
    "% Passing Math":[district_schools["% Passing Math"].mean(),charter_schools["% Passing Math"].mean()],
    "% Passing Reading":[district_schools["% Passing Reading"].mean(),charter_schools["% Passing Reading"].mean()],
    "% Overall Passing Rate":[district_schools["% Overall Passing Rate"].mean(),charter_schools["% Overall Passing Rate"].mean()]
})

# Set index and order columns
summary_schools_by_type = summary_schools_by_type.set_index("School Type")
summary_schools_by_type = summary_schools_by_type[["Average Math Score",
                                  "Average Reading Score",
                                  "% Passing Math",
                                  "% Passing Reading",
                                  "% Overall Passing Rate"]]

# Format summary table
summary_schools_by_type["% Passing Math"] = summary_schools_by_type["% Passing Math"].map('{0:.2f}%'.format)
summary_schools_by_type["% Passing Reading"] = summary_schools_by_type["% Passing Reading"].map('{0:.2f}%'.format)
summary_schools_by_type["Average Math Score"] = summary_schools_by_type["Average Math Score"].map('{0:.2f}'.format)
summary_schools_by_type["Average Reading Score"] = summary_schools_by_type["Average Reading Score"].map('{0:.2f}'.format)
summary_schools_by_type["% Overall Passing Rate"] = summary_schools_by_type["% Overall Passing Rate"].map('{0:.2f}%'.format)

summary_schools_by_type
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
      <th>District Schools</th>
      <td>76.96</td>
      <td>80.97</td>
      <td>66.55%</td>
      <td>80.80%</td>
      <td>73.67%</td>
    </tr>
    <tr>
      <th>Charter Schools</th>
      <td>83.47</td>
      <td>83.90</td>
      <td>93.62%</td>
      <td>96.59%</td>
      <td>95.10%</td>
    </tr>
  </tbody>
</table>
</div>


