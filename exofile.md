# Exofile

## Default configuration

```yaml
---
version: 0.0.1
revision: 1
name: my-config-name
mount-points:
  my-mount-point:
    handlers:
      my-handler:
        type: proxy
        upstream: my-upstream
        base-path: []
        replace-base-path: []
        priority: 10
upstreams:
  my-upstream:
    port: 3000
```
