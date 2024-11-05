- [TAM your tech hour - Next-level Red Hat OpenShift Virtualization automation and configuration](#tam-your-tech-hour---next-level-red-hat-openshift-virtualization-automation-and-configuration)
  - [Webinar facts](#webinar-facts)
- [Webinar content](#webinar-content)
  - [Setup](#setup)
    - [OpenShit Virtualization](#openshit-virtualization)
    - [OpenShift GitOps](#openshift-gitops)
    - [OpenShift Pipelines](#openshift-pipelines)
  - [Using OpenShift GitOps with VMs](#using-openshift-gitops-with-vms)
    - [Provision all VMs](#provision-all-vms)
    - [Start VM via Git](#start-vm-via-git)
    - [Stop VM via Git](#stop-vm-via-git)
  - [Using OpenShift Pipelines with VMs](#using-openshift-pipelines-with-vms)
    - [Provision the setup](#provision-the-setup)
    - [Create database VM via OpenShift Pipelines](#create-database-vm-via-openshift-pipelines)
    - [Create a mysqldump via OpenShift Pipelines](#create-a-mysqldump-via-openshift-pipelines)

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

## Using OpenShift GitOps with VMs

This part of the demo will create four `namespaces`
- `dev-demo-db`  -> Database for development
- `prod-demo-db` -> Database for production
- `dev-demo-vm`  -> VMs for development
- `prod-demo-vm` -> VMs for production

These `namespaces` (and other resources) are deployed via [OpenShift GitOps](https://docs.redhat.com/en/documentation/red_hat_openshift_gitops/1.14/html/understanding_openshift_gitops/index), which is based on the open source project [Argo CD](https://argoproj.github.io/cd/). The `namespaces`, i.e. multi-stage environments, are managed via [`kustomize`](https://kustomize.io/).
The overlay structure is as follows:
```sh
tree applicationsets 
.
├── demo-db
│   ├── applicationset-demo-db.yaml
│   └── kustomize
│       ├── base
│       │   ├── deployment.yaml
│       │   ├── kustomization.yaml
│       │   └── service.yaml
│       └── overlays
│           ├── dev
│           │   └── kustomization.yaml
│           └── prod
│               └── kustomization.yaml
└── demo-vm
    ├── applicationset-demo-vm.yaml
    └── kustomize
        ├── base
        │   ├── kustomization.yaml
        │   ├── route.yaml
        │   ├── service.yaml
        │   ├── virtualmachine-1.yaml
        │   └── virtualmachine-2.yaml
        └── overlays
            ├── dev
            │   └── kustomization.yaml
            └── prod
                └── kustomization.yaml
```

The database `namespaces` are pretty similar, as they just have a `Deployment` and `Service`, each.

The VM `namespaces` hold the following base configuration:
- Virtual machine 1 -> First web server VM
- Virtual machine 2 -> Second web server VM
- `service`         -> Enables the connection to all VMs 
- `route`           -> Enables HTTP(S) traffic to the VMs via the `service`
The `namespaces` differ in the overlays, because the development environment `dev-demo-vm` has the second VM stopped, as there is no need for it.

### Provision all VMs

```sh
# Obtain GitOps password
oc get secret/openshift-gitops-cluster -n openshift-gitops -o jsonpath='{.data.admin\.password}' | base64 -d

# Create ApplicationSets
oc apply -f applicationsets/demo-vm/applicationset-demo-vm.yaml
oc apply -f applicationsets/demo-db/applicationset-demo-db.yaml

# Apply permissions for namespaces
oc adm policy add-role-to-user admin system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller -n dev-demo-vm
oc adm policy add-role-to-user admin system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller -n prod-demo-vm
oc adm policy add-role-to-user admin system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller -n dev-demo-db
oc adm policy add-role-to-user admin system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller -n prod-demo-db
```

[Link to demonstration as GIF](./src/video/deploy_vms_via_git.gif)

### Start VM via Git

[Link to demonstration as GIF](./src/video/start_vm_via_git.gif)

### Stop VM via Git

[Link to demonstration as GIF](./src/video/stop_vm_via_git.gif)

## Using OpenShift Pipelines with VMs


### Provision the setup

```sh
oc new-project pipelines-demo

# Import Tekton tasks for KubeVirt
kubectl apply -f https://github.com/kubevirt/kubevirt-tekton-tasks/raw/v0.22.0/release/tasks/create-vm-from-manifest/create-vm-from-manifest.yaml
kubectl apply -f https://github.com/kubevirt/kubevirt-tekton-tasks/raw/v0.22.0/release/tasks/cleanup-vm/cleanup-vm.yaml
kubectl apply -f https://github.com/kubevirt/kubevirt-tekton-tasks/raw/v0.22.0/release/tasks/wait-for-vmi-status/wait-for-vmi-status.yaml
kubectl apply -f https://github.com/kubevirt/kubevirt-tekton-tasks/raw/v0.22.0/release/tasks/execute-in-vm/execute-in-vm.yaml
kubectl apply -f https://github.com/kubevirt/kubevirt-tekton-tasks/raw/v0.22.0/release/tasks/generate-ssh-keys/generate-ssh-keys.yaml
kubectl apply -f https://github.com/kubevirt/kubevirt-tekton-tasks/raw/v0.22.0/release/tasks/modify-data-object/modify-data-object.yaml
kubectl apply -f https://github.com/kubevirt/kubevirt-tekton-tasks/raw/v0.22.0/release/tasks/disk-virt-customize/disk-virt-customize.yaml
```

### Create database VM via OpenShift Pipelines
```sh
# Create Pipeline
oc apply -f pipelines/create-database-vm-pipeline.yaml 
pipeline.tekton.dev/create-database-vm created

# Trigger a pipeline run
oc create -f pipelines/create-database-vm-pipelinerun.yaml 
pipelinerun.tekton.dev/create-database-vm-5srgh created

# Verify database is filled with data
virtctl -n pipelines-demo console database-vm
mysql -u sample_user -ppassword123 -D sample_db -e "SELECT * from employees;"
+----+-------+-----------+-------------+
| id | name  | position  | employee_id |
+----+-------+-----------+-------------+
|  1 | Alice | Manager   | M010        |
|  2 | Bob   | Developer | D001        |
|  3 | Carol | Developer | D002        |
|  4 | Dave  | Sales     | S001        |
+----+-------+-----------+-------------+
```
### Create a mysqldump via OpenShift Pipelines

```sh
# Create Pipeline
oc apply -f pipelines/create-mysqldump-from-database-vm-pipeline.yaml 
pipeline.tekton.dev/create-mysqldump-from-database-vm created

# Trigger a pipeline run
oc create -f pipelines/create-mysqldump-from-database-vm-pipelinerun.yaml
pipelinerun.tekton.dev/create-mysqldump-from-database-vm-b5m9p created
```