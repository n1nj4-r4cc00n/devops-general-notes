###Website
https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
###Install
>curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" && sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
 ---
###If no root access
> chmod +x kubectl
mkdir -p ~/.local/bin/kubectl
mv ./kubectl ~/.local/bin/kubectl
> #######and then add ~/.local/bin/kubectl to $PATH

##Checking Config
> kubectl cluster-info dump

###Install kubectl convert plugin
> curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl-convert &&
sudo install -o root -g root -m 0755 kubectl-convert /usr/local/bin/kubectl-convert