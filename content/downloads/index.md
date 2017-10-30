---
date: 2016-03-09T20:08:11+01:00
title: Downloads
weight: 20
---

## Get Ernest
Below are instructions for installing the latest stable release of Ernest ([Release Notes](https://github.com/ernestio/ernest/releases)).

Requirements: [Docker](https://docs.docker.com/engine/installation/), [Docker Compose](https://docs.docker.com/compose/install/), [OpenSSL](https://www.openssl.org).

### Community Edition

1. Clone the repo containing the setup file: `git clone https://github.com/ernestio/ernest.git`

2. Change to the cloned directory: `cd ernest`

3. Run the setup script: `./setup`

4. Enter the hostname, admin username, and admin password when prompted.

Once the install is finished Ernest will be available on IP of the Docker host.

### Enterprise Edition

1. Clone the repo containing the setup file: `git clone https://github.com/ernestio/ernest.git`

2. Change to the cloned directory: `cd ernest`

3. Copy the Enterprise Edition license file here: `cp /tmp/license.txt .`

4. Modify `config/config-store/config.json` from this:

	```
	{
	  "postgres": {
	    "names": [
	      "users",
	      "clients",
	      "datacenters",
	      "services"
	    ],
	    "user": "",
	    "password": "",
	    "url": "postgres://postgres@postgres"
	  },
	  "logstash": {
	    "hostname": "ernest.local",
	    "port": 5000,
	    "timeout": 50000
	  },
	  "monitor": {
	    "host": "",
	    "port": "22000"
	  },
	  "logger": {
	    "host": "",
	    "port": "22001"
	  },
	  "authenticator": {
	    "providers": [
	      {
	        "type": "local"
	      }
	    ]
	  }
	}
	```

	to this

	```
	{
	  "postgres": {
	    "names": [
	      "users",
	      "clients",
	      "datacenters",
	      "services"
	    ],
	    "user": "",
	    "password": "",
	    "url": "postgres://postgres@postgres"
	  },
	  "logstash": {
	    "hostname": "ernest.local",
	    "port": 5000,
	    "timeout": 50000
	  },
	  "monitor": {
	    "host": "",
	    "port": "22000"
	  },
	  "logger": {
	    "host": "",
	    "port": "22001"
	  },
	  "authenticator": {
	    "providers": [
	      {
	        "type": "local"
	      },
	      {
	        "type": "federation",
	        "config": {
	          "url": "https://federation_services_endpoint",
	          "scope": "http://ernest_fqdn",
	          "domain": "your_active_directory"
	        }
	      }
	    ]
	  }
	}
	```

5. Run the setup script: `./setup`

6. Enter the hostname, admin username, and admin password when prompted.

Once the install is finished Ernest will be available on IP of the Docker host.

## Get Ernest CLI
The Ernest CLI is distributed as a binary package for all supported platforms and architectures. To install Ernest CLI, find the newest version of the [appropriate package](https://github.com/ErnestIO/ernest-cli/releases) for your system and download it.

After downloading Ernest CLI, unzip it and move the binary to a directory that is on the PATH. See [this page](http://stackoverflow.com/questions/14637979/how-to-permanently-set-path-on-linux) for instructions on setting the PATH on Linux and Mac. See [this](http://stackoverflow.com/questions/1618280/where-can-i-set-path-to-make-exe-on-windows) page for instructions on setting the PATH in Windows.

## Upgrade Ernest

### From Community Edition to Enterprise Edition

1. Copy the Enterprise Edition license file to the Ernest directory: `cp /tmp/license.txt .`

2. Run the upgrade script: `./upgrade`

3. Modify `config/config-store/config.json` from this:

	```
	{
	  "postgres": {
	    "names": [
	      "users",
	      "clients",
	      "datacenters",
	      "services"
	    ],
	    "user": "",
	    "password": "",
	    "url": "postgres://postgres@postgres"
	  },
	  "logstash": {
	    "hostname": "ernest.local",
	    "port": 5000,
	    "timeout": 50000
	  },
	  "monitor": {
	    "host": "",
	    "port": "22000"
	  },
	  "logger": {
	    "host": "",
	    "port": "22001"
	  },
	  "authenticator": {
	    "providers": [
	      {
	        "type": "local"
	      }
	    ]
	  }
	}
	```

	to this

	```
	{
	  "postgres": {
	    "names": [
	      "users",
	      "clients",
	      "datacenters",
	      "services"
	    ],
	    "user": "",
	    "password": "",
	    "url": "postgres://postgres@postgres"
	  },
	  "logstash": {
	    "hostname": "ernest.local",
	    "port": 5000,
	    "timeout": 50000
	  },
	  "monitor": {
	    "host": "",
	    "port": "22000"
	  },
	  "logger": {
	    "host": "",
	    "port": "22001"
	  },
	  "authenticator": {
	    "providers": [
	      {
	        "type": "local"
	      },
	      {
	        "type": "federation",
	        "config": {
	          "url": "https://federation_services_endpoint",
	          "scope": "http://ernest_fqdn",
	          "domain": "your_active_directory"
	        }
	      }
	    ]
	  }
	}
	```

4. Restart the authenticator and federation services: `docker-compose restart authenticator federation`

### From 3.x.x to 3.y.y

1. Run the upgrade script: `./upgrade`

### From 2.2.0 to 3.0.0

> ***Ernest 3.0.0 introduces backwards incompatible changes to authorisation and the internal naming structure for environments. Therefore it is necessary to re-import your existing environments into Ernest once you have upgraded to 3.0.0.***

1. Run the setup script: `./setup`

2. Unregister each environment from Ernest: `ernest env delete <project> <environment> --force`

3. Import each environment into Ernest: `ernest env import <project> <environment>`

### From 2.x.x to 2.y.y

1. Run the upgrade script: `./upgrade`

### From 1.12.0 to 2.0.0

> ***Ernest 2.0.0 introduces backwards incompatible changes to the internal naming structure for AWS environments. Therefore it is necessary to re-import your existing environments into Ernest once you have upgraded to 2.0.0.***

1. Shutdown Ernest: `docker-compose down`

2. Pull down the latest version of Ernest: `git checkout docker-compose.yml && git checkout config/nginx/ernest.local && git checkout master && git pull`

3. Run the setup script: `./setup`

4. Reapply the datacenter credentials: `ernest datacenter update aws --access_key_id <access_key_id> --secret_access_key <secret_access_key> <dc_name>`

5. For internet gateways update the 'Name' tag to match the 'Name' tag of the VPC the gateway is attached to. Also add the 'ernest.service' tag to the gateway with a value equal to the service name.

6. For instances prepend the 'ernest.instance_group' tag with `<dc_name>-<service_name>-`.

7. For volumes prepend the 'ernest.volume_group' tag with `<dc_name>-<service_name>-`.

8. Unregister each service from Ernest: `ernest service destroy <service_name> --force`

9. Import each service into Ernest: `ernest service import <dc_name> <service_name>`

### From 1.x.x to 1.y.y

1. Shutdown Ernest: `docker-compose down`

2. Pull down the latest version of Ernest: `git checkout docker-compose.yml && git checkout config/nginx/ernest.local && git checkout master && git pull`

3. Run the setup script: `./setup`