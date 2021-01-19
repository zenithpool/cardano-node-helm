# Cardano Node Helm Chart

# Pre reqs
- helm 3
- kubectl v1.17+
- kubernetes cluster 1.17+

# Current state
- no persistent data/volumes. All volumes use emptyDir that can be configured

# Setup

## Step 0 Setup Docker Image
 - Because Kubernetes has a limit on the size of attached configuration files. We need to build the docker image with the configuration files like the genesis files for Byron and Shelly.
 - For simple setup, set the remote repo to public so that you do not need to setup private registry secrets in your kubernetes cluster.
```
$ export VER=1.24.2
$ docker build -t <custom-image-repo>/cardano-node:${VER} --build-arg CARDANO_NODE_VERSION=${VER} ./cardano-node
```

# Setup 1 Deploy cardano node as relay

```
$ export VER=1.24.2
$ helm upgrade cardano-relay ./helm/cardano-node --install \
    --namespace default \
    --set cardano_node.image.repository="<custom-docker-registry-repo>/cardano-node" \
    --set cardano_node.image.tag=${VER} \
    --set cardano_node.relay.port=31400 \
    --set cardano_node.resources.requests.cpu=1 \
    --set cardano_node.resources.limits.cpu=2 \
    --set cardano_node.resources.requests.memory=2Gi \
    --set cardano_node.resources.limits.memory=6Gi \
    --set cardano_node.storageClass="none" \
    --set replicas=1
```

## Exposing the cardano node relay
 - By default the cardano node relay is only exposed from within the cluster. To expose it to the public, you can expose it as a NodePort or LoadBalancer
 ```
$ export VER=1.24.2
$ helm upgrade cardano-relay ./helm/cardano-node --install \
    --namespace default \
    --set cardano_node.image.repository="<custom-docker-registry-repo>/cardano-node" \
    --set cardano_node.image.tag=${VER} \
    --set cardano_node.relay.port=31400 \
    --set cardano_node.resources.requests.cpu=1 \
    --set cardano_node.resources.limits.cpu=2 \
    --set cardano_node.resources.requests.memory=2Gi \
    --set cardano_node.resources.limits.memory=6Gi \
    --set cardano_node.storageClass="none" \
    --set cardano_node.service.type="NodePort" \
    --set replicas=1
```

 # Setup 2 Deploy cardano node as a block producer
 TODO