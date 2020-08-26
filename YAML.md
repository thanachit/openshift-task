# Sample YAML

## Application

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example
  namespace: project2
spec:
  selector:
    matchLabels:
      app: hello-openshift
  replicas: 3
  template:
    metadata:
      labels:
        app: hello-openshift
    spec:
      containers:
        - name: hello-openshift
          image: openshift/hello-openshift
          ports:
            - containerPort: 8080
```

## Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: example
  namespace: project2
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```