---
layout: page
title: A8 Rules DSL
permalink: /docs/controller/rules-dsl/
category: Controller
order: 5
---

The Amalgam8 controller [API for managing rules](/docs/controller/rules-api) uses a Rules DSL based on JSON.
For example, a simple rule to send 100% of incoming traffic for the "reviews" microservice
to version "v1" can be described using the Rules DSL as follows.

```
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
* [Routing Rules](#routing-rules)
* [Action Rules](#action-rules)

## Base Rule Properties <a id="base-rules"></a>

Every rule corresponds to some destination microservice identified by a `destination` field in the A8 Rules DSL.
For example, all rules that apply to calls to the "reviews" microservice will include the following field.

```
{
  "destination": "reviews"
}
```

The order of evaluation of rules corresponding to a given destination, when there is more than one, can be specified 
by setting the `priority` field of the rule.

```
{
  "destination": "reviews",
  "priority": 1
}
```

The `priority` field is an optional integer value, 0 by default.
Rules with higher priority values are executed earlier.
If there is more than one rule with the same priority value, the order of execution is undefined.

When a rule is added to the system, the controller generates a unique string `id` field for the rule that can later be used to
refer to the rule if you want to update or delete it. 

```
{
  "destination": "reviews",
  "priority": 1,
  "id": "3ee8fcaf-929e-40bc-8eb3-a7f4e4ffdf96"
}
```

Rules can optionally be qualified to only apply to requests that match some specific criteria such
as a specific request source and/or headers. An optional `match` field is used for this purpose.

The `match` field is an object with the following nested fields:

* `source`
* `header`
* `all`
* `any`
* `none`

The `source` field is used to qualify a rule to only apply to requests from a specific caller.
It contains a `name` and optional list of `tags` to identify the source of the request. For example,
the following `match` clause qualifies the rule to only apply to calls from version "v2" of the "reviews" microservice.

```
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

The `headers` field is a set of one or more property-value pairs where each property is an HTTP header name and the corresponding value is
a regular expression that must match the header value. For example, the following rule will only apply
to an incoming request if it includes a "Cookie" header that contains the substring "user=jason".

```
{
  "destination": "ratings",
  "match": {
    "headers": {
      "Cookie": ".*?user=jason"
    }
  }
}
```

If more than one property-value pair is provided, then all of the corresponding headers must match for the rule to apply.

The `source` and `headers` fields can both be set in the `match` object in which case both criteria must pass.
For example, the following rule only applies if the source of the request is "reviews:v2" 
AND the "Cookie" header containing "user=jason" is present. 

```
{
  "destination": "ratings",
  "match": {
    "source": {
      "name": "reviews",
      "tags": [ "v2" ]
    },
    "headers": {
      "Cookie": ".*?user=jason"
    }
  }
}
```

The `all`, `any`, and `none` fields contain lists of additional match clauses (that is, `source` and `headers` fields) where
all (AND operation for the entries), any (OR operation for the entries), or none (negated OR operation for the entries) repectively
must match for the rule to apply. When used in conjuction with top level `source` and/or `headers` fields, the match criteria
within an `all`, `any`, or `none` list is implicitly ANDed with the top level match criteria. For example, the following rule
will apply if the source of the request is "reviews:v2"
but only if there is no header named "Foo" with the value "bar" or "baz" set in the request.

```
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

A routing rule is one that contains a `route` field in the A8 Rules DSL.
 
The `route` field is an object, although currently with only one nested field, `backends`, a list of weighted backends for the route.
*(Note: More fields will be added to the route object in future versions of the Rules DSL.)*

Each backend in the `backends` list is an object with the following fields.

* `tags`
* `weight`
* `name`
* `timeout`

Of these fields, `tags` is the only required one, the others are optional.

The `tags` field is a list of instance tags to identifying the target instances (e.g., version) to route requests to.
If there are multiple registered instances with the specified tag(s), they will be routed to in a round-robin fashion.

The `weight` field is an optional value between 0 and 1 that represents the percentage of requests to route
to instances associated with the corresponding backend. The default value is 1, which corresponds to all requests,
or more precisely "all remaining" requests after backends ahead of it in the list have been evaluated.

For example, the following rule will route 25% of traffic for the "reviews" service to instances with
the "v2" tag and the remaining traffic (i.e., 75%) to "v1".

```
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

The `name` field is optional and specifies the service name of the target instances. If not specified, it defaults
to the value of the rule's `destination` field.

The last optional field of a backend object is `timeout` which is the time, in seconds, to wait for a backend to respond
before timing out the request.

### Rule execution

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

```
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
request, it wil be executed and the rule-execution process will terminate. That's why it's very important to
carefully consider the priorities of each rule when there is more than one.

## Action Rule Properties <a id="action-rules"></a>

A action rule is one that contains an `actions` field in the A8 Rules DSL.

The `actions` field is a list of objects that specify one or more actions to execute in the rule's corresponding request path.
The kind of action to execute is indicated by the value of the `action` field of each object in the list.
Other fields of an action depends on the value of the `action` field, which is one of "delay", "abort", or "trace".

### delay action

A "delay" action is used to delay a request by a specified amount of time.

The `duration` field is used to indicate the ammount of delay, in seconds.

An optional `probability` field, a value between 0 and 1, can be used to only delay a certain percentage of requests.
All request are delayed by default.

Another optional field, `tags`, is a list of destination tags that can be used to only delay
requests that are routed to backends with the specified tags.

The following example will introduce a 5 second delay in 10% of the requests to the "v1" version of the "reviews" microserivice.

```
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

### abort action

An "abort" action is used to prematurely abort a request, usually to simulate a failure.

The `return_code` field is used to indicate a value to return from the request, instead of calling the backend.
Its value is an integer value between -5 and 599, usually an http 2xx, 3xx, 4xx, or 5xx status code.

An optional `probability` field, a value between 0 and 1, can be used to only abort a certain percentage of requests.
All request are aborted by default.

Another optional field, `tags`, is a list of destination tags that can be used to only abort
requests that are routed to backends with the specified tags.

The following example will return an http 400 error code for all requests from the "reviews" service "v2" to the "ratings" service "v1".

```
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

### trace action

A "trace" action is used to log a message in ElasticSearch indicating the request source and destination.

An optional field, `tags`, is a list of destination tags that can be used to only log
requests that are routed to backends with the specified tags.

The `log_key` and `log_value` fields are used to set a key and value for the entry in ElasticSearch.

```
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
