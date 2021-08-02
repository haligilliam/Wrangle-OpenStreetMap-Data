# Wrangle-OpenStreetMap-Data


## Area of Map Selected 
SeaTac, Washington 
https://www.openstreetmap.org/relation/237380
I have never been to SeaTac, and randomly chose this location for auditing, cleaning and analysis to see what interesting fact I could find out about the data!

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
To audit and clean the street names in this data set, I created an expected street type list, such as Street instead of St., and created a dictionary for all street types no in the expected street types dictionary. To correct any inconsistensies between some addresses displaying as St. and some dispalying as Street, I created a street_name_cleaning dictionary to change any abbreviations to the full words. The expected street type list also helped to clean up any street names that might have had mispellings. I used parts of the Udacity Nanodegree course module prsactice problems to craft the below auditing and cleaning. 

```
def audit_street_type(street_types, street_name):
    m = street_type_re.search(street_name)
    if m:
        street_type = m.group()
        if street_type not in expected:
            street_types[street_type].add(street_name)
```
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
While looking at zip code data, I noticed that some zip codes seem to belong outside of the SeaTac list for city zip codes. I also wanted to ensure that the zip codes followed a consistent 5-digit format, instead of a mix of the 5-digit and the full 9-digit postal zip code. To clean this for the first part, I created a list of expected zip codes of zip codes belonging to SeaTac. 
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
Here, I have included the queries performed on the dataset using Microsoft SQL Server Management Studio, and any interesting things I found.
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
Most speed limits seemed normal for the different areas of roads throughough the city. However, one did stand out at 8mph, which I assumed to be a typo as I only had 1 result.
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
This is a decent amount of users making updates to the map data!

```
SELECT COUNT(DISTINCT(u.nodeid))
FROM (SELECT nodeid FROM nodes UNION ALL
SELECT wayid FROM ways) as u;
```
```
2005
```
## Additional Ideas
The easiest way to keep data clean and consistent is to either limit the number of users, or to set a mandtory upload standard for all update types. For the OpenStreetMaps data, I would look into having a submission procedure where users have to follow one standard when uploading, or editing data. For example, when adding or updating street names, all users must type out the full words of "Street" or "Boulevard", and if the format is not followed then the entry would be rejected. With a lot of users uploading, and no standardization protocols in place. 

In the results below we can see the results for the waytag name of "name"  These results showed a mixture of street names, and what appeared to be the names of the businesses belonging to that street name. 

```
SELECT tagvalue FROM waytags
WHERE 1=1
AND tagname = 'name'
ORDER BY tagvalue
```

```
108th Avenue Southeast
108th Southeast Avenue
10th Avenue Southwest
11th Avenue South
122nd Avenue Southeast
123rd Place Southeast
127th Avenue Southeast
132nd Avenue Southeast
137th Way Southeast
144th Avenue Southeast
145th Place Southeast
150th Place Southeast
158th Avenue Southeast
15th Avenue South
15th Place South
16th Avenue Southwest
20th Avenue Southwest
27th Avenue South
28th Lane South
30th Avenue Southwest
32nd Place South
33rd Place South
34th Avenue South
40th Way South
47th Avenue Southwest
4th Avenue South
69th Avenue South
6th Place Southwest
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
Bicenntenial Park
BNSF Railway
Boeing 4-51
Boeing Access Road
Bonaci Fine Jewelers
Bryn Mawr Beach Mobile Court
Building K
Claremont
Coopers
Delta Airlines Maintenance Hanga
Des Moines Memorial Drive
Dick's Sporting Goods
DK Market
Dogwood Place
Duvall Place Northeast
Edmonds Avenue Northeast
Elma Court Northeast
Enterprise
Enterprise Plaza
Exofficio
Fairwood Community United Method
Ferry Dock
Fir Drive
Food Mart
Foss Audio
Garden Avenue North
Grace Fellowship of Kent
Helzberg Jewelers
Houser Way South
I-405 / WA 167 HOV Direct Connec
International Boulevard
La Vista
Landmarc IV
Maplewild Avenue Southwest
Maria Paul Courts
Martin Luther King Junior Way So
Massey Creek
McMonigal Veterinary Hospital
Meadow Glen Townhomes
Mighty Mugs Coffee
Miller Creek
Mizuki Buffet
Mr. Hyde
Myers Way South
North 3rd Place
Northeast 14th Place
Northeast 4th Place
Northeast 5th Place
Northeast 6th Court
Oberto
Old Milwaukee Substation
Pacific Highway South
Parkside Elementary School
Petrovitsky Park
Pierce Avenue Southeast
Point Vashon Drive Southwest
Puget Sound Park
Railroad Avenue South
Randy's Restaurant
Restoration Management Company
Safeguard Self Storage - RV and 
Soos Creek Park North Entrance
Soos Creek Trail
South 108th Street
South 126th Street
South 129th Place
South 132nd Street
South 136th Street
South 170th Street
South 170th Street
South 180th Street
South 189th Street
South 192nd Street
South 222nd Street
South 228th Street
South 248th Street
South 254th Place
South 259th Street
South 260th Lane
South 27th Street
South 28th Place
South 96th Street
South Carr Road
South Carr Road
South Prentice Street
South Ryan Street
South Ryan Way
South Victor Street
Southeast 120th Street
Southeast 133rd Court
Southeast 151st Lane
Southeast 171st Place
Southeast 176th Street
Southeast 180th Place
Southeast 183rd Drive
Southeast 222nd Place
Southeast 225th Street
Southeast 228th Street
Southeast 232nd Place
Southeast 232nd Street
Southeast 251st Place
Southeast 251st Street
Southeast 252nd Street
Southeast 259th Place
Southeast 262nd Place
Southeast 7th Court
Southeast Fairwood Boulevard
Southeast Kent Kangley Road
Southwest 102nd Street
Southwest 110th Street
Southwest 119th Street
Southwest 125th Street
Southwest 137th Street
Southwest 156th Street
Southwest 164th Place
Southwest 174th Street
Southwest 183rd Place
Southwest 202nd Street
Southwest 41st Street
Southwest Sunset Boulevard
Springbrook Trout Farm
The Stor House Self Storage
Tukwila International Boulevard
Tukwila International Boulevard
United Rentals
Valley Freeway
Vantage
Vashon Bookshop
Wendy's
West Tower
West Willis Street
```

# Conclusion 
The data for SeaTac, Washington did not seem to have a ton of errors, but there are some areas that needed some further auditing and cleaning, but I believe that for the purposes of the project, it has been well cleaned. It was interesting to further analyze that the waytag values for the waytag name of "name" held both street addresses, and names of places. With a better system in place for accepting user entries, I beleive that the data for both SeaTac, Washington and global would increase in cleanliness!
