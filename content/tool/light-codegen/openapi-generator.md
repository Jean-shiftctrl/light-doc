---
title: "OpenAPI Generator"
date: 2019-01-05T11:27:08-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---


This generator is based on OpenAPI 3.0 specification, and it is a very new specification that is supposed to replace Swagger 2.0 specification. It has some significant changes to enhance the spec definition and simply the validate with only JSON schema. In my opinion, it is much easier to use, and the implementation is much simpler then Swagger 2.0.
OpenAPI adds a number of features The only issue I can see is the entire toolchains around Swagger 2.0 are not migrated to OpenAPI 3.0 yet, and we have to build our own [openapi-parse][] for parsing and validation.

## Input

#### Model

In light-rest-4j framework generator, the model that drives code generation is the OpenAPI 3.0 specification previously named Swagger specification. When editing it, it usually will be in YAML format with separate files for readability and flexibility. Before leverage it in the light-rest-4j framework, all YAML files need to be bundled to a single file in YAML or JSON format to be consumed by the framework and generator. Also, validation needs to be done to make sure that the openapi.yaml or openapi.json is valid against JSON schema of OpenAPI 3.0 specification.

Note: currently, we support both OpenAPI 3.0 specification and Swagger 2.0 specification. There is a similar [Swagger 2.0 generator tutorial] available.

- [Swagger Editor][]

- [Swagger CLI][]


#### Config

Here is an example of config.json for openapi generator.

```
{
  "name": "petstore",
  "version": "1.0.1",
  "groupId": "com.networknt",
  "artifactId": "petstore",
  "rootPackage": "com.networknt.petstore",
  "handlerPackage":"com.networknt.petstore.handler",
  "modelPackage":"com.networknt.petstore.model",
  "overwriteHandler": true,
  "overwriteHandlerTest": true,
  "overwriteModel": true,
  "generateModelOnly": false,
  "generateValuesYml": fale,
  "regenerateCodeOnly":false,
  "httpPort": 8080,
  "enableHttp": false,
  "httpsPort": 8443,
  "enableHttps": true,
  "enableHttp2": true,  
  "enableRegistry": false,
  "enableParamDescription": true,
  "supportDb": true,
  "dbInfo": {
    "name": "mysql",
    "driverClassName": "com.mysql.jdbc.Driver",
    "jdbcUrl": "jdbc:mysql://mysqldb:3306/oauth2?useSSL=false",
    "username": "root",
    "password": "my-secret-pw"
  },
  "supportH2ForTest": false,
  "supportClient": false,
  "skipHealthCheck": false,
  "skipServerInfo": false,
  "prometheusMetrics": false
}
```

- name is used in generated pom.xml for project name
  - optionality: mandatory
- version is used in generated pom.xml for project version
  - optionality: mandatory
- groupId is used in generated pom.xml for project groupId
  - optionality: mandatory
- artifactId is used in generated pom.xml for project artifactId
  - optionality: mandatory
- rootPackage is the root package name for your project and it will normally be your domain plug project name.
  - optionality: optional
- handlerPackage is the Java package for all generated handlers.
  - optionality: mandatory
- modelPackage is the Java package for all generated models or POJOs.
  - optionality: mandatory
- overwriteHandler controls if you want to overwrite handler when regenerating the same project into the same folder.
  - optionality: mandatory
  - recommendation: set to false if you only want to upgrade the framework to another minor version and don't want to overwrite handlers
- overwriteHandlerTest controls if you want to overwrite handler test cases.
  - optionality: mandatory
- overwriteModel controls if you want to overwrite generated models.
  - optionality: mandatory
  - recommendation: set to false to prevent the model classes being overwritten
- generateModelOnly controls whether you wish to generate only the model classes
  - optionality: optional
  - recommendation: to be used by teams consuming an API and who wish to generate the model classes only
  - default: false
- regenerateCodeOnly controls whether you wish to regenerate only the model and handler classes, while skipping the underlying scripts, pom.xml and other files
  - optionality: optional
  - recommendation: to be used when there are changes in the model and teams wish to regenerate only the artifacts affected by such change: model, handler classes and handler.yml
  - default: false
- generateValuesYml controls whether a values.yml is to be generated, with commonly changed values across test and production environments
  - default: false
- httpPort is the port number of Http listener if enableHttp is true
  - optionality: mandatory
- enableHttp to specify if the server listens to http port.
  - optionality: mandatory
  - recommendation: Http should only be enabled in dev.
- httpsPort is the port number of Https listener if enableHttps is true.
  - optionality: mandatory
- enableHttps to specify if the server listens to https port.
  - optionality: mandatory
  - recommendation: Https should be used in any official environment for security reason.
  - note: when Https is enabled, Http will automatically be disabled
- enableHttp2 to specify if the server supports HTTP/2 connection.
  - optionality: mandatory
  - recommendation: it should be set always to true
- enableRegistry to control if built-in service registry/discovery is used.
  - optionality: mandatory
  - note: Only necessary if running as standalone java -jar xxx
- enableParamDescription is to decide if generate parameter description from specifications as annotation.
  - optionality: optional
  - default: true
- supportDb to control if db connection pool will be setup in service.yml and db dependencies are included in pom.xml
  - optionality: mandatory
- dbInfo section is the database connection pool configuration info.
  - optionality: mandatory
- supportH2ForTest if true, add H2 in pom.xml as test scope to support unit test with H2 database.
  - optionality: mandatory
- supportClient if true, add com.networknt.client module to pom.xml to support service to service call.
  - optionality: mandatory
- skipHealthCheck is to decide whether to enable the health check in the handler chain and expose the health check endpoint. Set to true to skip the wiring of the health check
  - optionality: optional
  - default: false
- skipServerInfo is to decide whether to wire the server info in the handler chain and expose the server info endpoint. Set to true to skip the wiring of the server info retrieval
    - optionality: optional
    - default: false
- prometheusMetrics is to decide whether to wire the Prometheus metrics collection handler in the handler chain. Set to true to skip the wiring of the Prometheus metrics collection handler
    - optionality: optional
    - default: false

In most of the cases, developers will only update handlers, handler tests and models in a generated project. Of course, you might need a different database for your project, and we have a [database tutorial] that can help you to further config Oracle and Postgres.   

Given we have most of our model and config files in [model-config][] repo, most generator input would come from the rest folder in model-config for the light-rest-4j framework.

Let's clone the project to your workspace as we will need it in the following steps. I am using ~/networknt as a workspace, but it can be anywhere in your home directory.


```
cd ~/networknt
git clone https://github.com/networknt/model-config.git
```


## Usage

#### Java Command line

Before using the command line to generate the code, you need to check out the light-codegen repo and build it. I am using ~/networknt as the workspace, but it can be anywhere in your home directory.  

```
cd ~/networknt
git clone https://github.com/networknt/light-codegen.git
cd light-codegen
mvn clean install
```

* JSON Model

Given we have test openapi.json and config.json in light-rest-4j/src/test/resources folder, the following command line will generate a RESTful  API at /tmp/openapi-json folder.

Working directory: light-codegen

```
cd ~/networknt/light-codegen
java -jar codegen-cli/target/codegen-cli.jar -f openapi -o /tmp/openapi-json -m light-rest-4j/src/test/resources/openapi.json -c light-rest-4j/src/test/resources/config.json
```

After you run the above command, you can build and start the service:
```
cd /tmp/openapi-json
mvn clean install exec:exec
```

To test the service from another terminal:
```
curl http://localhost:8080/server/info
```

* YAML Model

Given we have test openapi.yaml and config.json in light-rest-4j/src/test/resources folder, the following command line will generate a RESTful  API at /tmp/openapi-yaml folder.

Working directory: light-codegen

```
cd ~/networknt/light-codegen
java -jar codegen-cli/target/codegen-cli.jar -f openapi -o /tmp/openapi-yaml -m light-rest-4j/src/test/resources/openapi.yaml -c light-rest-4j/src/test/resources/config.json
```

After you run the above command, you can build and start the service:
```
cd /tmp/openapi-yaml
mvn clean install exec:exec
```

To test the service from another terminal:
```
curl http://localhost:8080/server/info
```


The above example use local OpenAPI specification and config file. Let's try to use files from github.com:

Working directory: light-codegen

```
rm -rf /tmp/openapi-petstore
cd ~/networknt/light-codegen
java -jar codegen-cli/target/codegen-cli.jar -f openapi -o /tmp/openapi-petstore -m https://raw.githubusercontent.com/networknt/model-config/master/rest/openapi/petstore/1.0.0/openapi.json -c https://raw.githubusercontent.com/networknt/model-config/master/rest/openapi/petstore/1.0.0/config.json
```

Please note that you need to use a raw url when accessing github files. The above command line will generate a petstore service in /tmp/openapi-petstore.

Given we have most of the model and config files in model-config repo, most generator input would from the rest folder in model-config. Here is the example to generate petstore. Assuming model-config is in the same workspace as light-codegen.

Working directory: networknt

```
rm -rf /tmp/openapi-petstore
cd ~/networknt
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f openapi -o /tmp/openapi-petstore -m model-config/rest/openapi/petstore/1.0.0/openapi.json -c model-config/rest/openapi/petstore/1.0.0/config.json

```

#### Docker Command Line

Above local build and command line utility works but it is very hard to use that in devops script. In order to make scripting easier, we have dockerized the command line utility.


The following command is using docker image to generate the code into /tmp/light-codegen/generated:
```
docker run -it -v ~/networknt/light-codegen/light-rest-4j/src/test/resources:/light-api/input -v /tmp/light-codegen:/light-api/out networknt/light-codegen -f openapi -m /light-api/input/openapi.json -c /light-api/input/config.json -o /light-api/out/generated
```
On Linux environment, the generated code might belong to root:root and you need to change the owner to yourself before building it.

```
cd /tmp/light-codegen
sudo chown -R steve:steve generated
cd generated
mvn clean install exec:exec
```
To test it.
```
curl localhost:8080/v1/pets/111
```

#### Docker Scripting

You can use docker run command to call the generator but it is very complicated for the parameters. In order to make things easier and friendlier to devops flow. Let's create a script to call the command line from docker image.

If you look at the docker run command you can see that we basically need one input folder for schema and config files and one output folder to generated code. Once these volumes are mapped to local directory and with framework specified, it is easy to derive other files based on
convention.


```
cd model-config
./generate.sh openapi ~/networknt/model-config/rest/openapi/petstore/1.0.0 /tmp/petstore
```
Now you should have a project generated in /tmp/petstore/genereted

#### Codegen Site

The service API is ready. We are working on the UI with a generation wizard.


[openapi-parser]: https://github.com/networknt/openapi-parser
[Swagger 2.0 generator tutorial]: /tutorial/generator/swagger/
[Swagger Editor]: /tool/swagger-editor/
[Swagger CLI]: /tool/swagger-cli/
[database tutorial]: /tutorial/rest/swagger/database/
[model-config]: https://github.com/networknt/model-config
