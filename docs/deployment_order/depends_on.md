---
title: Resource Deployment Order
description: Describe how Sveltos can be instructed to follow an order when deploying resources
tags:
    - Kubernetes
    - Sveltos
    - add-ons
    - order
authors:
    - Gianluca Mardente
---

ClusterProfile instances can leverage other ClusterProfiles to establish a deployment order for add-ons and applications. The *dependsOn* fields enables the definition of prerequisite ClusterProfiles. Within any managed cluster that matches the current ClusterProfile, the deployment of add-ons and applications will only start once all add-ons and applications in the specified dependency ClusterProfiles have been successfully deployed.

 For instance, a ClusterProfile encapsulates all Kyverno policies for a cluster and declares its dependency on the ClusterProfile named kyverno, which is responsible for installing the Kyverno Helm chart.

```yaml
  apiVersion: config.projectsveltos.io/v1alpha1
  kind: ClusterProfile
  metadata:
    name: kyverno-policies
  spec:
    clusterSelector: env=fv
    dependsOn:
    - kyverno
    policyRefs:
    - deploymentType: Remote
      kind: ConfigMap
      name: disallow-latest-tag
      namespace: default
      kind: ConfigMap
      name: restrict-wildcard-verbs
      namespace: default
```
[^1]

```yaml
  apiVersion: config.projectsveltos.io/v1alpha1
  kind: ClusterProfile
  metadata:
    name: kyverno
  spec:
    clusterSelector: env=fv
    helmCharts:
    - chartName: kyverno/kyverno
      chartVersion: v3.0.1
      helmChartAction: Install
      releaseName: kyverno-latest
      releaseNamespace: kyverno
      repositoryName: kyverno
      repositoryURL: https://kyverno.github.io/kyverno/
```

## Another example

For example, if the ClusterProfile instance *cp-kubevela* relies on the ClusterProfile instance *cp-kyverno*, this can be configured by simply setting the dependsOn field in the *cp-kubevela* ClusterProfile.

```yaml
apiVersion: config.projectsveltos.io/v1alpha1
kind: ClusterProfile 
metadata: 
  name: cp-kubevela
spec:
  dependsOn:
  - cp-kyverno
  clusterSelector: env=production
  syncMode: Continuous
  helmCharts:
  - repositoryURL: https://kubevela.github.io/charts
    repositoryName: kubevela
    chartName: kubevela/vela-core
    chartVersion: 1.9.6
    releaseName: kubevela-core-latest
    releaseNamespace: vela-system
    helmChartAction: Install
```

```yaml
apiVersion: config.projectsveltos.io/v1alpha1
kind: ClusterProfile
metadata:
  name: cp-kyverno
spec:
  clusterSelector: env=prod
  helmCharts:
  - repositoryURL:    https://kyverno.github.io/kyverno/
    repositoryName:   kyverno
    chartName:        kyverno/kyverno
    chartVersion:     v3.0.1
    releaseName:      kyverno-latest
    releaseNamespace: kyverno
    helmChartAction:  Install
```

This is equivalent of creating a single ClusterProfile. 

```yaml
apiVersion: config.projectsveltos.io/v1alpha1
kind: ClusterProfile
metadata:
  name: cp-kyverno
spec:
  clusterSelector: env=prod
  helmCharts:
  - repositoryURL:    https://kyverno.github.io/kyverno/
    repositoryName:   kyverno
    chartName:        kyverno/kyverno
    chartVersion:     v3.0.1
    releaseName:      kyverno-latest
    releaseNamespace: kyverno
    helmChartAction:  Install
  - repositoryURL: https://kubevela.github.io/charts
    repositoryName: kubevela
    chartName: kubevela/vela-core
    chartVersion: 1.9.6
    releaseName: kubevela-core-latest
    releaseNamespace: vela-system
    helmChartAction: Install
```

Separate ClusterProfiles promote better organization and maintainability, especially when different teams or individuals manage different ClusterProfiles.

[^1]: To create ConfigMaps with Kyverno policies used in this example
```
wget https://raw.githubusercontent.com/kyverno/policies/main/best-practices/disallow-latest-tag/disallow-latest-tag.yaml
kubectl create configmap disallow-latest-tag --from-file disallow-latest-tag.yaml
wget https://raw.githubusercontent.com/kyverno/policies/main/other/res/restrict-wildcard-verbs/restrict-wildcard-verbs.yaml
kubectl create configmap restrict-wildcard-verbs --from-file restrict-wildcard-verbs.yaml
```