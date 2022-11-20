# Common DevOps utilities in a docker image

## What's inside

- AWS CLI
- Terraform
- Kubectl
- Helm
- JQ
- YQ

Versions are defined as ARG's in the Dockerfile

## How to use it

### AWS CLI
- Pass AWS credentials via the .aws directory
```
_> docker run --rm -i \
     -v "$(pwd):/workspace" \
     -v "${HOME}/.aws:/root/.aws" \
     kickthemooon/utils \
     aws s3 ls
```
- Pass temporary AWS credentials as env variables
```
_> env | grep AWS_ > /tmp/aws-temp-creds && \
   docker run --rm -i \
     -v "$(pwd):/workspace" \
     --env-file="/tmp/aws-temp-creds" \
     kickthemooon/utils \
     aws s3 ls 
```

### Kubectl
- Pass a kubernetes configuration via .kube
```
_> docker run --rm -i \
     -v "$(pwd):/workspace" \
     -v "${HOME}/.kube:/root/.kube" \
     kickthemooon/utils \
     kubectl
```
- Specify a kube config file
```
_> KUBECONFIG="$HOME/.kube/my-kube-config-file" docker run --rm -i \
     -v "$(pwd):/workspace" \
     -v "${HOME}/.kube:/root/.kube" \
     -e "KUBECONFIG=${KUBECONFIG/$HOME/\/root}" \
     kickthemooon/utils \
     kubectl
```

### Helm, JQ and YQ
Helm, JQ and YQ are ready to use without any configuration e.g.:
```
docker run --rm -i kickthemooon/utils jq
```

## Add a shortcut
To avoid typing all that stuff every time, we can add a shell function to our profile:
```
# devops utils
dou() {
  # create .kube 
  if [ ! -d "$HOME/.kube" ]; then
    mkdir -p $HOME/.kube
  fi
  
  # create .kube/config
  if [ ! -f "$HOME/.kube/config" ]; then
    touch $HOME/.kube/config
  fi
  
  # adjust KUBECONFIG
  if [ -z "${KUBECONFIG}" ]; then
    KUBECONFIG="/root/.kube/config"
  fi
  
  env | grep AWS_ > /tmp/aws-temp-creds 
  
  docker run --rm -i \
    --env-file="/tmp/aws-temp-creds" \
    -v "${HOME}/.kube:/root/.kube" \
    -e "KUBECONFIG=${KUBECONFIG/$HOME/\/root}" \
    -v "$(pwd):/workspace" \
    kickthemooon/utils \
    $@
}
```