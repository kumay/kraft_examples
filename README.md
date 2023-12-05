# Kraft with SASML_SCRAM, SASL_PLAINTEXT and no security.

In Kraft mode authentication data of SASL_SCRAM is stored in `__cluster_metadata`.

Background Information:

https://cwiki.apache.org/confluence/display/KAFKA/KIP-801

https://cwiki.apache.org/confluence/display/KAFKA/KIP-900


In this example broker and controller coexists in broker container.

Using cp-kafka (confluent community edition) docker image.

https://hub.docker.com/r/confluentinc/cp-kafka


## how to use

1. create ClusterID

```
$ docker run -t confluentinc/cp-kafka:7.5.0 /bin/kafka-storage random-uuid
```

2. replace ClusterID in yaml file.

```
environment:
      CLUSTER_ID: '<CLUSTER_ID goes here!>'
```

3. run cluster

```
$ docker-compose -f <docker-compose>.yaml up
```


### cluster.yaml

this calls `scripts/update_run.sh` before broker start up.

Inter Controller connections are PLAINTEXT (no security).
External connections and Inter Broker connections are PLAINTEXT (no security).


### cluster_sasl_scram.yaml

this calls `scripts/update_scram-sasl-plain_run.sh` before broker start up.

Inter Controller connections are PLAINTEXT connection (no security).
External connections and Inter Broker connections are SASL_PLAINTEXT with SCRAM (SCRAM_SHA-256).


### cluster_sasl_scram_ctr_plain.yaml

this calls `scripts/update_scram_run.sh` before broker start up.

Inter Controller connections are SASL_PLAINTEXT connection.
External connections and Inter Broker connection ares SASL_PLAINTEXT with SCRAM (SCRAM_SHA-256).


## Extra information


### Generate UUID(Cluster ID)

```
$ docker run -t confluentinc/cp-kafka:7.5.0 /bin/kafka-storage random-uuid
```

### Create Topic command

**without authentication**

```
$　docker run -it --network kraft_network --rm confluentinc/cp-kafka:7.5.0 /bin/kafka-topics --bootstrap-server kafka1:19092 --create --topic test
```

**With Scram authentication**

```
$ cat client.config

sasl.mechanism=SCRAM-SHA-256

# Configure SASL_SSL if TLS/SSL encryption is enabled, otherwise configure SASL_PLAINTEXT
security.protocol=SASL_PLAINTEXT

sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required \
  username="admin" \
  password="admin-secret";


$　docker run -it -v ./configs/client.config:/tmp/client.config --network kraft_network --rm confluentinc/cp-kafka:7.5.0 /bin/kafka-topics --bootstrap-server kafka1:39092 --command-config /tmp/client.config --create --topic test-topic
```

**SCRAM authentication setup**

How to use.
```
# KRaft required step: Format the storage directory with a new cluster ID
# Have to add \n not to corrupt /etc/confluent/docker/ensure
# --config CONFIG, -c CONFIG
#                        The Kafka configuration file to use.
# --cluster-id CLUSTER_ID, -t CLUSTER_ID
#                        The cluster ID to use.
# --add-scram ADD_SCRAM, -S ADD_SCRAM
#                        A SCRAM_CREDENTIAL to add to the __cluster_metadata log e.g.
#                        'SCRAM-SHA-256=[user=alice,password=alice-secret]'
#                        'SCRAM-SHA-512=[user=alice,iterations=8192,salt="N3E=",saltedpassword="YCE="]'
# --ignore-formatted, -g
# --release-version RELEASE_VERSION, -r RELEASE_VERSION
#                        A KRaft release version to use for the  initial  metadata version. The minimum is 3.0, the
#                        default is 3.5-IV2
```

Example adding single credential (username:admin, password:admin-secret) to SCRAM.
```
$ kafka-storage format --add-scram 'SCRAM-SHA-256=[name=\"admin\",password=\"admin-secret\"]' --ignore-formatted -t <CLUSTER_ID> -c /etc/kafka/kafka.properties
```

Example adding multiple credencial to SCRAM.
```
$ kafka-storage format \
 --add-scram 'SCRAM-SHA-256=[name=\"admin\",password=\"admin-secret\"]' \
 --add-scram 'SCRAM-SHA-256=[name=\"kafka\",password=\"kafka-secret\"]' \
 --add-scram 'SCRAM-SHA-256=[name=\"connect\",password=\"connect-secret\"]' \
 --add-scram 'SCRAM-SHA-256=[name=\"schemaregistry\",password=\"schemaregistry-secret\"]' \
 --ignore-formatted -t <CLUSTER_ID> -c /etc/kafka/kafka.properties

```


### Memo

Version cp-kafka:7.3.3 did not support --add-scram option. (May work need more work.)
Version cp-kafka:7.5.0 supported --add-scram option


### Links

Authorizer class
for Karaft use StandardAuthorizer class
https://docs.confluent.io/platform/current/kafka/authorization.html#authorization-using-access-control-lists-acls


SCRAM Example
https://github.com/wurstmeister/kafka-docker/wiki/SASL-config-example


Kraft with SASL_SCRAM
https://docs.confluent.io/platform/current/kafka/authentication_sasl/authentication_sasl_scram.html


SCRAM samples
https://forum.confluent.io/t/clusterauthorizationexception-with-user-anonymous-when-setting-up-sasl-with-kraft-and-acl/8865/7
https://github.com/gschmutz/various-platys-platforms/tree/main/kafka-sasl-scram-kraft
https://github.com/TrivadisPF/platys-modern-data-platform