---
layout: page
title: A8 Rules DSL
permalink: /docs/control-plane-controller-rules-dsl.html
redirect_from: /docs/control-plane/controller/rules-dsl/
category: Control Plane
subcategory: Route Controller
order: 4
---

The Amalgam8 controller [API for managing rules](/docs/control-plane-controller-rules-api.html) uses a Rules DSL based on JSON.
For example, a simple rule to send 100% of incoming traffic for the "reviews" microservice
to version "v1" can be described using the Rules DSL as follows.

```json
{
  "destination": "reviews",
  "route": {
    "backends": [
      {
        "tags": [ "v1" ]
      }
    ]
  }
}
```

This is a simple example of a "routing rule".

There are two types of rules in Amalgam8, routing rules, which control request routing,
and action rules, which perform actions such as fault injection in the request path.
Regardless of the type, every rule has a set of base properties.
The rest of the rule's properties depends on the type.

A single rule describing both a route and actions for a microservice is not allowed.
In other words a single rule cannot include both routing and action specifications (that is, `route` and `action` fields, see below),
but must rather be represented using two seperate rules in the DSL.

* [Base Properties](#base-rules)
    * [destination](#destination)
    * [priority](#priority)
    * [id](#id)
    * [match](#match)
        * [source](#match-source)
        * [headers](#match-headers)
        * [all](#match-allanynone)
        * [any](#match-allanynone)
        * [none](#match-allanynone)
* [Routing Rules](#routing-rules)
    * [route](#route)
        * [route.backends](#route-backends)
            * [route.backends.tags](#route-backends-tags)
            * [route.backends.weight](#route-backends-weight)
            * [route.backends.name](#route-backends-name)
            * [route.backends.resilience](#route-backends-resilience)
            * [route.backends.lbtype](#route-backends-lbtype)
        * [route.httpreqtimeout](#route-httpreqtimeout)
        * [route.httpreqretries](#route-httpreqretries)
        * [route.uri](#route-uri)
* [Action Rules](#action-rules)
    * [actions](#actions)
        * [actions.action](#actions-action)
        * [Delay Action](#delay-action)
            * [actions.duration](#actions-delay-duration)
            * [actions.probability](#actions-delay-probability)
            * [actions.tags](#actions-delay-tags)
        * [Abort Action](#abort-action)
            * [actions.return_code](#actions-abort-return_code)
            * [actions.probability](#actions-abort-probability)
            * [actions.tags](#actions-abort-tags)
        * [Trace Action](#trace-action)
            * [actions.tags](#actions-trace-tags)
            * [actions.log_key](#actions-trace-log_keyvalue)
            * [actions.log_value](#actions-trace-log_keyvalue)

## Base Rule Properties <a id="base-rules"></a>

#### Property: destination <a id="destination"></a>

Every rule corresponds to some destination microservice identified by a `destination` field in the A8 Rules DSL.
For example, all rules that apply to calls to the "reviews" microservice will include the following field.

```json
{
  "destination": "reviews"
}
```

#### Property: priority <a id="priority"></a>

The order of evaluation of rules corresponding to a given destination, when there is more than one, can be specified 
by setting the `priority` field of the rule.

```json
{
  "destination": "reviews",
  "priority": 1
}
```

The `priority` field is an optional integer value, 0 by default.
Rules with higher priority values are executed earlier.
If there is more than one rule with the same priority value, the order of execution is undefined.

#### Property: id <a id="id"></a>

When a rule is added to the system, the controller generates a unique string `id` field for the rule that can later be used to
refer to the rule if you want to update or delete it. 

```json
{
  "destination": "reviews",
  "priority": 1,
  "id": "3ee8fcaf-929e-40bc-8eb3-a7f4e4ffdf96"
}
```

#### Property: match <a id="match"></a>

Rules can optionally be qualified to only apply to requests that match some specific criteria such
as a specific request source and/or headers. An optional `match` field is used for this purpose.

The `match` field is an object with the following nested fields:

* `source`
* `header`
* `all`
* `any`
* `none`

#### Property: match.source <a id="match-source"></a>

The `source` field is used to qualify a rule to only apply to requests from a specific caller.
It contains a `name` and optional list of `tags` to identify the source of the request. For example,
the following `match` clause qualifies the rule to only apply to calls from version "v2" of the "reviews" microservice.

```json
{
  "destination": "ratings",
  "match": {
    "source": {
      "name": "reviews",
      "tags": [ "v2" ]
    }
  }
}
```

#### Property: match.headers <a id="match-headers"></a>

The `headers` field is a set of one or more property-value pairs where each property is an HTTP header name and the corresponding value is
a regular expression that must match the header value. For example, the following rule will only apply
to an incoming request if it includes a "Cookie" header that contains the substring "user=jason".

```json
{
  "destination": "ratings",
  "match": {
    "headers": {
      "Cookie": "^(.*?;)?(user=jason)(;.*)?$"
    }
  }
}
```

If more than one property-value pair is provided, then all of the corresponding headers must match for the rule to apply.

The `source` and `headers` fields can both be set in the `match` object in which case both criteria must pass.
For example, the following rule only applies if the source of the request is "reviews:v2" 
AND the "Cookie" header containing "user=jason" is present. 

```json
{
  "destination": "ratings",
  "match": {
    "source": {
      "name": "reviews",
      "tags": [ "v2" ]
    },
    "headers": {
      "Cookie": "^(.*?;)?(user=jason)(;.*)?$"
    }
  }
}
```

#### Property: match.all, match.any, match.none <a id="match-allanynone"></a>

The `all`, `any`, and `none` fields contain lists of additional match clauses (that is, `source` and `headers` fields) where
all (AND operation for the entries), any (OR operation for the entries), or none (negated OR operation for the entries) repectively
must match for the rule to apply. When used in conjuction with top level `source` and/or `headers` fields, the match criteria
within an `all`, `any`, or `none` list is implicitly ANDed with the top level match criteria. For example, the following rule
will apply if the source of the request is "reviews:v2"
but only if there is no header named "Foo" with the value "bar" or "baz" set in the request.

```json
{
  "destination": "ratings",
  "match": {
    "source": {
      "name": "reviews",
      "tags": [ "v2" ]
    },
    "none": [
      {
        "headers": { "Foo": "bar" }
      },
      {
        "headers": { "Foo": "baz" }
      }
    ]
  }
}
```

## Routing Rule Properties <a id="routing-rules"></a>

#### Property: route <a id="route"></a>

A routing rule is one that contains a `route` field in the A8 Rules DSL.

The `route` field is an object, although currently with only one nested field, `backends`, a list of weighted backends for the route.
*(Note: More fields will be added to the route object in future versions of the Rules DSL.)*


#### Property route.httpreqtimeout <a id="route-httpreqtimeout"></a>

Timeout for a HTTP request. Includes retries as well. Unit is in floating point seconds. Default 15.0s

#### Property route.httpreqretries <a id="route-httpreqretries"></a>

Number of retries for a given request. The interval between retries will be determined automatically (25ms+).
Actual number of retries attempted depends on the http_req_timeout.

#### Property route.uri <a id="route-uri"></a>

* `path`
* `prefix`
* `prefix_rewrite`

#### Property: route.backends <a id="route-backends"></a>

Each backend in the `backends` list is an object with the following fields.

* `tags`
* `weight`
* `name`
* `resilience`
* `lb_type`

Of these fields, `tags` is the only required one, the others are optional.

#### Property: route.backends.tags <a id="route-backends-tags"></a>

The `tags` field is a list of instance tags to identify the target instances (e.g., version) to route requests to.
If there are multiple registered instances with the specified tag(s), they will be routed to in a round-robin fashion.

#### Property: route.backends.weight <a id="route-backends-weight"></a>

The `weight` field is an optional value between 0 and 1 that represents the percentage of requests to route
to instances associated with the corresponding backend. If not set, the backend will, along with
other backends without a `weight` field, handle the percentage of requests that are not handled by weighted backends.
In other words, the remaining traffic after totalling the percentages of all the weighted backends, 
will be distributed equally across any unweighted backends.

The total percentage of requests covered by the set of backends in the list, using explicit or implicit weighting, must equal 100.
If less or greater than 100, the behavior of the route is undefined.

For example, the following rule will route 25% of traffic for the "reviews" service to instances with
the "v2" tag and the remaining traffic (i.e., 75%) to "v1".

```json
{
  "destination": "reviews",
  "route": {
    "backends": [
      {
        "tags": [ "v2" ],
        "weight": 0.25
      },
      {
        "tags": [ "v1" ]
      }
    ]
  }
}
```

#### Property: route.backends.name <a id="route-backends-name"></a>

The `name` field is optional and specifies the service name of the target instances. If not specified, it defaults
to the value of the rule's `destination` field.

#### Property: route.backends.resilience <a id="route-backends-resilience"></a>

* `max_connections`: Maximum number of connections to a backend.  Defaults to 1024.
* `max_pending_requests`: Maximum number of pending requests to a backend.  Defaults to 1024.
* `max_requests`: Maximum number of requests to a backend.  Defaults to 1024
* `sleep_window`: Minimum time the circuit will be closed.  Defaults to 30s
* `consecutive_errors`: Number of 5XX errors before circuit is opened.  Defaults to 5
* `detection_interval`: Interval for checking state of circuit.  Defaults to 10s.
* `max_requests_per_connection`: Maximum number of requests per connection to an backend.

#### Property: route.backends.lbtype <a id="route-backends-lbtype"></a>

Supported proxy load balancing algorithms are:

* `round_robin`
* `least_request`
* `random`

Default is `round_robin`.

### Routing Rule Execution

Whenever the routing story for a particular microservice is purely weight base,
it can be specified in a single rule, as in the previous example.
When, on the other hand, other crieria (e.g., requests from a specific user) are being used to route traffic,
more than one rule will be needed to specify the routing.
This is where the rule `priorty` field must be set to make sure that the rules are executed in the right order.

A common pattern for generalized route specification is to provide one or more higher priority rules
that use the `match` field (see [Base Properties](#base-rules)) to provide specific rules,
and then provide a single weight-based rule with no match criteria at the lowest priority to provide the
weighted distribution of traffic for all other cases.

The following example specifies that all requests for the "reviews" service
that includes a header named "Foo" with the value "bar" will be sent to the "v2" instances.
All remaining requests will be sent to "v1".

```json
[
  {
    "destination": "reviews",
    "priority": 2,
    "match": {
      "headers": {
        "Foo": "bar"
      }
    },
    "route": {
      "backends": [
        {
          "tags": [ "v2" ]
        }
      ]
    }
  },
  {
    "destination": "reviews",
    "priority": 1,
    "route": {
      "backends": [
        {
          "tags": [ "v1" ]
        }
      ]
    }
  }
]
```

Notice that the header-based rule has the higher priority (2 vs. 1). If it was lower, these rules wouldn't work as expected since the
weight-based rule, with no specific match criteria, would be executed first which would then simply route all traffic
to "v1", even requests that include the matching "Foo" header. Once a rule is found that applies to the incoming
request, it will be executed and the rule-execution process will terminate. That's why it's very important to
carefully consider the priorities of each rule when there is more than one.

## Action Rule Properties <a id="action-rules"></a>

#### Property: actions <a id="actions"></a>

An action rule is one that contains an `actions` field in the A8 Rules DSL.

The `actions` field is a list of objects that specify one or more actions to execute in the rule's corresponding request path.

#### Property: actions.action <a id="actions-action"></a>

The kind of action to execute is indicated by the value of the `action` field of each object in the list.
Other fields of an action depends on the value of the `action` field, which is one of "delay", "abort", or "trace".

### Delay Action <a id="delay-action"></a>

A delay action is one that has the `action` field set to the value "delay". 
A delay action is used to delay a request by a specified amount of time.

#### Property: actions.duration <a id="actions-delay-duration"></a>

The `duration` field is used to indicate the amount of delay, in seconds.

#### Property: actions.probability <a id="actions-delay-probability"></a>

An optional `probability` field, a value between 0 and 1, can be used to only delay a certain percentage of requests.
All request are delayed by default.

#### Property: actions.tags <a id="actions-delay-tags"></a>

Another optional field, `tags`, is a list of destination tags that can be used to only delay
requests that are routed to backends with the specified tags.

The following example will introduce a 5 second delay in 10% of the requests to the "v1" version of the "reviews" microserivice.

```json
{
  "destination": "reviews",
  "actions": [
    {
      "action": "delay",
      "probability": 0.1,
      "tags": [ "v1" ],
      "duration": 5
    }
  ]
}
```

### Abort Action <a id="abort-action"></a>

An abort action is one that has the `action` field set to the value "abort". 
An abort action is used to prematurely abort a request, usually to simulate a failure.

#### Property: actions.return_code <a id="actions-abort-return_code"></a>

The `return_code` field is used to indicate a value to return from the request, instead of calling the backend.
Its value is an integer value between -5 and 599, usually an http 2xx, 3xx, 4xx, or 5xx status code.

#### Property: actions.probability <a id="actions-abort-probability"></a>

An optional `probability` field, a value between 0 and 1, can be used to only abort a certain percentage of requests.
All request are aborted by default.

#### Property: actions.tags <a id="actions-abort-tags"></a>

Another optional field, `tags`, is a list of destination tags that can be used to only abort
requests that are routed to backends with the specified tags.

The following example will return an http 400 error code for all requests from the "reviews" service "v2" to the "ratings" service "v1".

```json
{
  "destination": "ratings",
  "match": {
    "source": {
      "name": "reviews",
      "tags": [ "v2" ]
    }
  },
  "actions" : [
    {
      "action": "abort",
      "tags": [ "v1" ],
      "return_code": 400
    }
  ]
}
```

### Trace Action <a id="trace-action"></a>

A trace action is one that has the `action` field set to the value "trace". 
A trace action is used to log a message in ElasticSearch indicating the request source and destination.

#### Property: actions.tags <a id="actions-trace-tags"></a>

An optional field, `tags`, is a list of destination tags that can be used to only log
requests that are routed to backends with the specified tags.

#### Property: actions.log_key, actions.log_value <a id="actions-trace-log_keyvalue"></a>

The `log_key` and `log_value` fields are used to set a key and value for the entry in ElasticSearch.

```json
{
  "destination": "ratings",
  "match": {
    "source": {
      "name": "reviews",
      "tags": [ "v2" ]
    }
  },
  "actions" : [
    {
      "action": "trace",
      "tags": [ "v1" ],
      "log_key": "trace_id",
      "log_value": "my_test_123"
    }
  ]
}
```

### Action Rule Execution

Similar to routing rules, the action rules for a particular `destination` are executed in
the order dictated by their rule `priority` fields. However, the `priority` values of action
rules and routing rules are independent and can overlap because they are evaluated separately.

The first step in the rule execution process evaluates the routing rules for the `destination`,
if any are defined, to determine the tags (i.e., specific version) of the destination service
that the current request will be routed to. Next, the set of action rules, if any, are evaluated
according to their priorities, to find the action rule to apply.

One subtlety of the algorithm to keep in mind is that actions that are defined for specific
tagged destinations will only be applied if the corresponding tagged instances are explicity
routed to. For example, consider the following rule, as the one and only rule defined for
the "reviews" microservice.

```json
{
  "destination": "reviews",
  "actions": [
    {
      "action": "delay",
      "probability": 0.1,
      "tags": [ "v1" ],
      "duration": 5
    }
  ]
}
```

Since there is no specific routing rule defined for the "reviews" microservice, default
round-robin routing behavior will apply, which will persumably call "v1" instances on occasion,
maybe even alway if "v1" is the only running version. Nevertheless, the above action will never
be invoked since the default routing is done at a lower level. The rule execution engine will be
unaware of the final destination and therefore unable to match the action rule to the request. 

You can fix the above example in one of two ways. You can either remove the `"tags": [ "v1" ],`
from the rule, if "v1" is the only instance anyway, or, better yet, define proper routing rules
for the service. For example, you can add a simple routing rule for "reviews:v1".

```json
{
  "destination": "reviews",
  "route": {
    "backends": [
      {
        "tags": [ "v1" ]
      }
    ]
  }
}
```

