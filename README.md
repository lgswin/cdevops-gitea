# Gitea Deployment Report using Ansible, Helm, Minikube, and Ngrok

## Prerequisites

- This setup must be executed inside a **GitHub Codespace environment** with Docker and Kubernetes support.
- A valid **Ngrok account** is required.
- You must generate and configure your **Ngrok authtoken** from the Ngrok dashboard:
    
    https://dashboard.ngrok.com/get-started/setup
    

---

## values.yaml

- Defines Helm chart configurations used during Gitea deployment
- Sets up an initial admin account for Gitea
- Configures Gitea to use an external MySQL database instead of the default PostgreSQL
- Enables persistent volume for storing Gitea data across pod restarts

---

## up.yaml

- Ansible playbook for automating the deployment of MySQL and Gitea to Kubernetes
- Installs the required `kubernetes` Python package for Ansible
- Applies `mysql.yaml` to deploy MySQL with PersistentVolume and a dedicated service
- Adds the Gitea Helm chart repository: `https://dl.gitea.com/charts/`
- Installs the latest Gitea release using Helm with custom values from `values.yaml`
- Disables PostgreSQL and Redis services to avoid conflict with the external MySQL

---

## Execution Steps

```bash
minikube start
```

- Initializes a local Kubernetes cluster using the Docker driver
- Downloads Kubernetes images and creates the control plane
- Configures the local environment for `kubectl` access
- Enables default addons such as `storage-provisioner`

```bash
ansible-playbook up.yaml
```

- Runs all tasks defined in `up.yaml`
- Installs the Kubernetes Python client
- Deploys MySQL resources including PersistentVolumeClaim
- Adds the Gitea Helm repository and deploys Gitea using Helm
- Uses `values.yaml` to configure Gitea with MySQL and persistent storage

```bash
kubectl get svc
```

- Confirms that the following services are created:
    - `gitea-http`: Gitea web interface (port 3000)
    - `gitea-ssh`: Gitea SSH interface (port 22)
    - `mysql`: MySQL service used by Gitea (port 3306)

```bash
kubectl get pods
```

- Verifies that the Gitea pod and MySQL pod are both in `Running` state
- Indicates that all containers are healthy and ready

```bash
kubectl port-forward svc/gitea-http 3000:3000
```

- Forwards local port 3000 to Gitea's service inside the Kubernetes cluster
- Enables access to the Gitea web UI through `http://localhost:3000`
- Allows login using the pre-configured admin account

---

## Exposing Gitea to the Public using Ngrok

```bash
wget https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-stable-linux-amd64.zip
unzip ngrok-stable-linux-amd64.zip
sudo mv ngrok /usr/local/bin
```

- Installs `ngrok` manually from the official source
- Places the binary into the system path for global use

```bash
ngrok config add-authtoken <NGROK_TOKEN>
```

- Registers your Ngrok account for authenticated tunneling

```bash
kubectl port-forward svc/gitea-http 3000:3000
```

- Forwards port 3000 locally so Ngrok can connect to the service

```bash
ngrok http 3000
```

- Opens a secure public tunnel to [`localhost:3000`](http://localhost:3000) or codespace address
- Ngrok provides a public HTTPS URL (e.g., `https://abc123.ngrok.io`)
- You can now access your Gitea instance from outside the local network