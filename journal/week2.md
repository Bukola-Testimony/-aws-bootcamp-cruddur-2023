# Week 2 — Distributed Tracing

For week 2, I Instrumented the backend flask application to use Open Telemetry (OTEL) with Honeycomb.io as the provider.


First I exported and added the HoneyComb API key environment variables to Gitpod

```bash
export HONEYCOMB_API_KEY=""
gp env HONEYCOMB_API_KEY=""
```
![Honeycomb API](/images/week-2/honeycomb-env-API.png)

I also added the following Env Vars to backend-flask in docker compose

```bash
OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"
OTEL_SERVICE_NAME: "${HONEYCOMB_SERVICE_NAME}"
```

To create Dataset in Honeycomb, all these installation instructions were added to the requirements.txt file
```
opentelemetry-api 
opentelemetry-sdk 
opentelemetry-exporter-otlp-proto-http 
opentelemetry-instrumentation-flask 
opentelemetry-instrumentation-requests
```

I ran the following command to install the dependencies:

```
pip install -r requirements.txt
```

After running "docker-compose up" I was able to view the Datasets and trace activities on Honeycomb.io.

## Home
> ![](/images/week-2/honeycomb-awsbootcamp-home.png)

<br>

## Routes
> ![](/images/week-2/honeycomb-awsbootcamp-route.png)

<br>

## Service
> ![](/images/week-2/honeycomb-awsbootcamp-service.png)

<br>

## I ran queries to explore traces within Honeycomb.io
> ![](/images/week-2/honeycomb-awsbootcamp-query.png)

> ![](/images/week-2/honeycomb-awsbootcamp-trace1-mock-data.png)

<br>

## I also learnt some troubleshoot skills for honeycomb API:
- Run : env | grep HONEY
- copy env API keys
- Next, Go to the url : honeycomb-whoami.glitch.me
- Paste the API keys in the search box.  







