# Installing Rudr

## Prerequisites 

You will need both `kubectl` and `Helm 3` to install Rudr. Usage of [Chocolatey](https://chocolatey.org/) on Windows and [Homebrew](https://brew.sh/) on OS X can simplify the setup. On Linux, installation steps are different depending of your distributions.

1. Clone the repository. 

    ```bash 
    git clone https://github.com/oam-dev/rudr.git
    ```

2. Install `kubectl`. 
    - [Linux](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-linux) 
    - [MacOS](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-macos)
        ```
        brew install kubernetes-cli
        ```
    - [Windows](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-windows)
        ```
        choco install kubernetes-cli
        ```

3. Install `Helm 3`:
    - Linux
        [Use convenience script](https://helm.sh/docs/intro/install/#from-script)
    - MacOS
        ```
        brew install helm@3.0.0
        ```
    - Windows 
        ```
        choco install kubernetes-helm  --version 3.0.0
        ```

4. As of this writing, the supported versions of Kubernetes are 1.15 and 1.16, so make sure you have a Kubernetes cluster with a compatible version. To get started with a Kubernetes cluster, see the options below: 

    * [Azure Kubernetes Service](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal) 
    * [Alibaba - Container Service for Kubernetes](https://www.alibabacloud.com/help/product/85222.htm)
    * [Google Kuberenetes Engine](https://cloud.google.com/kubernetes-engine/docs/quickstart)

## Installing Rudr Using Helm 3

> Note: In its current version, Rudr will only listen for events in one namespace. This will change in the future. For now, though, you must install Rudr into the namespace into which you will deploy Rudr apps. You may install Rudr multiple times on the same cluster as long as you deploy to a different namespace each time.
 
> Tip: As there are some breaking changes (such as Configuration => ApplicationConfiguration, Component => ComponentSchematic). 
 
> Tip: As there are some breaking changes, if you reinstall Rudr, make sure your old CRDs are deleted. You must do this with `kubectl delete crd`.

1. Helm install Rudr

```console
$ helm install rudr ./charts/rudr --wait
NAME: rudr
LAST DEPLOYED: 2019-08-08 09:00:07.754179 -0600 MDT m=+0.710068733
NAMESPACE: default
STATUS: deployed

NOTES:
Rudr is a Kubernetes controller to manage Configuration CRDs.

It has been successfully installed.
```

This will install the CRDs and the controller into your Kubernetes cluster.

> Use the `--set image.tag=VERSION` to specify the version that you want installed. If you do not specify a version, the latest unstable developer release will be installed.

2. Verifying the Install

You can verify that Rudr is installed by fetching the CRDs:

```console
$ kubectl get crds -l app.kubernetes.io/part-of=core.oam.dev
NAME                                      CREATED AT
applicationconfigurations.core.oam.dev   2019-10-02T19:57:32Z
componentinstances.core.oam.dev          2019-10-02T19:57:32Z
componentschematics.core.oam.dev         2019-10-02T19:57:32Z
healthscopes.core.oam.dev                2019-10-02T19:57:32Z
scopes.core.oam.dev                      2019-10-02T19:57:32Z
traits.core.oam.dev                      2019-10-02T19:57:32Z
```

You should see at least those six CRDs. You can also verify that the Rudr deployment is running:

```console
$ kubectl get deployment rudr
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
rudr   1/1     1            1           2m47s
```

### Upgrading

To upgrade Rudr, typically you only need to use Helm.

> Tip: During the Alpha and Beta phase of Rudr, we recommend also deleting your CRDs manually. You must do this with `kubectl delete crd`.

```console
$ helm upgrade rudr charts/rudr
```

The above will update your Rudr to the latest version.

### Uninstalling

```console
$ helm delete rudr
```

This will leave the CRDs and configurations intact.

**NOTE: When you delete the CRDs, it will delete everything touching Open Application Model from configurations to secrets.**

```console
kubectl delete crd -l app.kubernetes.io/part-of=core.oam.dev
```

The above will delete the CRDs and clean up everything related with Open Application Model.

## Installing Implementations for Traits

Rudr provides several traits, including ingress and autoscaler. However, it does not install default implementations of some of these. This is because they map to primitive Kubernetes features that can be fulfilled by  different controllers.

The best place to find implementations for your traits is [Helm Hub](https://hub.helm.sh/).

### Manual Scaler

The manual scaler trait has no external dependencies.

### Ingress

To successfully use an `ingress` trait, you will need to install one of the Kubernetes Ingress controllers. We recommend [nginx-ingress](https://hub.helm.sh/charts/stable/nginx-ingress).

1. First, add the stable repo to your Helm installation. 

    ```bash
    helm repo add stable https://kubernetes-charts.storage.googleapis.com/
    ```

2. Install the NGINX ingress using Helm 3. 

    ```console
    $ helm install nginx-ingress stable/nginx-ingress
    ```

*Note:* You still must manage your DNS configuration as well. Mapping an ingress to `example.com` will not work if you do not also control the domain mapping for `example.com`.

### Autoscaler

To use the autoscaler trait, you must install a controller for Kubernetes `HorizontalPodAutoscaler`. We recommend [KEDA](https://hub.helm.sh/charts/kedacore/keda).

1. First, add the KEDA repo to your Helm installation. 

    ```bash
    helm repo add kedacore https://kedacore.github.io/charts
    ```

2. Update your Helm repo. 

    ```bash
    helm repo update
    ```

2. Install KEDA on your cluster. 

    ```
    helm install keda kedacore/keda
    ```

## Running for Development

Developers may prefer to run a local copy of the Rudr daemon. To do so:

1. Make sure the CRDs are installed on your target cluster
2. Make sure your current Kubernetes context is set to your target cluster. Rudr will inherit the credentials from this context entry.
3. From the base directory of the code, run `make run`. This will start Rudr in the foreground, running locally, but listening on the remote cluster.

### Development on Windows
On Windows, running for development is advised to be performed under the [Windows Subsystem For Linux](https://docs.microsoft.com/en-us/windows/wsl/install-win10). Following steps are for the default Ubuntu WSL implementation (Ubuntu).
1. Install Cargo  
    `sudo apt install cargo`
2. Install Kubectl (from [official doc](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-using-native-package-management))
    ```
    sudo apt-get update && sudo apt-get install -y apt-transport-https
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
    echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
    sudo apt-get update
    sudo apt-get install -y kubectl
    ```
3. Expose your Windows kubeConfig to WSL by setting the KUBECONFIG variable. By default, (no special kubeconfig setting) the config file is located at $HOME\<username>\.kube\config. If it is so, you can use the following command: 

    `export KUBECONFIG="/mnt/c/Users/<username>/.kube/config"`
4. run `make run`

## Next Steps

Deploy a sample rudr application using the [tutorial](../tutorials/deploy_and_update.md). 

## Appendix

You could check the [appendix doc](appendix.md) to find more information.
