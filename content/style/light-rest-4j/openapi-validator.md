---
title: "OpenAPI Validator"
date: 2017-11-22T22:46:26-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

This handler is part of the [light-rest-4j][] which is built on top of light-4j but focused on RESTful API only. Also, it only works with the OpenAPI 3.0 specification.

It encourages design driven implementation so OpenAPI specification should be done before the implementation starts. With the [light-codegen][] generator, the server stub can be generated and start running within seconds. However, we cannot rely on the generator for validation as specification will be changed along the life cycle of the API. This is why we have provided a validator that works on top of the specification at runtime. In this way, the generator should only be used once, and the validator will take the latest specification and validate according to the specification at runtime. 

### Fail fast

As you may notice that our Status object only supports one code and message. This is the indication the framework validation is designed to fail fast. Whenever there is an error, the server will stop processing the request and return the error to the consumer immediately. There are two reasons on this design, and it is documented in [fail-fast][] in the design section. 

### ValidatorHandler

It is the entry point of the validator, and it is injected during server startup if validator.yml enabled is true. By default, only RequestValidator will be called. Response validation should be done in the [client][] module. 

### Configuration

From release 1.5.18, the light platform supports multiple chains of middleware handlers and multiple frameworks mixed in the same service instance. To have a validator configuration file for different frameworks, a new `openapi-validator.yml` with the same content has been introduced. The `validator.yml` is still loaded if `openapi-validator.yml` doesn't exist for backward compatibility. 

Here is an example of `openapi-validator.yml`

```
# This is specific OpenAPI validator configuration file. It is introduced to support multiple
# frameworks in the same server instance and it is recommended. If this file cannot be found,
# the generic validator.yml will be loaded as a fallback.
---
# Enable request validation. Response validation is not done on the server but client.
enabled: true
# Log error message if validation error occurs
logError: true

```


### RequestValidator

It will validate the following:

* uri
* method
* header
* query parameters
* path parameters
* body if available

When necessary, [json-schema-validator][] will be called to do json schema validation.

### ResponseValidator

It will validate the following:

* header
* response code
* body if available

when necessary, [json-schema-validator][] will be called.

### SchemaValidator

If a schema is defined in openapi.yaml or openapi.json, then the [json-schema-validator][] will be called to validate the input against a JSON schema defined in draft v4.

### Test

In order to test validator, the test suite starts a light-4j server and serves the petstore api for testing. It is a demo on how to unit test your API during development.

[light-rest-4j]: https://github.com/networknt/light-rest-4j
[light-codegen]: /tool/light-codegen/
[fail-fast]: /architecture/fail-fast/
[client]: /concern/client/
[json-schema-validator]: https://github.com/networknt/json-schema-validator
