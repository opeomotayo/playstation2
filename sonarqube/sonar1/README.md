#create sonarqube
kubectl create namespace sonarqube
helm upgrade --install -n sonarqube sonarqube sonarqube/sonarqube

ref:
https://git.app.uib.no/caleno/helm-charts/-/tree/master/stable/sonarqube
https://github.com/gudpick/pods.git
