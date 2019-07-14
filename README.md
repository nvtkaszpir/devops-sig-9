# prometheus-operator

## About

This allows to create prometheus instances on demand using k8s CRD.

## Steps

1. Prepare base setup for demo
2. Use helm chart prometheus-operator with default setup.
3. Create example services and expand prometheus to gather
metrics from those new services.
4. Create deficated pmetheus instance with operator, so that
we can create customized prometheus instance to our needs.

## References

[Prometheus Operator - Getting started](https://github.com/coreos/prometheus-operator/blob/master/Documentation/user-guides/getting-started.md)

## 1. Base setup

- get k8s cluster, for example may be used with KIND:

  ```shell
  kind create cluster
  export KUBECONFIG="$(kind get kubeconfig-path --name="kind")"
  kubectl cluster-info
  ```

- install [helm client 2.14.2](https://github.com/helm/helm/releases) via
  [install script](https://raw.githubusercontent.com/helm/helm/master/scripts/get)
- install helm on k8s

  ```shell
  kubectl apply -f k8s/helm-tiller.rbac.yaml
  helm init --service-account tiller
  ```

## 2. Install prometheus with helm chart prometheus-operator

- install prometheus-operator with basic config

  ```shell
  helm install -n po stable/prometheus-operator -f helm/po.basic.yaml
  kubectl --namespace default get pods -l "release=po"
  ```

- access default prometheus web interface

  ```shell
  kubectl port-forward prometheus-promop-prometheus-0 9091:9090
  ```

  [default url](http://127.0.0.1:9091)

- investigate Services, Targets in prometheus web interface.

## 3. Creating example services

- create example app as service

  ```shell
  kubectl apply -f etcd/cluster-role.yaml
  kubectl apply -f etcd/cluster-role-binding.yaml
  kubectl apply -f etcd/etcd-operator.yaml

  kubectl apply -f etcd/crd.cluster.etcd.yaml
  ```

- see new pods, services, endpoints

## 3.1 Add service to montior via prometheus helm

- upgrade helm chart to scrape services

    ```shell
    helm upgrade  po stable/prometheus-operator \
      -f helm/po.extended.yaml \
      --description 'Add service scraping to prometheus'
    ```

- wait about 1 minute, see as prometheus shows new manages services,
  `etcd-operator` and `etcd-cluster`

- revert the changes

  ```shell
  helm rollback po 1
  ```

- wait about 1 minute, see as prometheus shows new manages services

## 3.2 Add service to monitor via prometheus CRD

- add ServiceMonitor for our app so that it is processed by main prometheus

  ```shell
  kubectl apply -f etcd/promop.sm.yaml
  ```

- wait 1 minute, see that prometheus config changed,
  you should see `etcd-cluster-generic` in Service Discovery

## 4. Create dedicated prometheus instance

- create dedicated prometheus instance just for our app

  ```shell
  kubectl apply -f etcd/prom-etcd.yaml
  ```

- see how pods are created
- investigate how labels and label selectors match
- connect to dedicated prometheus instance

  ```shell
  kubectl port-forward prometheus-etcd-0 9092:9090
  ```

- notice how rules and alertmanager is missing

## TODO

- create simple rule
- look into alertmanager secret

