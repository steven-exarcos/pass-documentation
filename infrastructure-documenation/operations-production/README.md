# Operations/Production

The operations and production workflows, architecture, and infrastructure are composed of a variety of technologies,
layers, and techniques. The main important concept and understanding of this guide, is that PASS is designed to be 
platform independent, with a few exceptions. This guide will describe and summarize the way JHU decided to deploy the 
application, but this doesn't mean it can't be done another way. One way of deploying PASS is using Amazon Web Services
(AWS) cloud computing services. By using the scalable infrastructure we are able to quickly adapt to different demands 
of usage. By using AWS, it opens up the ability to use infrastructure as code which gives more efficiencies by enabling
reuse of infrastructure between environments and aids in the CI/CD pipeline, providing consistent and quick deployments.
In addition to deploying PASS, other operation activities will be described such as monitoring, harvesting and loading 
data, and other communication between different services.

If you haven't already, a quick review of the [Welcome Guide PASS Architecture](../../welcome-guide/deployment-architecture.md)
article will provide a good foundation for understanding the operations and production environment of PASS.

In this guide we step through various topics on JHU Operations and Production:

* [Knowledge Needed / Skills Inventory](./ops-know-need.md)
* [Technologies Utilized](./ops-tech-util.md)
* Technical Deep Dive
  * [PASS Design & Amazon Web Services (AWS) Architecture](./ops-aws-arch.md)
  * [PASS AWS Architecture Cost Estimates](./ops-aws-cost.md)
  * [Versioning](./ops-version.md)
  * [How to Deploy](./ops-deploy.md)
  * [Monitoring](./ops-monitor.md)
  * [Data Loaders](./ops-loaders.md)
  * [Data & Backups](./ops-data-backup.md)
  * [Eclipse Demo Site](./ops-demo-eclipse.md)
  * [Eclipse Operations](./ops-eclipse.md)
* [Next Steps / Institution Configuration](./ops-new-institution.md)