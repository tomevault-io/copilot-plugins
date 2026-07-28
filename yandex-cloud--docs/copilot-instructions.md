## docs

> `k8s.tunnelClusters.agent` is a special role for creating [{{ k8s }} clusters](../../../managed-kubernetes/concepts/index.md#kubernetes-cluster) with tunnel mode. It enables you to create [node groups](../../../managed-kubernetes/concepts/index.md#node-group), disks, and internal load balancers. You can use previously created {{ kms-full-name }} [keys](../../../kms/concepts/key.md) to encrypt and decrypt secrets.

`k8s.tunnelClusters.agent` is a special role for creating [{{ k8s }} clusters](../../../managed-kubernetes/concepts/index.md#kubernetes-cluster) with tunnel mode. It enables you to create [node groups](../../../managed-kubernetes/concepts/index.md#node-group), disks, and internal load balancers. You can use previously created {{ kms-full-name }} [keys](../../../kms/concepts/key.md) to encrypt and decrypt secrets.

This role includes the `compute.admin`, `iam.serviceAccounts.user`, `k8s.viewer`, `kms.keys.encrypterDecrypter`, and `load-balancer.privateAdmin` permissions.

---
> Source: [yandex-cloud/docs](https://github.com/yandex-cloud/docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
