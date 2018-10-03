# DBRS Tech Assessment Process Notes:

### SOURCING THE DATA
I started my analysis by locally storing the 2017 311 complaint record sub-set provided at the following link in the description of this assessment. 
<https://s3.amazonaws.com/dbrs-recruit/2017_subset.csv>
For production use, I would rely on the Socrata API to pull the data programmatically. For testing and validation the local file made the most sense so I would not trip any API-call limits and throttling restrictions mentioned in the NYC Open Data developer notes. I used filtering arguments in the Socrata API call to ensure only data for 2017, and only the necessary fields would be pulled with any given API-call:

`field_list = ','.join(['unique_key','created_date','borough','incident_zip','city','complaint_type'])`

`results = client.get("fhrw-4uyv",limit,select=field_list,where="created_date='2017'")`



### VALIDATION
Next, I validated the data for gaps and noticed most glaringly the number of complaints unattributable to any borough. I noticed that most had zip codes included, which meant using a separate dimension table to merge into the main data set would solve most of these issues. 

### MAPPING THE MISSING BOROUGHS
I was able to locate a website that had a zip code to borough mapping, and saved this locally as tabular data in a csv. The URL of that website is: <https://www.nycbynatives.com/nyc_info/new_york_city_zip_codes.php>
And the file is saved as **zip_to_borough.csv**

### POPULATION DATA
The last raw piece of data that would be necessary for questions A2 and A3 was the national population count for all zip codes as derived from the 2010 census. I downloaded this data set from the provided link and utilized it as a locally stored csv file: **ZCTA.csv**

### ENVIRONMENT
For slicing merging and analyzing this data I employed Jupyter notebook and heavily relied on the Pandas python library as I typically would have done for a project such as this. This methodology was also the one recommended in the assessment description.  

### DATA CLEANING
For generating a cleaned master data frame, I joined in the zip-code-to-borough dimension table mentioned above. I also noticed many records had improper text data in the zip code field or too many digits. For the differentiating the properly mapped borough field from the one brought in with the raw data, I renamed the old one as **borough_raw**. I used regex to strip out the unnecessary data where possible prior to performing the join, so that as many records as possible would join properly.

### RUNNING THE ANALYSIS
To find the results of the three questions from the assessment I broke out the result sets into three separate function. Running each of these three will return its respective dataframe to the notebook’s user to view on screen. I have also included statements to save the results as a local csv for later analysis. The three result files are:
1. top10_by_borough.csv
2. top10_by_top_zips.csv
3. borough_complaint_indices.csv

### DEV mode vs. PRODUCTION MODE 
Since the Socrata API will use up some of the user's daily API call limit, I included an option for the user to pull the live data from the NYC Open Data site or pull from the local file. The default mode is DEV mode. To pull the live data change the below variable in the notebook to True:

`dev_mode = True`

# ANALYZING THE RESULTS

## A1: TOP 10 COMPLAINT TYPES PER BOROUGH

> Objective: Consider only the 10 most common overall complaint types. For each borough, how many of each of those 10 types were there in 2017?

**PROCEDURE:**
For this task I used pandas to group all records on the BOROUGH field I mapped in the data-cleaning step. I extracted only the top 10 complaint types and creating a function that returned the resultant data-frame. 
The findings are displayed below:

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>Complaint Count</th>
    </tr>
    <tr>
      <th>BOROUGH</th>
      <th>complaint_type</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="10" valign="top">BRONX</th>
      <th>HEAT/HOT WATER</th>
      <td>69085</td>
    </tr>
    <tr>
      <th>Noise - Residential</th>
      <td>57928</td>
    </tr>
    <tr>
      <th>UNSANITARY CONDITION</th>
      <td>24730</td>
    </tr>
    <tr>
      <th>Blocked Driveway</th>
      <td>24632</td>
    </tr>
    <tr>
      <th>PAINT/PLASTER</th>
      <td>19712</td>
    </tr>
    <tr>
      <th>PLUMBING</th>
      <td>16581</td>
    </tr>
    <tr>
      <th>Illegal Parking</th>
      <td>16244</td>
    </tr>
    <tr>
      <th>Noise - Street/Sidewalk</th>
      <td>14109</td>
    </tr>
    <tr>
      <th>DOOR/WINDOW</th>
      <td>11914</td>
    </tr>
    <tr>
      <th>Street Condition</th>
      <td>11181</td>
    </tr>
    <tr>
      <th rowspan="10" valign="top">BROOKLYN</th>
      <th>Noise - Residential</th>
      <td>67677</td>
    </tr>
    <tr>
      <th>HEAT/HOT WATER</th>
      <td>66977</td>
    </tr>
    <tr>
      <th>Illegal Parking</th>
      <td>55475</td>
    </tr>
    <tr>
      <th>Blocked Driveway</th>
      <td>49390</td>
    </tr>
    <tr>
      <th>UNSANITARY CONDITION</th>
      <td>26654</td>
    </tr>
    <tr>
      <th>Street Condition</th>
      <td>24889</td>
    </tr>
    <tr>
      <th>Noise - Street/Sidewalk</th>
      <td>21321</td>
    </tr>
    <tr>
      <th>Water System</th>
      <td>19511</td>
    </tr>
    <tr>
      <th>PAINT/PLASTER</th>
      <td>19398</td>
    </tr>
    <tr>
      <th>Request Large Bulky Item Collection</th>
      <td>16793</td>
    </tr>
    <tr>
      <th rowspan="10" valign="top">MANHATTAN</th>
      <th>Noise - Residential</th>
      <td>50228</td>
    </tr>
    <tr>
      <th>HEAT/HOT WATER</th>
      <td>45535</td>
    </tr>
    <tr>
      <th>Noise - Street/Sidewalk</th>
      <td>28772</td>
    </tr>
    <tr>
      <th>Noise</th>
      <td>27924</td>
    </tr>
    <tr>
      <th>Illegal Parking</th>
      <td>19112</td>
    </tr>
    <tr>
      <th>Noise - Commercial</th>
      <td>18015</td>
    </tr>
    <tr>
      <th>Homeless Person Assistance</th>
      <td>16703</td>
    </tr>
    <tr>
      <th>UNSANITARY CONDITION</th>
      <td>14187</td>
    </tr>
    <tr>
      <th>Street Condition</th>
      <td>13670</td>
    </tr>
    <tr>
      <th>PAINT/PLASTER</th>
      <td>11165</td>
    </tr>
    <tr>
      <th rowspan="10" valign="top">QUEENS</th>
      <th>Blocked Driveway</th>
      <td>54251</td>
    </tr>
    <tr>
      <th>Noise - Residential</th>
      <td>46348</td>
    </tr>
    <tr>
      <th>Illegal Parking</th>
      <td>45908</td>
    </tr>
    <tr>
      <th>Street Condition</th>
      <td>29411</td>
    </tr>
    <tr>
      <th>HEAT/HOT WATER</th>
      <td>29171</td>
    </tr>
    <tr>
      <th>Request Large Bulky Item Collection</th>
      <td>21354</td>
    </tr>
    <tr>
      <th>Water System</th>
      <td>18234</td>
    </tr>
    <tr>
      <th>Street Light Condition</th>
      <td>18230</td>
    </tr>
    <tr>
      <th>Derelict Vehicle</th>
      <td>16066</td>
    </tr>
    <tr>
      <th>Building/Use</th>
      <td>13455</td>
    </tr>
    <tr>
      <th rowspan="10" valign="top">STATEN ISLAND</th>
      <th>Electronics Waste</th>
      <td>12292</td>
    </tr>
    <tr>
      <th>Street Condition</th>
      <td>10351</td>
    </tr>
    <tr>
      <th>Illegal Parking</th>
      <td>8060</td>
    </tr>
    <tr>
      <th>Noise - Residential</th>
      <td>7037</td>
    </tr>
    <tr>
      <th>Street Light Condition</th>
      <td>6658</td>
    </tr>
    <tr>
      <th>Missed Collection (All Materials)</th>
      <td>5565</td>
    </tr>
    <tr>
      <th>Water System</th>
      <td>5273</td>
    </tr>
    <tr>
      <th>Sewer</th>
      <td>3907</td>
    </tr>
    <tr>
      <th>Blocked Driveway</th>
      <td>3794</td>
    </tr>
    <tr>
      <th>Derelict Vehicle</th>
      <td>3783</td>
    </tr>
  </tbody>
</table>

## A2: COMPLAINT STATISTICS FOR 10 MOST POPULOUS ZIP CODES IN NYC>

>Consider only the 10 most common overall complaint types.  For the 10 most populous zip codes, how many of each of those 10 types were there in 2017?

**PROCEDURE:** 
I extracted a set of all unique zip codes appearing in the data and merged them with the population data from a local file. Next I joined these zip codes and populations with their respective complaint records. I then summed the top record counts for each, grouped on their zip code, and returned the resultant dataframe to the user.
The findings are displayed below:

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>Complaint Count</th>
    </tr>
    <tr>
      <th>incident_zip</th>
      <th>complaint_type</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="10" valign="top">10025</th>
      <th>HEAT/HOT WATER</th>
      <td>2397</td>
    </tr>
    <tr>
      <th>Noise - Residential</th>
      <td>2085</td>
    </tr>
    <tr>
      <th>Noise</th>
      <td>1398</td>
    </tr>
    <tr>
      <th>Noise - Street/Sidewalk</th>
      <td>1224</td>
    </tr>
    <tr>
      <th>Illegal Parking</th>
      <td>736</td>
    </tr>
    <tr>
      <th>UNSANITARY CONDITION</th>
      <td>714</td>
    </tr>
    <tr>
      <th>Dirty Conditions</th>
      <td>703</td>
    </tr>
    <tr>
      <th>Homeless Person Assistance</th>
      <td>677</td>
    </tr>
    <tr>
      <th>Rodent</th>
      <td>663</td>
    </tr>
    <tr>
      <th>Street Condition</th>
      <td>628</td>
    </tr>
    <tr>
      <th rowspan="10" valign="top">10467</th>
      <th>HEAT/HOT WATER</th>
      <td>6041</td>
    </tr>
    <tr>
      <th>Noise - Residential</th>
      <td>5807</td>
    </tr>
    <tr>
      <th>UNSANITARY CONDITION</th>
      <td>2192</td>
    </tr>
    <tr>
      <th>Blocked Driveway</th>
      <td>2068</td>
    </tr>
    <tr>
      <th>PAINT/PLASTER</th>
      <td>1955</td>
    </tr>
    <tr>
      <th>PLUMBING</th>
      <td>1442</td>
    </tr>
    <tr>
      <th>DOOR/WINDOW</th>
      <td>1043</td>
    </tr>
    <tr>
      <th>Illegal Parking</th>
      <td>986</td>
    </tr>
    <tr>
      <th>WATER LEAK</th>
      <td>899</td>
    </tr>
    <tr>
      <th>Noise - Street/Sidewalk</th>
      <td>713</td>
    </tr>
    <tr>
      <th rowspan="10" valign="top">11207</th>
      <th>Noise - Residential</th>
      <td>3061</td>
    </tr>
    <tr>
      <th>HEAT/HOT WATER</th>
      <td>2461</td>
    </tr>
    <tr>
      <th>Blocked Driveway</th>
      <td>2062</td>
    </tr>
    <tr>
      <th>UNSANITARY CONDITION</th>
      <td>1621</td>
    </tr>
    <tr>
      <th>Illegal Parking</th>
      <td>1500</td>
    </tr>
    <tr>
      <th>Derelict Vehicles</th>
      <td>1224</td>
    </tr>
    <tr>
      <th>Street Condition</th>
      <td>1142</td>
    </tr>
    <tr>
      <th>PAINT/PLASTER</th>
      <td>1055</td>
    </tr>
    <tr>
      <th>PLUMBING</th>
      <td>977</td>
    </tr>
    <tr>
      <th>Street Light Condition</th>
      <td>932</td>
    </tr>
    <tr>
      <th rowspan="10" valign="top">11208</th>
      <th>Noise - Residential</th>
      <td>2795</td>
    </tr>
    <tr>
      <th>Blocked Driveway</th>
      <td>2756</td>
    </tr>
    <tr>
      <th>Illegal Parking</th>
      <td>2150</td>
    </tr>
    <tr>
      <th>HEAT/HOT WATER</th>
      <td>2052</td>
    </tr>
    <tr>
      <th>UNSANITARY CONDITION</th>
      <td>1341</td>
    </tr>
    <tr>
      <th>Derelict Vehicles</th>
      <td>1187</td>
    </tr>
    <tr>
      <th>Noise - Street/Sidewalk</th>
      <td>825</td>
    </tr>
    <tr>
      <th>Street Condition</th>
      <td>816</td>
    </tr>
    <tr>
      <th>PAINT/PLASTER</th>
      <td>800</td>
    </tr>
    <tr>
      <th>PLUMBING</th>
      <td>782</td>
    </tr>
    <tr>
      <th rowspan="10" valign="top">11220</th>
      <th>Illegal Parking</th>
      <td>2013</td>
    </tr>
    <tr>
      <th>HEAT/HOT WATER</th>
      <td>1634</td>
    </tr>
    <tr>
      <th>Blocked Driveway</th>
      <td>1558</td>
    </tr>
    <tr>
      <th>Noise - Residential</th>
      <td>1522</td>
    </tr>
    <tr>
      <th>UNSANITARY CONDITION</th>
      <td>719</td>
    </tr>
    <tr>
      <th>Street Condition</th>
      <td>690</td>
    </tr>
    <tr>
      <th>Water System</th>
      <td>674</td>
    </tr>
    <tr>
      <th>Request Large Bulky Item Collection</th>
      <td>663</td>
    </tr>
    <tr>
      <th>Graffiti</th>
      <td>654</td>
    </tr>
    <tr>
      <th>Noise - Park</th>
      <td>636</td>
    </tr>
    <tr>
      <th rowspan="10" valign="top">11226</th>
      <th>HEAT/HOT WATER</th>
      <td>7569</td>
    </tr>
    <tr>
      <th>Noise - Residential</th>
      <td>4854</td>
    </tr>
    <tr>
      <th>UNSANITARY CONDITION</th>
      <td>3155</td>
    </tr>
    <tr>
      <th>PAINT/PLASTER</th>
      <td>2639</td>
    </tr>
    <tr>
      <th>Blocked Driveway</th>
      <td>2203</td>
    </tr>
    <tr>
      <th>PLUMBING</th>
      <td>2064</td>
    </tr>
    <tr>
      <th>Noise - Street/Sidewalk</th>
      <td>1831</td>
    </tr>
    <tr>
      <th>WATER LEAK</th>
      <td>1569</td>
    </tr>
    <tr>
      <th>Illegal Parking</th>
      <td>1076</td>
    </tr>
    <tr>
      <th>DOOR/WINDOW</th>
      <td>1063</td>
    </tr>
    <tr>
      <th rowspan="10" valign="top">11236</th>
      <th>Blocked Driveway</th>
      <td>3041</td>
    </tr>
    <tr>
      <th>Noise - Residential</th>
      <td>1929</td>
    </tr>
    <tr>
      <th>Derelict Vehicles</th>
      <td>1742</td>
    </tr>
    <tr>
      <th>Illegal Parking</th>
      <td>1431</td>
    </tr>
    <tr>
      <th>HEAT/HOT WATER</th>
      <td>1145</td>
    </tr>
    <tr>
      <th>Street Condition</th>
      <td>1021</td>
    </tr>
    <tr>
      <th>Sanitation Condition</th>
      <td>647</td>
    </tr>
    <tr>
      <th>Water System</th>
      <td>623</td>
    </tr>
    <tr>
      <th>UNSANITARY CONDITION</th>
      <td>562</td>
    </tr>
    <tr>
      <th>Sewer</th>
      <td>504</td>
    </tr>
    <tr>
      <th rowspan="10" valign="top">11368</th>
      <th>Blocked Driveway</th>
      <td>4384</td>
    </tr>
    <tr>
      <th>Noise - Residential</th>
      <td>2460</td>
    </tr>
    <tr>
      <th>HEAT/HOT WATER</th>
      <td>1620</td>
    </tr>
    <tr>
      <th>Illegal Parking</th>
      <td>1251</td>
    </tr>
    <tr>
      <th>Noise - Street/Sidewalk</th>
      <td>684</td>
    </tr>
    <tr>
      <th>UNSANITARY CONDITION</th>
      <td>639</td>
    </tr>
    <tr>
      <th>Water System</th>
      <td>617</td>
    </tr>
    <tr>
      <th>Derelict Vehicles</th>
      <td>569</td>
    </tr>
    <tr>
      <th>Street Condition</th>
      <td>561</td>
    </tr>
    <tr>
      <th>Street Light Condition</th>
      <td>444</td>
    </tr>
    <tr>
      <th rowspan="10" valign="top">11373</th>
      <th>HEAT/HOT WATER</th>
      <td>3408</td>
    </tr>
    <tr>
      <th>Blocked Driveway</th>
      <td>2635</td>
    </tr>
    <tr>
      <th>Noise - Residential</th>
      <td>1842</td>
    </tr>
    <tr>
      <th>Illegal Parking</th>
      <td>1277</td>
    </tr>
    <tr>
      <th>UNSANITARY CONDITION</th>
      <td>756</td>
    </tr>
    <tr>
      <th>Street Condition</th>
      <td>691</td>
    </tr>
    <tr>
      <th>Water System</th>
      <td>372</td>
    </tr>
    <tr>
      <th>Building/Use</th>
      <td>364</td>
    </tr>
    <tr>
      <th>Street Light Condition</th>
      <td>362</td>
    </tr>
    <tr>
      <th>Noise - Street/Sidewalk</th>
      <td>304</td>
    </tr>
    <tr>
      <th rowspan="10" valign="top">11385</th>
      <th>Illegal Parking</th>
      <td>4135</td>
    </tr>
    <tr>
      <th>Request Large Bulky Item Collection</th>
      <td>3465</td>
    </tr>
    <tr>
      <th>Blocked Driveway</th>
      <td>3042</td>
    </tr>
    <tr>
      <th>Noise - Residential</th>
      <td>2609</td>
    </tr>
    <tr>
      <th>HEAT/HOT WATER</th>
      <td>1526</td>
    </tr>
    <tr>
      <th>Missed Collection (All Materials)</th>
      <td>1274</td>
    </tr>
    <tr>
      <th>Water System</th>
      <td>1240</td>
    </tr>
    <tr>
      <th>Street Condition</th>
      <td>1232</td>
    </tr>
    <tr>
      <th>Dirty Conditions</th>
      <td>970</td>
    </tr>
    <tr>
      <th>Derelict Vehicle</th>
      <td>834</td>
    </tr>
  </tbody>
</table>


## A3: COMPLAINT INDICES FOR EACH BOROUGH

>Considering all complaint types. Which boroughs are the biggest "complainers" relative to the size of the population in 2017? Meaning, calculate a complaint-index that adjusts for population of the borough.

**PROCEDURE**
To determine a complaint index for each borough I decided I would simply divide the total complaints per Borough with their respective populations to achieve an index that could be described as *complaints per each member of the population*. I merged a table containing total population per borough with one containing total complaints per borough and then divided the first fields by the second to achieve the resultant index. I then returned this dataframe to the user.  
The findings are displayed below:

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>BOROUGH</th>
      <th>POPULATION</th>
      <th>TOTAL COMPLAINTS</th>
      <th>COMPLAINT INDEX</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>BRONX</td>
      <td>1382480.0</td>
      <td>434913</td>
      <td>0.314589</td>
    </tr>
    <tr>
      <th>1</th>
      <td>BROOKLYN</td>
      <td>2504700.0</td>
      <td>746780</td>
      <td>0.298151</td>
    </tr>
    <tr>
      <th>2</th>
      <td>MANHATTAN</td>
      <td>1518994.0</td>
      <td>450011</td>
      <td>0.296256</td>
    </tr>
    <tr>
      <th>3</th>
      <td>STATEN ISLAND</td>
      <td>468730.0</td>
      <td>127836</td>
      <td>0.272728</td>
    </tr>
    <tr>
      <th>4</th>
      <td>QUEENS</td>
      <td>2233454.0</td>
      <td>570146</td>
      <td>0.255275</td>
    </tr>
  </tbody>
</table>
