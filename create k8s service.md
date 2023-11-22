# Creating kubernetes service
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
