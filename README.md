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
oc -n openshift process postgresql-persistent -p POSTGRESQL_USER=onlyoffice -p POSTGRESQL_PASSWORD=onlyoffice -p POSTGRESQL_DATABASE=onlyoffice | oc -n $PROJECT create -f -
```

### 2 Deploy Redis database

```[bash]
oc -n openshift process postgresql-ephemeral | oc -n $PROJECT create -f -
```

For now you need to remove the authentication password.

* Remove the REDIS_PASSWORD environment variable.
* Modify the readiness check to not use this password.

This can be changed as soon as OnlyOffice DocumentServer v.5.5.0 is released, see <https://github.com/ONLYOFFICE/DocumentServer/issues/353>.

### 3 Deploy RabbitMQ microservice

```[bash]
oc process -f https://raw.githubusercontent.com/jngrb/onlyoffice-communityserver-openshift/master/rabbitmq.yaml | oc -n $PROJECT create -f -
```

### 4 Deploy OnlyOffice DocumentServer

So far, the OnlyOffice DocumentServer container must run as root. Hence, enable this feature for the projects default service account:

```[bash]
oc create sa root-allowed
oc policy add-role-to-user system:deployer -z root-allowed
oc adm policy add-scc-to-user anyuid -z root-allowed
```

Now, we can run the multi-service DocumentServer image (Change `<POSTGRESQL-SECRET>` to the secret with the credentials for the postgresql deploment.).

```[bash]
oc process -f https://raw.githubusercontent.com/jngrb/onlyoffice-communityserver-openshift/master/onlyoffice-communityserver.yaml -p ONLYOFFICE_HOST=onlyoffice.example.com -p POSTGRESQL_SECERT=<POSTGRESQL-SECRET> | oc -n $PROJECT create -f -
```

Wait for the POD to start and run through all initialization steps. This may take a while.

## Open issues

* do not require root user to run the container
* seperate the all-in-one container into subservices
* what about a data-container (`onlyoffice-communityserver.alt.yaml`, inspired by official OnlyOffice docker-compose file, not working yet)

(See also the respective issues in the OnlyOffice DocumentServer repository.)

## Dependency on OnlyOffice DocumentServer Community Edition

This OpenShift template uses software provided by the OnlyOffice team, specially the official OnlyOffice DocumentServer docker image (see references below).

These components are licenced under AGPL-3.0 with their copyright belonging to the OnlyOffice team. Also the ONLYOFFICE trademark and logo belong to OnlyOffice.

References:

* <https://www.onlyoffice.com/de/download.aspx>
* <https://github.com/ONLYOFFICE/Docker-DocumentServer>
* <https://hub.docker.com/r/onlyoffice/documentserver/>
* <http://www.gnu.org/licenses/agpl-3.0.html>
