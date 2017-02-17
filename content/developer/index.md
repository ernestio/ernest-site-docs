---
date: 2016-03-09T20:10:46+01:00
title: Developer
weight: 30
---

## Architecture overview

Ernest is based on a microservice oriented architecture. The majority of the services are built using Go, however we still need to maintain some jruby based in order to interact with vCloud.

## Naming conventions
In order to keep it clean there are some naming conventions you need to follow in order to understand Ernest

### Service Naming
Every service family follows an easy to understand naming convention.

### Mappers
Mappers will be named following the structure -definition-mapper. ‘provider’ is the cloud service it will be mapping, i.e. AWS, vCloud …

### Adapters
We base the adapters naming on the component they’re representing -adapter. A component can be an instance, a network, an s3 bucket…

### Connectors
A connector name will look like ---component.

It is basically composed by some variable parts: - component : the component it represents: instances, networks, routers… - action : the action will implement: create, delete, update. Additionally you will find the action «all», which will represent is implementing all possible actions. - provider : the provider is connecting to. AWS, vCloud…

### Messages naming
All services on Ernest are communicating through [nats](https://nats.io/documentation/concepts/nats-messaging/) messages. There is a naming convention for any message on the platform, and you can find it in different forms:
```sh
<component>.<action>

# These messages are representing the communication between the workflow manager and the builders, and basically they’re managing a list of components to be processed.

<component>.<action>

# A single component to be processed; these messages are managed by builders and adapters.

<component>.<action>.<provider>

# They are sending the necessary info from the adapter to be processed by a specific connector.
```

All these messages are replied with an appended «.done» or «.error» word to the subject, so the sender will know if the action was a success or a failure.

## Service structure
The microservices can be categorized into different families:

- The Public family holds every service with a public or private interface intended to be used by the end user. Api Gateway Monit Logger

- Data stores are a crud over nats interface to interact with persistent data
- Definition Mappers map the input yaml on a valid internal definition
- The Workflow manager takes care of processing the necessary steps to build a service
- Builders are processing lists of components
- Adapters are processing a single component
- Connectors are connecting Ernest with third party providers in order to process a component.

Additionally we use ? to group these families into:

- Frontend (Public + Stores)
- Backend (Mappers + Core + Builders + Adapters + Connectors)

## Public
### API Gateway
The api-gateway is exposing a public REST api, locked with JWT authentication, so every authenticated user can interact with its endpoints

### Monit
This service is sending real time information about service creation to the end user by keeping a stream open with the client.

It is catching all internal messages and decides which ones will fire a new message through the stream

It exposes data streams through SSE(Server Side Events).

### Logger
Logger is listening at all events on the message bus and ofuscating sensible data and sending it to a third party logger system.

## Data stores
Microservices in this family are an interface to interact with persistent data. So you can query stored data through nats with messages including:

```sh
entity.get
entity.find
entity.del
entity.set
```
There is a microservice for each entity on the system:

```sh
datacenter-store

#Implements a nats interface to operate with datacenter stored data

user-store

#Will open a nats interface to operate with user stored data. Additionally it will manage all user sensible data encryption.

service-store

#Implements a nats interface to operate with service stored data

config-store

#Implements a nats interface to operate with system configuration, this configuration is based on a configuration file deployed on Ernest creation.

group-store

#Implements a nats interface to operate with groups stored data. System authorization levels are based on these groups
```
## Definition mappers
Definition mappers map your input yaml/json file on to a valid internal struct at the time they do an initial validation.

```sh
definition-mapper

#Definition mapper service basically decides which provider specific mapper will handle the input mapping.

vcloud-definition-mapper

#Validates and maps the vCloud yaml input on a valid internal service.

aws-definition-mapper

#Validates and maps the AWS yaml input on a valid internal service.
```

## Workflow manager
Workflow manager is a service to process grouped lists of components. It runs on top of nats.io. Workflow manager provides several key features:

- Adaptable workflow definition: Workflow definition is received as part of the entry point, so it is externally defined.
- Templating: You can use the templating system to fill some component details with other component properties.
- External persistence: Workflow manager will call an external service to persist the current definition.
### Quick start

```sh
#Install with git
git clone git@github.com:ernestio/workflow-manager.git

make deps && make install

workflow-manager
```
### dependencies

As workflow-manager does not provide any persistence system it directly depends on service-store, and its communcation is through nats.io.

### Input (definition)
The input definition is a json input with the following structure:

```sh
{
    "id": "test-generated-id",
    "workflow": {
			"arcs": [{
				"from": "created",
				"to": "started",
				"event": "service.create"
			}, {
				"from": "started",
				"to": "creating_components",
				"event": "components.create"
			}, {
				"from": "creating_components",
				"to": "components_created",
				"event": "components.create.done"
			}, {
				"from": "components_created",
				"to": "updating_components",
				"event": "components.update"
			}, {
				"from": "updating_components",
				"to": "components_updated",
				"event": "components.update.done"
			}, {
				"from": "components_updated",
				"to": "deleting_components",
				"event": "components.delete"
			}, {
				"from": "deleting_components",
				"to": "components_deleted",
				"event": "components.delete.done"
			}, {
				"from": "components_deleted",
				"to": "done",
				"event": "service.create.done"
			}, {
				"from": "pre-failed",
				"to": "failed",
				"event": "to_error"
			}, {
				"from": "failed",
				"to": "errored",
				"event": "service.create.error"
			}]
    },
    "components": {
      "status": "",
      "started": "",
      "finished": "",
      "items": [{
        "service": "test",
        "type": "vcloud",
        "name": "existing",
        "field": "existing"
      }]
    },
    "components_to_create": {
      "status": "",
      "started": "",
      "finished": "",
      "items": [{
        "service": "test",
        "type": "vcloud",
        "name": "added",
        "field": "created"
      },{
        "service": "test",
        "type": "vcloud",
        "name": "updated",
        "field": "created_to_be_updated"
      }]
    },
    "components_to_update": {
      "status": "",
      "started": "",
      "finished": "",
      "items": [{
        "service": "test",
        "type": "vcloud",
        "name": "updated",
        "field": "updated"
      }]
    },
    "components_to_delete": {
      "status": "",
      "started": "",
      "finished": "",
      "items": [{
        "service": "test",
        "type": "vcloud",
        "name": "existing"
      }]
    }
}
```
You can change, add or remove as many component batches as you want, they will need to be represented as part of the workflow too.

In order to build workflows, you can have a look at [workflow library](https://github.com/r3labs/workflow).

Workflow-manager will send a <b>components.verb</b> for each transition you’ve defined on your workflow, and will wait for <b>component.verb.status</b>, where status can be <b>done</b> or <b>error</b>.

You will find a more up to date documentation on the repo [README](https://github.com/ernestio/workflow-manager/blob/develop/README.md)

## Component builders
A builder is basically a component scheduler. It receives a list of components and processes each component sequentially or in parallel.

### Sequential processing
In order to process the component sequentially you should set the field «sequential_processing» to true .

### Dependencies
In order to track the state of all components in a collection, state is stored on redis. When all components have completed, a return event is sent.

## Adapters
An adapter is getting an order for a single component creation and it decides which connector will process this creation.

Some of them use a translator so the internal component structure gets adapted to the connector, however, just as we are doing with builders, we’re trying to deliver as many adapters as possible on top of the generic-adapter.

## Connectors
A connector is a service which is directly interacting with the provider in order to process (get, create, update or delete) a component on the provider.

## Testing
### Unit testing
Each service has its own test suite (per-service tests) based uniquely on how the specific service behaves.
### BDD tests
In order to test, and keep a live documentation of ernest-cli, we have implemented some end to end tests using [gucumber](https://github.com/gucumber/gucumber) and [gherkin](https://github.com/cucumber/cucumber/wiki/Gherkin).

You can find the features definitions describing ernest behaviour [here](https://github.com/ernestio/ernest-cli/tree/develop/internal/features/cli)
### Integration tests
As Ernest is a distributed system, it’s sometimes hard to have integration tests. This is why some key services have integration tests and per service tests.

These integration tests are building a partial Ernest environment with docker-compose, but they’re mocking all connectors with all-all-fake-connector which responds with what’s expected by the connectors.
