# Observability workshop with the LGTM-stack

Introduction to using Grafana Labs' LGTM-stack for observability. The LGTM-stack contains the following tools:

* Loki for log aggregation
* Grafana for data visualization
* Tempo as a tracing backend
* Mimir as a metrics backend

Most tasks in this workshop can be done in your own environment, if you have one, but we recommend and will refer to Grafana Labs' [Intro to MLTP](https://github.com/grafana/intro-to-mltp/) setup.

## Installation instructions

If you're running the Intro to MTLP setup locally, you'll need to:

1. Tools compatible with Docker and Docker compose files (Docker Desktop, Colima, Podman or other).
    Note: You will likely have to adjust the CPU and memory settings. E.g., for 4 CPU cores and 12 GB of memory with Colima: `colima start --cpu 4 --memory 12`

2. Clone the repository: `git clone https://github.com/grafana/intro-to-mltp` (or use a GUI of your choice)

3. Start the containers. From the repository root folder: `docker-compose up`

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

### Task X: Dashboard variables

## Extras



