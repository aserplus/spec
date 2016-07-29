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
- [Sub Ops](#sub-ops-run-instruction)

### Container Run Instruction

attributes

| name      | description                    | schema                  | required | default |
|:----------|:-------------------------------|:------------------------|:---------|:--------|
| container | Instruction to run a container | [Container](#container) | true     |         |

### Sub Op Run Instruction

attributes:

| name       | description                                                      | schema                                  | required | default |
|:-----------|:-----------------------------------------------------------------|:----------------------------------------|:---------|:--------|
| isParallel | If the sub op must be run in parallel with the preceeding sub op | [bool](http://yaml.org/type/bool.html)  | false    | false   |
| url        | Url of an op                                                     | [string](http://yaml.org/type/str.html) | true     |         |


### Sub Ops Run Instruction

attributes

| name   | description                    | schema                                                                                          | required | default |
|:-------|:-------------------------------|:------------------------------------------------------------------------------------------------|:---------|:--------|
| subOps | Sub op run instructions to run | [sequence](http://yaml.org/type/seq.html) of [Sub Op Run Instruction](#sub-op-run-instruction)s | true     |         |

## Parameter

Types:

- [Dir](#dir-parameter)
- [File](#file-parameter)
- [String](#string-parameter)

attributes

| name     | description                | schema                                    | required | default |
|:---------|:---------------------------|:------------------------------------------|:---------|:--------|
| type     | Type of parameter          | Name of a defined parameter type          | false    | String  |
| isSecret | If the parameter is secret | [bool](http://yaml.org/type/bool.html)    | false    | false   |
| name     | Name of the parameter      | [FILE_SAFE_NAME](index.md#file_safe_name) | true     |         |

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

| name    | description                                                                                | schema                                                                      | required | default |
|:--------|:-------------------------------------------------------------------------------------------|:----------------------------------------------------------------------------|:---------|:--------|
| image   | [MUST](index.md#mustmay) be a docker image reference (`[[registry/]namespace/]repo[:tag]`) | [string](http://yaml.org/type/str.html)                                     | true     |         |
| mounts  | Mount points inside container                                                              | [sequence](http://yaml.org/types/seq.html) of[Mount Point](#mount-point)s   | false    |         |
| ports   | Ports to bind                                                                              | [sequence](http://yaml.org/type/seq.html) of [Port Binding](#port-binding)s | false    |         |
| process | Process to run                                                                             | [Process](#process)                                                         | true     |         |

### Process

attributes

| name    | description                                             | schema                                                                                | required | default                      |
|:--------|:--------------------------------------------------------|:--------------------------------------------------------------------------------------|:---------|:-----------------------------|
| cmd     | Executable to launch in the container and any args      | [sequence](http://yaml.org/type/seq.html) of [string](http://yaml.org/type/str.html)s | false    | the image’s ENTRYPOINT & CMD |
| envVars | Additional env vars to set in the processes environment | [sequence](http://yaml.org/type/seq.html) of [Env Var](#env-var)s                     | false    | the image's ENV              |
| workDir | Working directory of the process                        | [string](http://yaml.org/type/str.html)                                               | false    | the image's WORKDIR          |

#### Env Var

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

attributes

| name      | description               | schema                                       | required | default   |
|:----------|:--------------------------|:---------------------------------------------|:---------|:----------|
| startPort | Start port number to bind | integer between 0 and 65536                  | true     |           |
| endPort   | End Port number to bind   | integer between 0 and 65536 and >= startPort | false     | startPort |
| protocol  | Procol that will be bound | A Transport layer (Layer 4) protocol         | false    | tcp       |


# Examples

```YAML
name: npm-install
description: installs npm packages
inputs:
- name: NPM_CONFIG_REGISTRY
  description: Registry to use
  isSecret: true
- name: PKG_FILE
  description: package.json file
  type: file
- name: MODULES_DIR
  description: node_modules dir (required for caching)
  type: dir
outputs:
- name: MODULES_DIR
  description: resulting node_modules dir
  type: dir
run:
  container:
    image: docker
    process: 
      cmd: [npm,install]
      env:
      - name: NPM_CONFIG_REGISTRY    
    mounts:
    - name: PKG_FILE
      path: /workdir/package.json
    - name: MODULES_DIR
      path: /workdir/node_modules
```

```YAML
name: gen-docker-config-file
description: generates a docker config file
inputs:
- name: DOCKER_PASSWORD
  description: Password for docker registry
  isSecret: true
- name: DOCKER_USERNAME
  description: Username for docker registry
  isSecret: true
outputs:
- name: DOCKER_CONFIG_FILE
  description: A docker config.json file
  isSecret: true
  type: file
run:
  container:
    image: docker
    process:
      cmd: sudo docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
      envVars:
      - name: DOCKER_PASSWORD
      - name: DOCKER_USERNAME
    mounts:
    - name: DOCKER_CRED_FILE
      path: /root/.docker/config.json
```

```YAML
name: build-and-publish
description: builds the docker image then pushes it to docker hub
inputs:
- name: DOCKER_PASSWORD
  description: Password for docker registry
  isSecret: true
- name: DOCKER_USERNAME
  description: Username for docker registry
  isSecret: true
run:
  subOps
    - url: build
    - url: push-to-docker-hub
```

