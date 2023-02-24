# MovR

This repo contains the source code for an example implementation of a single-region application for the fictional vehicle-sharing company MovR, as used by the CockroachDB for Python Developers course.

- [Overview](#overview) gives a high-level overview the MovR application stack. 

## Overview

The application stack consists of the following components:

- A CockroachCloud instance
- Python class definitions that map to the table(s) in the database.
- A backend API that defines the application's connection to the database and the database transactions.
- A Flask server that handles requests from client web browsers.
- HTML templates that define web pages that the Flask server renders.
- Flask Open Telemetry tracing to Jaeger server

## Setup

### Start Jaeger Server for Trace Collection

1. Start the all-in-one Jaeger server locally or remotely with `docker compose up -d`. The trace collection occurs in the OTLP `4317` port.

    ~~~ yaml
    version: "3.9"
    services:
    jaeger:
        container_name: jaeger-container
        image: jaegertracing/all-in-one:latest
        environment:
        - COLLECTOR_OTLP_ENABLED=true
        ports:
        - 6831:6831/udp # UDP protocol: accept jaeger.thrift in compact Thrift protocol used by most current Jaeger clients
        - 6832:6832/udp # UDP protocol: accept jaeger.thrift in binary Thrift protocol used by Node.js Jaeger client (because thriftrw npm package does not support compact protocol)
        - 16686:16686 # HTTP protocol: /api/* endpoints and Jaeger UI at /
        - 14268:14268 # HTTP protocol: can accept spans directly from clients in jaeger.thrift format over binary thrift protocol
        - 4318:4318 # HTTP protocol: accepts traces in OpenTelemetry OTLP format if --collector.otlp.enabled=true
        - 4317:4317 # OTel port
        restart: unless-stopped
    ~~~


### Database and Environment Setup

You'll need to connect to a [CockroachCloud](https://cockroachlabs.cloud/) cluster.

1. Configure environment variables
     
    When running your server, it's easiest if your CockroachCloud connection
    string is assigned to the DB_URI variable in the `.env` file. You can paste
    it in with the editor of your choice.

2. Initialize the database

    You can do this with

    ~~~ shell
    export crdb_url="postgresql://<db-user>:<db-pass>@<db-url>:26257/movr?sslmode=verify-full"
    cat dbinit.sql | cockroach sql --url ${crdb_url}
    ~~~
    
### Application setup

1. To run the application and tracing with OpenTelemetry, use the following commands in the shell to set the environment:

    ~~~ shell
    export OTEL_INSTRUMENTATION_HTTP_CAPTURE_HEADERS_SERVER_REQUEST="Accept-Encoding,User-Agent,Referer"
    export OTEL_INSTRUMENTATION_HTTP_CAPTURE_HEADERS_SERVER_RESPONSE="Last-Modified,Content-Type"
    export OTEL_EXPORTER_OTLP_TRACES_ENDPOINT="localhost:4317"
    export OTEL_SERVICE_NAME="movr-crdb"
    export OTEL_RESOURCE_ATTRIBUTES="crdb_release=12.2.4,app=movr"
    export OTEL_EXPORTER_OTLP_INSECURE=true
    export OTEL_EXPORTER_OTLP_SPAN_INSECURE=true
    export OTEL_EXPORTER_OTLP_METRIC_INSECURE=true
    ~~~

2. Start the application with Python and OTel parameters

    ~~~ shell
    opentelemetry-instrument --traces_exporter console,otlp \
    --metrics_exporter none \
    python server.py run --max-records 50 
    ~~~

3. Navigate to the url provided (defaults to [http://localhost:36257](http://localhost:36257)) to use the application.

### Clean up

1. To shut down the application, `Ctrl+C` out of the Python process.

## Note: Testing

We are still building out the testing suite. Please use at your own risk.
