# ose3-rails-sphinx-transifex

## Transifex

This image includes the official transifex client 0.12.x

## Aditional parameters

Additional configuration parameters you may specify via environment variables for the container.

**SPHINX_INTERVAL_TIME**: Time between sphinx indexer runs in seconds. Defaults to `3600` (10 minutes).
**SPHINX_CONFIG_PATH**: Location of the sphinx configuration file. Defaults to `config/production.sphinx.conf`.
**SPHINX_SKIP_INITIAL_INDEX**: Set to 1 if you do not want to run `rake ts:index` before starting searchd.

## Kubernetes liveness check

To your Pod.containers[x], add

    livenessProbe:
      exec:
        command: 
          - bash
          - /usr/libexec/s2i/k8s-liveness-check.sh
      initialDelaySeconds: 30
      periodSeconds: 10
