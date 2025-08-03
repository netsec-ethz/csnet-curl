# csnet-curl

This a fork of [curl](https://github.com/curl/curl) that enables HTTP/3 over SCION using [csnet](https://github.com/scionproto-contrib/csnet).

## Requirements

1. Install a release build of csnet: [csnet - Building and Installation](https://github.com/scionproto-contrib/csnet?tab=readme-ov-file#building-and-installation)
2. Install an SSL backend that works with ngtcp2/curl (e.g, [quictls](https://github.com/quictls/openssl/tree/OpenSSL_1_1_1w+quic))
3. Install ngtcp2 with:
    ```bash
   git clone --recursive https://github.com/ngtcp2/ngtcp2.git
   cd ngtcp2
   cmake -DBUILD_TESTING=OFF -DENABLE_LIB_ONLY=ON -DOPENSSL_ROOT_DIR=<QUICTLS_INSTALL_DIR> -B cmake-build
   cmake --build cmake-build
   sudo cmake --install cmake-build/
    ```
4. Install nghttp3 with:
    ```bash
   git clone --recursive https://github.com/ngtcp2/nghttp3
   cd nghttp3
   cmake -DENABLE_LIB_ONLY=ON -B cmake-build
   cmake --build cmake-build
   sudo cmake --install cmake-build/
    ```
5. Install libpsl with:
    ```bash
   sudo apt install libpsl-dev
    ```
6. Install the [SCION example HTTP/3 server](https://github.com/netsec-ethz/scion-apps/pull/277) with:
    ```bash
   sudo apt-get install -y libpam0g-dev
   git clone https://github.com/koflin/scion-apps.git
   cd scion-apps
   git checkout shttp3-server-example
   make setup_lint
   make example-shttp3-server
   openssl req -x509 -newkey rsa -nodes -keyout server.key -out server.cert
   ```

## Building

To build the curl fork run:
```bash
cmake -DCMAKE_BUILD_TYPE=Debug \
      -DCMAKE_INSTALL_RPATH=$ORIGIN/../lib \
      -DBUILD_STATIC_CURL=ON \
      -DBUILD_STATIC_LIBS=ON \
      -DBUILD_SHARED_LIBS=OFF \
      -DUSE_NGTCP2=ON \
      -DUSE_SCION=ON \
      -DSCION_INCLUDE_DIR=<CSNET_INSTALL_DIR>/include \
      -DSCION_LIBRARY_DIR=<CSNET_INSTALL_DIR>/lib \
      -DOPENSSL_ROOT_DIR=<QUICTLS_INSTALL_DIR> \
      -B cmake-build
      
cmake --build cmake-build
```

## Running the Example

1. **Start the local SCION network**  
   In the cloned `csnet` repository:
   ```bash
   sudo ./scripts/run-testnet.sh
   ```

2. **Start the SCION example HTTP/3 server**  
   In the cloned `scion-apps` repository:
   ```bash
   sudo SCION_DAEMON_ADDRESS="127.0.0.133:30255" \
   ./bin/example-shttp3-server -cert server.cert -key server.key
   ```
   > Note: The warnings `connection doesn't allow setting of receive buffer size` and `ERROR Unable to extract port from listener` are expected and can be ignored.

3. **Custom Topology**  
   If you are not using the default topology, overwrite the `topology.json` file in this repository with your modified version.

4. **Make an HTTP/3 SCION request**

   - **Using the example program**
     ```bash
     cmake --build cmake-build --target curl-example-http3-scion
     ./cmake-build/docs/examples/http3-scion
     ```

   - **Using the command-line curl tool**
     ```bash
     ./cmake-build/src/curl \
     --scion-dst-ia "2-ff00:0:221" \
     --scion-topology-path "topology.json" \
     --http3-only \
     --insecure \
     https://127.0.0.132/json
     ```