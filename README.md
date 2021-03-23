# Zesty.ai engineering test (full-stack)

## Background

Full-stack engineers at Zesty.ai develop our web applications end-to-end, working with modern front-end frameworks, APIs 
(ours and third parties'), and many kinds of data and imagery.

This test is an opportunity for you to demonstrate your comfort with developing both an API and a UI, similar to a 
day-to-day project you might encounter working on our team.

## Assignment

Your goal is to create a full-stack web application that allows developers to search for and retrieve information about 
real estate properties. Using your language(s) and framework(s) of choice, you will need to create a front-end and
back-end for your application and connect to the provided PostgreSQL database.

Your application should include some required features, and may also include optional features. Implementing required 
features is expected to take around 4 hours. If you feel like implementing optional features, you may spend more than 4 
hours. Pick whichever optional features you think you can do best.

Note that some features are more difficult than others, and you will be evaluated on more than just the number of
features completed. Quality is preferred over quantity. Design, organize, and comment your code as you would a typical 
production project. Be prepared to discuss decisions you made.

### Required features

* **List all properties:** Display, in a tabular format, all properties and their geographic location (longitude and 
  latitude).
  
* **Property detail page:** Show detailed information about a given property, including its image, geographic location, 
  and statistics (if applicable).

* **Containerization:** Include Docker image(s) of your application when submitting your final code.

### Optional features

* **Search by coordinates:** Prompt the user for a longitude, latitude, and search radius (default 10000 meters) and 
  display, in a tabular format, the results of the search, including the properties' geographic location (longitude and 
  latitude).

* **Map view:** Using the 
  [Google Maps JavaScript API](https://developers.google.com/maps/documentation/javascript/overview), display a map 
  centered around either the user's current location, or an address they enter. Display a marker on the map for each 
  property. Clicking on a marker should reveal an 
  [Info Window](https://developers.google.com/maps/documentation/javascript/examples/infowindow-simple) with key 
  property information.

* **Save for later:** Allow users to save properties from the List, Search, Detail, and/or Map pages and visit their 
  list of saved properties.

* **Image overlays:** Add polygonal overlays to property images to represent either the parcel, building, or both 
  (`parcel_geo` and `buildings_geo` fields in the database).

* **Statistics:** Calculate geographic data about all properties within a given distance from a reference property. 
  Take *propertyId* and *distance* (in meters) as inputs. The API should return the following:
  * parcel area (meters squared)
  * buildings areas (array, meters squared)
  * buildings distances to center (array, meters squared).  Distance to center is the distance from the building to the 
    `geocode_geo` field in the property table.
  * zone density (percentage).  Create a "zone" geography, which is a buffer of *distance* meters around the 
    `geocode_geo` point. Then, calculate the percentage of that zone geography which has buildings in it.

## Setup
### Development environment requirements

You will need to install [Docker](https://www.docker.com/products/docker-desktop) and 
[`docker-compose`](https://docs.docker.com/compose/install/) to run the example database.

### Database startup
From the repo root folder, run `docker-compose up -d` to start the PostgreSQL database needed for this example. The 
database server will be exposed on port **5555**. If this port is not available on your computer, feel free to change 
the port in the `docker-compose.yml` file.

In the test database, there is a single table called `properties` (with 5 sample rows), where each row represents a 
property or address. There are three geography<sup>*</sup> fields and one field with an image URL pointing to an image on [Google Cloud Storage](https://cloud.google.com/storage/).

<sup>*</sup> *If you are not familiar with [PostgreSQL](https://www.postgresql.org/) or [PostGIS](https://postgis.net/), you may need to read up beforehand.*

### API reference

The API you will be implementing for this project must follow the following API specification.

Note that some endpoints and parameters only apply to optional features.

#### GET /display/:*id*?(overlay=yes(&parcel=:*parcelColor*)(&building=:*buildingColor*))

*Fetches and displays property tile by ID. Optionally overlays parcel and building geometries on tile.*

`example: GET localhost:1235/display/f853874999424ad2a5b6f37af6b56610?overlay=yes&building=green&parcel=orange`

###### Request Parameters

- "id" |
  description: Property ID |
  type: string |
  required: true |
  validation: length greater than 0

- "overlay" |
  description: Overlays parcel and building geometries on tile |
  type: string |
  required: false |
  validation: enum("yes")

- "parcel" |
  description: Indicated building overlay color |
  type: string |
  required: false |
  validation: enum(~<color>~) ex. "red", "green", "orange"

- "building" |
  description: Indicates building overlay color |
  type: string |
  required: false |
  validation: enum(~<color>~) ex. "red", "green", "orange"

###### Response

JPEG image

#### GET /properties

*Lists all properties.*

`example: GET localhost:1235/properties`

###### Response

JSON array of property objects

#### POST /find

*Finds properties within X meters away from provided geojson point.*

`example: POST localhost:1235/find`

###### Request Body

- geojson object with x-distance property

```
''    example:
''
''    {
''     "type": "Feature",
''      "geometry": {
''        "type": "Point",
''        "coordinates": [-80.0782213, 26.8849731]
''      },
''      "x-distance": 1755000
''    }
```

###### Response

JSON array of property IDs

#### GET /statistics/:*id*?distance=:*distance*

*Returns various statistics for parcels and buildings found X meters around the requested property*

`example: GET localhost:1235/statistics/f853874999424ad2a5b6f37af6b56610?distance=1755000`

###### Request Parameters

- "id" |
  description: Property ID |
  type: string |
  required: true |
  validation: length greater than 0

- "distance" |
  description: Buffer distance |
  type: integer |
  required: true |
  validation: greater than 0

###### Response

JSON array including

- "parcel_area_sqm" |
  description: Total area of the property's parcel, in square meters |
  type: float

- "building_area_sqm" |
  description: Total area of buildings inside the property's parcel, in square meters |
  type: float

- "building_distances_m" |
  description: Array of [distance, from the centroid of the property, to the centroid of each building, in meters] |
  type: List[float]

- "zone_density" |
  description: Array of [density of each building's area as a ratio to parcel area, dimensionless] |
  type: List[float]


## Submission instructions

Send us your completed application's code by email, or create and give us access to a new private GitHub repository.

Include instructions on how to run your app, and a list of what features you implemented. Add any comments or things you 
want the reviewer to consider when looking at your submission. You don't need to be too detailed, as there will likely 
be a review done with you where you can explain what you've done.
