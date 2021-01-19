# Cardano Node Helm Chart

# Pre reqs
- helm 3
- kubernetes cluster 1.16+

# Current state
- no persistent data/volumes. All volumes use emptyDir that can be configured

# Setup

## Step 0 Create Secrets for Postgres
- helm install cardano-secrets ./helm/cardano-secrets

## Step 1 Deploy Prometheus
- helm install prometheus ./helm/prometheus

## Step 2 Deploy Grafana
- helm install grafana ./helm/grafana

## Step 3 Deploy Postgres
- helm install postgres ./helm/postgres

## Step 4 Deploy cardano-node
- helm install cardano-node ./helm/cardano-node
