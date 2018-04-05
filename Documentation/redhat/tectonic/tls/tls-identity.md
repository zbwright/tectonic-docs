# Enabling custom Tectonic Identity TLS certificates

Use Tectonic's [Identity TLS module][identity-module] to enable user provided Identity certificates.

The module does not contain any logic, but just passes user provided certificates from its input directly to its output. This is to prevent changing existing references to the `tls/identity/self-signed` module, hence all `tls/identity/*` modules share the same outputs.

## Usage

Comment out the existing self-signed identity TLS in your platform, i.e. `platforms/aws/tectonic.tf`:

```
/*
module "identity_certs" {
  source = "../../modules/tls/identity/self-signed"

  ca_cert_pem = "${module.kube_certs.ca_cert_pem}"
  ca_key_alg  = "${module.kube_certs.ca_key_alg}"
  ca_key_pem  = "${module.kube_certs.ca_key_pem}"
}
*/
```

Configure the user provided certificate paths in your platform, i.e. `platforms/aws/tectonic.tf`.

This certificate is used by Tectonic Console to communicate with Tectonic Identity's gRPC endpoint, securing communication between Console and the Tectonic Identity. The gRPC endpoint is provided for use in revoking tokens. Securing this channel ensures that tokens generated for cluster access are always secured.

```
module "identity_certs" {
  source = "../../modules/tls/identity/user-provided"

  client_key_pem_path  = "/path/to/identity-client.key"
  client_cert_pem_path = "/path/to/identity-client.crt"
  server_key_pem_path  = "/path/to/identity-server.key"
  server_cert_pem_path = "/path/to/identity-server.crt"
}
```

The signed identity client certificate must have the following Key Usage associations:

```
$ openssl x509 -noout -text -in /path/to/identity-client.crt
Certificate:
...
        X509v3 extensions:
            X509v3 Extended Key Usage:
                TLS Web Client Authentication
```

The signed identity server certificate must have the following Key Usage associations:

```
$ openssl x509 -noout -text -in /path/to/identity-server.crt
Certificate:
...
        X509v3 extensions:
            X509v3 Extended Key Usage:
                TLS Web Server Authentication
```

Note that the identity certificates listed above must use the same CA as the `modules/tls/kube` certificates.

Finally, use the generated certificates to boot the cluster with Terraform.


[identity-module]: https://github.com/coreos/tectonic-installer/tree/master/modules/tls/identity/
