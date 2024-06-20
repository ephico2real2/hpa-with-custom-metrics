To implement the restrictions only for the `/metrics` endpoint and not for the `/health` endpoint, you can adjust the middleware to check only the `/metrics` endpoint. Here’s the updated implementation:

### Step-by-Step Implementation

#### 1. Define `DEFAULT_METRICS_ALLOWED_HOSTS` and `METRICS_ALLOWED_HOSTS` in `settings.py`

```python
import os

ALLOWED_HOSTS = os.environ.get('ALLOWED_HOSTS', '').split(',')

MIDDLEWARE = [
    # ... other middleware classes ...
    'web_portal.metrics_middleware.RestrictMetricsMiddleware',
]

# Default allowed IPs for metrics endpoint
DEFAULT_METRICS_ALLOWED_HOSTS = ['127.0.0.1', '::1']

# Get additional allowed IPs for metrics from environment variable
additional_metrics_allowed_hosts = os.environ.get('METRICS_ALLOWED_HOSTS', '')
if additional_metrics_allowed_hosts:
    additional_metrics_allowed_hosts = additional_metrics_allowed_hosts.split(',')
else:
    additional_metrics_allowed_hosts = []

# Combine default and environment variable IPs
METRICS_ALLOWED_HOSTS = DEFAULT_METRICS_ALLOWED_HOSTS + additional_metrics_allowed_hosts
```

#### 2. Modify Middleware to Use `settings.METRICS_ALLOWED_HOSTS` and Only Check `/metrics`

Update `metrics_middleware.py` to read `METRICS_ALLOWED_HOSTS` from the settings and restrict access only to the `/metrics` endpoint.

```python
from django.conf import settings
from django.http import HttpResponseForbidden
import re

class RestrictMetricsMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
        self.metrics_allowed_hosts = settings.METRICS_ALLOWED_HOSTS

    def __call__(self, request):
        if re.match(r'^/metrics', request.path_info):
            host = request.get_host().split(':')[0]  # Get the hostname without port
            if host not in self.metrics_allowed_hosts:
                return HttpResponseForbidden("Forbidden")
        response = self.get_response(request)
        return response
```

#### 3. Ensure Middleware Path in `settings.py`

Ensure that the middleware is correctly referenced in the `MIDDLEWARE` list in `settings.py`.

```python
MIDDLEWARE = [
    # ... other middleware classes ...
    'web_portal.metrics_middleware.RestrictMetricsMiddleware',
]
```

#### 4. Update Deployment Configuration and `run.sh`

Ensure your Kubernetes deployment configuration and `run.sh` script are set correctly.

#### Kubernetes Deployment Configuration

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: your-deployment
  labels:
    app: your-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: your-app
  template:
    metadata:
      labels:
        app: your-app
    spec:
      containers:
      - name: your-container
        image: your-image
        env:
        - name: ALLOWED_HOSTS
          value: "localhost,127.0.0.1"
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: POD_HOSTNAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        command: ["/bin/sh"]
        args: ["-c", "/path/to/run.sh"]
```

#### `run.sh`

Update your `run.sh` script to dynamically append the pod's IP address and hostname to `METRICS_ALLOWED_HOSTS`.

```sh
#!/bin/sh

# Retrieve current ALLOWED_HOSTS value
ALLOWED_HOSTS=${ALLOWED_HOSTS:-""}

# Get pod's hostname and IP address from environment variables
POD_HOSTNAME=${POD_HOSTNAME:-$(hostname)}
POD_IP=${POD_IP:-$(hostname -i)}

# Construct new METRICS_ALLOWED_HOSTS value
METRICS_ALLOWED_HOSTS="${POD_HOSTNAME},${POD_IP}"

# Export the new METRICS_ALLOWED_HOSTS
export METRICS_ALLOWED_HOSTS

# Print the new METRICS_ALLOWED_HOSTS for debugging purposes (optional)
echo "Updated METRICS_ALLOWED_HOSTS: ${METRICS_ALLOWED_HOSTS}"

# Start Gunicorn
exec gunicorn --config /path/to/gunicorn/config.py myapp.wsgi:application
```

### Summary

1. **Define `DEFAULT_METRICS_ALLOWED_HOSTS` and `METRICS_ALLOWED_HOSTS` in `settings.py`**:
   - This ensures all configuration is centralized in `settings.py`.

2. **Modify Middleware to Use `settings.METRICS_ALLOWED_HOSTS` and Only Check `/metrics`**:
   - This ensures the middleware restricts access only to the `/metrics` endpoint.

3. **Ensure Middleware Path in `settings.py`**:
   - Reference the middleware correctly.

4. **Update Deployment Configuration and `run.sh`**:
   - Ensure the environment variables are set correctly to dynamically allow pod IP and hostname.

By following these steps, you ensure that the `/metrics` endpoint is secured and accessible only to the specified IP addresses or hostnames, while leaving the `/health` endpoint unrestricted. This approach provides a clean and maintainable configuration for securing the metrics endpoint.
