# Version 1.x.0

## Project config vs Client config

There are two types of configurations. The first one is client-level config, which typically lives in your git
repositories in the `Exofile.yml` file. Another config type is project-level config, which lives in the cloud.

There may be multiple client-level configs, but only one project-level config.

Some handlers may be defined in client-level config only, in project-level config only or in both config types. Each
handler documentation entry contains the information on which type of config is supported.

## Core entities

### Mount Point

Mount point is an identifier, which is used to connect the configuration with domain
name. [Learn more](/config-1_x_0?id=mount-points)

### Handlers

Handlers perform the actual work. Handlers are ordered by the `priority` field under the mount point, and are invoked
step-by-step.

Each handler contains the following fields:

- `base-path` and `replace-base-path`. Handler may be places under the different root. For instance, imaging you want to
  serve uploads from the S3 buckets under the path prefix `/uploads/`. Files in S3 bucket are stored in the root dir,
  not prefixed with `/uploads`. This is the handler configuration which make it work:

```yaml
uploads:
  kind: s3-bucket
  priority: 10
  bucket:
    name: my-test-bucket
    region: us-west-2
  base-path: [ "uploads" ]
  replace-base-path: [ ]
```

- `priority`. Gateway servers will apply handlers according to the priority. It starts from the smallest priority,
  moving to the larger values. Priority lives in a mount point, and the end ordering may include handlers from different
  configs.
- `rules` see [Rules section](/config-1_x_0?id=rules)

#### Rules

Each handler contains the `rule` block, which defines how handler should react on the request depending on its path,
query parameters, headers, etc. As a result of rule processing, the handler may be invoked, skipped, perform
static-response or throw the exception.

Example:

```yaml
rules:
  - filter:
      path: [ "none" ]
    action: next-handler
  - filter:
      path: [ "*" ]
    action: invoke
```

Filter is used to check if the rule should be applied to the request. Filter contains the following fields:

```yaml
filter:
  path: [ "segment1", "segment2" ]
  trailing-slash: require|allow|deny
  methods:
    - GET
    - POST
    - PUT
    - ...
  query-params:
    query-params:
      q1: "?"
      ..
```

### Path Matcher and trailing slash policy

Exogress treats patch segment as a first class citizen, so that you defined rules based on arrays of segments, not the
whole path matchers.

There are different kinds of segment(s) matchers available:

- Strict match. The simple string will be strictly matched
- Any segment. `?` symbol is used to match against any segment on the position
- Any number of segments. `*` is used when any number of segments is allowed. This symbol may be used only once in the
  path matcher.
- Choice. If the segment is an array, any of the provided values are allowed on the position
- Regular Expression. Segment starting and ending with `/` symbol is treated as a regular expression.

Array notation for path segments doesn't cover if path ends with `/` or not. `trailing-slash` is used for that. Possible
values are: `require`, `allow`, and `deny`

### Query Matcher

Query matchers are used to match the GET query parameters. They are mostly similar to path matchers, but have some
differences.

- Strict match. The simple string will be strictly matched
- Any segment. `?` symbol is used to match parameter value that doesn't contain `/` and exists
- Any number of segments. `*` is just like `?` but allows `/` symbol
- Choice. Any of the provided values are allowed on the position
- Regular Expression. Segment starting and ending with `/` symbol is treated as a regular expression.
- `~` (yaml notation for `nil`). Optionally any value. Used only to capture the value, does not affect the filtering.

### Exceptions

Exogress uses the concept of exceptions, just like most of the modern programming languages. Configuration may
explicitly instruct Exogress servers to throw the exception, or it may occur due to the incorrect configuration file,
processing error, etc.

Exception name has the following form

```
segment1:segment2:segment3
```

Exception name represents the inheritance, when we read it from the left to the right.

For example:

```
proxy-error:upstream-unreachable:connection-rejected
```

means that this exception belongs to the exception class `proxy-error` and it's
subclass `proxy-error:upstream-unreachable`. This is useful in the rescue blocks, so that we don't need to point all
possible exceptions which should lead, for example, to "Gateway Error" page, but just use the parent segment:

```yaml
rescue:
  - catch: "exception:proxy-error"
    action: respond
    status-code: 502
    static-response: bad-gateway
```

### Rescue

Rescue blocks are used to catch exceptions as well as to handle HTTP status codes, generated by the particular handler.

Rescue blocks are checked one-by-one according to the visibility rules.

Each item defined under the rescue block should contain the `catch` field. Exogress tries to match each catch field
against the thrown exception. There are two forms of the catch field:

```
exception:exception-segments
```

If the thrown exception equals to the one defined in catch field, or catch field contains the subclass of the thrown
exception, then the whole block is executed.

or

```
status-code:5xx
```

This rescue blocks are applied whe the response is successfully generated, and we need to customize the behavior
depending on the resulting status code.

Status-code matcher may be in the one of the following forms:

- `"200"`
- `"200,201,202"`
- `"4xx"`
- `"505-510"`

Each rescue block should contain the `action` field. Allowed values are:

1. `respond` - respond with static-response
   ```yaml
    catch: "exception:proxy-error"
    action: respond
    static-response: "@server-unreachable"
   ```

2. `throw` - throw the exception.
   ```yaml
    catch: "status-code:429"
    action: throw
    exception: "api-error:rate-limited"
    date:
      backed: "api-server"
   ```

3. `next-handler` - move on to the next handler

### Static Response

It is often required that the server just responds with the predefined static-response.

Static responses may be defined in parameters, client or project config or just defined in the place where they are
required.

Example:

```yaml
---
version: 1.0.0
revision: 1
name: config
mount-points:
  default:
    handlers:
      dir:
        kind: pass-through
        priority: 10
        rules:
          - filter:
              path: [ "ref" ]
            action: respond
            static-response: it-works
          - filter:
              path: [ "embed" ]
            action: respond
            static-response:
              kind: redirect
              destination: "https://google.com/"
              redirect-type: moved-permanently
          - filter:
              path: [ "param" ]
            action: respond
            static-response: "@static-resp-param"
static-responses:
  it-works:
    kind: raw
    status-code: 200
    body:
      - content-type: text/html
        content: "<html><body>It works!</body></html>"
        engine: handlebars
```

#### Redirections

Redirection is a kind of static-response. There are multiple status-codes fo redirection. User may choose which one to
use:

- `moved-permanently`
- `permanent-redirect`
- `found`
- `see-other`
- `temporary-redirect`
- `multiple-choices`
- `not-modified`

Redirection destination may be defined in different forms:

- String (e.g. `https://google.com`). The exact destination
- Path segments array. e.g. `["seg1"]`
- Path segments array starting with the protocol and host name. e.g. `["https://google.com", "seg1"]`

Path segments may contain substitution to customize redirects based on the request.

```yaml
rules:
  - filter:
      path: [ "?" ]
    action: respond
    static-response:
      kind: redirect
      destination: [ "a-{{ 0 }}-z" ]
      redirect-type: moved-permanently
```

And with query parameters:

```yaml
rules:
  - filter:
      path: [ ]
      query-params:
        search: "*"
    action: respond
    static-response:
      kind: redirect
      destination: [ "https://google.com", "search" ]
      query-params:
        strategy: remove
        set:
          q: "{{ search }}"
      redirect-type: moved-permanently
```

`query-params` contains the `strategy` field which may be one of `remove` and `keep`. Strategy defies how to deal with
query parameters from initial request.

- `keep` save all parameters but removes parameters defined in `remove` array
- `remove` deletes all parameters but keep parameters defined in `keep` array

In addition to that, `set` object defines query parameters which will be set. Values support substitutions.

### Scope of static-responses and catch blocks

`static-responses` and `rescue` blocks may be defined in both client and project level configs, on different levels:

- top-level
- mount-point
- handler

The visibility of the particular item depends on where it is defined. Generally, the visibility order is as follows:

1. Top-level Project config
2. Top-level Client config with the same config name
3. Mount Point Project config with the same mount point name
4. Mount Point client config with the same mount point name and config name
5. Handler Project config with the same handler and mount point name
6. Handler client config with the same mount point name and config name, and a handler name

In other worlds, static responses defined in the Top-level Project config is visible to any configuration block in the
project configuration, while static responses defined on the mount-point level in client config will be visible on this
mount point defined in this client config and is invisible for other client configs or project configs.

The lookup priority is in backward direction. Exogress will start lookup in the most precise point and end-up on the
top-level project config.

This system allows having static responses with the same name with different content, as well as rescue blocks catching
the same exceptions, defined on different levels, with different behavior on different parts of the system.

## Config top-level keys

### version

?> introduced in 1.0.0

?> Client & Project

`version` field defines the minimum version that exogress client should support. The Patch version is always expected to
be 0. Since this page describes the major version 1, it should always be 1. It's recommended to bump the version when
new minor version is released.

### name

?> introduced in 1.0.0

?> Client only

Config name represents the peace of configuration, which is dynamically loaded into one or more Mount Points. There may
be multiple config names related to a single Mount Point, as well as a single config that works with numerous Mount
Points.

Name uniquely identifies the particular `Exofile.yml` content. Exogress servers will validate that multiple Clients with
the same `name` and `revision` announce exactly the same config.

### revision

?> introduced in 1.0.0

?> Client only

Revision is used to define how different Clients with the same config `name` are performing upgrades. Because the
same `name` and `revision` should represent the same config, one needs to bump the revision number so that system may
perform a rolling-upgrade. Otherwise, the new config will never be applied if at least one Client with the same
`name` is still online.

### mount-points

?> introduced in 1.0.0

?> Client & Project

A list of mount points where this config will load its handlers. A hash-map, where keys are mount points, and the values
are the mount point body.

Mount Point is identified by name. Mount Point body contains mount-point level static responses and catch blocks along
with the array of handlers.

```yaml
---
version: 1.0.0
revision: 1
name: cfg
mount-points:
  default:
    handlers:
      dir:
        kind: static-dir
        dir: "./dir"
        priority: 5
        rules:
          - filter:
              path: [ "index.html" ]
            action: invoke
      s3:
        kind: s3-bucket
        priority: 10
        bucket:
          name: my-test-bucket
          region: us-west-2
```

When request is processed, Exogress builds the list of handlers ordered by priority for each mount point. It typically
includes multiple active client configs (Exofile.yml) and a project-level config.

### upstreams

?> introduced in 1.0.0

?> Client only

Upstream is the way to deliver the traffic to a running web servers.

Example upstream:

```yaml
upstreams:
  upstream:
    port: 3000
    health-checks:
      root:
        kind: liveness
        path: /health
        timeout: 1s
        period: 1s
        expected-status-code: 200
```

### Address

Upstream body contains two keys which define how to reach the upstream:

```yaml
port: 3000
host: 192.168.1.2
```

`port` is required, `host` may be avoided and is `127.0.0.1` by default. Exogress client typically runs on the same host
with the web server, so `127.0.0.1` is the reasonable default for the most of the cases.

### Health checks

There may be multiple health checks defined on each upstream. Exogress client performs upstreams health checks and
reports the result to the Exogress Cloud. Exogress gateways will not use the unhealthy upstream.

For now there is only one health check kind: `liveness`. More kinds will be added in the future.
`path` defines which path to query. `timeout` defines how long to wait for response. `period` defines how long to sleep
between requests. Health check is treated successful if response matched the status-code matcher defined
in `expected-status-code`. Requests headers may be defined in the `headers` object. `method` is responsible for HTTP
method to use.

## Handler

?> introduced in 1.0.0

?> Client & Project

[See also](/config-1_x_0?id=handlers)

### `proxy`

?> introduced in 1.0.0

?> Client

Proxy handler forwards all traffic to the upstream.

```yaml
kind: proxy
upstream: upstream-name
priority: 10
```

### `static-dir`

?> introduced in 1.0.0

?> Client

Forward the traffic to a running exogress client, which serves files from the local directory.

```yaml
kind: static-dir
dir: "./dir"
priority: 10
```

### `s3-bucket`

?> introduced in 1.0.0

?> Client & Project

Proxy requests to AWS S3 bucket.

```yaml
kind: s3-bucket
priority: 10
bucket:
  name: my-test-bucket
  region: us-west-2
credentials:
  secret_access_key: "..."
  access_key_id: "..."
```

Both `bucket` and `credentials` may be stored in parameters.

### `gcs-bucket`

?> introduced in 1.0.0

?> Client & Project

Example:

```yaml
kind: gcs-bucket
priority: 10
bucket:
  name: my-test-bucket
credentials: "@gcs-credentials"
```

Both `bucket` and `credentials` may be stored in parameters.

### `auth`

?> introduced in 1.0.0

?> Client & Project

```yaml
kind: auth
github:
  acl:
    - allow: github-user
google:
  acl: "@google-acl"
priority: 10
```

`acl` may be stored in parameters.

### `pass-through`

?> introduced in 1.0.0

?> Client & Project

```yaml
kind: pass-through
priority: 10
```

## Profiles

?> introduced in 1.0.0

?> Client

Sometimes it makes sense to vary configuration depending on some conditions. For instance, we may want to use some
handler only during the development, but skip it in production. Profiles may be used exactly for that.

Exogress client accepts the `profile` configuration option to define which profile to use.

Some configuration blocks on the client config accepts the `profiles` array, to define on which profiles it should be
active. In case if `profiles` is defined, the configuration block will be disabled by default, otherwise it will be
enabled and not affected by selected profile.

`profiles` is supported on the follow configuration blocks:

- Mount Point
- Handler
- Upstream
- Rule

## Static Response handlebars, facts

## Cache

Edge cache may be enabled and disabled on per-handler basis. Exogress respects HTTP headers and will cache responses
which contain the appropriate `Cache-Control` headers.

TODO: more about caching

Caching is disabled by default. Example config with enabled cache on the `proxy` handler.

```yaml
version: 1.0.0
revision: 1
name: proxy
mount-points:
  default:
    handlers:
      proxy:
        kind: proxy
        upstream: upstream
        priority: 40
        cache:
          enabled: true
upstreams:
  upstream:
    port: 11988
```

## Post-Processing

Exogress edge servers may perform the content optimization.

### WebP

WebP is an image format employing both lossy and lossless compression, along with animation and alpha transparency.
Developed by Google, it is designed to create smaller or better looking images compared to the JPEG, PNG, or GIF image
formats.

Exogress may perform the conversion of png and jpeg responses to WebP format. WebP is enabled by default. This is
controlled on the handler level.

```yaml
version: 1.0.0
revision: 1
name: proxy
mount-points:
  default:
    handlers:
      proxy:
        kind: proxy
        upstream: upstream
        priority: 10
        post-processing:
          image:
            webp:
              enabled: true
              jpeg: false
              png: false
upstreams:
  upstream:
    port: 11988
```

Note, that Exogress may skip optimizations on per-request basis, depending on many conditions. Typically, requests that
are not eligible for caching will not be converted to WebP.

## Modifications

Exogress supports both request and response modifications.

### Requests Modifications

Most of the handlers proxy the incoming request to some other destination. Often it is required to modify the request,
when it goes to its destination. This can be reached through [base paths](/config-1_x_0?id=handlers) and through
the `modify-request` rule config option.

```yaml
---
version: 1.0.0
revision: 1
name: modifications
mount-points:
  default:
    handlers:
      dir:
        kind: proxy
        upstream: upstream
        priority: 10
        rules:
          - filter:
              path: [ "?" ]
            action: invoke
            modify-request:
              path: [ "{{ 1 }}", "modified" ]
              trailing-slash: unset
              query-params:
                strategy: keep
                remove:
                  - q2
              headers:
                insert:
                  "x-inserted": "yes"
                  "x-sent-from-client3": "rewrite"
                append:
                  "x-sent-from-client2": "appended"
                remove:
                  - "x-sent-from-client"
upstreams:
  upstream:
    port: 11988
```

### Response Modifications

Sometimes it is required to modify the response, after it is generated. The most typical scenario is to enable the
caching through the `Cache-Control` header.

```yaml
---
version: 1.0.0
revision: 1
name: modifications
mount-points:
  default:
    handlers:
      dir:
        kind: proxy
        upstream: upstream
        priority: 10
        rules:
          - filter:
              path: ["*"]
            action: invoke
            on-response:
              - when:
                  status-code: "2xx"
                modifications:
                  headers:
                    insert:
                      "x-inserted": "yes"
                    append:
                      "x-appended": "yes"
                    remove:
                      - "x-remove"
upstreams:
  upstream:
    port: 11988
```


