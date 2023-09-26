# Telepresence

## Installation

https://www.telepresence.io/docs/latest/quick-start
https://www.getambassador.io/docs/telepresence/latest/quick-start

### Linux

```bash
curl -fL https://app.getambassador.io/download/tel2oss/releases/download/v2.15.1/telepresence-linux-amd64 -o telepresence

mkdir ${HOME}/.local/bin
chmod a+x telepresence && mv telepresence ${HOME}/.local/bin/telepresence
```

### OSX

```zsh
# Mac: Intel
curl -fL https://app.getambassador.io/download/tel2oss/releases/download/v2.15.1/telepresence-linux-amd64 -o telepresence
# Mac: Silicon
curl -fL https://app.getambassador.io/download/tel2oss/releases/download/v2.15.1/telepresence-linux-arm64 -o telepresence

mkdir ${HOME}/.local/bin
chmod a+x telepresence && mv telepresence ${HOME}/.local/bin/telepresence
```

---

## Usage

### Install to Cluster

```bash
$ telepresence helm install
Traffic Manager installed successfully

$ curl -ik https://kubernetes.default
HTTP/1.1 401 Unauthorized
Cache-Control: no-cache, private
Content-Type: application/json
...
```

```bash
$ telepresence list
...
example-service: ready to intercept (traffic-agent not yet installed)
...
```

* Intercept

```bash
$ telepresence intercept \
  <service-name> \
  --port <local-port>[:<remote-port>] \
  --env-file <path-to-env-file>
```

* Outbound proxy

```bash
$ telepresence connect
Launching Telepresence Daemon v2.3.7 (api v3)
Need root privileges to run "/usr/local/bin/telepresence daemon-foreground /home/<user>/.cache/telepresence/logs '' ''"
[sudo] password:
Connecting to traffic manager...
Connected to context default (https://<cluster public IP>)
```

* Check status

```bash
$ telepresence status
Root Daemon: Running
  Version     : v2.3.7 (api 3)
  Primary DNS : ""
  Fallback DNS: ""
User Daemon: Running
  Version           : v2.3.7 (api 3)
  Ambassador Cloud  : Logged out
  Status            : Connected
  Kubernetes server : https://<cluster public IP>
  Kubernetes context: default
  Telepresence proxy: ON (networking to the cluster is enabled)
  Intercepts        : 0 total

```

---

# OLD

### Launch a proxy to the existing deployment named `core`

```bash
# deployments.apps "core"
NAMESPACE='airuntime'
kubectl -n ${NAMESPACE} get deployments.apps
telepresence \
  --namespace ${NAMESPACE} \
  --new-deployment core \
  --env-json core_env.json

# Available options
  # --serviceaccount \
  # --context \
```

You can debug without modifying the existing deployment, `--new-deployment` option is recommended.

```console
$ telepresence \
    --namespace ${NAMESPACE} \
    --new-deployment core \
    --env-json env.json

NAME   READY   UP-TO-DATE   AVAILABLE   AGE
core   1/1     1            1           3h19m
T: Warning: kubectl 1.20.0 may not work correctly with cluster version 1.17.15-gke.800 due to the version discrepancy. See https://kubernetes.io/docs/setup/version-skew-policy/ for more information.

T: Using a Pod instead of a Deployment for the Telepresence proxy. If you experience problems, please file an issue!
T: Set the environment variable TELEPRESENCE_USE_DEPLOYMENT to any non-empty value to force the old behavior, e.g.,
T:     env TELEPRESENCE_USE_DEPLOYMENT=1 telepresence --run curl hello

T: How Telepresence uses sudo: https://www.telepresence.io/reference/install#dependencies
T: Invoking sudo. Please enter your sudo password.
[sudo] password for ${USER}: 
T: Starting proxy with method 'vpn-tcp', which has the following limitations: All processes are affected, only one telepresence can run per machine, and you can't use other VPNs. You may need to add cloud 
T: hosts and headless services with --also-proxy. For a full list of method limitations see https://telepresence.io/reference/methods.html
T: Volumes are rooted at $TELEPRESENCE_ROOT. See https://telepresence.io/howto/volumes.html for details.
T: Starting network proxy to cluster using new Pod core

T: No traffic is being forwarded from the remote Deployment to your local machine. You can use the --expose option to specify which ports you want to forward.

T: Setup complete. Launching your command.
@gke_ds-ai-platform_us-central1_kfserving-riesling|bash-5.0$ ping systemdb
ping is not supported under Telepresence.
See https://telepresence.io/reference/limitations.html for details.
@gke_ds-ai-platform_us-central1_kfserving-riesling|bash-5.0$ mysql -h ${SYSTEMDB_HOST} -u ${SYSTEMDB_USERNAME} -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 179445
Server version: 10.5.8-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> exit
Bye
@gke_ds-ai-platform_us-central1_kfserving-riesling|bash-5.0$ exit
exit
T: Your process has exited.
T: Exit cleanup in progress
T: Cleaning up Pod
```

In case of `--swap-deployment`:

```console
$ telepresence \
    --namespace ${NAMESPACE} \
    --swap-deployment core \
    --env-json core_env.json

T: Warning: kubectl 1.20.0 may not work correctly with cluster version 1.17.15-gke.800 due to the version discrepancy. See https://kubernetes.io/docs/setup/version-skew-policy/ for more information.

T: Using a Pod instead of a Deployment for the Telepresence proxy. If you experience problems, please file an issue!
T: Set the environment variable TELEPRESENCE_USE_DEPLOYMENT to any non-empty value to force the old behavior, e.g.,
T:     env TELEPRESENCE_USE_DEPLOYMENT=1 telepresence --run curl hello

T: Starting proxy with method 'vpn-tcp', which has the following limitations: All processes are affected, only one telepresence can run per machine, and you can't use other VPNs. You may need to add cloud 
T: hosts and headless services with --also-proxy. For a full list of method limitations see https://telepresence.io/reference/methods.html
T: Volumes are rooted at $TELEPRESENCE_ROOT. See https://telepresence.io/howto/volumes.html for details.
T: Starting network proxy to cluster by swapping out Deployment core with a proxy Pod
T: Forwarding remote port 17005 to local port 17005.

T: Guessing that Services IP range is ['10.8.0.0/20']. Services started after this point will be inaccessible if are outside this range; restart telepresence if you can't access a new Service.
T: Setup complete. Launching your command.
@gke_ds-ai-platform_us-central1_kfserving-riesling|bash-5.0$ exit
exit
T: Your process exited with return code 7.
T: Exit cleanup in progress
T: Cleaning up Pod
```

#### Deployment options

* `--new-deployment`: create a new deployment for the telepresence proxy, preserving the existing deployment, and connect your local machine as a pod. It needs `sudo` privilege.
* `--swap-deployment`: drop the existing deployment and create a new deployment using proxy from local, as a pod. you can launch multiple containers as the pod.
* `--deployment`: re-connect the existing telepresence proxy deployment.

Description:

```ascii
--new-deployment DEPLOYMENT_NAME, -n DEPLOYMENT_NAME
                        Create a new Deployment in Kubernetes where the
                        datawire/telepresence-k8s image will run. It will be
                        deleted on exit. If no deployment option is specified
                        this will be used by default, with a randomly
                        generated name.
  --swap-deployment DEPLOYMENT_NAME[:CONTAINER], -s DEPLOYMENT_NAME[:CONTAINER]
                        Swap out an existing deployment with the Telepresence
                        proxy, swap back on exit. If there are multiple
                        containers in the pod then add the optional container
                        name to indicate which container to use.
  --deployment EXISTING_DEPLOYMENT_NAME, -d EXISTING_DEPLOYMENT_NAME
                        The name of an existing Kubernetes Deployment where
                        the datawire/telepresence-k8s image is already
                        running.
```

