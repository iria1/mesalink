# MesaLink
MesaLink is a memory-safe and OpenSSL-compatible TLS library.

MesaLink and its dependencies are written in [Rust](https://www.rust-lang.org),
a programming language that guarantees memory safety. Furthermore, MesaLink
makes it a breeze to port an existing OpenSSL application, thanks to its
OpenSSL-compatible C APIs. Please refer to the examples to see how MesaLink can
serve as a drop-in replacement for OpenSSL.

## Feature highlights

MesaLink depends on two Rust crates: [rustls](https://github.com/ctz/rustls) and
[ring](https://github.com/briansmith/ring). With them, MesaLink provides the
following features that are considered secure for most use cases:

* TLS 1.2 and TLS 1.3 draft 22
* ALPN and SNI support
* Forced hostname validation
* Safe and fast crypto implementations from Google's BoringSSL
* ECDHE key exchange with forward secrecy
* AES-GCM and Chacha20Poly1305 bulk encryption
* Non-blocking I/O
* Built-in Mozilla's CA root certificates

## Dodged bullets

This section lists a few vulnerabilities that affected other TLS libraries in
2017 but would not be possible in MesaLink.

* [CVE-2017-3730](https://www.cvedetails.com/cve/CVE-2017-3730/) In OpenSSL
  1.1.0 before 1.1.0d, if a malicious server supplies bad parameters for a DHE
  or ECDHE key exchange then this can result in the client attempting to
  dereference a NULL pointer leading to a client crash. This could be exploited
  in a Denial of Service attack.
* [CVE-2017-3735](https://www.cvedetails.com/cve/CVE-2017-3735/): While OpenSSL
  parses an IPAddressFamily extension in an X.509 certificate, it is possible to
  do a one-byte overread.
* [CVE-2017-2784](https://www.cvedetails.com/cve/CVE-2017-2784/): An exploitable
  free of a stack pointer vulnerability exists in the x509 certificate parsing
  code of ARM mbed TLS before 1.3.19, 2.x before 2.1.7, and 2.4.x before 2.4.2.
* [CVE-2017-2800](https://www.cvedetails.com/cve/CVE-2017-2800/): A specially
  crafted x509 certificate can cause a single out of bounds byte overwrite in
  wolfSSL through 3.10.2 resulting in potential certificate validation
  vulnerabilities, denial of service and possible remote code execution.
* [CVE-2017-8854](https://www.cvedetails.com/cve/CVE-2017-8854/): wolfSSL before
  3.10.2 has an out-of-bounds memory access with loading crafted DH parameters,
  aka a buffer overflow triggered by a malformed temporary DH file.

## Building the MesaLink library from source

### To build MesaLink from source, the following tools are needed:

  * autoconf
  * automake
  * libtool
  * curl
  * make
  * gcc
  * rustc
  * cargo

On Ubuntu, you can install them with:
```
$ sudo apt-get install autoconf automake libtool make gcc curl
$ curl https://sh.rustup.rs -sSf | sh
```

If you prefer to build MesaLink with the cutting-edge
[nightly](https://doc.rust-lang.org/1.13.0/book/nightly-rust.html) releases of
Rust, add one more line:

```
$ rustup default nightly
```
On other platforms, please use the corresponding package managing tool to
install them before proceeding. Note that MesaLink always targets the
**current** stable and nightly release of Rust and Cargo. We do not gurantee
backward compatibility with older releases.

### Download the source code from icode:
```
$ git clone --recurse-submodules ssh://jingyiming@icode.baidu.com:8235/baidu/mesalink/mesalink
```

To build and install the MesaLink headers and library, execute the following:
```
$ ./autogen.sh
$ make
$ sudo make install
$ sudo ldconfig
```

## Building the MesaLink documentation
MesaLink uses Rust-style documentation. To generate the documents, execute the following:

```
$ cargo doc --no-deps
```
The documents would be located at `target/doc/mesalink/index.html`


## Examples
MesaLink comes with two examples that demonstrate a TLS client and a TLS
server. They are located at `examples/`.

The client example connects to a remote HTTPS server and prints the server's
response.

```
$ ./examples/client/client api.ipify.org
[+] Requesting api.ipify.org ...
[+] Sent 85 bytes

GET / HTTP/1.1
Host: api.ipify.org
Connection: close
Accept-Encoding: identity

HTTP/1.1 200 OK
Server: Cowboy
Connection: close
Content-Type: text/plain
Vary: Origin
Date: Fri, 15 Dec 2017 19:46:36 GMT
Content-Length: 10
Via: 1.1 vegur

1.2.3.4
[+] Received 177 bytes
```

The server example comes with a pair of certificate and private key. The
certificate file is in the PEM format and contains a chain of certificates from
the server's certificate to the root CA certificate. The private key file
contains a PKCS8-encoded private key in the PEM format. Once the server is up
and running, open [https://127.0.0.1:8443](https://127.0.0.1:8443) and expect to
see the hello message. 

```
$ ./examples/server/server
Usage: ./examples/server/server <portnum> <cert_file> <private_key_file>
$ cd examples/server/server
$ ./server 8443 certificates private_key
[+] Listening at 0.0.0.0:8443
[+] Received:
GET / HTTP/1.1
Host: 127.0.0.1:8443
Connection: keep-alive
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_2) AppleWebKit/537.36 
(KHTML, like Gecko) Chrome/63.0.3239.84 Safari/537.36
Upgrade-Insecure-Requests: 1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9
```

## Want libcurl? No problem!
MesaLink proudly supports libcurl as one of its TLS backends. To compile and
install libcurl with the MesaLink backend, run the following commands:

```
$ git clone https://github.com/kevinis/curl.git
$ cd curl && git checkout mesalink
$ autoreconf -i && ./configure --with-mesalink --without-ssl -without-gtls
$ make && make install
```

We have tested git 2.16.2 linked with MesaLink-powered libcurl and everything goes
well!

## Crypto benchmarks
MesaLink's underlying crypto library is
[**Ring**](https://github.com/briansmith/ring), a safe and fast crypto using
Rust. To evaluate the speed and throughput of MesaLink, we developed new
benchmarks for OpenSSL and wolfSSL based on the
[crypto-bench](https://github.com/briansmith/crypto-bench) project. A summary of
the available benchmarks is shown as follows:

| Benchmark                           | Ring | OpenSSL/LibreSSL | wolfSSL |
| ----------------------------------- | :--: | :--------------: | :-----: |
| SHA-1 & SHA-256 & SHA-512           |  ✔️   |        ✔️         |    ✔️    |
| AES-128-GCM & AES-256-GCM           |  ✔️   |        ✔️         |    ✔️    |
| Chacha20-Poly1305                   |  ✔️   |        ✔️         |    ✔️    |
| ECDH (suite B) key exchange         |  ✔️   |       TODO       |  TODO   |
| X25519 (Curve25519) key exchange    |  ✔️   |       TODO       |  TODO   |

To run the benchmarks, run the following. Note you must have OpenSSL/LibreSSL or
wolfSSL installed to run the corresponding benchmarks.
```
$ git clone https://github.com/kevinis/crypto-bench.git
$ cd crypto-bench && git checkout dev
$ (cd ring && cargo bench)
$ (cd openssl && cargo bench)
$ (cd wolfssl && cargo bench)
```

## Maintainer

 * Yiming Jing <jingyiming@baidu.com>

## License
MesaLink is provided under the 3-Clause BSD license. For a copy, see the LICENSE
file.
