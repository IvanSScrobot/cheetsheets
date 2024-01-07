### Set the kubectl completion 
kubectl completion bash | sudo tee /etc/bash_completion.d/kubectl > /dev/null


### Set an alias for kubectl 
echo 'alias k=kubectl' >>~/.bashrc


### Enable the kubelet alias for auto-completion.
echo 'complete -o default -F __start_kubectl k' >>~/.bashrc