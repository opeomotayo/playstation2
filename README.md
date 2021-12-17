
#fix terminating issue
kubectl patch pvc db-pv-claim -p '{"metadata":{"finalizers":null}}'
kubectl patch pod db-74755f6698-8td72 -p '{"metadata":{"finalizers":null}}'

# playstation
