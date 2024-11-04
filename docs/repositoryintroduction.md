Welcome to our GitHub repository where you'll find Ansible scripts written to facilitate the development of web applications by mirroring a typical production environment. This approach might address common challenges where hardware or time constraints make it difficult to set up a test or acceptance environment that accurately reflects various production settings.

## Purpose of This Repository

This repository hosts Ansible scripts that allow you to deploy a complex web application environment either locally on your development machine, on-premises hardware, or in a cloud setup. By leveraging these scripts, developers may accelerate the setup process and ensure consistency across different environments, from development to production.

## Environment Roles Defined

The provided Ansible scripts are prepared to configure the following roles essential for a full-featured web application infrastructure:

* **Web Server**: Hosts the web application interface.
* **Cache**: Improves response time and reduces load on the database by storing recently accessed data.
* **Database**: Manages data storage and retrieval.
* **Identity Server**: Handles authentication and authorization services.
* **REST API endpoints**: Facilitates communication between the client apps and server.
* **Load Balancer**: Distributes incoming network traffic across multiple servers.
* **Certificate Authority**: Issues SSL/TLS certificates for secured communications.
* **HTTPS offloading**: Relieves backend application servers of the processing load associated with encrypting and decrypting public HTTPS traffic.
* **Prometheus monitoring**: Collects and stores metrics as time-series data.
* **Grafana dashboards**: Provides visualization of the metrics collected by Prometheus.
