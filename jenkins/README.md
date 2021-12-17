#create jenkins
helm -n jenkins install jenkins jenkins/jenkins -f values-override.yaml
helm -n jenkins upgrade --install jenkins jenkins/jenkins -f values-override.yaml
#delete jenkins
helm delete jenkins

#get password
printf $(kubectl get secret --namespace jenkins jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo


kubectl -n jenkins exec -it jenkins-0 -c config-reload -- cat /var/jenkins_home/casc_configs/jcasc-default-config.yaml
kubectl -n jenkins exec -it jenkins-0 -c jenkins -- cat /var/jenkins_home/casc_configs/jcasc-default-config.yaml
```console
helm repo add jenkins https://charts.jenkins.io
```
You can then run `helm search repo jenkins` to see the charts.

ref:
https://github.com/helm/charts/tree/master/stable/jenkins
https://github.com/jenkinsci/kubernetes-plugin/tree/master/examples
https://github.com/marcel-dempers/docker-development-youtube-series/tree/master/jenkins




https://github.com/jenkinsci/job-dsl-plugin/blob/master/docs/JCasC.md
jobs:
- script: >
job('seed') {
scm {
git {
extensions {
wipeOutWorkspace()
}
remote {
url('https://github.com/opeomotayo/jenkins-course.git')
// credentials('github-creds')
}
branch('*/master')
}
}
triggers {
scm('*****')
}
steps {
jobDsl {
targets('job-dsl/*.groovy')
// additionalClasspath('srv/main/groovy')
removedJobAction('DELETE')
removedViewAction('DELETE')
lookupStrategy('JENKINS_ROOT')
}
}
}


