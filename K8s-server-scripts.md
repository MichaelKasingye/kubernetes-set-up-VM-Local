

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

kubectl apply -f environments/develop/namespace.yaml &&

kubectl apply -f secrets/dockerhubsecret.yaml &&

kubectl apply -f secrets/postgres-credentials.yaml &&

kubectl apply -f secrets/app-secrets.yaml &&

kubectl apply -f postgres/volume.yaml &&

kubectl apply -f postgres/volume-claim.yaml &&

kubectl apply -f postgres/deployment.yaml &&

kubectl apply -f postgres/service.yaml &&

kubectl apply -f environments/develop/deployment.yaml &&

kubectl apply -f environments/develop/service.yaml
