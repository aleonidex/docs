---
editable: false
sourcePath: en/_cli-ref/cli-ref/managed-greenplum/cli-ref/pxf-datasource/index.md
---

# yc managed-greenplum pxf-datasource

Manage Greenplum pxf datasources

#### Command Usage

Syntax: 

`yc managed-greenplum pxf-datasource <group|command>`

Aliases: 

- `pxf`

#### Command Tree

- [yc managed-greenplum pxf-datasource delete](delete.md) — Delete pxf-datasource for specified Greenplum cluster
- [yc managed-greenplum pxf-datasource get](get.md) — Get pxf-datasource for specified Greenplum cluster
- [yc managed-greenplum pxf-datasource list](list.md) — List pxf-datasources for specified Greenplum cluster
- [yc managed-greenplum pxf-datasource create](create/index.md) — Create pxf-datasource for specified Greenplum cluster
	- [yc managed-greenplum pxf-datasource create jdbc](create/jdbc.md) — Create jdbc pxf-datasource for specified Greenplum cluster
	- [yc managed-greenplum pxf-datasource create s3](create/s3.md) — Create s3 pxf-datasource for specified Greenplum cluster
- [yc managed-greenplum pxf-datasource update](update/index.md) — Update pxf-datasource for specified Greenplum cluster
	- [yc managed-greenplum pxf-datasource update jdbc](update/jdbc.md) — Get jdbc pxf-datasource for specified Greenplum cluster
	- [yc managed-greenplum pxf-datasource update s3](update/s3.md) — Update s3 pxf-datasource for specified Greenplum cluster

#### Global Flags

| Flag | Description |
|----|----|
|`--profile`|<b>`string`</b><br/>Set the custom configuration file.|
|`--debug`|Debug logging.|
|`--debug-grpc`|Debug gRPC logging. Very verbose, used for debugging connection problems.|
|`--no-user-output`|Disable printing user intended output to stderr.|
|`--retry`|<b>`int`</b><br/>Enable gRPC retries. By default, retries are enabled with maximum 5 attempts.<br/>Pass 0 to disable retries. Pass any negative value for infinite retries.<br/>Even infinite retries are capped with 2 minutes timeout.|
|`--cloud-id`|<b>`string`</b><br/>Set the ID of the cloud to use.|
|`--folder-id`|<b>`string`</b><br/>Set the ID of the folder to use.|
|`--folder-name`|<b>`string`</b><br/>Set the name of the folder to use (will be resolved to id).|
|`--endpoint`|<b>`string`</b><br/>Set the Cloud API endpoint (host:port).|
|`--token`|<b>`string`</b><br/>Set the OAuth token to use.|
|`--impersonate-service-account-id`|<b>`string`</b><br/>Set the ID of the service account to impersonate.|
|`--no-browser`|Disable opening browser for authentication.|
|`--format`|<b>`string`</b><br/>Set the output format: text (default), yaml, json, json-rest.|
|`--jq`|<b>`string`</b><br/>Query to select values from the response using jq syntax|
|`-h`,`--help`|Display help for the command.|
