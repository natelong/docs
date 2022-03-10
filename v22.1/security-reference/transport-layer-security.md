---
title: Transport Layer Security (TLS) in CockroachDB
summary: Overview of Transport Layer Security (TLS) in CockroachDB
toc: true
docs_area: reference.security
---

This pages provides a conceptual overview of Transport Layer Security (TLS) and details its role in securing cluster traffic in CockroachDB.

## What is TLS?

Transport Layer Security (TLS) is a protocol used to establish securely authenticated and encrypted traffic between a client (the party who initiates the session) and a server (the party receiving the connection request).

### Key pairs

The fundamental mechanism of TLS is a mathematically pair of cryptographic keys (usually referred to as a 'key pair' for short):

- The **private key** is randomly generated.
- The **public key** is derived mathematically from the private key in such a way that:
	- Text encrypted with either can be decrypted with the other.
	- It is prohibitively computationally expensive to compute the private from the public key.

The useful implication is that if you hold such a key pair, you can distribute the public key to anyone with whom you want to communicate securely, and as long as you retain sole posession of the private key, two things are guaranteed:

- Anyone who has the public key can use it to encrypt a message and send it to you, knowing that you alone can decrypt it with the private key.
- Anyone who receives a message that can be decrypted correctly using the public key knows it *must* have been encrypted with the private key, meaning it must have come from you.

TLS therefore provides two critical security features:

- **Encryption**: it prevents messaged from being eavesdropped or tampered with.
- **Authentication**: it allows one or both parties to prove their identity that cannot be impersonated (or 'spoofed' in cyberspeak).

TLS connections can be either *one-sided*, meaning that only one party must prove its identity before an encrypted session is established, or *two-sided* (or *mutual*), meaning that both must prove their identities. In mutually authenticated TLS connections, each parties must have a key-pair issued by a jointly trusted CA.

### Who certifies the certificate? (The Certificate Authority)

If you encrypt a message with a TLS public certificate, you know that only a holder of the matching private key will be able to decrypt it. And perhaps the holder of the public certificate identifies themself as your friend, your bank, your employer, or the government. But how do you know that the certificate was ever actually held by the party you want to reach, rather than an imposter?

This is solved via the notion of a **Certificate Authority.** When TLS key pairs are cryptographicall generated, they are 'signed' by another cryptographic key, known as a 'root certificate' or 'certificate authority certificate'. This is held by a party responsible for issuing key pairs. 

On the public internet, Certificate Authority providers such as Identrust, Digicert, and Let's Encrypt provide this service. Some large companies and other organizations maintain their own Certificate Authorities. Often, distributed systems such as K8s clusters, or (spoiler alert) CockroachDB clusters, may maintain their own internal Certificate Authority to allow their internal components to authenticate to one another.

CockroachDB clusters can generate their own CA certificates, allowing them to act as their own Certificate Authorities for TLS connections between nodes and from SQL clients to nodes. Alternatively, an external CA (such as your organization's CA) can be used to generate your certificates. It is even possible to employ a ['split certificate'](../create-security-certificates-custom-ca.html#split-node-certificates) scenario, where one set of key pairs is used for authentication in inter-node connections, and another pair is used for authentication in connections originated from SQL clients.

## TLS in CockroachDB

### Internode communication

Connections between CockroachDB nodes, and from SQL clients and web consoles are always mutually TLS authenticated, meaning that both client and server must each have their own key pair:

- A `node.crt` file, the node's public certificate.
- A `node.key` file, the node's private key.

In addition, the node must have a copy of the public CA certificate, called `ca.crt`.

CockroachDB nodes make double use of their key-pairs, using them as client credentials when they initiate connections to other nodes, and as server credentials when recieving requests.

### SQL client to CockroachDB node communication

Connections from CockroachDB SQL clients (the CockroachDB CLI client and the various support drivers and ORMs) can be either one-sidedly or mutually authenticated.

#### CockroachDB Cloud
{{ site.data.products.db }} currently does not support certificate-authenticated client requests, as required for mutually authenticated TLS, and client requests can only be authenticated with username/password.

#### Self-Hosted CockroachDB

To make mutually-TLS-authenticated SQL connections, a client must be provisioned with a key-pair:

- `client.<username>.crt`
- `client.<username>.key`

`<username>` here corresponds to the username of the SQL user as which the client will issue SQL statements if authentication is successful.





