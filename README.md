# consul-setup-poc
This project contains the Helm chart for starting a single node Consul cluster as well as kubernetes scripts to launch the API Gateway, 
Test Service and the User Service.

- [API Gateway](https://github.com/aardee/apigateway-nest) uses Nest Middleware to authenticate the user using User Service, 
generate a new internal JWT token and then proxy the request to the Test Service
- [Test Service](https://github.com/aardee/test-service-poc) This microservice simulates the APIs exposed to the client app. `/apis/test/headers` returns all the headers back to the client. 
- [User Service](https://github.com/aardee/user-service-poc) - API Gateway uses this service to authenticate the user and get the additional details about the user.

## Start Kubernetes environment
```bash
# Start the minikube
minikube start --memory 8192

# Check the status, confirm that kubernetes server is ready
minikube status
```

## Start the dashboard
```bash
 minikube service consul-server-consul-ui
```

## Running the app
```bash
# Clone the project
git@github.com:aardee/consul-setup-poc.git

cd consul-setup-poc

# run the helm chart to start the consul cluster (consists of 1 server node only)
helm install consul-server -f helm-consul-values.yaml ./consul-helm

# Check the list of services
minikube service list

# (Optinally) Launch the consul UI. Ensure that "consul-server-consul-ui" is listed in the minikube services
minikube service consul-server-consul-ui 
```

## Start POC services
```bash
# Start services
kubectl create -f poc-services

# Check status of the pods
kubectl get pods

# Once all PODs are running, check the services. Confirm that apigateway-service-load-balancer is running and has IP assigned to it.
 minikube service list
 
# Get the URL assigned to apigateway-service-load-balancer and run the following curl command (this assumes jq utility is installed)
curl --location --request GET 'http://192.168.64.8:31103/apis/test/headers' --header 'Authorization: Bearer sdiicHHBX1reBeqeE2hAPg' | jq

# You should see x-apigateway-authorization header returned by the test service
"x-apigateway-authorization": "token eyJhbGciOiJIUzI1NiJ9.ewogICAgICAgICAgICAiZW1haWwiOiJhZG1pbkBwb2MuY29tIiwKICAgICAgICAgICAgImZpcnN0X25hbWUiOiJKYW5lIiwKICAgICAgICAgICAgImxhc3RfbmFtZSI6IkRvZSIsCiAgICAgICAgICAgICJyb2xlcyI6WyJBRE1JTiIsIlVTRVIiLCJFRElUT1IiXQogICAgICAgIH0.PCDrA8v4_hMCvFm2wTosy0QjmC18k2QX6BYFSMh_QPA"
```
