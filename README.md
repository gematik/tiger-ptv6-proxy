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

Some methods that are not supported by the connector yet are mocked by the proxy:
- StartCardSession
- StopCardSession
- SecureSendAPDU

For now, these methods return static mock responses that are served by a Zion server.

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
To use the mocked responses for the methods that are not supported by the connector, you need to also copy the files that contain the mocked responses.
Alternatively, you can also provide your own mock responses and update the configuration to point to the correct files.

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

### Standalone
In order to start the ptv6 proxy standalone, you can use the following command:
```shell
mvn tiger:up
```

### License

Copyright 2025 gematik GmbH

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License.

See the [LICENSE](./LICENSE) for the specific language governing permissions and limitations under the License.

### Additional Notes and Disclaimer from gematik GmbH

1. Copyright notice: Each published work result is accompanied by an explicit statement of the license conditions for use. These are regularly typical conditions in connection with open source or free software. Programs described/provided/linked here are free software, unless otherwise stated.
2. Permission notice: Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions::
    1. The copyright notice (Item 1) and the permission notice (Item 2) shall be included in all copies or substantial portions of the Software.
    2. The software is provided "as is" without warranty of any kind, either express or implied, including, but not limited to, the warranties of fitness for a particular purpose, merchantability, and/or non-infringement. The authors or copyright holders shall not be liable in any manner whatsoever for any damages or other claims arising from, out of or in connection with the software or the use or other dealings with the software, whether in an action of contract, tort, or otherwise.
    3. The software is the result of research and development activities, therefore not necessarily quality assured and without the character of a liable product. For this reason, gematik does not provide any support or other user assistance (unless otherwise stated in individual cases and without justification of a legal obligation). Furthermore, there is no claim to further development and adaptation of the results to a more current state of the art.
3. Gematik may remove published results temporarily or permanently from the place of publication at any time without prior notice or justification.
4. Please note: Parts of this code may have been generated using AI-supported technology.’ Please take this into account, especially when troubleshooting, for security analyses and possible adjustments.
