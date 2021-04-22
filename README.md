# influx-db-grafana
Hosting Grafana with Influx DB backing in Docker Containers. Just messing around.

The goal here is to be able to have a light weight data gathering end point that is flexible to begin collecting custom data and visualizing it.

## Goals
* Have an easy setup that is useable for any system in the network to begin collecting data.
* Have visualizations to be able to prove ideas instead of guessing.
* Be reliable in a development environment.

## Non-Goals
* Scale. This is meant to be scrappy until a team requires an expert in data engineering.
* Be reliable in a production environment. This might come given the technologies, but this is not testing that.

# Manual Steps
Following instructions from: https://towardsdatascience.com/get-system-metrics-for-5-min-with-docker-telegraf-influxdb-and-grafana-97cfd957f0ac
1. Create network and volumes:
    1. `docker network create monitoring`
    1. `docker volume create grafana-volume`
    1. `docker volume create influxdb-volume`
        1. Check with:
            1. `docker network ls`
            1. `docker volume ls`
1. Compose Docker.yaml. This creates a new image for the custom environment defined in the `.yml`:
    1. `cd opt/monitoring`
    1. `docker-compose up -d`
        1. This will only look recursively upwards. That's why `docker-compose.yml` is in the root.

At this point, you are running InfluxDB and Grafana. The Grafana web UI can be located at [localhost:3000](localhost:3000)

1. When adding the data source, the initialized DB is setup by running the `influxDB` container image. This automatically creates a DB named `telegraf` with credentials for 
user: `telegraf` 
password: `password`

# Influx DB Data
Influx DB categorizes different data into __measurements__ based on the name of the data. 
A __tag__ (categorization) can be used to organize data points. Think of this as metadata about the data point. These are typically used to aggregate data, but not explicitly shown in visualizations.
A __field__ (data values) are the pieces of data that will be shown in visualizations in Grafana.

## Write via POST API
* Example: `curl -i -XPOST -u <username>:<password> "http://influxDBIP.com:<port_default=8086>/write?db=<name_of_db>" --data-binary 'weather,location=us-midwest temperature=82,longitude=123,latitude=-4'`
* Working Example with this setup: `curl -i -XPOST -u admin:password "http://localhost:8086/write?db=telegraf" --data-binary 'weather,location=us-midwest temperature=82,longitude=123,latitude=-4'`
    * In the example above, here is how the data is stored:
        * __Measurement__: weather
        * __Tag__ (categorization) 
            * location:us-midwest
        * __Field__ (data values)
            * temperature: 82
            * longitude: 123
            * latitude: -4

## Read via API
Example: `curl -G -u <username>:<password> "http://influxDBIP.com:<port_default=8086>/query?db=<name_of_db>" --data-urlencode 'q=SELECT * FROM "weather"'`

Working example with this setup: `curl -G -u admin:password "http://localhost:8086/query?db=telegraf" --data-urlencode 'q=SELECT * FROM "weather"'`
