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

Our setup has a client and a server, named `mythical-requester` and `mythical-server`. We'll start by looking at the server-side logs in the [Explore window](http://localhost:3000/explore). The Explore window is nice to use to explore data - you can send shared links that contain "everything" (data sources, time ranges, etc), but the visualization features are limited.

* Make sure Loki is selected as a data source in the top-right corner.
* Using the "Builder" feature is usually easier, but try to look at the generated code too.

Here are some steps to get you started exploring the logs:

1. Start by selecting `service_name="mythical-server"` to get all the server-side logs. You should get a list of logs in a key-value format. Add the `logfmt` operation to get all key-value pairs as labels. You can then filter the logs using label filter expression operation. Try filtering by `status`, `level`, and/or `http_method`.

2. In the log details view (click a log line to expand) there's a link to "Tempo" - try clicking it to get a Tracing sidebar. We'll look more at tracing later.

3. Explore is nice, but having it on a dashboard is nicer. Use explore to create a query showing only error logs, then click "+ Add" in the top-right menu to add it to a new dashboard. Give the newly created panel (visualization) a name, and save the dashboard.

Logs by themselves are nice, but showing numbers and stats are usually easier to work with. We'll try to create a dashboard showing panels using the [RED method](https://grafana.com/blog/2018/08/02/the-red-method-how-to-instrument-your-services/): rate, errors, duration.

* `count_over_time` is a good way to turn logs into workable numbers.
* `sum by(http_method) (count_over_time({service_name="mythical-server"} | logfmt [10s]))` shows the number of calls per HTTP method.
* Use it as a starting point to create panels showing the rate of requests over time and the number of errors.
* Try using different visualizations: stat, time chart and gauge are often used.
* Take a look at the pre-made MTL dashboard if you need inspiration (note however, that it doesn't use logs).
* Duration is not available server side, but is available in the client side logs (`service_name="mythical-requester"`).

If you have a large number of logs, or want to do more complex calculations (e.g., 99% percentile), metrics is usually the way to go. Logs are expensive to store, parse and visualize, but are very useful when trying to pinpoint errors.

### Task 3: Metrics with Mimir and PromQL

Metrics are useful for aggregating information about your system(s). Metrics will typically be used to observe the general state of your system, and therefore necessitates a highly scalable and performant, long-term storage service, such as Grafana's Mimir, which is compatible with Prometheus.

#### Explore Mimir
* Navigate to the explore window and select Mimir as data source.
* In the "Builder" window, take a look at the available metrics through the "Metric" dropdown. It might be easier to search for metrics through the "Metrics explorer", which should be the first option in the dropdown list.
* In addition to using the default metrics created by Grafana, it is often relevant to create custom metrics monitoring interesting parts of your system specifically. Try to locate some domain-specific metrics for our mythical beast application.
* A relevant metric to study is the `mythical_request_times_bucket`, which contains the response time for our server-side requests (`mythical-server`). Use this metric to inspect response times for different mythical beast endpoints. Use a relevant range function (e.g. [rate](https://prometheus.io/docs/prometheus/latest/querying/functions/#rate) or [increase](https://prometheus.io/docs/prometheus/latest/querying/functions/#increase)) together with an aggregation operation.
* Explore different labels and aggregations, and apply filters when necessary.
  * What can be said about the performance of the different beast endpoints?
  * What about the performance of different methods (i.e. GET, POST and DELETE)?
  * Some requests take a noticeably shorter amount of time to complete than others. Why is this?


#### Create a RED dashboard using metrics
Metrics are better suited for monitoring the rate, errors and duration of your services over time.
* Create panels for the rate and errors of requests. Add additional groupings as you see fit.
  
  <details>
  <summary>Hint – rate and errors</summary>
  
  To monitor request rate through metrics, the `mythical_request_times_count` is useful, together with `sum` and `rate`.

  For errors, it is also necessary to group the request rate by the status code.
  
  </details>

* Create a panel monitoring the duration of requests. Typically, we allow some anomalies in our data.
  Therefore, try to filter out anomalies, resulting in a panel that only includes a chosen quantile of data as basis for the aggregation.
  
  <details>
  <summary>Hint – duration</summary>
  
  To monitor duration through metrics, the `mythical_request_times_bucket` is relevant, as seen in an earlier task.
  
  To filter out anomalies, a histogram quantile function can be used, where a quantile for example can be set to the 95th percentile.
  For the histogram to work, it is also necessary to add the label `le`, 
  which indicates that the result is _less than or equal to_ the desired upper quantile limit.
  
  </details>

#### Metrics with exemplars
Although metrics are useful for observing your system at a glance, it can be difficult to pinpoint which factors are contributing to e.g. latency or unexpected traffic between parts of your system.
As you may have observed in your own or the pre-made dashboard panels, Grafana can display so-called "exemplars" – indicated by a highlighted star – alongside metrics.

These exemplars link to a "trace" instance, which we will explore further in the following task, and can be used to bridge metrics and traces.
Exemplars can be useful for identifying factors contributing to latency in a production-environment with vast amounts of data.
If you can't see any exemplars in your own dashboards, you can explore [this example](http://localhost:3000/explore?left=%7B%22datasource%22:%22mimir%22,%22queries%22:%5B%7B%22datasource%22:%7B%22type%22:%22prometheus%22,%22uid%22:%22mimir%22%7D,%22exemplar%22:true,%22expr%22:%22histogram_quantile%280.95,%20sum%28rate%28mythical_request_times_bucket%5B15s%5D%29%29%20by%20%28le,%20beast%29%29%22,%22interval%22:%22%22,%22refId%22:%22A%22%7D%5D,%22range%22:%7B%22from%22:%22now-5m%22,%22to%22:%22now%22%7D%7D&orgId=1). 

### Task 4: Tracing with Tempo and TraceQL

TODO: Metrics to tracing with exemplars
TODO: Logs to metrics with span/trace ids
TODO: TraceQL for long duration calls, errors and deep chains

### Extras

#### Dashboard variables
#### Alerts
####

## Extras



