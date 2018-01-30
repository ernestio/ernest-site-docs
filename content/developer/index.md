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

### Connectors
A connector name will look like component-action-provider-component.

It is basically composed by some variable parts: - component : the component it represents: instances, networks, routers… - action : the action will implement: create, delete, update. Additionally you will find the action «all», which will represent is implementing all possible actions and component types. - provider : the provider is connecting to. AWS, vCloud…

### Messages naming
All services on Ernest are communicating through [nats](https://nats.io/documentation/concepts/nats-messaging/) messages. There is a naming convention for any message on the platform, and you can find it in different forms:
```sh

<component>.<action>.<provider>

# They are sending the necessary info from the adapter to be processed by a specific connector.
```

All these messages are replied with an appended «.done» or «.error» word to the subject, so the sender will know if the action was a success or a failure.

# Component Message structure
There are a number of internal fields used by components (always prefixed with `_`) that must be present:

- `_action`:  The action required by the component. This can be one of `create`, `update`, `delete`, `find` or `get`.
- `_component`:  The component type, i.e. `instance`, `network`, `firewall`.
- `_component_id`:  The composite ID of the component. This is constructed by a components type and its name, i.e. `instance::web-1`
- `_provider`:  The type of provider that the component will be provisioned on.
- `_state`:  The current state of the component. Can be one of `errored`, `waiting`, `running`, `completed`

These fields are leveraged by scheduler to construct the omitted event subject.

## Service structure
The microservices can be categorized into different families:

- The Public family holds every service with a public or private interface intended to be used by the end user [Api Gateway, Monit, Logger].

- Data stores are a crud over nats interface to interact with persistent data.
- The Definition Mapper maps the input yaml on a valid internal mapping.
- The Scheduler takes care of processing the necessary steps to build a service.
- Connectors are connecting Ernest with third party providers in order to process a component.

We group these families into:

- Frontend (Public + Stores)
- Backend (Mapper + Core + Connectors)

## Public
### API Gateway
The api-gateway is exposing a public REST api, locked with JWT authentication, so every authenticated user can interact with its endpoints.

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
## Definition mapper
Definition mapper maps your input yaml/json file on to a valid internal struct at the time they do an initial validation.

```sh
definition-mapper

#Validates and maps the provider specific yaml input to a valid internal service mapping. It's also is responsible for converting the internal mapping back to an input yaml when importing.
```

## Scheduler
Scheduler is a service to process actions associated with a given environment and it's components. It runs on top of nats.io.

Scheduler provides several key features:

- Dependency graph scheduling. Each component is scheduled based on its relation to other components and allows independant components to be provisioned in parallel.
- Templating: You can use the templating system to fill some component details with other component properties.
- External persistence: Scheduler will call an external service to persist the current state of the service build and its components.

### Quick start

```sh
#Install with git
git clone git@github.com:ernestio/scheduler.git

make deps && make install

scheduler
```

### Running tests

```sh
make dev-deps && make test
```

### Operation

The scheduler will send a `components.verb.provider` for component. The order is defined by the dependencies specified on `edges` and will wait for `component.verb.provider.status`, where status can be `done` or `error`.

When a completed component event is received, the latest service mapping is retrieved from service store. The state of the component is then updated in both `changes` and `components`. The graph is then inspected for all dependants of the completed component. These dependant components are then scheduled when all of its dependencies are satisfied.

If an errored component is received, scheduler will wait for all other in flight components to complete before sending an error back to the user. No other components will be sent in this errored state.

The graph library utilised by the scheduler can be found at [graph library](https://github.com/ernestio/graph).

### External Dependencies

As scheduler does not provide any persistence system; it directly depends on service-store, and its communication is accomplished through nats.io.

### Input Mapping

The input mapping defines the steps a scheduler must take to complete a build. The required fields for each component are:

- `id`: The id of the service being provisioned.
- `action`: The action of the service
- `edges`: The edges define the directed connection between components, as defined by a `source` and a `destination`.
- `components`: This collection of components represents the _current_ state of the service.
- `changes`: This collection of components represents the _desired_ state of the service.

This output is generated by defintion-mapper with execution ordering and dependencies configured before being sent to scheduler.

An example of the input json to scheduler:

```json
{
    "id": "test-generated-id",
    "action": "service.delete",
    "edges": [
        {
            "destination": "vpc::test-vpc",
            "length": 1,
            "source": "internet_gateway::test-vpc"
        },
        {
            "destination": "vpc::test-vpc",
            "length": 1,
            "source": "firewall::test-vm"
        },
        {
            "destination": "internet_gateway::test-vpc",
            "length": 1,
            "source": "network::test-dmz"
        },
        {
            "destination": "vpc::test-vpc",
            "length": 1,
            "source": "firewall::test-elb"
        },
        {
            "destination": "firewall::test-vm",
            "length": 1,
            "source": "instance::test-web-1"
        },
        {
            "destination": "network::test-dmz",
            "length": 1,
            "source": "instance::test-web-1"
        },
        {
            "destination": "firewall::test-elb",
            "length": 1,
            "source": "elb::test-elb"
        },
        {
            "destination": "network::test-dmz",
            "length": 1,
            "source": "elb::test-elb"
        },
        {
            "destination": "instance::test-web-1",
            "length": 1,
            "source": "elb::test-elb"
        },
        {
            "destination": "end",
            "length": 1,
            "source": "vpc::test-vpc"
        },
        {
            "destination": "elb::test-elb",
            "length": 1,
            "source": "start"
        }
    ],
    "components": [
        {
            "_action": "delete",
            "_component": "vpc",
            "_component_id": "vpc::test-vpc",
            "_provider": "aws",
            "_state": "waiting",
            "auto_remove": true,
            "name": "test-vpc",
            "service": "test-generated-id",
            "subnet": "10.0.0.0/16",
            "tags": {
                "Name": "test-vpc",
                "ernest.service": "test"
            },
            "vpc_aws_id": "vpc-00000000"
        },
        {
            "_action": "delete",
            "_component": "internet_gateway",
            "_component_id": "internet_gateway::test-vpc",
            "_provider": "aws",
            "_state": "waiting",
            "internet_gateway_aws_id": "igw-00000000",
            "name": "test-vpc",
            "service": "test-generated-id",
            "tags": {
                "Name": "test-vpc",
                "ernest.service": "test"
            },
            "vpc": "test-vpc",
            "vpc_id": "vpc-00000000"
        },
        {
            "_action": "none",
            "_component": "credentials",
            "_component_id": "credentials::aws",
            "_provider": "aws",
            "_state": "waiting",
            "name": "test-aws",
            "region": "eu-west-1"
        }
    ],
    "changes": [
        {
            "_action": "delete",
            "_component": "vpc",
            "_component_id": "vpc::test-vpc",
            "_provider": "aws",
            "_state": "waiting",
            "auto_remove": true,
            "name": "test-vpc",
            "service": "test-generated-id",
            "subnet": "10.0.0.0/16",
            "tags": {
                "Name": "test-vpc",
                "ernest.service": "test"
            },
            "vpc_aws_id": "vpc-00000000"
        },
        {
            "_action": "delete",
            "_component": "internet_gateway",
            "_component_id": "internet_gateway::test-vpc",
            "_provider": "aws",
            "_state": "running",
            "internet_gateway_aws_id": "igw-00000000",
            "name": "test-vpc",
            "service": "test-generated-id",
            "tags": {
                "Name": "test-vpc",
                "ernest.service": "test"
            },
            "vpc": "test-vpc",
            "vpc_id": "vpc-00000000"
        },
        {
            "_action": "delete",
            "_component": "firewall",
            "_component_id": "firewall::test-vm",
            "_provider": "aws",
            "_state": "completed",
            "name": "test-vm",
            "network_aws_id": null,
            "rules": {
                "egress": [
                    {
                        "from_port": 0,
                        "ip": "0.0.0.0/0",
                        "protocol": "-1",
                        "to_port": 65535
                    }
                ],
                "ingress": [
                    {
                        "from_port": 0,
                        "ip": "10.0.0.0/16",
                        "protocol": "tcp",
                        "to_port": 65535
                    }
                ]
            },
            "security_group_aws_id": "sg-00000000",
            "service": "test-generated-id",
            "tags": {
                "Name": "test-vm",
                "ernest.service": "test"
            },
            "vpc": "test-vpc",
            "vpc_id": "vpc-00000000"
        },
        {
            "_action": "delete",
            "_component": "network",
            "_component_id": "network::test-dmz",
            "_provider": "aws",
            "_state": "completed",
            "availability_zone": "eu-west-1a",        
            "internet_gateway": "test-vpc",
            "internet_gateway_aws_id": "igw-00000000",
            "is_public": true,
            "name": "test-dmz",
            "network_aws_id": "subnet-00000000",
            "range": "10.0.11.0/24",
            "service": "test-generated-id",
            "tags": {
                "Name": "test-dmz",
                "ernest.service": "test"
            },
            "vpc": "test-vpc",
            "vpc_id": "vpc-00000000"
        },
        {
            "_action": "delete",
            "_component": "firewall",
            "_component_id": "firewall::test-elb",
            "_provider": "aws",
            "_state": "completed",
            "name": "test-elb",
            "network_aws_id": null,
            "rules": {
                "egress": [
                    {
                        "from_port": 0,
                        "ip": "0.0.0.0/0",
                        "protocol": "-1",
                        "to_port": 65535
                    }
                ],
                "ingress": [
                    {
                        "from_port": 0,
                        "ip": "10.0.0.0/16",
                        "protocol": "tcp",
                        "to_port": 65535
                    },
                    {
                        "from_port": 80,
                        "ip": "0.0.0.0/0",
                        "protocol": "tcp",
                        "to_port": 80
                    }
                ]
            },
            "security_group_aws_id": "sg-00000000",
            "service": "test-generated-id",
            "tags": {
                "Name": "test-elb",
                "ernest.service": "test"
            },
            "vpc": "test-vpc",
            "vpc_id": "vpc-00000000"
        },
        {
            "_action": "delete",
            "_component": "instance",
            "_component_id": "instance::test-web-1",
            "_provider": "aws",
            "_state": "completed",
            "assign_elastic_ip": false,    
            "elastic_ip": "",
            "image": "ami-ed82e39e",
            "instance_aws_id": "i-00000000000000000",
            "instance_type": "t2.micro",
            "ip": "10.0.11.11",
            "key_pair": "test",
            "name": "test-web-1",
            "network_aws_id": "subnet-00000000",
            "network_is_public": false,
            "network_name": "test-dmz",
            "public_ip": "52.48.98.107",
            "security_group_aws_ids": [
                "sg-00000000"
            ],
            "security_groups": [
                "test-vm"
            ],
            "service": "test-generated-id",
            "tags": {
                "Name": "test-web-1",
                "ernest.instance_group": "test-web",
                "ernest.service": "test"
            },
            "user_data": "",
            "volumes": null
        },
        {
            "_action": "delete",
            "_component": "elb",
            "_component_id": "elb::test-elb",
            "_provider": "aws",
            "_state": "completed",
            "dns_name": "test-elb-1103626973.eu-west-1.elb.amazonaws.com",
            "instance_aws_ids": [
                "i-00000000000000000"
            ],
            "instance_names": [
                "test-web-1"
            ],
            "instances": [
                "test-web"
            ],
            "is_private": false,
            "listeners": [
                {
                    "from_port": 80,
                    "protocol": "HTTP",
                    "ssl_cert": "",
                    "to_port": 80
                }
            ],
            "name": "test-elb",
            "network_aws_ids": [
                "subnet-00000000"
            ],
            "networks": [
                "test-dmz"
            ],
            "security_group_aws_ids": [
                "sg-00000000"
            ],
            "security_groups": [
                "test-elb"
            ],
            "service": "test-generated-id",
            "tags": {
                "ernest.service": "test"
            }
        }
    ]
}
```

You will find a more up to date documentation on the repo [README](https://github.com/ernestio/scheduler/blob/develop/README.md)


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
