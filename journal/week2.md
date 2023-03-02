# Week 2 — Distributed Tracing

# Set env vars for honeycomb
```
export HONEYCOMB_API_KEY=""
export HONEYCOMB_SERVICE_NAME="Cruddur"
gp env HONEYCOMB_API_KEY=""
gp env HONEYCOMB_SERVICE_NAME="Cruddur"
```

# Set service name
 ```OTEL_SERVICE_NAME: 'backend-flask'```
 
 # Added code to requirements.txt
```
opentelemetry-api 
opentelemetry-sdk 
opentelemetry-exporter-otlp-proto-http 
opentelemetry-instrumentation-flask 
opentelemetry-instrumentation-requests
```

# Install requirements
```
pip install -r requirements.txt
```

# Adding to app.py
```
# HoneyComb ------------
from opentelemetry import trace
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import ConsoleSpanExporter, BatchSpanProcessor
```


# Initialize tracing and an exporter that can send data to Honeycomb
```
provider = TracerProvider()
processor = BatchSpanProcessor(OTLPSpanExporter())
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)
tracer = trace.get_tracer(__name__)

app = Flask(__name__)

#HoneyComb -------
FlaskInstrumentor().instrument_app(app)
RequestsInstrumentor().instrument()
```
# Updated my gitpod.yml file to make ports public automatically
```
ports: 
  - name: frontend 
    port: 3000
    onOpen: open-browser
    visibility: public
  - name: backend 
    port: 4567
    visibility: public
  - name: xray-daemon
    port: 2000
    visibility: public
```    
# 	Added these lines to app.py

```
from opentelemetry.sdk.trace.export import ConsoleSpanExporter, SimpleSpanProcessor


simple_processor = SimpleSpanProcessor(ConsoleSpanExporter())
provider.add_span_processor(simple_processor)
```
# Docker compose up and started seeing traces on honeycomb 

![image](https://user-images.githubusercontent.com/124912958/222210686-64a41df8-6a31-4664-aa5f-f57f39f0a7b4.png)

# Added to requirements.txt
```
aws-xray-sdk
```
# Installed aws-xray-sdk
cd /backend-flask
```
pip install -r requirements.txt
```
# Add to app.py
```
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware


xray_url = os.getenv("AWS_XRAY_URL")
xray_recorder.configure(service='backend-flask', dynamic_naming=xray_url)
XRayMiddleware(app, xray_recorder)
```

# Create x-ray json
```
{
  "SamplingRule": {
      "RuleName": "Cruddur",
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
# Create aws xray group 
```
aws xray create-group \
   --group-name "Cruddur" \
   --filter-expression "service(\"$FLASK_ADDRESS\")"
   ```
   
 #   Create sampling rule
 ```
 aws xray create-sampling-rule --cli-input-json file://aws/json/xray.json
 ```
 
 # Add daemon service to docker compose
 ```
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

# Add env vars to backend-flask docker compose

```
AWS_XRAY_URL: "*4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}*"
AWS_XRAY_DAEMON_ADDRESS: "xray-daemon:2000"
```
