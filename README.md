# Observability workshop with the LGTM-stack

Introduction to using Grafana Labs' LGTM-stack for observability. The LGTM-stack contains the following tools:

* Loki for log aggregation
* Grafana for data visualization
* Tempo as a tracing backend
* Mimir as a metrics backend

Most tasks in this workshop can be done in your own environment, if you have one, but we recommend and will refer to Grafana Labs' [Intro to MLTP](https://github.com/grafana/intro-to-mltp/) setup. This setup provides a "mythical beasts" API, and utilizes Grafana k6 to generate traffic.

## Installation instructions

If you're running the Intro to MTLP setup locally, you'll need to:

1. Tools compatible with Docker and Docker compose files (Docker Desktop, Colima, Podman or other).
    Note: You will likely have to adjust the CPU and memory settings. E.g., for 4 CPU cores and 12 GB of memory with Colima: `colima start --cpu 4 --memory 12`

2. Clone the repository: `git clone https://github.com/grafana/intro-to-mltp` (or use a GUI of your choice)

3. Start the containers. From the repository root folder: `docker-compose up`. Note: If you have other applications or containers running you might have to stop them to free up the correct ports.

4. After the containers started successfully, navigate to [localhost:3000](http://localhost:3000/) to get started.

## Tasks

### Task 1: Explore Grafana

Navigate to [Dashboards](http://localhost:3000/dashboards). There are 3 different pre-made dashboards. Look at and explore the dashboards. Here are some things you should try out:

* Change the time range and refresh rate in the top right menu.
* Try changing the variables of the board in the top right menu (we'll look at how to create them in a later task).
* Each panel (visualization) has a triple dot menu at the top right, try going into the "Edit" view to look at how the panel was made.
    * At the bottom of the screen, you can choose "Builder" to view a simplified view of the query, you can also try using the "Explain" toggle to get a description of the query.
    * At the right-hand side, you can configure visualization settings.
    * Test out changing the queries and settings, use the discard button (top right) when you're done.

The different dashboards have different visualizations, different variables and uses different kinds of data. Make sure to take a look at all three dashboards before you move on.

### Task 2: Logs with Loki and LogQL

#### Listing logs

Our setup has a client and a server, named `mythical-requester` and `mythical-server`. We'll start by looking at the server-side logs in the [Explore window](http://localhost:3000/explore). The Explore window is nice to use to explore data - you can send share links that contain "everything" (data sources, time ranges, etc), but the visualization features are limited.

* Make sure Loki is selected as a data source in the top-right corner.
* Using the "Builder" feature is usually easier, but try to look at the generated code too.

Here are some steps to get you started exploring the logs:

1. Start by selecting `service_name="mythical-server"` to get all the server-side logs. You should get a list of logs in a key-value format. Add the `logfmt` operation to get all key-value pairs as labels. You can then filter the logs using label filter expression operation. Try filtering by `status`, `level`, and/or `http_method`.

2. In the log details view (click a logline to expand) there's a link to "Tempo" - try clicking it to get a Tracing sidebar. We'll look more at tracing later.

3. Explore is nice, but having it on a dashboard is nicer. Use explore to create a query showing only error logs, then click "+ Add" in the top-right menu to add it to a new dashboard. Give the newly created panel (visualization) a name, and save the dashboard.

Logs by themselves are nice, but showing numbers and stats are usually easier to work with. We'll try to create panel showing panels using the [RED method](https://grafana.com/blog/2018/08/02/the-red-method-how-to-instrument-your-services/): rate, errors, duration.

* `count_over_time` is a good way to turn logs into workable numbers.
* `sum by(http_method) (count_over_time({service_name="mythical-server"} | logfmt [10s]))` shows the number of calls per HTTP method.
* Use it as a starting point to create panels showing the rate of requests over time and the number of errors.
* Try using different visualizations: stat, time chart and gauge are often used.
* Take a look at the pre-made MTL dashboard if you need inspiration (note however, that it doesn't use logs).
* Duration is not available server side, but you is available in the client side logs (`service_name="mythical-requester"`).

If you have a large number of logs, or want to do more complex calculations (e.g., 99% percentile), metrics is usually the way to go. Logs are expensive to store, parse and visualize, but are very useful when trying to pin-point errors.

### Task 3: Metrics with Mimir and PromQL

TODO: Which metrics, labels are a good starting point
TODO: How to do RED visualizations

### Task 4: Tracing with Tempo and TraceQL

TODO: Metrics to tracing with exemplars
TODO: Logs to metrics with span/trace ids
TODO: TraceQL for long duration calls, errors and deep chains

### Extras

#### Dashboard variables
#### Alerts
####

## Extras



