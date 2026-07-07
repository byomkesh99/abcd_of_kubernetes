# Kustomize Commands

## All Credit Goes to Mumshad Mannambeth (KodeKloud) & and his team - This is my personal note from his teachings

Kustomize, a Kubernetes-native tool, takes a declarative approach by using patches and overlays to modify base configurations that require the use of templating languages.

With Kustomization you are managing your Kubernetes config files in a better way for Dev, PrePROD and PROD environment without repeating dublicate code.

Example Structure:

>>Base + Overlay (Most Common)<<

          k8s/                        # Parent Folder
          │
          ├── base/                   # Base folder
          │     ├── deployment.yaml
          │     ├── service.yaml
          │     └── kustomization.yaml
          │
          └── overlays/               # Different ENV folder
                ├── dev/
                ├── stage/
                └── prod/

>>Which Is More Common?<<

| Feature                            | Overlay | Component |
| ---------------------------------- | ------- | --------- |
| Environment-specific configuration | ✅ Yes   | ❌ No      |
| Reusable feature sets              | ❌ No    | ✅ Yes     |
| Used by most companies             | *****    |  **        |

## How you are going to manage your KUSTOMIZE Codes

Imagine a company with 200 microservices. Then how you are going manage all the codes for a team work colaborately.

**A Typycal Workflow*

        Developer
             │
             ▼
        Git Repository (Source of Truth)
             │
             ▼
        Pull Request
             │
             ▼
        Code Review
             │
             ▼
        Merge
             │
             ▼
        ArgoCD / Flux
             │
             ▼
        Kubernetes Cluster

>>A Common Repository Structure<<

        git-repo/
        │
        ├── apps/
        │   ├── payment-service/
        │   │      ├── base/
        │   │      └── overlays/
        │   │           ├── dev/
        │   │           ├── stage/
        │   │           └── prod/
        │   │
        │   ├── order-service/
        │   ├── customer-service/
        │   ├── inventory-service/
        │   └── ...
        │
        └── platform/
            ├── ingress/
            ├── monitoring/
            ├── logging/
            └── cert-manager/

## Industry Best Practices

If I were designing a platform for 200+ applications, then:

* Git as the single source of truth for all Kubernetes manifests.
* Kustomize or Helm to manage environment-specific configuration.
* ArgoCD or Flux to synchronize Git with the cluster.
* Branch protection to prevent direct changes to production configuration.
* Pull request reviews for every configuration change.
* Repository backups and regular disaster recovery testing.
* Infrastructure as Code (for example, Terraform) to provision the Kubernetes cluster and supporting cloud resources.

## Tool Installation

      $ brew install kustomize   # This is MACOS
      OR Below Command
      $ curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash

By-default it comes with "kubectl" but its better to update with recent stable version.


$ kustomize version --short   # Verify your customized installed version
$
$ kustomize build k8s    # k8s is a folder where kubernetes deployment and service yamls are there. And make sure your K8s cluster is running

          `
          apiVersion: kustomize.config.k8s.io/v1beta1
          kind: Kustomization
          #kubernetes resources to be managed by kustomize 
          resources:
              - nginx-deployment.yaml
              - nginx-service.yaml
          #Customizations that need to be made
          commonLabels:
            company: KhelaGhar
          `

To apply the kustomize menifest here is the commands
        $ kustomize build k8s/ | kubectl apply -f -    # its simply Linux bash command

## Managing Directories:

As example, you have subdirectoried inside k8ss folder call dev api and db. To apply kustom for all the files inside sub-directory
you need add the following lines in the kustomization.yaml file.

apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
    - api/api-depl.yaml
    - api/api-service.yaml
    - db/db-depl.yaml
    - db/db-service.yaml

Then run the command

        $ kustomize build k8ss/ | kubectl apply -f -        # k8ss = just folder where all the yaml configs are there
        OR
        $ kubectl apply -k k8ss/

-->> To apply the changes for this folder here is the command

    $ kubectl apply -k /root/code/k8s/overlays/QA                 # Apply to perticular folder
    $ kubectl apply -k /root/code/k8s/overlays/staging
    $ kubectl apply -k k8s-components-2/overlays/enterprise

### Common Transformer

This is basically to add common lebels. namespace, annotations, prefix/suffix across particular application.
Example, we wanted to add namespace call 'custom-ingress' in all my paricular sets of yaml file. I can add is using Kustomization.yaml file.

Example: Check the File -  k8ssss/kustomization.yaml

commonLabels - adds a label to all Kubernetes resources
naePrefix/Suffix - adds a common prefix-suffix too all resource names
Namespace - add a common name space to all resources
commonAnnotations - adds an annotations to all resources

### Play with Patches

* Kustomize patches provide another method to modifying Kubernetes configs 
* Unlike common transformers, patches provide a more "surgical" approach to targeting one or more  sections in a Kubernetes resource.

* To create a patch 3 parameters must be provided:
      * Operation Type: add/remove/replace 

      * Target: What resource should this patch be applied on
          * Kind
          * Version/Group
          * Name
          * Namespace
          * labelSelector
          * AnnotationSelector

Value: What is the value that will either be replaced or added with (only needed for add/replce operations) 

JSON 6902 vs Strategic Merge Patch - You can plan any one of them which suits particularly at the time of your Design

### Json 6902 Patch

        `
        patches:              # Here, with the "-target" block, we match the K8s object and then apply the patch i.e. new replicas in this case
          - target:
              kind: Deployment
              name: api-deployment
            patch: |-
              - op: replace
                path: /spec/replicas    # just a map to come down to replcas labels
                value: 5
        `
[Ref_Link](https://datatracker.ietf.org/doc/html/rfc6902)   - See the JSON syntax and details

### Strategic Merge Patch

        patches:                # Its easy to understand. We just copy all the "Deployment" yaml codes and then keep till replicas and remove rest.
          - patch: |-
              apiVersion: app/v1
              kind: Deployment
              metadata:
                name: api-deployment
              spec:
                replicas: 5 

* Two different types of Patches to Apply ( Decide the best you like to Apply i.e. Inline OR Separate File)

>> Inline << - Meaning all patch code in one kustomization.yaml file (Play_With_Patches)
File: Kustomization.yaml

        patches: 
          - target:
              kind: Deployment
              name: api-deployment
            patch: |-
              - op: replace
                path: /spec/replicas
                value: 5 

>> Separate File <<  - Meaning separating your "-op" i.e. operation line of code in another file and link it in kustomization.yaml file

File: Kustomization.yaml 

        patches:
          - path: replica-patch.yaml
            target:
              kind: Deployment
              name: nginx-deployment

File: replica-patch.yaml

        - op: replace
          path: /spec/replicas
          value: 5 

>> Replace Dictionary [Json6902] << (Play_With_Patches)

Before Apply Kustomization:

File: api-depl.yaml
        `
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: api-deployment
        spec:
          replicas: 1
          selector:
            matchLabels:
              component: api
          template:
            metadata:
              labels:
                component: api
            spec:
              containers:
                - name: nginx
                  image: nginx
        `

File: kustomization.yaml   ## here we are going to replace the "component value"

        patches:
          - target:
               kind: Deployment
               name: api-deployment
            patch: |-
              - op: replace
                path: /spec/template/metadata/lebels/component
                value: web

After APPLY the patch 

File: api-depl.yaml
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: api-deployment
        spec:
          replicas: 1
          selector:
            matchLabels:
              component: api
          template:
            metadata:
              labels:
                component: web   ### Here is the replace value
            spec:
              containers:
                - name: nginx
                  image: nginx

>> Same thing to achieve via Strategic Merge Patch

File: Kustomization.yaml
        patches:
          - label-patch.yaml
        File: label-patch.yaml
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: api-deployment
        spec:
          template:
            metadata:
              labels:
                component: web

>> How to Add new key in deployment file <<  (Play_With_Patches)

File: kustomization.yaml
        patches:
          - target:
              kind: Deployment
              name: api-deployment
            patch: |-
              - op: add
                path: /spec/template/metadata/labels/org
                value: Khelaghar

Output:
File: api-depl.yaml
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: api-deployment
        spec:
          replicas: 1
          selector:
            matchLabels:
              component: api
          template:
            metadata:
              labels:
                component: web
                org: Khelaghar      # here the 'ORG' gets added
            spec:
              containers:
                - name: nginx
                  image: nginx

>> Same thing to achieve via Strategic Merge Patch

File: kustomization.yaml
        patch:
          - label-patch.yaml

File: label-patch.yaml
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: api-deployment
        spec:
          template:
            metadata:
              labels:
                org: Khelaghar

>> How to Add new key in deployment file <<  (Play_With_Patches)

File: kustomization.yaml
        patches:
          - target:
              kind: Deployment
              name: api-deployment
            patch: |-
              - op: remove
                path: /spec/template/metadata/labels/org

>>  With Strategic Merge  <<  (Play_With_Patches)

File: kustomization.yaml
        patch:
          - label-patch.yaml
        File: label-patch.yaml
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: api-deployment
        spec:
          template:
            metadata:
              labels:
                org: null

>> PATCH List <<   (Play_With_Patches)

>> Replace "NGINX" image <<

Replace List according to Json6902 . Here we are going to replace "NGINX" image

Before Apply Kustomization:

File: api-depl.yaml
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: api-deployment
        spec:
          replicas: 1
          selector:
            matchLabels:
              component: api
          template:
            metadata:
              labels:
                component: api
            spec:
              containers:
                - name: nginx
                  image: nginx

File: kustomization.yaml
        patches:
          - target:
              kind: Deployment
              name: api-deployment
            patch: |-
              - op: replace
                path: /spec/template/spec/containers/0    ## please here it follows as index of dictionary
                value:
                  - name: haproxy
                    image: haproxy

>> Another way of Image replace <<  (Play_With_Patches)

        patches:
          - target:
              kind: Deployment
              name: api-deployment
            patch: |-
              - op: replace
                path: /spec/template/spec/containers/0/image
                value: haproxy

### After apply the kustomization

File: api-depl.yaml

        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: api-deployment
        spec:
          replicas: 1
          selector:
            matchLabels:
              component: api
          template:
            metadata:
              labels:
                component: api
            spec:
              containers:
                - name: haproxy
                  image: haproxy

>> Now Replace List according to Strategic Merge Patch  <<  (Play_With_Patches)

File: kustomization.yaml
        patch:
          - label-patch.yaml

File: label-patch.yaml

        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: api-deployment
        spec:
          template:
            spec:
              containers:
                - name: nginx
                  image: haproxy

>> Add List Json6902 << (Play_With_Patches)

File: kustomization.yaml

        patches:
          - target:
              kind: Deployment
              name: api-deployment
            patch: |-
              - op: replace
                path: /spec/template/spec/containers/-     # This '-' is compulsory
                value:
                  name: haproxy
                  image: haproxy

>> Add List with Strategic Merge Patch <<  (Play_With_Patches)

File: kustomization.yaml
        patch:
          - label-patch.yaml

File: label-patch.yaml

        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: api-deployment
        spec:
          template:
            spec:
              containers:
                - name: haproxy
                  image: haproxy


>> Delete List Json 6902 <<  (Play_With_Patches)

In this example we are taking 2 container in the list and then going to Delete one

File: api-depl.yaml

        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: api-deployment
        spec:
          replicas: 1
          selector:
            matchLabels:
              component: api
          template:
            metadata:
              labels:
                component: api
            spec:
              containers:
                - name: web     # Index 0
                  image: nginx
                - name: database  # Index 1
                  image: mongo


File: kustomization.yaml  [according to Json6902]

        patches:
          - target:
              kind: Deployment
              name: api-deployment
            patch: |-
              - op: replace
                path: /spec/template/spec/containers/1  # Note here we put Index 1 because we are removing 2nd container

>> Delete List Strategic Merge Patch << (Play_With_Patches)

File: kustomization.yaml
        patch:
          - label-patch.yaml

File: label-patch.yaml

        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: api-deployment
        spec:
          template:
            spec:
              containers:
                - $patch: delete
                  name: database

### Overlays

#### BASE STRUCTURES

        k8s/
        |___base/
        |   |__kustomization.yaml
        |   |__nginx-depl.yaml
        |   |__service.yaml
        |   |__ redis-depl.yaml
        |
        |___overlays/
            |___dev/
            |   |___kustomization.yaml
            |   |___config-map.yaml
            |
            |___stg/
            |   |___kustomization.yaml
            |   |___config-map.yaml
            |
            |___prod/
                |___kustomization.yaml
                |___config-map.yaml

NOTE: Unless you understand all the concept from all steps mentioned above of this document, the below short form of code will not be meaningfull

File: Base/kustomization.yaml
resources:
    - nginx-depl.yaml
    - service.yaml
    - redis-depl.yaml

File: Base/nginx-depl.yaml

        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: nginx-deployment
        spec:
          replicas: 1 

Now as per your needs in the env of DEV and PROD. EX: we are using 2 replica in Dev and 5 in PROD.

File: dev/kustomization.yaml

        bases:
          - ../../base
        patch: |-
          - op: replace
            path: /spec/replicas
            value" 2

File: prod/kustomization.yaml

      bases:
        - ../../base
      patch: |-
        - op: replace
          path: /spec/replicas
          value" 5

Note: You can add more resource which may not managed from Base folder

Example:

        |___overlays/
            |___dev/
            |   |___kustomization.yaml
            |   |___config-map.yaml
            |
            |___stg/
            |   |___kustomization.yaml
            |   |___config-map.yaml
            |
            |___prod/
                |___kustomization.yaml
                |___config-map.yaml
                |___grafana-depl.yaml
                |___prometheus-depl.yaml

File: prod/kustomization

        bases:
          - ../../base

        resources:
          - grafana-depl.yaml
          - prometheus-depl.yaml

        patch: |-
          - op: replace
            path: /spec/replicas
            value" 5

### Components

* Components provide the ability to define resuable pieces of configuration logic(resources + patches)
  that can be included in multiple overlays
* Components are useful in situations where applications support multiple optional features
  that need to be enabled only in the subset of overlays.

        |----------------------------------------------\-----------------------------------------------|
        |                                              |                                               |
        base                                           |                                          Components
                              |------------------------|--------------------------|                    |
                              |                        |                          |                    |
                             dev                    Premium                   Self_hosted              |
                                                                                                       |
                                                                                          |---------------------------|             
                                                                                          |                           |
                                                                                       Caching                        DB

### Folder Structure

        k8s/
        |___base/
        |   |__kustomization.yaml
        |   |__api-depl.yaml
        |   
        |___Components/
        |    |
        |    |___caching/
        |    |    |___kustomization.yaml
        |    |    |___deployment-patch.yaml 
        |    |    |___redis-depl.yaml
        |    |
        |    |___DB/
        |        |___kustomization.yaml
        |        |___deployment-patch.yaml
        |        |___postgres-depl.yaml
        |
        |___overlays/
            |___dev/
            |   |___kustomization.yaml
            |   
            |___premium/
            |   |___kustomization.yaml
            |
            |___standalone/
                |___kustomization.yaml

