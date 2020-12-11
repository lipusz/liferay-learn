# Configuring CCR: Settings Reference

To help with the CCR configuration process, the example configurations from the step-by-step instructions in this guide are collected here. These configuration are made as generic as possible, but paths, ports, etc. will need to be adjusted to match your environment. In addition, you must perform certain steps manually to complete the installation, so these cannot replace the step-by-step instructions.

The configurations below assume you enable encrypted communications (TLS/SSL) and user authentication through X-Pack Security in your installation.

```tip::
   `Configuration values provided by .config files <../../../system-administration/system-settings/using-configuration-files.md>`__ are propagated throughout a DXP cluster as soon as the file is deployed to a single node. However, it's a best practice to provide identical configurations for each cluster node. 

   You might notice that the instructions in this guide appear to violate the above statement. The CCR specific configuration files are not provided to the remote/leader Liferay DXP nodes. Because the CCR module is not deployed to these nodes, therefore the configurations are inert in these nodes.
```

## Remote DXP Cluster Node Configurations

These configuration files are deployed to `[Remote Liferay Home]/osgi/configs`.

### Remote DXP Cluster Node Configurations for Elasticsearch 7

File name: `com.liferay.portal.search.elasticsearch7.configuration.ElasticsearchConfiguration.config`

File contents:

```properties
productionModeEnabled="true"
networkHostAddresses=["http://localhost:9200"]
logExceptionsOnly="false"
```

The X-Pack security configuration file is not required on Liferay DXP 7.3. Security is configured in the `ElasticsearchConfiguration.config` (for the remote/leader connection) and in each local/follower connection's configuration file.

## Local DXP Cluster Node Configurations

Location: `[Liferay Home]/osgi/configs`

### Local DXP Cluster Node Configurations for Elasticsearch 7

Make sure this is identical to the Elasticsearch 7 configuration on the Remote Liferay DXP node, for both the Elasticsearch 7 configuration and the X-Pack Security configuration.

### Local DXP Cluster Node CCR Module Configurations

File name: `com.liferay.portal.search.elasticsearch.cross.cluster.replication.internal.configuration.CrossClusterReplicationConfiguration.config`

File contents:

```properties
ccrEnabled=B"true"
ccrLocalClusterConnectionConfigurations=["localhost:9080=follower"]
remoteClusterAlias="leader"
```

File name: `com.liferay.portal.search.elasticsearch.cross.cluster.replication.internal.configuration.ElasticsearchConnectionConfiguration-follower.config`

File contents:

```properties
connectionId="follower"
clusterName="LiferayElasticsearchCluster_FOLLOWER"
transportAddresses=["localhost:9301"]
networkHostAddress="https://localhost:9201"
authenticationEnabled=B"true"
username="elastic"
password="liferay"
sslKeyPath="/PATH/TO/ES_FOLLOWER_1/config/certs/elastic-certificates.key"
sslCertificatePath="/PATH/TO/ES_FOLLOWER_1/config/certs/elastic-certificates.crt"
certificateFormat="PEM"
sslCertificateAuthoritiesPaths="/PATH/TO/ES_FOLLOWER_1/config/certs/ca.crt"
transportSSLVerificationMode="certificate"
transportSSLEnabled=B"true"
```

## Leader Elasticsearch Cluster Node Configurations

Location: `ES_LEADER_HOME/config`

File name: `elasticsearch.yml`

File contents:
```yaml
cluster.name: LiferayElasticsearchCluster_LEADER

xpack.security.enabled: true

## TLS/SSL settings for Transport layer
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.key: certs/elastic-certificates.key
xpack.security.transport.ssl.certificate: certs/elastic-certificates.crt
xpack.security.transport.ssl.certificate_authorities : ["certs/ca.crt"]

# TLS/SSL settings for HTTP layer
xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.verification_mode: certificate
xpack.security.http.ssl.key: certs/elastic-certificates.key
xpack.security.http.ssl.certificate: certs/elastic-certificates.crt
xpack.security.http.ssl.certificate_authorities : ["certs/ca.crt"]

# For Kibana
xpack.monitoring.collection.enabled: true

# For CCR setup
http.port: 9200
node.name: es-leader-node-1
```

## Follower Elasticsearch Cluster Node Configurations

Location: `ES_FOLLOWER_HOME/config`

File name: `elasticsearch.yml`

File contents:
```yaml
cluster.name: LiferayElasticsearchCluster_FOLLOWER

xpack.security.enabled: true

## TLS/SSL settings for Transport layer
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.key: certs/elastic-certificates.key
xpack.security.transport.ssl.certificate: certs/elastic-certificates.crt
xpack.security.transport.ssl.certificate_authorities : ["certs/ca.crt"]

# TLS/SSL settings for HTTP layer
xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.verification_mode: certificate
xpack.security.http.ssl.key: certs/elastic-certificates.key
xpack.security.http.ssl.certificate: certs/elastic-certificates.crt
xpack.security.http.ssl.certificate_authorities : ["certs/ca.crt"]

# For Kibana
xpack.monitoring.collection.enabled: true

# For CCR setup
http.port: 9202
node.name: es-follower-node-1
```
