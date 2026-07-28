# Network Policy (Incoming: Ingress, & Expernal: Egress)

[Official Documentation Page](https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/#limit-access-to-the-nginx-service)

NOTE: It SUPPORTs 1) kube-router 2) Clico 3) Romana

It DOES NOT support as of date July 2026 - Flannel

The network policy defines network traffic Ingress or Egress to each App tier that runs in the PODs.

Ingress - Incoming traffic
Egress - Outgoing traffic

A) Web App (running under a POD)  B) API App (running under other POD)  C) DB App (running under another POD).
To make us understand if you consider the traffic condition of "B", any traffic coming from A to B is Ingress in respective to B and outgoing traffic from
"B" to "C" is Egress respective of "B". Bydefault there is no restriction in the Kubernetes cluster. You have defined the network policy.

Let's say if you wanted to restrict the network traffic for DB (C), here is the definition as an example:

file - networkpolicy.yaml

    apiVersion: network.k8s.io/v1
    kind: NetworkPolicy
    metadata:
      name: db-policy 
    spec: 
        podSelector:
          matchLabels:
              role: db     # The rule is for DB Pod, meaning Ingress/Incoming network traffic for DB Pod. And its from API-Pod. Check the below "podSelector"
        policyType:        # note, unless you define policy type all traffic allowded with in the K8s cluster
          - Ingress
          # - Egress
        ingress:
          - from:
             - podSelector:
                 matchLabels:
                   name: api-pod  # This the label mentioned in API-POD also, meaning from "api-pod" traffic allowed for DB Pod
            ports:
             - protocol: TCP
               port: 3306


What if you have multiple ENV like prod & stage, and have api-pod matchLabels, then you can separate them from namespaces.
Check below -

file - networkpolicy.yaml

    apiVersion: network.k8s.io/v1
    kind: NetworkPolicy
    metadata:
      name: db-policy
      namespace: prod 
    spec: 
        podSelector:
          matchLabels:
              role: db 
        policyType:        <- note, unless you define policy type all traffic allowded with in the K8s cluster
          - Ingress
          - Egress
        ingress:
          - from:
             - podSelector:
                 matchLabels:
                   name: api-pod
                namespaceSelector:          # <- if you wanted to restric traffic from namespace level then commen out podSelector blocks
                    matchLabels:
                      kubernetes.io/metadata.name: prod
             - ipBlock:                   # if wanted allow ingress from outside network, example backup
               cidr: 192.168.6.10/32
            ports:
             - protocol: TCP
               port: 3306
    
        egress: 
          - to:
             - ipBlock:
                  cidr: 192.168.6.10/32
            ports:
              - protocol: TCP
                port: 80

NOTE: By-default all namespace PODs allow to access with same matchLabels. Example, if you have "test-API" lables in in Dev and Prod env then PODs 
can be accessible from each env even though they are in separate namespace. To restic that you need you specify the namespace name in the Network Policy.
All THESE RESTRICTION policy works WITHIN the cluster.

Lets say you have another DB backup server in another network Ex: 192.168.6.10/32 network. Then you have to specify - ipBlock: as Ingress to allow traffic
from Backup DB server and then also specify Egress - ipBlock: for outgoing traffic from DB server to Backup DB server.

Commands:

    >> kubectl get networkpolicies
    >> kubectl describe netpol payroll       # payrole is policy name

Note: As of now this "Network Policy" supports by 1) Kube-router 2) Calico 3) Romana. But "Flannel" does not support Network Policy.

Another Example:

Create a network policy to allow egress traffic from the Internal application only to the payroll-service and db-service.
Use the spec given below. You might want to enable ingress traffic to the pod to test your rules in the UI.

Also, ensure that you allow egress traffic to DNS ports TCP and UDP (port 53) to enable DNS resolution from the internal pod.

. Policy Name: internal-policy
. Policy Type: Egress
. Egress Allow: payroll
. Payroll Port: 8080
. Egress Allow: mysql
. MySQL Port: 3306

File: internal-policy.yaml (here only EGRESS policy configured as an example)

     apiVersion: networking.k8s.io/v1
     kind: NetworkPolicy
     metadata:
       name: internal-policy
       namespace: default
     spec:
       podSelector:
         matchLabels:
           name: internal
       policyTypes:
         - Egress
       egress:
         - to:
           - podSelector:
               matchLabels:
                 name: payroll
           ports:
             - protocol: TCP
               port: 8080
         - to:
           - podSelector:
               matchLabels:
                 name: mysql
           ports:
             - protocol: TCP
               port: 3306


    > kubectl get networkpolici    
        NAME              POD-SELECTOR    AGE
        internal-policy   name=internal   9m20s
        payroll-policy    name=payroll    34m

**Scenario-Based Question on Network Policy*

If I have 500 Pods running in the Prod cluster and I have 10 different DB and I wanted to allow only 200 Apps to allow the direct DB access. Then, will not my Network Policy will be very BIG?

ANS:
No, a well-designed NetworkPolicy does not become huge. The trick is to use labels strategically, not to list every Pod individually.

* 500 Pods
* 200 applications need database access

Is a BAD Design

In this case we have to use "Common Labels" 

Lebels should look like this way (In the DB YAML POD) -

        labels:
          access-db: "true"

As example for 2 application's labels looks like this when  a Deployment gets database access gets:

APP1:

      template:
        metadata:
          labels:
            app: payment
            access-db: "true"   # <- Make a note this label - This is common label in Pod and DB as well

App2:

      template:
        metadata:
          labels:
            app: order
            access-db: "true"   # <- Make a note this label - This is common label in Pod and DB as well


Database NetworkPolicy
Your database Pods might have label:

    labels:
      app: postgres

NetworkPolicy:

        apiVersion: networking.k8s.io/v1
        kind: NetworkPolicy

        spec:
          podSelector:
            matchLabels:
              app: postgres

          ingress:
          - from:
            - podSelector:
                matchLabels:
                  access-db: "true"       # <- Make a note this label - This is common label in Pod and DB as well

**How it Flows*

        Payment Pod
        access-db=true
                │
                │ Allowed
                ▼
             PostgreSQL
        
        Order Pod
        access-db=true
                │
                │ Allowed
                ▼
             PostgreSQL
        
        Inventory Pod
        access-db=true
                │
                │ Allowed
                ▼
             PostgreSQL
        
        Frontend Pod
        access-db=false
                │
                │ ❌ Denied
                ▼
             PostgreSQL


**This Is Why Labels Matter So Much*

When you first learn Kubernetes, labels seem like just metadata.
In production, they're much more than that. They drive:

* Deployments
* Services
* NetworkPolicies
* PodDisruptionBudgets
* ServiceMonitors (Prometheus)
* Admission policies
* GitOps automation

A well-designed labeling strategy is one of the foundations of a scalable Kubernetes platform.
