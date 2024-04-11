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
        
    