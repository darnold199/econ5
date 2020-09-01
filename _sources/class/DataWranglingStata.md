# Data Wrangling in STATA

## Introduction and Overview

One of the most powerful and attractive features of STATA is how easy it is to construct and clean data. By the end of the notebook you will earn how to append datasets, merge datasets, reshape datasets and collapse datasets. 

### Appending data

Appending data is a simple way combine observations from two different datasets. In many datasets you will come across in the wild, the data are often stored in multiple files. For example, a file for each year or a file for each state, but when we go to data analysis, it will generally be convenient to have all the data together in the same file. For example, in the simple example given below, we have two datasets, each from a separate school. We can construct a new dataset that combines the school-specific datasets by using the ``append`` command in STATA.

<img src="img/append.png" width="800">

<!---To append two datasets you need to be careful about being consistent with variable names across datasets. For example, if dataset 1 had named the GPA variable ``GPA`` and dataset 2 had named it ``grade_point_average``, then both variables will appear in the appended dataset, but GPA will be missing for ever student in dataset 2.--> 

### Merging data

A similar, but slightly different scenario is when we want to ``merge`` two datasets. We do this when we have two datasets that have a common unit of observation, but potentially different variables. For example, in the simple example below, dataset 1 is a datset of students and their assigned school, while dataset 2 is a dataset of students and their grades. The goal of the ``merge`` command is to create a single dataset with student identifiers, school identifiers, and student grades:

<img src="img/merge_one.png" width="800">

### Reshaping Data

Often we have multiple observations for the same unit (for example, a GPA for a student each term). It is sometimes convenient to reshape the data so that we have a single observation per student. When we have multiple observations per unit, then our data is in ``long`` forma. If we have a single observation per unit, then we are in ``wide`` format. In STATA we can freely convert between the two:

<img src="img/reshape.png" width="800">

The syntax for reshaping can sometimes be hard to remember so don't forget to reference the ``help`` file if you find yourself forgetting how to reshape the data. 

## Application -- Opioid Prescriptions in the United States

Beginning in the 1990s up until today, opioid usage has been skyrocketing in the United States. In this application we are going to look at opioid usage across U.S. counties. We will be particularly interested in understanding what factors, such as labor-market conditions, may be associated with opioid usage.

The first dataset we will look at is named ``opioid_2006.dta`` and comes from the CDC [source](https://www.cdc.gov/drugoverdose/maps/rxrate-maps.html). The data reports the prescription rate for a large portion of U.S. counties. The prescription rate will be defined as the number of prescriptions (per year) per 100 people. So a prescription rate of 50 implies that in that year, there were 50 opioid prescriptions per 100 people. Because a single person can have multiple prescriptions per year, this rate is not equal to the fraction of people using opioids per year. That will be even clearer once we load the data.


```stata
use ../data/opioids_2006.dta, replace

describe
```

    
    
    
    Contains data from ../data/opioids_2006.dta
      obs:         2,733                          
     vars:             4                          19 Aug 2020 14:20
    --------------------------------------------------------------------------------
                  storage   display    value
    variable name   type    format     label      variable label
    --------------------------------------------------------------------------------
    county          str25   %25s                  County
    state           str5    %9s                   State
    fips_county     str16   %16s                  Numeric FIPS code for County
    prescrip_rate   double  %10.0g                Prescriptions per 100 persons
    --------------------------------------------------------------------------------
    Sorted by: fips_county


As mentioned above our key variable of interest will be ``prescription_rate`` which gives the 


```stata
%head
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
      <th>county</th>
      <th>state</th>
      <th>fips_county</th>
      <th>prescrip_rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>Autauga</td>
      <td>AL</td>
      <td>01001</td>
      <td>134.8</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Baldwin</td>
      <td>AL</td>
      <td>01003</td>
      <td>127.8</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Barbour</td>
      <td>AL</td>
      <td>01005</td>
      <td>78.09999999999999</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Bibb</td>
      <td>AL</td>
      <td>01007</td>
      <td>114.3</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Blount</td>
      <td>AL</td>
      <td>01009</td>
      <td>40.3</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Bullock</td>
      <td>AL</td>
      <td>01011</td>
      <td>34.6</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Butler</td>
      <td>AL</td>
      <td>01013</td>
      <td>105.4</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Calhoun</td>
      <td>AL</td>
      <td>01015</td>
      <td>165.7</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Chambers</td>
      <td>AL</td>
      <td>01017</td>
      <td>120.4</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Cherokee</td>
      <td>AL</td>
      <td>01019</td>
      <td>123.8</td>
    </tr>
  </tbody>
</table>
</div>


So looking at the data some of these numbers are already quite suprising. A ``prescip_rate`` over 100 implies that for every person in the county, there was at least one opioid prescription (this does not imply everyone is using opioids, many people are prescribed more than one prescription per year).

Now this is only data for 2006, but we want to see how prescription rates have evolved over time. Let's look at a decade later, in 2016. To keep track of the year we need to create a new variable:


```stata
gen year = 2006
```

Now we can ``append`` the 2016 dataset so that we have all the data together.


```stata
append using ../data/opioids_2016.dta
```

Now the dataset should include both the 2006 prescription rates as well as the 2016 prescription rates. To make sure we have everything here, let's count the number of observations


```stata
count
```

      5,466


Which is double the number we had before. One thing we need to do now before continuing is to fill in the variable ``year`` for the 2016 data. First, we can see that it is missing by typing


```stata
tab year 
```

    
           year |      Freq.     Percent        Cum.
    ------------+-----------------------------------
           2006 |      2,733      100.00      100.00
    ------------+-----------------------------------
          Total |      2,733      100.00


This reveals that year is only included for the 2006 observations, but missing for the 2,733 2016 observations. So let's fill in that variable 


```stata
replace year = 2016 if year==.
```

    (2,733 real changes made)


And let's check just to make sure that the command worked how we think it should.


```stata
tab year
```

    
           year |      Freq.     Percent        Cum.
    ------------+-----------------------------------
           2006 |      2,733       50.00       50.00
           2016 |      2,733       50.00      100.00
    ------------+-----------------------------------
          Total |      5,466      100.00


Ok so now we can start exploring prescription rates for opioid usage. Let's first look at the mean prescription rate in both years.


```stata
tabulate year, summarize(prescrip_rate)
```

    
                |  Summary of Prescriptions per 100
                |               persons
           year |        Mean   Std. Dev.       Freq.
    ------------+------------------------------------
           2006 |   80.658946   43.230614       2,733
           2016 |   80.098683   40.224893       2,733
    ------------+------------------------------------
          Total |   80.378814   41.751928       5,466


So, on average, opioid prescription rates are relatively similar between 2006 to 2016. This finding suprised me initially, given the media coverage of the opioid epidemic has only been relatively recent, and so I had assumed that prescription rates had been steadily increasing throughout the 2000s. After reading into this a little more, though, I discovered, that although prescriptions rates have been declining since 2012, illicit usage of close substitutes of prescription opioids, such as heroin, has been on the rise and fueling the opioid epidemic. So while legal prescriptions have been declining in recent years, overdoses due to illicit opioids has been on the rise.  

Next, we want to understand variation across places, and what factors may be associated with high prescription rates. To simplify things for now, I am going to now restrict once again to the 2006 data.


```stata
keep if year==2006
```

    (2,733 observations deleted)


Let's look at a histogram of the prescription rates across counties:


```stata
graph twoway histogram prescrip_rate, color(gs12) ///
    title("Distribution of Opioid Prescription Rates Across Counties") ///
    xtitle("Prescription Rate") ///
    ytitle("Density") ///
    note("Source: Center for Disease Control") ///
    legend(off) ///
    graphregion(color(white) fcolor(white)) 

```


                <iframe frameborder="0" scrolling="no" height="436" width="600"                srcdoc="<html><body>&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot; standalone=&quot;no&quot;?&gt;
&lt;!-- This is a Stata 16.1 generated SVG file (http://www.stata.com) --&gt;

&lt;svg version=&quot;1.1&quot; width=&quot;600px&quot; height=&quot;436px&quot; viewBox=&quot;0 0 3960 2880&quot; xmlns=&quot;http://www.w3.org/2000/svg&quot; xmlns:xlink=&quot;http://www.w3.org/1999/xlink&quot;&gt;
	&lt;desc&gt;Stata Graph - Graph&lt;/desc&gt;
	&lt;rect x=&quot;0&quot; y=&quot;0&quot; width=&quot;3960&quot; height=&quot;2880&quot; style=&quot;fill:#EAF2F3;stroke:none&quot;/&gt;
	&lt;rect x=&quot;0.00&quot; y=&quot;0.00&quot; width=&quot;3959.88&quot; height=&quot;2880.00&quot; style=&quot;fill:#FFFFFF&quot;/&gt;
	&lt;rect x=&quot;2.88&quot; y=&quot;2.88&quot; width=&quot;3954.12&quot; height=&quot;2874.24&quot; style=&quot;fill:none;stroke:#FFFFFF;stroke-width:5.76&quot;/&gt;
	&lt;rect x=&quot;374.10&quot; y=&quot;275.35&quot; width=&quot;3484.92&quot; height=&quot;2099.36&quot; style=&quot;fill:#FFFFFF&quot;/&gt;
	&lt;rect x=&quot;376.98&quot; y=&quot;278.23&quot; width=&quot;3479.16&quot; height=&quot;2093.60&quot; style=&quot;fill:none;stroke:#FFFFFF;stroke-width:5.76&quot;/&gt;
	&lt;line x1=&quot;374.10&quot; y1=&quot;2311.35&quot; x2=&quot;3859.02&quot; y2=&quot;2311.35&quot; style=&quot;stroke:#EAF2F3;stroke-width:8.64&quot;/&gt;
	&lt;line x1=&quot;374.10&quot; y1=&quot;1653.72&quot; x2=&quot;3859.02&quot; y2=&quot;1653.72&quot; style=&quot;stroke:#EAF2F3;stroke-width:8.64&quot;/&gt;
	&lt;line x1=&quot;374.10&quot; y1=&quot;996.22&quot; x2=&quot;3859.02&quot; y2=&quot;996.22&quot; style=&quot;stroke:#EAF2F3;stroke-width:8.64&quot;/&gt;
	&lt;rect x=&quot;438.32&quot; y=&quot;2140.94&quot; width=&quot;98.75&quot; height=&quot;170.41&quot; style=&quot;fill:#C0C0C0&quot;/&gt;
	&lt;rect x=&quot;442.64&quot; y=&quot;2145.26&quot; width=&quot;90.11&quot; height=&quot;161.77&quot; style=&quot;fill:none;stroke:#C0C0C0;stroke-width:8.64&quot;/&gt;
	&lt;rect x=&quot;537.08&quot; y=&quot;1942.07&quot; width=&quot;98.75&quot; height=&quot;369.28&quot; style=&quot;fill:#C0C0C0&quot;/&gt;
	&lt;rect x=&quot;541.39&quot; y=&quot;1946.39&quot; width=&quot;90.11&quot; height=&quot;360.64&quot; style=&quot;fill:none;stroke:#C0C0C0;stroke-width:8.64&quot;/&gt;
	&lt;rect x=&quot;635.83&quot; y=&quot;1544.45&quot; width=&quot;98.75&quot; height=&quot;766.90&quot; style=&quot;fill:#C0C0C0&quot;/&gt;
	&lt;rect x=&quot;640.15&quot; y=&quot;1548.77&quot; width=&quot;90.11&quot; height=&quot;758.26&quot; style=&quot;fill:none;stroke:#C0C0C0;stroke-width:8.64&quot;/&gt;
	&lt;rect x=&quot;734.58&quot; y=&quot;1365.87&quot; width=&quot;98.75&quot; height=&quot;945.48&quot; style=&quot;fill:#C0C0C0&quot;/&gt;
	&lt;rect x=&quot;738.90&quot; y=&quot;1370.19&quot; width=&quot;90.11&quot; height=&quot;936.84&quot; style=&quot;fill:none;stroke:#C0C0C0;stroke-width:8.64&quot;/&gt;
	&lt;rect x=&quot;833.33&quot; y=&quot;992.63&quot; width=&quot;98.75&quot; height=&quot;1318.72&quot; style=&quot;fill:#C0C0C0&quot;/&gt;
	&lt;rect x=&quot;837.65&quot; y=&quot;996.95&quot; width=&quot;90.11&quot; height=&quot;1310.08&quot; style=&quot;fill:none;stroke:#C0C0C0;stroke-width:8.64&quot;/&gt;
	&lt;rect x=&quot;932.09&quot; y=&quot;785.71&quot; width=&quot;98.75&quot; height=&quot;1525.64&quot; style=&quot;fill:#C0C0C0&quot;/&gt;
	&lt;rect x=&quot;936.40&quot; y=&quot;790.03&quot; width=&quot;90.11&quot; height=&quot;1517.00&quot; style=&quot;fill:none;stroke:#C0C0C0;stroke-width:8.64&quot;/&gt;
	&lt;rect x=&quot;1030.84&quot; y=&quot;741.04&quot; width=&quot;98.63&quot; height=&quot;1570.31&quot; style=&quot;fill:#C0C0C0&quot;/&gt;
	&lt;rect x=&quot;1035.16&quot; y=&quot;745.36&quot; width=&quot;89.99&quot; height=&quot;1561.67&quot; style=&quot;fill:none;stroke:#C0C0C0;stroke-width:8.64&quot;/&gt;
	&lt;rect x=&quot;1129.47&quot; y=&quot;1154.87&quot; width=&quot;98.75&quot; height=&quot;1156.48&quot; style=&quot;fill:#C0C0C0&quot;/&gt;
	&lt;rect x=&quot;1133.79&quot; y=&quot;1159.19&quot; width=&quot;90.11&quot; height=&quot;1147.84&quot; style=&quot;fill:none;stroke:#C0C0C0;stroke-width:8.64&quot;/&gt;
	&lt;rect x=&quot;1228.22&quot; y=&quot;1410.55&quot; width=&quot;98.75&quot; height=&quot;900.80&quot; style=&quot;fill:#C0C0C0&quot;/&gt;
	&lt;rect x=&quot;1232.54&quot; y=&quot;1414.87&quot; width=&quot;90.11&quot; height=&quot;892.16&quot; style=&quot;fill:none;stroke:#C0C0C0;stroke-width:8.64&quot;/&gt;
	&lt;rect x=&quot;1326.97&quot; y=&quot;1629.59&quot; width=&quot;98.75&quot; height=&quot;681.76&quot; style=&quot;fill:#C0C0C0&quot;/&gt;
	&lt;rect x=&quot;1331.29&quot; y=&quot;1633.91&quot; width=&quot;90.11&quot; height=&quot;673.12&quot; style=&quot;fill:none;stroke:#C0C0C0;stroke-width:8.64&quot;/&gt;
	&lt;rect x=&quot;1425.72&quot; y=&quot;1791.96&quot; width=&quot;98.75&quot; height=&quot;519.39&quot; style=&quot;fill:#C0C0C0&quot;/&gt;
	&lt;rect x=&quot;1430.04&quot; y=&quot;1796.28&quot; width=&quot;90.11&quot; height=&quot;510.76&quot; style=&quot;fill:none;stroke:#C0C0C0;stroke-width:8.64&quot;/&gt;
	&lt;rect x=&quot;1524.48&quot; y=&quot;1942.07&quot; width=&quot;98.75&quot; height=&quot;369.28&quot; style=&quot;fill:#C0C0C0&quot;/&gt;
	&lt;rect x=&quot;1528.80&quot; y=&quot;1946.39&quot; width=&quot;90.11&quot; height=&quot;360.64&quot; style=&quot;fill:none;stroke:#C0C0C0;stroke-width:8.64&quot;/&gt;
	&lt;rect x=&quot;1623.23&quot; y=&quot;2092.18&quot; width=&quot;98.75&quot; height=&quot;219.17&quot; style=&quot;fill:#C0C0C0&quot;/&gt;
	&lt;rect x=&quot;1627.55&quot; y=&quot;2096.50&quot; width=&quot;90.11&quot; height=&quot;210.53&quot; style=&quot;fill:none;stroke:#C0C0C0;stroke-width:8.64&quot;/&gt;
	&lt;rect x=&quot;1721.98&quot; y=&quot;2169.28&quot; width=&quot;98.75&quot; height=&quot;142.07&quot; style=&quot;fill:#C0C0C0&quot;/&gt;
	&lt;rect x=&quot;1726.30&quot; y=&quot;2173.60&quot; width=&quot;90.11&quot; height=&quot;133.43&quot; style=&quot;fill:none;stroke:#C0C0C0;stroke-width:8.64&quot;/&gt;
	&lt;rect x=&quot;1820.73&quot; y=&quot;2201.71&quot; width=&quot;98.75&quot; height=&quot;109.65&quot; style=&quot;fill:#C0C0C0&quot;/&gt;
	&lt;rect x=&quot;1825.05&quot; y=&quot;2206.02&quot; width=&quot;90.11&quot; height=&quot;101.01&quot; style=&quot;fill:none;stroke:#C0C0C0;stroke-width:8.64&quot;/&gt;
	&lt;rect x=&quot;1919.49&quot; y=&quot;2234.25&quot; width=&quot;98.75&quot; height=&quot;77.10&quot; style=&quot;fill:#C0C0C0&quot;/&gt;
	&lt;rect x=&quot;1923.81&quot; y=&quot;2238.57&quot; width=&quot;90.11&quot; height=&quot;68.46&quot; style=&quot;fill:none;stroke:#C0C0C0;stroke-width:8.64&quot;/&gt;
	&lt;rect x=&quot;2018.24&quot; y=&quot;2250.46&quot; width=&quot;98.75&quot; height=&quot;60.89&quot; style=&quot;fill:#C0C0C0&quot;/&gt;
	&lt;rect x=&quot;2022.56&quot; y=&quot;2254.78&quot; width=&quot;90.11&quot; height=&quot;52.25&quot; style=&quot;fill:none;stroke:#C0C0C0;stroke-width:8.64&quot;/&gt;
	&lt;rect x=&quot;2116.99&quot; y=&quot;2270.76&quot; width=&quot;98.75&quot; height=&quot;40.59&quot; style=&quot;fill:#C0C0C0&quot;/&gt;
	&lt;rect x=&quot;2121.31&quot; y=&quot;2275.08&quot; width=&quot;90.11&quot; height=&quot;31.95&quot; style=&quot;fill:none;stroke:#C0C0C0;stroke-width:8.64&quot;/&gt;
	&lt;rect x=&quot;2215.74&quot; y=&quot;2262.59&quot; width=&quot;98.75&quot; height=&quot;48.76&quot; style=&quot;fill:#C0C0C0&quot;/&gt;
	&lt;rect x=&quot;2220.06&quot; y=&quot;2266.91&quot; width=&quot;90.11&quot; height=&quot;40.12&quot; style=&quot;fill:none;stroke:#C0C0C0;stroke-width:8.64&quot;/&gt;
	&lt;rect x=&quot;2314.50&quot; y=&quot;2307.27&quot; width=&quot;98.75&quot; height=&quot;4.08&quot; style=&quot;fill:#C0C0C0&quot;/&gt;
	&lt;rect x=&quot;2318.82&quot; y=&quot;2307.03&quot; width=&quot;90.11&quot; height=&quot;4.56&quot; style=&quot;fill:none;stroke:#C0C0C0;stroke-width:8.64&quot;/&gt;
	&lt;rect x=&quot;2413.25&quot; y=&quot;2295.14&quot; width=&quot;98.75&quot; height=&quot;16.21&quot; style=&quot;fill:#C0C0C0&quot;/&gt;
	&lt;rect x=&quot;2417.57&quot; y=&quot;2299.46&quot; width=&quot;90.11&quot; height=&quot;7.57&quot; style=&quot;fill:none;stroke:#C0C0C0;stroke-width:8.64&quot;/&gt;
	&lt;rect x=&quot;2512.00&quot; y=&quot;2282.89&quot; width=&quot;98.75&quot; height=&quot;28.46&quot; style=&quot;fill:#C0C0C0&quot;/&gt;
	&lt;rect x=&quot;2516.32&quot; y=&quot;2287.21&quot; width=&quot;90.11&quot; height=&quot;19.82&quot; style=&quot;fill:none;stroke:#C0C0C0;stroke-width:8.64&quot;/&gt;
	&lt;rect x=&quot;2610.75&quot; y=&quot;2303.18&quot; width=&quot;98.75&quot; height=&quot;8.17&quot; style=&quot;fill:#C0C0C0&quot;/&gt;
	&lt;rect x=&quot;2615.07&quot; y=&quot;2307.03&quot; width=&quot;90.11&quot; height=&quot;0.47&quot; style=&quot;fill:none;stroke:#C0C0C0;stroke-width:8.64&quot;/&gt;
	&lt;rect x=&quot;2709.51&quot; y=&quot;2303.18&quot; width=&quot;98.75&quot; height=&quot;8.17&quot; style=&quot;fill:#C0C0C0&quot;/&gt;
	&lt;rect x=&quot;2713.83&quot; y=&quot;2307.03&quot; width=&quot;90.11&quot; height=&quot;0.47&quot; style=&quot;fill:none;stroke:#C0C0C0;stroke-width:8.64&quot;/&gt;
	&lt;rect x=&quot;2907.01&quot; y=&quot;2299.10&quot; width=&quot;98.75&quot; height=&quot;12.25&quot; style=&quot;fill:#C0C0C0&quot;/&gt;
	&lt;rect x=&quot;2911.33&quot; y=&quot;2303.42&quot; width=&quot;90.11&quot; height=&quot;3.61&quot; style=&quot;fill:none;stroke:#C0C0C0;stroke-width:8.64&quot;/&gt;
	&lt;rect x=&quot;3302.02&quot; y=&quot;2303.18&quot; width=&quot;98.75&quot; height=&quot;8.17&quot; style=&quot;fill:#C0C0C0&quot;/&gt;
	&lt;rect x=&quot;3306.34&quot; y=&quot;2307.03&quot; width=&quot;90.11&quot; height=&quot;0.47&quot; style=&quot;fill:none;stroke:#C0C0C0;stroke-width:8.64&quot;/&gt;
	&lt;rect x=&quot;3598.28&quot; y=&quot;2303.18&quot; width=&quot;98.75&quot; height=&quot;8.17&quot; style=&quot;fill:#C0C0C0&quot;/&gt;
	&lt;rect x=&quot;3602.60&quot; y=&quot;2307.03&quot; width=&quot;90.11&quot; height=&quot;0.47&quot; style=&quot;fill:none;stroke:#C0C0C0;stroke-width:8.64&quot;/&gt;
	&lt;rect x=&quot;3697.03&quot; y=&quot;2307.27&quot; width=&quot;98.63&quot; height=&quot;4.08&quot; style=&quot;fill:#C0C0C0&quot;/&gt;
	&lt;rect x=&quot;3701.35&quot; y=&quot;2307.03&quot; width=&quot;89.99&quot; height=&quot;4.56&quot; style=&quot;fill:none;stroke:#C0C0C0;stroke-width:8.64&quot;/&gt;
	&lt;line x1=&quot;374.10&quot; y1=&quot;2374.71&quot; x2=&quot;374.10&quot; y2=&quot;275.35&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;line x1=&quot;374.10&quot; y1=&quot;2311.35&quot; x2=&quot;334.12&quot; y2=&quot;2311.35&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;text x=&quot;284.02&quot; y=&quot;2311.35&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; transform=&quot;rotate(-90 284.02,2311.35)&quot; text-anchor=&quot;middle&quot;&gt;0&lt;/text&gt;
	&lt;line x1=&quot;374.10&quot; y1=&quot;1653.72&quot; x2=&quot;334.12&quot; y2=&quot;1653.72&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;text x=&quot;284.02&quot; y=&quot;1653.72&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; transform=&quot;rotate(-90 284.02,1653.72)&quot; text-anchor=&quot;middle&quot;&gt;.005&lt;/text&gt;
	&lt;line x1=&quot;374.10&quot; y1=&quot;996.22&quot; x2=&quot;334.12&quot; y2=&quot;996.22&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;text x=&quot;284.02&quot; y=&quot;996.22&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; transform=&quot;rotate(-90 284.02,996.22)&quot; text-anchor=&quot;middle&quot;&gt;.01&lt;/text&gt;
	&lt;line x1=&quot;374.10&quot; y1=&quot;338.71&quot; x2=&quot;334.12&quot; y2=&quot;338.71&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;text x=&quot;284.02&quot; y=&quot;338.71&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; transform=&quot;rotate(-90 284.02,338.71)&quot; text-anchor=&quot;middle&quot;&gt;.015&lt;/text&gt;
	&lt;text x=&quot;190.71&quot; y=&quot;1325.03&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; transform=&quot;rotate(-90 190.71,1325.03)&quot; text-anchor=&quot;middle&quot;&gt;Density&lt;/text&gt;
	&lt;line x1=&quot;374.10&quot; y1=&quot;2374.71&quot; x2=&quot;3859.02&quot; y2=&quot;2374.71&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;line x1=&quot;437.46&quot; y1=&quot;2374.71&quot; x2=&quot;437.46&quot; y2=&quot;2414.69&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;text x=&quot;437.46&quot; y=&quot;2504.54&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; text-anchor=&quot;middle&quot;&gt;0&lt;/text&gt;
	&lt;line x1=&quot;1270.17&quot; y1=&quot;2374.71&quot; x2=&quot;1270.17&quot; y2=&quot;2414.69&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;text x=&quot;1270.17&quot; y=&quot;2504.54&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; text-anchor=&quot;middle&quot;&gt;100&lt;/text&gt;
	&lt;line x1=&quot;2102.88&quot; y1=&quot;2374.71&quot; x2=&quot;2102.88&quot; y2=&quot;2414.69&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;text x=&quot;2102.88&quot; y=&quot;2504.54&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; text-anchor=&quot;middle&quot;&gt;200&lt;/text&gt;
	&lt;line x1=&quot;2935.60&quot; y1=&quot;2374.71&quot; x2=&quot;2935.60&quot; y2=&quot;2414.69&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;text x=&quot;2935.60&quot; y=&quot;2504.54&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; text-anchor=&quot;middle&quot;&gt;300&lt;/text&gt;
	&lt;line x1=&quot;3768.19&quot; y1=&quot;2374.71&quot; x2=&quot;3768.19&quot; y2=&quot;2414.69&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;text x=&quot;3768.19&quot; y=&quot;2504.54&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; text-anchor=&quot;middle&quot;&gt;400&lt;/text&gt;
	&lt;text x=&quot;2116.62&quot; y=&quot;2614.56&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; text-anchor=&quot;middle&quot;&gt;Prescription Rate&lt;/text&gt;
	&lt;text x=&quot;391.42&quot; y=&quot;2737.89&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:79.94px;fill:#000000&quot;&gt;Source: Center for Disease Control&lt;/text&gt;
	&lt;text x=&quot;2116.62&quot; y=&quot;215.98&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:139.96px;fill:#1E2D53&quot; text-anchor=&quot;middle&quot;&gt;Distribution of Opioid Prescription Rates Across Counties&lt;/text&gt;
&lt;/svg&gt;
</body></html>"></iframe>



To me, this looks like incredible variation across location. Some counties have basically zero prescriptions per person, while others have over 2 prescriptions per person. Understanding what drives this variation across locations is an incredibly important question. If we want to develop policies aimed at preventing opioid overdoses, we need to understand what is fueling the epidemic.

One hypothesis is that poor labor-market conditions are contributing to the epidemic. The logic being that workers who are laid off are more likely to abuse opioids. The causality could also work in the opposite direction. Individuals who are addicted to opioids may have a harder time finding a job, leading to higher unemployment. Providing a clear causal relationship from unemployment to opioid usage (or vice-versa) is a difficult and nuanced problem. In this application, however, we are going to look at whether there appears to be a correlation between the two, while being careful to not over-interpret any relationship we find.

However, to do this, we need data on unemployment rates. These data will come from the Bureau of Labor Statistics and will be named ``urate_2006.dta``. Now you can see the power of linking datasets. Very few datasets have both health and labor-market outcomes, but by combining datasets we can study a new questions. Let's look at ``urate_2006.dta`` so that we can try to understand how we can merge it to ``opioids_2006.dta``


```stata
use ../data/urate_2006.dta, replace
d
```

    
    
    
    Contains data from ../data/urate_2006.dta
      obs:         2,733                          
     vars:             4                          19 Aug 2020 14:20
    --------------------------------------------------------------------------------
                  storage   display    value
    variable name   type    format     label      variable label
    --------------------------------------------------------------------------------
    county          str47   %47s                  County
    labor_force     str14   %14s                  Total number of Workers in the
                                                    Labor Force
    urate           double  %10.0g                Unemployment Rate
    fips_county     str16   %16s                  Numeric FIPS code for County
    --------------------------------------------------------------------------------
    Sorted by: fips_county


If you recall, we had a variable called ``fips_county`` in the opioid dataset as well. We will be merging the dataset on this variable. You need to make sure in general that the variable you are matching on is standardized across datasets. For these datasets, I found that the FIPS variables were stored in slightly different ways across datasets. For example, the FIPS code for the first county "Autauga County" was stored as "01001" in one of the datasets but "1001" where the difference is the leading zero. So I had to add the zero to the latter variable so that the codes would match properly. If you don't make sure the merge variable is correct your analysis will be completely thrown off.

So now let's merge these two datsets using the ``merge`` command:


```stata
merge 1:1 fips_county using ../data/opioids_2006.dta
```

    
        Result                           # of obs.
        -----------------------------------------
        not matched                             0
        matched                             2,733  (_merge==3)
        -----------------------------------------


The merge command will always display a table like this as well as generate a variable called ``_merge`` (unless you use an option not to report or generate this variabl). In this example, I already made sure that the counties perfectly match between the opioids and the unemployment datasets. Therefore we got ``_merge==3`` for all the counties, because all the counties matched. Just to show you what happens, I'm going to reload the unemployment dataset, drop 1 observation, and then merge the two datasets again


```stata
use ../data/urate_2006.dta, replace

// drop 1st observation
drop if _n==1

// merge to opioid dataset
merge 1:1 fips_county using ../data/opioids_2006.dta

```

    
    
    (1 observation deleted)
    
    
        Result                           # of obs.
        -----------------------------------------
        not matched                             1
            from master                         0  (_merge==1)
            from using                          1  (_merge==2)
    
        matched                             2,732  (_merge==3)
        -----------------------------------------


Now we see that 2,732 of the counties in urate_2006.dta matched to a county in the opioids_2006 dataset. However, there was one county in the **using** dataset that did not match, by ``_merge==2`` in this table. In STATA, the **master** dataset is the dataset in memory before you apply the merge command. The **using** dataset is the dataset you merge to. So in this example, we loaded urate_2006.dta, making it the **master** dataset, and then we merged ``using`` opioids_2006.dta, making opioids_2006 the **using** dataset. Now let's get back to application at hand and see how opioid prescription rates relate to unemployment rates. The most transparent way to start exploring is to just plot all the data in a scatter plot and see if we can see any patterns.


```stata
use ../data/urate_2006.dta, replace
merge 1:1 fips_county using ../data/opioids_2006.dta, nogen noreport
```


```stata
twoway scatter prescrip_rate urate, msymbol(circle_hollow) msize(small) ///
    title("Relation Between Unemployment and Opioids") ///
    xtitle("Unemployment Rate") ///
    ytitle("Prescriptions Per 100 People") ///
    graphregion(color(white) fcolor(white)) 
```


                <iframe frameborder="0" scrolling="no" height="436" width="600"                srcdoc="<html><body>&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot; standalone=&quot;no&quot;?&gt;
&lt;!-- This is a Stata 16.1 generated SVG file (http://www.stata.com) --&gt;

&lt;svg version=&quot;1.1&quot; width=&quot;600px&quot; height=&quot;436px&quot; viewBox=&quot;0 0 3960 2880&quot; xmlns=&quot;http://www.w3.org/2000/svg&quot; xmlns:xlink=&quot;http://www.w3.org/1999/xlink&quot;&gt;
	&lt;desc&gt;Stata Graph - Graph&lt;/desc&gt;
	&lt;rect x=&quot;0&quot; y=&quot;0&quot; width=&quot;3960&quot; height=&quot;2880&quot; style=&quot;fill:#EAF2F3;stroke:none&quot;/&gt;
	&lt;rect x=&quot;0.00&quot; y=&quot;0.00&quot; width=&quot;3959.88&quot; height=&quot;2880.00&quot; style=&quot;fill:#FFFFFF&quot;/&gt;
	&lt;rect x=&quot;2.88&quot; y=&quot;2.88&quot; width=&quot;3954.12&quot; height=&quot;2874.24&quot; style=&quot;fill:none;stroke:#FFFFFF;stroke-width:5.76&quot;/&gt;
	&lt;rect x=&quot;390.80&quot; y=&quot;275.35&quot; width=&quot;3468.22&quot; height=&quot;2213.83&quot; style=&quot;fill:#FFFFFF&quot;/&gt;
	&lt;rect x=&quot;393.68&quot; y=&quot;278.23&quot; width=&quot;3462.46&quot; height=&quot;2208.07&quot; style=&quot;fill:none;stroke:#FFFFFF;stroke-width:5.76&quot;/&gt;
	&lt;line x1=&quot;390.80&quot; y1=&quot;2425.82&quot; x2=&quot;3859.02&quot; y2=&quot;2425.82&quot; style=&quot;stroke:#EAF2F3;stroke-width:8.64&quot;/&gt;
	&lt;line x1=&quot;390.80&quot; y1=&quot;1908.28&quot; x2=&quot;3859.02&quot; y2=&quot;1908.28&quot; style=&quot;stroke:#EAF2F3;stroke-width:8.64&quot;/&gt;
	&lt;line x1=&quot;390.80&quot; y1=&quot;1390.87&quot; x2=&quot;3859.02&quot; y2=&quot;1390.87&quot; style=&quot;stroke:#EAF2F3;stroke-width:8.64&quot;/&gt;
	&lt;line x1=&quot;390.80&quot; y1=&quot;873.33&quot; x2=&quot;3859.02&quot; y2=&quot;873.33&quot; style=&quot;stroke:#EAF2F3;stroke-width:8.64&quot;/&gt;
	&lt;line x1=&quot;390.80&quot; y1=&quot;355.79&quot; x2=&quot;3859.02&quot; y2=&quot;355.79&quot; style=&quot;stroke:#EAF2F3;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;1728.35&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;1764.48&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;2021.77&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;1834.40&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2217.30&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2385.28&quot; cy=&quot;2246.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1880.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1568.33&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;1802.85&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;1785.27&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;1861.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;2114.33&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;1801.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1835.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2360.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;1812.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1407.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1690.48&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;2295.51&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;1379.98&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;1940.96&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;1698.27&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2094.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2233.32&quot; cy=&quot;2022.76&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2164.08&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;1996.40&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;1822.03&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;1772.77&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2022.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1596.30&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2269.65&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;2258.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2119.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2213.71&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;1610.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1862.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;1808.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2151.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;1739.11&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2188.34&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2060.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2115.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2103.13&quot; cy=&quot;2378.80&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2184.75&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;1829.21&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1835.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;1308.08&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;1482.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;1765.10&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1756.01&quot; cy=&quot;1730.95&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;1921.28&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;1640.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1983.40&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;1968.92&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2134.01&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;1703.97&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2016.57&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1039.99&quot; cy=&quot;1854.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1929.63&quot; cy=&quot;2224.10&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;1848.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1996.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;1882.05&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;852.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;2230.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2341.84&quot; cy=&quot;2190.94&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;1797.65&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2055.92&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;2142.80&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;1932.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2233.32&quot; cy=&quot;1964.84&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;1678.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2081.47&quot; cy=&quot;2066.81&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2081.47&quot; cy=&quot;1999.00&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2298.41&quot; cy=&quot;2185.25&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2623.99&quot; cy=&quot;2343.16&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2062.11&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2140.82&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;1859.77&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1910.51&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;2156.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2054.81&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;1819.43&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2059.82&quot; cy=&quot;2076.09&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1979.32&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2068.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2103.13&quot; cy=&quot;2262.84&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;1940.96&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;3708.91&quot; cy=&quot;2183.14&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2103.13&quot; cy=&quot;2093.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2081.47&quot; cy=&quot;1961.75&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;1800.25&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2002.09&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;1637.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1972.95&quot; cy=&quot;1861.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1951.29&quot; cy=&quot;2044.54&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2256.65&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2255.10&quot; cy=&quot;2162.48&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2115.94&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2211.66&quot; cy=&quot;2190.94&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;1954.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1756.01&quot; cy=&quot;1960.63&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2235.49&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;1566.35&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1811.14&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1929.63&quot; cy=&quot;1891.83&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2016.38&quot; cy=&quot;2055.92&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1864.54&quot; cy=&quot;2020.16&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2428.59&quot; cy=&quot;1685.40&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2190.00&quot; cy=&quot;2099.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;1924.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2354.54&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2200.22&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;1507.82&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2309.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1629.47&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1948.75&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;1985.01&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1815.22&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;1537.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;2092.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2124.91&quot; cy=&quot;1733.42&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2146.57&quot; cy=&quot;1997.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2289.32&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1929.63&quot; cy=&quot;2277.94&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2190.00&quot; cy=&quot;2255.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2001.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2201.33&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2076.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2111.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2178.07&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2145.40&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2146.57&quot; cy=&quot;1876.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2016.38&quot; cy=&quot;2163.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2114.83&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2190.00&quot; cy=&quot;1926.48&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2152.20&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2211.66&quot; cy=&quot;1620.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;2115.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1726.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1734.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;1911.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;1892.82&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2016.38&quot; cy=&quot;2017.06&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2385.28&quot; cy=&quot;2045.03&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;1963.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;1419.83&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1537.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2278.43&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;1612.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;1804.33&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1886.20&quot; cy=&quot;1829.21&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1756.01&quot; cy=&quot;2068.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;1938.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1888.24&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2300.71&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2220.52&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;1882.54&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;1628.48&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;1973.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;3101.30&quot; cy=&quot;2259.25&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2164.08&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1951.29&quot; cy=&quot;1768.20&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1995.90&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2190.00&quot; cy=&quot;2107.65&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2190.00&quot; cy=&quot;1990.70&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;1741.22&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;3795.66&quot; cy=&quot;2158.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2032.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2081.47&quot; cy=&quot;2119.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2276.75&quot; cy=&quot;2115.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2124.91&quot; cy=&quot;1706.57&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2168.22&quot; cy=&quot;1880.94&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2229.80&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1972.95&quot; cy=&quot;2174.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2137.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;2254.05&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;1835.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2493.69&quot; cy=&quot;2112.35&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2146.57&quot; cy=&quot;1833.91&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2116.93&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1951.29&quot; cy=&quot;2102.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2217.92&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;1903.21&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2175.96&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2035.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2124.91&quot; cy=&quot;1876.85&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2150.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2126.22&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2016.38&quot; cy=&quot;2170.77&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2180.05&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2198.24&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2204.92&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2059.82&quot; cy=&quot;2112.85&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2022.76&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2222.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2080.67&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2269.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;2026.35&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1886.20&quot; cy=&quot;1496.43&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2190.00&quot; cy=&quot;1870.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2175.96&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2063.10&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2190.00&quot; cy=&quot;1916.70&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2385.28&quot; cy=&quot;1730.95&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1864.54&quot; cy=&quot;1859.77&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2602.22&quot; cy=&quot;1802.35&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2298.41&quot; cy=&quot;2052.83&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;1772.28&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2115.94&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2137.11&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2363.50&quot; cy=&quot;2032.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2153.69&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1532.20&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2131.41&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2167.18&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2009.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;2256.65&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2119.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2157.90&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;1966.82&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1039.99&quot; cy=&quot;2193.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2176.46&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1929.63&quot; cy=&quot;2258.76&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2005.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2204.43&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2138.71&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2157.28&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2301.20&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2082.78&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2082.78&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;1942.56&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2161.49&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2168.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1756.01&quot; cy=&quot;1942.07&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2052.21&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2027.46&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2050.23&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2010.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2087.48&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;1818.81&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2108.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;1944.05&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;1929.57&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2024.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;1981.42&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2059.51&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;1877.84&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2168.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2229.30&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2304.30&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;2019.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;1896.03&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2166.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;1979.81&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2203.93&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2006.67&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2034.64&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2030.06&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2133.52&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1018.22&quot; cy=&quot;2093.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2165.07&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2069.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2079.19&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2077.08&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2046.52&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2048.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2178.56&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2044.54&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2007.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2003.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;1878.95&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;2261.85&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1039.99&quot; cy=&quot;1973.63&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1061.65&quot; cy=&quot;1660.03&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;1802.85&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;1939.47&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;1931.67&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2083.27&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2026.96&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;1896.53&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;1908.41&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;1934.77&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2032.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;1767.08&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2114.83&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2142.80&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;1958.65&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;1796.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1950.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;1854.08&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2123.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;2116.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;1814.23&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2123.12&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2136.12&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;2013.48&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1901.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2044.54&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;1854.08&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;1984.52&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1913.61&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2015.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;2089.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1061.65&quot; cy=&quot;2280.54&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;1983.90&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;1945.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1039.99&quot; cy=&quot;2017.56&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;1929.08&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1018.22&quot; cy=&quot;2358.62&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2042.93&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;1954.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;1949.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;1997.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1061.65&quot; cy=&quot;2255.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;996.56&quot; cy=&quot;1855.57&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;1899.62&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;974.90&quot; cy=&quot;1819.43&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1905.81&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2105.05&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2096.27&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2044.54&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;1855.57&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;1905.81&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;1982.41&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;1897.52&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1039.99&quot; cy=&quot;2004.69&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2071.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;1855.07&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;1953.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;2076.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1061.65&quot; cy=&quot;2161.49&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2030.55&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;1885.64&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1018.22&quot; cy=&quot;2119.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;1922.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;996.56&quot; cy=&quot;2031.54&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;931.47&quot; cy=&quot;1881.43&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;1706.57&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;1965.83&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2170.27&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1671.91&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1986.00&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2389.69&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;1949.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;1921.28&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1736.02&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1907.91&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1691.10&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;1758.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2034.14&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2139.21&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2022.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;2053.32&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1982.91&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1975.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;1753.10&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;1804.95&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;1710.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1951.35&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1921.28&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;1890.84&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2035.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;1839.11&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;1863.86&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;2156.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;1832.30&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2038.35&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;1671.91&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2050.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1953.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;1989.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;1992.19&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;1570.93&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2091.07&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;1708.67&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1883.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2189.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;1877.84&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1756.01&quot; cy=&quot;1821.41&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;1891.33&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;1883.53&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2101.46&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1842.76&quot; cy=&quot;1736.64&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1844.18&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1683.80&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;1935.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2009.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1667.71&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2141.81&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;1424.53&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2146.51&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;1883.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;1757.80&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;1808.54&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2116.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;1899.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2125.72&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;1798.64&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;1906.30&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;1677.61&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2392.78&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1842.76&quot; cy=&quot;2048.62&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2015.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;1843.19&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2021.77&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1924.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2183.76&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1994.73&quot; cy=&quot;1768.20&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1842.76&quot; cy=&quot;1774.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2166.06&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2177.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2262.84&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;2061.62&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;1554.47&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2244.77&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;2162.48&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;1958.65&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2100.97&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;1754.21&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2044.54&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2124.91&quot; cy=&quot;2100.97&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;1957.54&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1842.76&quot; cy=&quot;2181.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;1743.32&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2047.63&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2023.75&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2188.83&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2045.53&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;1908.41&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;1961.75&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2047.63&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2068.30&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;1980.31&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;1809.53&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2161.49&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;1714.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1918.80&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2096.27&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;1687.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1842.76&quot; cy=&quot;2074.98&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;1960.63&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1931.18&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2114.83&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1831.31&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;1781.07&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;1721.54&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1886.20&quot; cy=&quot;2046.02&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;1938.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2108.64&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2103.13&quot; cy=&quot;2425.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1939.97&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1774.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;1438.51&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2064.21&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;2038.84&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;2007.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;2271.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;1665.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;1949.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2261.35&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1941.57&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;1327.76&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;2018.05&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1749.51&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2098.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2050.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;1814.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2045.53&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2064.71&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;996.56&quot; cy=&quot;2261.35&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1018.22&quot; cy=&quot;1975.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;996.56&quot; cy=&quot;2131.91&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1039.99&quot; cy=&quot;2022.76&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;1993.80&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;1863.86&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2146.57&quot; cy=&quot;1972.51&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2159.38&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1039.99&quot; cy=&quot;2136.61&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2032.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1039.99&quot; cy=&quot;2003.08&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2103.13&quot; cy=&quot;1898.01&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;1986.00&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2072.51&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1885.64&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2276.75&quot; cy=&quot;1932.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2397.98&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2149.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;996.56&quot; cy=&quot;1863.86&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2266.55&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;1930.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;2134.01&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;2131.41&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;2253.56&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2104.56&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;1939.97&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;2096.27&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;1996.40&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1061.65&quot; cy=&quot;1657.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1018.22&quot; cy=&quot;2194.53&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2254.67&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;1636.64&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2168.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2120.52&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2016.38&quot; cy=&quot;1938.98&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;888.03&quot; cy=&quot;2126.83&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;1913.61&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;1823.51&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2073.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;1948.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2190.00&quot; cy=&quot;2247.87&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2109.76&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2177.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2328.06&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1756.01&quot; cy=&quot;2181.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2331.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2071.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2115.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1980.31&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;1989.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2211.11&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2344.64&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1971.52&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2209.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2335.85&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2165.57&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2197.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2304.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2166.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2021.27&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;1832.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;2167.67&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2325.46&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1886.20&quot; cy=&quot;2019.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2102.95&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2054.31&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2323.97&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2003.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2250.96&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2038.04&quot; cy=&quot;1694.69&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2160.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2113.84&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;1875.24&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2109.14&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1897.02&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2234.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2247.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2192.05&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2035.75&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2116.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;1897.52&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2146.51&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;1981.92&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;1953.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2152.70&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2077.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2037.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2077.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2104.56&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2124.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;1975.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2222.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;1876.85&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1817.33&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2213.21&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;2141.31&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2172.38&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2247.87&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2137.11&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1756.01&quot; cy=&quot;1875.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1860.76&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2301.70&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2365.93&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;1999.00&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;1934.77&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2279.42&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2130.92&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2185.25&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1936.38&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2061.62&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;2078.69&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;1445.20&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1923.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;1863.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2388.70&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2232.40&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2106.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;1994.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;1785.27&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;1922.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1885.64&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2075.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2393.28&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2110.25&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1959.64&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2112.35&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2141.31&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1589.12&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;1950.86&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2315.19&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2094.16&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2032.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;1784.65&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2317.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1886.20&quot; cy=&quot;1926.48&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2053.82&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2157.28&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2242.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2049.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;1821.41&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1861.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1963.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1994.73&quot; cy=&quot;2115.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;1811.63&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1832.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;1906.80&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;2048.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1756.01&quot; cy=&quot;1769.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;1852.97&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2031.05&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2190.00&quot; cy=&quot;1752.11&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;1533.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;1857.67&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;2209.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1888.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2022.76&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1994.73&quot; cy=&quot;1808.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1942.56&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2087.48&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1968.92&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2023.75&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;1998.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;1828.71&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1886.20&quot; cy=&quot;1589.62&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2039.83&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;1870.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1848.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1909.40&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;1808.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;1827.10&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;1933.78&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;1788.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2028.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2339.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;2003.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1919.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1972.95&quot; cy=&quot;1699.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;1662.01&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;1952.96&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2051.72&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;1909.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;2112.85&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;1931.67&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;1847.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1739.11&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2227.20&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;2160.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2065.70&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2016.38&quot; cy=&quot;1857.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2102.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2147.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1890.34&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1992.19&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;1964.35&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2058.52&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2061.12&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2093.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1951.29&quot; cy=&quot;1970.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;1973.63&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1948.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;1978.21&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1864.54&quot; cy=&quot;1367.11&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1966.82&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2358.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1886.20&quot; cy=&quot;1798.64&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1842.76&quot; cy=&quot;1947.76&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;2024.86&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2073.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2010.38&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;1972.02&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2052.83&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;1708.67&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1864.54&quot; cy=&quot;1871.16&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1842.76&quot; cy=&quot;1719.56&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;1935.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2003.08&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;1970.53&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;1824.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2027.46&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2023.75&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;1919.30&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2228.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2037.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2142.30&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1857.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2138.22&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2073.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;2216.31&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2317.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2235.98&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2166.06&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2243.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2186.85&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1018.22&quot; cy=&quot;2063.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2066.81&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2316.18&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;1961.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2077.08&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2284.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2227.69&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2034.64&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2141.81&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2068.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2148.00&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1061.65&quot; cy=&quot;2280.54&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2375.70&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2291.92&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2232.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1950.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2096.27&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2160.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2100.35&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2062.61&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2023.75&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2279.42&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2258.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2112.35&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2389.69&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2322.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2140.20&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2292.42&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2162.48&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2030.55&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2142.30&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2160.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2199.72&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;1991.69&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2370.51&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2239.57&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2059.02&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2040.33&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1018.22&quot; cy=&quot;2227.20&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2358.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2233.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;2031.54&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2108.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2304.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2010.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;953.12&quot; cy=&quot;2272.24&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2163.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;1990.70&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2096.76&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2057.90&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2256.16&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2282.52&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1998.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;1999.00&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;1921.90&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2085.87&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;2240.07&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2317.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2067.80&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2175.96&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2240.69&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;2230.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;1999.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;1970.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2131.91&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2022.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2244.77&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2031.05&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2274.84&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;974.90&quot; cy=&quot;2296.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1018.22&quot; cy=&quot;2245.27&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2324.96&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2183.14&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2007.78&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1939.47&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2186.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;2248.98&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2012.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2023.75&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2206.53&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2219.40&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2127.33&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2126.83&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2234.38&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1839.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2074.98&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;1912.00&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;1997.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2373.10&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1952.96&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2350.83&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2181.16&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2049.12&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2145.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2145.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;1886.63&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;1908.41&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1799.75&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;1847.27&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2140.82&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2031.54&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1061.65&quot; cy=&quot;1683.80&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2068.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;1918.18&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2137.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2151.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;1731.94&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;2348.23&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;1283.70&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2000.98&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;1949.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2216.31&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2365.93&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2033.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2155.30&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;1929.57&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;2175.96&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2393.28&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2118.05&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2069.91&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2389.19&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2341.55&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2324.47&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2289.32&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1844.80&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1061.65&quot; cy=&quot;1436.91&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2246.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2053.32&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2161.49&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2039.34&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2169.28&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2289.32&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;1540.98&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1990.21&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;1820.91&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2225.09&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2104.06&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2120.52&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;1773.27&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1061.65&quot; cy=&quot;1909.40&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1886.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;1770.18&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;1905.81&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;2103.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;1998.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;1916.70&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2226.21&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2047.63&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;1806.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;1837.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2357.14&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;1686.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2103.13&quot; cy=&quot;2076.09&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1984.52&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1864.54&quot; cy=&quot;2104.06&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2034.14&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;1901.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;1921.90&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2146.57&quot; cy=&quot;633.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1740.23&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;1806.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;1402.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;1551.25&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2038.04&quot; cy=&quot;1590.61&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1886.20&quot; cy=&quot;2131.91&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;2071.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2168.22&quot; cy=&quot;1811.14&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;1579.22&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1783.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1791.46&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1756.01&quot; cy=&quot;2272.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;1596.80&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2103.13&quot; cy=&quot;1760.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1929.63&quot; cy=&quot;2046.02&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1886.20&quot; cy=&quot;1942.56&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1642.96&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2623.99&quot; cy=&quot;1088.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;1196.82&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2038.04&quot; cy=&quot;2034.14&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;1796.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1929.63&quot; cy=&quot;2130.42&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1864.54&quot; cy=&quot;1885.64&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;1889.72&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1864.54&quot; cy=&quot;2234.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1842.76&quot; cy=&quot;1063.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1849.87&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2103.13&quot; cy=&quot;1429.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2226.21&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;2065.70&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1858.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1994.73&quot; cy=&quot;1853.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2255.10&quot; cy=&quot;1781.07&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;1711.76&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;1786.76&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1953.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2190.00&quot; cy=&quot;1627.86&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1745.42&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;2035.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1911.01&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1932.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1886.20&quot; cy=&quot;1978.21&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;1662.63&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2602.22&quot; cy=&quot;2175.96&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1809.03&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1943.55&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1864.54&quot; cy=&quot;1093.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;1940.46&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1972.95&quot; cy=&quot;2248.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1929.63&quot; cy=&quot;1761.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2006.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;1760.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2255.10&quot; cy=&quot;1700.38&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2038.04&quot; cy=&quot;1552.86&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2341.84&quot; cy=&quot;1249.05&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2081.47&quot; cy=&quot;1540.98&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2211.66&quot; cy=&quot;2065.70&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1994.73&quot; cy=&quot;2025.85&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;2164.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;2066.81&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;1935.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;1353.12&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2645.65&quot; cy=&quot;1588.63&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2081.47&quot; cy=&quot;2138.71&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;1885.64&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2667.31&quot; cy=&quot;1656.32&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;1992.81&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;1862.87&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2059.82&quot; cy=&quot;1882.54&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;1581.82&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1842.76&quot; cy=&quot;2126.22&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2298.41&quot; cy=&quot;2133.02&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;1854.08&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2112.85&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;1460.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1756.01&quot; cy=&quot;1331.34&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2341.84&quot; cy=&quot;2053.32&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2428.59&quot; cy=&quot;1912.49&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;1940.46&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1756.01&quot; cy=&quot;1699.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2105.55&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2002.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2233.32&quot; cy=&quot;2307.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;2225.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1951.29&quot; cy=&quot;1388.77&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;420.52&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2016.38&quot; cy=&quot;1638.75&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1864.54&quot; cy=&quot;1712.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1929.63&quot; cy=&quot;1964.35&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1521.80&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1864.54&quot; cy=&quot;1137.30&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1907.42&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2053.82&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1670.31&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1756.01&quot; cy=&quot;2271.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1780.57&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;2063.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;2251.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;1759.41&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;1856.18&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;2112.85&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;1293.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2061.12&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;865.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;1979.81&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;1973.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;1827.10&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1811.14&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2245.76&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1851.48&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1975.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;1857.67&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1811.14&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;1800.25&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1840.10&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2143.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1864.54&quot; cy=&quot;1878.95&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2261.85&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;1796.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2298.41&quot; cy=&quot;1995.90&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2090.08&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1456.09&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1864.54&quot; cy=&quot;1972.02&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2322.98&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;1850.49&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;2002.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2209.63&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;1896.03&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;1712.75&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;1931.67&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;1893.43&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2042.43&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1928.09&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;2012.98&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;1974.12&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2048.62&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;1784.16&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2034.14&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1667.71&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;1979.32&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2197.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2305.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2168.22&quot; cy=&quot;2109.14&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;2250.46&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1713.25&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2018.67&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1828.71&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1716.34&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;1849.87&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2135.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;1926.97&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2355.53&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;1772.77&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2041.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2107.16&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1929.63&quot; cy=&quot;2144.41&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2275.34&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2083.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1946.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1864.54&quot; cy=&quot;1964.84&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2014.96&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1973.63&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2007.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;1900.12&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2040.33&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2083.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1950.86&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;1885.14&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;1822.03&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2061.12&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1929.63&quot; cy=&quot;2030.06&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2058.52&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2081.47&quot; cy=&quot;2074.98&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2092.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;1857.67&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2022.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2059.02&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2028.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2131.91&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2093.67&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;1967.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2006.67&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;2068.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2072.51&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2060.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2055.43&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;2248.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;1765.10&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;2224.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2216.31&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2038.35&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2079.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2161.49&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2049.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1961.75&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2009.27&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1756.01&quot; cy=&quot;1897.02&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1842.76&quot; cy=&quot;2133.52&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1971.52&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;1911.01&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;1973.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2187.35&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2097.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2133.02&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1756.01&quot; cy=&quot;2012.98&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2123.12&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2151.09&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;974.90&quot; cy=&quot;2162.97&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2121.64&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2070.40&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2129.43&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2050.23&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2797.49&quot; cy=&quot;2252.57&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2211.66&quot; cy=&quot;1976.72&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;2202.82&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2168.22&quot; cy=&quot;1860.76&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2190.00&quot; cy=&quot;2210.12&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2667.31&quot; cy=&quot;1988.11&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2949.46&quot; cy=&quot;1895.41&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2281.53&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2016.38&quot; cy=&quot;1951.85&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2168.22&quot; cy=&quot;2081.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2038.04&quot; cy=&quot;1959.64&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2103.13&quot; cy=&quot;2052.83&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2016.38&quot; cy=&quot;1881.43&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2233.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2190.00&quot; cy=&quot;1968.43&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2645.65&quot; cy=&quot;1889.72&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2363.50&quot; cy=&quot;2054.31&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2775.84&quot; cy=&quot;1722.65&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2249.97&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2168.22&quot; cy=&quot;1787.87&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2059.82&quot; cy=&quot;2035.75&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;1967.93&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2005.18&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2363.50&quot; cy=&quot;1813.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2298.41&quot; cy=&quot;1851.98&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2645.65&quot; cy=&quot;1730.95&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2124.91&quot; cy=&quot;2031.05&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;1717.46&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2385.28&quot; cy=&quot;2071.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2190.00&quot; cy=&quot;2130.42&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1994.73&quot; cy=&quot;2224.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2276.75&quot; cy=&quot;2108.64&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1864.54&quot; cy=&quot;2066.19&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2298.41&quot; cy=&quot;2110.25&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2493.69&quot; cy=&quot;1827.72&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2081.47&quot; cy=&quot;1925.98&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2124.24&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2059.82&quot; cy=&quot;1938.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1993.30&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2190.00&quot; cy=&quot;1783.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;2049.12&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2124.91&quot; cy=&quot;2110.75&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2103.13&quot; cy=&quot;2079.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2125.72&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2146.57&quot; cy=&quot;1180.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2602.22&quot; cy=&quot;2061.62&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1972.95&quot; cy=&quot;1955.43&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2341.84&quot; cy=&quot;1956.05&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;2019.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2233.32&quot; cy=&quot;1764.48&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2124.91&quot; cy=&quot;1979.32&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1886.20&quot; cy=&quot;2094.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;2038.84&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2190.00&quot; cy=&quot;2370.51&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;2018.67&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;3340.01&quot; cy=&quot;1969.42&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;3122.95&quot; cy=&quot;2125.23&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1994.73&quot; cy=&quot;1803.34&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2059.82&quot; cy=&quot;1971.52&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;2049.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2341.84&quot; cy=&quot;2111.24&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2276.75&quot; cy=&quot;1745.92&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2233.32&quot; cy=&quot;2134.01&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2103.13&quot; cy=&quot;2014.47&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2797.49&quot; cy=&quot;1869.55&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2298.41&quot; cy=&quot;1835.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2136.12&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;3144.74&quot; cy=&quot;2048.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2515.47&quot; cy=&quot;1848.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2124.91&quot; cy=&quot;1990.70&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2168.22&quot; cy=&quot;1982.41&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;1940.46&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2363.50&quot; cy=&quot;2209.63&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2819.27&quot; cy=&quot;1835.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2276.75&quot; cy=&quot;1923.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2363.50&quot; cy=&quot;2110.75&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1951.29&quot; cy=&quot;2033.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2140.20&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2233.32&quot; cy=&quot;1999.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2276.75&quot; cy=&quot;1712.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;1988.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2149.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2186.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2109.76&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2175.96&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;1928.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2075.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2077.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2063.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2226.21&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;2225.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2330.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2270.14&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2160.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2580.56&quot; cy=&quot;2088.97&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2040.83&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2040.33&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2084.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2156.78&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2390.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2019.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2199.72&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2366.42&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2172.87&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2094.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2307.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2183.76&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2406.27&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2115.94&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2202.82&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1756.01&quot; cy=&quot;2051.72&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2278.43&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1972.95&quot; cy=&quot;1995.28&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2108.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2308.38&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1886.20&quot; cy=&quot;1954.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2185.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2312.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2306.40&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2129.93&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2066.19&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;1964.35&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1929.63&quot; cy=&quot;2332.27&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2042.93&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2223.61&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;2067.80&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2168.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2175.47&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2257.77&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2307.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2237.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2150.10&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2275.83&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2140.82&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;1992.19&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1864.54&quot; cy=&quot;2296.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2084.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2219.40&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2004.19&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2184.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;2246.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2203.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2277.94&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2179.55&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1061.65&quot; cy=&quot;2079.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;2145.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;1996.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2205.91&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2232.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2344.64&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2148.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2217.30&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2082.28&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2146.51&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2263.95&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2047.14&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2375.70&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1929.63&quot; cy=&quot;1765.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2212.72&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2130.92&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2198.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;1993.30&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2139.70&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2155.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2119.53&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1929.63&quot; cy=&quot;1479.85&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2016.38&quot; cy=&quot;1399.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;2171.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2124.91&quot; cy=&quot;1835.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2385.28&quot; cy=&quot;2248.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2233.32&quot; cy=&quot;2037.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2103.13&quot; cy=&quot;2028.94&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2341.84&quot; cy=&quot;1990.21&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2059.82&quot; cy=&quot;2379.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2775.84&quot; cy=&quot;2100.97&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;2134.51&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2775.84&quot; cy=&quot;1880.94&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2472.03&quot; cy=&quot;2045.03&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1929.63&quot; cy=&quot;1999.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;2050.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;1873.76&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1285.31&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1886.20&quot; cy=&quot;1977.71&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2406.94&quot; cy=&quot;1561.16&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;1677.61&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2537.12&quot; cy=&quot;2066.81&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2623.99&quot; cy=&quot;1739.11&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;2036.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2667.31&quot; cy=&quot;2198.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2385.28&quot; cy=&quot;2235.98&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;1735.52&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2146.57&quot; cy=&quot;1640.85&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2059.82&quot; cy=&quot;1863.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1654.84&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2233.32&quot; cy=&quot;2317.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1938.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2251.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;1617.09&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2059.82&quot; cy=&quot;1970.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;2019.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;1694.19&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2276.75&quot; cy=&quot;1934.27&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;1769.19&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1886.20&quot; cy=&quot;1806.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2032.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;1482.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2103.13&quot; cy=&quot;2276.82&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2320.19&quot; cy=&quot;2127.33&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2385.28&quot; cy=&quot;1947.27&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;1686.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;2120.52&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2906.02&quot; cy=&quot;2082.28&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;2097.75&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2103.13&quot; cy=&quot;1838.49&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1886.20&quot; cy=&quot;1672.90&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1886.20&quot; cy=&quot;2047.14&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1864.54&quot; cy=&quot;1659.42&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;2010.38&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1972.95&quot; cy=&quot;1911.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;1931.18&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;1902.72&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2450.37&quot; cy=&quot;2230.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;1813.12&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2059.82&quot; cy=&quot;1583.43&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2428.59&quot; cy=&quot;2212.22&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2059.82&quot; cy=&quot;2344.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1864.54&quot; cy=&quot;1975.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;1851.48&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1994.73&quot; cy=&quot;1698.77&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;1815.22&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1842.76&quot; cy=&quot;2143.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;1870.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2493.69&quot; cy=&quot;2021.77&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1756.01&quot; cy=&quot;1781.56&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2168.22&quot; cy=&quot;2285.61&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;1873.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2081.47&quot; cy=&quot;2236.97&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2255.10&quot; cy=&quot;2102.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;1931.67&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2067.80&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2045.53&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1809.03&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2198.24&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;2270.64&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;1983.40&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2148.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1780.57&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;1310.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2143.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1845.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1814.23&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1888.24&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1900.61&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;1897.52&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2071.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;1939.97&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2314.07&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;1791.96&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2341.05&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;2113.84&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1745.42&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2308.38&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2250.96&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;1918.80&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2258.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1972.95&quot; cy=&quot;1662.63&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2013.48&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2298.11&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2215.32&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;1716.96&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1896.03&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2017.56&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1601.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2256.65&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;1749.51&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2231.90&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;2014.47&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;1638.75&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2017.06&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2183.76&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1699.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2343.16&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2050.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2188.34&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2109.14&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1864.54&quot; cy=&quot;2121.14&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;1813.12&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2412.46&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2014.96&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;1913.11&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;1660.53&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2241.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2131.91&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1842.76&quot; cy=&quot;2192.05&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2271.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2089.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1886.20&quot; cy=&quot;2292.91&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1842.76&quot; cy=&quot;2210.62&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2237.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2088.47&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2276.82&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2254.05&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2039.34&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1886.20&quot; cy=&quot;1760.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;1978.82&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2008.28&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;1905.31&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2158.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2115.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;1935.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2138.71&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2068.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2093.67&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1584.42&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2011.87&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2231.90&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;1619.07&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2049.12&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2231.28&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2078.69&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;1745.42&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2269.65&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;2073.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;2211.61&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1951.29&quot; cy=&quot;1701.86&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2201.33&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2221.51&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2217.30&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2190.00&quot; cy=&quot;1987.61&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;2305.91&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2182.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1781.56&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1951.29&quot; cy=&quot;2145.40&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;2003.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1951.29&quot; cy=&quot;2211.61&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2282.02&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2189.95&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;1957.54&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1061.65&quot; cy=&quot;2115.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;1918.18&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;2009.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;1918.18&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;888.03&quot; cy=&quot;2064.21&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2011.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;1770.18&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;974.90&quot; cy=&quot;2116.93&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1929.63&quot; cy=&quot;2211.11&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2034.64&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2380.41&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2163.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2019.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2233.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1951.29&quot; cy=&quot;1870.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2298.11&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2154.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;1892.32&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;1989.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2121.64&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2148.49&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2104.56&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2040.33&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1039.99&quot; cy=&quot;2064.71&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;2390.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;2233.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2029.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;1887.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1061.65&quot; cy=&quot;2189.95&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;822.94&quot; cy=&quot;2116.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2093.67&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1061.65&quot; cy=&quot;2019.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;1943.06&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;1983.40&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;2033.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1061.65&quot; cy=&quot;2212.72&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2222.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;2067.31&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;953.12&quot; cy=&quot;2045.53&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2227.69&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1039.99&quot; cy=&quot;2140.20&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2296.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;953.12&quot; cy=&quot;2386.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;974.90&quot; cy=&quot;2022.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;974.90&quot; cy=&quot;2130.42&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2260.86&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;974.90&quot; cy=&quot;1904.20&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2107.65&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;2148.49&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2190.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1039.99&quot; cy=&quot;2184.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2391.30&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;1957.54&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2058.52&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2037.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2054.81&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;2092.06&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;1974.12&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;953.12&quot; cy=&quot;2219.40&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1018.22&quot; cy=&quot;2001.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1039.99&quot; cy=&quot;2066.19&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;2065.20&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2151.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2312.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;953.12&quot; cy=&quot;2144.90&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1018.22&quot; cy=&quot;1928.09&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1039.99&quot; cy=&quot;2083.27&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;2310.98&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1061.65&quot; cy=&quot;2062.61&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1061.65&quot; cy=&quot;2008.28&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;1931.18&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2114.33&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1018.22&quot; cy=&quot;2163.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1813.12&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1039.99&quot; cy=&quot;1640.85&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2044.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;953.12&quot; cy=&quot;1770.18&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;2289.32&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1061.65&quot; cy=&quot;2071.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1061.65&quot; cy=&quot;2300.21&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1039.99&quot; cy=&quot;2027.46&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1898.01&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1039.99&quot; cy=&quot;2168.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;2128.32&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2287.22&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;1907.42&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1039.99&quot; cy=&quot;2123.12&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1039.99&quot; cy=&quot;2186.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1951.29&quot; cy=&quot;2222.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;2231.28&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;996.56&quot; cy=&quot;2193.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;996.56&quot; cy=&quot;1945.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;1853.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;1982.41&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1958.03&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2060.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2023.25&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1976.72&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;2068.30&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;1961.75&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;1872.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;1953.95&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;1753.10&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1630.46&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2051.22&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2094.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2055.43&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2033.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;2011.87&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2054.81&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;1964.35&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2076.09&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;1939.47&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;2081.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;1987.12&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2186.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2119.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;1984.52&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1972.95&quot; cy=&quot;1944.05&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1972.95&quot; cy=&quot;1985.01&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;2205.91&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;1996.40&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;2235.49&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2241.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2144.90&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2189.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2115.94&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2181.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2042.43&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;2200.22&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2039.83&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2232.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2157.90&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2202.32&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2111.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2073.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;1998.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2373.10&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2193.54&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2187.35&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2159.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2076.09&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2083.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2234.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2081.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;1921.28&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;974.90&quot; cy=&quot;2166.06&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2797.49&quot; cy=&quot;2172.38&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;2298.11&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2104.06&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2062.11&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;1933.28&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2079.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2160.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2129.43&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2065.20&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2014.47&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2070.90&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2076.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2367.91&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;2106.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2060.01&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2126.83&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2102.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;2286.23&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2025.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2038.35&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2260.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2012.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2009.27&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2090.57&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2241.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2124.24&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2250.46&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2114.83&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2153.19&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2005.18&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2245.76&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2313.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2024.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2133.52&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2054.81&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2227.69&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2337.96&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2300.21&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2383.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2188.34&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2213.21&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2158.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;2091.07&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2187.84&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2215.32&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;1978.21&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2128.81&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2160.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2143.91&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2085.87&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;2109.76&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;2240.69&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2171.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2213.71&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2305.91&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2137.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2166.06&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2242.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;2336.35&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2161.49&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2125.72&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2141.81&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2285.61&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2282.02&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2063.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2110.25&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2031.05&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2253.56&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2197.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2129.93&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2003.08&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2105.55&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2156.78&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2227.20&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2226.70&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2139.21&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;1974.12&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2327.07&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2002.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2081.47&quot; cy=&quot;2091.56&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;1964.84&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;1814.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1860.76&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1864.54&quot; cy=&quot;2164.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1842.76&quot; cy=&quot;1883.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1822.03&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;1860.27&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1786.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1887.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2081.47&quot; cy=&quot;1814.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1851.98&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;1779.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2230.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1779.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2010.38&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2021.77&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;1749.01&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;1562.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;1812.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2046.52&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2302.19&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1800.25&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;1988.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;1931.18&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2104.06&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2062.61&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2190.00&quot; cy=&quot;2165.57&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2009.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2183.14&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;1741.71&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1842.76&quot; cy=&quot;1740.23&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2183.76&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2170.27&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1958.65&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1864.54&quot; cy=&quot;1858.16&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2057.90&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;1840.10&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;1922.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;1824.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2275.83&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;1777.97&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2137.11&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2047.14&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2309.00&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1771.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;2064.71&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;1920.78&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;2000.48&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;1826.11&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2070.90&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2075.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2094.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;1718.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;1987.12&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;1966.33&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;1939.47&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;1872.64&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;2400.08&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2105.05&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2189.95&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2122.63&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;1934.77&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2188.34&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2181.16&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;1960.14&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2022.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2076.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2075.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2081.47&quot; cy=&quot;1637.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;1871.65&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;1681.20&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;1988.11&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2124.91&quot; cy=&quot;1956.55&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1990.70&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2385.28&quot; cy=&quot;1890.34&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1870.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2030.06&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1643.95&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;2077.08&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;1891.33&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;1965.34&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2081.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2081.47&quot; cy=&quot;1742.83&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2109.14&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;2218.41&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;2106.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2011.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2077.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1806.93&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1994.73&quot; cy=&quot;1971.52&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2050.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;1856.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;1776.98&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2028.94&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1756.01&quot; cy=&quot;2398.97&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2245.27&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;931.47&quot; cy=&quot;2168.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1039.99&quot; cy=&quot;2041.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1018.22&quot; cy=&quot;2161.49&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1039.99&quot; cy=&quot;2110.75&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2129.43&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2296.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2202.82&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;2108.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2261.85&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2208.51&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2200.84&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2313.08&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2287.22&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2209.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2340.06&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1061.65&quot; cy=&quot;1959.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2209.63&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2181.16&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2144.41&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2136.12&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1756.01&quot; cy=&quot;2192.05&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;2167.18&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2026.96&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;1567.84&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2079.19&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2318.28&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2210.62&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2428.59&quot; cy=&quot;2177.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1018.22&quot; cy=&quot;1989.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;2034.64&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2257.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2279.92&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2055.92&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2173.86&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2153.69&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;909.69&quot; cy=&quot;2077.08&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2103.13&quot; cy=&quot;1851.98&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;1917.69&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2175.47&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;1985.01&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;1932.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2228.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;1852.47&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1951.29&quot; cy=&quot;1894.42&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1933.78&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;2179.55&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2059.51&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;1776.49&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1834.40&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;1817.33&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1864.54&quot; cy=&quot;2014.47&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1972.95&quot; cy=&quot;2017.56&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1864.54&quot; cy=&quot;1856.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2086.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;2160.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;2023.25&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2073.00&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;1924.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;1957.54&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;1809.03&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1902.22&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;2046.02&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;1642.96&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2114.83&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2018.05&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1864.54&quot; cy=&quot;1875.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1948.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2048.62&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;1990.70&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1756.01&quot; cy=&quot;2180.67&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;2153.19&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;1764.48&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;1980.31&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2326.57&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2059.82&quot; cy=&quot;1863.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2081.47&quot; cy=&quot;1068.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1951.29&quot; cy=&quot;1790.97&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2141.31&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2051.72&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;1755.70&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2044.54&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1937.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2032.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;1909.40&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2093.67&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1842.76&quot; cy=&quot;1906.30&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1884.03&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2072.51&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2276.75&quot; cy=&quot;1797.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2176.95&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2075.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2884.36&quot; cy=&quot;2118.05&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1756.01&quot; cy=&quot;1888.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2428.59&quot; cy=&quot;2136.61&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;2198.24&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2016.38&quot; cy=&quot;1749.51&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2016.38&quot; cy=&quot;2107.65&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1972.95&quot; cy=&quot;2096.76&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2187.84&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2059.82&quot; cy=&quot;1962.24&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;1899.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2385.28&quot; cy=&quot;1813.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2105.55&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1756.01&quot; cy=&quot;2181.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2207.52&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;1901.11&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;1794.55&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;2042.93&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2103.13&quot; cy=&quot;1570.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2068.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2064.71&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;1909.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;1959.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;1913.11&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2003.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2031.05&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2143.91&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2168.22&quot; cy=&quot;2225.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2064.21&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;1832.30&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2078.69&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1756.01&quot; cy=&quot;2165.07&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2179.06&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;2192.05&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2333.75&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2306.40&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1039.99&quot; cy=&quot;1735.03&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;1783.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2030.55&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;1903.71&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;1506.33&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;1994.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2123.12&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2020.65&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;1805.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;1851.98&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1912.00&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;1805.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;1811.14&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;1743.82&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2118.54&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;1892.32&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;1951.85&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2142.30&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2279.42&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2078.20&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2129.43&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;2222.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;1775.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2250.46&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;1874.25&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1039.99&quot; cy=&quot;2077.08&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2129.93&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2250.96&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2347.24&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2175.47&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;1932.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2148.49&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;1883.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2278.43&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2258.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2017.56&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;2302.19&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1709.16&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2316.67&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2321.38&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;1737.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2005.18&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2297.61&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1851.98&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2062.61&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;1899.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1478.86&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;1793.56&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;1592.71&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;1862.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1980.80&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1974.12&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;1696.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2256.65&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;1822.03&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;1637.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2283.14&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;1843.69&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2220.52&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1039.99&quot; cy=&quot;1752.11&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;1806.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2174.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2000.98&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;1544.08&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;2066.19&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;1679.71&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;2120.03&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1929.63&quot; cy=&quot;1687.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1900.12&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2059.82&quot; cy=&quot;1796.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2255.10&quot; cy=&quot;1967.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2190.00&quot; cy=&quot;1837.00&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1890.34&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1799.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;2107.16&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1886.20&quot; cy=&quot;1761.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1886.20&quot; cy=&quot;1892.82&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2081.47&quot; cy=&quot;1874.25&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1870.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1756.01&quot; cy=&quot;1736.64&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1864.54&quot; cy=&quot;1882.54&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;1964.84&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;2032.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1864.54&quot; cy=&quot;2313.08&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2002.09&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2154.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;1872.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;1976.72&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;1964.84&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1864.54&quot; cy=&quot;2031.54&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1590.61&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2060.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1932.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2169.28&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2020.65&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2019.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2011.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;2168.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2160.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1904.82&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2138.71&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2086.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2056.42&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;1917.19&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;1840.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;2064.21&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2119.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2169.78&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2059.02&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2011.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2111.24&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2099.86&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;1999.00&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2090.57&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2105.55&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2041.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1999.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1997.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;1874.75&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1994.73&quot; cy=&quot;2284.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2122.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2399.46&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;2086.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2162.97&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2090.57&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;1997.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2140.82&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;1862.87&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2137.11&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1865.96&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2151.09&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2182.65&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1947.76&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2055.43&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;1984.52&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;1854.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2001.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2018.67&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2108.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2078.69&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2077.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;1993.80&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2170.27&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;2067.80&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;2150.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;2040.83&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2069.91&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2120.03&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2175.47&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2319.77&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2245.27&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2057.41&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2188.83&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2018.05&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2072.51&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2031.54&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2031.05&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2049.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2054.31&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2123.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2144.90&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1941.57&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2050.23&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2025.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2016.57&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2363.50&quot; cy=&quot;2131.91&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1842.76&quot; cy=&quot;1995.90&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2645.65&quot; cy=&quot;2208.02&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1929.63&quot; cy=&quot;1873.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2602.22&quot; cy=&quot;1981.42&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2623.99&quot; cy=&quot;1948.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2066.19&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;2169.78&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1994.73&quot; cy=&quot;2303.80&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2006.67&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2124.91&quot; cy=&quot;1864.35&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2667.31&quot; cy=&quot;2142.30&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2515.47&quot; cy=&quot;1966.82&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2450.37&quot; cy=&quot;2013.48&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1864.54&quot; cy=&quot;1808.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2124.91&quot; cy=&quot;1806.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2472.03&quot; cy=&quot;1931.18&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1945.16&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2124.91&quot; cy=&quot;2306.90&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2406.94&quot; cy=&quot;1976.23&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2038.04&quot; cy=&quot;1713.25&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1994.73&quot; cy=&quot;1976.72&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1818.81&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2190.00&quot; cy=&quot;1851.48&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2016.38&quot; cy=&quot;2111.24&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;1831.80&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2083.27&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;2082.78&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2406.94&quot; cy=&quot;1954.94&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1951.29&quot; cy=&quot;2133.02&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2537.12&quot; cy=&quot;2148.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1961.75&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2840.93&quot; cy=&quot;2232.40&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;3057.86&quot; cy=&quot;1870.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2884.36&quot; cy=&quot;2135.62&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1842.76&quot; cy=&quot;2033.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2385.28&quot; cy=&quot;1768.69&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2450.37&quot; cy=&quot;2012.98&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;1943.06&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;2096.76&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;2161.49&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;1781.56&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2146.57&quot; cy=&quot;2108.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2819.27&quot; cy=&quot;1769.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2602.22&quot; cy=&quot;2035.75&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1842.76&quot; cy=&quot;2042.43&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;2068.30&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2329.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;974.90&quot; cy=&quot;2234.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1018.22&quot; cy=&quot;2181.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;2071.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1039.99&quot; cy=&quot;2318.28&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2247.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;2262.34&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;2054.31&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;2189.95&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;2080.67&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1039.99&quot; cy=&quot;2277.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1061.65&quot; cy=&quot;2155.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2152.70&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2084.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2127.33&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2214.82&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1039.99&quot; cy=&quot;2247.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1061.65&quot; cy=&quot;2144.90&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;996.56&quot; cy=&quot;2039.34&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2020.16&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1061.65&quot; cy=&quot;2004.69&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2164.08&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2184.75&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;2049.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;974.90&quot; cy=&quot;2301.20&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2248.98&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2277.94&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2127.82&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2311.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1901.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2303.80&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1061.65&quot; cy=&quot;2081.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2216.31&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;2033.65&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;2238.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;1707.56&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2279.92&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;1952.96&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2274.22&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2288.21&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2105.55&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;2052.21&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1374.78&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;1919.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1994.73&quot; cy=&quot;2334.86&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1864.54&quot; cy=&quot;1889.72&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;1810.64&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;1722.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;1424.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1763.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;1343.22&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;1769.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2009.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1949.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1439.01&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2819.27&quot; cy=&quot;1788.86&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2038.04&quot; cy=&quot;1629.47&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;1291.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1951.29&quot; cy=&quot;2021.77&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;1551.87&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;1861.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1864.54&quot; cy=&quot;1573.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1466.48&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1622.78&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;1487.65&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;2161.49&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1886.20&quot; cy=&quot;2212.22&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1843.19&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2124.91&quot; cy=&quot;1730.95&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1994.73&quot; cy=&quot;2075.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;2015.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2168.22&quot; cy=&quot;1592.71&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2190.00&quot; cy=&quot;1632.07&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;970.11&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;1706.57&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1972.95&quot; cy=&quot;1730.33&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2016.38&quot; cy=&quot;2039.34&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;1573.53&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;2067.31&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2103.13&quot; cy=&quot;1837.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;1741.71&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1994.73&quot; cy=&quot;1267.24&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2289.32&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2059.82&quot; cy=&quot;2120.03&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;1930.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1929.63&quot; cy=&quot;2092.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1693.57&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1886.20&quot; cy=&quot;1918.80&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;1521.31&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2190.00&quot; cy=&quot;1806.93&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2906.02&quot; cy=&quot;1694.69&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2016.38&quot; cy=&quot;1516.11&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;1564.25&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;1775.87&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1465.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;1420.94&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1929.63&quot; cy=&quot;1751.61&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1696.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1693.08&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;1704.46&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;1692.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;1896.03&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1969.42&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2274.84&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;1534.80&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;1837.00&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;1870.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;2274.22&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2276.75&quot; cy=&quot;1620.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;2276.33&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1722.65&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;1692.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1566.85&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;1750.00&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;1921.28&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;1789.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1506.33&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1656.32&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;2005.18&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;1570.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2103.13&quot; cy=&quot;2339.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1130.00&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;1915.09&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;2028.94&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;1871.16&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;1420.94&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;1783.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2406.94&quot; cy=&quot;1713.25&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1560.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2710.74&quot; cy=&quot;1668.33&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1929.63&quot; cy=&quot;2153.69&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2428.59&quot; cy=&quot;1830.20&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2086.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1982.41&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;1929.57&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2131.41&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1859.28&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1986.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2077.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2233.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2156.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2268.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2106.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1952.34&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;2134.01&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2213.71&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2130.42&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2109.76&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;1536.28&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2074.49&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2124.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;1989.10&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;2116.93&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;1883.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2168.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1930.19&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2216.31&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2002.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1886.20&quot; cy=&quot;2247.87&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;1946.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;2324.47&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2371.62&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2080.18&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;1592.22&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2231.28&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2114.33&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2246.38&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2039.34&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2074.49&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;1914.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1842.76&quot; cy=&quot;2216.31&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2158.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2314.07&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2372.61&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2030.06&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2113.84&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1864.54&quot; cy=&quot;2158.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2221.51&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2073.00&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2184.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2103.13&quot; cy=&quot;2033.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2012.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2353.92&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;1815.22&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2010.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2051.72&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;2257.77&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;1965.34&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;2199.23&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1756.01&quot; cy=&quot;2131.41&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2092.06&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;2329.67&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2159.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2204.92&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2132.53&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2157.90&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2181.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2174.97&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1938.98&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;1975.11&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2214.20&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2158.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;1977.71&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;1815.22&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1581.33&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2277.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2180.05&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2173.86&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;1917.69&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;1906.80&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2044.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2035.75&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2167.18&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1950.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2038.04&quot; cy=&quot;2264.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;1938.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2076.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;1827.10&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;1927.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;2148.00&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2032.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;1955.43&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;1992.81&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2089.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;1973.63&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1756.01&quot; cy=&quot;1848.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2004.19&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2028.94&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2306.90&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;2166.06&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2039.34&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2042.93&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;1841.09&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;1989.10&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2092.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1677.61&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2146.51&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2383.00&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;1926.48&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2193.54&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2261.85&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1939.47&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1943.55&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1957.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2001.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2281.03&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2189.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2020.16&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;1937.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1972.95&quot; cy=&quot;2086.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;3231.48&quot; cy=&quot;2281.03&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2274.22&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2071.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2121.14&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;1855.07&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2018.67&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2186.85&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1923.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1933.28&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2041.94&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1994.73&quot; cy=&quot;2111.24&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2039.34&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2028.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;2139.21&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;1912.00&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;1989.10&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;1991.69&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2036.25&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2097.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1756.01&quot; cy=&quot;1990.70&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2011.87&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2091.56&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2028.94&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;974.90&quot; cy=&quot;2208.02&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2265.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;2085.38&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2170.77&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2104.06&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2134.01&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2142.30&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2341.84&quot; cy=&quot;2116.93&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;2000.98&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;2364.32&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2124.24&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2004.19&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;1899.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2113.84&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1928.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;1728.35&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2971.11&quot; cy=&quot;2300.71&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1882.54&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2032.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;1810.02&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2193.54&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;1777.48&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1941.57&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2064.71&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;2017.56&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;2148.00&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2370.01&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;2066.81&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;2247.87&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2019.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;1928.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2110.75&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2285.61&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2078.69&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2102.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2277.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2111.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;1636.64&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;1854.08&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;1447.30&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2406.94&quot; cy=&quot;2388.08&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2060.01&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2211.61&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;1976.72&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2317.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1877.35&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2042.43&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2171.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;2354.54&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2992.77&quot; cy=&quot;2291.30&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;1935.88&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2025.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;953.12&quot; cy=&quot;2086.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;1470.07&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;1997.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1061.65&quot; cy=&quot;2106.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;1776.49&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2017.06&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;1847.27&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;1962.24&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2042.43&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2147.01&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;1909.40&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;2260.86&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;1936.87&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;2248.98&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;1925.49&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;1778.96&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;2088.47&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;1976.72&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;974.90&quot; cy=&quot;2011.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;2045.53&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;1911.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;1917.69&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;1958.65&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2301.20&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;1977.71&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1964.84&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2175.96&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2099.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2287.22&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2230.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2289.82&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2222.00&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;1993.30&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2122.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;1992.81&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;2174.97&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2158.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;996.56&quot; cy=&quot;2052.83&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2100.97&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;2145.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2007.78&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2060.01&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;888.03&quot; cy=&quot;2277.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1039.99&quot; cy=&quot;2267.05&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2063.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;2243.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;1854.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1018.22&quot; cy=&quot;2257.77&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2298.11&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2035.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2201.83&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2281.53&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2161.98&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;2224.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1061.65&quot; cy=&quot;2078.20&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2042.93&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2347.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1284.32&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2386.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;1996.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;931.47&quot; cy=&quot;2241.18&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;996.56&quot; cy=&quot;1968.43&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2175.96&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;974.90&quot; cy=&quot;2276.82&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;1997.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1039.99&quot; cy=&quot;2240.69&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;1604.59&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1039.99&quot; cy=&quot;1932.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;996.56&quot; cy=&quot;2311.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2399.46&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;2086.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;996.56&quot; cy=&quot;2037.24&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;2050.23&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2129.43&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2150.10&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1039.99&quot; cy=&quot;1874.75&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2272.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1061.65&quot; cy=&quot;2247.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;1698.27&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;1797.65&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;931.47&quot; cy=&quot;2214.20&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2156.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;1994.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;1967.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1061.65&quot; cy=&quot;2259.25&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1061.65&quot; cy=&quot;2108.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1869.05&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2269.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2138.22&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2082.78&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2079.19&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2039.83&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2323.97&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1018.22&quot; cy=&quot;2195.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;1932.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;974.90&quot; cy=&quot;2224.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;1446.81&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2137.11&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1018.22&quot; cy=&quot;1952.34&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;974.90&quot; cy=&quot;2301.70&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1756.01&quot; cy=&quot;1546.18&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;1402.75&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;2127.82&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1905.81&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2397.48&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;996.56&quot; cy=&quot;2107.16&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;996.56&quot; cy=&quot;2123.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2215.81&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;1379.98&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1061.65&quot; cy=&quot;1971.52&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;1701.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2160.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1275.53&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1898.51&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1039.99&quot; cy=&quot;2194.03&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;931.47&quot; cy=&quot;2227.20&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;1078.76&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;1912.49&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;2022.76&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2025.85&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;997.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2016.38&quot; cy=&quot;1152.77&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2363.50&quot; cy=&quot;1604.10&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;1299.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;996.56&quot; cy=&quot;1972.02&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;1941.57&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;1502.12&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;1101.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;439.20&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2023.25&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;1738.12&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1849.38&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1365.01&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;1677.61&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1018.22&quot; cy=&quot;1777.97&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2038.04&quot; cy=&quot;338.84&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2076.09&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2170.77&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;1123.81&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;1943.55&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;974.90&quot; cy=&quot;1964.35&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2002.09&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;1736.64&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2129.93&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;1800.25&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;1301.40&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;1496.93&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2129.93&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;2093.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;1369.09&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1083.31&quot; cy=&quot;885.34&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1929.63&quot; cy=&quot;2255.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1758.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;1865.47&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;1847.27&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;1710.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;2062.11&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2472.03&quot; cy=&quot;2317.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1994.73&quot; cy=&quot;1901.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2093.67&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2623.99&quot; cy=&quot;2216.80&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2103.13&quot; cy=&quot;2026.35&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1994.73&quot; cy=&quot;1957.54&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2103.13&quot; cy=&quot;1832.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;2266.55&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2069.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1256.93&quot; cy=&quot;2075.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2049.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;2086.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2124.91&quot; cy=&quot;2289.82&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2081.47&quot; cy=&quot;1837.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;2013.97&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;2005.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2016.38&quot; cy=&quot;1948.26&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1994.73&quot; cy=&quot;1988.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2190.00&quot; cy=&quot;1696.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;2007.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2358.62&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;1892.32&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2124.91&quot; cy=&quot;2173.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1923.38&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1894.92&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2124.91&quot; cy=&quot;2001.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2028.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2016.38&quot; cy=&quot;1964.84&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1734.23&quot; cy=&quot;1973.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;1995.90&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2113.84&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2059.82&quot; cy=&quot;2075.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1971.03&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2010.38&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1512.02&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;1829.21&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2059.82&quot; cy=&quot;1833.91&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;1518.71&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2016.38&quot; cy=&quot;2203.93&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1951.29&quot; cy=&quot;1990.70&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2244.77&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;1812.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2060.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1756.01&quot; cy=&quot;1919.30&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;1519.20&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2022.76&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2103.13&quot; cy=&quot;1607.19&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;1986.00&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1662.63&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;1818.81&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;1937.37&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;1663.62&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1651.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;2134.51&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;613.08&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2124.91&quot; cy=&quot;1492.84&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1836.51&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1723.64&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1972.95&quot; cy=&quot;1696.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;1406.46&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2007.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;1126.41&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;1967.93&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2296.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1962.24&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1486.66&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;1869.05&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2063.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;1993.30&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2146.57&quot; cy=&quot;1915.09&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2164.58&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;1954.94&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;1472.18&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;1788.86&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2182.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;1872.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1821.11&quot; cy=&quot;1957.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;1963.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1799.45&quot; cy=&quot;2209.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2016.38&quot; cy=&quot;2099.36&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1962.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;1946.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;1919.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2233.32&quot; cy=&quot;1883.04&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;2203.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;1771.78&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;1318.97&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;2216.80&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;1988.11&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2107.65&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1842.76&quot; cy=&quot;2338.95&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2081.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2287.22&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;2153.69&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2379.29&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1582.39&quot; cy=&quot;2166.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2304.79&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1999.00&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2212.72&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2111.24&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2287.71&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2137.60&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;1911.01&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2172.87&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2104.06&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2098.87&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1886.20&quot; cy=&quot;2260.86&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2170.77&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2218.41&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2231.28&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2292.42&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2233.32&quot; cy=&quot;1965.83&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2296.50&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2104.06&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;1968.92&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1625.83&quot; cy=&quot;2056.42&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2275.34&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2034.64&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2338.45&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1756.01&quot; cy=&quot;2108.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2086.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2090.08&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2176.95&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1864.54&quot; cy=&quot;2100.35&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1756.01&quot; cy=&quot;2257.77&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;1983.40&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2175.47&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;2200.84&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;1930.19&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1452.21&quot; cy=&quot;2072.51&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;2107.16&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2072.51&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2226.70&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1712.58&quot; cy=&quot;2262.34&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2100.97&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1647.48&quot; cy=&quot;2122.13&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1690.92&quot; cy=&quot;2045.53&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;2082.28&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2050.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1907.85&quot; cy=&quot;2245.27&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1517.30&quot; cy=&quot;2187.84&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2056.42&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1842.76&quot; cy=&quot;2168.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1560.73&quot; cy=&quot;2103.44&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1322.02&quot; cy=&quot;2086.99&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2213.21&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2266.06&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1495.64&quot; cy=&quot;2239.08&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1777.67&quot; cy=&quot;2105.55&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2128.81&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1886.20&quot; cy=&quot;2203.93&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1343.68&quot; cy=&quot;2082.78&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1278.59&quot; cy=&quot;2057.90&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1538.95&quot; cy=&quot;2088.47&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1669.14&quot; cy=&quot;2259.25&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1430.55&quot; cy=&quot;2093.17&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1604.05&quot; cy=&quot;2033.15&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1018.22&quot; cy=&quot;2145.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1365.46&quot; cy=&quot;2185.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;909.69&quot; cy=&quot;1941.57&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1170.18&quot; cy=&quot;2005.68&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1191.84&quot; cy=&quot;2033.65&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1061.65&quot; cy=&quot;2346.25&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1387.11&quot; cy=&quot;2092.06&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2021.27&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;1726.74&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1126.74&quot; cy=&quot;2070.90&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1300.37&quot; cy=&quot;2067.80&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;1963.73&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;1960.14&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;1972.02&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1408.77&quot; cy=&quot;2031.54&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1148.40&quot; cy=&quot;1907.91&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;844.59&quot; cy=&quot;2119.53&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;996.56&quot; cy=&quot;1922.89&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;996.56&quot; cy=&quot;1997.39&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1105.09&quot; cy=&quot;1918.18&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1235.27&quot; cy=&quot;1886.63&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1213.49&quot; cy=&quot;2054.81&quot; r=&quot;14.97&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;line x1=&quot;390.80&quot; y1=&quot;2489.19&quot; x2=&quot;390.80&quot; y2=&quot;275.35&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;line x1=&quot;390.80&quot; y1=&quot;2425.82&quot; x2=&quot;350.83&quot; y2=&quot;2425.82&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;text x=&quot;300.72&quot; y=&quot;2425.82&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; transform=&quot;rotate(-90 300.72,2425.82)&quot; text-anchor=&quot;middle&quot;&gt;0&lt;/text&gt;
	&lt;line x1=&quot;390.80&quot; y1=&quot;1908.28&quot; x2=&quot;350.83&quot; y2=&quot;1908.28&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;text x=&quot;300.72&quot; y=&quot;1908.28&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; transform=&quot;rotate(-90 300.72,1908.28)&quot; text-anchor=&quot;middle&quot;&gt;100&lt;/text&gt;
	&lt;line x1=&quot;390.80&quot; y1=&quot;1390.87&quot; x2=&quot;350.83&quot; y2=&quot;1390.87&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;text x=&quot;300.72&quot; y=&quot;1390.87&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; transform=&quot;rotate(-90 300.72,1390.87)&quot; text-anchor=&quot;middle&quot;&gt;200&lt;/text&gt;
	&lt;line x1=&quot;390.80&quot; y1=&quot;873.33&quot; x2=&quot;350.83&quot; y2=&quot;873.33&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;text x=&quot;300.72&quot; y=&quot;873.33&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; transform=&quot;rotate(-90 300.72,873.33)&quot; text-anchor=&quot;middle&quot;&gt;300&lt;/text&gt;
	&lt;line x1=&quot;390.80&quot; y1=&quot;355.79&quot; x2=&quot;350.83&quot; y2=&quot;355.79&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;text x=&quot;300.72&quot; y=&quot;355.79&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; transform=&quot;rotate(-90 300.72,355.79)&quot; text-anchor=&quot;middle&quot;&gt;400&lt;/text&gt;
	&lt;text x=&quot;190.71&quot; y=&quot;1382.33&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; transform=&quot;rotate(-90 190.71,1382.33)&quot; text-anchor=&quot;middle&quot;&gt;Prescriptions Per 100 People&lt;/text&gt;
	&lt;line x1=&quot;390.80&quot; y1=&quot;2489.19&quot; x2=&quot;3859.02&quot; y2=&quot;2489.19&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;line x1=&quot;454.16&quot; y1=&quot;2489.19&quot; x2=&quot;454.16&quot; y2=&quot;2529.16&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;text x=&quot;454.16&quot; y=&quot;2619.14&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; text-anchor=&quot;middle&quot;&gt;0&lt;/text&gt;
	&lt;line x1=&quot;1539.08&quot; y1=&quot;2489.19&quot; x2=&quot;1539.08&quot; y2=&quot;2529.16&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;text x=&quot;1539.08&quot; y=&quot;2619.14&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; text-anchor=&quot;middle&quot;&gt;5&lt;/text&gt;
	&lt;line x1=&quot;2623.99&quot; y1=&quot;2489.19&quot; x2=&quot;2623.99&quot; y2=&quot;2529.16&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;text x=&quot;2623.99&quot; y=&quot;2619.14&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; text-anchor=&quot;middle&quot;&gt;10&lt;/text&gt;
	&lt;line x1=&quot;3708.91&quot; y1=&quot;2489.19&quot; x2=&quot;3708.91&quot; y2=&quot;2529.16&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;text x=&quot;3708.91&quot; y=&quot;2619.14&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; text-anchor=&quot;middle&quot;&gt;15&lt;/text&gt;
	&lt;text x=&quot;2124.91&quot; y=&quot;2729.16&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; text-anchor=&quot;middle&quot;&gt;Unemployment Rate&lt;/text&gt;
	&lt;text x=&quot;2124.91&quot; y=&quot;215.98&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:139.96px;fill:#1E2D53&quot; text-anchor=&quot;middle&quot;&gt;Relation between Unemployment and Opioids&lt;/text&gt;
&lt;/svg&gt;
</body></html>"></iframe>



I'm not sure about you but it is difficult for me to see any relationship in this graph. It looks perhaps vaguely that increases in unemployment rates are associated with higher prescription rates, but here hard to tell given how many counties there are. What we are going to do next is to use a dimension reduction technique that will allow us to better visualize the relationship between unemployment and prescription rates.

What we are going to do this is to place counties into bins based on the unemployment rate and then compute the average prescription rate within each bin. This will also introduce a number of commands that can be very helpful with data wrangling. 

The first command we are going to use is the ``cut`` command. This will ``cut`` a numeric variable into a number of different bins. 


```stata
egen bin_u = cut(urate), group(10)
```


```stata
tab bin_u
```

    
          bin_u |      Freq.     Percent        Cum.
    ------------+-----------------------------------
              0 |        261        9.55        9.55
              1 |        232        8.49       18.04
              2 |        319       11.67       29.71
              3 |        265        9.70       39.41
              4 |        240        8.78       48.19
              5 |        279       10.21       58.40
              6 |        277       10.14       68.53
              7 |        295       10.79       79.33
              8 |        274       10.03       89.35
              9 |        291       10.65      100.00
    ------------+-----------------------------------
          Total |      2,733      100.00


The option ``group(10)`` indicated we wanted 10 different bins. There is not a hard science in computing the number of bins, it will often depend on what variable you are looking at. Choosing 10 here is more or less arbitrary.

Now we are going to ``collapse`` the data so that we have a single observation per value of ``bin_u``. ``collapse`` can be extremely useful as it allows you to change the unit of observation of your dataset quickly. For example, imagine we want to go from counties to states, from individuals to firms, from students to schools. Collapse will allow you to quickly transform the unit of observation in your dataset. In our data, we would like an observation for each bin.


```stata
collapse (mean) prescrip_rate urate, by(bin_u)
```


```stata
%head
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
      <th>bin_u</th>
      <th>prescrip_rate</th>
      <th>urate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>68.59348659003832</td>
      <td>2.784674329501916</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>70.88275862068967</td>
      <td>3.351293103448272</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2</td>
      <td>73.89278996865207</td>
      <td>3.818808777429465</td>
    </tr>
    <tr>
      <th>4</th>
      <td>3</td>
      <td>70.92981132075475</td>
      <td>4.203396226415093</td>
    </tr>
    <tr>
      <th>5</th>
      <td>4</td>
      <td>82.70625000000004</td>
      <td>4.502500000000001</td>
    </tr>
    <tr>
      <th>6</th>
      <td>5</td>
      <td>81.48136200716843</td>
      <td>4.847311827956991</td>
    </tr>
    <tr>
      <th>7</th>
      <td>6</td>
      <td>80.91768953068592</td>
      <td>5.24765342960289</td>
    </tr>
    <tr>
      <th>8</th>
      <td>7</td>
      <td>88.80711864406781</td>
      <td>5.710508474576269</td>
    </tr>
    <tr>
      <th>9</th>
      <td>8</td>
      <td>93.96167883211683</td>
      <td>6.480656934306568</td>
    </tr>
    <tr>
      <th>10</th>
      <td>9</td>
      <td>92.04261168384876</td>
      <td>8.413402061855663</td>
    </tr>
  </tbody>
</table>
</div>


To break down the command above ``(mean)`` indicates that STATA will compute the average value of ``prescrip_rate`` and ``urate`` within each ``bin_u`` and then collapse the data so that there is a single observation for each value of ``bin_u``. Therefore, for ``bin_u=0`` the average unemployment rate is 2.8 while the average prescriptions per 100 people is 68.7. Moving to ``bin_u=1`` the average unemployment rate is 3.4 while the average prescriptions per 100 people is 70.9. To visualize the data, we can try the scatterplot again, but with the bins.


```stata
twoway scatter prescrip_rate urate, msymbol(circle_hollow) msize(large) ///
    title("Relationship Between Unemployment and Opioids") ///
    xtitle("Unemployment Rate") ///
    ytitle("Prescriptions Per 100 People") ///
    graphregion(color(white) fcolor(white)) 
```


                <iframe frameborder="0" scrolling="no" height="436" width="600"                srcdoc="<html><body>&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot; standalone=&quot;no&quot;?&gt;
&lt;!-- This is a Stata 16.1 generated SVG file (http://www.stata.com) --&gt;

&lt;svg version=&quot;1.1&quot; width=&quot;600px&quot; height=&quot;436px&quot; viewBox=&quot;0 0 3960 2880&quot; xmlns=&quot;http://www.w3.org/2000/svg&quot; xmlns:xlink=&quot;http://www.w3.org/1999/xlink&quot;&gt;
	&lt;desc&gt;Stata Graph - Graph&lt;/desc&gt;
	&lt;rect x=&quot;0&quot; y=&quot;0&quot; width=&quot;3960&quot; height=&quot;2880&quot; style=&quot;fill:#EAF2F3;stroke:none&quot;/&gt;
	&lt;rect x=&quot;0.00&quot; y=&quot;0.00&quot; width=&quot;3959.88&quot; height=&quot;2880.00&quot; style=&quot;fill:#FFFFFF&quot;/&gt;
	&lt;rect x=&quot;2.88&quot; y=&quot;2.88&quot; width=&quot;3954.12&quot; height=&quot;2874.24&quot; style=&quot;fill:none;stroke:#FFFFFF;stroke-width:5.76&quot;/&gt;
	&lt;rect x=&quot;390.80&quot; y=&quot;275.35&quot; width=&quot;3468.22&quot; height=&quot;2213.83&quot; style=&quot;fill:#FFFFFF&quot;/&gt;
	&lt;rect x=&quot;393.68&quot; y=&quot;278.23&quot; width=&quot;3462.46&quot; height=&quot;2208.07&quot; style=&quot;fill:none;stroke:#FFFFFF;stroke-width:5.76&quot;/&gt;
	&lt;line x1=&quot;390.80&quot; y1=&quot;2314.69&quot; x2=&quot;3859.02&quot; y2=&quot;2314.69&quot; style=&quot;stroke:#EAF2F3;stroke-width:8.64&quot;/&gt;
	&lt;line x1=&quot;390.80&quot; y1=&quot;1919.55&quot; x2=&quot;3859.02&quot; y2=&quot;1919.55&quot; style=&quot;stroke:#EAF2F3;stroke-width:8.64&quot;/&gt;
	&lt;line x1=&quot;390.80&quot; y1=&quot;1524.28&quot; x2=&quot;3859.02&quot; y2=&quot;1524.28&quot; style=&quot;stroke:#EAF2F3;stroke-width:8.64&quot;/&gt;
	&lt;line x1=&quot;390.80&quot; y1=&quot;1129.13&quot; x2=&quot;3859.02&quot; y2=&quot;1129.13&quot; style=&quot;stroke:#EAF2F3;stroke-width:8.64&quot;/&gt;
	&lt;line x1=&quot;390.80&quot; y1=&quot;733.86&quot; x2=&quot;3859.02&quot; y2=&quot;733.86&quot; style=&quot;stroke:#EAF2F3;stroke-width:8.64&quot;/&gt;
	&lt;line x1=&quot;390.80&quot; y1=&quot;338.71&quot; x2=&quot;3859.02&quot; y2=&quot;338.71&quot; style=&quot;stroke:#EAF2F3;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;454.04&quot; cy=&quot;2425.95&quot; r=&quot;35.02&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;790.51&quot; cy=&quot;2245.02&quot; r=&quot;35.02&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1067.96&quot; cy=&quot;2007.04&quot; r=&quot;35.02&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1296.28&quot; cy=&quot;2241.31&quot; r=&quot;35.02&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1473.86&quot; cy=&quot;1310.43&quot; r=&quot;35.02&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1678.55&quot; cy=&quot;1407.33&quot; r=&quot;35.02&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;1916.27&quot; cy=&quot;1451.88&quot; r=&quot;35.02&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2190.99&quot; cy=&quot;828.28&quot; r=&quot;35.02&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;2648.25&quot; cy=&quot;420.89&quot; r=&quot;35.02&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;circle cx=&quot;3795.66&quot; cy=&quot;572.49&quot; r=&quot;35.02&quot; style=&quot;fill:none;stroke:#1A476F;stroke-width:8.64&quot;/&gt;
	&lt;line x1=&quot;390.80&quot; y1=&quot;2489.19&quot; x2=&quot;390.80&quot; y2=&quot;275.35&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;line x1=&quot;390.80&quot; y1=&quot;2314.69&quot; x2=&quot;350.83&quot; y2=&quot;2314.69&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;text x=&quot;300.72&quot; y=&quot;2314.69&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; transform=&quot;rotate(-90 300.72,2314.69)&quot; text-anchor=&quot;middle&quot;&gt;70&lt;/text&gt;
	&lt;line x1=&quot;390.80&quot; y1=&quot;1919.55&quot; x2=&quot;350.83&quot; y2=&quot;1919.55&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;text x=&quot;300.72&quot; y=&quot;1919.55&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; transform=&quot;rotate(-90 300.72,1919.55)&quot; text-anchor=&quot;middle&quot;&gt;75&lt;/text&gt;
	&lt;line x1=&quot;390.80&quot; y1=&quot;1524.28&quot; x2=&quot;350.83&quot; y2=&quot;1524.28&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;text x=&quot;300.72&quot; y=&quot;1524.28&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; transform=&quot;rotate(-90 300.72,1524.28)&quot; text-anchor=&quot;middle&quot;&gt;80&lt;/text&gt;
	&lt;line x1=&quot;390.80&quot; y1=&quot;1129.13&quot; x2=&quot;350.83&quot; y2=&quot;1129.13&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;text x=&quot;300.72&quot; y=&quot;1129.13&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; transform=&quot;rotate(-90 300.72,1129.13)&quot; text-anchor=&quot;middle&quot;&gt;85&lt;/text&gt;
	&lt;line x1=&quot;390.80&quot; y1=&quot;733.86&quot; x2=&quot;350.83&quot; y2=&quot;733.86&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;text x=&quot;300.72&quot; y=&quot;733.86&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; transform=&quot;rotate(-90 300.72,733.86)&quot; text-anchor=&quot;middle&quot;&gt;90&lt;/text&gt;
	&lt;line x1=&quot;390.80&quot; y1=&quot;338.71&quot; x2=&quot;350.83&quot; y2=&quot;338.71&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;text x=&quot;300.72&quot; y=&quot;338.71&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; transform=&quot;rotate(-90 300.72,338.71)&quot; text-anchor=&quot;middle&quot;&gt;95&lt;/text&gt;
	&lt;text x=&quot;190.71&quot; y=&quot;1382.33&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; transform=&quot;rotate(-90 190.71,1382.33)&quot; text-anchor=&quot;middle&quot;&gt;Prescriptions Per 100 People&lt;/text&gt;
	&lt;line x1=&quot;390.80&quot; y1=&quot;2489.19&quot; x2=&quot;3859.02&quot; y2=&quot;2489.19&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;line x1=&quot;582.00&quot; y1=&quot;2489.19&quot; x2=&quot;582.00&quot; y2=&quot;2529.16&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;text x=&quot;582.00&quot; y=&quot;2619.14&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; text-anchor=&quot;middle&quot;&gt;3&lt;/text&gt;
	&lt;line x1=&quot;1175.62&quot; y1=&quot;2489.19&quot; x2=&quot;1175.62&quot; y2=&quot;2529.16&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;text x=&quot;1175.62&quot; y=&quot;2619.14&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; text-anchor=&quot;middle&quot;&gt;4&lt;/text&gt;
	&lt;line x1=&quot;1769.25&quot; y1=&quot;2489.19&quot; x2=&quot;1769.25&quot; y2=&quot;2529.16&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;text x=&quot;1769.25&quot; y=&quot;2619.14&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; text-anchor=&quot;middle&quot;&gt;5&lt;/text&gt;
	&lt;line x1=&quot;2363.01&quot; y1=&quot;2489.19&quot; x2=&quot;2363.01&quot; y2=&quot;2529.16&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;text x=&quot;2363.01&quot; y=&quot;2619.14&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; text-anchor=&quot;middle&quot;&gt;6&lt;/text&gt;
	&lt;line x1=&quot;2956.64&quot; y1=&quot;2489.19&quot; x2=&quot;2956.64&quot; y2=&quot;2529.16&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;text x=&quot;2956.64&quot; y=&quot;2619.14&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; text-anchor=&quot;middle&quot;&gt;7&lt;/text&gt;
	&lt;line x1=&quot;3550.26&quot; y1=&quot;2489.19&quot; x2=&quot;3550.26&quot; y2=&quot;2529.16&quot; style=&quot;stroke:#000000;stroke-width:5.76&quot;/&gt;
	&lt;text x=&quot;3550.26&quot; y=&quot;2619.14&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; text-anchor=&quot;middle&quot;&gt;8&lt;/text&gt;
	&lt;text x=&quot;2124.91&quot; y=&quot;2729.16&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:99.99px;fill:#000000&quot; text-anchor=&quot;middle&quot;&gt;Unemployment Rate&lt;/text&gt;
	&lt;text x=&quot;2124.91&quot; y=&quot;215.98&quot; style=&quot;font-family:&#x27;Helvetica&#x27;;font-size:139.96px;fill:#1E2D53&quot; text-anchor=&quot;middle&quot;&gt;Relation between Unemployment and Opioids&lt;/text&gt;
&lt;/svg&gt;
</body></html>"></iframe>



Now it is clear that the prescription rates for opioids are higher in areas with higher unemployment, as shown in this figure. Again, it is important to stress that this relationship is by no means causal and the proper policy response likely depends on establishing causality. For example, imagine the goal of the government is to reduce opioid usage and they have two policies at their disposal. Policy one is to increase funding for prescription monitoring programs which can identify inappropriate prescription behavior by doctors. Policy two is to increase local employment subsidies increase the employment rates, which will in turn decrease opioid abuse.

Whether policy one or two is effective depends on the main driver of the epidemic (not to mention the implementation details of each). If, for the sake of argument, the high prescription rates in areas with high unemployment is due completely to physician's being more likely to prescribe opioids in these regions (for whatever reason), then the employment subsidies would likely be ineffective at reducing opioid usage. Instead, one should use the monitoring program to identify and dissuade inappropriate prescribing behavior. 
