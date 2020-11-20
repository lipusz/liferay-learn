# Securing Elasticsearch 

```note::
   **Elasticsearch 6.x:** If you're using Elasticsearch 6, you'll need a Liferay Enterprise Search (LES) subscription and the Liferay Enterprise Search Security application to use Elastic's X-Pack Security. Starting with the Liferay Connector to Elasticsearch 7 (available on `the Customer Downloads portal <https://customer.liferay.com/downloads>`_ and bundled in Liferay 7.3), support for Elastic's X-Pack security is included by default. To integrate with Elastic's X-Pack monitoring, LES is required.
```

## Securing Elasticsearch on Liferay DXP 7.2

The very first thing you must do to secure Elasticsearch is [enable X-Pack Security](#enable-x-pack-security). After that you can begin configuring authentication and Transport Layer Security.

### Enable X-Pack Security

To enable security, add this setting in each Elasticsearch node's `[Elasticsearch Home]/config/elasticsearch.yml` file:

```yaml
xpack.security.enabled: true
```

Now you can set up X-Pack users.

### Set Up X-Pack Users

In a system using X-Pack, these [built-in X-Pack users](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/built-in-users.html) are important:

* `kibana`
* `elastic`

On your Elasticsearch server, use the [`setup-passwords` command](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/setup-passwords.html) to set passwords for them:

```bash
./bin/elasticsearch-setup-passwords interactive
```

```note::
  The configurations shown below assume all passwords are set to *liferay*. Use your own passwords for your installation.

  To update a built-in user's password, use Kibana's UI or the `Change Password API <https://www.elastic.co/guide/en/elasticsearch/reference/7.x/security-api-change-password.html>`_.
```

### Encrypting Elasticsearch Communication

Enabling Transport Layer Security (TLS) involves generating node certificates and keys and applying them to the Elasticsearch servers and Liferay servers.

#### Generate Node Certificates

[Generate an X-Pack certificate](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/configuring-tls.html#node-certificates) for each node. Alternatively, use a certificate authority to obtain node certificates.

1. Generate X-Pack certificate authority using the X-Pack's [`certutil`](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/certutil.html) command:

    ```bash
    ./bin/elasticsearch-certutil ca --pem --ca-dn CN=elastic-ca
    ```

    This generates a ZIP file.

1. Unzip the certificate authority ZIP file contents in the `[Elasticsearch Home]/config/certs` folder.

1. Generate X.509 certificates and private keys using the CA you created:

    ```bash
    ./bin/elasticsearch-certutil cert --pem --ca-cert config/certs/ca.crt --ca-key config/certs/ca.key --dns localhost --ip 127.0.0.1 --name elastic-nodes
    ```

    To generate a certificate that works with multiple hosts (for example to use the same certificate on all Elasticsearch and DXP servers) use

    ```
    --dns localhost,dxp.liferay.com,es-node1,es-node2,es-node3 --ip 127.0.0.1,<IPv4-address-2>,<IPv4-address-3>,<IPv4-address-4>
    ```

    This generates another ZIP file.

    ```note::
       The ``certutil`` command defaults to using the *PKSC#12* format for certificate generation. Since Kibana does not work with PKSC#12 certificates, the ``--pem`` option (generates the certificate in PEM format) is important if you're using X-Pack monitoring.
    ```

1. Extract the ZIP file contents in the `[Elasticsearch Home]/config/certs` folder.

    **Checkpoint:** You now have the following files in your `[Elasticsearch Home]/config/certs` folder:

    ```bash
    ca.crt
    ca.key
    elastic-nodes.crt
    elastic-nodes.key
    ```

1. Copy the files to the same folder on each Elasticsearch and Liferay server node.

The certificates and keys are ready to use in your Elasticsearch configuration.

#### Configure TLS for Elasticsearch 7

[Enable TLS](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/configuring-tls.html#enable-ssl) on each node via its `[Elasticsearch Home]/config/elasticsearch.yml` file.

1. Enable transport layer TLS with these settings in `elasticsearch.yml` for inter-node communication:

    ```yaml
    xpack.security.transport.ssl.enabled: true
    ```

1. Add the certificate, key and certificate authority paths to each node's `elasticsearch.yml`:

    ```yaml
    xpack.security.transport.ssl.certificate_authorities: [ "certs/ca.crt" ]
    xpack.security.transport.ssl.certificate: certs/elastic-nodes.key
    xpack.security.transport.ssl.key: certs/elastic-nodes.crt
    xpack.security.transport.ssl.verification_mode: certificate
    ```

    The example paths above assume you added the certificate to `[Elasticsearch Home]/config/`. 

1. Enable TLS on the HTTP layer to encrypt client communication:

    ```yaml
    xpack.security.http.ssl.enabled: true
    ```

1. Configure the certificate, key, and certificate authority paths to each node's `elasticsearch.yml`:

    ```yaml
    xpack.security.http.ssl.certificate_authorities: [ "certs/ca.crt" ]
    xpack.security.http.ssl.certificate: certs/elastic-nodes.crt
    xpack.security.http.ssl.key: certs/elastic-nodes.key
    xpack.security.http.ssl.verification_mode: certificate
    ```

#### Configure TLS for Elasticsearch 6

The settings for Elasticsearch 6 are slightly different from those presented above for Elasticsearch 7. [Enable TLS](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/configuring-tls.html#enable-ssl) on each node via its `[Elasticsearch Home]/config/elasticsearch.yml` file.

1. Add the certificate, key and certificate authority paths to each node's `elasticsearch.yml`:

    ```yaml
    xpack.ssl.certificate_authorities: [ "certs/ca.crt" ]
    xpack.ssl.certificate: certs/elastic-nodes.crt 
    xpack.ssl.key: certs/elastic-nodes.key
    xpack.ssl.verification_mode: certificate 
    ```

    The example paths above assume you added the certificate to `[Elasticsearch Home]/config/`. 

1. Enable transport layer TLS with these settings in `elasticsearch.yml`:

    ```yaml
    xpack.security.transport.ssl.enabled: true
    ```

1. Enable TLS on the HTTP layer to encrypt client communication:

    ```yaml
    xpack.security.http.ssl.enabled: true
    ```

After X-Pack is installed and encryption is enabled, configure the corresponding security settings in Liferay (discussed below).

#### Example Elasticsearch Security Configuration

Here is the complete Elasticsearch 7 configuration (`elasticsearch.yml`; applies equally to Elasticsearch 6.x):

```yaml
cluster.name: LiferayElasticsearchCluster

# X-Pack Security
xpack.security.enabled: true

## TLS/SSL settings for Transport layer
xpack.security.transport.ssl.enabled: true 
xpack.security.transport.ssl.certificate_authorities : [ "certs/ca.crt" ]
xpack.security.transport.ssl.certificate: certs/elastic-nodes.crt
xpack.security.transport.ssl.key: certs/elastic-nodes.key
xpack.security.transport.ssl.verification_mode: certificate

# TLS/SSL settings for HTTP layer
xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.certificate_authorities : [ "certs/ca.crt" ]
xpack.security.http.ssl.certificate: certs/elastic-nodes.crt
xpack.security.http.ssl.key: certs/elastic-nodes.key
xpack.security.http.ssl.verification_mode: certificate 

# Comment out when Kibana and Liferay's X-Pack Monitoring are also configured
#xpack.monitoring.collection.enabled: true
```

### Configure Liferay's Secure Connection to Elasticsearch

All Liferay connector's to Elasticsearch 7 include X-Pack Security support.

```note::
   If you are on Liferay 7.2 and Elasticsearch 6.x and have a Liferay Enterprise Search subscription, `download <https://customer.liferay.com/downloads>`__ the `Liferay Enterprise Search Security` application. Install the LPKG file by copying it into the ``[Liferay Home]/deploy`` folder.
```

On Liferay, security can be configured in the Control Panel or using a configuration file. Navigate to *Control Panel* &rarr; *Configuration* &rarr; *System Settings*. Find the *Search* category and click on the *X-Pack Security* entry. You can enter the property values here, but it's more common to use a configuration file deployed to `[Liferay Home]/osgi/configs`. Create a file called

```bash
com.liferay.portal.search.elasticsearch7.configuration.XPackSecurityConfiguration.config
```

The exact contents of the file depend on your X-Pack setup. To configure the adapter according to the Elasticsearch setup documented here, populate the file like this:

```properties
sslKeyPath="/path/to/elastic-nodes.key"
sslCertificatePath="/path/to/elastic-nodes.crt"
certificateFormat="PEM"
requiresAuthentication="true"
username="elastic"
password="liferay"
sslCertificateAuthoritiesPaths="/path/to/ca.crt"
transportSSLVerificationMode="certificate"
transportSSLEnabled="true"
```

The `password` must match the one you set during the X-Pack user password setup above. 

The certificate and key files referenced here are the same ones used on the Elasticsearch server. Copy them to the Liferay server and update their paths in the configuration accordingly.

Enable authentication by setting `requiresAuthentication` to `true` and providing the credentials for the Elasticsearch user. For TLS, enable transport TLS, set the certificate verification mode and certificate format, and provide the path to the certificate, key, and certificate authority. Of course, the exact values depend on your X-Pack configuration. 

Here's the complete list of settings for the X-Pack Security configuration:

* `sslKeyPath`
* `sslCertificatePath`
* `sslCertificateAuthoritiesPaths`
* `certificateFormat`
* `requiresAuthentication`
* `username`
* `password`
* `transportSSLVerificationMode`
* `transportSSLEnabled`
* `sslKeystorePath`
* `sslKeystorePassword`
* `sslTruststorePath`
* `sslTruststorePassword`

When you're finished configuring security, restart Elasticsearch. These steps require a full Elasticsearch cluster restart.

### Disable Elasticsearch Deprecation Logging

Sometimes, Elasticsearch APIs used by Liferay's Elasticsearch connectors are deprecated. Even when there's no impact to the functionality required by Liferay, warning loge messages can result:

```
[2019-07-16T14:47:05,779][WARN ][o.e.d.c.j.Joda           ] [
ode_name]'y' year should be replaced with 'u'. Use 'y' for year-of-era. Prefix your date format with '8' to use the new specifier.
[2019-07-16T14:47:06,007][WARN ][o.e.d.c.s.Settings       ] [
ode_name][xpack.ssl.certificate] setting was deprecated in Elasticsearch and will be removed in a future release! See the breaking changes documentation for the next major version.
[2019-07-16T14:47:06,007][WARN ][o.e.d.c.s.Settings       ] [
ode_name][xpack.ssl.certificate_authorities] setting was deprecated in Elasticsearch and will be removed in a future release! See the breaking changes documentation for the next major version.
[2019-07-16T14:47:06,008][WARN ][o.e.d.c.s.Settings       ] [
ode_name][xpack.ssl.key] setting was deprecated in Elasticsearch and will be removed in a future release! See the breaking changes documentation for the next major version.
[2019-07-16T14:47:06,463][WARN ][o.e.d.x.c.s.SSLService   ] [
ode_name]SSL configuration [xpack.http.ssl] relies upon fallback to another configuration for [key configuration, trust configuration], which is deprecated.
[2019-07-16T14:47:06,464][WARN ][o.e.d.x.c.s.SSLService   ] [
ode_name]SSL configuration [xpack.security.transport.ssl.] relies upon fallback to another configuration for [key configuration, trust configuration], which is deprecated.
1. 07-16T14:47:05,779][WARN ][o.e.d.c.j.Joda           ] [
```

These warnings do not signal any functional issues, and can be disabled (see [Deprecation Logging](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/logging.html#deprecation-logging) to learn how).

## Securing Elasticsearch on Liferay DXP 7.3

Coming Soon!

```tip::
   The `Installing Elasticsearch <./installing-elasticsearch.md>`__ and `Connecting to Elasticsearch <./connecting-to-elasticsearch.md>`__ articles default to enabling and configuring security, so you can visit those articles for the 7.3 applicable security configurations.
```
