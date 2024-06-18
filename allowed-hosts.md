
# enhancement
---
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
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: POD_HOSTNAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: ALLOW_HOSTS
          value: "localhost,127.0.0.1"
        command: ["/bin/sh"]
        args: ["-c", "/path/to/run.sh"]
```

# run.sh

```bash
#!/bin/sh

# Retrieve current ALLOWED_HOSTS value
ALLOWED_HOSTS=${ALLOWED_HOSTS:-""}

# Get pod's hostname and IP address from environment variables or use fallback commands
POD_HOSTNAME=${POD_HOSTNAME:-$(hostname)}
POD_IP=${POD_IP:-$(hostname -i)}

# Construct new ALLOWED_HOSTS value
NEW_ALLOWED_HOSTS="$ALLOWED_HOSTS,$POD_HOSTNAME,$POD_IP"

# Export the new ALLOWED_HOSTS
export ALLOWED_HOSTS=$NEW_ALLOWED_HOSTS

# Print the new ALLOWED_HOSTS for debugging purposes (optional)
echo "Updated ALLOWED_HOSTS: $ALLOWED_HOSTS"

# Start Gunicorn
exec gunicorn --config /path/to/gunicorn/config.py myapp.wsgi:application
```

---
