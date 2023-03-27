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

<br>

# AWS X-ray

The following command was added to the requirement.txt file in the backend-flask path. 
```
aws-xray-sdk
```
I installed AWS sdk for python with  the following command:
```bash
pip install aws-xray-sdk
```




Next I added the following to json file path: `aws/json/xray.json`
```json
{
  "SamplingRule": {
      "RuleName": "Cruddur",
      "ResourceARN": "*",
      "Priority": 9000,
      "FixedRate": 0.1,
      "ReservoirSize": 5,
      "ServiceName": "Cruddur",
      "ServiceType": "*",
      "Host": "*",
      "HTTPMethod": "*",
      "URLPath": "*",
      "Version": 1
  }
}
```

The `app.py` file was updated with the following:
```python
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware

xray_url = os.getenv("AWS_XRAY_URL")
xray_recorder.configure(service='Cruddur', dynamic_naming=xray_url)
XRayMiddleware(app, xray_recorder)
```
<br>

### To create a group named Cruddur in AWS-xray, I ran the following command.
```bash
aws xray create-group \
   --group-name "Cruddur" \
   --filter-expression "service(\"backend-flask\")"
```

![](/images/week-2/gitpod-create-xray-group.png)

### Group created in AWS
![](/images/week-2/aws-xray-group-cruddur.png)

### To create sampling rules in AWS. I ran the following command:

```bash
aws xray create-sampling-rule --cli-input-json file://aws/json/xray.json
```

![](/images/week-2/gitpod-create-sampling-rule.png)

![](/images/week-2/aws-xray-sampling-rule.png)


I  Updated docker-compose.yaml with aws-xray daemon environment variables
```yaml
  xray-daemon:
    image: "amazon/aws-xray-daemon"
    environment:
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
      AWS_REGION: "us-east-1"
    command:
      - "xray -o -b xray-daemon:2000"
    ports:
      - 2000:2000/udp
```
And this:
```yaml
AWS_XRAY_URL: "*4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}*"
      AWS_XRAY_DAEMON_ADDRESS: "xray-daemon:2000"
```
### Troubleshoot
Next I ran docker-compose up.
Here only the frontend port was running. I did some troubleshooting by checking the log files and was able to trace the problem to a file in the backend-flask. I corrected it and ran docker compose up again. Now the backend port was running. AWS-xray port was still not running at this point.

### AWS-xray traces and queries
![](/images/week-2/aws-xray-traces-query2.png)
![](/images/week-2/aws-xray-group-traces.png)
![](/images/week-2/aws-xray-traces-query-bukola-mockdata.png)
![](/images/week-2/aws-xray-traces-query3.png)
![](/images/week-2/aws-xray-traces-query.png)

<br>


# CloudWatch Logs

To Install watchtower, the following command was added to the requirements.txt file.
```
watchtower
```
The follwing command was used to install the requirements.txt file.
```bash
pip install -r requirements.txt
```

I added the following lines to app.py
```python
import watchtower
import logging
from time import strftime
```

```python
# Configuring Logger to Use CloudWatch
LOGGER = logging.getLogger(__name__)
LOGGER.setLevel(logging.DEBUG)
console_handler = logging.StreamHandler()
cw_handler = watchtower.CloudWatchLogHandler(log_group='cruddur')
LOGGER.addHandler(console_handler)
LOGGER.addHandler(cw_handler)
LOGGER.info("some message")
@app.after_request
def after_request(response):
```


```python
    timestamp = strftime('[%Y-%b-%d %H:%M]')
    LOGGER.error('%s %s %s %s %s %s', timestamp, request.remote_addr, request.method, request.scheme, request.full_path, response.status)
    return response
```

I Set the env variables in the backend-flask for docker-compose.yaml

```yaml
      AWS_DEFAULT_REGION: "${AWS_DEFAULT_REGION}"
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
```      
After all the additions, I ran the `docker-compose up` command.


![](/images/week-2/cloudwatch-ports-gitpod.png)

![](/images/week-2/cloudwatch-gitpod-backend.png)

### I went on to check the logs created in AWS Cloudwatch.
### Log group "Cruddur"
![](/images/week-2/cloudwatch-aws-logs-group.png)

### Logstream
![](/images/week-2/cloudwatch-aws-logstream1.png)

### Log events
![](/images/week-2/cloudwatch-aws-log-events1.png)

![](/images/week-2/cloudwatch-aws-log-events2.png)
