## Cluster Setup
Setup a minikube cluster using docker driver. See [OpenFaas page](https://docs.openfaas.com/deployment/kubernetes/) for other cluster options. <br/>
``` minikube -p openfaas –nodes 2 –disk-size 12G –memory 16G ```

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
<br/><br/>
## Deploy Hello-Retail
Test the hello-retail application on the original OpenFaas. [Hello Retail](https://github.com/qfaas-project/hello-retail)

```
git clone https://github.com/qfaas-project/hello-retail.git 
cd hello-retail
```
<br/> **NOTE:** The repository has minor edits to be made before using.
- Since deployed functions need to be pulled from dockerhub, set dockerhub credentials in your system beforehand.
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
- at ./deploy/deploy_mysql.sh file, change the user password (pass to password). <br/>
  ```
  kubectl run -n openfaas-fn --restart=Never --image=mysql:5.6 mysql-client-temp -- mysql -h mysql -ppass -e "CREATE USER ...
  ```
  to 
  ```
  kubectl run -n openfaas-fn --restart=Never --image=mysql:5.6 mysql-client-temp -- mysql -h mysql -ppassword -e "CREATE USER ..
  ```

- in the ./functions/ folder, for each protocol subdirectory(tcp, tls, quic), change the yaml file in all directories with the name 'product-purchase-goapi' <br/>
  ``` gateway: http://192.168.122.138:31112 ``` to ``` gateway: http://127.0.0.1:8080 ```
<br/>
Finally, run the ./deploy/deploy_functions.py file by passing the protocol you want to run for the function:

```
cd deploy
python3 deploy_functions.py --protocol tcp --task deploy
```
  
  
  

