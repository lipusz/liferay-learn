# CCR Basic Use Case Config Reference

To help you configuring CCR, we have collected here the example configurations we use in the [Basic Use Case setup guide](./configuring-ccr-a-basic-use-case.md). We tried to make them _generic_ as much as possible, however paths, ports etc. may still need to be adjusted to match your environment. In addition, you still need to perform certain steps manually to complete the installation. This is really just the collection of the configuration files.

> The configurations below assume you enable encrypted communications (TLS/SSL) and user authentication through X-Pack Security in your installation.

## Remote DXP Cluster Node Configs

Location: `[Liferay Home]/osgi/configs`

### Remote DXP Cluster Node Configs for Elasticsearch 6

File: `com.liferay.portal.search.elasticsearch6.configuration.ElasticsearchConfiguration.config`

Content:
```properties
clusterName = "LiferayElasticsearchCluster_LEADER"
operationMode = "REMOTE"
transportAddresses = ["localhost:9300"]
additionalIndexConfigurations = "index.soft_deletes.enabled: true"
logExceptionsOnly = B"false"
```

File: `com.liferay.portal.search.elasticsearch6.xpack.security.internal.configuration.XPackSecurityConfiguration.config`

Content:
```properties
requiresAuthentication=B"true"
username = "elastic"
password = "liferay"
sslKeyPath="/PATH/TO/ES_LEADER_1/config/certs/elastic-certificates.key"
sslCertificatePath="/PATH/TO/ES_LEADER_1/config/certs/elastic-certificates.crt"
certificateFormat="PEM"
sslCertificateAuthoritiesPaths="/PATH/TO/ES_LEADER_1/config/certs/ca.crt"
transportSSLVerificationMode="certificate"
transportSSLEnabled=B"true"
```

### Remote DXP Cluster Node Configs for Elasticsearch 7

File: `com.liferay.portal.bundle.blacklist.internal.BundleBlacklistConfiguration.config`

Content:
```properties
blacklistBundleSymbolicNames=[ \
	"com.liferay.portal.search.elasticsearch6.api", \
	"com.liferay.portal.search.elasticsearch6.impl", \
	"com.liferay.portal.search.elasticsearch6.spi", \
	"com.liferay.portal.search.elasticsearch6.xpack.security.impl", \
	"Liferay Connector to X-Pack Security [Elastic Stack 6.x] - Impl" \
]
```

File: `com.liferay.portal.search.elasticsearch7.configuration.ElasticsearchConfiguration.config`

Content:
```properties
operationMode="REMOTE"
logExceptionsOnly = B"false"
```

File: `com.liferay.portal.search.elasticsearch7.configuration.XPackSecurityConfiguration.config`

Content:
```properties
requiresAuthentication=B"true"
username = "elastic"
password = "liferay"
sslKeyPath="/PATH/TO/ES_LEADER_1/config/certs/elastic-certificates.key"
sslCertificatePath="/PATH/TO/ES_LEADER_1/config/certs/elastic-certificates.crt"
certificateFormat="PEM"
sslCertificateAuthoritiesPaths="/PATH/TO/ES_LEADER_1/config/certs/ca.crt"
transportSSLVerificationMode="certificate"
transportSSLEnabled=B"true"
```

## Local DXP Cluster Node Configs

Location: `[Liferay Home]/osgi/configs`

### Local DXP Cluster Node Configs for Elasticsearch 6

File: `com.liferay.portal.search.elasticsearch6.configuration.ElasticsearchConfiguration.config`

Content:
```properties
clusterName = "LiferayElasticsearchCluster_LEADER"
operationMode = "REMOTE"
transportAddresses = ["localhost:9300"]
logExceptionsOnly = B"false"
```

File: `com.liferay.portal.search.elasticsearch6.xpack.security.internal.configuration.XPackSecurityConfiguration.config`

Content:
```properties
requiresAuthentication=B"true"
username = "elastic"
password = "liferay"
sslKeyPath="/PATH/TO/ES_LEADER_1/config/certs/elastic-certificates.key"
sslCertificatePath="/PATH/TO/ES_LEADER_1/config/certs/elastic-certificates.crt"
certificateFormat="PEM"
sslCertificateAuthoritiesPaths="/PATH/TO/ES_LEADER_1/config/certs/ca.crt"
transportSSLVerificationMode="certificate"
transportSSLEnabled=B"true"
```

### Local DXP Cluster Node Configs for Elasticsearch 7

Same as the Elasticsearch 7 configs for the Remote DXP node.

### Local DXP Cluster Node CCR Module Configs

File: `com.liferay.portal.search.elasticsearch.cross.cluster.replication.internal.configuration.CrossClusterReplicationConfiguration.config`

Content:
```properties
ccrEnabled = B"true"
ccrLocalClusterConnectionConfigurations = ["localhost:9080=follower"]
remoteClusterAlias = "leader"
```

File: `com.liferay.portal.search.elasticsearch.cross.cluster.replication.internal.configuration.ElasticsearchConnectionConfiguration-follower.config`

Content:
```properties
connectionId = "follower"
clusterName = "LiferayElasticsearchCluster_FOLLOWER"
transportAddresses = ["localhost:9301"]
networkHostAddress = "https://localhost:9201"
authenticationEnabled = B"true"
username = "elastic"
password = "liferay"
sslKeyPath = "/PATH/TO/ES_FOLLOWER_1/config/certs/elastic-certificates.key"
sslCertificatePath = "/PATH/TO/ES_FOLLOWER_1/config/certs/elastic-certificates.crt"
certificateFormat = "PEM"
sslCertificateAuthoritiesPaths = "/PATH/TO/ES_FOLLOWER_1/config/certs/ca.crt"
transportSSLVerificationMode = "certificate"
transportSSLEnabled = B"true"
```

## Leader Elasticsearch Cluster Node Configs

Location: `ES_LEADER_HOME/config`

File: `elasticsearch.yml`

Content:
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
transport.port: 9300
```

## Follower Elasticsearch Cluster Node Configs

Location: `ES_FOLLOWER_HOME/config`

File: `elasticsearch.yml`

Content:
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
http.port: 9201
node.name: es-follower-node-1
transport.port: 9301
```
