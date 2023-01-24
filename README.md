## vector-config.yaml for my youtube video "Kubernetes Log Management: How to Export Logs to Azure Blob Storage"

```
resources:
  requests:
    memory: "256Mi"
    cpu: "300m"
  limits:
    memory: "500Mi"
    cpu: "400m"

service:
  ports:
    - name: http
      port: 9090

role: Agent

customConfig:
  data_dir: /var/lib/vector
  sources:
    kube_log:
      type: "kubernetes_logs"
      timezone: "local"
      auto_partial_merge: true
      extra_namespace_label_selector: "kubernetes.io/metadata.name in (test-ns)"
  transforms:
    my_remap_id:
      type: remap
      inputs:
      - kube_log
      source: |-
        .test = del(.)
        .container_name = del(.test.kubernetes.container_name)
        .message = del(.test.message)
        .timestamp = del(.test.timestamp)
        del(.test)

  sinks:
    azure_blob:
      type: azure_blob
      inputs:
        - my_remap_id
      connection_string: ""
      container_name: akslogs
      blob_prefix: |-
        {{print "%Y%m%d/{{ container_name }}/"}}
      blob_append_uuid: true
      compression: none
      encoding:
        codec: json
```

