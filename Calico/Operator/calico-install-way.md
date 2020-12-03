There are many different ways to install Calico. The Calico docs site includes recommended install options across a range of environments, so you don’t need to be an expert to get started.

Broadly there are 4 different approaches to install.

Manifest

This is the most basic method for installing Calico. The Calico docs include a range of manifests for different environments. If you are an advanced user, you can customize the manifests to give you ultimate flexibility and control over your installation. 

Operator

Calico 3.15 introduces the option to install Calico using an open-source operator, created by Tigera. This offers simplification for installing and configuring Calico without needing to customize manifests. Additionally, the operator allows you to have a uniform, self-healing environment. Using the Tigera operator is highly recommended.

Managed Kubernetes Services

Support for Calico is included with many of the most popular managed Kubernetes services (e.g. EKS, AKS, GKE, IKS), either enabled by default, or optionally enabled using the cloud provider’s management consoles or command line tools, depending on the specific managed Kubernetes service.

Kubernetes Distros and Installers

Many Kubernetes distros and installers include support for installing Calico. (e.g. kops, kubespray, microk8s, etc). Most of these currently use manifest based installs under the covers.
