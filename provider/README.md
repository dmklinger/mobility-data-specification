# Mobility Data Specification: **Provider**

This specification contains a data standard for *mobility as a service* providers to define a RESTful API that municipalities can access during the pilot phases of a program.

* Authors: LADOT
* Date: 23 May 2018
* Version: ALPHA

## Notes

* All response fields should use `lower_case_with_underscores` to implement the API
* Responses may be paginated

## Table of Contents

* [Trips](#trips)
* [Availability](#availability)
* [Service Areas](#service-areas)
* [Examples](#examples)

## Trips

A trip represents a journey taken by a *mobility as a service* customer with a geo-tagged start and stop point.

The trips API allows a user to query historical trip data. The API should allow querying trips at least by ID, geofence for start or end, and time.

Endpoint: `/trips`  
Method: `GET`  
Response:

| Field | Type     | Required/Optional | Other |
| ----- | -------- | ----------------- | ----- |
| `company_name` | String | Required | |
| `device_type` | String | Required | | 
| `trip_id` | UUID | Required | a unique ID for each trip | 
| `trip_duration` | Integer | Required | Time, in Seconds | 
| `trip_distance` | Integer | Required | Trip Distance, in Meters | 
| `start_point` | Point | Required | Must be in WGS 84 (EPSG:4326) standard GPS projection | 
| `end_point` | Point | Required | Must be in WGS 84 (EPSG:4326) standard GPS projection | 
| `accuracy` | Integer | Required | The approximate level of accuracy, in meters, represented by start_point and end_point. |
| `route` | Array of Points with UNIX Timestamps | Optional | | 
| `sample_rate` | Integer | Optional | The frequency, in seconds, in which the route is sampled | 
| `device_id` | UUID | Required | | 
| `start_time` | Unix Timestamp | Required | | 
| `end_time` | Unix Timestamp | Required | |
| `parking_verification` | String | Optional | A URL to a photo (or other evidence) of proper vehicle parking | 
| `standard_cost` | Integer | Optional | The cost, in cents that it would cost to perform that trip in the standard operation of the System. | 
| `actual_cost` | Integer | Optional | The actual cost paid by the user of the Mobility as a service provider |

## Availability

Availability is the inventory of vehicles available for customer use.

The availability API allows a user to query the historical availability for a system within a time range. The API should allow queries at least by time period and geographical area.

Endpoint: `/availability`  
Method: `GET`  
Response:

| Field | Type | Required/Optional | Other | 
| ----- | ---- | ----------------- | ----- | 
| `company_name` | String | Required | |
| `device_type` | String | Required | | 
| `device_id` | UUID | Required | |
| `availability_start_time` | Unix Timestamp | Required | |
| `availability_end_time` | Unix Timestamp | Required | If a device is still available, use NaN  |
| `placement_reason` | Enum | Required | Reason for placement (`user_drop_off`, `rebalancing_drop_off`) | 
| `allowed_placement` | Bool | Required | Indicates whether provider believes placement was allowable under service area rules. | 
| `pickup_reason` | Enum | Required | Reason for removal (`user_pick_up`, `rebalacing_pick_up`, `out_of_service_area_pick_up`, `maintenance_pick_up`) | 
| `associated_trips` | [UUID] | Optional | list of associated maintenance | 

### Realtime Data

All MDS compatible APIs should expose a GBFS feed as well. For historical data, a `time` parameter should be provided to access what the GBFS feed showed at a given time.

## Service Areas 

Service areas are the geographic regions within which a *mobility as a service* provider is permitted to operate. 

Endpoint: `/service_areas`  
Method: `GET`  
Response:

| Field | Type | Required/Optional | Other | 
| ----- | ---- | ----------------- | ----- | 
| `operator_name` | String | Required |  |
| `service_area_id` | UUID | Required |  | 
| `service_start_date` | Unix Timestamp | Required | Date at which this service area became effective | 
| `service_end_date` | Unix Timestamp | Required | Date at which this service area was replaced. If current effictive, place NaN | 
| `service_area` | MultiPolygon | Required | Must be in WGS 84 (EPSG:4326) standard GPS projection | 
| `prior_service_area` | UUID | Optional | If exists, the UUID of the prior service area. | 
| `replacement_service_area` | UUID | Optional | If exists, the UUID of the service area that replaced this one | 

## Examples

Trip endpoint:

```
{
    "company_name": "ScooterCompany",
    "device_type": "scooter",
    "trip_id": "7ee01a8a-2dc9-47aa-8ad6-edf854fd8388",
    "trip_duration": 124.86813232663945,
    "trip_distance": 321.4725293065578,
    "start_point": {
        "type": "Point",
        "coordinates": [
            -118.46416532993315,
            33.99017277784858
        ] 
    },
    "end_point": {
        "type": "Point",
        "coordinates": [
            -118.46710503101347,
            33.9909333514159
        ]
    },
    "route": {
        "type": "LineString",
        "coordinates" : [
            [
                -118.46416264772414,
                33.99018612130317
            ],
            [
                -118.46598118543626,
                33.99065981258349
            ],
            [
                -118.46710503101347,
                33.99093557530522
            ]
        ]
    },
    "sample_rate": 60,
    "accuracy": 5,
    "device_id": "184ba280-4c6f-41a9-853f-8c2c997b357f",
    "start_time": 1529968782.421409,
    "end_time": 1529968907.289541,
    "parking_verification": "urlhere",
    "standard_cost": 0.01,
    "actual_cost": 1.15
}
```

Availability endpoint:

```
{
    "company_name": "ScooterCompany",
    "device_type": "scooter",
    "device_id": "184ba280-4c6f-41a9-853f-8c2c997b357f",
    "availability_start_time": 1529968907.289541,
    "availability_end_time": 1529968345.345325,
    "placement_reason": "user_drop_off"
    "allowed_placement": true
    "pickup_reason": "user_pick_up"
}
```

Service Area endpoint:

```
{
    "operator_name": "ScooterCompany",
    "service_start_date": 1533128400.0,
    "service_end_date": 1535796000.0,
    "service_area": {
        "type": "MultiPolygon",
        "coordinates": [
            [
                [
                    -118.47793579101561,
                    33.99176508196857
                ],
                [
                    -118.47081184387206,
                    33.98415021813989
                ],
                [
                    -118.45767974853514,
                    33.99162275432302
                ],
                [
                    -118.28390999909007,
                    34.063582999985535
                ],
                [
                    -118.2830799995123,
                    34.063214999434024
                ]
	    ]
        ]
    },
    "service_area_id": "1dc76b0c-fe95-4b10-9f65-623646b08508"
}
```
   
