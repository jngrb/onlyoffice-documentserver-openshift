# OnlyOffice DocumentServer for OpenShift 3

OpenShift Template for the OnlyOffice DocumentServer (Community Edition).

## Installation

### 0 Preparations

Clone this repository and open a terminal in the sandbox.

```[bash]
git clone https://github.com/jngrb/onlyoffice-documentserver-openshift.git
cd onlyoffice-documentserver-openshift
```

Login into OpenShift `oc login ...`.

Create an OpenShift project for OnlyOffice.

```[bash]
PROJECT=onlyoffice
oc new-project $PROJECT
```

### 1 Deploy PostgresSQL database

```[bash]
oc project $PROJECT
oc -n openshift process postgresql-persistent -p POSTGRESQL_DATABASE=onlyoffice | oc create -f -
```

(If you want to keep things simple for testing, use -p POSTGRESQL_USER=onlyoffice -p POSTGRESQL_PASSWORD=onlyoffice.)

### 2 Deploy Redis database

```[bash]
oc -n openshift process redis-ephemeral | oc create -f -
```

If the OnlyOffice pod is to be deployed only on selected nodes, apply the node selector also to the Redis deployment (here, we use the node selector 'appclass=main'):

```[bash]
oc project $PROJECT
oc patch dc redis --patch='{"spec":{"template":{"spec":{"nodeSelector":{"appclass":"main"}}}}}'
```

For now you need to remove the authentication password.

* Remove the REDIS_PASSWORD environment variable.
* Modify the readiness check to not use this password (remove `-a $REDIS_PASSWORD` from the command line arguments).

A fix of this for OnlyOffice DocumentServer v5.5.0 is no longer relevant, since redis was removed as dependency for DocumentServer Community Edition v5.6.0+, see <https://github.com/ONLYOFFICE/DocumentServer/issues/353>.

### 3 Deploy RabbitMQ microservice

```[bash]
oc process -f https://raw.githubusercontent.com/jngrb/onlyoffice-documentserver-openshift/master/rabbitmq.yaml | oc -n $PROJECT create -f -
```

Also apply the node selector for this deployment.

### 4 Deploy OnlyOffice DocumentServer

So far, the OnlyOffice DocumentServer container must run as root. Hence, enable this feature for the projects default service account:

```[bash]
oc create sa root-allowed
oc policy add-role-to-user system:deployer -z root-allowed
oc adm policy add-scc-to-user anyuid -z root-allowed
```

Now, we can run the multi-service DocumentServer image (Change `<POSTGRESQL-SECRET>` to the secret with the credentials for the postgresql deployment, usually 'postgresql'.).

```[bash]
oc process -f https://raw.githubusercontent.com/jngrb/onlyoffice-documentserver-openshift/master/onlyoffice-documentserver.yaml -p ONLYOFFICE_HOST=onlyoffice.example.com -p POSTGRESQL_SECRET=<POSTGRESQL-SECRET> -p APPCLASS=main | oc -n $PROJECT create -f -
```

Wait for the POD to start and run through all initialization steps. This may take a while.

## Open issues

* do not require root user to run the container
* seperate the all-in-one container into subservices
* what about a data-container (see 'testing' template `onlyoffice-documentserver.alt.yaml`, inspired by official OnlyOffice docker-compose file, not working yet)

(See also the respective issues in the OnlyOffice DocumentServer repository.)

## Dependency on OnlyOffice DocumentServer Community Edition

This OpenShift template uses software provided by the OnlyOffice team, specially the official OnlyOffice DocumentServer docker image (see references below).

These components are licenced under AGPL-3.0 with their copyright belonging to the OnlyOffice team. Also the ONLYOFFICE trademark and logo belong to OnlyOffice.

References:

* <https://www.onlyoffice.com/de/download.aspx>
* <https://github.com/ONLYOFFICE/Docker-DocumentServer>
* <https://hub.docker.com/r/onlyoffice/documentserver/>
* <http://www.gnu.org/licenses/agpl-3.0.html>

## License for the OpenShift template

For compatibility with the OnlyOffice software components, that this template depends on, this work is published under the same license, the AGPL-3.0.

Copyright (C) 2020, Jan Grieb

> This program is free software: you can redistribute it and/or modify
> it under the terms of the GNU Affero General Public License as published by
> the Free Software Foundation, either version 3 of the License, or
> (at your option) any later version.
>
> This program is distributed in the hope that it will be useful,
> but WITHOUT ANY WARRANTY; without even the implied warranty of
> MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
> GNU Affero General Public License for more details.
>
> You should have received a copy of the GNU Affero General Public License
> along with this program.  The license is location in the file `LICENSE`
> in this repository. Also, see the public document on
> <http://www.gnu.org/licenses/>.

## Contributions

Very welcome!

1. Fork it (<https://github.com/jngrb/openproject-openshift/fork>)
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request
