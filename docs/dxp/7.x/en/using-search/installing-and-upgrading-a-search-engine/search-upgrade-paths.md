# Search Upgrade Paths 

The following table provides the steps for upgrading from your current Liferay installation to Liferay 7.3 and a supported search engine stacks. 

Find the scenario that matches your Liferay version and Liferay Enterprise Search (LES) version (if you're using LES), and your current search engine stack. The *Upgrade Steps* column summarizes your upgrade.

| Scenario | Liferay Version \[+ LES Version\] | Search Engine Stack | Upgrade Steps |
| :-- | :-------- | :---------------- | :-------------- |
| 1.  | **Liferay 7.2** | Elasticsearch 7.9+ | 1. [Connect Liferay to Elasticsearch 7.](./elasticsearch/connecting-to-elasticsearch.md)<br><br>2. [Configure security.](./elasticsearch/securing-elasticsearch.md)<br><br>3. [Upgrade Liferay.](../../installation-and-upgrades/upgrading-liferay/upgrade-basics/upgrade-overview.md)<br><br>4. [Re-index search & spell check indexes.](../../installation-and-upgrades/upgrading-liferay/upgrade-basics/post-upgrade-considerations.md) |
| 2.  | **Liferay 7.2 + LES 3.0** (*Monitoring*, *Learning to Rank*) | Elasticsearch 7.9+ | 1. Connect Liferay to Elasticsearch 7.<br><br>2. Configure security.<br><br>3. Install Kibana 7.9+ if you are currently using *Kibana and Monitoring*.<br><br>4. Install and deploy LES Monitoring if you are currently using Kibana and *Elasticsearch Monitoring/X-Pack Monitoring*.<br><br>5. Configure the *Elasticsearch Monitoring* connector if you are using *LES Monitoring* or *Connector to X-Pack Monitoring*.<br><br>6. [Upgrade Liferay.](../../installation-and-upgrades/upgrading-liferay/upgrade-basics/upgrade-overview.md)<br><br>7. [Re-index search & spell check indexes.](../../installation-and-upgrades/upgrading-liferay/upgrade-basics/post-upgrade-considerations.md) |
| 3.  | **Liferay 7.2** | Elasticsearch 7.3.x-7.8.x | 1. [Install Elasticsearch 7.9+.](./elasticsearch/installing-elasticsearch.md)<br><br>2. Follow *Scenario 1.* |
| 4.  | **Liferay 7.2 + LES 2.0** (*Monitoring*, *Learning to Rank*) | Elasticsearch 7.3.x-7.8.x | 1. [Install Elasticsearch 7.9+.](./elasticsearch/installing-elasticsearch.md)<br><br>2. Follow *Scenario 2.* |
| 5.  | **Liferay 7.2** | Elasticsearch 6.x | 1. [Install Elasticsearch 7.9+.](./elasticsearch/installing-elasticsearch.md)<br><br>2. Follow *Scenario 1.* |
| 6.  | **Liferay 7.2 + LES 2.0** (*Monitoring*, *Learning to Rank*) | Elasticsearch 6.x | 1. [Install Elasticsearch 7.9+.](./elasticsearch/installing-elasticsearch.md)<br><br>2. Follow *Scenario 2.* |
| 7.  | **Liferay 7.1** | Elasticsearch 6.x | 1. [Install Elasticsearch 7.9+.](./elasticsearch/installing-elasticsearch.md)<br><br>2. Follow *Scenario 1.* |
| 8.  | **Liferay 7.1 + LES 2.0** (*Monitoring*, *Learning to Rank*) | Elasticsearch 6.x | 1. [Install Elasticsearch 7.9+.](./elasticsearch/installing-elasticsearch.md)<br><br>2. Follow *Scenario 2.* |
| 9.  | **Liferay 7.0** | Elasticsearch 6.x | 1. [Install Elasticsearch 7.9+.](./elasticsearch/installing-elasticsearch.md)<br><br>2. Follow *Scenario 1.* |
| 10. | **Liferay 7.0 + LES 2.0** (*Monitoring*, *Learning to Rank*) | Elasticsearch 5.x | 1. [Install Elasticsearch 7.9+.](./elasticsearch/installing-elasticsearch.md)<br><br>2. Follow *Scenario 2.* |
| 11. | **Liferay 7.0** | Elasticsearch 2.x | 1. [Install Elasticsearch 7.9+.](./elasticsearch/installing-elasticsearch.md)<br><br>2. Follow *Scenario 1.* |
| 12. | **Liferay 7.2, 7.1** | Solr 7.x | **Switch to Elasticsearch:**<br><br>1. [Install Elasticsearch 7.9+.](./elasticsearch/installing-elasticsearch.md)<br><br>2. Follow *Scenario 1* for configuring Liferay 7.3 with Elasticsearch<br> or follow *Scenario 2* for Liferay 7.3 + Liferay Enterprise Search (LES) 3.0<br><br>**or**<br><br><br>**Upgrade Solr (deprecated):**<br><br>1. [Set up Solr 8.x.](./solr.md)<br><br>2. [Upgrade Liferay.](../../installation-and-upgrades/upgrading-liferay/upgrade-basics/upgrade-overview.md)<br><br>3. [Re-index search & spell check indexes.](../../installation-and-upgrades/upgrading-liferay/upgrade-basics/post-upgrade-considerations.md) |
| 13. | **Liferay 7.0** | Solr 5.x | Same as *Scenario 12.* |

## What's Next 

Now that you know your upgrade path, start upgrading to use Liferay 7.3 with the latest [Elasticsearch](./elasticsearch/upgrading-elasticsearch.md) (*recommended*) or [Solr](./solr.md) (now deprecated as of Liferay 7.3) search engine.

## Additional Information 

* [Upgrading Elasticsearch](./elasticsearch/getting-started-with-elasticsearch.md)
* [Getting Started with Elasticsearch](./elasticsearch/getting-started-with-elasticsearch.md)
* [Installing Elasticsearch](./elasticsearch/installing-elasticsearch.md)
* [Connecting to Elasticsearch](./elasticsearch/connecting-to-elasticsearch.md)
* [Securing Elasticsearch](./elasticsearch/securing-elasticsearch.md)
* [Upgrading Liferay](../../installation-and-upgrades/upgrading-liferay/upgrade-basics/upgrade-overview.md)