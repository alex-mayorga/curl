# HTTP3 (and QUIC)

## Resources

[HTTP/3 Explained](https://daniel.haxx.se/http3-explained/) - the online free
book describing the protocols involved.

[QUIC implementation](https://github.com/curl/curl/wiki/QUIC-implementation) -
the wiki page describing the plan for how to support QUIC and HTTP/3 in curl
and libcurl.

[quicwg.org](https://quicwg.org/) - home of the official protocol drafts

## QUIC libraries

QUIC libraries we're experiementing with:

[ngtcp2](https://github.com/ngtcp2/ngtcp2)

[quiche](https://github.com/cloudflare/quiche)

## Experimental!

HTTP/3 and QUIC support in curl is considered **EXPERIMENTAL** until further
notice. It needs to be enabled at build-time.

Further development and tweaking of the HTTP/3 support in curl will happen in
in the master branch using pull-requests, just like ordinary changes.

# ngtcp2 version

## Build

1. clone ngtcp2 from git (the draft-22 branch)
2. build and install ngtcp2's custom OpenSSL version (the quic-draft-22 branch)
3. build and install nghttp3
4. build and install ngtcp2 according to its instructions
5. configure curl with ngtcp2 support: `./configure --with-ngtcp2=<install prefix>`
6. build curl "normally"

## Running

Make sure the custom OpenSSL library is the one used at run-time, as otherwise
you'll just get ld.so linker errors.

## Invoke from command line

    curl --http3 https://nghttp2.org:8443/

# quiche version

## build

Clone quiche and BoringSSL:

     % git clone https://github.com/cloudflare/quiche
     % cd quiche/
     % mkdir deps
     % cd deps
     % git clone https://github.com/google/boringssl
     % cd boringssl

Build BoringSSL (it needs to be built manually so it can be reused with curl):

     % mkdir build
     % cd build
     % cmake -DCMAKE_POSITION_INDEPENDENT_CODE=on ..
     % make -j`nproc`
     % cd ..
     % mkdir .openssl/lib -p
     % cp build/crypto/libcrypto.a build/ssl/libssl.a .openssl/lib
     % ln -s $PWD/include .openssl

Build quiche:

     % cd ../..
     % QUICHE_BSSL_PATH=$PWD/deps/boringssl cargo build --release

Clone and build curl:

     % cd ..
     % git clone https://github.com/curl/curl
     % cd curl
     % ./buildconf
     % ./configure --with-ssl=$PWD/../quiche/deps/boringssl/.openssl --with-quiche=$PWD/../quiche --enable-debug
     % make -j`nproc`

## Running

Make an HTTP/3 request.

     % src/curl --http3 https://cloudflare-quic.com/
     % src/curl --http3 https://facebook.com/
     % src/curl --http3 https://quic.aiortc.org:4433/
     % src/curl --http3 https://quic.rocks:4433/
