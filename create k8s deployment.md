# Create a deployment with 3 replicas
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gs-app
  namespace: gs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: gs-app
  template:
    metadata:
      labels:
        app: gs-app
    spec:
      containers:
      - name: gs-app
        image: imagerepo/image			# the desired image
        ports:
        - containerPort: 8088
```