### Jira Story

#### Summary:
Enhance Django App to Dynamically Configure ALLOWED_HOSTS for Prometheus Scraping

#### Description:
We need to enhance our Django application to dynamically configure the `ALLOWED_HOSTS` setting to include the pod's IP address and hostname. This change is crucial to ensure that Prometheus can successfully scrape metrics from the application, as Prometheus uses the pod IPs as targets, leading to `400 Bad Request` errors when the IP is not included in `ALLOWED_HOSTS`.

#### Reason for Errors:
The `400 Bad Request` errors occur due to Django's security feature, which uses the `ALLOWED_HOSTS` setting to prevent HTTP Host header attacks. When an incoming request's Host header does not match any value in `ALLOWED_HOSTS`, Django responds with a `400 Bad Request` error. Since Prometheus scrapes metrics using the pod's IP address, it is essential to dynamically include these IPs in the `ALLOWED_HOSTS` setting to allow Prometheus to access the metrics endpoint.

#### Acceptance Criteria:
1. Update the Helm deployment templates to use the Downward API for injecting the pod's IP address and hostname as environment variables.
2. Modify the `run.sh` script to dynamically append the pod IP and hostname to the `ALLOWED_HOSTS` environment variable.
3. Ensure the Django settings file is updated to use the `ALLOWED_HOSTS` environment variable.
4. Validate that Prometheus can successfully scrape metrics without encountering `400 Bad Request` errors.

#### Tasks:

1. **Update Helm Deployment Templates:**
   - Add environment variables for `POD_IP` and `POD_HOSTNAME` using the Downward API in the deployment YAML.
   - Example configuration to add in deployment YAML:
     ```yaml
     env:
     - name: POD_IP
       valueFrom:
         fieldRef:
           fieldPath: status.podIP
     - name: POD_HOSTNAME
       valueFrom:
         fieldRef:
           fieldPath: metadata.name
     - name: ALLOWED_HOSTS
       value: "<KNOWN_URLS>"
     ```

2. **Modify `run.sh` Script:**
   - Update the script to dynamically append the pod's IP address and hostname to `ALLOWED_HOSTS` and use fallback commands if necessary.
   - Ensure the script checks if the environment variables are set and falls back to using Linux commands to retrieve the pod IP and hostname if not set.
   - Ensure the script injects these values into memory prior to starting Gunicorn so that they are used in subsequent `settings.py` configuration.

3. **Update Django Settings:**
   - Ensure `settings.py` uses the `ALLOWED_HOSTS` environment variable.
   - Example configuration in `settings.py`:
     ```python
     import os

     ALLOWED_HOSTS = os.environ.get('ALLOWED_HOSTS', '').split(',')
     ```

4. **Testing and Validation:**
   - Deploy the updated application to a test environment.
   - Verify that the application correctly sets `ALLOWED_HOSTS` and is accessible via both `localhost` and the pod's IP address.
   - Ensure Prometheus can successfully scrape metrics without encountering `400 Bad Request` errors.

5. **Document Prometheus Scraping Errors:**
   - Note the errors observed due to Prometheus scraping using pod IPs.
   - Ensure the solution addresses the following error:
     ```
     ERROR Invalid HTTP_HOST header: '<pod-ip>:5000'. You may need to add '<pod-ip>' to ALLOWED_HOSTS.
     ```
---
Hello

---

### Jira Story

**Title**: Create Helm Template for ServiceMonitor with Enabled/Disabled Options and Parameterization

**Description**:
As a DevOps engineer, I need to create a Helm template for `ServiceMonitor` resources, providing options to enable or disable them and allowing parameterization for flexibility. This will facilitate easier management and deployment of Prometheus monitoring configurations in Kubernetes.

**Acceptance Criteria**:
1. A Helm template for `ServiceMonitor` is created under the `templates` directory.
2. The Helm template includes parameters to configure:
   - ServiceMonitor name
   - Namespace
   - Selector labels
   - Endpoints (port, interval, path, scheme)
   - TLS configuration
   - Job label
   - Target labels
   - Label matchers
3. The Helm chart includes values to enable or disable the creation of the `ServiceMonitor`.
4. The Helm values file includes default values for the `ServiceMonitor` configuration.
5. Documentation is updated to describe how to configure and use the `ServiceMonitor` Helm template.

**Tasks**:
1. **Create Helm Template**:
   - Create a new template file `servicemonitor.yaml` in the `templates` directory.
   - Define the `ServiceMonitor` resource with placeholders for parameterization.
  
2. **Add Enabled/Disabled Option**:
   - Add a condition in the template to create the `ServiceMonitor` only if enabled.
   - Update the `values.yaml` file to include an `enabled` field for the `ServiceMonitor`.

3. **Parameterize the Template**:
   - Add parameters for `ServiceMonitor` configurations (name, namespace, selector labels, endpoints, etc.).
   - Update the `values.yaml` file with default values for these parameters.

4. **Update Documentation**:
   - Update the `README.md` or relevant documentation files to describe how to enable and configure the `ServiceMonitor` using the Helm chart.

**Estimated LOE**: 8 hours

**Example Configuration in `values.yaml`**:
- `enabled`: Boolean to enable or disable the ServiceMonitor.
- `name`: Name of the ServiceMonitor.
- `namespace`: Namespace where the ServiceMonitor will be created.
- `selector.matchLabels`: Labels to select the target services.
- `endpoints`: List of endpoints to be monitored, including port, interval, path, scheme, and TLS configuration.
- `jobLabel`: Label to identify the job.
- `targetLabels`: Labels to transfer to the target.
- `labelMatchers`: Matchers to select specific pods or containers based on labels.

