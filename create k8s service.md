# To create a k8s service create .yaml file with below config
```
apiVersion: v1
kind: Service
metadata:
  name: gs-svc
  namespace: gs
spec:
  type: LoadBalancer
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  selector:
    app: gs-app
```