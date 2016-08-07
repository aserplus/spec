# Name

[MUST](index.md#mustmay) be named `op.yml`

# Format

[MUST](index.md#mustmay) be
[v1.2 yaml](http://www.yaml.org/spec/1.2/spec.html)

# Schema

attributes:

| name        | description           | schema                                                                | required | default |
|:------------|:----------------------|:----------------------------------------------------------------------|:---------|:--------|
| name        | Name of the op        | [FILE_SAFE_NAME](index.md#file_safe_name)                             | true     |         |
| description | Description of the op | [string](http://yaml.org/type/str.html)                               | true     |         |
| inputs      | Inputs to the op      | [sequence](http://yaml.org/type/seq.html) of [Parameter](#parameter)s | false    |         |
| outputs     | Outputs from the op   | [sequence](http://yaml.org/type/seq.html) of [Parameter](#parameter)s | false    |         |
| run         | Run instruction       | [Run Instruction](#run-instruction)                                   | true     |         |
| version     | Version of the op     | [v2.0.0 SemVer](http://semver.org/spec/v2.0.0.html)                   | false    |         |

## Run Instruction

Types:

- [Container](#container-run-instruction)
- [Parallel](#parallel-run-instruction)
- [Serial](#serial-run-instruction)

## Container Run Instruction

attributes

| name      | description                    | schema                  | required | default |
|:----------|:-------------------------------|:------------------------|:---------|:--------|
| container | Instruction to run a container | [Container](#container) | true     |         |

## Serial Run Instruction

attributes

| name   | description                              | schema                                                                  | required | default |
|:-------|:-----------------------------------------|:------------------------------------------------------------------------|:---------|:--------|
| serial | Sequence of \[bound] ops to run serially | [sequence](http://yaml.org/type/seq.html) of [Op Binding](#op-binding)s | true     |         |

## Parallel Run Instruction

attributes

| name     | description                                 | schema                                                                  | required | default |
|:---------|:--------------------------------------------|:------------------------------------------------------------------------|:---------|:--------|
| parallel | Sequence of \[bound] ops to run in parallel | [sequence](http://yaml.org/type/seq.html) of [Op Binding](#op-binding)s | true     |         |

### Op Binding

attributes:

| name | description         | schema                                                                          | required | default |
|:-----|:--------------------|:--------------------------------------------------------------------------------|:---------|:--------|
| ->   | Input bindings      | [sequence](http://yaml.org/type/seq.html) of [Input Binding](#input-binding)s   | false    |         |
| op   | Reference to the op | [OP_REF](index.md#op_ref)                                                       | true     |         |
| <-   | Output bindings     | [sequence](http://yaml.org/type/seq.html) of [Output Binding](#output-binding)s | false    |         |

#### Input Binding

A [string](http://yaml.org/type/str.html) matching regex
`INPUT_PARAM(=CONTEXT_VAR|STRING)?`

where:

- `STRING` is a [string](http://yaml.org/type/str.html)
- `=CONTEXT_VAR|STRING` is optional only when `INPUT_PARAM` and
  `CONTEXT_VAR` have the same name.

#### Output Binding

A [string](http://yaml.org/type/str.html) matching regex
`CONTEXT_VAR(=OUTPUT_PARAM|STRING)?`

where:

- `STRING` is a [string](http://yaml.org/type/str.html)
- `=OUTPUT_PARAM|STRING` is optional only when `OUTPUT_PARAM` and
  `CONTEXT_VAR` have the same name.

## Parameter

Types:

- [Dir](#dir-parameter)
- [File](#file-parameter)
- [String](#string-parameter)

attributes

| name        | description                  | schema                                    | required | default |
|:------------|:-----------------------------|:------------------------------------------|:---------|:--------|
| type        | Type of parameter            | Name of a defined parameter type          | false    | String  |
| description | Description of the parameter | [string](http://yaml.org/type/str.html)   | false    |         |
| isSecret    | If the parameter is secret   | [bool](http://yaml.org/type/bool.html)    | false    | false   |
| name        | Name of the parameter        | [FILE_SAFE_NAME](index.md#file_safe_name) | true     |         |

### Dir Parameter

inherits [Parameter](#parameter) attributes

### File Parameter

inherits [Parameter](#parameter) attributes

### String Parameter

inherits [Parameter](#parameter) attributes

additional attributes:

| name    | description   | schema                                  | required | default |
|:--------|:--------------|:----------------------------------------|:---------|:--------|
| default | default value | [string](http://yaml.org/type/str.html) | false    |         |

## Container

attributes

| name    | description                                             | schema                                                                                | required | default                      |
|:--------|:--------------------------------------------------------|:--------------------------------------------------------------------------------------|:---------|:-----------------------------|
| image   | Docker image to use                                     | [Docker Image Ref](index.md#docker_image_ref)                                         | true     |                              |
| mounts  | Mount points inside container                           | [sequence](http://yaml.org/types/seq.html) of[Mount Point](#mount-point)s             | false    |                              |
| ports   | Ports to bind                                           | [sequence](http://yaml.org/type/seq.html) of [Port Binding](#port-binding)s           | false    |                              |
| cmd     | Executable to launch in the container and any args      | [sequence](http://yaml.org/type/seq.html) of [string](http://yaml.org/type/str.html)s | false    | the image’s ENTRYPOINT & CMD |
| envVars | Additional env vars to set in the processes environment | [sequence](http://yaml.org/type/seq.html) of [Env Var](#env-var)s                     | false    | the image's ENV              |
| workDir | Working directory of the process                        | [string](http://yaml.org/type/str.html)                                               | false    | the image's WORKDIR          |

### Env Var

attributes  

| name    | description                                                                      | schema                                  | required | default |
|:--------|:---------------------------------------------------------------------------------|:----------------------------------------|:---------|:--------|
| default | Default value                                                                    | [string](http://yaml.org/type/str.html) | false    |         |
| name    | Name of the mount point. To bind to input and/or output parameter use same name. | [string](http://yaml.org/type/str.html) | true     |         |

### Mount Point

attributes  

| name | description                                                                      | schema                                    | required | default |
|:-----|:---------------------------------------------------------------------------------|:------------------------------------------|:---------|:--------|
| path | Path within the container at which the volume should be mounted                  | [string](http://yaml.org/type/str.html)   | true     |         |
| name | Name of the mount point. To bind to input and/or output parameter use same name. | [FILE_SAFE_NAME](index.md#file_safe_name) | true     |         |

### Port Binding

[string](http://yaml.org/type/str.html) matching regex
`[1-65535](-[1-65535])?`

#### examples

`8080-8090`


