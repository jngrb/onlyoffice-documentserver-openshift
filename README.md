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
