# KubeMite

Welcome to KubeMite, the central GitHub organization hosting the Infrastructure as Code (IaC), GitOps manifests, and application source code for my personal infrastructure and website, [yahav.me](https://yahav.me).

This project serves as a comprehensive, living showcase of modern DevOps methodologies, infrastructure automation, and system administration. It demonstrates a complete end-to-end professional workflow, from bare-metal network configuration up to automated application delivery.

## Architecture Overview

The underlying environment is a custom homelab built with a focus on security, high availability, and GitOps methodologies.

* **Hardware & Virtualization:** The infrastructure runs on a [Proxmox](https://www.proxmox.com/) virtual environment hosted on a single Mini-PC.
* **Networking & Security:** Network traffic is routed through an [OPNSense](https://opnsense.org/) firewall installed on a dual-NIC Mini-PC, which, together with a smart switch, manages strict VLAN isolation between management, host, VM, and Kubernetes networks. Remote access to the management and Proxmox layers is securely handled via [Tailscale](https://tailscale.com/).
* **Kubernetes Stack:** The environment runs a highly available [Kubernetes](https://kubernetes.io/) cluster (3 control plane nodes, 3 worker nodes). The cluster is configured with:
  * **CRI:** [containerd](https://containerd.io/).
  * **CNI:** [Cilium](https://cilium.io/), utilizing eBPF, [WireGuard](https://www.wireguard.com/) for inter-cluster encryption, and L2 Announcement for local routing over ARP.
  * **CSI:** [SeaweedFS](https://seaweedfs.com/) for distributed, resource-efficient storage designed to handle massive amounts of small files.
  * **Secret Management:** [External-Secrets Operator](https://external-secrets.io/) pulling dynamically from [Bitwarden Secrets Manager](https://bitwarden.com/products/secrets-manager/).
  * **Traffic & SSL:** [Cert-manager](https://cert-manager.io/) for local certificates, with public traffic securely routed through [Cloudflare Tunnels](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/) to bypass the need for open ingress ports.

## Repositories

This organization is divided into five core repositories that map to the deployment pipeline:

### Infrastructure

* [**kubernetes-golden-image**](https://github.com/KubeMite/kubernetes-golden-image): Uses [Packer](https://developer.hashicorp.com/packer) to build a hardened, pre-configured [Debian](https://www.debian.org/) ISO template for the Proxmox environment. A [GitHub Actions](https://github.com/features/actions) CI/CD pipeline securely connects to the Proxmox server via Tailscale to automatically build and store the template upon a successful run.
* [**kubernetes-cluster-creation**](https://github.com/KubeMite/kubernetes-cluster-creation): Uses [Terraform](https://developer.hashicorp.com/terraform) to provision the 6-node Kubernetes cluster from the Packer template. [Cloud-init](https://cloud-init.io/) handles the initial credentials configuration, bootstrapping and node configuration until [ArgoCD](https://argo-cd.readthedocs.io/en/stable/) takes over the cluster management.

### Deployment & GitOps

* [**gitops**](https://github.com/KubeMite/gitops): The source of truth for the cluster state. ArgoCD monitors this repository to synchronize all applications, [Helm charts](https://helm.sh/), and system configurations (including the Cloudflare remote tunnel and the website) into the Kubernetes environment.

### Application

* [**wiki**](https://github.com/KubeMite/wiki): The source code for [yahav.me](https://yahav.me), built using the [Hugo](https://gohugo.io/) static site generator and the [Blowfish](https://blowfish.page/) theme. The CI pipeline utilizes [Hadolint](https://hadolint.com/) for Dockerfile checks, [Trivy](https://trivy.dev/) for vulnerability scanning, and [ZAP Baseline](https://www.zaproxy.org/) for automated DAST scanning. Merging to the `main` branch automatically builds a new Docker image, and tagging the merged commit triggers a version bump in the Helm chart.
* [**wiki-helm**](https://github.com/KubeMite/wiki-helm): Contains the Helm chart definitions for deploying the website Docker image into Kubernetes. A GitHub Actions workflow tests the chart against a [KinD](https://kind.sigs.k8s.io/) cluster, packages it, and publishes it to a chart registry hosted on [GitHub Pages](https://pages.github.com/) so the ArgoCD operator can detect and pull the latest version.

## The Pipeline

1. **Base Image:** `kubernetes-golden-image` builds the secure OS foundation.
1. **Provisioning:** `kubernetes-cluster-creation` deploys the cluster infrastructure via Terraform.
1. **Content Updates:** Edits to the `wiki` repository trigger container builds and automated security scans.
1. **Packaging:** Successful wiki container builds automatically update the `wiki-helm` chart versions.
1. **Delivery:** ArgoCD detects changes in the `gitops` and `wiki-helm` registries, automatically syncing the live cluster state.

---

*Maintained by Yahav Deutsch — DevOps Engineer specializing in Kubernetes, GitOps pipelines, and infrastructure automation.*
