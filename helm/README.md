# Helm

[![KodeKloud Ref Video](http://img.youtube.com/vi/kJscDZfHXrQ/0.jpg)](http://www.youtube.com/watch?v=zJ6WbK9zFpI)

Helm is like a package manager that helps to install, upgrade, rollback, and uninstall K8s objects.
We do not need to micro-manage each K8s object for us.

## Helm Architecture

                Helm CLI
                    |
                    |
          -------------------
          |                 |
     Kubernetes API Server
                    |
             Kubernetes Cluster

Currently Helm 3 is running most of the K8s platform. Earlier it was Helm 2
Differences between them - Helm 3 differs from Helm 2 primarily through the removal of Tiller, namespace-scoped releases, and secrets-based storage. Tiller removed for security reasons because with Tiller user will get more powerfull resource access to the Cluster.

Commands:
        $ helm install <workpress>
        $ helm upgrade <wordpress>
        $ helm rollback <wordpress> 
        $ helm uninstall <wordpress> ## It should be your release name 
        $ helm --help 
        $ helm pull 
        $ helm list 

Sub commands & sub-sub commands 

        $ helm repo --help 

          Available Commands:
             add         add a chart repository
             index       generate an index file given a directory containing packaged charts
             list        list chart repositories
             remove      remove one or more chart repositories
             update      update information of available charts locally from chart repositories

        $ helm repo update --help

[Artifact-HUB](https://artifacthub.io/) -    # Check & Know this site for helm chart 

In the above site you will find all packages. Its like docker Hub.
Also can be search from command line. Example:

        $ helm search wordpress  OR 
        $ helm search hub wordpress 

To search in specific repository you need to mention the repo name with repo urls .
Example :

        $ helm repo add bitnami https://charts.bitnami.com/bitnami 
        $ helm install my-rease bitnami/wordpress 

The chart deploy it as a release, to check the release run the following command

        $ helm list 
        $ helm uninstall my-release 

- [Helm hub:](https://hub.helm.sh/)
- [Helm charts GitHub Project:](https://github.com/helm/charts)
- [Installing Helm:](https://helm.sh/docs/intro/install/)
- [Helm v3 release notes:](https://helm.sh/blog/helm-3-released/)

Example:

        $ helm create webappl-1    # <- this will create the folllowing files

folder structure =
-> webapp-1 
      -> templates
           - configmap.yaml 
           - deployment.yaml 
           - service.yaml 
      -> chart.yaml
      -> and values.yaml   # this file is very useful for us and templates.

After values.yml gets updated you can run the folllowing commands

        $ helm upgrade mywebapp mywebapp-release webapp-1/ --values webapp-1/values.yaml 

## From - KodeKloud ---

### Helm Chart

A Basic Example of using Helm: Check the templating part

File: deployment.yaml

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: Hello-India-deployment
      labels:
        app: Hello-India
    spec:
      replicas: {{ .Values.replicaCount }}    # Mark this Point
      selector:
        matchLabels:
          app: Hello-India
      template:
        metadata:
          labels:
            app: Hello-India
        spec:
          containers:
          - name: Hello-India
            image: {{ .Values.image.repository }}   # Mark this Point
            ports:
            - containerPort: 80

File: service.yaml

     apiVersion: v1
     kind: Service
     metadata:
       name: Hello-India-Service
     spec:
       type: NodePort
       ports:
         - port: 80
           targetPort: 80
           protocol: TCP
           name: http
       selector:
         app: Hello-India

File: Values.yaml

     replicaCount: 2
     image:
       repository: helloIndia:1.1

File: Chart.yaml

     apiVersion: v2
     appVersion: "1.10.1"
     name: hello India
     description: a exmaple App
     
     type: application

Along with all yaml file like, deployment.yaml, service.yaml you can use file like values.yaml for 
templating the other yaml codes plus "Chart.yaml" would be for Helm chart to run .

Which actually to run the Application via Helm chart 

Example Code (Chart.yaml):

        apiVersion: v2                    # This comes in Helm-3 to differentiate between Helm 2 or 3
        appVersion: 5.8.1                 # Application version i.e. wordpress
        version: 12.1.27                  # Chart version
        name: wordpress                   # Name of the chart (you name it according to App)
        description: Web publishing platform
        type: application                 # There are 2 type - application and Library
        dependencies:
          -  condition: mariadb.enable
             name: mariadb
             repository: https://charts.bitnami.com/bitnami
             version: 9.x.x 
             <code hidden>
        keywords:
          - application 
          - blog
          - wordpress
        maintainers:
          - email: container@bitnami.com 
            name: Bitnami
       home: https://github.com/bitnami/charts/tree/master/bitnami/wordpress
       icon: https://bitnami.com/assets/stacks/wordpress/img/wordpress-stack-220x234.png


## Helm Chart Structure

       > (Folder) Hello-India-chart 
           > templates       # Template directory
           > values.yaml     # Configurable values (file)
           > Chart.yaml      # Chart information (file)
           > LICENSE         # Chart License (file)
           > README.md       # Readme file
           > chart           # Dependency Chart directory
 
Another Example:

      myapp-chart/
      
          Chart.yaml
          values.yaml
      
          templates/
      
              deployment.yaml
              service.yaml
              ingress.yaml
              configmap.yaml
              secret.yaml

## $ helm --help

Common Actions for Helm:

          - helm search       # search for charts
          - helm pull         # download a chart to your local directory to view
          - helm install      # upload the chart to your Kubernetes cluster
          - helm list 

(PLEASE CHECK ALL THE EXAMPLE COMMAND BELOW)

Usage : 
  helm [command]

Available Commands:
  completion        # generate autocompletion scripts for the specified shell
  create            # create a new chart with the given name
  dependency        # manage a chart's dependencies
  env               # helm client environment information
  get               # download extended information of a named release
  help              # Help about any command
  history           # fetch release history

[artifacthub](https://artifacthub.io/) - for most popular website for helm chart application package 

        $ helm search wordpress ... ... ...  # it requires to specify where to search hub or repo 
                                             # hub means artifacthub and repo mean in a specific repo 
        $ helm search hub wordpress 

## Deploying Wordpress

        $ helm repo add bitnami https://charts.bitnami.com/bitnami   # "bitnami" has been added to your repositories
        $ helm install [release-name] [chart-name]
        $ helm install my-release bitnami/wordpress 

## Helm Release 

(Once you deploy with helm, it deploy as release and  you will see it in the following list command)

        $ helm list
        $ helm uninstall my-release  (to remove the package)

## Helm Repo

        $ helm repo
        $ helm repo list
        $ helm repo update

## Customizing Chart Parameter

A File: values.yaml - just for an example to understand

        image:
          registry: docker.io 
          repository: bitnami/wordpress
          tag: 5.8.2-debien-10-r0 
        ## @param wordpressUsername WordPress wordpressUsername
        ##
        wordpressUsername: user
        #
        ## @param wordpressPassword WordPress wordpressPassword 
        # Defaults to a random 10-character alphanumeric string if not set 
        ##
        wordpressPassword: ""
        ## @param existingSecret
        ##
        existingSecret: "" 
        ## @param wordpressEmail wordpress User Email 
        wordpressEmail: user@example.com 
        #
        ## @param wordpressBlog 
        #
        wordpressBlogName: User's Blog! 

Here is the custom command to edit the exiting parameters 

        $ helm install --set wordpressBlogName="Helm Learning" my-release bitnami/wordpress     # wordpressBlogName is there in values.yaml
        $ helm install --set wordpressEmail="abc@wordpress.com" my-release bitnami/wordpress
        OR
        $ helm install --set wordpressBlogName="Helm Tutorials" my-release bitnami/wordpress --set wordpressEmail="antoine@example.com"
        $
        OR
        $ cat custom-values.yaml
        
        wordpressBlogName: Helm Learning
        wordpressEmail: abc@wordpress.com

        $ helm install --values custom-values.yaml my-release bitnami/wordpress
        OR
        $ helm pull bitnami/wordpress         # it will download the package in a archive format in your local machine
        $ helm pull --untar bitnami/wordpress # Now untar your bitnami package
        $ ls wordpress    # go to desire folder/file and then EDIT it and then install it
        $
        $ helm install my-release ./wordpress
      

## Lifecycle Management with Helm

        $ helm install nginx-release bitnami/nginx --version 7.1.0
        $ helm upgrade nginx-release bitnami/nginx 
        $ helm list   (list of release) 
        $ helm history nginx-release   ## (lists of releases and revision)

Note: each revision number is notning but to display as each rollback

      helm create myapp
              │
              ▼
      Edit values.yaml
              │
              ▼
      Edit templates/
              │
              ▼
      helm lint
              │
              ▼
      helm template
              │
              ▼
      helm install
              │
              ▼
      helm upgrade
              │
              ▼
      helm rollback
              │
              ▼
      helm uninstall

## Some IMPORTANT Commands to remember

        $ helm repo add bitnami https://charts.bitnami.com/bitnami
        $ helm install my-release oci://REGISTRY_NAME/REPOSITORY_NAME/wordpress    

        $ helm search hub wordpress  # All wordpress version in Artifact hub website 
        $ helm search repo wordpress  # to find word press App version 
        $ helm repo list   ## How many helm chart repositories are there in the controlplane node now?
        $ helm uninstall my-wordpress or apache or my-browse or my-release ## these are release name while you are going to uninstall
        $ helm repo remove hashicorp ## Remove particular repository 

We can run mutiple Application Instance with different name of release in the same system.

Example:

        $ helm install my-first-instance bitnami/wordpress --version 26.0.0
        $ helm install my-second-instance bitnami/wordpress --version 26.0.0 

## Helm vs Kustomize

      | Feature                | Helm                             | Kustomize                      |
      | ---------------------- | -------------------------------- | ------------------------------ |
      | Uses templates         | ✅ Yes                            | ❌ No                          |
      | Uses overlays          | Limited                          | ✅ Yes                          |
      | Package manager        | ✅ Yes                            | ❌ No                          |
      | Downloads applications | ✅ Yes                            | ❌ No                          |
      | Versioned releases     | ✅ Yes                            | ❌ No                          |
      | Rollback support       | ✅ Yes                            | ❌ No                          |
      | Built into `kubectl`   | ❌ No                             | ✅ Yes                         |
      | Best for               | Installing reusable applications | Customizing your own manifests |
