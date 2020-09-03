






minikube start --vm-driver=virtualbox

minikube status

minikube dashboard

minikube docker-env

eval $(minikube docker-env)

docker container ls

minikube ssh

docker container ls

exit

kubectl config current-context

kubectl get nodes

minikube stop

minikube start

minikube delete

minikube start \
    --vm-driver=virtualbox \
    --kubernetes-version="v1.9.4"

kubectl version --output=yaml

minikube delete
