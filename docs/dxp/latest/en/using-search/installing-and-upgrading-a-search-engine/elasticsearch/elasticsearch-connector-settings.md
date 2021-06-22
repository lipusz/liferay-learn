# Elasticsearch Connection Settings

Elasticsearch is Liferay's default search engine. The connection to Elasticsearch is primarily defined in the Elasticsearch 6/7 configuration entry in System Settings (or via corresponding configuration file). Liferay 7.3 introduced the possibility to define multiple connections to Elasticsearch, through the [factory configuration](../../../system-administration/configuring-liferay/configuration-files-and-factories/using-factory-configuration.md) Elasticsearch Connections configuration. Both configuration entries are configurable through [System Settings](../../../system-administration/configuring-liferay/system-settings.md) or an [OSGi configuration file](../../../system-administration/configuring-liferay/configuration-files-and-factories/using-configuration-files.md). 

<!-- If we like these tables, they'll require reorganization -->
**Configuration Files and System Settings Entries:**

| Connecting Servers | <div style="width:380px">System Settings Entry/Configuration File</div> |
| --------------- | -------------------------------------- | 
| Liferay 7.2.x<br />Elasticsearch 6.x           | Elasticsearch 6<br />`com.liferay.portal.search.elasticsearch6.configuration.ElasticsearchConfiguration.config` |
| Liferay 7.2.x<br />Elasticsearch 7.x           | Elasticsearch 7<br />`com.liferay.portal.search.elasticsearch7.configuration.ElasticsearchConfiguration.config` |
| Liferay 7.3.x<br />Elasticsearch 7.x           | Elasticsearch 7<br />`com.liferay.portal.search.elasticsearch7.configuration.ElasticsearchConfiguration.config`<br /><br />Elasticsearch Connections (factory)<br />`com.liferay.portal.search.elasticsearch7.configuration.ElasticsearchConnectionConfiguration-[key].config` |

**Configuration Properties Overview:**

| System Settings Field | <div style="width:280px">Configuration File Field**<br />(default value) | Liferay/Elasticsearch Version Details |
| ----------------------------------- | -------------------------------------------------- | ------------------------------------- | 
| | GENERAL CONNECTION SETTINGS |
| Node Name | `nodeName=` | Liferay 7.3+ |
| Track Total Hits | `trackTotalHits="true"` | Liferay 7.2+ |
| Production Mode Enabled | `productionModeEnabled="false"` | Liferay 7.3+ |
| Index Name Prefix | `indexNamePrefix=liferay-` | Liferay 7.2+ |
| 7.3.x&rarr;Number of Company and System Index Replicas<br />7.2.x&rarr;Index Number of Replicas | `indexNumberOfReplicas=""` | Liferay 7.2+ |
| 7.3.x&rarr;Number of Company and System Index Shards<br />7.2.x&rarr;Index Number of Shards | `indexNumberOfShards=""` | Liferay 7.2+ |
| Log Exceptions Only | `logExceptionsOnly="true"` | Liferay 7.2+ |
| Retry On Conflict | `retryOnConflict="5"` | Liferay 7.2+ |
| | *SECURITY SETTINGS* |
| Authentication Enabled | `authenticationEnabled="false"` | Liferay 7.3+ |
| Username | `username="elastic"` | Liferay 7.3+ |
| Password | `password=` | Liferay 7.3+ |
| Http SSL Enabled | `httpSSLEnabled="false"` | Liferay 7.3+ |
| Truststore Type | `truststoreType="pkcs12"` | Liferay 7.3+ |
| Truststore Path | `truststorePath="/path/to/localhost.p12"` | Liferay 7.3+ |
| Truststore Password | `truststorePassword=` | Liferay 7.3+ |
| | *ELASTICSEARCH CONNECTIONS SETTINGS* |
| Active | `active="false"` | Liferay 7.3+ |
| Connection ID | `connectionId=` | Liferay 7.3+ |
| | *CONNECTION DECLARATION SETTINGS* |
| Remote Cluster Connection ID | `remoteClusterConnectionId=` | Liferay 7.3+ if using the `ElasticsearchConnectionConfiguration` |
| | *REST CLIENT SETTINGS* |
| Network Host Addresses | `networkHostAddresses=http://localhost:9200` | Liferay 7.3+ |
| REST Client Logger Level | `RESTClientLoggerLevel="ERROR"` | Liferay 7.3+ |
| | *TRANSPORT CLIENT SETTINGS* |
| Cluster Name | `clusterName=LiferayElasticsearchCluster` | Liferay 7.2-<br />On 7.3+, applies only to development mode |
| Transport Addresses | `transportAddresses=localhost:9300` | Liferay 7.2- |
| Client Transport Sniff | `clientTransportSniff=true` | Liferay 7.2- |
| Client Transport Ignore Cluster Name | `clientTransportIgnoreClusterName=false` | Liferay 7.2- |
| Client Transport Ping Timeout | `clientTransportPingTimeout=` | Liferay 7.2- |
| Client Transport Nodes Sampler Interval | `clientTransportNodesSamplerInterval=` | Liferay 7.2- |
| | *DEVELOPMENT MODE SETTINGS* | 
| Additional Configurations | `additionalConfigurations=` | Liferay 7.2+ |
| Bootstrap Mlock All | `bootstrapMlockAll="false"` | Liferay 7.2+ |
| Http CORS Allow Origin | `httpCORSAllowOrigin="/https?:\\/\\/localhost(:[0-9]+)?/"` | Liferay 7.2+ |
| Http CORS Configurations | `httpCORSConfigurations=` | Liferay 7.2+ |
| Http CORS Enabled | `httpCORSEnabled="true"` | Liferay 7.2+ |
| Network Host | `networkHost=""` | Liferay 7.2+  |
| Network Bind Host | `networkBindHost=""` | Liferay 7.2+ |
| Network Publish Host | `networkPublishHost=""` | Liferay 7.2+ |
| Sidecar Debug | `sidecarDebug="false"` | Liferay 7.3+ |
| Sidecar Debug Settings | `sidecarDebugSettings="-agentlib:jdwp=transport=dt_socket,address=8001,server=y,suspend=y,quiet=y"` | Liferay 7.3+ |
| Sidecar Heartbeat Interval | `sidecarHeartbeatInterval="10000"` | Liferay 7.3+ |
| Sidecar Home | `sidecarHome="elasticsearch7"` | Liferay 7.3+ |
| Sidecar HTTP Port | `sidecarHttpPort="9200"` | Liferay 7.3+ |
| Sidecar JVM Options | `sidecarJVMOptions="-Xms1g|-Xmx1g|-XX:+AlwaysPreTouch"` | Liferay 7.3+ |
| Sidecar Shutdown Timeout | `sidecarShutdownTimeout="10000"` | Liferay 7.3+ |
| Transport Tcp Port | `transportTcpPort=""` | Liferay 7.2+ |
| Zen Discovery Unicast Hosts Port | `discoveryZenPingUnicastHostsPort="9300-9400"` | Liferay 7.2+ |
| | ADVANCED CONFIGURATION |
| Additional Index Configurations | `additionalIndexConfigurations=` | Liferay 7.2+ |
| Additional Type Mappings | `additionalTypeMappings=` | Liferay 7.2+ |
| Override Type Mappings | `overrideTypeMappings=` | Liferay 7.2+ |
| Proxy Host | `proxyHost=` | Liferay 7.3+ |
| Proxy Port | `proxyPort="0"` | Liferay 7.3+ |
| Proxy Username | `proxyUserName=` | Liferay 7.3+ |
| Proxy Password | `proxyPassword=` | Liferay 7.3+ |
| | *DEPRECATED* |
| Operation Mode | `operationMode="EMBEDDED"` | Deprecated in Liferay 7.3.x |
| Embedded HTTP Port | `embeddedHttpPort="9201"` | Deprecated in Liferay 7.3.x |
| Http Enabled | `httpEnabled="true"` | Deprecated in Liferay 7.1.x<br />Deprecated Elasticsearch 6.3.x |

## Elasticsearch Connection Configuration Files

In Liferay 7.3 and beyond (or on earlier versions with [LES Cross-Cluster Replication](../../liferay-enterprise-search/cross-cluster-replication/cross-cluster-replication.md)), there's an additional connection configuration entry, Elasticsearch Connections. You can use this to define any connection to Elasticsearch, but if you are only configuring one connection you can still use the Elasticsearch 7 configuration entry.

If managing only one connection for Elasticsearch 7 the configuration file is

```
com.liferay.portal.search.elasticsearch7.configuration.ElasticsearchConfiguration.config
```

If using multiple connections, define connections with files named accordingly:

```
com.liferay.portal.search.elasticsearch7.configuration.ElasticsearchConnectionConfiguration-[key].config
```

If using Elasticsearch 6 on Liferay 7.2, the configuration file is named

```
com.liferay.portal.search.elasticsearch6.configuration.ElasticsearchConfiguration.config
```

If configuring security on Elasticsearch 6, a separate Liferay configuration (as well as a LES subscription) is required. See [Securing Elasticsearch](securing-elasticsearch.md) for more information.

Deploy the files to `[Liferay_Home]/osgi/configs` and a listener auto-detects the configurations and writes them to the database.

## Elasticsearch Connection Settings

The configuration settings listed are for Liferay's Elasticsearch connector as of Liferay DXP 7.3.10 SP1. Most of these settings can be defined in either the Elasticsearch 7 entry or Elasticsearch Connections, under the System Settings &rarr; Search category.

### Properties Unique to the Elasticsearch 7 Configuration

**Production Mode Enabled, `productionModeEnabled="false"`** \
Enable production mode. In Liferay 7.3, `productionModeEnabled` replaces the deprecated setting `operationMode`. If this is checked, production mode is enabled and the Operation Mode configuration is ignored. Enabling production mode requires connecting to a remote standalone Elasticsearch cluster. If left disabled, the Operation Mode configuration is used.

[Deprecated] **Operation Mode, `operationMode="EMBEDDED"`** \
There are two operation modes you can choose from: EMBEDDED or REMOTE. Set to REMOTE to connect to a remote standalone Elasticsearch cluster. Set to EMBEDDED to start Liferay with an internal Elasticsearch instance. EMBEDDED operation mode is unsupported for production environments.

**Remote Cluster Connection ID, `remoteClusterConnectionId`** \
Choose the connection ID for a connection to the remote Elasticsearch cluster. The available connections are defined in the Elasticsearch Connections System Settings entry. If this value is not set then the connection configurations in the Elasticsearch 7 entry are used for the remote cluster connection.

<!-- embedded only on 7.3 -->
**Cluster Name, `clusterName=LiferayElasticsearchCluster`** \
A String value that sets the name of the cluster to integrate with. This name should match the remote cluster when Operation Mode is set to remote.  (See also: remote operation mode)


### Properties Unique to the Elasticsearch Connections Configuration

**Active, `active="false"`** \

**Connection ID, `connectionId=`** \
Set a unique ID for the connection. This is referenced in the Elasticsearch 7 property Remote Cluster Connection ID.

### Properties Shared Between the Elasticsearch 7 and Elasticsearch Connections Configurations

**Index Name Prefix, `indexNamePrefix=liferay-`** \
Set a String value to use as the prefix for the search index name. The default value should not be changed under normal conditions. If you change it, you must also perform a *reindex all* operation for the portal and then manually delete the old index using the Elasticsearch administration console.

**Index Number of Replicas, `indexNumberOfReplicas=`** \
Set the number of replicas for each index. If left unset, no replicas are used.  A full reindex is required to make changes take effect.

**Index Number of Shards, `indexNumberOfShards=`** \
Set the number of index shards to use when a Liferay index is created. If left unset, a single shard is used. A full reindex is required to make changes take effect. 

**Bootstrap Mlock All, `bootstrapMlockAll=false`** \
A boolean setting that, when set to `true`, tries to lock the process address space into RAM, preventing any Elasticsearch memory from being swapped out (see [here](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/setup-configuration-memory.html#bootstrap-memory_lock)) for more information)

**Log Exceptions Only, `logExceptionsOnly=true`** \
A boolean setting that, when set to true, only logs exceptions from Elasticsearch, and does not rethrow them.

**Retry On Conflict, `retryOnConflict=5`** \
Set an int value for the number of retries to attempt if a version conflict occurs because the document was updated between getting it and updating it (see [here](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/docs-update.html#docs-update-api-query-params) for more information).

**Discovery Zen Ping Unicast Hosts Port, `discoveryZenPingUnicastHostsPort=9300-9400`** \
Set a String value for the range of ports to use when building the value for discovery.zen.ping.unicast.hosts. Multiple Elasticsearch nodes on a range of ports can act as gossip routers at the same computer (see [here](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/modules-discovery-hosts-providers.html) for more information).

**Network Host, `networkHost=`** \
Set this String value to instruct the node to bind to this hostname or IP address and publish (advertise) this host to other nodes in the cluster. This is a shortcut which sets the bind host and the publish host at the same time (see [here](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/modules-network.html#common-network-settings) for more information).

**Network Bind Host, `networkBindHost=`** \
Set the String value of the network interface(s) a node should bind to in order to listen for incoming requests (see [here](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/modules-network.html#advanced-network-settings) for more information).

**Network Publish Host, `networkPublishHost=`** \
Set the String value of a single interface that the node advertises to other nodes in the cluster, so that those nodes can connect to it (see [here](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/modules-network.html#advanced-network-settings) for more information).

**Transport Tcp Port, `transportTcpPort=`** \
Set the String value for the port to bind for communication between nodes.  Accepts a single value or a range (see [here](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/modules-transport.html#_tcp_transport) for more information).

**Transport Addresses, `transportAddresses=localhost:9300`** \
Set the String values for the addresses of the remote Elasticsearch nodes to connect to. This value is required when Operation Mode is set to remote (see [here](https://www.elastic.co/guide/en/elasticsearch/client/java-api/7.x/transport-client.html) for more information). Specify as many or few nodes as you see fit.

**Client Transport Sniff, `clientTransportSniff=true`** \
Set this boolean to true to enable cluster sniffing and dynamically discover available data nodes in the cluster (see [here](https://www.elastic.co/guide/en/elasticsearch/client/java-api/7.x/transport-client.html) for more information).

**Client Transport Ignore Cluster Name, `clientTransportIgnoreClusterName=false`** \
Set this boolean to true to ignore cluster name validation of connected nodes (see [here](https://www.elastic.co/guide/en/elasticsearch/client/java-api/7.x/transport-client.html) for more information).

**Client Transport Ping Timeout, `clientTransportPingTimeout=`** \
Set the time (in seconds) the client node waits for a ping response from a node. If unset, the default Elasticsearch `client.transport.ping_timeout` is used.

**Client Transport Nodes Sampler Interval, `clientTransportNodesSamplerInterval=`** \
Set this String value to instruct the client node on how often to sample / ping the nodes listed and connected (see [here](https://www.elastic.co/guide/en/elasticsearch/client/java-api/7.x/transport-client.html) for more information).

**Http Enabled, `httpEnabled=true`** \
Set this boolean to false to disable the HTTP layer entirely on nodes which are not meant to serve REST requests directly. As this setting was [deprecated in Elasticsearch 6.3](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/release-notes-6.3.0.html#deprecation-6.3.0), the connector's corresponding setting is now also deprecated. This setting was only used for configuring the embedded Elasticsearch server, so its deprecation should have minimal impact to production deployments.

**Http CORS Enabled, `httpCORSEnabled=true`** \
Set this boolean to false to disable cross-origin resource sharing. When set to `false`, a browser on another origin cannot make requests to Elasticsearch. Web front end tools like Elasticsearch Head may be unable to connect (see [here](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/modules-http.html#_settings) for more information).

**Http CORS Allow Origin, `httpCORSAllowOrigin=/https?:\\/\\/localhost(:[0-9]+)?/`** \
Set the String origins to allow when HTTP CORS is enabled (see [here](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/modules-http.html#_settings) for more information).

**Http CORS Configurations, `httpCORSConfigurations=`** \
Set the String values for custom settings for HTTP CORS, in YML format (`elasticsearch.yml`) (see [here](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/modules-http.html#_settings) for more information).

**Additional Configurations, `additionalConfigurations=`** \
Set the String values for custom settings for embedded Elasticsearch, in YML format. See: Adding Settings to the Liferay Elasticsearch Connector

**Additional Index Configurations, `additionalIndexConfigurations=`** \
Set the String values for custom settings for the Liferay index, in JSON or YML format (refer to the Elasticsearch Create Index API for more information).  See: Adding Settings to the Liferay Elasticsearch Connector

**Additional Type Mappings, `additionalTypeMappings=`** \
Set the String values for custom mappings for the `LiferayDocumentType`, in JSON format (refer to the Elasticsearch Put Mapping API for more information) See: Adding Settings to the Liferay Elasticsearch Connector

**Override Type Mappings, `overrideTypeMappings=`** \
Settings here override Liferay's default type mappings. This is an advanced feature that should be used only if strictly necessary. If you set this value, the default mappings used to define the Liferay Document Type in Liferay source code (for example, `liferay-type-mappings.json`) are ignored entirely, so include the whole mappings definition in this property, not just the segment you're modifying.

**Proxy Host, `proxyHost`** \
Set the proxy host for the client connection.

**Proxy Port, `proxyPort=0`** \
Set the proxy port for the client connection.

**Proxy Username, `proxyUserName`** \
Set the proxy user name for a proxy connection.

**Proxy Password, `proxyPassword`** \
Set the password for connecting to the proxy.

## Configurations only Affecting the Embedded Elasticsearch Server

These settings (defined above) are only meant to use while configuring the [embedded Elasticsearch server](using-the-sidecar-or-embedded-elasticsearch.md)). Configuring these will elicit no effect on remote Elasticsearch installations:

- `bootstrapMlockAll`
- `discoveryZenPingUnicastHostsPort`
- `networkHost`
- `networkBindHost`
- `networkPublishHost` 
- `transportTcpPort` 
- `httpEnabled`
- `httpCORSEnabled` 
- `httpCORSAllowOrigin` 
- `httpCORSConfigurations` 

You can configure these settings in the System Setting application, or as mentioned above, you can specify them in a deployable OSGi `.config` file.

## Related Topics

- [What's New in Search for 7.3?](../../getting-started/whats-new-in-search-for-73.md)
- [Securing Elasticsearch](securing-elasticsearch.md)
- [Connecting to Elasticsearch](connecting-to-elasticsearch.md)
