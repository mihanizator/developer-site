# Exofile

## Version

Exogress is growing, and eventually new and non backward-compatible functionality will be introduced.
Exofile version following [semantic versioning rules](https://semver.org/). However PATCH version relates
to "bug fixes", which doesn't really needed for configuration file format. It's expected to be "0" in most of cases.

Exofile documentaiton is sectioned based on MAJOR version, while the MINOR version, where the functionality is introduces
is placed on the field documentation.

### 1.x.x

#### Default configuration

```yaml
---
version: 1.0.0
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

#### name

Config name represents the peace of configuration, which is dynamically loaded into one or mote Mount Points. There may be multiple config names related to a single Mount Point,
as well as a single config which works with multiple Mount Points.

Name uniquely identifies the particular Exofile. Exogress servers will validate that multiple instances with the same `name` and `revision` announces exactly the same config.

#### revision

Revison is used to define how defferent instances with the same config `name` are performing upgrades. Due to the fact, that the same `name` and `revision` should represent the
same config, one needs to bump the revision number to as the system to perform rolling-upgrade. Otherwise new config will never be applied, it at least one instance with the same
`name` is still online.

#### mount-points

A list of mount points where this config will load it's handlers.
A hash-map, where kets are mount point names, and the values is the mount point body. Mount point body contains only one key: [`handlers`](/exofile?id=handlers).

#### handlers

The hash-map, where keys are handler identifiers (should be unique across mount point), and values are the handler body

#### handler

Handler represents the particular "step", which Exogress edge server should perform. There are different kinds of handlers. Field `type` defines which handler should be used. There are few common fields:

- `base-path` and `replace-base-path`. See [paths](/exofile?id=paths)
- `priority`. Gateway servers will apply handlers accoring to the priority. It start from the smallest priorty, moving to the larger values. Priority lives in mount point, and the end ordering may include
 handlers from different configs.
- `filters` see [Filters secions](/exofile?id=filters)

##### Handler `static-dir`

Serve the static directory from the computer where exogress client is running.

##### Handler `proxy`

Reverse proxy / Load Balancer handler. Forward requests to one of the defined [upstreams](/exofile?id=upstream)


#### filters

#### paths

#### upstreams
