Since I haven't managed to figure out how to add documentation for custom filters to auto generated docs, I add the additional description here. This document should be removed as soon as auto generated docs can support custom filters.

| field name   | description                              |
| ------------ | ---------------------------------------- |
| start_time   | DateTime to specify the start time of the time range where the query should be applied. DateTime should be supplied in a ISO format: YYYY-MM-DDThh:mm[:ss[.uuuuuu]][+HH:MM|-HH:MM|Z] |
| end_time     | DateTime value to specify the end time of the time range where the query should be applied.DateTime should be supplied in a ISO format: YYYY-MM-DDThh:mm[:ss[.uuuuuu]][+HH:MM|-HH:MM|Z] |
| start_strict | boolean to specify whether query should be executed in with strict mode for start_time.<br />- Non strict start time request means that sensor data samples time period could intersect with start time request. When it does, sensor data samples will be included in the request result.<br />- Strict start time request means that start time of sensor data sample must be equal to  or later than the request start time. |
| end_strict   | boolean to specify whether query should be executed in with strict mode for end_time.<br />- Non strict end time request means that sensor data samples time period could intersect with end time request. When it does, sensor data samples will be included in the request result.<br />- Strict end time request means that end time of sensor data sample must be less than the request end time |
| tags         | URI encoded string. You can encode the string by `encodeURIComponent('<tag query string>')` in JS. For the examples of notation for tag filter, see the section below.

## Example notations for `tags` field

### 1. a & b & !c & !d 

```json
{
  "$and": [
    "a", 
    "b", 
    {"$not": "c"},
    {"$not": "d"}
  ]
}
```


### 2. a & (b | c)

```json
{
  "$and": 
      [ "a", 
       { "$or": ["b", "c"] }
      ]
}
```


### 3. a & !(b | c)

```json
{
  "$and": [
      "a",
      {"$not": {"$or": ["b", "c"] }}
  ]
}
```
### 4. (a & b) | (c & d)

```json
{
    "$or":[
      {"$and":["a","b"]},
      {"$and":["c","d"]}
    ]
}
```


### 5 a & !(b & (c | d))

```json
{
   "$and": [
       "a",
       { 
          "$not": {
              "$and": ["b", { "$or": [ "c", "d"]} ]
          }
       }
    ]
}
```
