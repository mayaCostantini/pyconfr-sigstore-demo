# PyConFR 2023: "Introduction to Sigstore: cryptographic signatures made easier" demo

This repository hosts the demo of the `sigstore-python` client from the talk "Introduction to Sigstore: cryptographic signatures made easier" presented at PyCon France 2023.

In this demo, we will sign a package with the `sigstore-python` GitHub Action, publish it to PyPI and then verify the signature on download using the client command line interface.

## 1. Running the sign and publish workflow

First ensure you have a PyPI [API token](https://pypi.org/help/#apitoken) configured on your repository fork to upload your package to PyPI on a new release. See the [Encrypted Secrets section](https://docs.github.com/en/actions/security-guides/encrypted-secrets) of the GitHub documentation for adding secrets to your repository.

The workflow is triggered on a new project release. The workflow executes the following steps:
- Generate a `checksums-*.txt` file containing sha256 hashes for all the files contained in the project (`*` stands for a timestamp from the time when the file was generated).
- Sign `checksums-*.txt` and generate Sigstore verification materials: `checksums-*.txt.sig`, `checksums-*.txt.crt`, `checksums-*.txt.bundle`
- Commit and push the newly created files to the repository
- Build the project in a Python package and upload it to PyPI

### Testing the workflow locally

[`nektos/act`](https://github.com/nektos/act) is used to run the GitHub Action workflow locally. Run the command `act` in the repository root to test the action.

**Note:** If `act` has an issue finding the specified Python version for your architecture, run the `act` command with the `-P ubuntu-latest=ghcr.io/catthehacker/ubuntu:act-latest` argument. This workaround will pull a medium-sized image where Python should be installed normally (see the following [`nektos/act` GitHub issue](https://github.com/nektos/act/issues/251) for more information).

## 2. Verifying the project signature

PyPI does not yet support Sigstore signature and verification materials (certificate, bundle) in its API (see the relevant [discussion](https://discuss.python.org/t/pep-694-upload-2-0-api-for-python-package-repositories/16879/35)). Therefore, the second part of the demo will consist in pulling newly created materials from the workflow and verify the signature according to claims specific to GitHub Actions using the [`sigstore-python`](https://github.com/sigstore/sigstore-python) command line.

1. Clone the repository or run `git pull` to get updated `checksums-*` files from the workflow
2. Run the `sigstore verify github` command at the root of the repository, for example with the following arguments:
  - `--cert-identity https://github.com/mayaCostantini/pyconfr-sigstore-demo/.github/workflows/release-to-pypi.yaml@refs/tags/v0.0.13`
    
    This argument corresponds to the Subject Alternative Name (SAN) present in the signing certificate.
    Specify the latest version published for the project and eventually your fork of the repository.
  - `--trigger release`
    
    This is the GitHub event that should have triggered the workflow.
  - `--repository mayaCostantini/pyconfr-sigstore-demo`
    
    The name of the repository on which the workflow was triggered
  - Add the checksums file name at the end of the command: `checksums-*.txt`. The names of the verification materials are inferred from the checksums file name.
  
Example command and output:

```
sigstore verify github --cert-identity https://github.com/mayaCostantini/pyconfr-sigstore-demo/.github/workflows/release-to-pypi.yaml@refs/tags/v0.0.13 --trigger release --repository mayaCostantini/pyconfr-sigstore-demo  checksums-1676301410.txt
OK: checksums-1676301410.txt
```

The `--offline` flag can be passed to the command to verify the signature without connecting to the Rekor instance in which the signature entry was stored with a Sigstore bundle file (`checksums-*.sigstore` file).

See the [`sigstore-python`](https://github.com/sigstore/sigstore-python) documentation for more information on all the arguments available.

    
