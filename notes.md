```sql
select *
from (
  select stations.id as station_id, date(start.start_date) as day, count(start.rental_id) as rides_from_station
  from `mailchimp-350820.london_bicycle.cycle_stations_merged` stations
  join `mailchimp-350820.london_bicycle.cycle_hire_merged` start on start.start_station_id = stations.id
  where start.start_station_id = 395
  group by station_id, day
) start
join (
  select stations.id as station_id, date(endd.start_date) as day, count(endd.rental_id) as rides_to_station
  from `mailchimp-350820.london_bicycle.cycle_stations_merged` stations
  join `mailchimp-350820.london_bicycle.cycle_hire_merged` endd on endd.end_station_id = stations.id
  where endd.end_station_id = 395
  group by station_id, day
) endd on start.day = endd.day
order by start.day
```


cycle_stations has 791 rows
cycle_stations_update has 791 rows = 100% of rows

cycle_hire has 24,369,201 rows
cycle_hire_update has 1,217,716 rows = 5% of rows



ORIGINAL
{"id":"260","installed":"true","latitude":"51.5136846","locked":"false","longitude":"-0.135580879","name":"Broadwick Street, Soho","bikes_count":"18","docks_count":"18","nbEmptyDocks":"0","temporary":"false","terminal_name":"3489","install_date":"2010-07-21","removal_date": null}

UPDATE
{"id":"260","installed":false,"latitude":51.5136846,"locked":true,"longitude":-0.135580879,"name":"Broadwick Street, Soho","bikes_count":"180","docks_count":"18","nbEmptyDocks":"0","temporary":false,"terminal_name":"3489","install_date":"2010-07-21"}



in english:
I want to create a table from two existing tables, original and update.
I want to choose all values from update, unless it doesn't exist, and then I'll keep original.
There are 3 new rows in update, so I want to insert those. Total rows should be 791 + 3 = 794



ORIGINAL
{"rental_id":"62592875","duration":"2280","bike_id":"12872","end_date":"2017-02-22T23:18:00Z","end_station_id":"12","end_station_name":"Malet Street, Bloomsbury","start_date":"2017-02-22T22:40:00Z","start_station_id":"14","start_station_name":"Belgrove Street , King\u0027s Cross","end_station_logical_terminal": null,"start_station_logical_terminal": null,"end_station_priority_id": null}

UPDATE
{"rental_id":"62592875","duration":"2280","bike_id":"12872","end_date":"2017-02-22 23:22:00 UTC","end_station_id":"12","end_station_name":"Malet Street, Bloomsbury","start_date":"2017-02-22 22:40:00 UTC","start_station_id":"14","start_station_name":"Belgrove Street , King's Cross"}

--------------

station_stats

ONE ROW FOR EACH DAY AND EACH STATION

# of rides (and bikes) that started at the station
# of rides (and bikes) that ended at the station
The total amount of time spent on rides
The avg amount of time spent on rides
The median amount of time spent on a ride
The most common station that people rode to from that station
The most common station that people rode from to that station
Any other useful stats you can think of (including from other datasets if you’re feeling adventurous) 


station_id
day
rides_from_station = int
rides_to_station = int
total_ride_time = int
average_ride_time = float64
median_ride_time = float64
most_popular_destination = int
most_common_origin = int





WIP
select start.station_id, start.day, start.rides_from_station, endd.rides_to_station, start.total_ride_time, average_ride_time
from (
  select stations.id as station_id, date(start.start_date) as day, count(start.rental_id) as rides_from_station, sum(start.duration) as total_ride_time, avg(start.duration) as average_ride_time
  from `mailchimp-350820.london_bicycle.cycle_stations_merged` stations
  join `mailchimp-350820.london_bicycle.cycle_hire_merged` start on start.start_station_id = stations.id
  where start.start_station_id = 395
  group by station_id, day
) start
join (
  select stations.id as station_id, date(endd.start_date) as day, count(endd.rental_id) as rides_to_station
  from `mailchimp-350820.london_bicycle.cycle_stations_merged` stations
  join `mailchimp-350820.london_bicycle.cycle_hire_merged` endd on endd.end_station_id = stations.id
  where endd.end_station_id = 395
  group by station_id, day
) endd on start.day = endd.day
join (
  select stations.id as station_id, date(hires.start_date) as day, percentile_cont(hires.duration, 0.5) over (partition by hires.rental_id) as median_ride_time
  from `mailchimp-350820.london_bicycle.cycle_stations_merged` stations
  join `mailchimp-350820.london_bicycle.cycle_hire_merged` hires on hires.start_station_id = stations.id
  where hires.start_station_id = 395
  group by station_id, day, hires.duration, hires.rental_id
)
order by start.day


[
  {
    "description": "",
    "mode": "REQUIRED",
    "name": "station_id",
    "type": "INTEGER"
  },
  {
    "description": "",
    "mode": "REQUIRED",
    "name": "day",
    "type": "TIMESTAMP"
  },
    {
    "description": "",
    "mode": "NULLABLE",
    "name": "rides_from_station",
    "type": "INTEGER"
  },
    {
    "description": "",
    "mode": "NULLABLE",
    "name": "rides_to_station",
    "type": "INTEGER"
  },
    {
    "description": "",
    "mode": "NULLABLE",
    "name": "total_ride_time",
    "type": "INTEGER"
  },
    {
    "description": "",
    "mode": "NULLABLE",
    "name": "average_ride_time",
    "type": "FLOAT"
  },
    {
    "description": "",
    "mode": "NULLABLE",
    "name": "median_ride_time",
    "type": "FLOAT"
  },
    {
    "description": "",
    "mode": "NULLABLE",
    "name": "most_popular_destination",
    "type": "INTEGER"
  },
    {
    "description": "",
    "mode": "NULLABLE",
    "name": "most_common_origin",
    "type": "INTEGER"
  }
]