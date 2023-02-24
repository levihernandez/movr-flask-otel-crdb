```shell
pip install opentelemetry-distro \
opentelemetry-instrumentation-flask \
opentelemetry-distro opentelemetry-exporter-otlp \
opentelemetry-exporter-otlp-proto-grpc \
opentelemetry-instrumentation-psycopg2 \
flask \
requests

export OTEL_INSTRUMENTATION_HTTP_CAPTURE_HEADERS_SERVER_REQUEST="Accept-Encoding,User-Agent,Referer"
export OTEL_INSTRUMENTATION_HTTP_CAPTURE_HEADERS_SERVER_RESPONSE="Last-Modified,Content-Type"
export OTEL_EXPORTER_OTLP_TRACES_ENDPOINT="192.168.86.62:4317"
export OTEL_SERVICE_NAME="movr-crdb"
export OTEL_RESOURCE_ATTRIBUTES="crdb_release=12.2.4,app=movr"
export OTEL_EXPORTER_OTLP_INSECURE=true
export OTEL_EXPORTER_OTLP_SPAN_INSECURE=true
export OTEL_EXPORTER_OTLP_METRIC_INSECURE=true

opentelemetry-instrument --traces_exporter console,otlp \
--metrics_exporter none \
python server.py run --max-records 50 
```
