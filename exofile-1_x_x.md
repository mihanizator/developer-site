
# Version 1.x.x

## Project config vs Instance config

There are two types of configurations. The first one is instance-level config, which typically lives in your git repositories in the Exofile file. Another config type is project-level config, which lives in the cloud (currently on the project page on the Exogress web application).

There may be multiple instance-level configs, but only one project-level config.

Some handlers may be defined in instance-level config only, others in project-level config only. Each handler documentation entry contains the information on which type of config is supported.


## Default instance configuration

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

?> Instance & Project

`version` field defines the minimum version that exogress client should support. The Patch version is always expected to be 0. Since this page describes the major version 1, it should always be 1.
All features, introduced in a later minor version will still be available, even if it's lower in this field. It's recommended to bump the version if new features are required.

### name

?> introduced in 1.0.0

?> Instance only

Config name represents the peace of configuration, which is dynamically loaded into one or more Mount Points. There may be multiple config names related to a single Mount Point,
as well as a single config that works with numerous Mount Points.

Name uniquely identifies the particular Exofile. Exogress servers will validate that multiple instances with the same `name` and `revision` announce the same config.

### revision

?> introduced in 1.0.0

?> Instance only

Revision is used to define how different instances with the same config `name` are performing upgrades. Because the same `name` and `revision` should represent the
same config, one needs to bump the revision number so that system may perform a rolling-upgrade. Otherwise, the new config will never be applied if at least one instance with the same
`name` is still online.

### mount-points

?> introduced in 1.0.0

?> Instance & Project

A list of mount points where this config will load its handlers.
A hash-map, where keys are mount points, and the values are the mount point body. Mount point body contains only one key: [`handlers`](/exofile-1_x_x?id=handlers).

### static-responses

?> introduced in 1.0.0

?> Instance & Project

### upstreams

?> introduced in 1.0.0

?> Instance only

### handlers

?> introduced in 1.0.0

?> Instance & Project

The hash-map, where keys are handler identifiers (should be unique across mount point), and values are the handler body.

### handler

?> introduced in 1.0.0

?> Instance & Project

Handler represents the particular step that Exogress edge server should perform. There are different kinds of handlers. Field `type` defines which handler should be used. There are a few common fields:

- `base-path` and `replace-base-path`. See [URL Paths](/exofile-1_x_x?id=url-paths)
- `priority`. Gateway servers will apply handlers according to the priority. It starts from the smallest priority, moving to the larger values. Priority lives in a mount point, and the end ordering may include handlers from different configs.
- `filters` see [Filters secions](/exofile-1_x_x?id=filters)

#### static-dir

?> introduced in 1.0.0

?> Instance only

Serve the static directory from the computer where the exogress client is running.

#### proxy

?> introduced in 1.0.0

?> Instance only

Reverse proxy / Load Balancer handler. Forward requests to one of the defined [upstreams](/exofile-1_x_x?id=upstream)

### upstreams

?> introduced in 1.0.0

?> Instance only

## Rules

?> introduced in 1.0.0

?> Instance & Project

Rules may be defined on any handler— their goal is to provide behavior modifications on handler processing.

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

Rules are processed from top to bottom. When the filter matches the requested one - the action is performed.

There are a few types of actions:

- `invoke`. Pass the request to the handler, where the rule is defined.
- `next-handler`. Skip this handler and move on to the next one.
- `none`. Just skip this rule and move on to the next one. Will later be used for request modifications.
- `throw`. Throw the [exception](/exofile-1_x_x?id=exceptions).
- `respond`. Respond with the [static response](/exofile-1_x_x?id=static-responses). `static-response` field defines the name of static response. This action will stop processing both rules and handlers
and finish the whole chain with the response.


## URL Paths

?> introduced in 1.0.0

Exogress use array-based notation for representing paths. Instead of writing the path as a string (/segment1/segment2) it needs to written as an array: `["segment1", "segment2"]`.
This has a lot of benefits - better readability, good looking URL rewrites, and wildcard matching.

### Wildcards

?> introduced in 1.0.0

If the path may be written as a wildcard (e.g., filter), special wildcard syntax may be used.

- Single wildcard segment: `["segment1", "?", "segment2"]`
- Multiple wildcard segments: `["segment1", "*", "segment2"]`. Any number of segments may exist between the first and the last one. There may be only one `*` wildcard segment.
- Regular expression: `["/segment\d+/"]`

### Path Substitution

?> not available in the version 1.0.0

Paths may be rewritten with other paths following the rewriting templates. They work in cooperation with wildcard paths.

- mathcher `["segment1", "*", "segment2"]` with rewrite: `["api", "$1"]`. This will lead to catching any number of segments between `segment1` and `segment2` and prepending `api` before them. `/segment1/a/b/c/segment2` will beome `/api/a/b/c`
- mathcher `["segment1", "?", "segment2"]` with rewrite: `["api", "$1"]`. This will lead to catching a single segments between `segment1` and `segment2` and prepending `api` before it. `/segment1/a/segment2` will beome `/api/a`

## Exceptions

?> introduced in 1.0.0

Exceptions are used to stop the processing chain. They may be caught by one of the `catch` blocks.
