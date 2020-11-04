
# Version 1.x.x

## Default configuration

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

## Fields

### version

?> introduced in 1.0.0

`version` field defines the minimium version that exogress client should support. Patch version is expected to alway be 0. Since this page describes the major version 1, it should alway be 1.
All features, introduced in a later minor version will still be available, even if it's lower in this field. It's recommended to bump the version if new features are required.

### name

?> introduced in 1.0.0

Config name represents the peace of configuration, which is dynamically loaded into one or mote Mount Points. There may be multiple config names related to a single Mount Point,
as well as a single config which works with multiple Mount Points.

Name uniquely identifies the particular Exofile. Exogress servers will validate that multiple instances with the same `name` and `revision` announces exactly the same config.

### revision

?> introduced in 1.0.0

Revison is used to define how defferent instances with the same config `name` are performing upgrades. Due to the fact, that the same `name` and `revision` should represent the
same config, one needs to bump the revision number to as the system to perform rolling-upgrade. Otherwise new config will never be applied, it at least one instance with the same
`name` is still online.

### mount-points

?> introduced in 1.0.0

A list of mount points where this config will load it's handlers.
A hash-map, where kets are mount point names, and the values is the mount point body. Mount point body contains only one key: [`handlers`](/exofile-1_x_x?id=handlers).

### static-responses

?> introduced in 1.0.0

### upstreams

?> introduced in 1.0.0

### handlers

?> introduced in 1.0.0

The hash-map, where keys are handler identifiers (should be unique across mount point), and values are the handler body

### handler

?> introduced in 1.0.0

Handler represents the particular "step", which Exogress edge server should perform. There are different kinds of handlers. Field `type` defines which handler should be used. There are few common fields:

- `base-path` and `replace-base-path`. See [URL Paths](/exofile-1_x_x?id=url-paths)
- `priority`. Gateway servers will apply handlers accoring to the priority. It start from the smallest priorty, moving to the larger values. Priority lives in mount point, and the end ordering may include
 handlers from different configs.
- `filters` see [Filters secions](/exofile-1_x_x?id=filters)

#### static-dir

?> introduced in 1.0.0

Serve the static directory from the computer where exogress client is running.

#### proxy

?> introduced in 1.0.0

Reverse proxy / Load Balancer handler. Forward requests to one of the defined [upstreams](/exofile-1_x_x?id=upstream)

### upstreams

?> introduced in 1.0.0

## Rules

?> introduced in 1.0.0

Rules may be defined on any handler. Their goal to provide additional rules on handler processing.

Rules example:

```yaml
mount-points:
  my-mount-point:
    handlers:
      backend:
        type: proxy
        upstream: server
        priority: 10
        rules:
          - filter:
              path: ["static", "*"]
            action:
              kind: next-handler
          - filter:
              path: ["*"]
            action:
              kind: invoke
```

`path` field is the [path with possible wildcards](/exofile-1_x_x?id=wildcards)

Rules are processed from top to bottom. When filter matches the requested one the action is performed.

There are few types of actions:

- `invoke`. Pass the request to the handler, where the rule is defined.
- `next-handler`. Skip this handler and move on to the next one.
- `none`. Just skip this rule and move on to the next one. Will later be used for request modifications.
- `throw`. Throw the [exception](/exofile-1_x_x?id=exceptions).
- `respond`. Respond with the [static response](/exofile-1_x_x?id=static-responses). `static-response` field defines the name of static response. This action will stop processing of both rules and and handlers,
and finish the whole chain with the response.


## URL Paths

?> introduced in 1.0.0

Exogress use array-based notation for representing paths. Instead of writing the path as a string (/segment1/segment2) it needs to written as an array: `["segment1", "segment2"]`.
This have a lot of benefits - better readability, good looking URL rewrites, and wildcard matching

### Wildcards

?> introduced in 1.0.0

If path may be written as a wildcard (e.g. filter), special wildcard syntax may be used.

- Single wildcard segment: `["segment1", "?", "segment2"]`
- Multiple wildcard segments: `["segment1", "*", "segment2"]`. Any number of segments may exist between the first and the last one. There may be only one `*` wildcard segment.
- Regular expression: `["/segment\d+/"]`

### Path Substitution

?> not available in the version 1.0.0

Paths may be rewritten with another paths following the rewriing templates. They work in cooperation with wildcard paths

- mathcher `["segment1", "*", "segment2"]` with rewrite: `["api", "$1"]`. This will lead to catching any number of segments between `segment1` and `segment2` and prepending `api` before them. `/segment1/a/b/c/segment2` will beome `/api/a/b/c`
- mathcher `["segment1", "?", "segment2"]` with rewrite: `["api", "$1"]`. This will lead to catching a single segments between `segment1` and `segment2` and prepending `api` before it. `/segment1/a/segment2` will beome `/api/a`

## Exceptions

?> introduced in 1.0.0

Exceptions are used to stop the processing chain. They may be caught by one of `catch` blocks.