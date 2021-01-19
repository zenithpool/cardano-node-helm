FROM ubuntu:18.04 as base

RUN apt-get update -y && \
 apt-get install automake python3 git curl build-essential pkg-config libffi-dev libgmp-dev libssl-dev libtinfo-dev libsystemd-dev zlib1g-dev make g++ tmux jq wget libncursesw5 gnupg libtool autoconf -y && \
 apt clean all

FROM base as dev

ARG CABAL_VERSION=3.2.0.0
RUN curl -LO "https://downloads.haskell.org/~cabal/cabal-install-$CABAL_VERSION/cabal-install-$CABAL_VERSION-x86_64-unknown-linux.tar.xz" && \
    tar -xf cabal-install-$CABAL_VERSION-x86_64-unknown-linux.tar.xz && \
    rm -rf cabal-install-$CABAL_VERSION-x86_64-unknown-linux.tar.xz cabal.sig && \
    mv cabal /usr/local/bin/ && \
    cabal update
    
ARG GHC_VERSION=8.10.2
RUN curl -LO https://downloads.haskell.org/~ghc/$GHC_VERSION/ghc-$GHC_VERSION-x86_64-deb9-linux.tar.xz && \
    tar -xf ghc-$GHC_VERSION-x86_64-deb9-linux.tar.xz && \
    rm ghc-$GHC_VERSION-x86_64-deb9-linux.tar.xz && \
    cd ghc-$GHC_VERSION && \
    ./configure && \
    make install && \
    rm -rf /ghc-$GHC_VERSION

ARG LIBSODIUM=66f017f1
RUN git clone https://github.com/input-output-hk/libsodium && \
    cd libsodium && \
    git checkout $LIBSODIUM && \
    ./autogen.sh && \
    ./configure && \
    make && \
    make install

ENV LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH

ARG CARDANO_NODE_VERSION=tags/pioneer-3
RUN git clone https://github.com/input-output-hk/cardano-node.git && \
    cd cardano-node && \
    git fetch --all --recurse-submodules --tags && \
    git checkout $CARDANO_NODE_VERSION && \
    cabal configure --with-compiler=ghc-$GHC_VERSION && \
    echo "package cardano-crypto-praos" >>  cabal.project.local && \
    echo "  flags: -external-libsodium-vrf" >>  cabal.project.local && \
    cabal clean && \
    cabal build all

FROM base

COPY --from=dev /usr/local/lib /usr/local/lib
ENV LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
ARG GHC_VERSION=8.10.2
ARG CARDANO_NODE_VERSION=tags/pioneer-3
COPY --from=dev /cardano-node/dist-newstyle/build/x86_64-linux/ghc-${GHC_VERSION}/cardano-cli-${CARDANO_NODE_VERSION}/x/cardano-cli/build/cardano-cli/cardano-cli /usr/local/bin
COPY --from=dev /cardano-node/dist-newstyle/build/x86_64-linux/ghc-${GHC_VERSION}/cardano-node-${CARDANO_NODE_VERSION}/x/cardano-node/build/cardano-node/cardano-node /usr/local/bin
WORKDIR /genesis
COPY byron-genesis.json .
COPY shelley-genesis.json .
WORKDIR /
RUN mkdir -p /data/db
RUN mkdir -p /ipc
