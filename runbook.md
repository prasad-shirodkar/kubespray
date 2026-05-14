# Pre-requisites
Install docker and git

# set ssh private key
SSH_PRIVATE_KEY="<insert the full path to ssh private key>"

# Setup
git clone --branch aranya-k8s-scratch https://github.com/prasad-shirodkar/kubespray.git
cd kubespray

TAG="v2.31.0"
docker pull quay.io/kubespray/kubespray:$TAG
docker run --rm -it --mount type=bind,source="$(pwd)"/inventory/mycluster,dst=/inventory \
  --mount type=bind,source="$SSH_PRIVATE_KEY",dst=/root/.ssh/privatekey \
  quay.io/kubespray/kubespray:$TAG bash -c 'ansible-playbook -i /inventory/inventory.ini --private-key /root/.ssh/privatekey --become --become-user=root cluster.yml'

# Install Gateway API CRDS - needed for cert-manager
kubectl apply --server-side -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.5.0/standard-install.yaml

# Install clusterdos install manifest
kubectl apply -f clusterdos/install.yaml

# nginx demo app
kubectl apply -f nginx-demo.yaml

# Teardown
docker run --rm -it --mount type=bind,source="$(pwd)"/inventory/mycluster,dst=/inventory \
  --mount type=bind,source="$SSH_PRIVATE_KEY",dst=/root/.ssh/privatekey \
  quay.io/kubespray/kubespray:$TAG bash -c 'ansible-playbook -i /inventory/inventory.ini --private-key /root/.ssh/privatekey --become --become-user=root reset.yml'