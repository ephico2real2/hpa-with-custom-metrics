```
metrics:
  serviceMonitor:
    # -- Additional labels that can be used so ServiceMonitor will be discovered by Prometheus
    additionalLabels: {}
    # -- Specify honorLabels parameter to add the scrape endpoint
    honorLabels: false
    # -- Interval at which metrics should be scraped.
    interval: "30s"
    # -- The name of the label on the target service to use as the job name in Prometheus
    jobLabel: ""
    # -- MetricRelabelConfigs to apply to samples before ingestion
    metricRelabelings: {}
    # -- Namespace for the ServiceMonitor Resource (defaults to the Release Namespace)
    namespace: ""
    # -- The path used by Prometheus to scrape metrics
    path: "/metrics"
    # -- RelabelConfigs to apply to samples before scraping
    relabelings: {}
    # -- Timeout after which the scrape is ended
    scrapeTimeout: ""
    # -- Prometheus instance selector labels
    selector: {}
```

```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: {{ template "a-helm-chart.fullname" . }}
  namespace: {{ .Values.metrics.serviceMonitor.namespace | default .Release.Namespace | quote }}
  labels:
    {{- include "a-helm-chart.labels" . | nindent 4 }}
    {{- with .Values.metrics.serviceMonitor.additionalLabels }}
      {{- toYaml . | nindent 4 }}
    {{- end }}
spec:
  endpoints:
    - port: http-metrics
      {{- if .Values.metrics.serviceMonitor.honorLabels }}
      honorLabels: {{ .Values.metrics.serviceMonitor.honorLabels }}
      {{- end }}
      {{- if .Values.metrics.serviceMonitor.interval }}
      interval: {{ .Values.metrics.serviceMonitor.interval | quote }}
      {{- end }}
      {{- with .Values.metrics.serviceMonitor.metricRelabelings }}
      metricRelabelings:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      path: {{ .Values.metrics.serviceMonitor.path | quote }}
      {{- with .Values.metrics.serviceMonitor.relabelings }}
      relabelings:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- if .Values.metrics.serviceMonitor.scrapeTimeout }}
      scrapeTimeout: {{ .Values.metrics.serviceMonitor.scrapeTimeout | quote }}
      {{- end }}
  {{- if .Values.metrics.serviceMonitor.jobLabel }}
  jobLabel: {{ .Values.metrics.serviceMonitor.jobLabel | quote }}
  {{- end }}
  namespaceSelector:
    matchNames:
      - {{ .Release.Namespace | quote }}
  selector:
    matchLabels:
      {{- include "a-helm-chart.selectorLabels" . | nindent 6 }}
      {{- with .Values.metrics.serviceMonitor.selector }}
      {{- toYaml . | nindent 6 }}
      {{- end }}
```
