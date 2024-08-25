# fluxcd repo

This repo assumes you already have a kubernetes cluster set up. If you don't, you can run the [ansible playbook](https://github.com/jab416171/ansible-k8s) that I created that will create a cluster from freshly installed Debian servers, either bare metal or VMs.

This was copied from [geek-cookbook](https://geek-cookbook.funkypenguin.co.nz/kubernetes/deployment/flux/install/) and their template repo [here](https://github.com/geek-cookbook/template-flux/). I have added everything I think a base kubernetes cluster should have, in addition to support for intel iGPU, nvidia GPU, and coral TPU. It also sets up PVCs with an nfs server.

## Recommended Changes

- Update the addresses in metallb-config/ipaddresspool.yaml to a range of IPs on your LAN that are free.
- Update server and path in nfs-provisioner/helmrelease-nfs-provisioner.yaml to point to your nfs server. This is the IP and share path for the nfs server.
- Update `cert-manager-config/clusterissuer.yaml` with your email address for letsencrypt.
- Update `cert-manager-system/secret-cert-manager.yaml` with your cloudflare api token.

## Prerequisites

- A working kubernetes cluster
- flux installed locally. You can find the instructions [here](https://fluxcd.io/flux/installation/).
- Once you have flux installed and a cluster set up, you can check if your cluster is able to run flux with the following command:

```bash
flux check --pre
```

## Bootstrap

1. I am using self-hosted gitlab with ssh, so the arguments may be different if you are using github or gitlab.com or another git provider. You will want to adjust the bootstrap command accordingly. You can find the instructions [here](https://fluxcd.io/flux/installation/bootstrap/).
2. For gitlab and github, you will need to set a personal access token. You can either pass it in to the bootstrap command, or simply copy it and the bootstrap command will prompt you for it.

```bash
flux bootstrap gitlab \
  --token-auth \
  --hostname=git.example.com \
  --owner=bob \
  --repository=k8s \
  --branch=main \
  --path=bootstrap \
  --personal \
  --token-auth=false \
  --read-write-key=true \
  --ssh-hostname=git.example.com.
  ```

3. You can check the status of the bootstrap with the following command:

```bash
flux check
```

4. Once the bootstrap is complete, you can check the status of the flux pods with the following command:

```bash
kubectl get pods -n flux-system
```

5. Now that flux is up and running, you can start making changes to the repo and see them reflected in the cluster.

6. To add a new service, at a minimum you will likely want a new namespace at `bootstrap/namespaces`, a file in `bootstrap/kustomizations` and a new folder in the root of the repo with any yaml files you want to be deployed. For a helm chart, this folder should have a helmrelease yaml, and you will want to create a file in `bootstrap/helmrepositories` with the helm repository information.
