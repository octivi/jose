[![build](https://github.com/latchset/jose/workflows/build/badge.svg)](https://github.com/latchset/jose/actions)

# Welcome to José!

José is a C-language implementation of the Javascript Object Signing and
Encryption standards. Specifically, José aims towards implementing the
following standards:

  * RFC 7515 - JSON Web Signature (JWS)
  * RFC 7516 - JSON Web Encryption (JWE)
  * RFC 7517 - JSON Web Key (JWK)
  * RFC 7518 - JSON Web Algorithms (JWA)
  * RFC 7519 - JSON Web Token (JWT)
  * RFC 7520 - Examples of ... JOSE
  * RFC 7638 - JSON Web Key (JWK) Thumbprint

José is extensively tested against the RFC test vectors.

# Supported Algorithms

| Algorithm          | Supported | Algorithm Type | JWK Type |
|--------------------|:---------:|:--------------:|:--------:|
| HS256              |    YES    |   Signature    |    oct   |
| HS384              |    YES    |   Signature    |    oct   |
| HS512              |    YES    |   Signature    |    oct   |
| RS256              |    YES    |   Signature    |    RSA   |
| RS384              |    YES    |   Signature    |    RSA   |
| RS512              |    YES    |   Signature    |    RSA   |
| ES256              |    YES    |   Signature    |     EC   |
| ES384              |    YES    |   Signature    |     EC   |
| ES512              |    YES    |   Signature    |     EC   |
| ES256K             |    YES    |   Signature    |     EC   |
| PS256              |    YES    |   Signature    |    RSA   |
| PS384              |    YES    |   Signature    |    RSA   |
| PS512              |    YES    |   Signature    |    RSA   |
| none               |     NO    |   Signature    |    N/A   |
| RSA1_5             |    YES    |   Key Wrap     |    RSA   |
| RSA-OAEP           |    YES    |   Key Wrap     |    RSA   |
| RSA-OAEP-256       |    YES    |   Key Wrap     |    RSA   |
| A128KW             |    YES    |   Key Wrap     |    oct   |
| A192KW             |    YES    |   Key Wrap     |    oct   |
| A256KW             |    YES    |   Key Wrap     |    oct   |
| dir                |    YES    |   Key Wrap     |    oct   |
| ECDH-ES            |    YES    |   Key Wrap     |     EC   |
| ECDH-ES+A128KW     |    YES    |   Key Wrap     |     EC   |
| ECDH-ES+A192KW     |    YES    |   Key Wrap     |     EC   |
| ECDH-ES+A256KW     |    YES    |   Key Wrap     |     EC   |
| A128GCMKW          |    YES    |   Key Wrap     |    oct   |
| A192GCMKW          |    YES    |   Key Wrap     |    oct   |
| A256GCMKW          |    YES    |   Key Wrap     |    oct   |
| PBES2-HS256+A128KW |    YES    |   Key Wrap     |    N/A   |
| PBES2-HS384+A192KW |    YES    |   Key Wrap     |    N/A   |
| PBES2-HS512+A256KW |    YES    |   Key Wrap     |    N/A   |
| A128CBC-HS256      |    YES    |   Encryption   |    oct   |
| A192CBC-HS384      |    YES    |   Encryption   |    oct   |
| A256CBC-HS512      |    YES    |   Encryption   |    oct   |
| A128GCM            |    YES    |   Encryption   |    oct   |
| A192GCM            |    YES    |   Encryption   |    oct   |
| A256GCM            |    YES    |   Encryption   |    oct   |

# José Command-Line Utility
José provides a command-line utility which encompasses most of the JOSE
features. This allows for easy integration into your project and one-off
scripts. Below you will find examples of the common commands.

### Key Management

José can generate keys, remove private keys and show thumbprints. For example:

```sh
# Generate three different kinds of keys
$ jose jwk gen -i '{"alg": "A128GCM"}' -o oct.jwk
$ jose jwk gen -i '{"alg": "RSA1_5"}' -o rsa.jwk
$ jose jwk gen -i '{"alg": "ES256"}' -o ec.jwk

# Remove the private keys
$ jose jwk pub -i oct.jwk -o oct.pub.jwk
$ jose jwk pub -i rsa.jwk -o rsa.pub.jwk
$ jose jwk pub -i ec.jwk -o ec.pub.jwk

# Calculate thumbprints
$ jose jwk thp -i oct.jwk
9ipMcxQLsI56Mqr3yYS8hJguJ6Mc8Zh6fkufoiKokrM
$ jose jwk thp -i rsa.jwk
rS6Yno3oQYRIztC6np62nthbmdydhrWmK2Zn_Izmerw
$ jose jwk thp -i ec.jwk
To8yMD92X82zvGoERAcDzlPP6awMYGM2HYDc1G5xOtc
```

### Signatures
José can sign and verify data. For example:

```sh
$ echo hi | jose jws sig -i- -k ec.jwk -o msg.jws
$ jose jws ver -i msg.jws -k ec.pub.jwk
hi
$ jose jws ver -i msg.jws -k oct.jwk
No signatures validated!
```

### Encryption
José can encrypt and decrypt data. For example:

```sh
$ echo hi | jose jwe enc -i- -k rsa.pub.jwk -o msg.jwe
$ jose jwe dec -i msg.jwe -k rsa.jwk
hi
$ jose jwe dec -i msg.jwe -k oct.jwk
Decryption failed!
```

# Building and Installing from Source
Building Jose is fairly straightforward:

    $ mkdir build && cd build
    $ meson setup .. --prefix=/usr
    $ ninja
    $ sudo ninja install

You can even run the tests if you'd like:

    $ meson test

To build a FreeBSD, HardenedBSD or OPNsense package
use:

    (as root) # pkg install meson pkgconf jansson openssl asciidoc jq

    $ mkdir build && cd build
    $ meson setup .. --prefix=/usr/local
    $ ninja
    $ meson test
    (as root) # ninja install

Once built it does not require meson and pkgconf,
but still requires jansson and openssl.

## Static Linking

### Static Building libjose Library

By default only shared `libjose` library is built.

You can build static version in two ways:

1. Building both a static and shared library. Source files will be compiled
only once and object files will be reused to build both shared and static
libraries:
    ```bash
    meson setup .. --prefix=/usr/local -Ddefault_library=both
    ```

2. Building only a static library:
    ```bash
    meson setup .. --prefix=/usr/local -Ddefault_library=static
    ```

### Static Linking with libjose Library

The `libjose` library uses GCC/Clang's constructor attribute for plugin
registration. For example, in [`lib/openssl/ec.c`](lib/openssl/ec.c):

```c
static void __attribute__((constructor)) constructor(void)
```

When statically linking with `libjose`, you must link all contents of the
library, regardless of whether they are used or not. This requirement exists
because of how constructor attributes work in static linking.

You can achieve this in two ways:

1. Using GCC/Clang linker flags:
    ```bash
    -Wl,-whole-archive
    ```

2. Using Meson build option:
    ```meson
    link_whole: libjose_lib
    ```

For more information, see:
- [Issue #41: Static linking problems](https://github.com/latchset/jose/issues/41)
- [Stack Overflow: GCC constructor attribute linking](https://stackoverflow.com/questions/6589772/gcc-functions-with-constructor-attribute-are-not-being-linked)
- [`./cmd/meson.build`](cmd/meson.build)

### jose Command-Line Tool

To build the `jose` command-line tool with static linking it is required
to force build static library.

```bash
meson setup .. --prefix=/usr -Ddefault_library=static 
```

By default, the command-line tool `jose` is statically linked only with
`libjose`. Other libraries are linked dynamically (e.g. `libjansson`).
