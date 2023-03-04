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
# CloudWatch logs
Added ```watchtower``` to requirements.txt 
```
cd /backend-flask
```
```
pip install -r requirements.txt
```
# Add to app.py
```
import watchtower
import logging
from time import strftime
```
```
# Configuring Logger to Use CloudWatch
LOGGER = logging.getLogger(__name__)
LOGGER.setLevel(logging.DEBUG)
console_handler = logging.StreamHandler()
cw_handler = watchtower.CloudWatchLogHandler(log_group='cruddur')
LOGGER.addHandler(console_handler)
LOGGER.addHandler(cw_handler)

```
```
@app.after_request
def after_request(response):
    timestamp = strftime('[%Y-%b-%d %H:%M]')
    LOGGER.error('%s %s %s %s %s %s', timestamp, request.remote_addr, request.method, request.scheme, request.full_path, response.status)
    return response
```
# Add to ```docker-compose.yml``` to set env var in backend-flask
```
	  AWS_DEFAULT_REGION: "${AWS_DEFAULT_REGION}"
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
```	  

# Docker Compose Up gives me errors on the backend-flask
![OeSxFzv](https://user-images.githubusercontent.com/124912958/222915070-66ffb6c9-25d0-4531-b439-4286e00c9a24.png)

Code exactly like Andrew's video so troubleshooting


# Success after changing logger.info to Logger.info in home_activities.py
![KR3C921](https://user-images.githubusercontent.com/124912958/222914934-4019c81f-7482-4eaf-a133-3f73e454a71b.png)


# However, no CloudWatch group created, investigating

I was signed into the wrong AWS account 
![pk1iUeX](https://user-images.githubusercontent.com/124912958/222914884-ffaa6edc-9d46-4363-bca4-ee681d5912f7.png)


# Install rollbar
Add to ```requirements.txt```

```
blinker
rollbar
```
# Install dependencies
```
pip install -r requirements.txt
```
# Set rollbar access token
```
export ROLLBAR_ACCESS_TOKEN=""
gp env ROLLBAR_ACCESS_TOKEN=""
```
# Add to backend-flask for ```docker-compose.yml```
```
ROLLBAR_ACCESS_TOKEN: "${ROLLBAR_ACCESS_TOKEN}"
```
# Import for rollbar into ```app.py```
```
import rollbar
import rollbar.contrib.flask
from flask import got_request_exception
```

```
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
	
# Add endpoint to ```app.py``` to test rollbar
```
@app.route('/rollbar/test')
def rollbar_test():
    rollbar.report_message('Hello World!', 'warning')
    return "Hello World!"	
```
# Rollbar Commit
https://github.com/MailGirl2TechGirl/aws-bootcamp-cruddur-2023/commit/39545278609befd1c176e30a7dee3a142d278ec6
# Working! 

![iKBwc6E](https://user-images.githubusercontent.com/124912958/222914803-f8eab4ab-e338-47f5-98dc-0220c820dfc2.png)

![8H1L9R5](https://user-images.githubusercontent.com/124912958/222914757-e1f6d5b7-772d-4157-a7c3-19f8bd3b3f21.png)

	

