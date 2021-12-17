

#8 Namespaces:prevents resource conflicts by logical grouping/organisation
original ns are default, kube-system, kube-public and kube-node-lease
to group by by resource-name(jenkins, sonarqube) 
to group by environments(common(ingress-controller, elk-stack), dev, stage)
to group by team (team-solaar, team-argo)
to group by deployment-type in prod (prod-blue, prod-green)
to group by limit(cpu/ram/storage) / access(create/update/delete) resources within a (secured&isolated)namespace
configmap in ns projA can consume database service in ns DB i.e db_url: mysql-svc.DB
global namespaced resources(kubectl api-resources --namespaced=false) volume, node
kubens allows you to switch ns easily (kubens jenkins)

#9 Service: provides stable ip address, loadbalancing, loose coupling within/outside cluster
service types: clusterip. nodeport, headless and loadbalancer service
clusterip service:
only accessible withing the cluster
each pod get assigned an ip address within the residing node ip address range
microservice pods or ingress pod handover requests to cluster ip service using svc ip address and svc port
svc uses selector to identity it's member pods by matching its selector's kv with the deployment kv attribute
svc use its targetport attribute to identify which container within the pod to target
svc endpoints object uses the member's ip address:port lists to track the pods to forward traffic to
the defined svc port is arbitrary but pod's port isn't it has to match the target pod's port
svc may have 2 ports one for communicating between 2 microservice apps the other between prometheus pod and the sidecar container ie the app logs exporter within the pod
when a svc has 2 or more ports it should use name attribute for identification purposes
headless service:
used when service want to communicate with a specific pod directly
used when pod wants to taalk directly with a specific pod
used in stateful apps like databases where replicas are not identical
in databases only master pod writes the database, the worker pod does continuos data reading/cloning/syncronization after each write
new or addition worker clone data from the existing worker node
client discover specific pod ip addresses via 
option1(it's slow and inefficient): api call to k8s apiserver
option2(prefer way): dns lookup for a service return it's ip address, but by setting clusterIp to none, it returns the member pod ip address(es) to the client instead
client(app pod) communication with databases usually require clusterip(for pod to talk to db pod) and headless(for comms from client to pod directly or for data syncronization between dbs) services for each usecase
nodeport(not very secure):
it's an extension of the clusterip service type
makes external traffic has access to static/fixed port of each worker node
the nodeport can be defined, if not defined it randomly generate a port number between a valid range of 30000 - 32767
it uses the worker node ip address
when we create a nodeport service it also create a clusterip addr for the serivce, the traffic will route viaa the ip and the defined port to the target pod
service spans across the number of the worker nodes, and forwards traffic to one of the pods
loadbalancer:
service becomes accessible through a cloud provider's url
it's an extension of the nodeport service type
when create loadbalancer service type, nodeport(accessible only by LB) and clusterip ports are automatically created for traffic routing

#10 Ingress: external requests -> ingress -> service -> pod
ingress servicename should be thesame as the service name, serviceport thesame as service port
ingress host should be a valid domain address, this host domain address should be mapped to an entrypoint(node's ip)
ingress controller creates pods that evaluates and process the ingress rules, maanages all redirections, it's the entrypoint to cluster
in cloud: external requests -> cloud loadbalancer -> ingress controller -> ingress -> service -> pod
on premise/bare metal: external requests -> proxy server -> ingress controller -> ingress -> service -> pod
ingress use-cases: 
multiple pathas for the same host i.e: (host: myapp.com, path: /analytics path: shopping)
multiple subdomains or domains i.e: (host: analytics.myapp.com host: shopping.myapp.com)
configuring tls certificate for https: add tls under spec, under tls specify the hosts and secretname(points to secret created to hold base64 encoded values of tls.crt and tls.key under data, type must be kubernetes.io/tls)
the above secret has to be in the same namespace as the ingress file

#11 Volumes: persist data in kubernetes using volumes
storage requirements:
storage that doesn't depend on the pod lifecycle
storage must be available on all nodes
storage needs to survive even if cluster crashes
3 volume components
persistent volume (pysical storage and pv are created by aadmin/deops guys)
a cluster resource with defined storage size in a yaml file
pv is a abstract/virtual component that take the actual storage size from physical storage like local disk, external storage(nfs server and aws ebs)
physical storages are seen as plugins to k8s cluster
pv consumes physical storages (local or remote physical storages)
the pv yaml file mostly consist of the storage capacity, the accessmodes and the external storage param
pv are not namespaced(its available accross the cluster)

persistent volume claim (created by developers or devops guys)
pods uses pvc claims or consumes storage from pv
key components defined in pvc aare storageclass, requested storage and accessmodes(must be matching with pv)
any pv that matches or satifies the criteria is used
within the pod claimaname must match the pvc name
pvc must be in the same namespace as the pod using the claim

storage class
sc creates pv dynamically whenever pvc claaims it

the key components of storageclass is provisioner(describe which external storage to use)
within pvc yaml the storageclassname should match the storageclass name
in conclustion  pod claims storage from pvc, pvc reqeust storage from sc and sc provision a pv using its provisioner attribute

#configmap aand secret volume types

#stateful and stateless apps
stateful apps such as mongo, mysql elasticsearch etc aaare deployed using statefulset , stateless apps are deployed using deployments
in statefulset apps if you only one pod msql-0 is running, the master pod does the reading and writing into the db
in statefulset apps when 2 or more pods are running workers pods msql-1 and mysql-2 only does reading from the db
in statefulset apps each pod uses different physical storage (pv volumes)
in statefulset apps each pod continually syncs the other to ensure they have the exact copy of data (master changes data as per request from a stateless app, each worrker updates their own data)
in statefulset apps if mysql-3 is added it clones from mysql-2 and does updates with continuos synchronisation with mysql-2
in statefulset apps mysql-0 must be up and running first before the worker starts up, and during deletion the reverse is the cas, mysql-3 gets deleted first
in statefulset apps svc1 load balances aacross the statefulset pods, but each pod also have a individual service name like mysql-0.svc2, mysql-1.svc2 etc
in statefulset apps identity is sticky because when pods restart, ip changes but name aand endpoint stays the same (state and role is retained)
managing statefulst app can be complex, yo'll need to config the cloning and data synchronisation, make remote storage available and managing backup

#managed kubernetes service
a typical usecaase will be building a web app with database
browser with hhtps
securty for cluster
data persistence for db
dev and prod environment

#helm
a typical usecase are complicated and multiple yaml files (elasticsearch, sonarqube, prometheus etc) are packages and bundled in a public repository
also helm uses templating engine to substitute values in yaml file for different enviroment(dev, stage or prod) therefore preventing duplication of yaml files enabling reusability
in a ci/cd pipeline the values can be replacced on the fly
chart directory structure
mychart/ (name of the chart)
    Chart.yaml (metadata of the chart)
    values.yaml (values for the template files)
    charts/ (chart dependencies)
    templates/ (actual template files)
the default values.yaml merges with my-values-override.yaml to produce a .Values object
use command helm install mychart --values my-values-override.yaml or helm install mychart --set version=2.0.0 and helm upgrade(rollback) mychart

helm repo add stable https://helm.nginx.com/stable
helm repo update
helm search repo stable/nginx-ingress
helm install nginx-ingress nginx-stable/nginx-ingress --set controller.publishService.enable=true
kubectl scale --replicas=0 statefulset/mongodb

#image pull secret for private registry
kubectl create secret docker-registry regcred --docker-server=https://index.docker.io/v1/ --docker-username=opeomotayo --docker-password=Ayodeji79

kubernetes operator
operator is used for stateful apps
operator is yaml file that replaces human/software operator in a attempt to automate the manual tasks done by human
this is because kubernetes natively doesn't have the business logic neccessary to manage stateful apps
it manages how to deploy stateful apps, create cluster replicas and how to recover making tasks automated and reusable
operator at it's core has a control loop mechanish that watches(observe, check differences and take action) for changes in stateful apps
it uses CRD (custom resouce definition) to extend the basic kubernetes resouces(staefulset, coonfigmap, service etc)
ideally mysql operator will be created by experts with core working knowledge of mysql, same for postgres,prometheus, elastic etc

operator &helm demo
setup prometheus monitoring in k8s, prometheus server process and stores metrics data and alertmanager send alerts to the likes of email, slack, hipchat etc
grafana is usually used for data analysis visualisation
prometheus operator can be seen as the manager of all the created prometheus components 
