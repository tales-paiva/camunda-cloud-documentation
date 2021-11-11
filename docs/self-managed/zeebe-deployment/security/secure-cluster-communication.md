---
id: secure-cluster-communication
title: "Secure Cluster Communication"
---



# Secure cluster communication security

Zeebe supports transport layer security (TLS v1.3) between all nodes in a Zeebe cluster. This means it's possible to encrypt all TCP traffic between all nodes of a given cluster.

Enabling TLS for cluster communication is an all or nothing feature: either all nodes are configured to use TLS, or none are. It's not currently possible to only configure some nodes to enable TLS.

Additionally, a small portion of Zeebe traffic is done over UDP, which is left unencrypted. This is purely used for the nodes to gossip topology information amongst themselves, and no sensitive or user given data is transmitted this way.

Finally, note that transport layer security for cluster communication is disabled by default.

## How it works

When enabled for each node, communication over TCP between these will be securely encrypted using the provided certificates, in a client-server model. So given two nodes, `A` and `B`, when `A` (the client) sends a request to `B` (the server), they will perform a TLS handshake wherein `B`'s certificate will be exchanged and verified by `A`, after which the request will be encrypted such that only a node with `B`'s private key may decrypt it (i.e. in this instance, `B`).

When the roles are reverse (e.g. `B` sends a request to `A`), the same handshake will occur but in reverse - `B` is now the client, and `A` the server, therefore `A`'s certificate is exchanged and verified by `B`, after which all communication is encrypted and can only be decrypted with `A`'s private key.

> In this model, only the client verifies the identity of the server, as opposed to mTLS, in which both client and server would exchange and verify each other's identities. If you need mTLS, it's currently recommended that you explore a solution which provides this transparently like a service mesh (e.g. Linkerd or Istio).

## Configuration

If you wish to enable TLS for cluster communication, you will need to provide two things: a certificate file, and its private key.

The certificate chain file is expected to be a PEM public certificate file, which should contain a x509 public certificate, and may additionally contain an entire certificate chain. If it does include the chain, it should simply be concatenated after the node's certificate.

For example, a simple certificate file with only a single certificate:

```
-----BEGIN CERTIFICATE-----
MIIC+DCCAeACAQEwDQYJKoZIhvcNAQELBQAwQjELMAkGA1UEBhMCWFgxFTATBgNV
BAcMDERlZmF1bHQgQ2l0eTEcMBoGA1UECgwTRGVmYXVsdCBDb21wYW55IEx0ZDAe
Fw0yMTExMTEyMjI1MjhaFw0yMjExMTEyMjI1MjhaMEIxCzAJBgNVBAYTAlhYMRUw
EwYDVQQHDAxEZWZhdWx0IENpdHkxHDAaBgNVBAoME0RlZmF1bHQgQ29tcGFueSBM
dGQwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC6OBgCktwd9+QD7IYG
BiyAgvVYdpIasGZGWtbnRJwO9ogQuIFk2rjwpj8mddCwMkJ2o8yifHmkWVkQwXJh
QFxwljVN4HIKT8xsDUkNGG8+xh1uX1Dn+8kuxfukEccRqaXdy5zfWZLwpfnplzvY
iJA9dS51hMnBiNh9D1ND2/rjF4fZbxc2m6XPKz8lOOMJfRFJPwly6hO/03YlVyQM
JnT9a5DtlmYxsOXnXf2x7Q0LAr9pK/eAR+XYiGAOdTIVSvwtypUXZrxsIyF1fvWr
2EODVkBYSJZOGVeQPnqlfjIrkQM6TrOPWPsKIp4iGirdBBsGz3D55ZooHPN4lAoX
2KvbAgMBAAEwDQYJKoZIhvcNAQELBQADggEBAJZhdWAkH94cmjMWMzpTRUScjWfX
mQrDepw7yDbljoUJPOmP9K2hj5IusahJ5YqLnCFkHUhc0i+fTKJJd1Eypp8msWUc
ubvfltNfA6fo0fKKqjgSax9igrKBk1ETFYO8g2s7tXXGlF5QsJxiSSvQt5X0+M+t
vomlTP+qDclx69WLUWwqot/IuEUf9/VY6+XAiszClr1vQPbdz52l5FuF8QlamhfH
PQbjI3vzlN8EnJphydu5Vp6nYRaUn/flvoSuP/qEpABALI+5UeNmRE07WYdPgouX
XGuLA/JjibtGK8xm3n0lyAvxnydxr+R1slKdZ5x8F/X5oNnpjoAw1ojTB8k=
-----END CERTIFICATE-----
```

It you wanted to include its signing authority, for example, it would look like this:

```
-----BEGIN CERTIFICATE-----
MIIC+DCCAeACAQEwDQYJKoZIhvcNAQELBQAwQjELMAkGA1UEBhMCWFgxFTATBgNV
BAcMDERlZmF1bHQgQ2l0eTEcMBoGA1UECgwTRGVmYXVsdCBDb21wYW55IEx0ZDAe
Fw0yMTExMTEyMjI1MjhaFw0yMjExMTEyMjI1MjhaMEIxCzAJBgNVBAYTAlhYMRUw
EwYDVQQHDAxEZWZhdWx0IENpdHkxHDAaBgNVBAoME0RlZmF1bHQgQ29tcGFueSBM
dGQwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC6OBgCktwd9+QD7IYG
BiyAgvVYdpIasGZGWtbnRJwO9ogQuIFk2rjwpj8mddCwMkJ2o8yifHmkWVkQwXJh
QFxwljVN4HIKT8xsDUkNGG8+xh1uX1Dn+8kuxfukEccRqaXdy5zfWZLwpfnplzvY
iJA9dS51hMnBiNh9D1ND2/rjF4fZbxc2m6XPKz8lOOMJfRFJPwly6hO/03YlVyQM
JnT9a5DtlmYxsOXnXf2x7Q0LAr9pK/eAR+XYiGAOdTIVSvwtypUXZrxsIyF1fvWr
2EODVkBYSJZOGVeQPnqlfjIrkQM6TrOPWPsKIp4iGirdBBsGz3D55ZooHPN4lAoX
2KvbAgMBAAEwDQYJKoZIhvcNAQELBQADggEBAJZhdWAkH94cmjMWMzpTRUScjWfX
mQrDepw7yDbljoUJPOmP9K2hj5IusahJ5YqLnCFkHUhc0i+fTKJJd1Eypp8msWUc
ubvfltNfA6fo0fKKqjgSax9igrKBk1ETFYO8g2s7tXXGlF5QsJxiSSvQt5X0+M+t
vomlTP+qDclx69WLUWwqot/IuEUf9/VY6+XAiszClr1vQPbdz52l5FuF8QlamhfH
PQbjI3vzlN8EnJphydu5Vp6nYRaUn/flvoSuP/qEpABALI+5UeNmRE07WYdPgouX
XGuLA/JjibtGK8xm3n0lyAvxnydxr+R1slKdZ5x8F/X5oNnpjoAw1ojTB8k=
-----END CERTIFICATE-----
-----BEGIN TRUSTED CERTIFICATE-----
MIIDCzCCAfMCFFVfBL21+krAsnBFelsHEYEfNF0YMA0GCSqGSIb3DQEBCwUAMEIx
CzAJBgNVBAYTAlhYMRUwEwYDVQQHDAxEZWZhdWx0IENpdHkxHDAaBgNVBAoME0Rl
ZmF1bHQgQ29tcGFueSBMdGQwHhcNMjExMTExMjIyNTIzWhcNMjIxMTExMjIyNTIz
WjBCMQswCQYDVQQGEwJYWDEVMBMGA1UEBwwMRGVmYXVsdCBDaXR5MRwwGgYDVQQK
DBNEZWZhdWx0IENvbXBhbnkgTHRkMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIB
CgKCAQEAxqlzwEHRZhrK+24Y5tKyTJGilpQvNW4RZtbbn3ZfSZsB+IReZEPhimz2
H8+VR4V/sRkTjW5OoP4b9FYCQjkoqdiP4xQOgbHSMIk0mwN5CZkxf02nVJd8i5YM
jC0YzOtr/kw6mUFyQLl43w5SW7sSb0X92Z6Bqi/2+xUkflNPG3R+bQMuMVRX2teF
ResaKydy8cbdTvF1RxqFNQ1fGw5jDeLUWjrhhn4agT5zfjTgFEIuILqTg6T98tmt
4UdoMDz2m9egDcx9Z6jjmdlqsVP3cFVe9YTMhk5ZsRLE3atYpcKqc+hozzDh6Akm
3Rj3L2VMSZFF1NrO2wm+ALUzX4RBKQIDAQABMA0GCSqGSIb3DQEBCwUAA4IBAQBj
f6owN6rDC+Xd26ze0Ol/iIidbsg14YbsZNOg6j6eldCokq08BeJsbZj9tn64v1fi
8QHstKXVmP56Ucac4OqQOJFDLYJSMsJT4GqKiMbJjI8VynG42DN/YpaueAlyWmcX
1IPbMME6Ill2caQT85Q94kMEJgLpjGDu5Wz7cZGcB2eJY6WoSZPiY2FhDJDG0KWd
F4bUiEiSStui+dHPmGJWtfx4e2vQsJHFXTPazA34syM79KDowceGc3Eng7cTDUWm
bUuALvnPNHsX3FoZ9gAebpq+7OE9BIGKcbMV9ZXQotRZOOxoJyeT5v2l9aYKFLBU
tHO+m8ZOHttRBjChRDfC
-----END TRUSTED CERTIFICATE-----
```

While each node will use the default Java trust store to verify incoming certificates (configurable via `javax.net.ssl.trustStore`), which by default uses the system's root certificates, it's recommended to include the complete certificate chain in the file. These will also be used by each node to verify the other nodes' certificates. **More specifically, the certificate chain will be part of the trust store of the node, and will be used to verify other node's certificates**. That will allow you to configure each node with a different leaf certificate sharing the same root certificate (or at least an intermediate authority), as long as they're contained in the chain. Of course, if all nodes use the same certificate, or if you're certain that the certificate is trusted by the root certificates available on each node, then it's sufficient for the file to only contain the leaf certificate.

The private key file should be a PEM private key file, and should be the one during generation of the node's public certificate. Algorithms supported for the private keys are: RSA, DSA, and EC.

> Note that currently, Zeebe does not support password protected private keys. Since storing the certificates and private keys unencrypted on disk is a security risk, we recommend you use a secret management solution like Vault to inject your certificates in memory at runtime.

## Broker

To configure secure communication for a broker, you will need to configure its `zeebe.broker.network.security` section, which looks like this:

```yaml
security:
    # Enables TLS authentication between this gateway and other nodes in the cluster
    # This setting can also be overridden using the environment variable ZEEBE_BROKER_NETWORK_SECURITY_ENABLED.
    enabled: false

    # Sets the path to the certificate chain file.
    # This setting can also be overridden using the environment variable ZEEBE_BROKER_NETWORK_SECURITY_CERTIFICATECHAINPATH.
    certificateChainPath:

    # Sets the path to the private key file location
    # This setting can also be overridden using the environment variable ZEEBE_BROKER_NETWORK_SECURITY_PRIVATEKEYPATH.
    privateKeyPath:
```

> The `certificateChainPath` and the `privateKeyPath` can be relative to your broker's working directory, or can be absolute paths.


## Gateway

To configure secure communication for a standalone gateway with the rest of the cluster, you will need to configure its `zeebe.gateway.cluster.security` section, which looks like this:

```yaml
security:
    # Enables TLS authentication between this gateway and other nodes in the cluster
    # This setting can also be overridden using the environment variable ZEEBE_GATEWAY_CLUSTER_SECURITY_ENABLED.
    enabled: false

    # Sets the path to the certificate chain file.
    # This setting can also be overridden using the environment variable ZEEBE_GATEWAY_CLUSTER_SECURITY_CERTIFICATECHAINPATH.
    certificateChainPath:

    # Sets the path to the private key file location
    # This setting can also be overridden using the environment variable ZEEBE_GATEWAY_CLUSTER_SECURITY_PRIVATEKEYPATH.
    privateKeyPath:
```

> The `certificateChainPath` and the `privateKeyPath` can be relative to the gateway's working directory, or can be absolute paths.

## Self signed certificates

If you wish to use self signed certificates for testing or development purposes, the simplest way is to have all nodes share the same certificate. As aforementioned, the certificate chain configured on a node is also part of its trust store. As such, if all nodes share the same certificate, they will have no trouble verifying the identity of the other nodes.

Of course, you can still configure a different self signed certificate for each node, _provided that they can be verified by the other nodes' certificate chain_. For example, let's say you have your own root certificate authority you use to sign your own certificates, and one certificate for each node that you signed with that authority. For each node, you can then create a certificate chain file which would consist of the node's public certificate, followed by the root certificate authority's public certificate. Even though each node would have a different leaf certificate it uses to identify itself, the other nodes could verify its identity since their certificate chain contains an authority used to sign it.

### Testing

To generate your own self signed certificates for testing, you must first create a certificate authority.

> NOTE: for this example, whenever you are asked for input, feel free to just press enter and leave the defaults there.

```shell
openssl req -new -newkey rsa:2048 -nodes -out ca.csr -keyout ca.key
openssl x509 -trustout -signkey ca.key -days 365 -req -in ca.csr -out ca.pem
```

Once we have our certificate authority, we can now generate certificates for each node. Let's say we have a cluster of three nodes, `A`, `B`, and `C`.

First, we have to generate a private key for each node.

```shell
openssl genpkey -out nodeA.key -algorithm RSA -pkeyopt rsa_keygen_bits:2048
openssl genpkey -out nodeB.key -algorithm RSA -pkeyopt rsa_keygen_bits:2048
openssl genpkey -out nodeC.key -algorithm RSA -pkeyopt rsa_keygen_bits:2048
```

Then, we have to create a certificate signing request (CSR) for each as well:

```shell
openssl req -new -key nodeA.key -out nodeA.csr
openssl req -new -key nodeB.key -out nodeB.csr
openssl req -new -key nodeC.key -out nodeC.csr
```

Only now can we create the final certificates for each node:

```shell
openssl x509 -req -days 365 -in nodeA.csr -CA ca.pem -CAkey ca.key -set_serial 01 -out nodeA.pem
openssl x509 -req -days 365 -in nodeB.csr -CA ca.pem -CAkey ca.key -set_serial 01 -out nodeB.pem
openssl x509 -req -days 365 -in nodeC.csr -CA ca.pem -CAkey ca.key -set_serial 01 -out nodeC.pem
```

Finally, we now have to create our certificate chain so that each node will be able to verify the identity of the others:

```shell
cat nodeA.pem ca.pem > chainNodeA.pem
cat nodeB.pem ca.pem > chainNodeB.pem
cat nodeC.pem ca.pem > chainNodeC.pem
```

You can now configure each node using its respective final `chainNode*.pem` file and `node*.key` file. For example, if node `A` was a broker:

```yaml
security:
    enabled: true
    certificateChainPath: chainNodeA.pem
    privateKeyPath: nodeA.key
```
