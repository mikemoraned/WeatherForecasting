# Install DB

```shell
docker run -it --rm --name influxdb3core \
  -v `pwd`/data:/home/influxdb3/data \
  -v `pwd`/influxdb3/plugins:/var/lib/influxdb3_plugins \
  -v `pwd`/influxdb3/data:/var/lib/influxdb3_data \
  -p 8181:8181 \
  quay.io/influxdb/influxdb3-core:latest serve \
  --node-id my_host \
  --object-store file \
  --data-dir /var/lib/influxdb3_data
  --plugin-dir /var/lib/influxdb3_plugins
```

# Get Auth

open orb shell (uses https://orbstack.dev/)

get token
```shell
influxdb3 create token --admin
export TOKEN=...
```

# Create table

```shell
influxdb3 create database my_awesome_db --token $TOKEN
influxdb3 show databases --token $TOKEN
cd /home/influxdb3
influxdb3 write --database my_awesome_db --file ./data/london_weather_ns.lp --token $TOKEN
```

```shell
influxdb3 query \
  --database my_awesome_db \
  --token $TOKEN \
  "SELECT * FROM london_weather"
```

# Do predictions

```shell
influxdb3 install package pandas neuralprophet plotly matplotlib --package-manager uv --token $TOKEN
```

```shell
influxdb3 create trigger \
  --trigger-spec "every:1m" \
  --plugin-filename /var/lib/influxdb3_plugins/forecast_london_weather.py \
  --token $TOKEN \
  --database my_awesome_db \
  london_weather_forecast
```

```shell
influxdb3 query \
  --database my_awesome_db \
  --token $TOKEN \
  "SELECT * FROM forecast_weather"
```

... I got a 500 at this point and logs show:
```
2025-06-06T11:24:00.059334Z ERROR influxdb3_processing_engine::plugins::python_plugin: error running scheduled plugin: ModuleNotFoundError: No module named 'pandas' self.trigger_definition=TriggerDefinition { trigger_id: TriggerId(1), trigger_name: "london_weather_forecast", plugin_filename: "/var/lib/influxdb3_plugins/forecast_london_weather.py", database_name: "my_awesome_db", node_id: "my_host", trigger: Every { duration: 60s }, trigger_settings: TriggerSettings { run_async: false, error_behavior: Log }, trigger_arguments: None, disabled: false }
```

```shell
influxdb3 disable trigger --database my_awesome_db london_weather_forecast --token $TOKEN 
influxdb3 delete trigger --database my_awesome_db london_weather_forecast --token $TOKEN 
```