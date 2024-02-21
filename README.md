# lsdmesp-confluent
LSD Managed Event Streaming Platform (MESP) Charts for Confluent (CfK)

## Helm setup

Prepare for offline install

```
helm dependency update .
```

## Deploy LSDMESP with Confluent for Kubernetes

Create the namespaces
```
kubectl create ns lsdmesp-confluent
```

Set PROJECT_HOME env var to project directory
```
PROJECT_HOME=$PWD
```

Create certs
```
openssl genrsa -out $PROJECT_HOME/certs/ca-key.pem 2048
```

```
openssl req -new -key $PROJECT_HOME/certs/ca-key.pem -x509 \
  -days 3650 \
  -out $PROJECT_HOME/certs/ca.pem \
  -subj "/C=US/ST=CA/L=MountainView/O=Confluent/OU=Operator/CN=LocalCA"
```

```
kubectl create secret tls ca-pair-sslcerts \
  --cert=$PROJECT_HOME/certs/ca.pem \
  --key=$PROJECT_HOME/certs/ca-key.pem -n lsdmesp-confluent
```

### Deploy:

`(Optional)` Deploy OpenLDAP for RBAC (if no external LDAP server is available):
```
LDAP_CHART_HOME=$PROJECT_HOME/assets/openldap
helm upgrade --install -f $LDAP_CHART_HOME/ldaps-rbac.yaml ldap-dev $LDAP_CHART_HOME --namespace lsdmesp-confluent
```

### [RBAC] Create MDS secrets for LDAP


### [RBAC] Create credential secrets for Confluent components

connect-login.txt
controlcenter-login.txt
kafka-login.txt
kafkarestclass-login.txt
kafkarestproxy-login.txt
ksqldb-login.txt


### Deploy LSDMESP:
```
helm install lsdmesp-confluent . -f values.yaml -n lsdmesp-confluent
```

## Uninstall LSDMESP

Tear down:
```
helm uninstall lsdmesp-confluent -n lsdmesp-confluent
kubectl patch controlcenter controlcenter -p '{"metadata":{"finalizers":[]}}' --type merge; kubectl patch kafkarestproxy kafkarestproxy -p '{"metadata":{"finalizers":[]}}' --type merge; kubectl patch connect connect -p '{"metadata":{"finalizers":[]}}' --type merge; kubectl patch ksqldb ksqldb -p '{"metadata":{"finalizers":[]}}' --type merge; kubectl patch schemaregistry schemaregistry -p '{"metadata":{"finalizers":[]}}' --type merge; kubectl patch kafka kafka -p '{"metadata":{"finalizers":[]}}' --type merge; kubectl patch kraftcontroller kraftcontroller -p '{"metadata":{"finalizers":[]}}' --type merge
kubectl -n lsdmesp-confluent delete secret ca-pair-sslcerts
for crd in $(kubectl get crd --no-headers -ojsonpath='{.items[*].metadata.name}' | grep confluent); do kubectl delete crd $crd; done
kubectl delete ns lsdmesp-confluent
```