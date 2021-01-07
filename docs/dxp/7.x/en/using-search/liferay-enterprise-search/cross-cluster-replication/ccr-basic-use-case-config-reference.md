# Configuring CCR: Settings Reference

To help with the CCR configuration process, the example configurations from the step-by-step instructions in this guide are collected here. These configuration are made as generic as possible, but paths, ports, etc. will need to be adjusted to match your environment. In addition, you must perform certain steps manually to complete the installation, so these cannot replace the step-by-step instructions.

The configurations below assume you enable encrypted communications (TLS/SSL) and user authentication through X-Pack Security in your installation. See [Securing Elasticsearch](../../installing-and-upgrading-a-search-engine/elasticsearch/securing-elasticsearch.md) for more information.

```tip::
   `Configuration values provided by .config files <../../../system-administration/system-settings/using-configuration-files.md>`__ are propagated throughout a DXP cluster as soon as the file is deployed to a single node. However, it's a best practice to provide identical configurations for each cluster node. 
```

## Remote DXP Cluster Node Configurations

These configuration files are deployed to `[Remote Liferay Home]/osgi/configs`.

The Remote DXP cluster needs three configuration files.

File name: `com.liferay.portal.search.elasticsearch7.configuration.ElasticsearchConfiguration.config`

File contents:

```properties
productionModeEnabled="true"
remoteClusterConnectionId="remote"
logExceptionsOnly="false"
```

```tip::
   The X-Pack security configuration file is not required on Liferay DXP 7.3. Security is configured in each connection's ``.config`` file. The values should be identical on all Liferay nodes.
```

File name: `com.liferay.portal.search.elasticsearch7.configuration.ElasticsearchConnectionConfiguration-remote.config`

File contents:

```properties
active="true"
connectionId="remote"
networkHostAddresses=["https://localhost:9200"]
username="elastic"
password="liferay"
authenticationEnabled=B"true"
httpSSLEnabled=B"true"
truststorePassword="liferay"
truststorePath="/PATH/to/elastic-nodes.p12"
truststoreType="pkcs12"
```

File name: `com.liferay.portal.search.elasticsearch7.configuration.ElasticsearchConnectionConfiguration-ccr.config`

File contents:

```properties
active="true"
connectionId="ccr"
networkHostAddresses=["https://localhost:9202"]
username="elastic"
password="liferay"
authenticationEnabled=B"true"
httpSSLEnabled=B"true"
truststorePassword="liferay"
truststorePath="/PATH/to/elastic-nodes.p12"
truststoreType="pkcs12"
```

## Local DXP Cluster Node Configurations

These configuration files are deployed to `[Local Liferay Home]/osgi/configs`.

Provide identical Elasticsearch 7 configurations (including the connection configuration files) to the local DXP nodes that you provided to the remote DXP nodes.

### Local DXP Cluster Node CCR Module Configurations

Configure the LES Cross-Cluster Configuration application from the System Settings UI on any node (even one in the remote data center). If the LPKG is deployed, the configuration entry is in System Settings &rarr; Search &rarr; Cross-Cluster Replication.

Set these values:

* The _Enabled_ checkbox is checked
* The Cross-Cluster Replication Local Cluster Connection Configuration has the value _localhost:9080,ccr_

![Configure CCR from System Settings.](./ccr-basic-use-case-config-reference/images/01.png)

```important::
   Never set the value to the remote data center here (in the example, it would be ``localhost:8080,remote``). Setting this would cause follower indexes to be created in the remote cluster, where leader indexes of the same name already reside.
```

## Leader Elasticsearch Cluster Node Configurations

Location: `ES_LEADER_HOME/config`

File name: `elasticsearch.yml`

File contents:
```yaml
cluster.name: LiferayElasticsearchCluster_LEADER
node.name: es-leader-node-1

http.port: 9200

xpack.security.enabled: true

### TLS/SSL settings for Transport layer
xpack.security.transport.ssl.enabled: true

# PKCS#12
xpack.security.transport.ssl.keystore.path: certs/elastic-nodes.p12
xpack.security.transport.ssl.keystore.password: liferay
xpack.security.transport.ssl.truststore.path: certs/elastic-nodes.p12
xpack.security.transport.ssl.truststore.password: liferay
xpack.security.transport.ssl.verification_mode: certificate

## TLS/SSL settings for HTTP layer
xpack.security.http.ssl.enabled: true

# PKCS#12
xpack.security.http.ssl.keystore.path: certs/elastic-nodes.p12
xpack.security.http.ssl.keystore.password: liferay
xpack.security.http.ssl.truststore.path: certs/elastic-nodes.p12
xpack.security.http.ssl.truststore.password: liferay
xpack.security.http.ssl.verification_mode: certificate

# For Kibana
xpack.monitoring.collection.enabled: true
```

## Follower Elasticsearch Cluster Node Configurations

Location: `ES_FOLLOWER_HOME/config`

File name: `elasticsearch.yml`

File contents:

```yaml
cluster.name: LiferayElasticsearchCluster_FOLLOWER
node.name: es-follower-node-1

http.port: 9202
transport.port: 9301

xpack.security.enabled: true

### TLS/SSL settings for Transport layer
xpack.security.transport.ssl.enabled: true

# PKCS#12
xpack.security.transport.ssl.keystore.path: certs/elastic-nodes.p12
xpack.security.transport.ssl.keystore.password: liferay
xpack.security.transport.ssl.truststore.path: certs/elastic-nodes.p12
xpack.security.transport.ssl.truststore.password: liferay
xpack.security.transport.ssl.verification_mode: certificate

## TLS/SSL settings for HTTP layer
xpack.security.http.ssl.enabled: true

# PKCS#12
xpack.security.http.ssl.keystore.path: certs/elastic-nodes.p12
xpack.security.http.ssl.keystore.password: liferay
xpack.security.http.ssl.truststore.path: certs/elastic-nodes.p12
xpack.security.http.ssl.truststore.password: liferay
xpack.security.http.ssl.verification_mode: certificate

# For Kibana
xpack.monitoring.collection.enabled: true
```
