

kubectl apply -f environments/develop/namespace.yaml 

kubectl apply -f secrets/dockerhubsecret.yaml 

kubectl apply -f secrets/postgres-credentials.yaml

kubectl apply -f secrets/app-secrets.yaml

kubectl apply -f  postgres/volume.yaml 

kubectl apply -f  postgres/volume-claim.yaml 

kubectl apply -f  postgres/deployment.yaml 

kubectl apply -f  postgres/service.yaml 

kubectl apply -f  environments/develop/deployment.yaml 

kubectl apply -f  environments/develop/service.yaml 

### OR

kubectl delete -f environments/develop/namespace.yaml && \
kubectl delete -f postgres/volume.yaml && \
kubectl delete -f postgres/volume-claim.yaml && \
kubectl delete -f secrets/app-secrets.yaml && \
kubectl delete -f secrets/postgres-credentials.yaml && \
kubectl delete -f secrets/dockerhubsecret.yaml && \
kubectl delete -f postgres/service.yaml && \
kubectl delete -f postgres/deployment.yaml && \
kubectl delete -f environments/develop/service.yaml && \
kubectl delete -f environments/develop/deployment.yaml 

