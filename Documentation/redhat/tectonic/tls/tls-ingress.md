# Enabling custom Tectonic Console TLS certificates

Use Tectonic's [Ingress TLS module][ingress-module] to enable user provided Ingress certificates.

The module does not contain any logic, but just passes user provided certificates from its input directly to its output. This is to prevent changing existing references to the `tls/ingress/self-signed` module, hence all `tls/ingress/*` modules share the same outputs.

The certificates provided here secure the TLS endpoint used to access Tectonic Console and Tectonic Identity services. They are not used to secure applications when using the Tectonic Ingress controller.

This certificate must be provided to ensure user access to Tectonic Console's web browser interface without errors.

## Usage

Comment out the existing self-signed ingress TLS in your platform, i.e. `platforms/aws/tectonic.tf`:

```
/*
module "ingress_certs" {
  source = "../../modules/tls/ingress/self-signed"

  base_address = "${module.dns.ingress_internal_fqdn}"
  ca_cert_pem  = "${module.kube_certs.ca_cert_pem}"
  ca_key_alg   = "${module.kube_certs.ca_key_alg}"
  ca_key_pem   = "${module.kube_certs.ca_key_pem}"
}
*/
```

Configure the user provided certificate paths in your platform, i.e. `platforms/aws/tectonic.tf`:

```
module "ingress_certs" {
  source = "../../modules/tls/ingress/user-provided"

  ca_cert_pem_path = "/path/to/ca.crt"
  cert_pem_path    = "/path/to/ingress.crt"
  key_pem_path     = "/path/to/ingress.key"
}
```

All browsers have deprecated the fallback to the `commonName` in the absence of the `subjectAlternativeName` extension. The signed ingress TLS certificate must include the following Subject Alternative Name (SAN) and Key Usage associations:

```
$ openssl x509 -noout -text -in /path/to/ingress.crt
Certificate:

...
        X509v3 extensions:
...
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage:
                TLS Web Server Authentication, TLS Web Client Authentication
...
            X509v3 Subject Alternative Name:
                DNS:<tectonic_cluster_name>.<tectonic_base_domain>
```

Finally, use the generated certificates to boot the cluster with Terraform.


[ingress-module]: https://github.com/coreos/tectonic-installer/tree/master/modules/tls/ingress/
