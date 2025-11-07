# Creating an Ubuntu 24.04 VM on Linux
This guide explains how to create an Ubuntu 24.04 virtual machine instance on a Linux host using `virt-install`.

```bash
mkdir ubuntu24-test
cd buntu24-test
wget https://cloud-images.ubuntu.com/noble/current/noble-server-cloudimg-amd64.img
cat > user-data <<EOF
#cloud-config
users:
  - name: ubuntu
    plain_text_passwd: 'ubuntu'
    lock_passwd: false
    sudo: ALL=(ALL) NOPASSWD:ALL
    shell: /bin/bash

chpasswd:
  list: |
    ubuntu:ubuntu
  expire: False

ssh_pwauth: true
EOF

cat > meta-data <<EOF
instance-id: ubuntu24-test
local-hostname: ubuntu24-test
EOF

cloud-localds seed.iso user-data meta-data
qemu-img resize noble-server-cloudimg-amd64.img +10G

sudo virt-install \
  --name ubuntu24-test \
  --memory 2048 \
  --vcpus 2 \
  --disk path=noble-server-cloudimg-amd64.img,format=qcow2 \
  --disk path=seed.iso,device=cdrom,readonly=on \
  --os-type linux \
  --os-variant ubuntu24.04 \
  --graphics none \
  --network network=default \
  --import
```

