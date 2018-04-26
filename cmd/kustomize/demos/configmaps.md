[overlay]: ../docs/glossary.md#overlay
[target]: ../docs/glossary.md#target

# Demo: combining config data from devops and developers

Scenario: you have a Java-based server storefront in
production that various development teams (signups,
checkout, search, etc.) contribute to.

The server runs in different environments:
_development_, _testing_ and _production_, accepting
configuration parameters from java property files.

The current setup, which we'll speculate uses one big
properties file for each environment, is difficult to
manage, as the files all change frequently, and have to
be changed by devops exclusively because 1) they must
at least partially agree in certain values, and 2)
because the production properties contain sensitive
data like database credentials.

## Property sharding

The properties are seperable into categories like

### common properties

E.g. internationalization data, static data like
physical constants, location of external services, etc.

Things that are the same regardless of environment.

Only one set of values is needed.

### plumbing properties

E.g. serving location of static content (HTML, CSS,
javascript), location of product and customer database
tables, ports expected by load balancers, log sinks,
etc.

The different values for these properties are
precisely what sets the environments apart.

Devops or SRE will want full control over the values
used in production.  Testing will have fixed
databases supporting testing.  Developers will want
to do whatever they want to try scenarios under
development.

### secret properties

E.g. location of actual user tables, database
credentials, decryption keys, etc.

Things that are a subset of devops controls, that
nobody else has (or should want) access to.

## A mixin approach to management

The way to create _n_ cluster environments that share
some common information is to create _n_ overlays of a
common base.

Lets speculate


that have
A cluster environment is created by
running `kustomize build` on a [target] that happens to
be an [overlay].


As with all demos, there will be one overlay for each
environment, and they will patch a common base.

Define a place to work:

<!-- @makeWorkplace @test -->
```
DEMO_HOME=$(mktemp -d)
```


--from-file=src/main/resources/application.properties

First, spec
Make a place to put the base configuration:

<!-- @baseDir @test -->
```
mkdir -p $DEMO_HOME/base
```


<!-- @baseKustomization @test -->
```
cat <<EOF >$DEMO_HOME/base/kustomization.yaml
resources:
- configMap.yaml
EOF
```

<!-- @hey @test -->
```
cat <<EOF >$DEMO_HOME/base/configMap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: the-map
data:
  altGreeting: "Have a pineapple!"
  enableRisky: "true"
EOF
```
dbpassword=password
database=localhost
dbuser=mkyong
