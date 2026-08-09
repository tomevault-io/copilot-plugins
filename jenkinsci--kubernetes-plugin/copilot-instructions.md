## kubernetes-plugin

> All tests in this plugin which touch an actual Kubernetes cluster (mainly those extending @src/test/java/org/csanchez/jenkins/plugins/kubernetes/pipeline/AbstractKubernetesPipelineTest.java) require you to pass `-Pktunnel` to Maven.

All tests in this plugin which touch an actual Kubernetes cluster (mainly those extending @src/test/java/org/csanchez/jenkins/plugins/kubernetes/pipeline/AbstractKubernetesPipelineTest.java) require you to pass `-Pktunnel` to Maven.
Also use `-DforkCount=1` when running multiple cluster-based test classes at once to disable concurrency, since the tests use fixed port numbers.

---
> Source: [jenkinsci/kubernetes-plugin](https://github.com/jenkinsci/kubernetes-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
