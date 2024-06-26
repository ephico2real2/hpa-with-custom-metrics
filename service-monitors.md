### Explanation of `metricRelabelings`

`metricRelabelings` in Prometheus `ServiceMonitor` configuration allows you to modify metrics as they are ingested into Prometheus. This can be useful for cleaning up, modifying, or dropping unwanted labels and metrics. Each `metricRelabeling` rule consists of several fields that control how the metrics are processed.

#### Fields in `metricRelabelings`

1. **action:**
   - Defines the action to be performed on the metric.
   - Common actions include:
     - `replace`: Replace the value of a label.
     - `keep`: Keep only the metrics that match the criteria.
     - `drop`: Drop the metrics that match the criteria.
     - `hashmod`: Replace the value of a label with a modulo hash.
     - `labelmap`: Rename label names according to a regex pattern.
     - `labeldrop`: Drop labels that match the regex pattern.
     - `labelkeep`: Keep only labels that match the regex pattern.

2. **regex:**
   - A regular expression used to match the label values.
   - Only metrics whose label values match the regex will be affected by the action.

3. **replacement:**
   - The new value to replace the original label value.
   - Used in combination with the `replace` action.

4. **targetLabel:**
   - The label that the action will apply to.
   - Specifies which label's value should be modified.

5. **sourceLabels:**
   - A list of labels whose values are concatenated and matched against the `regex`.
   - Useful when you want to perform an action based on the values of multiple labels.

#### Example `metricRelabelings`

Here's an example configuration in the context of your `values.yaml`:

```yaml
serviceMonitor:
  enabled: true
  team:
    name: golden-state
    channel: team-golden-state
    thanosMonitored: "true"
  additionalLabels: {}
  honorLabels: false
  interval: "30s"
  jobLabel: ""
  metricRelabelings:
    - action: replace
      replacement: golden-state
      targetLabel: team
    - action: replace
      replacement: demo-insights
      targetLabel: service
    - action: keep
      regex: 'demo_insights_.*'
      sourceLabels:
        - __name__
    - action: drop
      regex: '/metrics'
      sourceLabels:
        - __name__
  path: "/metrics"
  relabelings: {}
  scrapeTimeout: "25s"
namespace: datajar-django-sbox
```

### Exclude Polling for Metrics on `/metrics`

To exclude polling for metrics on the `/metrics` endpoint from being counted, you can add a `metricRelabeling` rule with the `drop` action. This will drop any metrics that have the `/metrics` path in their label value.

Here's how to do it in the `metricRelabelings` section:

```yaml
serviceMonitor:
  enabled: true
  team:
    name: golden-state
    channel: team-golden-state
    thanosMonitored: "true"
  additionalLabels: {}
  honorLabels: false
  interval: "30s"
  jobLabel: ""
  metricRelabelings:
    - action: replace
      replacement: golden-state
      targetLabel: team
    - action: replace
      replacement: demo-insights
      targetLabel: service
    - action: keep
      regex: 'demo_insights_.*'
      sourceLabels:
        - __name__
    - action: drop
      regex: '/metrics'
      sourceLabels:
        - __name__
  path: "/metrics"
  relabelings: {}
  scrapeTimeout: "25s"
namespace: datajar-django-sbox
```

### Updated `servicemonitor.yaml`

Here is the updated Helm template for the `ServiceMonitor`, including the new `metricRelabeling` rule:

```yaml
{{- if .Values.serviceMonitor.enabled }}
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: {{ include "portal.name" . }}-service-monitor
  namespace: {{ .Values.namespace | quote }}
  labels:
    {{- include "portal.labels" . | nindent 4 }}
    demo.com/team-name: {{ .Values.serviceMonitor.team.name }}
    demo.com/team-channel: {{ .Values.serviceMonitor.team.channel }}
    demo.com/thanos-monitored: {{ .Values.serviceMonitor.team.thanosMonitored }}
    {{- with .Values.serviceMonitor.additionalLabels }}
      {{- toYaml . | nindent 4 }}
    {{- end }}
spec:
  namespaceSelector:
    matchNames:
      - {{ .Values.namespace | quote }}
  selector:
    matchLabels:
      {{- include "portal.labels" . | nindent 6 }}
  endpoints:
    - port: {{ .Values.serviceMonitor.port | quote }}
      path: {{ .Values.serviceMonitor.path | quote }}
      interval: {{ .Values.serviceMonitor.interval | quote }}
      scrapeTimeout: {{ .Values.serviceMonitor.scrapeTimeout | quote }}
      {{- if .Values.serviceMonitor.honorLabels }}
      honorLabels: {{ .Values.serviceMonitor.honorLabels }}
      {{- end }}
      {{- with .Values.serviceMonitor.metricRelabelings }}
      metricRelabelings:
        {{- range . }}
        - action: {{ .action }}
          {{- if .regex }}
          regex: '{{ .regex }}'
          {{- end }}
          {{- if .replacement }}
          replacement: {{ .replacement | quote }}
          {{- end }}
          {{- if .targetLabel }}
          targetLabel: {{ .targetLabel | quote }}
          {{- end }}
          {{- if .sourceLabels }}
          sourceLabels:
            {{- range .sourceLabels }}
            - {{ . }}
            {{- end }}
          {{- end }}
        {{- end }}
      {{- end }}
      {{- with .Values.serviceMonitor.relabelings }}
      relabelings:
        {{- toYaml . | nindent 8 }}
      {{- end }}
{{- end }}
```

### Summary

The `metricRelabelings` configuration allows you to manipulate metrics before Prometheus stores them. You can use it to:
- Replace labels with new values.
- Keep only metrics that match certain criteria.
- Drop unwanted metrics.
- Rename labels based on a regex pattern.
- Exclude specific metrics from being counted (e.g., metrics from the `/metrics` endpoint).

By adding a `drop` rule for the `/metrics` endpoint, you ensure that polling metrics are not included in Prometheus, enhancing the security and accuracy of your monitoring setup.

---
- https://github.com/prometheus-community/helm-charts/blob/main/charts/kube-prometheus-stack/templates/prometheus/servicemonitor.yaml
- https://christianhuth.de/making-your-helm-chart-observable-for-prometheus/

---
