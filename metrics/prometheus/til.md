# Prometheus

TIL that the timestamp of a datapoint in the Prometheus tsdb is the point in time when the scraping of the `metrics/` endpoint occured.

Below is the raw data (`{value} @{timestamp}`) of a gauge that I changed the value of via a GET request while the service was running. Notice that it's 15 seconds between each timestamp which is the interval that the scrape job was configured to run at.

```plain
160 @1612358915.91
160 @1612358930.91
170 @1612358945.91
180 @1612358960.91
140 @1612358975.91
```

Another TIL that I confirmed was that changes in the gauge that occured between scrapes were never tracked (the gauge was set to `190` between `180` and `140`). Not surprising but nice to actually observe.

Got the raw data by going to `http://localhost:9090/graph` and then executing this query `trial_gauge[5m]` and viewing it in the tables view
