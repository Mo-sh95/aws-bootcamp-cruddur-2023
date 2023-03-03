# Week 2 — Distributed Tracing
## Honeycomb
### Installing packages, Initializing and Configuring Honeycomb:
By refering to the python section in this [doc](https://ui.honeycomb.io/mo.shaaban1995-gettingstarted/environments/cruddur/send-data#)
#### adding these packages to the ```requirements.txt``` file
```
opentelemetry-api 
opentelemetry-sdk 
opentelemetry-exporter-otlp-proto-http 
opentelemetry-instrumentation-flask 
opentelemetry-instrumentation-requests
```
Either:
```sh
pip install -r requirements.txt
```
Or:
Trigger a new gitpod environment!

#### adding this section to the ```app.py``` file
```py
# app.py updates
    
from opentelemetry import trace
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor

# Initialize tracing and an exporter that can send data to Honeycomb
provider = TracerProvider()
processor = BatchSpanProcessor(OTLPSpanExporter())
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)
tracer = trace.get_tracer(__name__)

# Initialize automatic instrumentation with Flask
app = Flask(__name__)
FlaskInstrumentor().instrument_app(app)
RequestsInstrumentor().instrument()
```
#### Configuring Honeycomb Env Vars for backend-flask in the ```docker-compose.yml``` file
under ```services:backend_flask:environment```, define these vars
```yml
OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"
OTEL_SERVICE_NAME: "backend_flask"
```
then define this var in gitpod:
```sh
gp env HONEYCOMB_API_KEY="UjDlY...."
```

## AWS X-Ray
### Instrumenting the backend-flask:
#### Installing the required python dependencies
```sh
cat << EOF >> /backend-flask/requirements.txt
aws-xray-sdk
EOF
```
```sh
pip install -r requirements.txt
```
#### Editing the ```app.py``` file
```py
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware
```
#### Creating a sampling rule for collecting specific traces
**Note**: you can do it via GUI as well but in our case we'll do it using AWS CLI:

vim ```~/json/xray.json```
```json
{
  "SamplingRule": {
      "RuleName": "Cruddur-backend-flask",
      "ResourceARN": "*",
      "Priority": 9000,
      "FixedRate": 0.1,
      "ReservoirSize": 5,
      "ServiceName": "backend-flask",
      "ServiceType": "*",
      "Host": "*",
      "HTTPMethod": "*",
      "URLPath": "*",
      "Version": 1
  }
}
```
``` aws xray create-sampling-rule --cli-input-json file://json/xray.json ```

#### Creating an X-Ray group for the backend-flask service to group its traces together
```sh
aws xray create-group \
   --group-name "Cruddur-Backend-flask" \
   --filter-expression "service(\"backend-flask\")"
```
#### Running the X-Ray daemon as a container:
```vim ~/docker-compose.yml ```
then add this section:
```yml
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
then add these env vars to the **backend-flask** container
```yml
 AWS_XRAY_URL: "*4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}*"
 AWS_XRAY_DAEMON_ADDRESS: "xray-daemon:2000"
```
#### Checking service data for the last 5 mins:
```sh
EPOCH=$(date +%s)
aws xray get-service-graph --start-time $(($EPOCH-300)) --end-time $EPOCH
```
<details>
<summary>click to view stdout</summary>

```json
    {
    "Services": [
        {
            "ReferenceId": 0,
            "Name": "4567-mosh95-awsbootcampcrudd-zl9lgt6sp3k.ws-eu89.gitpod.io",
            "Names": [
                "4567-mosh95-awsbootcampcrudd-zl9lgt6sp3k.ws-eu89.gitpod.io"
            ],
            "Root": true,
            "State": "active",
            "StartTime": "2023-03-03T16:05:54+00:00",
            "EndTime": "2023-03-03T16:07:49+00:00",
            "Edges": [],
            "SummaryStatistics": {
                "OkCount": 4,
                "ErrorStatistics": {
                    "ThrottleCount": 0,
                    "OtherCount": 1,
                    "TotalCount": 1
                },
                "FaultStatistics": {
                    "OtherCount": 0,
                    "TotalCount": 0
                },
                "TotalCount": 5,
                "TotalResponseTime": 0.004
            },
            "DurationHistogram": [
                {
                    "Value": 0.001,
                    "Count": 5
                }
            ],
            "ResponseTimeHistogram": [
                {
                    "Value": 0.001,
                    "Count": 5
                }
            ]
        },
        {
            "ReferenceId": 1,
            "Name": "4567-mosh95-awsbootcampcrudd-zl9lgt6sp3k.ws-eu89.gitpod.io",
            "Names": [
                "4567-mosh95-awsbootcampcrudd-zl9lgt6sp3k.ws-eu89.gitpod.io"
            ],
            "Type": "client",
            "State": "unknown",
            "StartTime": "2023-03-03T16:05:54+00:00",
            "EndTime": "2023-03-03T16:07:49+00:00",
            "Edges": [
                {
                    "ReferenceId": 0,
                    "StartTime": "2023-03-03T16:05:54+00:00",
                    "EndTime": "2023-03-03T16:07:49+00:00",
                    "SummaryStatistics": {
                        "OkCount": 4,
                        "ErrorStatistics": {
                            "ThrottleCount": 0,
                            "OtherCount": 1,
                            "TotalCount": 1
                        },
                        "FaultStatistics": {
                            "OtherCount": 0,
                            "TotalCount": 0
                        },
                        "TotalCount": 5,
                        "TotalResponseTime": 0.004
                    },
                    "ResponseTimeHistogram": [
                        {
                            "Value": 0.001,
                            "Count": 5
                        }
                    ],
                    "Aliases": []
                }
            ]
        },
        {
            "ReferenceId": 2,
            "Name": "backend-flask",
            "Names": [
                "backend-flask"
            ],
            "Root": true,
            "State": "active",
            "StartTime": "2023-03-03T16:04:04+00:00",
            "EndTime": "2023-03-03T16:08:05+00:00",
            "Edges": [],
            "SummaryStatistics": {
                "OkCount": 5,
                "ErrorStatistics": {
                    "ThrottleCount": 0,
                    "OtherCount": 0,
                    "TotalCount": 0
                },
                "FaultStatistics": {
                    "OtherCount": 0,
                    "TotalCount": 0
                },
                "TotalCount": 5,
                "TotalResponseTime": 0.004
            },
            "DurationHistogram": [
                {
                    "Value": 0.001,
                    "Count": 5
                }
            ],
            "ResponseTimeHistogram": [
                {
                    "Value": 0.001,
                    "Count": 5
                }
            ]
        },
        {
            "ReferenceId": 3,
            "Name": "backend-flask",
            "Names": [
                "backend-flask"
            ],
            "Type": "client",
            "State": "unknown",
            "StartTime": "2023-03-03T16:04:04+00:00",
            "EndTime": "2023-03-03T16:08:05+00:00",
            "Edges": [
                {
                    "ReferenceId": 2,
                    "StartTime": "2023-03-03T16:04:04+00:00",
                    "EndTime": "2023-03-03T16:08:05+00:00",
                    "SummaryStatistics": {
                        "OkCount": 5,
                        "ErrorStatistics": {
                            "ThrottleCount": 0,
                            "OtherCount": 0,
                            "TotalCount": 0
                        },
                        "FaultStatistics": {
                            "OtherCount": 0,
                            "TotalCount": 0
                        },
                        "TotalCount": 5,
                        "TotalResponseTime": 0.004
                    },
                    "ResponseTimeHistogram": [
                        {
                            "Value": 0.001,
                            "Count": 5
                        }
                    ],
                    "Aliases": []
                }
            ]
        }
    ],
    "StartTime": "2023-03-03T16:04:04+00:00",
    "EndTime": "2023-03-03T16:08:04+00:00",
    "ContainsOldGroupVersions": false
    }
```
</details>

![](assets/xray.png)
![](assets/xray2.png)

## CloudWatch Logs
#### Installing the required python dependencies
```sh
cat << EOF >> /backend-flask/requirements.txt
watchtower
EOF
```

#### Editing the ```app.py``` file
```py
import watchtower
import logging
from time import strftime
```
```py
# Configuring Logger to Use CloudWatch
LOGGER = logging.getLogger(__name__)
LOGGER.setLevel(logging.DEBUG)
console_handler = logging.StreamHandler()
cw_handler = watchtower.CloudWatchLogHandler(log_group='cruddur')
LOGGER.addHandler(console_handler)
LOGGER.addHandler(cw_handler)
LOGGER.info("Test Log")
```
```py
@app.after_request
def after_request(response):
    timestamp = strftime('[%Y-%b-%d %H:%M]')
    LOGGER.error('%s %s %s %s %s %s', timestamp, request.remote_addr, request.method, request.scheme, request.full_path, response.status)
    return response
```
```py
@app.route("/api/activities/home", methods=['GET'])
def data_home():
  data = HomeActivities.run(logger=LOGGER)
  return data, 200
```
in the ```backend-flask/services/home_activities.py ``` file add this section:
```py
class HomeActivities:
  def run(logger):
    logger.info("HomeActivities")
```
Add these env vars in the ```docker-compose.yml``` file for backend-flask:
```yml
      AWS_DEFAULT_REGION: "${AWS_DEFAULT_REGION}"
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
```
![](assets/cw.png)
![](assets/cw2.png)

**Note** I disabled both X-Ray and CloudWatch after testing them

## Rollbar
#### Create a new project in Rollbar named **Cruddur**
#### Installing the required python dependencies
```sh
cat << EOF >> /backend-flask/requirements.txt
blinker
rollbar
EOF
```
```sh
pip install -r requirements.txt
```
```sh
export ROLLBAR_ACCESS_TOKEN="21dcd84...."
gp env ROLLBAR_ACCESS_TOKEN="21dcd84...."
```
Add this env var to **backend-flask** service in the **docker-compose** file:
```yml
ROLLBAR_ACCESS_TOKEN: "${ROLLBAR_ACCESS_TOKEN}"
```
#### Editing the ```app.py``` file
```py
import os
import rollbar
import rollbar.contrib.flask
from flask import got_request_exception
```
```py
rollbar_access_token = os.getenv('ROLLBAR_ACCESS_TOKEN')
@app.before_first_request
def init_rollbar():
    """init rollbar module"""
    rollbar.init(
        # access token
        rollbar_access_token,
        # environment name
        'production',
        # server root directory, makes tracebacks prettier
        root=os.path.dirname(os.path.realpath(__file__)),
        # flask already sets up logging
        allow_logging_basic_config=False)

    # send exceptions from `app` to rollbar, using flask's signal system.
    got_request_exception.connect(rollbar.contrib.flask.report_exception, app)
```
```py
@app.route('/rollbar/test')
def rollbar_test():
    rollbar.report_message('Hello World!', 'warning')
    return "Hello World!"
```
