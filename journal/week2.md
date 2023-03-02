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
then running ```pip install -r requirements.tx```
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
