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
