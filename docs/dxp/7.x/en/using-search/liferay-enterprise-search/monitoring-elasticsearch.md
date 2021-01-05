# Monitoring Elasticsearch

> LES Subscribers

You must have a Liferay Enterprise Search (LES) [subscription](https://www.liferay.com/products/dxp/enterprise-search) to use Liferay's intergration with Elastic's [Kibana monitoring UI](https://www.elastic.co/guide/en/kibana/7.x/introduction.html).

To monitor the [secured](../installing-and-upgrading-a-search-engine/elasticsearch/securing-elasticsearch.md) Elasticsearch cluster where Liferay's data is indexed,

1. Tell Elasticsearch to enable data collection.

1. Download and install Kibana.

1. Configure Kibana with the proper security settings.

1. Install the Liferay Enterprise Search Monitoring app.

1. Configure the LES Monitoring app to communicate with Elasticsearch.

## Enable Data Collection

Monitoring is enabled on Elasticsearch by default, but data collection isn't.  Enable data collection by adding this line to `elasticsearch.yml`.

```yaml
xpack.monitoring.collection.enabled: true
```

<!-- Do we need this in elasticsearch.yml? 
# xpack.security.http.ssl.client_authentication: "[optional/required]"
If so we should just do it here 
-->

Now install Kibana.

## Install Kibana

Make sure to install the correct version of Kibana. Check the [Liferay Enterprise Search compatibility matrix](https://help.liferay.com/hc/en-us/articles/360016511651) for details.

1. [Download Kibana](https://www.elastic.co/downloads/kibana) and extract it. The root folder is referred to as *Kibana Home*.

1. Tell Kibana where to send monitoring data by setting Elasticsearch's URL in `kibana.yml`:

   ```yaml
   elasticsearch.hosts: [ "https://localhost:9200" ]
   ```

   If TLS/SSL is not enabled on Elasticsearch, this is an `http` URL, otherwise use `https`.

   If you're not configuring security, start Kibana.

1. Now configure authentication. Set the password for the built-in `kibana` user in `[Kibana Home]/config/kibana.yml`:

   ```yaml
   elasticsearch.username: "kibana_system"
   elasticsearch.password: "liferay"
   ```

   Use the `kibana_system` user password from your [security configuration](../installing-and-upgrading-a-search-engine/elasticsearch/securing-elasticsearch.md). Once Kibana is installed, you can change the built-in user passwords from the *Management* user interface.

1. Begin configuring encryption by providing certificate files. See [Elastic's guide](https://www.elastic.co/guide/en/kibana/7.x/using-kibana-with-security.html#using-kibana-with-security) for more details.

   To reuse the files [created for Elasticsearch itself](../installing-and-upgrading-a-search-engine/elasticsearch/securing-elasticsearch.md#generate-node-certificates), copy the `[Elasticsearch Home]/config/certs` folder into the `[Kibana Home]/config/` folder.

   If you wish to generate a separate certificate for your Kibana instance, make sure it is signed by the same CA as the Elasticsearch node certificates.

1. Add these settings to `kibana.yml`:

   ```yaml
   xpack.security.encryptionKey: "xsomethingxatxleastx32xcharactersx"
   xpack.security.session.idleTimeout: "1h"
   xpack.security.session.lifespan: "30d"

   elasticsearch.ssl.truststore.path: "/path/to/kibana_home/config/certs/elastic-stack-ca.p12"
   elasticsearch.ssl.truststore.password: "liferay"   

   elasticsearch.ssl.keystore.path: "/path/to/kibana_home/config/certs/elastic-nodes.p12"
   elasticsearch.ssl.keystore.password: "liferay"

   elasticsearch.ssl.verificationMode: certificate

   server.ssl.enabled: true

   server.ssl.truststore.path: "/path/to/kibana_home/config/certs/elastic-stack-ca.p12"
   server.ssl.truststore.password: "liferay"

   server.ssl.keystore.path: "/path/to/kibana_home/config/certs/elastic-nodes.p12"    
   server.ssl.keystore.password: "liferay"
   ```

<!-- questions about these settings: 

1. the elastic docs don't say to use the password setting in the kibana.yml, but to use the command "bin/kibana-keystore add server.ssl.keystore.password". Should we do the same?

2. This usage of PKCS#12 can work on Elasticsearch 7.x, but 6.x needs PEM certs. How should we handle this mix? We show both ways in the securing article, therefore we'll need to assume they could have either situation. I vote to default to PKCS#12 and treat PEM as the "if you need it" (like if on 7.2 with ES 6) scenario.


3. According to the docs, you'll also need to set this in elasticsearch.yml: xpack.security.http.ssl.client_authentication: "optional" OR "requireda. What do we do?"
-->

After this step you can access Kibana at `https://localhost:5601` and sign in with a Kibana user. The last step is to connect Kibana to Liferay.

## Install and Configure the LES Monitoring App

Download the LES Monitoring app and install the LPKG file by copying it into the `Liferay Home/deploy` folder. 
<!--does it require a restart? -->

1.  Once the connector is installed and Kibana and Elasticsearch are securely configured, create a [configuration file](../../system-administration/configuring-liferay/configuration-files-and-factories/using-configuration-files.md) named

   ```bash
   com.liferay.portal.search.elasticsearch.monitoring.web.internal.configuration.MonitoringConfiguration.config
   ```

1. Place these settings in the `.config` file:

   ```properties
   kibanaPassword="liferay"
   kibanaUserName="elastic"
   kibanaURL="https://localhost:5601"
   proxyServletLogEnable=B"true"
   ```

   During development and testing, enabling proxy servlet logging can be helpful:

   ```properties
   proxyServletLogEnable=B"true"
   ```

   The exact configuration values depend on your Kibana configuration. For example, use a URL such as `kibanaURL="http://localhost:5601"` if you are not enabling TLS.

   Alternatively, configure the monitoring adapter from [System Settings](../../system-administration/configuring-liferay/system-settings.md). In the Global Menu, navigate to *Control Panel* &rarr; *Configuration* &rarr; *System Settings* and find the Elasticsearch Monitoring entry in the Search category. All the configuration options for the monitoring connector appear there.

1. Deploy the configuration file to `Liferay Home/osgi/configs`, and your running instance applies the settings.

1. Add two more settings to Kibana itself. The first forbids Kibana from rewriting requests prefixed with `server.basePath`. The second sets Kibana's base path for the Monitoring portlet to act as a proxy for Kibana's monitoring UI. Add this to `kibana.yml`:

   ```yaml
   server.rewriteBasePath: false
   server.basePath: "/o/portal-search-elasticsearch-monitoring/monitoring-proxy"
   ```

   Once you set the `server.basePath`, you cannot access the Kibana UI through Kibana's URL (e.g., `https://localhost:5601`). All access to the Kibana UI is through the Monitoring portlet, which is only accessible to signed in Liferay users. Navigate directly to the portlet using the URL

   [http://localhost:8080/o/portal-search-elasticsearch-monitoring/monitoring-proxy/app/monitoring](http://localhost:8080/o/portal-search-elasticsearch-monitoring/monitoring-proxy/app/monitoring)

1. Because you're using the Monitoring portlet in Liferay as a proxy to Kibana's UI, if you are using TLS, you must configure the application server's startup JVM parameters to recognize a valid *truststore* and *password*.

   First, navigate to Elasticsearch Home and generate a PKSC#12 certificate from the CA you created when [setting up security](../installing-and-upgrading-a-search-engine/elasticsearch/securing-elasticsearch.md):

   ```bash
   ./bin/elasticsearch-certutil cert --ca-cert /path/to/ca.crt --ca-key /path/to/ca.key --ip 127.0.0.1 --dns localhost --name localhost --out /path/to/Elasticsearch_Home/config/localhost.p12
   ```

   Next use the `keytool` command to generate a truststore:

   ```bash
   keytool -importkeystore -deststorepass liferay -destkeystore /path/to/truststore.jks -srckeystore /path/to/Elasticsearch_Home/config/localhost.p12 -srcstoretype PKCS12 -srcstorepass liferay
   ```

   Add the trustore path and password to your application server's startup JVM parameters. Here are example truststore and path parameters for appending to a Tomcat server's `CATALINA_OPTS`:

    ```properties
    -Djavax.net.ssl.trustStore=/path/to/truststore.jks -Djavax.net.ssl.trustStorePassword=liferay
    ```

Restart Liferay and Kibana.

## Monitoring in Liferay

Once Kibana and LES Monitoring are successfully installed and configured and all the
servers are running, add the Elasticsearch Monitoring widget to a page:

1. Open the *Fragments and Widgets* menu on a page and choose *Widgets*.

1. Search for *monitoring* and drag the *Elasticsearch Monitoring* widget from the Search category onto the page.

See the Elastic documentation for information on [monitoring Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/es-monitoring.html).

For more information about monitoring and security best practices in a clustered environment, refer to [Elastic's documentation](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/setup-xpack.html).

## Example Kibana Configuration

Here are the `kibana.yml` properties demonstrated in this article, for convenient copy/pasting:

```yaml
# X-Pack Security enabled (Basic Auth)
elasticsearch.username: "kibana_system"
# if on Elasticsearch 6.x, use this username
# elasticsearch.username: "kibana"
elasticsearch.password: "liferay"

# When TLS/SSL is enabled in X-Pack Security
xpack.security.encryptionKey: "xsomethingxatxleastx32xcharactersx"
xpack.security.session.idleTimeout: "1h"
xpack.security.session.lifespan: "30d"

# If on Elasticsearch 6.5 or below, replace elasticsearch.hosts with:
# elasticsearch.url: "http://localhost:9200"
elasticsearch.hosts: [ "https://localhost:9200" ]

# Enable TLS/SSL for out-bound traffic: from Kibana to Elasticsearch
elasticsearch.ssl.verificationMode: certificate
elasticsearch.ssl.certificateAuthorities: [ "config/certs/ca.crt" ]
elasticsearch.ssl.certificate: config/certs/localhost.crt
elasticsearch.ssl.key: config/certs/localhost.key

# Enable TLS/SSL for in-bound traffic: from browser or
#  DXP (Elasticsearch Monitoring widget, proxy) to Kibana
server.ssl.enabled: true
server.ssl.certificateAuthorities: [ "config/certs/ca.crt" ]
server.ssl.certificate: config/certs/localhost.crt
server.ssl.key: config/certs/localhost.key

# These configurations require you to add these settings to elasticsearch.yml:
# xpack.monitoring.collection.enabled: true
# xpack.security.http.ssl.client_authentication: "[optional/required]"

# To use Kibana inside the Elasticsearch Monitoring widget
server.rewriteBasePath: false
server.basePath: "/o/portal-search-elasticsearch-monitoring/monitoring-proxy"

# HOPEFULLY THESE WORK TO REPLACE OR COMPLEMENT OUR PEM-SPECIFIC SETTINGS:

elasticsearch.ssl.truststore.path: "/path/to/kibana_home/config/certs/elastic-stack-ca.p12"
elasticsearch.ssl.truststore.password: "liferay"   

elasticsearch.ssl.keystore.path: "/path/to/kibana_home/config/certs/elastic-nodes.p12"
elasticsearch.ssl.keystore.password: "liferay"

elasticsearch.ssl.verificationMode: certificate

server.ssl.enabled: true

server.ssl.truststore.path: "/path/to/kibana_home/config/certs/elastic-stack-ca.p12"
server.ssl.truststore.password: "liferay"

server.ssl.keystore.path: "/path/to/kibana_home/config/certs/elastic-nodes.p12"    
server.ssl.keystore.password: "liferay"
```
