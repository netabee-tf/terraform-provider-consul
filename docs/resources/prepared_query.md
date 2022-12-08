---
layout: "consul"
page_title: "Consul: consul_prepared_query"
sidebar_current: "docs-consul-resource-prepared-query"
description: |-
  Allows Terraform to manage a Consul prepared query
---

# consul_prepared_query

Allows Terraform to manage a Consul prepared query.

Managing prepared queries is done using Consul's REST API. This resource is
useful to provide a consistent and declarative way of managing prepared
queries in your Consul cluster using Terraform.

## Example Usage

```hcl
# Creates a prepared query myquery.query.consul that finds the nearest
# healthy myapp.service.consul instance that has the active tag and not
# the standby tag.
resource "consul_prepared_query" "myapp-query" {
  name         = "myquery"
  datacenter   = "us-central1"
  token        = "abcd"
  stored_token = "wxyz"
  only_passing = true
  near         = "_agent"

  service = "myapp"
  tags    = ["active", "!standby"]

  failover {
    nearest_n   = 3
    datacenters = ["us-west1", "us-east-2", "asia-east1"]
  }

  dns {
    ttl = "30s"
  }
}

# Creates a Prepared Query Template that matches *-near-self.query.consul
# and finds the nearest service that matches the glob character (e.g.
# foo-near-self.query.consul will find the nearest healthy foo.service.consul).
resource "consul_prepared_query" "service-near-self" {
  datacenter   = "nyc1"
  token        = "abcd"
  stored_token = "wxyz"
  name         = ""
  only_passing = true
  connect      = true
  near         = "_agent"

  template {
    type   = "name_prefix_match"
    regexp = "^(.*)-near-self$"
  }

  service = "$${match(1)}"

  failover {
    nearest_n   = 3
    datacenters = ["dc2", "dc3", "dc4"]
  }

  dns {
    ttl = "5m"
  }
}
```

## Argument Reference

The following arguments are supported:

* `datacenter` - (Optional) The datacenter to use. This overrides the
  agent's default datacenter and the datacenter in the provider setup.

* `token` - (Optional) The ACL token to use when saving the prepared query.
  This overrides the token that the agent provides by default.

* `stored_token` - (Optional) The ACL token to store with the prepared
  query. This token will be used by default whenever the query is executed.

* `name` - (Required) The name of the prepared query. Used to identify
  the prepared query during requests. Can be specified as an empty string
  to configure the query as a catch-all.

* `service` - (Required) The name of the service to query.

* `session` - (Optional) The name of the Consul session to tie this query's
  lifetime to.  This is an advanced parameter that should not be used without a
  complete understanding of Consul sessions and the implications of their use
  (it is recommended to leave this blank in nearly all cases).  If this
  parameter is omitted the query will not expire.

* `tags` - (Optional) The list of required and/or disallowed tags.  If a tag is
  in this list it must be present.  If the tag is preceded with a "!" then it is
  disallowed.

* `only_passing` - (Optional) When `true`, the prepared query will only
  return nodes with passing health checks in the result.

*  `connect` - (Optional) When `true` the prepared query will return connect
  proxy services for a queried service.  Conditions such as `tags` in the
  prepared query will be matched against the proxy service. Defaults to false.

* `near` - (Optional) Allows specifying the name of a node to sort results
  near using Consul's distance sorting and network coordinates. The magic
  `_agent` value can be used to always sort nearest the node servicing the
  request.

* `ignore_check_ids` - (Optional) Specifies a list of check IDs that should be
  ignored when filtering unhealthy instances. This is mostly useful in an
  emergency or as a temporary measure when a health check is found to be
  unreliable. Being able to ignore it in centrally-defined queries can be
  simpler than de-registering the check as an interim solution until the check
  can be fixed.

* `node_meta` - (Optional) Specifies a list of user-defined key/value pairs that
  will be used for filtering the query results to nodes with the given metadata
  values present.

* `service_meta` - (Optional) Specifies a list of user-defined key/value pairs
  that will be used for filtering the query results to services with the given
  metadata values present.


* `failover` - (Optional) Options for controlling behavior when no healthy
  nodes are available in the local DC.

  * `nearest_n` - (Optional) Return results from this many datacenters,
    sorted in ascending order of estimated RTT.

  * `datacenters` - (Optional) Remote datacenters to return results from.

* `dns` - (Optional) Settings for controlling the DNS response details.

  * `ttl` - (Optional) The TTL to send when returning DNS results.

* `template` - (Optional) Query templating options. This is used to make a
  single prepared query respond to many different requests.

  * `type` - (Required) The type of template matching to perform. Currently
    only `name_prefix_match` is supported.

  * `regexp` - (Required) The regular expression to match with. When using
    `name_prefix_match`, this regex is applied against the query name.

## Attributes Reference

The following attributes are exported:

* `id` - The ID of the prepared query, generated by Consul.

## Import

`consul_prepared_query` can be imported with the query's ID in the Consul HTTP API.

```
$ terraform import consul_prepared_query.my_service 71ecfb82-717a-4258-b4b6-2fb75144d856
```