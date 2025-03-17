# Tiger-PTV6-Proxy

The Tiger PTV6 proxy is placed in front of a connector that meets the PTV5 specification and simulates connector
requests
which meet the PTV6 specification by setting the encryption to be used to ECC in any case, even if previously
was set to RSA. **However, this requires Generation 2.1 cards.**

This is necessary for the following operations:
- SignDocument
- EncryptDocument
- ExternalAuthenticate
- ReadCardCertificate
- CheckCertificateExpiration

## Deployment

### GitHub

The project is located on [GitHub](https://github.com/gematik/tiger-ptv6-proxy).

### Configuration

In order to use the ptv6 functionality copy the *tiger.yaml* of this repository into your project.
Write your connector configuration settings into the file called data.yaml. Place it next to the tiger.yaml.
This looks like this:

```yaml
connector:
  mandantId: mandantId
  workplaceId: wpId
  clientsystemId: csId
  address_konnektor: https://URL_OF_Connector
  port: 12345
  adminport: 12346
```

Now replace the values in the data.yaml with the connector data you chose so that they fit your system.

### Notice: fixed namespace

The ptv6 proxy basically works by ensuring (for most) operations that the crypt tag in the SOAP request is set to the
value
“ECC” (or “RSA_ECC”) is set. If the tag is not set in the SOAP request, it will be inserted.
However, the namespace of this tag is hardcoded, which can cause problems if the rest of the SOAP file uses a different
version.
Therefore, the namespaces used here are described in the following section.

If you want to make sure this is really the namespace in use, check out the *modifications* section
in *tiger.yaml*.

#### SignDocument

It ensures that the `Crypt` tag is set to `RSA_ECC`.  
If the `Crypt` tag is not present, it will be included with the following namespace:
```xml
<Crypt xmlns="http://ws.gematik.de/conn/SignatureService/v7.5">RSA_ECC</Crypt>
```

#### CheckCertificateExpiration

If the `Crypt` tag is not present, it will be inserted with the following namespace:
```xml
<Crypt xmlns="http://ws.gematik.de/conn/CertificateService/v6.0">ECC</Crypt>
```

#### EncryptDocument

Es wird sichergestellt, dass der `Crypt`-Tag auf `ECC` gesetzt wird.  
```xml
<Crypt xmlns="http://ws.gematik.de/conn/EncryptionService/v6.1">ECC</Crypt>
```

#### ReadCardCertificate

If the `Crypt` tag is not present, it will be inserted with the following namespace:
```xml
<Crypt xmlns="http://ws.gematik.de/conn/CertificateService/v6.0">ECC</Crypt>
```

#### ExternalAuthenticate

Regardless of whether the `OptionalInputs` tag is present or not, it will be inserted or replaced with the following
namespace:
```xml
<OptionalInputs xmlns="http://ws.gematik.de/conn/SignatureService/v7.4">
    <SignatureType xmlns="urn:oasis:names:tc:dss:1.0:core:schema">urn:bsi:tr:03111:ecdsa</SignatureType>
</OptionalInputs>
```
