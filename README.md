# Qfaas-setup

This repository contains steps to reproduce the QFaas project in a minikube cluster. <br/>
Visit the project [QFaaS: Accelerating and Securing Serverless Cloud
Networks with QUIC](http://faculty.washington.edu/wlloyd/courses/tcss562/papers/QFaaS-AcceleratingandSecuringSeNetworksWithQUIC.pdf )

<br/>

## Cluster Setup

See [OpenFaas page](https://docs.openfaas.com/deployment/kubernetes/) for cluster options. 

<br/>
Install OpenFaas Community Edition using arkade(recommended)

```
# install arkade
curl -SLsf https://get.arkade.dev/ | sudo sh
arkade install openfaas

# install faas-cli
curl -SLsf https://cli.openfaas.com | sudo sh

# wait for succesful rollout status
kubectl rollout status -n openfaas deploy/gateway

# forward gateway svc port to 8080
kubectl port-forward -n openfaas svc/gateway 8080:8080 &

# If basic auth is enabled, you can now log into your gateway:
PASSWORD=$(kubectl get secret -n openfaas basic-auth -o jsonpath="{.data.basic-auth-password}" | base64 --decode; echo)
echo -n $PASSWORD | faas-cli login --username admin --password-stdin
```
<br/>

## Use Qfaas version of faas-netes

```
git clone https://github.com/qfaas-project/faas-netes.git
cd faas-netes
```
1. Change the 1st line of Dockerfile, from
```FROM teamserverless/license-check:0.3.9 as license-check ``` to ```FROM ghcr.io/openfaas/license-check:0.4.2 as license-check ```

2. Now, build and push the docker image 
```
export USERNAME="alexellis"
docker build -t $USERNAME/faas-netes:qfaas .
docker push $USERNAME/faas-netes:qfaas
```
3. Run the arkade command with new faas-netes image
```
arkade install openfaas \
--set faasnetes.image=$USERNAME/faas-netes:qfaas
```
<br/>

## Deploy Hello-Retail
Test the hello-retail application on the original OpenFaas. [Hello Retail](https://github.com/qfaas-project/hello-retail)

```
git clone https://github.com/qfaas-project/hello-retail.git 
cd hello-retail
```
<br/> **NOTE:** The repository requires minor edits to be made before usage.
- Since deployed functions need to be pulled from dockerhub, set dockerhub credentials in your environment. See [Private Registries](https://docs.openfaas.com/reference/private-registries/) for more info.
  ```
  export DOCKER_USERNAME=<your_docker_username>
  export DOCKER_PASSWORD=<your_docker_password>
  export DOCKER_EMAIL=<your_docker_email>

  # create the secret for dockerhub in openfaas-fn namespace
  kubectl create secret docker-registry dockerhub \
    -n openfaas-fn \
    --docker-username=$DOCKER_USERNAME \
    --docker-password=$DOCKER_PASSWORD \
    --docker-email=$DOCKER_EMAIL
  ```
- in `deploy/deploy_mysql.sh` file, change the user password (pass to password) and run the script. <br/>
  ```
  kubectl run -n openfaas-fn --restart=Never --image=mysql:5.6 mysql-client-temp -- mysql -h mysql -ppassword -e "CREATE USER ..
  ./deploy_mysql.sh
  ```

- in the `functions/` folder, for each protocol subdirectory(tcp, tls, quic), edit the YAML files in all directories to reflect the following change for Gateway <br/>
  ``` gateway: http://192.168.122.138:31112 ``` to ``` gateway: http://127.0.0.1:8080 ```
<br/>



Finally, run the Python script (pass protocol and action as paramaters):

```
python3 deploy_functions.py --protocol tcp --task deploy
```
You should now be able to invoke any of the deployed function using ```faas-cli invoke <function-name>```

<br/><br/>

  
  
  

