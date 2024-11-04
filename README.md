- [TAM your tech hour - Next-level Red Hat OpenShift Virtualization automation and configuration](#tam-your-tech-hour---next-level-red-hat-openshift-virtualization-automation-and-configuration)
  - [Webinar facts](#webinar-facts)
- [Webinar content](#webinar-content)
  - [Setup](#setup)
    - [OpenShit Virtualization](#openshit-virtualization)
    - [OpenShift GitOps](#openshift-gitops)
    - [OpenShift Pipelines](#openshift-pipelines)

**Disclaimer: All demonstrations as GIFs are linked via hyperlinks at the bottom of each (sub)section to reduce the loading time of the README.**

# TAM your tech hour - Next-level Red Hat OpenShift Virtualization automation and configuration

Content of the TAM your tech hour webinar about OpenShift Virtualization: `Next-level Red Hat OpenShift Virtualization automation and configuration`. 

## Webinar facts

- Previous webinar: [Github](https://github.com/Skalador/tam-your-tech-hour-openshift-virtualization)
- Link to sign up: TBD
- Link to recording: TBD
- Date: December 5, 2024, 10:00 a.m. CEST (UTC+ 2)
- Contributor: [Amr Elganzory](https://github.com/AmrGanz), [Kevin Niederwanger](https://github.com/Skalador)

# Webinar content

## Setup 

### OpenShit Virtualization

```sh
# OpenShift Virtualization
oc create -f operators/virtualization/operator-virtualization.yaml
# HyperConvergedController CR
oc apply -f operators/virtualization/hyperconverged.yaml
```

### OpenShift GitOps

```sh
# GitOps Operator
oc apply -f operators/gitops/operator-gitops.yaml

# Obtain GitOps password
oc get secret/openshift-gitops-cluster -n openshift-gitops -o jsonpath='{.data.admin\.password}' | base64 -d
```

### OpenShift Pipelines

```sh
# Pipelines Operator
oc apply -f operators/pipelines/operator-pipelines.yaml

# Enable console plugin
oc patch console.operator cluster --type='json' -p='[{"op": "add", "path": "/spec/plugins/-", "value": "pipelines-console-plugin"}]'
```
