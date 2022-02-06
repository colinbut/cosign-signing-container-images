# cosign-signing-container-images

Outlines the steps on signing and verifying container images using a tool called [cosign](https://github.com/sigstore/cosign).

## Process
The overall process is as follows:

1. Generate a (private/public) key pair
2. Sign the container image with the private key & store the signature in the container registry
3. locate the signatures for container images and verify them using known public key

## Usage Workflow

Install Cosign:
```bash
❯ brew install sigstore/tap/cosign
```

One of the first things that needs to be done is to generate a key pair:

```bash
❯ cosign generate-key-pair
Enter password for private key:
Enter again:
Private key written to cosign.key
Public key written to cosign.pub
```

Sign the docker image:
```bash
❯ cosign sign --key cosign.key colinbut/test-image:latest
Enter password for private key:
```

Locating the cosign image reference from the container registry:

```bash
❯ cosign triangulate colinbut/test-image:latest
index.docker.io/colinbut/test-image:sha256-3b935143ff0ed20de1827c4d1409df6179ad351924a085b16900ca9cb5e556dc.sig
```

Lastly, verifying the docker image against the public key and verify the signature's authenticity.

```bash
❯ cosign verify --key cosign.pub colinbut/test-image:latest

Verification for index.docker.io/colinbut/test-image:latest --
The following checks were performed on each of these signatures:
  - The cosign claims were validated
  - The signatures were verified against the specified public key
  - Any certificates were verified against the Fulcio roots.

[{"critical":{"identity":{"docker-reference":"index.docker.io/colinbut/test-image"},"image":{"docker-manifest-digest":"sha256:3b935143ff0ed20de1827c4d1409df6179ad351924a085b16900ca9cb5e556dc"},"type":"cosign container image signature"},"optional":null}]
```

## Signing SBOM

Cosign allows signing of sboms. Sbom is Software Bill of Materials which is a list of components/software libraries (application & OS) that gets bundled as part of the software artifact.

Using another tool - [Syft](https://github.com/anchore/syft) - that is specifically designed for generating sboms as part of the process:

e.g.
```bash
❯ syft packages colinbut/test-image:latest --scope all-layers -o spdx > test-image.spdx
```

The next step is to attach the sbom to the container image:

```bash
❯ cosign attach sbom --sbom test-image.spdx colinbut/test-image:latest
Uploading SBOM file for [index.docker.io/colinbut/test-image:latest] to [index.docker.io/colinbut/test-image:sha256-3b935143ff0ed20de1827c4d1409df6179ad351924a085b16900ca9cb5e556dc.sbom] with mediaType [text/spdx].
```

Lastly, signing the sbom in the registry and checking it:

```bash
❯ cosign sign --key cosign.key index.docker.io/colinbut/test-image:sha256-3b935143ff0ed20de1827c4d1409df6179ad351924a085b16900ca9cb5e556dc.sbom
```

```bash
❯ cosign verify --key cosign.pub index.docker.io/colinbut/test-image:sha256-3b935143ff0ed20de1827c4d1409df6179ad351924a085b16900ca9cb5e556dc.sbom

Verification for index.docker.io/colinbut/test-image:sha256-3b935143ff0ed20de1827c4d1409df6179ad351924a085b16900ca9cb5e556dc.sbom --
The following checks were performed on each of these signatures:
  - The cosign claims were validated
  - The signatures were verified against the specified public key
  - Any certificates were verified against the Fulcio roots.

[{"critical":{"identity":{"docker-reference":"index.docker.io/colinbut/test-image"},"image":{"docker-manifest-digest":"sha256:30e6c2193fa68b86a1c132330106efa7933bf9d4cd2ba7679a1b015db21e7d47"},"type":"cosign container image signature"},"optional":null}]
```

## Authors

Colin But.
