# Deploy web app on Kubernetes

## Use yaml to create deployment in k8s

- use docker image to package a app
- push docker image to dockerhub
- use yaml to pull the image 
```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: result
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: result
    spec:
      containers:
      - image: dockersamples/examplevotingapp_result:before
        name: result
```
- `kubectl create -f deployment.yaml`
## Use yaml to create a service and expose it

```yaml
apiVersion: v1
kind: Service
metadata:
  name: result
spec:
  type: NodePort
  ports:
  - name: "result-service"
    port: 5001
    targetPort: 80
    nodePort: 31001
  selector:
    app: result
```
- kubectl create -f service.yaml

