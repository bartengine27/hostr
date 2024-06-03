Setting up a CA chain with Ansible and automating the process of generating CSRs (Certificate Signing Requests) for each virtual machine/container is a robust and secure way to manage certificates in our infrastructure. For developers, it is probably sufficient to generate certificates used to encrypt internal traffic and for your *public* endpoint in your LAN. If you'd like to publish your *public* endpoint on the Internet, you should use a CA like **Let's Encrypt**.  

1. **Ansible Roles**:
    * **CA Role**: This role sets up a Certificate Authority (CA).
    * **Client Role**: This role will generate CSRs for each virtual machine and handle the signing of certificates by the CA.

2. **CA Role**:
    * Configure the CA server: Install and configure software like OpenSSL to act as the CA.
    * Generate the CA certificate and private key.
    * Create necessary directories to store certificates and keys.
    * Set up appropriate permissions to ensure security.

3. **Client Role**:
    * Install necessary tools like OpenSSL for generating CSRs.
    * Generate CSRs for each virtual machine.
    * Securely transmit the CSRs to the CA server.
    * Sign the CSRs.
    * Retrieve signed certificates from the CA.

By following the steps above, we can set up our own (small/simple) CA chain with Ansible and automate the process of generating CSRs and signing certificates for our virtual machines/internal traffic.

To ensure that consumers trust our client certificates and/or to establish secure communication channels within our infrastructure (network communication within the datacentre), we have several options:

* **Distribute CA Certificate**: Distribute the CA certificate (which was used to sign the client certificates) to the consumers of the client certificates. Consumers can then import the CA certificate into their trust store (e.g., browser's certificate store, operating system's certificate store) as a trusted root CA. This allows the consumers to trust any certificates signed by that CA. :cupid: If you run the Ansible playbooks on Ubuntu, the `caclient` playbook will install the root certificate for you, on Windows, you will have to install the certificate manually (at least, for now).
* **Certificate Chain**: Along with the client certificate, provide the entire certificate chain up to the root CA certificate. This includes the client certificate, any intermediate CA certificates, and the root CA certificate. Consumers can then verify the certificate chain, ensuring that each certificate in the chain is signed by the preceding one, up to the trusted root CA certificate. :fire: In this particular implementation, we have no certificate chain.
* **Publicly Trusted CA**: If you require wider trust, consider obtaining client certificates from a publicly trusted CA (e.g., Let's Encrypt, DigiCert). Certificates issued by publicly trusted CAs are automatically trusted by most systems, as they are included in the trust stores of popular browsers and operating systems. :fire: For our public endpoint, we will use Let's Encrypt, internal traffic, however, will be encrypted with the self-generated client certificates.
* **Custom Trust Store:** Create a custom trust store containing the CA certificate or certificate chain and distribute it to the consumers of the client certificates. Applications can then be configured to use this custom trust store to verify the authenticity of client certificates. :fire: Might be used in a future implementation.
* **Certificate Pinning**: Implement certificate pinning, where consumers explicitly trust a specific client certificate or CA certificate by embedding its fingerprint or public key within the application code. This ensures that only the specified certificate or CA is trusted, providing an additional layer of security. :fire: Might be used in a future implementation.
* **Education and Documentation**: Educate consumers about the trust model and security practices involved in certificate authentication. Provide clear documentation on how to verify and trust client certificates, including instructions for importing CA certificates or certificate chains into trust stores. :fire: Might be used in a future implementation.

In our implementation, the **Client Role** will copy the CA certificate to the default *Ubuntu* certificate stores.

### Install Certificate Authority

```shell
ansible-playbook ca.yml -i hosts -K -vvv
```

### Generate Client Certificates

```shell
ansible-playbook caclient.yml -i hosts -K -vvv
```
