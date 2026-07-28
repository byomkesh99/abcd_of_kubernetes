# Helm

Helm is like a package manager that helps to install, upgrade, rollback, and uninstall K8s objects.
We do not need to micro-manage each K8s object for us. 

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
        $ helm repo update --help     #  etc 


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

- [Helm hub:] (https://hub.helm.sh/)
- [Helm charts GitHub Project:] (https://github.com/helm/charts)
- ]Installing Helm:] (https://helm.sh/docs/intro/install/)
- ]Helm v3 release notes:] (https://helm.sh/blog/helm-3-released/)

        $ helm create webappl-1  <-|  # this will create the folllowing files

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

Helm Chart :

Along with all yaml file like, deployment.yaml, service.yaml you can use file like values.yaml for 
templating the other yaml codes plus "Chart.yaml" would be for Helm chart to run .

Which actually to run the Application via Helm chart 

Example Code (Chart.yaml):

        apiVersion: v2                    # This comes in Helm-3 to differentiate between Helm 2/3
        appVersion: 5.8.1                 # Application version i.e. wordpress
        version: 12.1.27                  # Chart version
        name: wordpress                   # Name of the chart (you name it according to App)
        description:
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

[home:] (https://github.com/bitnami/charts/tree/master/bitnami/wordpress)
[icom:] (https://bitnami.com/assets/stacks/wordpress/img/wordpress-stack-220x234.png)


## Helm Chart Structure

        > (Folder) Hello-World-chart 
          > templates       # Template directory
          > values.yaml     # Configurable values (file)
          > Chart.yaml      # Chart information (file)
          > LICENSE         # Chart License (file)
          > README.md       # Readme file
          > chart           # Dependency Chart directory


        $ helm --help 

Common Actions for Helm:

          - helm search
          - helm pull 
          - helm install 
          - helm list 

Usage : 
  helm [command]

Available Commands:
  completion
  create
  Dependency
  env
  get
  help
  history

[artifacthub](https://artifacthub.io/) - for most popular website for helm chart application package 

        $ helm search wordpress ... ... ...  # it requires to specify where to search hub or repo 
                                             # hub means artifacthub and repo mean in a specific repo 
        $ helm search hub wordpress 

## Deploying Wordpress

        $ helm repo add bitnami https://charts.bitnami.com/bitnami   # "bitnami" has been added to your repositories

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


Check Google for quick help 

### Example

File: values.yaml

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
        $ helm install --set wordpressBlogName="Helm Tutorials" my-release bitnami/wordpress --set wordpressEmail="antoine@example.com" 

OR You can create your own custom values file 

File: custom-values.yaml

        wordpressBlogName: Helm Tutorials 
        wordpressEmail: antoine@example.com 

        COMMANDS:

        $ helm pull bitnami/wordpress  # This will download the archived file, you need to un-tar it after 
        $ helm pull --untar bitname/wordpress  # This will pull the file and untar it

        $ helm install my-release ./wordpress 

## Lifecycle Management with Helm

        $ helm install nginx-release bitnami/nginx --version 7.1.0
        $ helm upgrade nginx-release bitnami/nginx 
        $ helm list   (list of release) 
        $ helm history nginx-release   ## (lists of releases and revision)

Note: each revision number is notning but to display as each rollback 

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
