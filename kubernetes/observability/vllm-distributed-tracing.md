## vLLM Distributed Tracing

This [Containerfile](./vllm-Containerfile)
shows the necessary packages to generate vLLM traces. In the future, these packages may be added to the default vLLM image available from
Red Hat OpenShift AI.

Here is how you would build vLLM with the tracing packages:

```bash
podman build --platform x86_64 -t quay.io/<your-quay-username>/vllm:otlp-tracing -f vllm-Containerfile .
podman push quay.io/[your-quay-username]/vllm:otlp-tracing
```

Then, add the following updates to the vLLM deployment.yaml.


```yaml
---
  template:
    metadata:
      labels:
        app: granite-8b
    spec:
      containers:
      - args:
        - --model
        - ibm-granite/granite-3.2-8b-instruct
        ---
        # tracing-specific flags and options
        - --otlp-traces-endpoint
        - http://otel-collector.observability-hub.svc.cluster.local::4317
        - --collect-detailed-traces
        - "all"
        image: 'quay.io/<your-quay-username>/vllm:otlp-tracing'
        env:
        - name: OTEL_SERVICE_NAME
          value: "vllm-granite8b"
        - name: OTEL_EXPORTER_OTLP_TRACES_INSECURE
          value: "true"
---
```

With the updated vLLM image and the updated deployment, distributed trace data will be generated and exported
to the central observability-hub as outlined in the [README.md](./README.md) with a `TempoStack` as a tracing backend.

There is a performance impact with enabling tracing, so it's recommended to update the deployment to enable tracing only when debugging to
avoid the performance impact. A complete list of vLLM engine arguments can be found [here](https://docs.vllm.ai/en/latest/serving/engine_args.html).
