

### Step-by-Step Implementation

#### 1. Update `settings.py` to Handle `METRICS_ALLOWED_HOSTS`

Update `settings.py` to read the `METRICS_ALLOWED_HOSTS` environment variable and combine it with default allowed IPs.

```python
import os

ALLOWED_HOSTS = os.environ.get('ALLOWED_HOSTS', '').split(',')

# Default allowed IPs for metrics
DEFAULT_METRICS_ALLOWED_HOSTS = ['127.0.0.1', '::1']

# Get additional allowed IPs for metrics from environment variable
METRICS_ALLOWED_HOSTS = os.environ.get('METRICS_ALLOWED_HOSTS', '')
if METRICS_ALLOWED_HOSTS:
    METRICS_ALLOWED_HOSTS = METRICS_ALLOWED_HOSTS.split(',')
else:
    METRICS_ALLOWED_HOSTS = []

# Combine default and environment variable IPs
METRICS_ALLOWED_HOSTS = DEFAULT_METRICS_ALLOWED_HOSTS + METRICS_ALLOWED_HOSTS
```

#### 2. Create Middleware (`metrics_middleware.py`) to Use Environment Variables

```python
from django.http import HttpResponseForbidden
import re
import os

class RestrictMetricsMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
        self.default_metrics_allowed_hosts = ['127.0.0.1', '::1']
        self.metrics_allowed_hosts = os.environ.get('METRICS_ALLOWED_HOSTS', '')

        if self.metrics_allowed_hosts:
            self.metrics_allowed_hosts = self.metrics_allowed_hosts.split(',')
        else:
            self.metrics_allowed_hosts = []

        self.metrics_allowed_hosts = self.default_metrics_allowed_hosts + self.metrics_allowed_hosts

    def __call__(self, request):
        if re.match(r'^/metrics', request.path_info):
            ip = request.META.get('REMOTE_ADDR')
            if ip not in self.metrics_allowed_hosts:
                return HttpResponseForbidden("Forbidden")
        response = self.get_response(request)
        return response
```

#### 3. Update Django `settings.py` to Include Middleware

```python
MIDDLEWARE = [
    # ... other middleware classes ...
    'myapp.middleware.metrics_middleware.RestrictMetricsMiddleware',
]
```

#### 4. Configure Kubernetes Deployment

Use Kubernetes Downward API to inject necessary environment variables for pod IP and hostname.

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

### Integrate with `run.sh`

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

1. **Update `settings.py`** to handle `METRICS_ALLOWED_HOSTS`.
2. **Create Middleware** (`metrics_middleware.py`) to restrict access to the `/metrics` endpoint.
3. **Update `settings.py`** to include the middleware.
4. **Configure Kubernetes Deployment** to use the Downward API for injecting the pod's IP address and hostname into environment variables.
5. **Update `run.sh`** to dynamically set `METRICS_ALLOWED_HOSTS` with the pod's IP address and hostname.

This setup ensures that the `/metrics` endpoint is secured and accessible only to the specified IP addresses, providing a robust and dynamic solution for Prometheus scraping. Here is the complete example:

#### `metrics_middleware.py`

```python
from django.http import HttpResponseForbidden
import re
import os

class RestrictMetricsMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
        self.default_metrics_allowed_hosts = ['127.0.0.1', '::1']
        self.metrics_allowed_hosts = os.environ.get('METRICS_ALLOWED_HOSTS', '')

        if self.metrics_allowed_hosts:
            self.metrics_allowed_hosts = self.metrics_allowed_hosts.split(',')
        else:
            self.metrics_allowed_hosts = []

        self.metrics_allowed_hosts = self.default_metrics_allowed_hosts + self.metrics_allowed_hosts

    def __call__(self, request):
        if re.match(r'^/metrics', request.path_info):
            ip = request.META.get('REMOTE_ADDR')
            if ip not in self.metrics_allowed_hosts:
                return HttpResponseForbidden("Forbidden")
        response = self.get_response(request)
        return response
```

#### `settings.py`

```python
import os

ALLOWED_HOSTS = os.environ.get('ALLOWED_HOSTS', '').split(',')

MIDDLEWARE = [
    # ... other middleware classes ...
    'myapp.middleware.metrics_middleware.RestrictMetricsMiddleware',
]

# Default allowed IPs for metrics
DEFAULT_METRICS_ALLOWED_HOSTS = ['127.0.0.1', '::1']

# Get additional allowed IPs for metrics from environment variable
METRICS_ALLOWED_HOSTS = os.environ.get('METRICS_ALLOWED_HOSTS', '')
if METRICS_ALLOWED_HOSTS:
    METRICS_ALLOWED_HOSTS = METRICS_ALLOWED_HOSTS.split(',')
else:
    METRICS_ALLOWED_HOSTS = []

# Combine default and environment variable IPs
METRICS_ALLOWED_HOSTS = DEFAULT_METRICS_ALLOWED_HOSTS + METRICS_ALLOWED_HOSTS
```

#### `run.sh`

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

This approach dynamically sets and appends the necessary IP addresses and hostnames to `METRICS_ALLOWED_HOSTS`, ensuring secure and flexible access to the `/metrics` endpoint.




#############

Understood. We need to ensure that `METRICS_ALLOWED_HOSTS` functions similarly to `ALLOWED_HOSTS`, but specifically for the `/metrics` endpoint, and we should not use `REMOTE_ADDR` for validation. Instead, we can directly use `POD_IP` and `POD_HOSTNAME` to configure allowed hosts for the metrics endpoint.

### Solution Approach

1. **Set Environment Variables for `POD_IP` and `POD_HOSTNAME`.**
2. **Modify the Middleware to Check `POD_IP` and `POD_HOSTNAME` Against `METRICS_ALLOWED_HOSTS`.**

### Step-by-Step Implementation

#### 1. Update `settings.py` to Handle `METRICS_ALLOWED_HOSTS`

Update `settings.py` to read the `METRICS_ALLOWED_HOSTS` environment variable and combine it with default allowed hosts.

```python
import os

ALLOWED_HOSTS = os.environ.get('ALLOWED_HOSTS', '').split(',')

MIDDLEWARE = [
    # ... other middleware classes ...
    'web_portal.metrics_middleware.RestrictMetricsMiddleware',
]

# Default allowed hosts for metrics
DEFAULT_METRICS_ALLOWED_HOSTS = ['127.0.0.1', '::1']

# Get additional allowed hosts for metrics from environment variable
METRICS_ALLOWED_HOSTS = os.environ.get('METRICS_ALLOWED_HOSTS', '')
if METRICS_ALLOWED_HOSTS:
    METRICS_ALLOWED_HOSTS = METRICS_ALLOWED_HOSTS.split(',')
else:
    METRICS_ALLOWED_HOSTS = []

# Combine default and environment variable hosts
METRICS_ALLOWED_HOSTS = DEFAULT_METRICS_ALLOWED_HOSTS + METRICS_ALLOWED_HOSTS
```

#### 2. Create Middleware (`metrics_middleware.py`) to Use `POD_IP` and `POD_HOSTNAME`

Modify the middleware to check against the `METRICS_ALLOWED_HOSTS` list.

```python
from django.http import HttpResponseForbidden
import re
import os

class RestrictMetricsMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
        self.default_metrics_allowed_hosts = ['127.0.0.1', '::1']
        self.metrics_allowed_hosts = os.environ.get('METRICS_ALLOWED_HOSTS', '')

        if self.metrics_allowed_hosts:
            self.metrics_allowed_hosts = self.metrics_allowed_hosts.split(',')
        else:
            self.metrics_allowed_hosts = []

        self.metrics_allowed_hosts = self.default_metrics_allowed_hosts + self.metrics_allowed_hosts

    def __call__(self, request):
        if re.match(r'^/metrics', request.path_info):
            host = request.get_host().split(':')[0]  # Get the hostname without port
            if host not in self.metrics_allowed_hosts:
                return HttpResponseForbidden("Forbidden")
        response = self.get_response(request)
        return response
```

#### 3. Update Django `settings.py` to Include Middleware

Ensure that the middleware is correctly added to the `MIDDLEWARE` list in `settings.py`:

```python
MIDDLEWARE = [
    # ... other middleware classes ...
    'web_portal.metrics_middleware.RestrictMetricsMiddleware',
]
```

#### 4. Configure Kubernetes Deployment

Use Kubernetes Downward API to inject necessary environment variables for pod IP and hostname.

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

#### 5. Update `run.sh`

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

1. **Update `settings.py`** to handle `METRICS_ALLOWED_HOSTS`.
2. **Create Middleware** (`metrics_middleware.py`) to restrict access to the `/metrics` endpoint based on hostname or IP address.
3. **Update `settings.py`** to include the middleware.
4. **Configure Kubernetes Deployment** to use the Downward API for injecting the pod's IP address and hostname into environment variables.
5. **Update `run.sh`** to dynamically set `METRICS_ALLOWED_HOSTS` with the pod's IP address and hostname.

This approach ensures that the `/metrics` endpoint is secured and accessible only to the specified hostnames or IP addresses, providing a robust and dynamic solution for Prometheus scraping.
