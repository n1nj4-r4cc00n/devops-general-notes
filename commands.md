###Verify the virtualization support on your Linux OS (a non-empty output indicates supported virtualization):
> $ grep -E --color 'vmx|svm' /proc/cpuinfo

###Enable docker for non-root users (post installation)
1. https://docs.docker.com/engine/install/linux-postinstall/
> sudo usermod -aG docker $USER && newgrp docker (add the current user $USER to a new docker group)
----
>newgrp docker (applies changes)