#Lab 00: Prerequisites

To be completed if Docker, Kubectl, or Helm not installed, or if you'd like a local Kubernetes cluster.

## A. Install Docker

Docker has a convenient installation script if you're on a *nix-based system. If you're on Windows, [Docker Desktop](https://docs.docker.com/desktop/install/windows-install/) is recommended.
1. In a terminal, download the Docker install script.

```
curl -fsSL https://get.docker.com -o get-docker.sh
```

2. Run the Docker install script.  
sh get-docker.sh

3. Set permissions so you can run Docker without sudo.

```
sudo groupadd docker
sudo usermod -aG docker $USER
```

4. For the permissions to take effect, logout and back in and/or open a new user shell.

5. Test the Docker installation.

```
docker run hello-world
```

Let your instructor know if you get a permissions error and/or are unable to run Docker without sudo.

## B. Install kubectl 

Depending on your system, follow [these instructions](https://kubernetes.io/docs/tasks/tools/#kubectl) to install kubectl. The below steps are for Ubuntu x86_64.

1. Update the package manager.

```
sudo apt-get update
```

2. Install dependencies.

```
sudo apt-get install -y apt-transport-https ca-certificates curl
```

3. Download public signing key.

```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

4. Add the Kubernetes repository.

```
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

5. Update package index.

```
sudo apt-get update
```

6. Install kubectl.

```
sudo apt-get install -y kubectl
```

7. Check for successful kubectl installation.

```
kubectl version --client
```

## C. Install Helm

Helm offers a script that can install the package via a one-liner. Refer to the [Helm docs](https://helm.sh/docs/intro/install/) for more detailed platform specific install instructions.

1. Install Helm from the installer script.

```
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

2. Verify installation.

```
helm version
```

## D. (Optional) Install k3d

k3d offers a nifty tool that lets you run Kubernetes on your laptop via Docker.

1. Install k3d via the installer script.

```
curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
```

2. Verify installation.

```
k3d version
```

3. Create a local cluster.

```
k3d cluster create mycluster
```

4. Verify cluster.

```
k3d cluster list
kubectl cluster-info
helm list
```











