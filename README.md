# QPKI

**Post-Quantum X.509 PKI toolkit**

[![Release](https://img.shields.io/github/v/release/qpki/qpki)](https://github.com/qpki/qpki/releases)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

QPKI is a quantum-safe PKI toolkit to help organizations prepare for post-quantum cryptography (PQC) with interoperable, standards-compliant certificates.

> **For education and prototyping** — Learn PKI concepts, experiment with PQC migration, and test crypto-agility. See [Qlab](https://github.com/qpki/qlab) for step-by-step tutorials.

## Features

- **State-of-the-art X.509 certificates** (RFC 5280 compliant)
- **Post-Quantum Cryptography (PQC)** support via ML-DSA, SLH-DSA and ML-KEM
- **CSR generation** for all algorithms including RFC 9883 ML-KEM attestation
- **Catalyst certificates** (ITU-T X.509 Section 9.8) - dual keys via extensions
- **Composite certificates** (IETF draft-13, **DRAFT**) - dual keys bound together
- **Hybrid certificates** (classical + PQC via combined or separate modes)
- **SSH Certificates** (OpenSSH format) - user and host certificate issuance (classical algorithms)
- **CMS Signatures & Encryption** (RFC 5652) - sign and encrypt with PQC
- **Crypto-agility** - seamless migration between algorithms (ECDSA -> ML-DSA)
- **Profiles** (certificate templates) - define certificate policies in YAML
- **Credentials** - group certificates with coupled lifecycle
- **HSM support** via PKCS#11
- **Cross-validated** with external implementations (OpenSSL, BouncyCastle)
- **CLI-first** - simple, scriptable, no database required
- **PQC via [Cloudflare CIRCL](https://github.com/cloudflare/circl)** — FIPS 203/204/205 implementations, NIST ACVP test vectors validated
- **Pure Go by default** - CGO optional (only for HSM/PKCS#11)

## Supported Algorithms

### Classical
| Algorithm | Security | Notes |
|-----------|----------|-------|
| ECDSA (P-256, P-384, P-521) | ~128/192/256-bit | NIST curves, P-384 recommended |
| EdDSA (Ed25519, Ed448) | ~128/224-bit | Fast, constant-time |
| RSA (2048, 4096) | ~112/140-bit | Legacy compatibility |

*EC keys support both ECDSA (signature) and ECDH (key agreement) depending on certificate keyUsage.*

### Post-Quantum
| Algorithm | Security | Notes |
|-----------|----------|-------|
| ML-DSA-44/65/87 | NIST Level 1/3/5 | FIPS 204, lattice-based |
| SLH-DSA-128/192/256 | NIST Level 1/3/5 | FIPS 205, hash-based |
| ML-KEM-512/768/1024 | NIST Level 1/3/5 | FIPS 203, key encapsulation |

*Classical security levels reflect resistance to classical attacks only. Post-quantum algorithms are designed to remain secure against quantum adversaries.*

## Installation

### Quick Install (recommended)

**Linux / macOS:**
```bash
curl -sSL get.qpki.io | sh
```

**Windows (PowerShell):**
```powershell
irm https://qpki.io/install.ps1 | iex
```

### Download pre-built binaries

Download the latest release for your platform from [GitHub Releases](https://github.com/qpki/qpki/releases/latest).

**Linux / macOS:**
```bash
# Download (replace VERSION, OS, and ARCH as needed)
curl -LO https://github.com/qpki/qpki/releases/latest/download/qpki_VERSION_OS_ARCH.tar.gz

# Extract
tar -xzf qpki_*.tar.gz

# Install
sudo mv qpki /usr/local/bin/

# Verify
qpki --version
```

**Available platforms:**
| OS | Architecture | File |
|----|--------------|------|
| Linux | amd64 | `qpki_VERSION_linux_amd64.tar.gz` |
| Linux | arm64 | `qpki_VERSION_linux_arm64.tar.gz` |
| macOS | Intel | `qpki_VERSION_darwin_amd64.tar.gz` |
| macOS | Apple Silicon | `qpki_VERSION_darwin_arm64.tar.gz` |
| macOS | Universal | `qpki_VERSION_darwin_all.tar.gz` |
| Windows | amd64 | `qpki_VERSION_windows_amd64.zip` |

**Linux packages:**
```bash
# Debian/Ubuntu
sudo dpkg -i qpki_VERSION_linux_amd64.deb

# RHEL/Fedora
sudo rpm -i qpki_VERSION_linux_amd64.rpm
```

### Install via Homebrew (macOS)

```bash
brew tap qpki/qpki
brew install qpki
```

### Verify release signatures

All releases are signed with GPG. To verify:

```bash
# Import public key
gpg --keyserver keyserver.ubuntu.com --recv-keys 39CD0BF9647E3F56

# Download checksums and signature
curl -LO https://github.com/qpki/qpki/releases/download/vX.Y.Z/checksums.txt
curl -LO https://github.com/qpki/qpki/releases/download/vX.Y.Z/checksums.txt.sig

# Verify signature
gpg --verify checksums.txt.sig checksums.txt
```

## Quick Start

### Initialize a Root CA

```bash
# Create a CA with ECDSA P-384 (recommended)
qpki ca init --profile ec/root-ca --ca-dir ./root-ca --var cn="My Root CA"

# Create a hybrid CA (ECDSA + ML-DSA, ITU-T X.509 Section 9.8)
qpki ca init --profile hybrid/catalyst/root-ca --ca-dir ./hybrid-ca --var cn="Hybrid Root CA"

# Create a pure PQC CA (ML-DSA-87)
qpki ca init --profile ml/root-ca --ca-dir ./pqc-ca --var cn="PQC Root CA"
```

### Create a Subordinate CA

```bash
# Create a subordinate/issuing CA signed by the root
qpki ca init --profile ec/issuing-ca --ca-dir ./issuing-ca \
  --parent ./root-ca --var cn="Issuing CA"
```

### Generate Keys

```bash
# Generate an ECDSA key
qpki key generate --algorithm ecdsa-p256 --out key.pem

# Generate an ML-DSA-65 (PQC lattice-based) key
qpki key generate --algorithm ml-dsa-65 --out ml-dsa-key.pem

# Extract public key from private key
qpki key pub --key key.pem --out key.pub
```

### Issue Certificates

```bash
# Generate CSR + key
qpki csr gen --algorithm ecdsa-p256 --keyout server.key --cn server.example.com --out server.csr

# Issue certificate from CSR
qpki cert issue --ca-dir ./myca --profile ec/tls-server \
  --csr server.csr --out server.crt \
  --var cn=api.example.com \
  --var dns_names=api.example.com,api-v2.example.com
```

### Inspect & Verify

```bash
# Show certificate details
qpki inspect certificate.crt

# Verify certificate chain
qpki cert verify server.crt --ca ./myca/ca.crt
```

### Sign & Encrypt with CMS

```bash
# Sign a document (detached signature)
qpki cms sign --data doc.pdf --cert signer.crt --key signer.key --out doc.p7s

# Encrypt for recipient (supports ECDH, RSA, ML-KEM)
qpki cms encrypt --recipient bob.crt --in secret.txt --out secret.p7m
```

## Documentation

Full documentation is available at **[qpki.io](https://qpki.io)**.

## About

Maintained by [@remiblancher](https://github.com/remiblancher).

- Questions & feedback: [GitHub Issues](https://github.com/qpki/qpki/issues)
- Contact: remi.blancher@proton.me

## License

Apache License 2.0 - See [LICENSE](LICENSE) for details.
