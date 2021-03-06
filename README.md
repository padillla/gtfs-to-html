# GTFS to HTML

GTFS-to-HTML converts transit data in [GTFS format](https://developers.google.com/transit/gtfs/) into user-friendly HTML schedules. Many transit agencies have schedule data in GTFS format but need to show each route's schedule to users on a website. This project aims to automate the process of creating these schedules. Automating HTML schedule generation makes it easy to keep schedules up to date when data changes and reduces the likelihood of errors.

GTFS-to-HTML is currently used by [Sonoma Country Transit](http://sctransit.com/) to generate schedule pages for each route.

<img width="1265" src="https://cloud.githubusercontent.com/assets/96217/10262598/87674f70-6983-11e5-8150-15b6372c989c.png">

## Setup

### Install dependencies

    npm install

## Configure

Copy `config-sample.json` to `config.json`.

    cp config-sample.json config.json

### Agencies

Before you can use gtfs-to-html you must specify the transit agencies you'd
like to use.

You can specify agencies using a `url` to the GTFS file or a local `path`.

To find an agency's GTFS URL, visit transitfeeds.com. You can use the direct
URL from the agency or you can use a URL generated from the transitfeeds.com
API along with your API token.

* Specify a download URL:
```
{
  "agencies": [
    {
      "agency_key": "county-connection",
      "url": "http://cccta.org/GTFS/google_transit.zip"
    }
  ]
}
```

* Specify a path to a zipped GTFS file:
```
{
  "agencies": [
    {
      "agency_key": "myAgency",
      "path": "/path/to/the/gtfs.zip"
    }
  ]
}
```
* Specify a path to an unzipped GTFS file:
```
{
  "agencies": [
    {
      "agency_key": "myAgency",
      path: "/path/to/the/unzipped/gtfs/"
    }
  ]
}
```

### MongoDB URI

Add the MongoDB URI to `config.json` with the key `mongo_url`. Running locally, you may want to use `mongodb://localhost:27017/gtfs`.
```
{
  "mongo_url": "mongodb://localhost:27017/gtfs",
  "agencies": [
    {
      "agency_key": "myAgency",
      path: "/path/to/the/unzipped/gtfs/"
    }
  ]
}
```

### `effectiveDate`

{String} This is printed at the top of the timetable.

```
    effectiveDate: 'July 8, 2015'
```

### `noServiceSymbol`

{String} The symbol to be used when a specific trip does not serve a specified stop.

```
    noServiceSymbol: '—'
```

### `requestStopSymbol`

{String} The symbol to be used to indicate that riders must request a stop.

```
    requestStopSymbol: '***'
```

### `showMap`

{Boolean} Whether or not to show a map of the route on the timetable.

```
    showMap: false
```

### `showOnlyTimepoint`

{Boolean} Whether or not all stops should be shown, or only stops with a `timepoint` value in [stop_times.txt](https://developers.google.com/transit/gtfs/reference?hl=en#stop_times_fields) that is considered exact (i.e. empty or `1`). Default is `false`, all stops shown.

```
    showOnlyTimepoint: false
```

### `zipOutput`

{Boolean} Whether or not to zip the output into one zip file

```
    zipOutput: false
```

## Build `timetables.txt`

This project requires that an additional file `timetables.txt` be added to an agencies GTFS. This file specifies which HTML timetables should be built.

An example of this file is located in [examples/timetables.txt](examples/timetables.txt). The format of this file is:

| column name | description |
| ----------- | ----------- |
| `timetable_id` | A unique ID for the timetable |
| `route_id` | The ID of the route the timetable is for from `routes.txt`. |
| `direction_id` | The `direction_id` from `trips.txt` for the timetable. This can be blank. |
| `start_date` | The start date for this timetable in `YYYY-MM-DD` format. |
| `end_date` | The end date for this timetable in `YYYY-MM-DD` format. |
| `monday` | A binary value that indicates whether this timetable should include service on Mondays.  Valid options are `0` and `1`. |
| `tuesday` | A binary value that indicates whether this timetable should include service on Tuesdays.  Valid options are `0` and `1`. |
| `wednesday` | A binary value that indicates whether this timetable should include service on Wednesdays.  Valid options are `0` and `1`. |
| `thursday` | A binary value that indicates whether this timetable should include service on Thursdays.  Valid options are `0` and `1`. |
| `friday` | A binary value that indicates whether this timetable should include service on Fridays.  Valid options are `0` and `1`. |
| `saturday` | A binary value that indicates whether this timetable should include service on Saturdays.  Valid options are `0` and `1`. |
| `sunday` | A binary value that indicates whether this timetable should include service on Sundays.  Valid options are `0` and `1`. |
| `route_label` | A short text label describing the route, for instance "4". |
| `service_notes` | Text shown on the timetable about the service represented, for instance "Mon-Fri". |
| `orientation` | Determines if the top row should be a list of trips or stops. Valid options are `vertical` and `horizontal`. `vertical` shows stops across the top row with each row being a list of stop times for each trip. `horizontal` shows trips across the top row with each row being stop times for a specific stop.  `horizontal` orientation is best for routes with lots of stops and fewer trips while `vertical` orientation is best for routes with lots of trips and a smaller number of stops. |

### Multi-route Timetables

To allow creating a single timetable for multiple routes that overlap, you can have multiple entries in `timetables.txt` for the same `timetable_id`. These multi-route entries should have the same values `timetable_id`, `start_date`, `end_date`, calendar date, `service_notes` and `orientation` fields and should have different values for the `route_id` and `route_label` fields.

### Build `timetable_stop_order.txt`

This is an optional file that can specify stop order for a particular timetable. It is useful when generating combined timetables for multiple overlapping routes, or exerting fine-grained control on stop order.

An example of this file is located in [examples/timetable_stop_order.txt](examples/timetable_stop_order.txt). The format of this file is:

| column name | description |
| ----------- | ----------- |
| `timetable_id` | The ID of the timetable from `timetables.txt` |
| `stop_id` | The ID of the stop from `stops.txt`. |
| `stop_sequence` | An assigned integer identifying the order of stops to be presented in the timetable. The values for `stop_sequence` must be non-negative integers, and they must increase along the trip. This value does not need to match the `stop_sequence` found in `stop_times.txt`. |

## Running

Ensure than mongodb is running locally.

    mongod

To generate HTML timetables, run the 'gtfs-to-html' script.

    npm run gtfs-to-html

This will download the GTFS file specified in `config.js` .  Then, it will build the HTML timetables and save them in `html/:agency_key`.

### Options

Note the use of two sets of `--` when running commands with arguments from npm.

`nohead`

    npm run gtfs-to-html -- --nohead

This will generate embeddable HTML without an `<html>`, `<head>` or `<body>` tag.


## Testing

After an initial run of the `gtfs-to-html` script, the GTFS data will be downloaded and loaded into mongo.

You can view an individual route HTML on demand by running the included express app:

    DEBUG=gtfs-to-html npm start

With this running, you can open [http://localhost:3000](http://localhost:3000) in your browser and view all timetables.  Note that this only works after GTFS has been imported to mongodb and mongodb is running locally.

### Linting

    npm run lint
