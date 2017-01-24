## RabbitMQ authorisation Backend for [Cloud Foundry UAA](https://github.com/cloudfoundry/uaa)

Allows to use access tokens provided by CF UAA to authorize in RabbitMQ.
Make requests to `/check_token` endpoint on UAA server. See https://github.com/cloudfoundry/uaa/blob/master/docs/UAA-APIs.rst#id32

### Usage

First, enable the plugin. Then, configure access to UAA:

``` erlang
{rabbitmq_auth_backend_uaa,
  [{uri,      <<"https://your-uaa-server">>},
   {username, <<"uaa-client-id">>},
   {password, <<"uaa-client-secret">>},
   {resource_server_id, <<"your-resource-server-id"}]}

```

where

 * `your-uaa-server` is a UAA server host
 * `uaa-client-id` is a UAA client ID
 * `uaa-client-secret` is the shared secret
 * `your-resource-server-id` is a resource server ID (e.g. 'rabbitmq')

To learn more about UAA/OAuth 2 clients, see [UAA docs](https://github.com/cloudfoundry/uaa/blob/master/docs/UAA-APIs.rst#id73).

Then you can use `access_tokens` acquired from UAA as username to authenticate in RabbitMQ.

### Scopes

Scopes define token permissions for rabbitmq resources.

Current scope format is `<permission>:<vhost_pattern>/<name_pattern>[/<routing_key_pattern>]`, where

 * `<permission>` is an access permission (`configure`, `read`, or `write`)
 * `<vhost_pattern>` is a wildcard pattern for vhosts, token has acces to.
 * `<name_pattern>` is a wildcard pattern for resource name
 * `<routing_key_pattern>` is an optional wildcard pattern for routing key in topic authorization

Wildcard patterns are strings with optional wildcard symbols `*` that match
any sequence of characters.

Wildcard patterns match as wollowing:

 * `*` matches any strings
 * `foo*` matches any strings, starting with `foo`
 * `*foo` matches any strings, ending with `foo`
 * `foo*bar` matches any strings, starting with `foo` and ending with `bar`

There can be multiple wildcards in a pattern:

 * `start*middle*end`
 * `*before*after*`

**If you want to use special characters like `*`, `%`, or `/` in a wildacrd pattern,
the pattern should be urlencoded.**

See `test/wildcard_match_SUITE.erl` test for more examples

### Authorization workflow

#### Prerequisites

1. There should be application client registered on UAA server.
2. Client id and secret should be set in plugin env as `username` and `password`
3. Client authorities should include `uaa.resource`
4. RabbitMQ auth_backends should include `rabbit_auth_backend_uaa`

#### Authorization

1. Client authorize with UAA, requesting `access_token` (using any grant type)
2. Token scope should contain rabbitmq resource scopes (e.g. configure:%2F/foo - configure queue 'foo' on vhost '/')
3. Client use token as username to connect to RabbitMQ server

