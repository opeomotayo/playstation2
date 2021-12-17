https://github.com/prometheus-community/helm-charts

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

authorization with rbac
role only defines resources and their access permissions, they are limited to a namespace, it doesn't include who gets the access
rolebinding links/binds a roleref with kind role to a subjects with kind user(karl)/group(dev)/serviceaccount(default)
cluserrole defines resouces i.e (nodes, namespaces & namespacced resoucres line pod, services etc)  and its cluster-wide permissions/verbs i.e (get, create, list, delete and updaate), they are not limited to a namespace, it's cluster-wide access
clusterrolebinding links/binds a cluserrole to a adminuser(ope)group(devops)/serviceaccount(default)
external sources for authentication/creating users are via static token file, certificates or 3rd third identity service(ldap)
an admin/devops member cconfigres on the above external sources
api server handles all the authentication requests coming from one of these external sources i.e kube-apiserver --token-auth-file=/users.csv
serviceacccount is a component created within the cluter to authorise/give acccess/permissions to services/apps running within or outside the cluster
also serviceaccount component can be linked to a role/clusterrole with their corresponding rolebinding/clusterrolebinding, 
the permissions associated to the role/clusterrole will be passed on to the serviceaccount
example of resouces aare pod, seccrets, configmap, examples of verbs are get, create, list, delete etc
role is scoped to a namespace by adding the the naamespace in the metadata section
you can set a more granular access using the resourcename (i.e myapp) attribute
using kubectl auth can-i get/view/list/create/delete pods/secrets/deployments/services --namespace dev etc or cheack specific user's access or permissions
a typical request from jenkins -> apiserver -> checks for authentication(can jenkins user/sa connect to the cluster) 
->  checks for authorisation using rbac(within the defined rbac what can this user do within the cluster )