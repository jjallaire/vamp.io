---
date: 2016-09-13T09:00:00+00:00
title: API - Blueprints
menu:
  main:
    parent: "API"
    identifier: "api-reference-blueprints"
    weight: 30
draft: true
---
Blueprints are static artifacts. They describe how breeds work in runtime and what properties they should have. Read about [using blueprints](documentation/using-vamp/blueprints/).

## Methods
 
 * [List](/documentation/api/api-blueprints/#list-blueprints) - return a list of all blueprints
 * [Get](/documentation/api/api-blueprints/#get-a-single-blueprint) - get a single blueprint
 * [Create](/documentation/api/api-blueprints/#create-a-blueprint) - create a new blueprint 
 * [Update](/documentation/api/api-blueprints/#update-a-blueprint) - update an existing blueprint
 * [Delete](/documentation/api/api-blueprints/#delete-a-blueprint) - delete a blueprint

## Blueprint resource

Vamp requests can be in JSON or YAML format (default JSON). See [common parameters](/documentation/api/api-common-parameters) for details on how to set this.

### JSON

```
[
  {
    "name": "sava:1.0",
    "kind": "blueprint",
    "gateways": {
      "9050": {
        "port": "9050",
        "sticky": null,
        "virtual_hosts": [],
        "routes": {
          "sava/webport": {
            "lookup_name": "58f685a567309e7e6a29d40190bdfd530188d650",
            "weight": null,
            "balance": "default",
            "condition": null,
            "condition_strength": null,
            "rewrites": []
          }
        }
      }
    },
    "clusters": {
      "sava": {
        "services": [
          {
            "breed": {
              "name": "sava:1.0.0",
              "kind": "breed",
              "deployable": {
                "type": "container/docker",
                "definition": "magneticio/sava:1.0.0"
              },
              "ports": {
                "webport": "8080/http"
              },
              "environment_variables": {},
              "constants": {},
              "arguments": [],
              "dependencies": {}
            },
            "environment_variables": {},
            "scale": {
              "cpu": 0.2,
              "memory": "64.00MB",
              "instances": 1
            },
            "arguments": [],
            "dialects": {}
          }
        ],
        "gateways": {},
        "dialects": {}
      }
    },
    "environment_variables": {}
  }
]
```

 Field name        | description          
 -----------------|-----------------
 name |
 gateways | Can be created separately and referenced from here or defined inline as part of the blueprint. See [gateway resource](/documentation/api/api-gateways)
 clusters |
 services |
 breed | Can be created separately and referenced from here or defined inline as part of the blueprint. See [breed resource](/documentation/)
 environment variables |
 scale | Can be created separately and referenced from here or defined inline as part of the blueprint. See [scale resource](/documentation/)

## List blueprints

Returns a list of all stored blueprint resources. For details on pagination see [common parameters](/documentation/api/api-common-parameters)

### Request syntax
    GET /api/v1/blueprints

| Request parameters         | options           | default          | description       |
| ----------------- |:-----------------:|:----------------:| -----------------:|
| `expand_references` | true or false     | false            | all breed references will be replaced (recursively) with full breed definitions. `400 Bad Request` in case some breeds are not yet fully defined.
| `only_references`   | true or false     | false            | all breeds will be replaced with their references.

### Request body
The request body should be empty.

### Response syntax
If successful, will return a list of [blueprint resources](/documentation/api/api-blueprints/#blueprint-resource) in the specified `accept` format (default JSON).  

### Errors
* **400 bad request** `expand_references` set to true and some breeds are not yet fully defined.

## Get a single blueprint

Returns details of a single, specified blueprint.

### Request syntax

    GET /api/v1/blueprints/{blueprint_name}

| Request parameters         | options           | default          | description       |
| ----------------- |:-----------------:|:----------------:| -----------------:|
| `expand_references` | true or false     | false            | all breed references will be replaced (recursively) with full breed definitions. `400 Bad Request` in case some breeds are not yet fully defined.
| `only_references`   | true or false     | false            | all breeds will be replaced with their references.

### Request body
The request body should be empty.

### Response syntax
If successful, will return a single [blueprint resource](/documentation/api/api-blueprints/#blueprint-resource) in the specified `accept` format (default JSON).

### Errors
* **400 bad request** `expand_references` set to true and some breeds are not yet fully defined.


## Create a blueprint

Creates a new blueprint. Create operations are idempotent: sending a second request with the same content will not result in an error response (4xx).

### Request syntax

    POST /api/v1/blueprints


| Request parameters     | options           | default          | description      |
| ------------- |:-----------------:|:----------------:| ----------------:|
| `validate_only` | true or false     | false            | validates the blueprint and returns a `201 Created` if the blueprint is valid.  

### Request body
Should contain a [blueprint resource](/documentation/api/api-blueprints/#blueprint-resource) in the specified `content-type` format (default JSON).

### Response syntax
A successful create operation has status code 201 `Created` and the response body will contain the created [blueprint resource](/documentation/api/api-blueprints/#blueprint-resource) in the specified `accept` format (default JSON). 

### Errors

* ???

## Update a blueprint

Updates the content of a specific blueprint.

### Request syntax

    PUT /api/v1/blueprints/{blueprint_name}

| Request parameters     | options           | default          | description      |
| ------------- |:-----------------:|:----------------:| ----------------:|
| `validate_only` | true or false     | false            | validates the blueprint and returns a `200 OK` if the blueprint is valid.  

### Request body
Should contain a [blueprint resource](/documentation/api/api-blueprints/#blueprint-resource) in the specified `content-type` (default JSON).

### Response syntax
A successful update operation has status code 200 `OK` or 202 `Accepted` and the response body will contain the updated [blueprint resource](/documentation/api/api-blueprints/#blueprint-resource) in the specified `accept` format (default JSON).

### Errors

* **4xx** - an update will fail if a resource does not exist

## Delete a blueprint

Deletes a blueprint. Delete operations are idempotent: sending a second request with the same content will not result in an error response (4xx).

### Request syntax

    DELETE /api/v1/blueprints/{blueprint_name}

| Request parameters     | options           | default          | description      |
| ------------- |:-----------------:|:----------------:| ----------------:|
| `validate_only` | true or false     | false            | returns a `204 No Content` without actual delete of the blueprint.

### Request body

???

### Response syntax

A successful delete operation has status code 204 `No Content` or 202 `Accepted` with an empty response body.

### Errors

* ???