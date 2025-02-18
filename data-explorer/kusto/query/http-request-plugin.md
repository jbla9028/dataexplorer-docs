---
title: http_request plugin - Azure Data Explorer
description: This article describes http_request plugin in Azure Data Explorer.
services: data-explorer
ms.reviewer: zivc
ms.topic: reference
ms.date: 03/07/2022
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
---
# http_request plugin / http_request_post plugin

::: zone pivot="azuredataexplorer"

The `http_request` (GET) and `http_request_post` (POST) plugins send an HTTP request and convert the response into a table.

> [!IMPORTANT]
> Both plugins are disabled by default, as they allow queries to send data
> and the user's security token to external user-specified network endpoints.
> Once enabled, both plugins are further subject to the configured
> [callout policy](../management/calloutpolicy.md) that cluster admins
> can use to granularly control which URIs the request can be sent to.

## Syntax

`evaluate` `http_request` `(` *Uri* [`,` *RequestHeaders* [`,` *Options*]] `)`

`evaluate` `http_request_post` `(` *Uri* [`,` *RequestHeaders* [`,` *Options* [`,` *Content*]]] `)`

## Arguments

| Name | Type | Required | Description |
|--|--|--|--|
| *Uri* | string | &check; | The destination URI for the HTTP or HTTPS request. |
| *RequestHeaders* | dynamic |  | A property bag containing [HTTP headers](#headers) to send with the request. |
| *Options* | dynamic |  | A property bag containing additional properties of the request. |
| *Content* | string |  | The body content to send with the request. The content is encoded in `UTF-8` and the media type for the `Content-Type` attribute is `application/json`. |

## Returns

Both plugins return a table that has a single record with the following dynamic columns:

* *ResponseHeaders*: A property bag with the response header.
* *ResponseBody*: The response body parsed as a value of type `dynamic`.

## Prerequisites

Before you use the `http_request` and `http_request_post` plugins, make sure that requests meet the following requirements:

* The specified *Uri* value must be a destination that is enabled for `webapi` callout by the [Callout policy](../management/calloutpolicy.md). Otherwise, running the query results in an error.

* If you're using authentication, you must use the HTTPS protocol. Attempts to use HTTP with authentication enabled results in an error.

## Authentication

You can use the query arguments to specify authentication parameters for the `http_request` and `http_request_post` plugins. The following scenarios are supported:

| Argument | Description |
|--|--|
| *Uri* | The URI to authenticate with. |
| *RequestHeaders* | Using the HTTP standard `Authorization` header or any custom header supported by the web service. |

<!--
| *Options* | Using the HTTP standard `Authorization` header.<br />If you want to use Azure Active Directory (Azure AD) authentication, you must use an HTTPS URI for the request and set the following values:<br />* `azure_active_directory` to `Active Directory Integrated`<br />* `AadResourceId` to the Azure AD ResourceId value of the target web service. |
-->

> [!WARNING]
> Be extra careful not to send secret information, such as
> authentication tokens, over HTTP connections. Additionally, if the query includes
> confidential information, make sure that the relevant parts of the
> query text are obfuscated so that they'll be omitted from any tracing.
> For more information, see [obfuscated string literals](./scalar-data-types/string.md#obfuscated-string-literals).

## Headers

The *RequestHeaders* argument can be used to add custom headers
to the outgoing HTTP request. In addition to the standard HTTP request headers
and the user-provided custom headers, the plugin also adds the following
custom headers:

| Name | Description |
|--|--|
| `x-ms-client-request-id` | A correlation ID that identifies the request. Multiple invocations of the plugin in the same query will all have the same ID. |
| `x-ms-readonly` | A flag indicating that the processor of this request shouldn't make any persistent changes. |

> [!WARNING]
> The `x-ms-readonly` flag is set for every HTTP request sent by the plugin
> that was triggered by a query and not a control command. Web services should
> treat any requests with this flag as one that does not make internal
> state changes, otherwise they should refuse the request. This protects users from being
> sent seemingly-innocent queries that end up making unwanted changes by using
> a Kusto query as the launchpad for such attacks.

## Examples

The following example retrieves the canonical list of country codes:

<!-- csl -->
```kusto
evaluate http_request('http://services.groupkt.com/country/get/all')
| project CC=ResponseBody.RestResponse.result
| mv-expand CC limit 10000
| project
    name        = tostring(CC.name),
    alpha2_code = tostring(CC.alpha2_code),
    alpha3_code = tostring(CC.alpha3_code)
| where name startswith 'b'
```

name                              | alpha2_code  | alpha3_code
----------------------------------|--------------|-------------
Bahamas                           | BS           | BHS
Bahrain                           | BH           | BHR
Bangladesh                        | BD           | BGD
Barbados                          | BB           | BRB
Belarus                           | BY           | BLR
Belgium                           | BE           | BEL
Belize                            | BZ           | BLZ
Benin                             | BJ           | BEN
Bermuda                           | BM           | BMU
Bhutan                            | BT           | BTN
Bolivia (Plurinational State of)  | BO           | BOL
Bonaire, Sint Eustatius and Saba  | BQ           | BES
Bosnia and Herzegovina            | BA           | BIH
Botswana                          | BW           | BWA
Bouvet Island                     | BV           | BVT
Brazil                            | BR           | BRA
British Indian Ocean Territory    | IO           | IOT
Brunei Darussalam                 | BN           | BRN
Bulgaria                          | BG           | BGR
Burkina Faso                      | BF           | BFA
Burundi                           | BI           | BDI

The following example is for a hypothetical HTTPS web service that accepts additional request headers and must be authenticated to using Azure AD:

<!-- csl -->
```kusto
let uri='https://example.com/node/js/on/eniac';
let headers=dynamic({'x-ms-correlation-vector':'abc.0.1.0', 'authorization':'bearer ...Azure-AD-bearer-token-for-target-endpoint...'});
evaluate http_request_post(uri, headers)
```

::: zone-end

::: zone pivot="azuremonitor"

This capability isn't supported in Azure Monitor

::: zone-end
