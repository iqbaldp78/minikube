# minikube
this repo used for step by step installing minikube in our local devices


```note : you need to install minikube first. refer to this docs : https://minikube.sigs.k8s.io/docs/start/```

## Starting Minikube
- goto your terminal
- `$ minikube start`
- `$ minikube dashboard` => for running GUI of kubernetes

## Deploy Apps
- `eval $(minikube docker-env)`

    it outputs a set of environment variables that configure your local Docker client to interact with the Docker daemon running inside the Minikube virtual machine.
- build your local docker images

    ```docker build -f "docker/Dockerfile" -t radiance-k8s .```
- verify your docker images already build

    `docker images` => your can see your image here. 

    `note: if your image not present here, you need to run eval $(minikube docker-env) and then build your image again`

- create ConfigMap resources
    - global env

        `$ kubectl create configmap global-env --from-env-file=./app.env`
        
    - radiance env

        `$ kubectl create configmap radiance-env --from-env-file=./workspaces/radiance/.env`

- cd to `sources/global` directory
    `$ kubectl create -f namespace.yaml`
- ex : cd to `radiance/`

    `$ kubectl apply -f deployment.yaml`

- exposing your app
    - create resources k8s, called `services`

        `$ kubectl expose deployment [YOUR_DEPLOYMENT_NAME] --name=[YOUR_SERVICE_NAME] --type=NodePort --port=8080`
    
- access your app
    - access your app via NodePort resources
        - via port-forwarding

            `$ kubectl port-forward deployment/radiance-local [HOST_PORT]:[APP_PORT]`
            
            you can access your apps via `127.0.0.1:[HOST_PORT]`
        - via minikube url

            `$ minikube service <service-name> --url`
    - access your app via LoadBalancer
        - create tunel 
        
            `$ minikube tunnel`

            `ps : dont note stop this terminal`
        - `$ kubectl apply -f deployment.yaml`
        - `$ kubectl expose deployment [YOUR_DEPLOYMENT_NAME] --name=[YOUR_SERVICE_NAME] --type=LoadBalancer --port=8080`
        - `kubectl get svc`
        - go to browser and `http://REPLACE_WITH_EXTERNAL_IP:8080`

## Deploy Global Manifest
`$cd sources\global> kubectl -f apply global-manifest.yaml`

## Deploy Glimmer (kong api gateway)
- build docker images 

    `$ docker build -f "workspace/glimmer/Dockerfile" --build-arg USERNAME=xxxxx --build-arg PASSWORD=xxxxx --build-arg SANGE_BRANCH=staging -t glimmer-k8s .
    `
- create personal access token in gembus (gitlab) to pull image registry
    - go to `https://gembus.mbizmarket.my.id:13443/-/profile/personal_access_tokens` with your login username and password
    - create a new one access token and check `read_repository` and `wrire_repository`
    - create minikube secret resources
        - `kubectl create secret docker-registry gitlab-stguat --docker-server=gembus.mbizmarket.my.id:14567  --namespace=dmp --docker-username=xxxx --docker-password="xxxx" --docker-email=xxxx`

        `ps : please change docker-username, docker-password, docker-email with your login user gembus`
    - deploy kong postgres 
        `$ cd glimmer> kubectl apply -f postgres.yaml`
    - deploy kong (glimmer) `$ cd glimmer> kubectl apply -f glimmer-deployment.yaml`

- expose your services with port forwarding

    `kubectl port-forward svc/glimmer 8070:80 -n dmp-local`

## Deploy Radiance 
user services manifest 
- `$ cd radiance> kubectl apply -f deployment.yaml`
- test your service 

    `curl --location 'localhost:8070/users/user/active' \
--header 'Authorization: Bearer eyJ0eXAiOiJhcHAiLCJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJzeXN0ZW0iLCJhdXRoIjoiUk9MRV9BUFBMSUNBVElPTiIsImV4cCI6MzIwMzk5MzgwMn0.Y7YcBUw-SIpKnkLq6rUMUj0z-sKaEffsRhNcoM8fA-nQzD5TS8iX4W234NqihRZkEb4Q7kQe_AXnvxC4H0jN8A'`

    
