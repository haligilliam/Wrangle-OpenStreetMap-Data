# Wrangle-OpenStreetMap-Data


## Area of Map Selected 
SeaTac, Washington, United States
https://www.openstreetmap.org/relation/237380
I have never been to SeaTac, and randomly chose this location for auditing, cleaning and analysis, to see what interesting facts I could find out about the data! I chose SeaTac specifically because I had never heard of the city before and I hoped to learn a little more about it through this data wrangling process. 


## Problems encountered in map
While auditing the data there were two areas that caught my attention the most. 

* Issue 1: Inconsistent Street Names
* Issue 2: Inconsistent zip codes and zip codes that did not belong to SeaTac Washington.

###  Inconsistent Street Names
There were some inconsitencies in the way that street names were written in the data. Below we can see some examples of abbrevaited street names, where others had the full spelling. While neither of these options are an incorrect way to display a street name, it is best to clean and update to reflect one formatting standard. This helps with data consistency and allows for easier and more accurate analysis.  

* S 240th St would update to South 240th Street.
* 124th Ave would update to 124th Avenue. 
* Marine View Dr would update to Marine View Drive.
* Andover Park W would update to Andover Park West.
* Benson Rd S would update to Benson Road South.

To audit and clean the street names in this data set, I created an expected street type list, such as Street instead of St., and created a dictionary for all street types in the expected street types dictionary. To correct any inconsistensies between some addresses displaying as St. and some dispalying as Street, I created a street_name_cleaning dictionary to change any abbreviations to the full spellings. I used parts of the Udacity Nanodegree course module practice problems to craft the below auditing and cleaning. 


```
street_name_cleanup = {'Ave': 'Avenue', 'AVE': 'Avenue', 'Ave.': 'Avenue',
 'Blvd': 'Boulevard', 'Blvd.': 'Boulevard',
 'Cir': 'Circle', 'Cir.': 'Circle',
 'Crt': 'Court', 'Crt.': 'Court',
 'Ct': 'Court', 'Ct.': 'Court',
 'Dr': 'Drive', 'Dr.': 'Drive',
 'E.': 'East',
 'Fwy': 'Freeway', 'Fwy.': 'Freeway',
 'Hwy': 'Highway', 'Hwy.': 'Highway',
 'Ln': 'Lane', 'Ln.': 'Lane',
 'Mt': 'Mountain', 'Mt.': 'Mountain',
 'N.': 'North',
 'Pkwy': 'Parkway', 'Pkwy.': 'Parkway',
 'Pl': 'Place', 'Pl.': 'Place',
 'Pt': 'Point', 'Pt.': 'Point',
 'Rd': 'Road', 'Rd.': 'Road',
 'Rte': 'Route', 'Rte.': 'Route',
 'S.': 'South',
 'Sq': 'Square', 'Sq.': 'Square',
 'St': 'Street', 'St.': 'Street',
 'Ter': 'Terrace', 'Ter.': 'Terrace',
 'Tr': 'Trail', 'Tr.': 'Trail',
 'W.': 'West',
 'Wy': 'Way', 'Wy.': 'Way'}
 ```
 
### Inconsistent Zip Codes
While looking at zip code data, I noticed that some zip codes seem to belong outside of the SeaTac list for city zip codes. I also wanted to ensure that the zip codes followed a consistent 5-digit format, instead of a mix of the 5-digit and the full 9-digit postal zip code. 

I pulled a list of the postal codes for SeaTac, Washington, and only found the 6 listed below in the expected list. I then only looked at addresses containing a zip code found in the expected list. 

Some zip codes found that were not part of the list were for surrounding areas, such as Seattle, Washington.

* 98166
* 98146
* 98178

To clean for this, I created a list of expected zip codes of zip codes belonging to SeaTac. 
```
expected =  ['98032'
'98148',
'98158',
'98168',
'98188',
'98198']
```
I also wanted to make sure that the zip code format was consistent, and only displayed the 5-digit format, instead of a mix of 5-digit zip codes, and the full 9-digit zip codes. 

* 98032-1404 will update to 98032
* 98148-2029 will update to 98148

To clean for a consistent digit format, I looked for any zip code that had 9 digits, and only returned the 5 left-most digits.

## Overview of the Data

### Size of File
The SeaTac Washingting OSM File is 319 MB


Here, I have included the queries performed on the dataset using Microsoft SQL Server Management Studio, and any interesting things I found.

### Number of Unique Users
This is a decent amount of users making updates to the map data!

```
SELECT COUNT(DISTINCT(u.nodeid))
FROM (SELECT nodeid FROM nodes UNION ALL
SELECT wayid FROM ways) as u;
```
```
2005
```
### Number of Nodes: 

```
SELECT COUNT(*)FROM nodes;
```
```
1349314
```
### Number of Ways: 
```
SELECT COUNT(*)FROM ways;
```
```
1996
```
### Max Speed Limits 
Most speed limits seemed normal for the different areas of roads throughout the city. However, one did stand out at 8mph, which I assumed to be a typo as I only had 1 result.
```
with name_type AS (
SELECT
wtg.TagValue AS name,
wts.TagValue AS type
FROM
ways way
INNER JOIN waytags wts ON way.wayid = wts.wayid AND wts.TagName = 'maxspeed'
LEFT JOIN waytags wtg ON way.wayid = wtg.wayid AND wtg.TagName = 'Name'

)
```

```
SELECT  DISTINCT type FROM name_type
```
```
5 mph
8 mph
10 mph
15 mph
20 mph
30 mph
40 mph
45 mph
60 mph
```

## Other ideas about the dataset
The easiest way to keep data clean and consistent is to either limit the number of users, or to set a mandatory upload standard for all update types. For the OpenStreetMaps data, I would look into having a submission procedure where users have to follow one standard when uploading, or editing data. For example, when adding or updating street names, all users must type out the full words of "Street" or "Boulevard", and if the format is not followed then the entry would be rejected. With a lot of users uploading, and no standardization protocols in place, the chances for inconsistent data are high.  

Below we can see the results for the waytag name of "name"  These results showed a mixture of street names, and what appeared to be the names of the businesses belonging to that street name. 


```
72nd Avenue South
7th Place Southwest
84th Avenue South
97th Place South
Airport Plaza
Ambaum Boulevard South
Anacortes Court Northeast
Aqua Way South
Art Wright Field
B

```

# Conclusion 
The data for SeaTac, Washington did not seem to have a ton of errors, but there are some areas that needed some further auditing and cleaning. However, I believe that for the purposes of the project, it has been well cleaned. It was interesting to further analyze that the waytag values for the waytag name of "name" held both street addresses, and names of places. With a better system in place for accepting user entries, I beleive that the data for both SeaTac, Washington and global would increase in cleanliness!
