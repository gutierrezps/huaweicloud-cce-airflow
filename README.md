# AirFlow on Huawei Cloud CCE

This repository contains the files and instructions required to deploy AirFlow
in a [Huawei Cloud Container Engine (CCE)][cce] cluster, using the
[official AirFlow Helm Chart][airflow-helm].

## Requirements

- CCE Cluster with one or more CCE nodes
- [kubectl][kubectl] installed and [configured to connect to the target CCE Cluster][kubectl-cce]
- [helm][helm] installed

## Installation

<!-- markdownlint-disable MD029 -->

1. [Create two Parallel File Systems][pfs-create], one for DAGs and other for
   logs;
2. Create an IAM User with programmatic access (do not assign to any group)
   and download the credentials file (with AK/SK);
3. In both PFS, [create a policy that allows bucket read/write][pfs-policy]
   only for the IAM User created previously;
4. Apply base64 encoding to AK and to SK;
5. Update `obs-airflow.yml` with AK and SK encoded, and also with PFS names and
   Enterprise Project ID;
6. Create the namespace and the PV/PVCs for PFS:

```sh
kubectl apply -f airflow-namespace.yml
kubectl apply -f obs-airflow.yml
```

7. This repository uses a separate container for PostgreSQL. Generate a random
   password, update the `POSTGRES_PASSWORD` value in `postgresql-workload.yml`
   and `data.metadataConnection.pass` in `values.yml`;
8. Create the PostgreSQL Workload:

```sh
kubectl apply -f postgresql-workload.yml
```

9. Generate a new password for the Airflow web interface, and a random value
   for `webserverSecretKey` and update `values.yml`;
10. Install the Helm chart in the CCE Cluster:

```sh
helm repo add apache-airflow https://airflow.apache.org
helm upgrade --install airflow apache-airflow/airflow --namespace airflow -f values.yml
```

11. Create an ingress to access the AirFlow web interface, or update the
    `airflow-webserver` service to be NodePort or DNAT instead. If you choose
    to create an ingress, you can [use the CCE console to create an ELB Ingress][cce-ingress].
    If you choose NodePort, make sure the CCE Node has an [EIP bound][ecs-eip].
    If you choose DNAT, create a public NAT Gateway first and then use the
    [CCE console to create a DNAT rule][cce-dnat].

## Uninstall

```sh
helm delete airflow --namespace airflow
kubectl delete pvc -n airflow redis-db-airflow-redis-0
kubectl delete pvc -n airflow data-airflow-postgresql-0
kubectl delete -f postgresql-workload.yml
kubectl delete -f obs-airflow.yml
kubectl delete -f airflow-namespace.yml
```

[cce]: <https://support.huaweicloud.com/intl/en-us/cce/index.html>
[airflow-helm]: <https://airflow.apache.org/docs/helm-chart/stable/index.html>
[kubectl]: <https://kubernetes.io/docs/tasks/tools/>
[kubectl-cce]: <https://support.huaweicloud.com/intl/en-us/usermanual-cce/cce_10_0107.html>
[helm]: <https://helm.sh/docs/intro/install/>
[pfs-create]: <https://support.huaweicloud.com/intl/en-us/pfsfg-obs/obs_13_0002.html>
[pfs-policy]: <https://support.huaweicloud.com/intl/en-us/usermanual-obs/obs_03_0142.html>
[cce-ingress]: <https://support.huaweicloud.com/intl/en-us/usermanual-cce/cce_10_0251.html>
[ecs-eip]: <https://support.huaweicloud.com/intl/en-us/usermanual-ecs/en-us_topic_0174917535.html>
[cce-dnat]: <https://support.huaweicloud.com/intl/en-us/usermanual-cce/cce_10_0058.html>
