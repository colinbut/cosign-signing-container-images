# cosign-signing-container-images


Install Cosign:
```bash
brew install sigstore/tap/cosign
```

One of the first things that needs to be done is to generate a key pair:

```bash
$ cosign generate-key-pair
Enter password for private key:
Enter again:
Private key written to cosign.key
Public key written to cosign.pub
```

Sign the docker image:
```bash
$ cosign sign --key cosign.key colinbut/test-image:latest
Enter password for private key:
```

Locating the cosign image reference from the container registry:

```bash
```

## Authors

Colin But.
