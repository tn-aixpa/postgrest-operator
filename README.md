# PostgREST Operator
![GitHub Actions Workflow Status](https://img.shields.io/github/actions/workflow/status/tn-aixpa/postgrest-operator/release.yaml?event=release) [![license](https://img.shields.io/badge/license-Apache%202.0-blue)](https://github.com/tn-aixpa/postgrest-operator/LICENSE) ![GitHub Release](https://img.shields.io/github/v/release/tn-aixpa/postgrest-operator)
![Status](https://img.shields.io/badge/status-beta-blue)


A Kubernetes operator to automatically configure and start instances of [PostgREST](https://postgrest.org) connected to databases running in the same cluster.

## Installation
To deploy the operator for testing and evaluation, use the images and configuration files distributed with the package. 

Install operator and CRD:
```sh
kubectl apply -f deployment.yaml
```

The CRD included in the deployment file is found at `config/crd/bases/operator.postgrest.org_postgrests.yaml`.

## Quick start
To deploy an instance of PostgREST connected to a database, add a Custom Resource (CR) with the proper configuration.
An example CR is found at `config/samples/operator_v1_postgrest.yaml`. 

Add the CR:
```sh
kubectl apply -f config/samples/operator_v1_postgrest.yaml
```

Read the following sections to properly configure the deployment.

## Configuration: PostgREST custom resource
A PostgREST custom resource's properties are:

- `schema`: **Required**. The schema PostgREST will expose.
- `anonRole`: *Optional*. The role PostgREST will use to authenticate. If specified, it is assumed to already exist and already have the intended permissions on tables. If not specified, will be auto-generated as as `<CR name>_postgrest_role`.
- `tables`: Do not set if you already set `anonRole`, otherwise required. List of tables within the schema to expose.
- `grants`: *Optional*. Ignored if you already set `anonRole`. Comma-separated string listing actions permitted on tables. Defaults to `SELECT` if not specified. A "full" string is `INSERT, SELECT, UPDATE, DELETE, TRUNCATE, REFERENCES, TRIGGER`, but you may also use `ALL`.
- `connection`: **Required**. A structure to indicate the database to connect to. Its sub-properties are:
  - `host`: *Optional*. Must be provided if `secretName` is unspecified or the secret does not contain `POSTGREST_URL`.
  - `port`: *Optional*.
  - `database`: *Optional*. Must be provided if `secretName` is unspecified or the secret does not contain `POSTGREST_URL`.
  - `user`: Used with `password` to initialize PostgREST. Do not provide if `secretName` is provided.
  - `password`: Used with `user` to initialize PostgREST. Do not provide if `secretName` is provided.
  - `extraParams`: *Optional*. String for extra connection parameters, in the format `parameter1=value&parameter2=value`.
  - `secretName`: Name of a Kubernetes secret containing connection properties. Do not provide if `user` and `password` are provided. More information in a later section.
 
Note that you must provide either `secretName`, or `user` and `password`, but if you provide the former, do not provide the latter, and vice versa.

Note that the user you provide **must have permissions to handle roles** in the database.

### Using a K8S secret to authenticate

Instead of writing user and password as properties, you can provide a `connection.secretName` property, containing a string with the name of a Kubernetes secret to use to authenticate.

Here is a sample file you can apply with `kubectl apply -f secret-file.yml` to create the secret:
``` yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
  namespace: postgrest-operator-system
stringData:
  POSTGRES_URL: postgresql://postgres:postgres@192.168.123.123:5432/postgres?sslmode=disable
  USER: postgres # Only required if POSTGRES_URL is not provided
  PASSWORD: postgres # Only required if POSTGRES_URL is not provided
```
If you omit `POSTGRES_URL`, then `USER` and `PASSWORD` are required, but if you provide it, they will be ignored.

`POSTGRES_URL` uses the format:
```
postgresql://user:password@host:port/database?parameter1=value&parameter2=value
```
 
### Sample configurations

A valid sample spec configuration is:
``` yaml
...
spec:
  schema: operator
  anonRole: anon
  connection:
    host: 192.168.123.123
    database: postgres
    user: postgres
    password: postgres
```

Another valid sample (the secret contains `POSTGRES_URL`):
``` yaml
...
spec:
  schema: operator
  tables:
    - test
  grants: SELECT, UPDATE, INSERT, DELETE
  connection:
    secretName: mysecret
```

Another valid sample (the secret contains `USER` and `PASSWORD`):
``` yaml
...
spec:
  schema: operator
  tables:
    - test
  grants: SELECT, UPDATE, INSERT, DELETE
  connection:
    host: 192.168.123.123
    database: postgres
    secretName: mysecret
```

## Development

The operator is built with [kubebuilder](https://kubebuilder.io/). 
To locally build a version of the program, make sure fullfill all the prerequisites:

*    go version v1.20.0+
*    docker version 17.03+.
*    kubectl version v1.11.3+.
*    Access to a Kubernetes v1.11.3+ cluster.

You can build an executable with the `Makefile`:

```sh
$ make all
```

Afterwards, you can deploy and the via 
```sh
$ make install
$ make run
```


## Security Policy

The current release is the supported version. Security fixes are released together with all other fixes in each new release.

If you discover a security vulnerability in this project, please do not open a public issue.

Instead, report it privately by emailing us at dslab@fbk.eu. Include as much detail as possible to help us understand and address the issue quickly and responsibly.

## Contributing

To report a bug or request a feature, please first check the existing issues to avoid duplicates. If none exist, open a new issue with a clear title and a detailed description, including any steps to reproduce if it's a bug.

To contribute code, start by forking the repository. Clone your fork locally and create a new branch for your changes. Make sure your commits follow the [Conventional Commits v1.0](https://www.conventionalcommits.org/en/v1.0.0/) specification to keep history readable and consistent.

Once your changes are ready, push your branch to your fork and open a pull request against the main branch. Be sure to include a summary of what you changed and why. If your pull request addresses an issue, mention it in the description (e.g., “Closes #123”).

Please note that new contributors may be asked to sign a Contributor License Agreement (CLA) before their pull requests can be merged. This helps us ensure compliance with open source licensing standards.

We appreciate contributions and help in improving the project!

## Authors

This project is developed and maintained by **DSLab – Fondazione Bruno Kessler**, with contributions from the open source community. A complete list of contributors is available in the project’s commit history and pull requests.

For questions or inquiries, please contact: [dslab@fbk.eu](mailto:dslab@fbk.eu)

## Copyright and license

Copyright © 2025 DSLab – Fondazione Bruno Kessler and individual contributors.

This project is licensed under the Apache License, Version 2.0.
You may not use this file except in compliance with the License. Ownership of contributions remains with the original authors and is governed by the terms of the Apache 2.0 License, including the requirement to grant a license to the project.






