---
title: deck dump
source_url: https://github.com/Kong/deck/tree/main/cmd
content-type: reference
---

The dump command reads all entities present in Kong
and writes them to a local file.

The file can then be read using the sync command or diff command to
configure Kong.

## Syntax

```
deck dump [command-specific flags] [global flags]
```

## Flags

`--all-workspaces`
:  dump configuration of all Workspaces (Kong Enterprise only). (Default: `false`)

`--format`
:  output file format: json or yaml. (Default: `"yaml"`)

`-h`, `--help`
:  help for dump (Default: `false`)

`-o`, `--output-file`
:  file to which to write Kong's configuration.Use `-` to write to stdout. (Default: `"kong"`)

`--rbac-resources-only`
:  export only the RBAC resources (Kong Enterprise only). (Default: `false`)

`--select-tag`
:  only entities matching tags specified with this flag are exported.
When this setting has multiple tag values, entities must match every tag.

{% if_version gte:1.12.x %}

`--skip-ca-certificates`
:  do not dump CA certificates. (Default: `false`)

{% endif_version %}

`--skip-consumers`
:  skip exporting consumers and any plugins associated with consumers. (Default: `false`)

`--with-id`
:  write ID of all entities in the output (Default: `false`)

`-w`, `--workspace`
:  dump configuration of a specific Workspace(Kong Enterprise only).

`--yes`
:  assume `yes` to prompts and run non-interactively. (Default: `false`)

{% if_version gte:1.7.x %}

## Global flags

`--analytics`
:  Share anonymized data to help improve decK.
Use `--analytics=false` to disable this. (Default: `true`)

`--ca-cert`
:  Custom CA certificate (raw contents) to use to verify Kong's Admin TLS certificate.
This value can also be set using DECK_CA_CERT environment variable.
This takes precedence over `--ca-cert-file` flag.

{% if_version gte:1.8.x %}

`--ca-cert-file`
:  Path to a custom CA certificate to use to verify Kong's Admin TLS certificate.
This value can also be set using DECK_CA_CERT_FILE environment variable.

{% endif_version %}

`--config`
:  Config file (default is $HOME/.deck.yaml).

`--headers`
:  HTTP headers (key:value) to inject in all requests to Kong's Admin API.
This flag can be specified multiple times to inject multiple headers.

`--kong-addr`
:  HTTP address of Kong's Admin API.
This value can also be set using the environment variable DECK_KONG_ADDR
 environment variable. (Default: `"http://localhost:8001"`)

{% if_version gte: 1.10.x %}

`--kong-cookie-jar-path`
:  Absolute path to a cookie-jar file in the Netscape cookie format for auth with Admin Server.
You may also need to pass in as header the User-Agent that was used to create the cookie-jar.

{% endif_version %}

`--konnect-addr`
:  Address of the {{site.konnect_short_name}} endpoint. (Default: `"https://us.api.konghq.com"`)

`--konnect-email`
:  Email address associated with your {{site.konnect_short_name}} account.

`--konnect-password`
:  Password associated with your {{site.konnect_short_name}} account, this takes precedence over `--konnect-password-file` flag.

`--konnect-password-file`
:  File containing the password to your {{site.konnect_short_name}} account.

{% if_version gte:1.12.x %}

`--konnect-runtime-group-name`
:  {{site.konnect_short_name}} Runtime group name.

{% endif_version %}

`--no-color`
:  Disable colorized output (Default: `false`)

`--skip-workspace-crud`
:  Skip API calls related to Workspaces (Kong Enterprise only). (Default: `false`)

{% if_version gte:1.8.x %}

`--timeout`
:  Set a request timeout for the client to connect with Kong (in seconds). (Default: `10`)

{% endif_version %}

{% if_version gte:1.11.x %}

`--tls-client-cert`
:  PEM-encoded TLS client certificate to use for authentication with Kong's Admin API.
This value can also be set using DECK_TLS_CLIENT_CERT environment variable. Must be used in conjunction with tls-client-key


`--tls-client-cert-file`
:  Path to the file containing TLS client certificate to use for authentication with Kong's Admin API.
This value can also be set using DECK_TLS_CLIENT_CERT_FILE environment variable. Must be used in conjunction with tls-client-key-file

`--tls-client-key`
:  PEM-encoded private key for the corresponding client certificate .
This value can also be set using DECK_TLS_CLIENT_KEY environment variable. Must be used in conjunction with tls-client-cert

`--tls-client-key-file`
:  Path to file containing the private key for the corresponding client certificate.
This value can also be set using DECK_TLS_CLIENT_KEY_FILE environment variable. Must be used in conjunction with tls-client-cert-file

{% endif_version %}

`--tls-server-name`
:  Name to use to verify the hostname in Kong's Admin TLS certificate.
This value can also be set using DECK_TLS_SERVER_NAME environment variable.

`--tls-skip-verify`
:  Disable verification of Kong's Admin TLS certificate.
This value can also be set using DECK_TLS_SKIP_VERIFY environment variable. (Default: `false`)

`--verbose`
:  Enable verbose logging levels
Setting this value to 2 outputs all HTTP requests/responses
between decK and Kong. (Default: `0`)


{% endif_version %}

## See also

* [deck](/deck/{{page.kong_version}}/reference/deck)	 - Administer your Kong clusters declaratively

