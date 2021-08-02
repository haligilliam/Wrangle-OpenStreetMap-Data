# Wrangle-OpenStreetMap-Data


## Area of Map Selected 
SeaTac, Washington 



## Data Auditing
I first took the downloaded OSM file and parsed with ElementTree to find the number of each type of element: 

```
import xml.etree.cElementTree as ET
import pprint

def count_tags(filename):
        tags = {}
        for k, v in ET.iterparse(filename):
            if v.tag in tags:
                tags[v.tag] +=1
            else:
                tags[v.tag] =1
        return tags

tags = count_tags('seatac_wa[1].osm')
pprint.pprint(tags)
```
```
{'bounds': 1,
 'member': 44856,
 'meta': 1,
 'nd': 1578182,
 'node': 1349314,
 'note': 1,
 'osm': 1,
 'relation': 1675,
 'tag': 929276,
 'way': 199637}
 ```

I then wanted to further focus on Street Names and Zip Codes. Upon examination of a sample of the Seatacc area data, I came across some cases of the following issues: 

* Issue 1: Inconsistent Street Names
* Issue 2: Inconsistent zip codes

### Inconsistent Street Names
To audit and clean the street names in this data set, I created an expected street type list, such as Street instead of St., and created a dictionary for all street types no in the expected street types dictionary. To correct any inconsistensies between some addresses displaying as St. and some dispalying as Street, I created a street_name_cleaning dictionary to change any abbreviations to the full words: 
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
While looking at zip code data, I noticed that some zip codes seem to belong outside of the SeaTac lis to city zip codes. I also wanted to ensure that the zip codes followed a consistent 5-digit format, instead of a mix of the 5-digit and the full 9-digit postal zip code. To clean this for the first part, I created a list of expected zip codes of zip codes belonging to SeaTac. 
```
expected =  ['98148',
'98158',
'98168',
'98188',
'98198']
```
To clean for a consistent digit format, I looked for any zip code that had 9 digits, and only returned the 5 left-most digits.
```
elif len(name) == 10: #If the zip is not 5-digit, take the left-most five digits
        return name[0:5]
```
## Overview of the Data
### Number of Nodes: 
```
SELECT COUNT(*)FROM nodes;
```
```
1349314
```
### Number of Ways: 
```
1996
```
### Max Speed Limits 
Most speed limits seemed normal for the different areas of roads. However, one did stand out at 8mph, which I assumed to be a typo as I only had 1 result.
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
### Number of Unique Users
```
SELECT COUNT(DISTINCT(u.nodeid))
FROM (SELECT nodeid FROM nodes UNION ALL
SELECT wayid FROM ways) as u;
```
```
2005
```
