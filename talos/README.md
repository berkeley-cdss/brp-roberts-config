# Generating the Talos configuration

## To generate the controlplane config:

`talosctl gen secrets --output-file secrets.yaml`

`talosctl gen config roberts https://128.32.135.223:6443 --with-secrets secrets.yaml --config-patch @general-patch.yaml --config-patch @general-cp-patch.yaml -t controlplane --output controlplane.yaml`

`talosctl gen config roberts https://128.32.135.223:6443 --with-secrets secrets.yaml --config-patch @general-patch.yaml -t worker --output worker.yaml`

## To generate the talosconfig to be able to run talosctl commands against the cluster:

`talosctl gen config roberts https://128.32.135.223:6443 --with-secrets secrets.yaml -t talosconfig`

This will generate the `talosconfig` files in the current directory. Copy it to `~/.talos/config` and add endpoints (`???`) and optionally nodes.

## To generate the kubectl config:

`talosctl kubeconfig --merge=false -n 128.32.135.223 -e 128.32.135.223:50000 --talosconfig=talosconfig`

(read docs about merging with the current kubectl config and other options)

---

talosctl apply-config --nodes 128.32.135.223 --file controlplane.yaml --config-patch @k8s-m-0.yaml -e 128.32.135.223:50000 --insecure

talosctl bootstrap --nodes 128.32.135.223 -e 128.32.135.223 --talosconfig talosconfig

---
## Calico:

kubectl create namespace tigera-operator

<!--kubectl label ns tigera-operator pod-security.kubernetes.io/enforce=privileged-->

!!!! TODO: ADD IPPOOL !!!!

<!--helm install --create-namespace calico projectcalico/tigera-operator --version v3.32.1 --namespace tigera-operator-->

helm install calico projectcalico/tigera-operator --version v3.31.5 --namespace tigera-operator --create-namespace

---

network:
  ethernets:
    enp33s0np0:
      addresses:
        - 10.16.32.10/24
        - 2620:83:8000:157:1000::/64
      nameservers:
        addresses:
          - 131.243.5.1
          - 2620:83:8000:a:53::1
      routes:
        - to: "default"
          via: 10.16.32.1
        - to: "::/0"
          via: 2620:83:8000:157::1

---

k8s-set: `ssh -J sls.lbl.gov,scs-otp1.lbl.gov hpcs@10.16.31.8 -A`

---

AMD:

https://factory.talos.dev/?arch=amd64&platform=metal&schematic-id=9c1d1b442d73f96dcd04e81463eb20000ab014062d22e1b083e1773336bc1dd5&target=metal&version=1.13.6

Your image schematic ID is: 
9c1d1b442d73f96dcd04e81463eb20000ab014062d22e1b083e1773336bc1dd5

 customization:
    systemExtensions:
        officialExtensions:
            - siderolabs/amd-ucode
            - siderolabs/amdgpu

---

talosctl reset --nodes 128.32.135.223 -e 128.32.135.223 --system-labels-to-wipe EPHEMERAL --reboot

Disk wipe:

talosctl wipe disk --nodes 128.32.135.223 -e 128.32.135.223 dm-0 --insecure --skip-secondary-check --drop-partition
