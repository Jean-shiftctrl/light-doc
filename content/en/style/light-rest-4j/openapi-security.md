---
title: "OpenAPI Security"
date: 2017-11-22T22:46:18-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

This module is part of [light-rest-4j][], built on top of light-4j but focused on RESTful API security only. It only works with the OpenAPI 3.0 specification. 

The JwtVerifyHandler supports OAuth2 with JWT token distributed verification and can be extended to other authentication and authorization approaches with UnifiedSecurityHandler. 

### JwtVerifyHandler

This is the handler that is injected during server startup if security.yml enableVerifyJwt is true. It does further scope verification if enableVerifyScope is true against OpenAPI specification.

From release 1.5.18, Light4J supports multiple chains of middleware handlers and multiple frameworks mixed in the same service instance. In order to have a security configuration file for each framework, a new openapi-security.yml with the same content has been introduced. The security.yml is still loaded if openapi-security.yml doesn’t exist for backward compatibility.

Here is a built-in openapi-security.yml in the module with default configurations.

```
# Security configuration for openapi-security in light-rest-4j. It is a specific config
# for OpenAPI framework security. It is introduced to support multiple frameworks in the
# same server instance. If this file cannot be found, the generic security.yml will be
# loaded for backward compatibility.
---
# Enable the JWT verification flag. The JwtVerifierHandler will skip the JWT token verification
# if this flag is false. It should only be set to false on the dev environment for testing
# purposes. If you have some endpoints that want to skip the JWT verification, you can put the
# request path prefix in skipPathPrefixes.
enableVerifyJwt: ${openapi-security.enableVerifyJwt:true}

# Extract JWT scope token from the X-Scope-Token header and validate the JWT token
enableExtractScopeToken: ${openapi-security.enableExtractScopeToken:true}

# Enable JWT scope verification. This flag is valid when enableVerifyJwt is true. When using the
# light gateway as a centralized gateway without backend API specifications, you can still enable
# this flag to allow the admin endpoints to have scopes verified. And all backend APIs without
# specifications skip the scope verification if the spec does not exist with the skipVerifyScopeWithoutSpec
# flag to true. Also, you need to have the openapi.yml specification file in the config folder to
# enable it, as the scope verification compares the scope from the JWT token and the scope in the
# endpoint specification.
enableVerifyScope: ${openapi-security.enableVerifyScope:true}

# Users should only use this flag in a shared light gateway if the backend API specifications are
# unavailable in the gateway config folder. If this flag is true and the enableVerifyScope is true,
# the security handler will invoke the scope verification for all endpoints. However, if the endpoint
# doesn't have a specification to retrieve the defined scopes, the handler will skip the scope verification.
skipVerifyScopeWithoutSpec: ${openapi-security.skipVerifyScopeWithoutSpec:false}

# Enable JWT scope verification. 
# Only valid when (enableVerifyJwt is true) AND (enableVerifyScope is true)
enableVerifyJwtScopeToken: ${openapi-security.enableVerifyJwtScopeToken:true}

# If set true, the JWT verifier handler will pass if the JWT token is expired already. Unless
# you have a strong reason, please use it only on the dev environment if your OAuth 2 provider
# doesn't support long-lived token for dev environment or test automation.
ignoreJwtExpiry: ${openapi-security.ignoreJwtExpiry:false}

# set true if you want to allow http 1/1 connections to be upgraded to http/2 using the UPGRADE method (h2c).
# By default this is set to false for security reasons. If you choose to enable it make sure you can handle http/2 w/o tls.
enableH2c: ${openapi-security.enableH2c:false}

# User for test only. should be always be false on official environment.
enableMockJwt: ${openapi-security.enableMockJwt:false}

# JWT signature public certificates. kid and certificate path mappings.
jwt:
  certificate: ${openapi-security.certificate:100=primary.crt&101=secondary.crt}
#    '100': primary.crt
#    '101': secondary.crt
  clockSkewInSeconds: ${openapi-security.clockSkewInSeconds:60}
  # Key distribution server standard: JsonWebKeySet for other OAuth 2.0 provider| X509Certificate for light-oauth2
  keyResolver: ${openapi-security.keyResolver:JsonWebKeySet}

# Enable or disable JWT token logging for audit. This is to log the entire token
# or choose the next option that only logs client_id, user_id and scope.
logJwtToken: ${openapi-security.logJwtToken:true}

# Enable or disable client_id, user_id and scope logging if you don't want to log
# the entire token. Choose this option or the option above.
logClientUserScope: ${openapi-security.logClientUserScope:false}

# Enable JWT token cache to speed up verification. This will only verify expired time
# and skip the signature verification as it takes more CPU power and a long time. If
# each request has a different jwt token, like authorization code flow, this indicator
# should be turned off. Otherwise, the cached jwt will only be removed after 15 minutes
# and the cache can grow bigger if the number of requests is very high. This will cause
# memory kill in a Kubernetes pod if the memory setting is limited.
enableJwtCache: ${openapi-security.enableJwtCache:true}

# If enableJwtCache is true, then an error message will be shown up in the log if the
# cache size is bigger than the jwtCacheFullSize. This helps the developers to detect
# cache problem if many distinct tokens flood the cache in a short period of time. If
# you see JWT cache exceeds the size limit in logs, you need to turn off the enableJwtCache
# or increase the cache full size to a bigger number from the default 100.
jwtCacheFullSize: ${openapi-security.jwtCacheFullSize:100}

# If you are using light-oauth2, then you don't need to have oauth subfolder for public
# key certificate to verify JWT token, the key will be retrieved from key endpoint once
# the first token is arrived. Default to false for dev environment without oauth2 server
# or official environment that use other OAuth 2.0 providers.
bootstrapFromKeyService: ${openapi-security.bootstrapFromKeyService:false}

# Used in light-oauth2 and oauth-kafka key service for federated deployment. Each instance
# will have a providerId, and it will be part of the kid to allow each instance to get the
# JWK from other instance based on the providerId in the kid.
providerId: ${openapi-security.providerId:}

# Define a list of path prefixes to skip the security to ease the configuration for the
# handler.yml so that users can define some endpoint without security even through it uses
# the default chain. This is particularly useful in the light-gateway use case as the same
# instance might be shared with multiple consumers and providers with different security
# requirement. The format is a list of strings separated with commas or a JSON list in
# values.yml definition from config server, or you can use yaml format in this file.
skipPathPrefixes: ${openapi-security.skipPathPrefixes:}

```

For detailed information about the properties in the above configuration file, please refer to the light-4j [security][] module.

Unlike the simple web token that the resource server has to contact the Authorization server to validate, JWT can be verified by the resource server as long as the token signing certificate is available at the resource server. 

### UnifiedSecurityHandler

This all-in-one security handler combines Basic, OAuth and API Key to be used in a shared light-gateway instance to handle multiple consumers and providers. For configuration and usage, please visit [UnifiedSecurityHandler][] in the light-gateway. 



[light-rest-4j]: https://github.com/networknt/light-rest-4j
[security]: /concern/security/
[UnifiedSecurityHandler]: /service/gateway/unified-security/


