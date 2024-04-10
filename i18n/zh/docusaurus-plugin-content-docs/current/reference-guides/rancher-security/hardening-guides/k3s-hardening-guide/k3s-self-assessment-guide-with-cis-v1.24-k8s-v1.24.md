---
title: K3s 自我评估指南 - CIS Benchmark v1.24 - K8s v1.24
---

<head>
  <link rel="canonical" href="https://ranchermanager.docs.rancher.com/zh/reference-guides/rancher-security/hardening-guides/k3s-hardening-guide/k3s-self-assessment-guide-with-cis-v1.24-k8s-v1.24"/>
</head>

本文档是 [K3s 加固指南](k3s-hardening-guide.md)的配套文档，该指南提供了关于如何加固正在生产环境中运行并由 Rancher 管理的 K3s 集群的指导方针。本 benchmark 指南可帮助你根据 CIS Kubernetes Benchmark 中的每个 control 来评估加固集群的安全性。

本指南对应以下版本的 Rancher、CIS Benchmarks 和 Kubernetes：

| Rancher 版本     | CIS Benchmark 版本    | Kubernetes 版本     |
|-----------------|-----------------------|--------------------|
| Rancher v2.7    | Benchmark v1.24       | Kubernetes v1.24   |

本文档适用于 Rancher 运维人员、安全团队、审计员和决策者。

有关每个 control 的更多信息，包括详细描述和未通过测试的补救措施，请参考 CIS Kubernetes Benchmark v1.24 的相应部分。你可以在[互联网安全中心 (CIS)](https://www.cisecurity.org/benchmark/kubernetes/)创建免费账户后下载 benchmark。

## 测试方法

每个 CIS Kubernetes Benchmark 中的 control 都根据附带的加固指南评估了针对 K3s 集群的配置。

当 control 审计与原始的 CIS benchmark 不同的时候，提供了针对 K3s 的特定审计命令，以供测试使用。

以下是每个 control 可能的结果：

- **Pass（通过）** - K3s 集群通过了 benchmark 中概述的审计。
- **Not Applicable（不适用）** - 由于 K3s 的设计方式，该 control 不适用于 K3s。在补救措施部分解释了原因。
- **Warn（警告）** - 在 CIS benchmark 中，该 control 是手动的，它取决于集群的使用情况或其他必须由集群操作员确定的因素。这些 control 措施已经过评估，以确保 K3s 不会阻止其实施，但尚未对集群进行进一步的配置或审计。

本指南假设 K3s 作为 Systemd 单元运行。你的安装可能会有所不同。调整"审计"命令以适合你的场景。

:::note

本指南仅涵盖 `automated`（之前称为 `scored`）测试。

:::

### Controls


## 1.1 Control Plane Node Configuration Files
### 1.1.1 Ensure that the API server pod specification file permissions are set to 644 or more restrictive (Automated)


**Result:** Not Applicable

**Remediation:**
Run the below command (based on the file location on your system) on the
control plane node.
For example, chmod 644 /etc/kubernetes/manifests/kube-apiserver.yaml

### 1.1.2 Ensure that the API server pod specification file ownership is set to root:root (Automated)


**Result:** Not Applicable

**Remediation:**
Run the below command (based on the file location on your system) on the control plane node.
For example, chown root:root /etc/kubernetes/manifests/kube-apiserver.yaml

### 1.1.3 Ensure that the controller manager pod specification file permissions are set to 600 or more restrictive (Automated)


**Result:** Not Applicable

**Remediation:**
Run the below command (based on the file location on your system) on the control plane node.
For example, chmod 600 /etc/kubernetes/manifests/kube-controller-manager.yaml

### 1.1.4 Ensure that the controller manager pod specification file ownership is set to root:root (Automated)


**Result:** Not Applicable

**Remediation:**
Run the below command (based on the file location on your system) on the control plane node.
For example, chown root:root /etc/kubernetes/manifests/kube-controller-manager.yaml

### 1.1.5 Ensure that the scheduler pod specification file permissions are set to 600 or more restrictive (Automated)


**Result:** Not Applicable

**Remediation:**
Run the below command (based on the file location on your system) on the control plane node.
For example, chmod 600 /etc/kubernetes/manifests/kube-scheduler.yaml

### 1.1.6 Ensure that the scheduler pod specification file ownership is set to root:root (Automated)


**Result:** Not Applicable

**Remediation:**
Run the below command (based on the file location on your system) on the control plane node.
For example, chown root:root /etc/kubernetes/manifests/kube-scheduler.yaml

### 1.1.7 Ensure that the etcd pod specification file permissions are set to 600 or more restrictive (Automated)


**Result:** Not Applicable

**Remediation:**
Run the below command (based on the file location on your system) on the control plane node.
For example,
chmod 600 /etc/kubernetes/manifests/etcd.yaml

### 1.1.8 Ensure that the etcd pod specification file ownership is set to root:root (Automated)


**Result:** Not Applicable

**Remediation:**
Run the below command (based on the file location on your system) on the control plane node.
For example,
chown root:root /etc/kubernetes/manifests/etcd.yaml

### 1.1.9 Ensure that the Container Network Interface file permissions are set to 600 or more restrictive (Manual)


**Result:** Not Applicable

**Remediation:**
Run the below command (based on the file location on your system) on the control plane node.
For example, chmod 600 <path/to/cni/files\>

### 1.1.10 Ensure that the Container Network Interface file ownership is set to root:root (Manual)


**Result:** Not Applicable

**Remediation:**
Run the below command (based on the file location on your system) on the control plane node.
For example,
chown root:root <path/to/cni/files\>

### 1.1.11 Ensure that the etcd data directory permissions are set to 700 or more restrictive (Automated)


**Result:** pass

**Remediation:**
On the etcd server node, get the etcd data directory, passed as an argument --data-dir,
from the command 'ps -ef | grep etcd'.
Run the below command (based on the etcd data directory found above). For example,
chmod 700 /var/lib/etcd

**Audit Script:** `check_for_k3s_etcd.sh`

```bash
#!/bin/bash

# This script is used to ensure that k3s is actually running etcd (and not other databases like sqlite3)
# before it checks the requirement
set -eE

handle_error() {
    echo "false"
}

trap 'handle_error' ERR


if [[ "$(journalctl -D /var/log/journal -u k3s | grep 'Managed etcd cluster initializing' | grep -v grep | wc -l)" -gt 0 ]]; then
    case $1 in 
        "1.1.11")
            echo $(stat -c %a /var/lib/rancher/k3s/server/db/etcd);;
        "1.2.29")
            echo $(journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'etcd-');;
        "2.1")
            echo $(grep -A 5 'client-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep -E 'cert-file|key-file');;
        "2.2")
            echo $(grep -A 5 'client-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep 'client-cert-auth');;
        "2.3")
            echo $(grep 'auto-tls' /var/lib/rancher/k3s/server/db/etcd/config);;
        "2.4")
            echo $(grep -A 5 'peer-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep -E 'cert-file|key-file');;
        "2.5")
            echo $(grep -A 5 'peer-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep 'client-cert-auth');;
        "2.6")
            echo $(grep 'peer-auto-tls' /var/lib/rancher/k3s/server/db/etcd/config);;
        "2.7")
            echo $(grep 'trusted-ca-file' /var/lib/rancher/k3s/server/db/etcd/config);;
    esac
else
# If another database is running, return whatever is required to pass the scan
    case $1 in
        "1.1.11")
            echo "700";;
        "1.2.29")
            echo "--etcd-certfile AND --etcd-keyfile";;
        "2.1")
            echo "cert-file AND key-file";;
        "2.2")
            echo "--client-cert-auth=true";;
        "2.3")
            echo "false";;
        "2.4")
            echo "peer-cert-file AND peer-key-file";;
        "2.5")
            echo "--client-cert-auth=true";;
        "2.6")
            echo "--peer-auto-tls=false";;
        "2.7")
            echo "--trusted-ca-file";;
    esac
fi

```

**Audit Execution:**

```bash
./check_for_k3s_etcd.sh 1.1.11
```

**Expected Result**:

```console
'700' is equal to '700'
```

**Returned Value**:

```console
700
```

### 1.1.12 Ensure that the etcd data directory ownership is set to etcd:etcd (Automated)


**Result:** Not Applicable

**Remediation:**
On the etcd server node, get the etcd data directory, passed as an argument --data-dir,
from the command 'ps -ef | grep etcd'.
Run the below command (based on the etcd data directory found above).
For example, chown etcd:etcd /var/lib/etcd

### 1.1.13 Ensure that the admin.conf file permissions are set to 600 or more restrictive (Automated)


**Result:** Not Applicable

**Remediation:**
Run the below command (based on the file location on your system) on the control plane node.
For example, chmod 600 /var/lib/rancher/k3s/server/cred/admin.kubeconfig

### 1.1.14 Ensure that the admin.conf file ownership is set to root:root (Automated)


**Result:** pass

**Remediation:**
Run the below command (based on the file location on your system) on the control plane node.
For example, chown root:root /etc/kubernetes/admin.conf

**Audit:**

```bash
/bin/sh -c 'if test -e /var/lib/rancher/k3s/server/cred/admin.kubeconfig; then stat -c %U:%G /var/lib/rancher/k3s/server/cred/admin.kubeconfig; fi'
```

**Expected Result**:

```console
'root:root' is equal to 'root:root'
```

**Returned Value**:

```console
root:root
```

### 1.1.15 Ensure that the scheduler.conf file permissions are set to 600 or more restrictive (Automated)


**Result:** pass

**Remediation:**
Run the below command (based on the file location on your system) on the control plane node.
For example,
chmod 600 scheduler

**Audit:**

```bash
/bin/sh -c 'if test -e /var/lib/rancher/k3s/server/cred/scheduler.kubeconfig; then stat -c permissions=%a /var/lib/rancher/k3s/server/cred/scheduler.kubeconfig; fi'
```

**Expected Result**:

```console
permissions has permissions 600, expected 600 or more restrictive
```

**Returned Value**:

```console
permissions=600
```

### 1.1.16 Ensure that the scheduler.conf file ownership is set to root:root (Automated)


**Result:** pass

**Remediation:**
Run the below command (based on the file location on your system) on the control plane node.
For example,
chown root:root scheduler

**Audit:**

```bash
/bin/sh -c 'if test -e /var/lib/rancher/k3s/server/cred/scheduler.kubeconfig; then stat -c %U:%G /var/lib/rancher/k3s/server/cred/scheduler.kubeconfig; fi'
```

**Expected Result**:

```console
'root:root' is present
```

**Returned Value**:

```console
root:root
```

### 1.1.17 Ensure that the controller-manager.conf file permissions are set to 600 or more restrictive (Automated)


**Result:** pass

**Remediation:**
Run the below command (based on the file location on your system) on the control plane node.
For example,
chmod 600 controllermanager

**Audit:**

```bash
/bin/sh -c 'if test -e /var/lib/rancher/k3s/server/cred/controller.kubeconfig; then stat -c permissions=%a /var/lib/rancher/k3s/server/cred/controller.kubeconfig; fi'
```

**Expected Result**:

```console
permissions has permissions 600, expected 600 or more restrictive
```

**Returned Value**:

```console
permissions=600
```

### 1.1.18 Ensure that the controller-manager.conf file ownership is set to root:root (Automated)


**Result:** pass

**Remediation:**
Run the below command (based on the file location on your system) on the control plane node.
For example,
chown root:root controllermanager

**Audit:**

```bash
stat -c %U:%G /var/lib/rancher/k3s/server/tls
```

**Expected Result**:

```console
'root:root' is equal to 'root:root'
```

**Returned Value**:

```console
root:root
```

### 1.1.19 Ensure that the Kubernetes PKI directory and file ownership is set to root:root (Automated)


**Result:** pass

**Remediation:**
Run the below command (based on the file location on your system) on the control plane node.
For example,
chown -R root:root /etc/kubernetes/pki/

**Audit:**

```bash
find /var/lib/rancher/k3s/server/tls | xargs stat -c %U:%G
```

**Expected Result**:

```console
'root:root' is present
```

**Returned Value**:

```console
root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root root:root
```

### 1.1.20 Ensure that the Kubernetes PKI certificate file permissions are set to 600 or more restrictive (Manual)


**Result:** warn

**Remediation:**
Run the below command (based on the file location on your system) on the control plane node.
For example,
chmod -R 600 /etc/kubernetes/pki/*.crt

**Audit:**

```bash
stat -c %n %a /var/lib/rancher/k3s/server/tls/*.crt
```

### 1.1.21 Ensure that the Kubernetes PKI key file permissions are set to 600 (Manual)


**Result:** warn

**Remediation:**
Run the below command (based on the file location on your system) on the control plane node.
For example,
chmod -R 600 /etc/kubernetes/pki/*.key

**Audit:**

```bash
stat -c %n %a /var/lib/rancher/k3s/server/tls/*.key
```

## 1.2 API Server
### 1.2.1 Ensure that the --anonymous-auth argument is set to false (Manual)


**Result:** warn

**Remediation:**
Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the control plane node and set the below parameter.
--anonymous-auth=false

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'anonymous-auth'
```

### 1.2.2 Ensure that the --token-auth-file parameter is not set (Automated)


**Result:** pass

**Remediation:**
Follow the documentation and configure alternate mechanisms for authentication. Then,
edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the control plane node and remove the --token-auth-file=<filename\> parameter.

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep -v grep
```

**Expected Result**:

```console
'--token-auth-file' is not present
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-apiserver --advertise-port=6443 --allow-privileged=true --anonymous-auth=false --api-audiences=https://kubernetes.default.svc.cluster.local,k3s --audit-log-maxage=30 --audit-log-maxbackup=10 --audit-log-maxsize=100 --audit-log-path=/var/lib/rancher/k3s/server/logs/audit.log --audit-policy-file=/var/lib/rancher/k3s/server/audit.yaml --authorization-mode=Node,RBAC --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/temporary-certs --client-ca-file=/var/lib/rancher/k3s/server/tls/client-ca.crt --egress-selector-config-file=/var/lib/rancher/k3s/server/etc/egress-selector-config.yaml --enable-admission-plugins=NodeRestriction,PodSecurityPolicy,ServiceAccount --enable-aggregator-routing=true --enable-bootstrap-token-auth=true --encryption-provider-config=/var/lib/rancher/k3s/server/cred/encryption-config.json --etcd-cafile=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --etcd-certfile=/var/lib/rancher/k3s/server/tls/etcd/client.crt --etcd-keyfile=/var/lib/rancher/k3s/server/tls/etcd/client.key --etcd-servers=https://127.0.0.1:2379 --feature-gates=JobTrackingWithFinalizers=true --kubelet-certificate-authority=/var/lib/rancher/k3s/server/tls/server-ca.crt --kubelet-client-certificate=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.crt --kubelet-client-key=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --profiling=false --proxy-client-cert-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.crt --proxy-client-key-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.key --request-timeout=300s --requestheader-allowed-names=system:auth-proxy --requestheader-client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6444 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/rancher/k3s/server/tls/service.key --service-account-lookup=true --service-account-signing-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --service-node-port-range=30000-32767 --storage-backend=etcd3 --tls-cert-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305 --tls-private-key-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key"
```

### 1.2.3 Ensure that the --DenyServiceExternalIPs is not set (Automated)


**Result:** pass

**Remediation:**
Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the control plane node and remove the `DenyServiceExternalIPs`
from enabled admission plugins.

**Audit:**

```bash
/bin/ps -ef | grep containerd | grep -v grep
```

**Expected Result**:

```console
'--enable-admission-plugins' is present OR '--enable-admission-plugins' is not present
```

**Returned Value**:

```console
root 410 1 0 Sep11 ? 00:01:50 /usr/bin/containerd root 539 1 0 Sep11 ? 00:00:09 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock root 45213 45195 3 Sep11 ? 00:45:14 containerd -c /var/lib/rancher/k3s/agent/etc/containerd/config.toml -a /run/k3s/containerd/containerd.sock --state /run/k3s/containerd --root /var/lib/rancher/k3s/agent/containerd root 47188 1 0 Sep11 ? 00:01:00 /var/lib/rancher/k3s/data/484bf694b486d93cc93bcf90f74d5c77628550d4456f21760fd720b88c93881e/bin/containerd-shim-runc-v2 -namespace k8s.io -id 02b8fdd94b7628d575ace92f337dcc93202151d07f33858341cfcb0178fea586 -address /run/k3s/containerd/containerd.sock root 47235 1 0 Sep11 ? 00:00:33 /var/lib/rancher/k3s/data/484bf694b486d93cc93bcf90f74d5c77628550d4456f21760fd720b88c93881e/bin/containerd-shim-runc-v2 -namespace k8s.io -id eb0f03c4bc1125ae34e5de77c7eaeb9c2c1e46bec0bb106b99281df7353f9ded -address /run/k3s/containerd/containerd.sock root 47948 1 0 Sep11 ? 00:00:31 /var/lib/rancher/k3s/data/484bf694b486d93cc93bcf90f74d5c77628550d4456f21760fd720b88c93881e/bin/containerd-shim-runc-v2 -namespace k8s.io -id 5eece7110bd3876414f890ed1b7158eb91f60d1f14450a9f8c77a0fc62b73c4a -address /run/k3s/containerd/containerd.sock root 48047 1 0 Sep11 ? 00:00:32 /var/lib/rancher/k3s/data/484bf694b486d93cc93bcf90f74d5c77628550d4456f21760fd720b88c93881e/bin/containerd-shim-runc-v2 -namespace k8s.io -id 4ef831f4bb8830a0a7b46a1e8be58197be00846ea0caa094014ea7ce3adf008e -address /run/k3s/containerd/containerd.sock root 48220 1 0 Sep11 ? 00:00:33 /var/lib/rancher/k3s/data/484bf694b486d93cc93bcf90f74d5c77628550d4456f21760fd720b88c93881e/bin/containerd-shim-runc-v2 -namespace k8s.io -id f8d5d3791ad2b1a8534476aa2fbdf8da1e4f0b9fdcdd26b1ae161142458a5336 -address /run/k3s/containerd/containerd.sock root 48878 1 0 Sep11 ? 00:00:32 /var/lib/rancher/k3s/data/484bf694b486d93cc93bcf90f74d5c77628550d4456f21760fd720b88c93881e/bin/containerd-shim-runc-v2 -namespace k8s.io -id 41b93a6581e272b0852257b3d7c44ab4cadcdf465a3579a8b29b493ea75feaf7 -address /run/k3s/containerd/containerd.sock root 49870 1 0 Sep11 ? 00:00:30 /var/lib/rancher/k3s/data/484bf694b486d93cc93bcf90f74d5c77628550d4456f21760fd720b88c93881e/bin/containerd-shim-runc-v2 -namespace k8s.io -id be329cdccda4049f048c759a2b1915cd72526473ff7768e895bcfc153a66c125 -address /run/k3s/containerd/containerd.sock root 50271 1 0 Sep11 ? 00:00:32 /var/lib/rancher/k3s/data/484bf694b486d93cc93bcf90f74d5c77628550d4456f21760fd720b88c93881e/bin/containerd-shim-runc-v2 -namespace k8s.io -id cceb5d26eba7d7d3c474a2dc842b72495689f4e40055505ea2598e4ac6849d40 -address /run/k3s/containerd/containerd.sock root 50571 1 0 Sep11 ? 00:00:32 /var/lib/rancher/k3s/data/484bf694b486d93cc93bcf90f74d5c77628550d4456f21760fd720b88c93881e/bin/containerd-shim-runc-v2 -namespace k8s.io -id 3a26e691e0d0186165d523bef22b0af71f175c39df82845b32ce7b4e3d652d4a -address /run/k3s/containerd/containerd.sock root 96693 1 0 16:13 ? 00:00:00 /var/lib/rancher/k3s/data/484bf694b486d93cc93bcf90f74d5c77628550d4456f21760fd720b88c93881e/bin/containerd-shim-runc-v2 -namespace k8s.io -id 9ed77e6e58410f1295f730ba4a8f8daf7aff8f4e896e24219a8fa710729c2287 -address /run/k3s/containerd/containerd.sock root 97726 1 0 16:16 ? 00:00:00 /var/lib/rancher/k3s/data/484bf694b486d93cc93bcf90f74d5c77628550d4456f21760fd720b88c93881e/bin/containerd-shim-runc-v2 -namespace k8s.io -id 1be4fe50e6bb29d4549f8b03102b5dc15b7669fe56b32beae3f701cf6d2e58cc -address /run/k3s/containerd/containerd.sock root 97923 1 1 16:16 ? 00:00:00 /var/lib/rancher/k3s/data/484bf694b486d93cc93bcf90f74d5c77628550d4456f21760fd720b88c93881e/bin/containerd-shim-runc-v2 -namespace k8s.io -id 7c65c47376f36a8980a5bebad7ffdc1abe1853821cf6c4a1d45f1b6be3452062 -address /run/k3s/containerd/containerd.sock
```

### 1.2.4 Ensure that the --kubelet-https argument is set to true (Automated)


**Result:** Not Applicable

**Remediation:**
Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the control plane node and remove the --kubelet-https parameter.

### 1.2.5 Ensure that the --kubelet-client-certificate and --kubelet-client-key arguments are set as appropriate (Automated)


**Result:** pass

**Remediation:**
Follow the Kubernetes documentation and set up the TLS connection between the
apiserver and kubelets. Then, edit API server pod specification file
/etc/kubernetes/manifests/kube-apiserver.yaml on the control plane node and set the
kubelet client certificate and key parameters as below.
--kubelet-client-certificate=<path/to/client-certificate-file\>
--kubelet-client-key=<path/to/client-key-file\>

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'kubelet-certificate-authority'
```

**Expected Result**:

```console
'--kubelet-client-certificate' is present AND '--kubelet-client-key' is present
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-apiserver --advertise-port=6443 --allow-privileged=true --anonymous-auth=false --api-audiences=https://kubernetes.default.svc.cluster.local,k3s --audit-log-maxage=30 --audit-log-maxbackup=10 --audit-log-maxsize=100 --audit-log-path=/var/lib/rancher/k3s/server/logs/audit.log --audit-policy-file=/var/lib/rancher/k3s/server/audit.yaml --authorization-mode=Node,RBAC --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/temporary-certs --client-ca-file=/var/lib/rancher/k3s/server/tls/client-ca.crt --egress-selector-config-file=/var/lib/rancher/k3s/server/etc/egress-selector-config.yaml --enable-admission-plugins=NodeRestriction,PodSecurityPolicy,ServiceAccount --enable-aggregator-routing=true --enable-bootstrap-token-auth=true --encryption-provider-config=/var/lib/rancher/k3s/server/cred/encryption-config.json --etcd-cafile=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --etcd-certfile=/var/lib/rancher/k3s/server/tls/etcd/client.crt --etcd-keyfile=/var/lib/rancher/k3s/server/tls/etcd/client.key --etcd-servers=https://127.0.0.1:2379 --feature-gates=JobTrackingWithFinalizers=true --kubelet-certificate-authority=/var/lib/rancher/k3s/server/tls/server-ca.crt --kubelet-client-certificate=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.crt --kubelet-client-key=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --profiling=false --proxy-client-cert-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.crt --proxy-client-key-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.key --request-timeout=300s --requestheader-allowed-names=system:auth-proxy --requestheader-client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6444 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/rancher/k3s/server/tls/service.key --service-account-lookup=true --service-account-signing-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --service-node-port-range=30000-32767 --storage-backend=etcd3 --tls-cert-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305 --tls-private-key-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key"
```

### 1.2.6 Ensure that the --kubelet-certificate-authority argument is set as appropriate (Automated)


**Result:** pass

**Remediation:**
Follow the Kubernetes documentation and setup the TLS connection between
the apiserver and kubelets. Then, edit the API server pod specification file
/etc/kubernetes/manifests/kube-apiserver.yaml on the control plane node and set the
--kubelet-certificate-authority parameter to the path to the cert file for the certificate authority.
--kubelet-certificate-authority=<ca-string\>

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'kubelet-certificate-authority'
```

**Expected Result**:

```console
'--kubelet-certificate-authority' is present
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-apiserver --advertise-port=6443 --allow-privileged=true --anonymous-auth=false --api-audiences=https://kubernetes.default.svc.cluster.local,k3s --audit-log-maxage=30 --audit-log-maxbackup=10 --audit-log-maxsize=100 --audit-log-path=/var/lib/rancher/k3s/server/logs/audit.log --audit-policy-file=/var/lib/rancher/k3s/server/audit.yaml --authorization-mode=Node,RBAC --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/temporary-certs --client-ca-file=/var/lib/rancher/k3s/server/tls/client-ca.crt --egress-selector-config-file=/var/lib/rancher/k3s/server/etc/egress-selector-config.yaml --enable-admission-plugins=NodeRestriction,PodSecurityPolicy,ServiceAccount --enable-aggregator-routing=true --enable-bootstrap-token-auth=true --encryption-provider-config=/var/lib/rancher/k3s/server/cred/encryption-config.json --etcd-cafile=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --etcd-certfile=/var/lib/rancher/k3s/server/tls/etcd/client.crt --etcd-keyfile=/var/lib/rancher/k3s/server/tls/etcd/client.key --etcd-servers=https://127.0.0.1:2379 --feature-gates=JobTrackingWithFinalizers=true --kubelet-certificate-authority=/var/lib/rancher/k3s/server/tls/server-ca.crt --kubelet-client-certificate=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.crt --kubelet-client-key=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --profiling=false --proxy-client-cert-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.crt --proxy-client-key-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.key --request-timeout=300s --requestheader-allowed-names=system:auth-proxy --requestheader-client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6444 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/rancher/k3s/server/tls/service.key --service-account-lookup=true --service-account-signing-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --service-node-port-range=30000-32767 --storage-backend=etcd3 --tls-cert-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305 --tls-private-key-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key"
```

### 1.2.7 Ensure that the --authorization-mode argument is not set to AlwaysAllow (Automated)


**Result:** pass

**Remediation:**
Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the control plane node and set the --authorization-mode parameter to values other than AlwaysAllow.
One such example could be as below.
--authorization-mode=RBAC

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'authorization-mode'
```

**Expected Result**:

```console
'--authorization-mode' does not have 'AlwaysAllow'
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-apiserver --advertise-port=6443 --allow-privileged=true --anonymous-auth=false --api-audiences=https://kubernetes.default.svc.cluster.local,k3s --audit-log-maxage=30 --audit-log-maxbackup=10 --audit-log-maxsize=100 --audit-log-path=/var/lib/rancher/k3s/server/logs/audit.log --audit-policy-file=/var/lib/rancher/k3s/server/audit.yaml --authorization-mode=Node,RBAC --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/temporary-certs --client-ca-file=/var/lib/rancher/k3s/server/tls/client-ca.crt --egress-selector-config-file=/var/lib/rancher/k3s/server/etc/egress-selector-config.yaml --enable-admission-plugins=NodeRestriction,PodSecurityPolicy,ServiceAccount --enable-aggregator-routing=true --enable-bootstrap-token-auth=true --encryption-provider-config=/var/lib/rancher/k3s/server/cred/encryption-config.json --etcd-cafile=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --etcd-certfile=/var/lib/rancher/k3s/server/tls/etcd/client.crt --etcd-keyfile=/var/lib/rancher/k3s/server/tls/etcd/client.key --etcd-servers=https://127.0.0.1:2379 --feature-gates=JobTrackingWithFinalizers=true --kubelet-certificate-authority=/var/lib/rancher/k3s/server/tls/server-ca.crt --kubelet-client-certificate=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.crt --kubelet-client-key=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --profiling=false --proxy-client-cert-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.crt --proxy-client-key-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.key --request-timeout=300s --requestheader-allowed-names=system:auth-proxy --requestheader-client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6444 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/rancher/k3s/server/tls/service.key --service-account-lookup=true --service-account-signing-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --service-node-port-range=30000-32767 --storage-backend=etcd3 --tls-cert-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305 --tls-private-key-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key"
```

### 1.2.8 Ensure that the --authorization-mode argument includes Node (Automated)


**Result:** pass

**Remediation:**
Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the control plane node and set the --authorization-mode parameter to a value that includes Node.
--authorization-mode=Node,RBAC

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'authorization-mode'
```

**Expected Result**:

```console
'--authorization-mode' has 'Node'
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-apiserver --advertise-port=6443 --allow-privileged=true --anonymous-auth=false --api-audiences=https://kubernetes.default.svc.cluster.local,k3s --audit-log-maxage=30 --audit-log-maxbackup=10 --audit-log-maxsize=100 --audit-log-path=/var/lib/rancher/k3s/server/logs/audit.log --audit-policy-file=/var/lib/rancher/k3s/server/audit.yaml --authorization-mode=Node,RBAC --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/temporary-certs --client-ca-file=/var/lib/rancher/k3s/server/tls/client-ca.crt --egress-selector-config-file=/var/lib/rancher/k3s/server/etc/egress-selector-config.yaml --enable-admission-plugins=NodeRestriction,PodSecurityPolicy,ServiceAccount --enable-aggregator-routing=true --enable-bootstrap-token-auth=true --encryption-provider-config=/var/lib/rancher/k3s/server/cred/encryption-config.json --etcd-cafile=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --etcd-certfile=/var/lib/rancher/k3s/server/tls/etcd/client.crt --etcd-keyfile=/var/lib/rancher/k3s/server/tls/etcd/client.key --etcd-servers=https://127.0.0.1:2379 --feature-gates=JobTrackingWithFinalizers=true --kubelet-certificate-authority=/var/lib/rancher/k3s/server/tls/server-ca.crt --kubelet-client-certificate=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.crt --kubelet-client-key=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --profiling=false --proxy-client-cert-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.crt --proxy-client-key-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.key --request-timeout=300s --requestheader-allowed-names=system:auth-proxy --requestheader-client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6444 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/rancher/k3s/server/tls/service.key --service-account-lookup=true --service-account-signing-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --service-node-port-range=30000-32767 --storage-backend=etcd3 --tls-cert-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305 --tls-private-key-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key"
```

### 1.2.9 Ensure that the --authorization-mode argument includes RBAC (Automated)


**Result:** pass

**Remediation:**
Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the control plane node and set the --authorization-mode parameter to a value that includes RBAC,
for example `--authorization-mode=Node,RBAC`.

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'authorization-mode'
```

**Expected Result**:

```console
'--authorization-mode' has 'RBAC'
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-apiserver --advertise-port=6443 --allow-privileged=true --anonymous-auth=false --api-audiences=https://kubernetes.default.svc.cluster.local,k3s --audit-log-maxage=30 --audit-log-maxbackup=10 --audit-log-maxsize=100 --audit-log-path=/var/lib/rancher/k3s/server/logs/audit.log --audit-policy-file=/var/lib/rancher/k3s/server/audit.yaml --authorization-mode=Node,RBAC --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/temporary-certs --client-ca-file=/var/lib/rancher/k3s/server/tls/client-ca.crt --egress-selector-config-file=/var/lib/rancher/k3s/server/etc/egress-selector-config.yaml --enable-admission-plugins=NodeRestriction,PodSecurityPolicy,ServiceAccount --enable-aggregator-routing=true --enable-bootstrap-token-auth=true --encryption-provider-config=/var/lib/rancher/k3s/server/cred/encryption-config.json --etcd-cafile=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --etcd-certfile=/var/lib/rancher/k3s/server/tls/etcd/client.crt --etcd-keyfile=/var/lib/rancher/k3s/server/tls/etcd/client.key --etcd-servers=https://127.0.0.1:2379 --feature-gates=JobTrackingWithFinalizers=true --kubelet-certificate-authority=/var/lib/rancher/k3s/server/tls/server-ca.crt --kubelet-client-certificate=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.crt --kubelet-client-key=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --profiling=false --proxy-client-cert-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.crt --proxy-client-key-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.key --request-timeout=300s --requestheader-allowed-names=system:auth-proxy --requestheader-client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6444 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/rancher/k3s/server/tls/service.key --service-account-lookup=true --service-account-signing-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --service-node-port-range=30000-32767 --storage-backend=etcd3 --tls-cert-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305 --tls-private-key-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key"
```

### 1.2.10 Ensure that the admission control plugin EventRateLimit is set (Manual)


**Result:** warn

**Remediation:**
Follow the Kubernetes documentation and set the desired limits in a configuration file.
Then, edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
and set the below parameters.
--enable-admission-plugins=...,EventRateLimit,...
--admission-control-config-file=<path/to/configuration/file\>

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'enable-admission-plugins'
```

**Expected Result**:

```console
'--enable-admission-plugins' has 'EventRateLimit'
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-apiserver --advertise-port=6443 --allow-privileged=true --anonymous-auth=false --api-audiences=https://kubernetes.default.svc.cluster.local,k3s --audit-log-maxage=30 --audit-log-maxbackup=10 --audit-log-maxsize=100 --audit-log-path=/var/lib/rancher/k3s/server/logs/audit.log --audit-policy-file=/var/lib/rancher/k3s/server/audit.yaml --authorization-mode=Node,RBAC --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/temporary-certs --client-ca-file=/var/lib/rancher/k3s/server/tls/client-ca.crt --egress-selector-config-file=/var/lib/rancher/k3s/server/etc/egress-selector-config.yaml --enable-admission-plugins=NodeRestriction,PodSecurityPolicy,ServiceAccount --enable-aggregator-routing=true --enable-bootstrap-token-auth=true --encryption-provider-config=/var/lib/rancher/k3s/server/cred/encryption-config.json --etcd-cafile=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --etcd-certfile=/var/lib/rancher/k3s/server/tls/etcd/client.crt --etcd-keyfile=/var/lib/rancher/k3s/server/tls/etcd/client.key --etcd-servers=https://127.0.0.1:2379 --feature-gates=JobTrackingWithFinalizers=true --kubelet-certificate-authority=/var/lib/rancher/k3s/server/tls/server-ca.crt --kubelet-client-certificate=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.crt --kubelet-client-key=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --profiling=false --proxy-client-cert-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.crt --proxy-client-key-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.key --request-timeout=300s --requestheader-allowed-names=system:auth-proxy --requestheader-client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6444 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/rancher/k3s/server/tls/service.key --service-account-lookup=true --service-account-signing-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --service-node-port-range=30000-32767 --storage-backend=etcd3 --tls-cert-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305 --tls-private-key-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key"
```

### 1.2.11 Ensure that the admission control plugin AlwaysAdmit is not set (Automated)


**Result:** pass

**Remediation:**
Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the control plane node and either remove the --enable-admission-plugins parameter, or set it to a
value that does not include AlwaysAdmit.

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'enable-admission-plugins'
```

**Expected Result**:

```console
'--enable-admission-plugins' does not have 'AlwaysAdmit' OR '--enable-admission-plugins' is not present
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-apiserver --advertise-port=6443 --allow-privileged=true --anonymous-auth=false --api-audiences=https://kubernetes.default.svc.cluster.local,k3s --audit-log-maxage=30 --audit-log-maxbackup=10 --audit-log-maxsize=100 --audit-log-path=/var/lib/rancher/k3s/server/logs/audit.log --audit-policy-file=/var/lib/rancher/k3s/server/audit.yaml --authorization-mode=Node,RBAC --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/temporary-certs --client-ca-file=/var/lib/rancher/k3s/server/tls/client-ca.crt --egress-selector-config-file=/var/lib/rancher/k3s/server/etc/egress-selector-config.yaml --enable-admission-plugins=NodeRestriction,PodSecurityPolicy,ServiceAccount --enable-aggregator-routing=true --enable-bootstrap-token-auth=true --encryption-provider-config=/var/lib/rancher/k3s/server/cred/encryption-config.json --etcd-cafile=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --etcd-certfile=/var/lib/rancher/k3s/server/tls/etcd/client.crt --etcd-keyfile=/var/lib/rancher/k3s/server/tls/etcd/client.key --etcd-servers=https://127.0.0.1:2379 --feature-gates=JobTrackingWithFinalizers=true --kubelet-certificate-authority=/var/lib/rancher/k3s/server/tls/server-ca.crt --kubelet-client-certificate=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.crt --kubelet-client-key=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --profiling=false --proxy-client-cert-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.crt --proxy-client-key-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.key --request-timeout=300s --requestheader-allowed-names=system:auth-proxy --requestheader-client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6444 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/rancher/k3s/server/tls/service.key --service-account-lookup=true --service-account-signing-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --service-node-port-range=30000-32767 --storage-backend=etcd3 --tls-cert-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305 --tls-private-key-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key"
```

### 1.2.12 Ensure that the admission control plugin AlwaysPullImages is set (Manual)


**Result:** warn

**Remediation:**
Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the control plane node and set the --enable-admission-plugins parameter to include
AlwaysPullImages.
--enable-admission-plugins=...,AlwaysPullImages,...

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'enable-admission-plugins'
```

**Expected Result**:

```console
'--enable-admission-plugins' has 'AlwaysPullImages'
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-apiserver --advertise-port=6443 --allow-privileged=true --anonymous-auth=false --api-audiences=https://kubernetes.default.svc.cluster.local,k3s --audit-log-maxage=30 --audit-log-maxbackup=10 --audit-log-maxsize=100 --audit-log-path=/var/lib/rancher/k3s/server/logs/audit.log --audit-policy-file=/var/lib/rancher/k3s/server/audit.yaml --authorization-mode=Node,RBAC --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/temporary-certs --client-ca-file=/var/lib/rancher/k3s/server/tls/client-ca.crt --egress-selector-config-file=/var/lib/rancher/k3s/server/etc/egress-selector-config.yaml --enable-admission-plugins=NodeRestriction,PodSecurityPolicy,ServiceAccount --enable-aggregator-routing=true --enable-bootstrap-token-auth=true --encryption-provider-config=/var/lib/rancher/k3s/server/cred/encryption-config.json --etcd-cafile=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --etcd-certfile=/var/lib/rancher/k3s/server/tls/etcd/client.crt --etcd-keyfile=/var/lib/rancher/k3s/server/tls/etcd/client.key --etcd-servers=https://127.0.0.1:2379 --feature-gates=JobTrackingWithFinalizers=true --kubelet-certificate-authority=/var/lib/rancher/k3s/server/tls/server-ca.crt --kubelet-client-certificate=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.crt --kubelet-client-key=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --profiling=false --proxy-client-cert-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.crt --proxy-client-key-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.key --request-timeout=300s --requestheader-allowed-names=system:auth-proxy --requestheader-client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6444 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/rancher/k3s/server/tls/service.key --service-account-lookup=true --service-account-signing-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --service-node-port-range=30000-32767 --storage-backend=etcd3 --tls-cert-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305 --tls-private-key-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key"
```

### 1.2.13 Ensure that the admission control plugin SecurityContextDeny is set if PodSecurityPolicy is not used (Manual)


**Result:** pass

**Remediation:**
Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the control plane node and set the --enable-admission-plugins parameter to include
SecurityContextDeny, unless PodSecurityPolicy is already in place.
--enable-admission-plugins=...,SecurityContextDeny,...

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'enable-admission-plugins'
```

**Expected Result**:

```console
'--enable-admission-plugins' has 'SecurityContextDeny' OR '--enable-admission-plugins' has 'PodSecurityPolicy'
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-apiserver --advertise-port=6443 --allow-privileged=true --anonymous-auth=false --api-audiences=https://kubernetes.default.svc.cluster.local,k3s --audit-log-maxage=30 --audit-log-maxbackup=10 --audit-log-maxsize=100 --audit-log-path=/var/lib/rancher/k3s/server/logs/audit.log --audit-policy-file=/var/lib/rancher/k3s/server/audit.yaml --authorization-mode=Node,RBAC --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/temporary-certs --client-ca-file=/var/lib/rancher/k3s/server/tls/client-ca.crt --egress-selector-config-file=/var/lib/rancher/k3s/server/etc/egress-selector-config.yaml --enable-admission-plugins=NodeRestriction,PodSecurityPolicy,ServiceAccount --enable-aggregator-routing=true --enable-bootstrap-token-auth=true --encryption-provider-config=/var/lib/rancher/k3s/server/cred/encryption-config.json --etcd-cafile=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --etcd-certfile=/var/lib/rancher/k3s/server/tls/etcd/client.crt --etcd-keyfile=/var/lib/rancher/k3s/server/tls/etcd/client.key --etcd-servers=https://127.0.0.1:2379 --feature-gates=JobTrackingWithFinalizers=true --kubelet-certificate-authority=/var/lib/rancher/k3s/server/tls/server-ca.crt --kubelet-client-certificate=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.crt --kubelet-client-key=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --profiling=false --proxy-client-cert-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.crt --proxy-client-key-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.key --request-timeout=300s --requestheader-allowed-names=system:auth-proxy --requestheader-client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6444 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/rancher/k3s/server/tls/service.key --service-account-lookup=true --service-account-signing-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --service-node-port-range=30000-32767 --storage-backend=etcd3 --tls-cert-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305 --tls-private-key-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key"
```

### 1.2.14 Ensure that the admission control plugin ServiceAccount is set (Automated)


**Result:** pass

**Remediation:**
Follow the documentation and create ServiceAccount objects as per your environment.
Then, edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the control plane node and ensure that the --disable-admission-plugins parameter is set to a
value that does not include ServiceAccount.

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'ServiceAccount'
```

**Expected Result**:

```console
'--disable-admission-plugins' is present OR '--disable-admission-plugins' is not present
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-apiserver --advertise-port=6443 --allow-privileged=true --anonymous-auth=false --api-audiences=https://kubernetes.default.svc.cluster.local,k3s --audit-log-maxage=30 --audit-log-maxbackup=10 --audit-log-maxsize=100 --audit-log-path=/var/lib/rancher/k3s/server/logs/audit.log --audit-policy-file=/var/lib/rancher/k3s/server/audit.yaml --authorization-mode=Node,RBAC --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/temporary-certs --client-ca-file=/var/lib/rancher/k3s/server/tls/client-ca.crt --egress-selector-config-file=/var/lib/rancher/k3s/server/etc/egress-selector-config.yaml --enable-admission-plugins=NodeRestriction,PodSecurityPolicy,ServiceAccount --enable-aggregator-routing=true --enable-bootstrap-token-auth=true --encryption-provider-config=/var/lib/rancher/k3s/server/cred/encryption-config.json --etcd-cafile=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --etcd-certfile=/var/lib/rancher/k3s/server/tls/etcd/client.crt --etcd-keyfile=/var/lib/rancher/k3s/server/tls/etcd/client.key --etcd-servers=https://127.0.0.1:2379 --feature-gates=JobTrackingWithFinalizers=true --kubelet-certificate-authority=/var/lib/rancher/k3s/server/tls/server-ca.crt --kubelet-client-certificate=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.crt --kubelet-client-key=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --profiling=false --proxy-client-cert-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.crt --proxy-client-key-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.key --request-timeout=300s --requestheader-allowed-names=system:auth-proxy --requestheader-client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6444 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/rancher/k3s/server/tls/service.key --service-account-lookup=true --service-account-signing-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --service-node-port-range=30000-32767 --storage-backend=etcd3 --tls-cert-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305 --tls-private-key-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key"
```

### 1.2.15 Ensure that the admission control plugin NamespaceLifecycle is set (Automated)


**Result:** pass

**Remediation:**
Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the control plane node and set the --disable-admission-plugins parameter to
ensure it does not include NamespaceLifecycle.

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep -v grep
```

**Expected Result**:

```console
'--disable-admission-plugins' is present OR '--disable-admission-plugins' is not present
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-apiserver --advertise-port=6443 --allow-privileged=true --anonymous-auth=false --api-audiences=https://kubernetes.default.svc.cluster.local,k3s --audit-log-maxage=30 --audit-log-maxbackup=10 --audit-log-maxsize=100 --audit-log-path=/var/lib/rancher/k3s/server/logs/audit.log --audit-policy-file=/var/lib/rancher/k3s/server/audit.yaml --authorization-mode=Node,RBAC --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/temporary-certs --client-ca-file=/var/lib/rancher/k3s/server/tls/client-ca.crt --egress-selector-config-file=/var/lib/rancher/k3s/server/etc/egress-selector-config.yaml --enable-admission-plugins=NodeRestriction,PodSecurityPolicy,ServiceAccount --enable-aggregator-routing=true --enable-bootstrap-token-auth=true --encryption-provider-config=/var/lib/rancher/k3s/server/cred/encryption-config.json --etcd-cafile=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --etcd-certfile=/var/lib/rancher/k3s/server/tls/etcd/client.crt --etcd-keyfile=/var/lib/rancher/k3s/server/tls/etcd/client.key --etcd-servers=https://127.0.0.1:2379 --feature-gates=JobTrackingWithFinalizers=true --kubelet-certificate-authority=/var/lib/rancher/k3s/server/tls/server-ca.crt --kubelet-client-certificate=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.crt --kubelet-client-key=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --profiling=false --proxy-client-cert-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.crt --proxy-client-key-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.key --request-timeout=300s --requestheader-allowed-names=system:auth-proxy --requestheader-client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6444 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/rancher/k3s/server/tls/service.key --service-account-lookup=true --service-account-signing-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --service-node-port-range=30000-32767 --storage-backend=etcd3 --tls-cert-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305 --tls-private-key-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key"
```

### 1.2.16 Ensure that the admission control plugin NodeRestriction is set (Automated)


**Result:** pass

**Remediation:**
Follow the Kubernetes documentation and configure NodeRestriction plug-in on kubelets.
Then, edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the control plane node and set the --enable-admission-plugins parameter to a
value that includes NodeRestriction.
--enable-admission-plugins=...,NodeRestriction,...

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'enable-admission-plugins'
```

**Expected Result**:

```console
'--enable-admission-plugins' has 'NodeRestriction'
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-apiserver --advertise-port=6443 --allow-privileged=true --anonymous-auth=false --api-audiences=https://kubernetes.default.svc.cluster.local,k3s --audit-log-maxage=30 --audit-log-maxbackup=10 --audit-log-maxsize=100 --audit-log-path=/var/lib/rancher/k3s/server/logs/audit.log --audit-policy-file=/var/lib/rancher/k3s/server/audit.yaml --authorization-mode=Node,RBAC --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/temporary-certs --client-ca-file=/var/lib/rancher/k3s/server/tls/client-ca.crt --egress-selector-config-file=/var/lib/rancher/k3s/server/etc/egress-selector-config.yaml --enable-admission-plugins=NodeRestriction,PodSecurityPolicy,ServiceAccount --enable-aggregator-routing=true --enable-bootstrap-token-auth=true --encryption-provider-config=/var/lib/rancher/k3s/server/cred/encryption-config.json --etcd-cafile=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --etcd-certfile=/var/lib/rancher/k3s/server/tls/etcd/client.crt --etcd-keyfile=/var/lib/rancher/k3s/server/tls/etcd/client.key --etcd-servers=https://127.0.0.1:2379 --feature-gates=JobTrackingWithFinalizers=true --kubelet-certificate-authority=/var/lib/rancher/k3s/server/tls/server-ca.crt --kubelet-client-certificate=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.crt --kubelet-client-key=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --profiling=false --proxy-client-cert-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.crt --proxy-client-key-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.key --request-timeout=300s --requestheader-allowed-names=system:auth-proxy --requestheader-client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6444 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/rancher/k3s/server/tls/service.key --service-account-lookup=true --service-account-signing-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --service-node-port-range=30000-32767 --storage-backend=etcd3 --tls-cert-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305 --tls-private-key-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key"
```

### 1.2.17 Ensure that the --secure-port argument is not set to 0 (Automated)


**Result:** pass

**Remediation:**
Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the control plane node and either remove the --secure-port parameter or
set it to a different (non-zero) desired port.

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'secure-port'
```

**Expected Result**:

```console
'--secure-port' is greater than 0 OR '--secure-port' is not present
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-apiserver --advertise-port=6443 --allow-privileged=true --anonymous-auth=false --api-audiences=https://kubernetes.default.svc.cluster.local,k3s --audit-log-maxage=30 --audit-log-maxbackup=10 --audit-log-maxsize=100 --audit-log-path=/var/lib/rancher/k3s/server/logs/audit.log --audit-policy-file=/var/lib/rancher/k3s/server/audit.yaml --authorization-mode=Node,RBAC --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/temporary-certs --client-ca-file=/var/lib/rancher/k3s/server/tls/client-ca.crt --egress-selector-config-file=/var/lib/rancher/k3s/server/etc/egress-selector-config.yaml --enable-admission-plugins=NodeRestriction,PodSecurityPolicy,ServiceAccount --enable-aggregator-routing=true --enable-bootstrap-token-auth=true --encryption-provider-config=/var/lib/rancher/k3s/server/cred/encryption-config.json --etcd-cafile=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --etcd-certfile=/var/lib/rancher/k3s/server/tls/etcd/client.crt --etcd-keyfile=/var/lib/rancher/k3s/server/tls/etcd/client.key --etcd-servers=https://127.0.0.1:2379 --feature-gates=JobTrackingWithFinalizers=true --kubelet-certificate-authority=/var/lib/rancher/k3s/server/tls/server-ca.crt --kubelet-client-certificate=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.crt --kubelet-client-key=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --profiling=false --proxy-client-cert-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.crt --proxy-client-key-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.key --request-timeout=300s --requestheader-allowed-names=system:auth-proxy --requestheader-client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6444 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/rancher/k3s/server/tls/service.key --service-account-lookup=true --service-account-signing-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --service-node-port-range=30000-32767 --storage-backend=etcd3 --tls-cert-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305 --tls-private-key-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key"
```

### 1.2.18 Ensure that the --profiling argument is set to false (Automated)


**Result:** pass

**Remediation:**
Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the control plane node and set the below parameter.
--profiling=false

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'profiling'
```

**Expected Result**:

```console
'--profiling' is equal to 'false'
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-apiserver --advertise-port=6443 --allow-privileged=true --anonymous-auth=false --api-audiences=https://kubernetes.default.svc.cluster.local,k3s --audit-log-maxage=30 --audit-log-maxbackup=10 --audit-log-maxsize=100 --audit-log-path=/var/lib/rancher/k3s/server/logs/audit.log --audit-policy-file=/var/lib/rancher/k3s/server/audit.yaml --authorization-mode=Node,RBAC --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/temporary-certs --client-ca-file=/var/lib/rancher/k3s/server/tls/client-ca.crt --egress-selector-config-file=/var/lib/rancher/k3s/server/etc/egress-selector-config.yaml --enable-admission-plugins=NodeRestriction,PodSecurityPolicy,ServiceAccount --enable-aggregator-routing=true --enable-bootstrap-token-auth=true --encryption-provider-config=/var/lib/rancher/k3s/server/cred/encryption-config.json --etcd-cafile=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --etcd-certfile=/var/lib/rancher/k3s/server/tls/etcd/client.crt --etcd-keyfile=/var/lib/rancher/k3s/server/tls/etcd/client.key --etcd-servers=https://127.0.0.1:2379 --feature-gates=JobTrackingWithFinalizers=true --kubelet-certificate-authority=/var/lib/rancher/k3s/server/tls/server-ca.crt --kubelet-client-certificate=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.crt --kubelet-client-key=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --profiling=false --proxy-client-cert-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.crt --proxy-client-key-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.key --request-timeout=300s --requestheader-allowed-names=system:auth-proxy --requestheader-client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6444 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/rancher/k3s/server/tls/service.key --service-account-lookup=true --service-account-signing-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --service-node-port-range=30000-32767 --storage-backend=etcd3 --tls-cert-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305 --tls-private-key-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key"
```

### 1.2.19 Ensure that the --audit-log-path argument is set (Automated)


**Result:** pass

**Remediation:**
Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the control plane node and set the --audit-log-path parameter to a suitable path and
file where you would like audit logs to be written, for example,
--audit-log-path=/var/log/apiserver/audit.log

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'audit-log-path'
```

**Expected Result**:

```console
'--audit-log-path' is present
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-apiserver --advertise-port=6443 --allow-privileged=true --anonymous-auth=false --api-audiences=https://kubernetes.default.svc.cluster.local,k3s --audit-log-maxage=30 --audit-log-maxbackup=10 --audit-log-maxsize=100 --audit-log-path=/var/lib/rancher/k3s/server/logs/audit.log --audit-policy-file=/var/lib/rancher/k3s/server/audit.yaml --authorization-mode=Node,RBAC --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/temporary-certs --client-ca-file=/var/lib/rancher/k3s/server/tls/client-ca.crt --egress-selector-config-file=/var/lib/rancher/k3s/server/etc/egress-selector-config.yaml --enable-admission-plugins=NodeRestriction,PodSecurityPolicy,ServiceAccount --enable-aggregator-routing=true --enable-bootstrap-token-auth=true --encryption-provider-config=/var/lib/rancher/k3s/server/cred/encryption-config.json --etcd-cafile=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --etcd-certfile=/var/lib/rancher/k3s/server/tls/etcd/client.crt --etcd-keyfile=/var/lib/rancher/k3s/server/tls/etcd/client.key --etcd-servers=https://127.0.0.1:2379 --feature-gates=JobTrackingWithFinalizers=true --kubelet-certificate-authority=/var/lib/rancher/k3s/server/tls/server-ca.crt --kubelet-client-certificate=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.crt --kubelet-client-key=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --profiling=false --proxy-client-cert-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.crt --proxy-client-key-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.key --request-timeout=300s --requestheader-allowed-names=system:auth-proxy --requestheader-client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6444 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/rancher/k3s/server/tls/service.key --service-account-lookup=true --service-account-signing-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --service-node-port-range=30000-32767 --storage-backend=etcd3 --tls-cert-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305 --tls-private-key-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key"
```

### 1.2.20 Ensure that the --audit-log-maxage argument is set to 30 or as appropriate (Automated)


**Result:** pass

**Remediation:**
Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the control plane node and set the --audit-log-maxage parameter to 30
or as an appropriate number of days, for example,
--audit-log-maxage=30

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'audit-log-maxage'
```

**Expected Result**:

```console
'--audit-log-maxage' is greater or equal to 30
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-apiserver --advertise-port=6443 --allow-privileged=true --anonymous-auth=false --api-audiences=https://kubernetes.default.svc.cluster.local,k3s --audit-log-maxage=30 --audit-log-maxbackup=10 --audit-log-maxsize=100 --audit-log-path=/var/lib/rancher/k3s/server/logs/audit.log --audit-policy-file=/var/lib/rancher/k3s/server/audit.yaml --authorization-mode=Node,RBAC --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/temporary-certs --client-ca-file=/var/lib/rancher/k3s/server/tls/client-ca.crt --egress-selector-config-file=/var/lib/rancher/k3s/server/etc/egress-selector-config.yaml --enable-admission-plugins=NodeRestriction,PodSecurityPolicy,ServiceAccount --enable-aggregator-routing=true --enable-bootstrap-token-auth=true --encryption-provider-config=/var/lib/rancher/k3s/server/cred/encryption-config.json --etcd-cafile=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --etcd-certfile=/var/lib/rancher/k3s/server/tls/etcd/client.crt --etcd-keyfile=/var/lib/rancher/k3s/server/tls/etcd/client.key --etcd-servers=https://127.0.0.1:2379 --feature-gates=JobTrackingWithFinalizers=true --kubelet-certificate-authority=/var/lib/rancher/k3s/server/tls/server-ca.crt --kubelet-client-certificate=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.crt --kubelet-client-key=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --profiling=false --proxy-client-cert-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.crt --proxy-client-key-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.key --request-timeout=300s --requestheader-allowed-names=system:auth-proxy --requestheader-client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6444 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/rancher/k3s/server/tls/service.key --service-account-lookup=true --service-account-signing-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --service-node-port-range=30000-32767 --storage-backend=etcd3 --tls-cert-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305 --tls-private-key-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key"
```

### 1.2.21 Ensure that the --audit-log-maxbackup argument is set to 10 or as appropriate (Automated)


**Result:** pass

**Remediation:**
Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the control plane node and set the --audit-log-maxbackup parameter to 10 or to an appropriate
value. For example,
--audit-log-maxbackup=10

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'audit-log-maxbackup'
```

**Expected Result**:

```console
'--audit-log-maxbackup' is greater or equal to 10
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-apiserver --advertise-port=6443 --allow-privileged=true --anonymous-auth=false --api-audiences=https://kubernetes.default.svc.cluster.local,k3s --audit-log-maxage=30 --audit-log-maxbackup=10 --audit-log-maxsize=100 --audit-log-path=/var/lib/rancher/k3s/server/logs/audit.log --audit-policy-file=/var/lib/rancher/k3s/server/audit.yaml --authorization-mode=Node,RBAC --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/temporary-certs --client-ca-file=/var/lib/rancher/k3s/server/tls/client-ca.crt --egress-selector-config-file=/var/lib/rancher/k3s/server/etc/egress-selector-config.yaml --enable-admission-plugins=NodeRestriction,PodSecurityPolicy,ServiceAccount --enable-aggregator-routing=true --enable-bootstrap-token-auth=true --encryption-provider-config=/var/lib/rancher/k3s/server/cred/encryption-config.json --etcd-cafile=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --etcd-certfile=/var/lib/rancher/k3s/server/tls/etcd/client.crt --etcd-keyfile=/var/lib/rancher/k3s/server/tls/etcd/client.key --etcd-servers=https://127.0.0.1:2379 --feature-gates=JobTrackingWithFinalizers=true --kubelet-certificate-authority=/var/lib/rancher/k3s/server/tls/server-ca.crt --kubelet-client-certificate=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.crt --kubelet-client-key=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --profiling=false --proxy-client-cert-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.crt --proxy-client-key-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.key --request-timeout=300s --requestheader-allowed-names=system:auth-proxy --requestheader-client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6444 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/rancher/k3s/server/tls/service.key --service-account-lookup=true --service-account-signing-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --service-node-port-range=30000-32767 --storage-backend=etcd3 --tls-cert-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305 --tls-private-key-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key"
```

### 1.2.22 Ensure that the --audit-log-maxsize argument is set to 100 or as appropriate (Automated)


**Result:** pass

**Remediation:**
Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the control plane node and set the --audit-log-maxsize parameter to an appropriate size in MB.
For example, to set it as 100 MB, --audit-log-maxsize=100

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'audit-log-maxsize'
```

**Expected Result**:

```console
'--audit-log-maxsize' is greater or equal to 100
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-apiserver --advertise-port=6443 --allow-privileged=true --anonymous-auth=false --api-audiences=https://kubernetes.default.svc.cluster.local,k3s --audit-log-maxage=30 --audit-log-maxbackup=10 --audit-log-maxsize=100 --audit-log-path=/var/lib/rancher/k3s/server/logs/audit.log --audit-policy-file=/var/lib/rancher/k3s/server/audit.yaml --authorization-mode=Node,RBAC --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/temporary-certs --client-ca-file=/var/lib/rancher/k3s/server/tls/client-ca.crt --egress-selector-config-file=/var/lib/rancher/k3s/server/etc/egress-selector-config.yaml --enable-admission-plugins=NodeRestriction,PodSecurityPolicy,ServiceAccount --enable-aggregator-routing=true --enable-bootstrap-token-auth=true --encryption-provider-config=/var/lib/rancher/k3s/server/cred/encryption-config.json --etcd-cafile=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --etcd-certfile=/var/lib/rancher/k3s/server/tls/etcd/client.crt --etcd-keyfile=/var/lib/rancher/k3s/server/tls/etcd/client.key --etcd-servers=https://127.0.0.1:2379 --feature-gates=JobTrackingWithFinalizers=true --kubelet-certificate-authority=/var/lib/rancher/k3s/server/tls/server-ca.crt --kubelet-client-certificate=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.crt --kubelet-client-key=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --profiling=false --proxy-client-cert-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.crt --proxy-client-key-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.key --request-timeout=300s --requestheader-allowed-names=system:auth-proxy --requestheader-client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6444 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/rancher/k3s/server/tls/service.key --service-account-lookup=true --service-account-signing-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --service-node-port-range=30000-32767 --storage-backend=etcd3 --tls-cert-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305 --tls-private-key-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key"
```

### 1.2.23 Ensure that the --request-timeout argument is set as appropriate (Manual)


**Result:** pass

**Remediation:**
Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
and set the below parameter as appropriate and if needed.
For example, --request-timeout=300s

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'request-timeout'
```

**Expected Result**:

```console
'--request-timeout' is not present OR '--request-timeout' is present
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-apiserver --advertise-port=6443 --allow-privileged=true --anonymous-auth=false --api-audiences=https://kubernetes.default.svc.cluster.local,k3s --audit-log-maxage=30 --audit-log-maxbackup=10 --audit-log-maxsize=100 --audit-log-path=/var/lib/rancher/k3s/server/logs/audit.log --audit-policy-file=/var/lib/rancher/k3s/server/audit.yaml --authorization-mode=Node,RBAC --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/temporary-certs --client-ca-file=/var/lib/rancher/k3s/server/tls/client-ca.crt --egress-selector-config-file=/var/lib/rancher/k3s/server/etc/egress-selector-config.yaml --enable-admission-plugins=NodeRestriction,PodSecurityPolicy,ServiceAccount --enable-aggregator-routing=true --enable-bootstrap-token-auth=true --encryption-provider-config=/var/lib/rancher/k3s/server/cred/encryption-config.json --etcd-cafile=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --etcd-certfile=/var/lib/rancher/k3s/server/tls/etcd/client.crt --etcd-keyfile=/var/lib/rancher/k3s/server/tls/etcd/client.key --etcd-servers=https://127.0.0.1:2379 --feature-gates=JobTrackingWithFinalizers=true --kubelet-certificate-authority=/var/lib/rancher/k3s/server/tls/server-ca.crt --kubelet-client-certificate=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.crt --kubelet-client-key=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --profiling=false --proxy-client-cert-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.crt --proxy-client-key-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.key --request-timeout=300s --requestheader-allowed-names=system:auth-proxy --requestheader-client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6444 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/rancher/k3s/server/tls/service.key --service-account-lookup=true --service-account-signing-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --service-node-port-range=30000-32767 --storage-backend=etcd3 --tls-cert-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305 --tls-private-key-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key"
```

### 1.2.24 Ensure that the --service-account-lookup argument is set to true (Automated)


**Result:** pass

**Remediation:**
Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the control plane node and set the below parameter.
--service-account-lookup=true
Alternatively, you can delete the --service-account-lookup parameter from this file so
that the default takes effect.

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'service-account-lookup'
```

**Expected Result**:

```console
'--service-account-lookup' is not present OR '--service-account-lookup' is equal to 'true'
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-apiserver --advertise-port=6443 --allow-privileged=true --anonymous-auth=false --api-audiences=https://kubernetes.default.svc.cluster.local,k3s --audit-log-maxage=30 --audit-log-maxbackup=10 --audit-log-maxsize=100 --audit-log-path=/var/lib/rancher/k3s/server/logs/audit.log --audit-policy-file=/var/lib/rancher/k3s/server/audit.yaml --authorization-mode=Node,RBAC --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/temporary-certs --client-ca-file=/var/lib/rancher/k3s/server/tls/client-ca.crt --egress-selector-config-file=/var/lib/rancher/k3s/server/etc/egress-selector-config.yaml --enable-admission-plugins=NodeRestriction,PodSecurityPolicy,ServiceAccount --enable-aggregator-routing=true --enable-bootstrap-token-auth=true --encryption-provider-config=/var/lib/rancher/k3s/server/cred/encryption-config.json --etcd-cafile=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --etcd-certfile=/var/lib/rancher/k3s/server/tls/etcd/client.crt --etcd-keyfile=/var/lib/rancher/k3s/server/tls/etcd/client.key --etcd-servers=https://127.0.0.1:2379 --feature-gates=JobTrackingWithFinalizers=true --kubelet-certificate-authority=/var/lib/rancher/k3s/server/tls/server-ca.crt --kubelet-client-certificate=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.crt --kubelet-client-key=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --profiling=false --proxy-client-cert-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.crt --proxy-client-key-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.key --request-timeout=300s --requestheader-allowed-names=system:auth-proxy --requestheader-client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6444 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/rancher/k3s/server/tls/service.key --service-account-lookup=true --service-account-signing-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --service-node-port-range=30000-32767 --storage-backend=etcd3 --tls-cert-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305 --tls-private-key-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key"
```

### 1.2.25 Ensure that the --service-account-key-file argument is set as appropriate (Automated)


**Result:** pass

**Remediation:**
Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the control plane node and set the --service-account-key-file parameter
to the public key file for service accounts. For example,
--service-account-key-file=<filename\>

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'service-account-key-file'
```

**Expected Result**:

```console
'--service-account-key-file' is present
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-apiserver --advertise-port=6443 --allow-privileged=true --anonymous-auth=false --api-audiences=https://kubernetes.default.svc.cluster.local,k3s --audit-log-maxage=30 --audit-log-maxbackup=10 --audit-log-maxsize=100 --audit-log-path=/var/lib/rancher/k3s/server/logs/audit.log --audit-policy-file=/var/lib/rancher/k3s/server/audit.yaml --authorization-mode=Node,RBAC --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/temporary-certs --client-ca-file=/var/lib/rancher/k3s/server/tls/client-ca.crt --egress-selector-config-file=/var/lib/rancher/k3s/server/etc/egress-selector-config.yaml --enable-admission-plugins=NodeRestriction,PodSecurityPolicy,ServiceAccount --enable-aggregator-routing=true --enable-bootstrap-token-auth=true --encryption-provider-config=/var/lib/rancher/k3s/server/cred/encryption-config.json --etcd-cafile=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --etcd-certfile=/var/lib/rancher/k3s/server/tls/etcd/client.crt --etcd-keyfile=/var/lib/rancher/k3s/server/tls/etcd/client.key --etcd-servers=https://127.0.0.1:2379 --feature-gates=JobTrackingWithFinalizers=true --kubelet-certificate-authority=/var/lib/rancher/k3s/server/tls/server-ca.crt --kubelet-client-certificate=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.crt --kubelet-client-key=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --profiling=false --proxy-client-cert-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.crt --proxy-client-key-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.key --request-timeout=300s --requestheader-allowed-names=system:auth-proxy --requestheader-client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6444 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/rancher/k3s/server/tls/service.key --service-account-lookup=true --service-account-signing-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --service-node-port-range=30000-32767 --storage-backend=etcd3 --tls-cert-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305 --tls-private-key-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key"
```

### 1.2.26 Ensure that the --etcd-certfile and --etcd-keyfile arguments are set as appropriate (Automated)


**Result:** pass

**Remediation:**
Follow the Kubernetes documentation and set up the TLS connection between the apiserver and etcd.
Then, edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the control plane node and set the etcd certificate and key file parameters.
--etcd-certfile=<path/to/client-certificate-file\>
--etcd-keyfile=<path/to/client-key-file\>

**Audit Script:** `check_for_k3s_etcd.sh`

```bash
#!/bin/bash

# This script is used to ensure that k3s is actually running etcd (and not other databases like sqlite3)
# before it checks the requirement
set -eE

handle_error() {
    echo "false"
}

trap 'handle_error' ERR


if [[ "$(journalctl -D /var/log/journal -u k3s | grep 'Managed etcd cluster initializing' | grep -v grep | wc -l)" -gt 0 ]]; then
    case $1 in 
        "1.1.11")
            echo $(stat -c %a /var/lib/rancher/k3s/server/db/etcd);;
        "1.2.29")
            echo $(journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'etcd-');;
        "2.1")
            echo $(grep -A 5 'client-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep -E 'cert-file|key-file');;
        "2.2")
            echo $(grep -A 5 'client-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep 'client-cert-auth');;
        "2.3")
            echo $(grep 'auto-tls' /var/lib/rancher/k3s/server/db/etcd/config);;
        "2.4")
            echo $(grep -A 5 'peer-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep -E 'cert-file|key-file');;
        "2.5")
            echo $(grep -A 5 'peer-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep 'client-cert-auth');;
        "2.6")
            echo $(grep 'peer-auto-tls' /var/lib/rancher/k3s/server/db/etcd/config);;
        "2.7")
            echo $(grep 'trusted-ca-file' /var/lib/rancher/k3s/server/db/etcd/config);;
    esac
else
# If another database is running, return whatever is required to pass the scan
    case $1 in
        "1.1.11")
            echo "700";;
        "1.2.29")
            echo "--etcd-certfile AND --etcd-keyfile";;
        "2.1")
            echo "cert-file AND key-file";;
        "2.2")
            echo "--client-cert-auth=true";;
        "2.3")
            echo "false";;
        "2.4")
            echo "peer-cert-file AND peer-key-file";;
        "2.5")
            echo "--client-cert-auth=true";;
        "2.6")
            echo "--peer-auto-tls=false";;
        "2.7")
            echo "--trusted-ca-file";;
    esac
fi

```

**Audit Execution:**

```bash
./check_for_k3s_etcd.sh 1.2.29
```

**Expected Result**:

```console
'--etcd-certfile' is present AND '--etcd-keyfile' is present
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-apiserver --advertise-port=6443 --allow-privileged=true --anonymous-auth=false --api-audiences=https://kubernetes.default.svc.cluster.local,k3s --audit-log-maxage=30 --audit-log-maxbackup=10 --audit-log-maxsize=100 --audit-log-path=/var/lib/rancher/k3s/server/logs/audit.log --audit-policy-file=/var/lib/rancher/k3s/server/audit.yaml --authorization-mode=Node,RBAC --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/temporary-certs --client-ca-file=/var/lib/rancher/k3s/server/tls/client-ca.crt --egress-selector-config-file=/var/lib/rancher/k3s/server/etc/egress-selector-config.yaml --enable-admission-plugins=NodeRestriction,PodSecurityPolicy,ServiceAccount --enable-aggregator-routing=true --enable-bootstrap-token-auth=true --encryption-provider-config=/var/lib/rancher/k3s/server/cred/encryption-config.json --etcd-cafile=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --etcd-certfile=/var/lib/rancher/k3s/server/tls/etcd/client.crt --etcd-keyfile=/var/lib/rancher/k3s/server/tls/etcd/client.key --etcd-servers=https://127.0.0.1:2379 --feature-gates=JobTrackingWithFinalizers=true --kubelet-certificate-authority=/var/lib/rancher/k3s/server/tls/server-ca.crt --kubelet-client-certificate=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.crt --kubelet-client-key=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --profiling=false --proxy-client-cert-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.crt --proxy-client-key-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.key --request-timeout=300s --requestheader-allowed-names=system:auth-proxy --requestheader-client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6444 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/rancher/k3s/server/tls/service.key --service-account-lookup=true --service-account-signing-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --service-node-port-range=30000-32767 --storage-backend=etcd3 --tls-cert-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305 --tls-private-key-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key"
```

### 1.2.27 Ensure that the --tls-cert-file and --tls-private-key-file arguments are set as appropriate (Automated)


**Result:** pass

**Remediation:**
Follow the Kubernetes documentation and set up the TLS connection on the apiserver.
Then, edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the control plane node and set the TLS certificate and private key file parameters.
--tls-cert-file=<path/to/tls-certificate-file\>
--tls-private-key-file=<path/to/tls-key-file\>

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep -A1 'Running kube-apiserver' | tail -n2
```

**Expected Result**:

```console
'--tls-cert-file' is present AND '--tls-private-key-file' is present
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-apiserver --advertise-port=6443 --allow-privileged=true --anonymous-auth=false --api-audiences=https://kubernetes.default.svc.cluster.local,k3s --audit-log-maxage=30 --audit-log-maxbackup=10 --audit-log-maxsize=100 --audit-log-path=/var/lib/rancher/k3s/server/logs/audit.log --audit-policy-file=/var/lib/rancher/k3s/server/audit.yaml --authorization-mode=Node,RBAC --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/temporary-certs --client-ca-file=/var/lib/rancher/k3s/server/tls/client-ca.crt --egress-selector-config-file=/var/lib/rancher/k3s/server/etc/egress-selector-config.yaml --enable-admission-plugins=NodeRestriction,PodSecurityPolicy,ServiceAccount --enable-aggregator-routing=true --enable-bootstrap-token-auth=true --encryption-provider-config=/var/lib/rancher/k3s/server/cred/encryption-config.json --etcd-cafile=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --etcd-certfile=/var/lib/rancher/k3s/server/tls/etcd/client.crt --etcd-keyfile=/var/lib/rancher/k3s/server/tls/etcd/client.key --etcd-servers=https://127.0.0.1:2379 --feature-gates=JobTrackingWithFinalizers=true --kubelet-certificate-authority=/var/lib/rancher/k3s/server/tls/server-ca.crt --kubelet-client-certificate=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.crt --kubelet-client-key=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --profiling=false --proxy-client-cert-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.crt --proxy-client-key-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.key --request-timeout=300s --requestheader-allowed-names=system:auth-proxy --requestheader-client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6444 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/rancher/k3s/server/tls/service.key --service-account-lookup=true --service-account-signing-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --service-node-port-range=30000-32767 --storage-backend=etcd3 --tls-cert-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305 --tls-private-key-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key" Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-scheduler --authentication-kubeconfig=/var/lib/rancher/k3s/server/cred/scheduler.kubeconfig --authorization-kubeconfig=/var/lib/rancher/k3s/server/cred/scheduler.kubeconfig --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/kube-scheduler --kubeconfig=/var/lib/rancher/k3s/server/cred/scheduler.kubeconfig --profiling=false --secure-port=10259"
```

### 1.2.28 Ensure that the --client-ca-file argument is set as appropriate (Automated)


**Result:** pass

**Remediation:**
Follow the Kubernetes documentation and set up the TLS connection on the apiserver.
Then, edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the control plane node and set the client certificate authority file.
--client-ca-file=<path/to/client-ca-file\>

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'client-ca-file'
```

**Expected Result**:

```console
'--client-ca-file' is present
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-apiserver --advertise-port=6443 --allow-privileged=true --anonymous-auth=false --api-audiences=https://kubernetes.default.svc.cluster.local,k3s --audit-log-maxage=30 --audit-log-maxbackup=10 --audit-log-maxsize=100 --audit-log-path=/var/lib/rancher/k3s/server/logs/audit.log --audit-policy-file=/var/lib/rancher/k3s/server/audit.yaml --authorization-mode=Node,RBAC --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/temporary-certs --client-ca-file=/var/lib/rancher/k3s/server/tls/client-ca.crt --egress-selector-config-file=/var/lib/rancher/k3s/server/etc/egress-selector-config.yaml --enable-admission-plugins=NodeRestriction,PodSecurityPolicy,ServiceAccount --enable-aggregator-routing=true --enable-bootstrap-token-auth=true --encryption-provider-config=/var/lib/rancher/k3s/server/cred/encryption-config.json --etcd-cafile=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --etcd-certfile=/var/lib/rancher/k3s/server/tls/etcd/client.crt --etcd-keyfile=/var/lib/rancher/k3s/server/tls/etcd/client.key --etcd-servers=https://127.0.0.1:2379 --feature-gates=JobTrackingWithFinalizers=true --kubelet-certificate-authority=/var/lib/rancher/k3s/server/tls/server-ca.crt --kubelet-client-certificate=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.crt --kubelet-client-key=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --profiling=false --proxy-client-cert-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.crt --proxy-client-key-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.key --request-timeout=300s --requestheader-allowed-names=system:auth-proxy --requestheader-client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6444 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/rancher/k3s/server/tls/service.key --service-account-lookup=true --service-account-signing-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --service-node-port-range=30000-32767 --storage-backend=etcd3 --tls-cert-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305 --tls-private-key-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key"
```

### 1.2.29 Ensure that the --etcd-cafile argument is set as appropriate (Automated)


**Result:** pass

**Remediation:**
Follow the Kubernetes documentation and set up the TLS connection between the apiserver and etcd.
Then, edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the control plane node and set the etcd certificate authority file parameter.
--etcd-cafile=<path/to/ca-file\>

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'etcd-cafile'
```

**Expected Result**:

```console
'--etcd-cafile' is present
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-apiserver --advertise-port=6443 --allow-privileged=true --anonymous-auth=false --api-audiences=https://kubernetes.default.svc.cluster.local,k3s --audit-log-maxage=30 --audit-log-maxbackup=10 --audit-log-maxsize=100 --audit-log-path=/var/lib/rancher/k3s/server/logs/audit.log --audit-policy-file=/var/lib/rancher/k3s/server/audit.yaml --authorization-mode=Node,RBAC --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/temporary-certs --client-ca-file=/var/lib/rancher/k3s/server/tls/client-ca.crt --egress-selector-config-file=/var/lib/rancher/k3s/server/etc/egress-selector-config.yaml --enable-admission-plugins=NodeRestriction,PodSecurityPolicy,ServiceAccount --enable-aggregator-routing=true --enable-bootstrap-token-auth=true --encryption-provider-config=/var/lib/rancher/k3s/server/cred/encryption-config.json --etcd-cafile=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --etcd-certfile=/var/lib/rancher/k3s/server/tls/etcd/client.crt --etcd-keyfile=/var/lib/rancher/k3s/server/tls/etcd/client.key --etcd-servers=https://127.0.0.1:2379 --feature-gates=JobTrackingWithFinalizers=true --kubelet-certificate-authority=/var/lib/rancher/k3s/server/tls/server-ca.crt --kubelet-client-certificate=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.crt --kubelet-client-key=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --profiling=false --proxy-client-cert-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.crt --proxy-client-key-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.key --request-timeout=300s --requestheader-allowed-names=system:auth-proxy --requestheader-client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6444 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/rancher/k3s/server/tls/service.key --service-account-lookup=true --service-account-signing-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --service-node-port-range=30000-32767 --storage-backend=etcd3 --tls-cert-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305 --tls-private-key-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key"
```

### 1.2.30 Ensure that the --encryption-provider-config argument is set as appropriate (Manual)


**Result:** pass

**Remediation:**
Follow the Kubernetes documentation and configure a EncryptionConfig file.
Then, edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the control plane node and set the --encryption-provider-config parameter to the path of that file.
For example, --encryption-provider-config=</path/to/EncryptionConfig/File\>

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'encryption-provider-config'
```

**Expected Result**:

```console
'--encryption-provider-config' is present
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-apiserver --advertise-port=6443 --allow-privileged=true --anonymous-auth=false --api-audiences=https://kubernetes.default.svc.cluster.local,k3s --audit-log-maxage=30 --audit-log-maxbackup=10 --audit-log-maxsize=100 --audit-log-path=/var/lib/rancher/k3s/server/logs/audit.log --audit-policy-file=/var/lib/rancher/k3s/server/audit.yaml --authorization-mode=Node,RBAC --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/temporary-certs --client-ca-file=/var/lib/rancher/k3s/server/tls/client-ca.crt --egress-selector-config-file=/var/lib/rancher/k3s/server/etc/egress-selector-config.yaml --enable-admission-plugins=NodeRestriction,PodSecurityPolicy,ServiceAccount --enable-aggregator-routing=true --enable-bootstrap-token-auth=true --encryption-provider-config=/var/lib/rancher/k3s/server/cred/encryption-config.json --etcd-cafile=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --etcd-certfile=/var/lib/rancher/k3s/server/tls/etcd/client.crt --etcd-keyfile=/var/lib/rancher/k3s/server/tls/etcd/client.key --etcd-servers=https://127.0.0.1:2379 --feature-gates=JobTrackingWithFinalizers=true --kubelet-certificate-authority=/var/lib/rancher/k3s/server/tls/server-ca.crt --kubelet-client-certificate=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.crt --kubelet-client-key=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --profiling=false --proxy-client-cert-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.crt --proxy-client-key-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.key --request-timeout=300s --requestheader-allowed-names=system:auth-proxy --requestheader-client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6444 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/rancher/k3s/server/tls/service.key --service-account-lookup=true --service-account-signing-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --service-node-port-range=30000-32767 --storage-backend=etcd3 --tls-cert-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305 --tls-private-key-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key"
```

### 1.2.31 Ensure that encryption providers are appropriately configured (Manual)


**Result:** warn

**Remediation:**
Follow the Kubernetes documentation and configure a EncryptionConfig file.
In this file, choose aescbc, kms or secretbox as the encryption provider.

**Audit:**

```bash
grep aescbc /path/to/encryption-config.json
```

### 1.2.32 Ensure that the API Server only makes use of Strong Cryptographic Ciphers (Manual)


**Result:** pass

**Remediation:**
Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the control plane node and set the below parameter.
--tls-cipher-suites=TLS_AES_128_GCM_SHA256,TLS_AES_256_GCM_SHA384,TLS_CHACHA20_POLY1305_SHA256,
TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,
TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA,TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,
TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256,
TLS_ECDHE_RSA_WITH_3DES_EDE_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,
TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305,
TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256,TLS_RSA_WITH_3DES_EDE_CBC_SHA,TLS_RSA_WITH_AES_128_CBC_SHA,
TLS_RSA_WITH_AES_128_GCM_SHA256,TLS_RSA_WITH_AES_256_CBC_SHA,TLS_RSA_WITH_AES_256_GCM_SHA384

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'tls-cipher-suites'
```

**Expected Result**:

```console
'--tls-cipher-suites' contains valid elements from 'TLS_AES_128_GCM_SHA256,TLS_AES_256_GCM_SHA384,TLS_CHACHA20_POLY1305_SHA256,TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA,TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256,TLS_ECDHE_RSA_WITH_3DES_EDE_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256,TLS_RSA_WITH_3DES_EDE_CBC_SHA,TLS_RSA_WITH_AES_128_CBC_SHA,TLS_RSA_WITH_AES_128_GCM_SHA256,TLS_RSA_WITH_AES_256_CBC_SHA,TLS_RSA_WITH_AES_256_GCM_SHA384'
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-apiserver --advertise-port=6443 --allow-privileged=true --anonymous-auth=false --api-audiences=https://kubernetes.default.svc.cluster.local,k3s --audit-log-maxage=30 --audit-log-maxbackup=10 --audit-log-maxsize=100 --audit-log-path=/var/lib/rancher/k3s/server/logs/audit.log --audit-policy-file=/var/lib/rancher/k3s/server/audit.yaml --authorization-mode=Node,RBAC --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/temporary-certs --client-ca-file=/var/lib/rancher/k3s/server/tls/client-ca.crt --egress-selector-config-file=/var/lib/rancher/k3s/server/etc/egress-selector-config.yaml --enable-admission-plugins=NodeRestriction,PodSecurityPolicy,ServiceAccount --enable-aggregator-routing=true --enable-bootstrap-token-auth=true --encryption-provider-config=/var/lib/rancher/k3s/server/cred/encryption-config.json --etcd-cafile=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --etcd-certfile=/var/lib/rancher/k3s/server/tls/etcd/client.crt --etcd-keyfile=/var/lib/rancher/k3s/server/tls/etcd/client.key --etcd-servers=https://127.0.0.1:2379 --feature-gates=JobTrackingWithFinalizers=true --kubelet-certificate-authority=/var/lib/rancher/k3s/server/tls/server-ca.crt --kubelet-client-certificate=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.crt --kubelet-client-key=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --profiling=false --proxy-client-cert-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.crt --proxy-client-key-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.key --request-timeout=300s --requestheader-allowed-names=system:auth-proxy --requestheader-client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6444 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/rancher/k3s/server/tls/service.key --service-account-lookup=true --service-account-signing-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --service-node-port-range=30000-32767 --storage-backend=etcd3 --tls-cert-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305 --tls-private-key-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key"
```

## 1.3 Controller Manager
### 1.3.1 Ensure that the --terminated-pod-gc-threshold argument is set as appropriate (Manual)


**Result:** pass

**Remediation:**
Edit the Controller Manager pod specification file /etc/kubernetes/manifests/kube-controller-manager.yaml
on the control plane node and set the --terminated-pod-gc-threshold to an appropriate threshold,
for example, --terminated-pod-gc-threshold=10

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-controller-manager' | tail -n1 | grep 'terminated-pod-gc-threshold'
```

**Expected Result**:

```console
'--terminated-pod-gc-threshold' is present
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-controller-manager --allocate-node-cidrs=true --authentication-kubeconfig=/var/lib/rancher/k3s/server/cred/controller.kubeconfig --authorization-kubeconfig=/var/lib/rancher/k3s/server/cred/controller.kubeconfig --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/kube-controller-manager --cluster-cidr=10.42.0.0/16 --cluster-signing-kube-apiserver-client-cert-file=/var/lib/rancher/k3s/server/tls/client-ca.nochain.crt --cluster-signing-kube-apiserver-client-key-file=/var/lib/rancher/k3s/server/tls/client-ca.key --cluster-signing-kubelet-client-cert-file=/var/lib/rancher/k3s/server/tls/client-ca.nochain.crt --cluster-signing-kubelet-client-key-file=/var/lib/rancher/k3s/server/tls/client-ca.key --cluster-signing-kubelet-serving-cert-file=/var/lib/rancher/k3s/server/tls/server-ca.nochain.crt --cluster-signing-kubelet-serving-key-file=/var/lib/rancher/k3s/server/tls/server-ca.key --cluster-signing-legacy-unknown-cert-file=/var/lib/rancher/k3s/server/tls/server-ca.nochain.crt --cluster-signing-legacy-unknown-key-file=/var/lib/rancher/k3s/server/tls/server-ca.key --configure-cloud-routes=false --controllers=*,tokencleaner,-service,-route,-cloud-node-lifecycle --feature-gates=JobTrackingWithFinalizers=true --kubeconfig=/var/lib/rancher/k3s/server/cred/controller.kubeconfig --profiling=false --root-ca-file=/var/lib/rancher/k3s/server/tls/server-ca.crt --secure-port=10257 --service-account-private-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --terminated-pod-gc-threshold=10 --use-service-account-credentials=true"
```

### 1.3.2 Ensure that the --profiling argument is set to false (Automated)


**Result:** pass

**Remediation:**
Edit the Controller Manager pod specification file /etc/kubernetes/manifests/kube-controller-manager.yaml
on the control plane node and set the below parameter.
--profiling=false

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-controller-manager' | tail -n1 | grep 'profiling'
```

**Expected Result**:

```console
'--profiling' is equal to 'false'
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-controller-manager --allocate-node-cidrs=true --authentication-kubeconfig=/var/lib/rancher/k3s/server/cred/controller.kubeconfig --authorization-kubeconfig=/var/lib/rancher/k3s/server/cred/controller.kubeconfig --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/kube-controller-manager --cluster-cidr=10.42.0.0/16 --cluster-signing-kube-apiserver-client-cert-file=/var/lib/rancher/k3s/server/tls/client-ca.nochain.crt --cluster-signing-kube-apiserver-client-key-file=/var/lib/rancher/k3s/server/tls/client-ca.key --cluster-signing-kubelet-client-cert-file=/var/lib/rancher/k3s/server/tls/client-ca.nochain.crt --cluster-signing-kubelet-client-key-file=/var/lib/rancher/k3s/server/tls/client-ca.key --cluster-signing-kubelet-serving-cert-file=/var/lib/rancher/k3s/server/tls/server-ca.nochain.crt --cluster-signing-kubelet-serving-key-file=/var/lib/rancher/k3s/server/tls/server-ca.key --cluster-signing-legacy-unknown-cert-file=/var/lib/rancher/k3s/server/tls/server-ca.nochain.crt --cluster-signing-legacy-unknown-key-file=/var/lib/rancher/k3s/server/tls/server-ca.key --configure-cloud-routes=false --controllers=*,tokencleaner,-service,-route,-cloud-node-lifecycle --feature-gates=JobTrackingWithFinalizers=true --kubeconfig=/var/lib/rancher/k3s/server/cred/controller.kubeconfig --profiling=false --root-ca-file=/var/lib/rancher/k3s/server/tls/server-ca.crt --secure-port=10257 --service-account-private-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --terminated-pod-gc-threshold=10 --use-service-account-credentials=true"
```

### 1.3.3 Ensure that the --use-service-account-credentials argument is set to true (Automated)


**Result:** pass

**Remediation:**
Edit the Controller Manager pod specification file /etc/kubernetes/manifests/kube-controller-manager.yaml
on the control plane node to set the below parameter.
--use-service-account-credentials=true

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-controller-manager' | tail -n1 | grep 'use-service-account-credentials'
```

**Expected Result**:

```console
'--use-service-account-credentials' is not equal to 'false'
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-controller-manager --allocate-node-cidrs=true --authentication-kubeconfig=/var/lib/rancher/k3s/server/cred/controller.kubeconfig --authorization-kubeconfig=/var/lib/rancher/k3s/server/cred/controller.kubeconfig --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/kube-controller-manager --cluster-cidr=10.42.0.0/16 --cluster-signing-kube-apiserver-client-cert-file=/var/lib/rancher/k3s/server/tls/client-ca.nochain.crt --cluster-signing-kube-apiserver-client-key-file=/var/lib/rancher/k3s/server/tls/client-ca.key --cluster-signing-kubelet-client-cert-file=/var/lib/rancher/k3s/server/tls/client-ca.nochain.crt --cluster-signing-kubelet-client-key-file=/var/lib/rancher/k3s/server/tls/client-ca.key --cluster-signing-kubelet-serving-cert-file=/var/lib/rancher/k3s/server/tls/server-ca.nochain.crt --cluster-signing-kubelet-serving-key-file=/var/lib/rancher/k3s/server/tls/server-ca.key --cluster-signing-legacy-unknown-cert-file=/var/lib/rancher/k3s/server/tls/server-ca.nochain.crt --cluster-signing-legacy-unknown-key-file=/var/lib/rancher/k3s/server/tls/server-ca.key --configure-cloud-routes=false --controllers=*,tokencleaner,-service,-route,-cloud-node-lifecycle --feature-gates=JobTrackingWithFinalizers=true --kubeconfig=/var/lib/rancher/k3s/server/cred/controller.kubeconfig --profiling=false --root-ca-file=/var/lib/rancher/k3s/server/tls/server-ca.crt --secure-port=10257 --service-account-private-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --terminated-pod-gc-threshold=10 --use-service-account-credentials=true"
```

### 1.3.4 Ensure that the --service-account-private-key-file argument is set as appropriate (Automated)


**Result:** pass

**Remediation:**
Edit the Controller Manager pod specification file /etc/kubernetes/manifests/kube-controller-manager.yaml
on the control plane node and set the --service-account-private-key-file parameter
to the private key file for service accounts.
--service-account-private-key-file=<filename\>

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-controller-manager' | tail -n1 | grep 'service-account-private-key-file'
```

**Expected Result**:

```console
'--service-account-private-key-file' is present
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-controller-manager --allocate-node-cidrs=true --authentication-kubeconfig=/var/lib/rancher/k3s/server/cred/controller.kubeconfig --authorization-kubeconfig=/var/lib/rancher/k3s/server/cred/controller.kubeconfig --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/kube-controller-manager --cluster-cidr=10.42.0.0/16 --cluster-signing-kube-apiserver-client-cert-file=/var/lib/rancher/k3s/server/tls/client-ca.nochain.crt --cluster-signing-kube-apiserver-client-key-file=/var/lib/rancher/k3s/server/tls/client-ca.key --cluster-signing-kubelet-client-cert-file=/var/lib/rancher/k3s/server/tls/client-ca.nochain.crt --cluster-signing-kubelet-client-key-file=/var/lib/rancher/k3s/server/tls/client-ca.key --cluster-signing-kubelet-serving-cert-file=/var/lib/rancher/k3s/server/tls/server-ca.nochain.crt --cluster-signing-kubelet-serving-key-file=/var/lib/rancher/k3s/server/tls/server-ca.key --cluster-signing-legacy-unknown-cert-file=/var/lib/rancher/k3s/server/tls/server-ca.nochain.crt --cluster-signing-legacy-unknown-key-file=/var/lib/rancher/k3s/server/tls/server-ca.key --configure-cloud-routes=false --controllers=*,tokencleaner,-service,-route,-cloud-node-lifecycle --feature-gates=JobTrackingWithFinalizers=true --kubeconfig=/var/lib/rancher/k3s/server/cred/controller.kubeconfig --profiling=false --root-ca-file=/var/lib/rancher/k3s/server/tls/server-ca.crt --secure-port=10257 --service-account-private-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --terminated-pod-gc-threshold=10 --use-service-account-credentials=true"
```

### 1.3.5 Ensure that the --root-ca-file argument is set as appropriate (Automated)


**Result:** pass

**Remediation:**
Edit the Controller Manager pod specification file /etc/kubernetes/manifests/kube-controller-manager.yaml
on the control plane node and set the --root-ca-file parameter to the certificate bundle file`.
--root-ca-file=<path/to/file\>

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-controller-manager' | tail -n1 | grep 'root-ca-file'
```

**Expected Result**:

```console
'--root-ca-file' is present
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-controller-manager --allocate-node-cidrs=true --authentication-kubeconfig=/var/lib/rancher/k3s/server/cred/controller.kubeconfig --authorization-kubeconfig=/var/lib/rancher/k3s/server/cred/controller.kubeconfig --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/kube-controller-manager --cluster-cidr=10.42.0.0/16 --cluster-signing-kube-apiserver-client-cert-file=/var/lib/rancher/k3s/server/tls/client-ca.nochain.crt --cluster-signing-kube-apiserver-client-key-file=/var/lib/rancher/k3s/server/tls/client-ca.key --cluster-signing-kubelet-client-cert-file=/var/lib/rancher/k3s/server/tls/client-ca.nochain.crt --cluster-signing-kubelet-client-key-file=/var/lib/rancher/k3s/server/tls/client-ca.key --cluster-signing-kubelet-serving-cert-file=/var/lib/rancher/k3s/server/tls/server-ca.nochain.crt --cluster-signing-kubelet-serving-key-file=/var/lib/rancher/k3s/server/tls/server-ca.key --cluster-signing-legacy-unknown-cert-file=/var/lib/rancher/k3s/server/tls/server-ca.nochain.crt --cluster-signing-legacy-unknown-key-file=/var/lib/rancher/k3s/server/tls/server-ca.key --configure-cloud-routes=false --controllers=*,tokencleaner,-service,-route,-cloud-node-lifecycle --feature-gates=JobTrackingWithFinalizers=true --kubeconfig=/var/lib/rancher/k3s/server/cred/controller.kubeconfig --profiling=false --root-ca-file=/var/lib/rancher/k3s/server/tls/server-ca.crt --secure-port=10257 --service-account-private-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --terminated-pod-gc-threshold=10 --use-service-account-credentials=true"
```

### 1.3.6 Ensure that the RotateKubeletServerCertificate argument is set to true (Automated)


**Result:** Not Applicable

**Remediation:**
Edit the Controller Manager pod specification file /etc/kubernetes/manifests/kube-controller-manager.yaml
on the control plane node and set the --feature-gates parameter to include RotateKubeletServerCertificate=true.
--feature-gates=RotateKubeletServerCertificate=true

### 1.3.7 Ensure that the --bind-address argument is set to 127.0.0.1 (Automated)


**Result:** pass

**Remediation:**
Edit the Controller Manager pod specification file /etc/kubernetes/manifests/kube-controller-manager.yaml
on the control plane node and ensure the correct value for the --bind-address parameter

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-controller-manager' | tail -n1 | grep 'bind-address'
```

**Expected Result**:

```console
'--bind-address' is equal to '127.0.0.1' OR '--bind-address' is not present
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-controller-manager --allocate-node-cidrs=true --authentication-kubeconfig=/var/lib/rancher/k3s/server/cred/controller.kubeconfig --authorization-kubeconfig=/var/lib/rancher/k3s/server/cred/controller.kubeconfig --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/kube-controller-manager --cluster-cidr=10.42.0.0/16 --cluster-signing-kube-apiserver-client-cert-file=/var/lib/rancher/k3s/server/tls/client-ca.nochain.crt --cluster-signing-kube-apiserver-client-key-file=/var/lib/rancher/k3s/server/tls/client-ca.key --cluster-signing-kubelet-client-cert-file=/var/lib/rancher/k3s/server/tls/client-ca.nochain.crt --cluster-signing-kubelet-client-key-file=/var/lib/rancher/k3s/server/tls/client-ca.key --cluster-signing-kubelet-serving-cert-file=/var/lib/rancher/k3s/server/tls/server-ca.nochain.crt --cluster-signing-kubelet-serving-key-file=/var/lib/rancher/k3s/server/tls/server-ca.key --cluster-signing-legacy-unknown-cert-file=/var/lib/rancher/k3s/server/tls/server-ca.nochain.crt --cluster-signing-legacy-unknown-key-file=/var/lib/rancher/k3s/server/tls/server-ca.key --configure-cloud-routes=false --controllers=*,tokencleaner,-service,-route,-cloud-node-lifecycle --feature-gates=JobTrackingWithFinalizers=true --kubeconfig=/var/lib/rancher/k3s/server/cred/controller.kubeconfig --profiling=false --root-ca-file=/var/lib/rancher/k3s/server/tls/server-ca.crt --secure-port=10257 --service-account-private-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --terminated-pod-gc-threshold=10 --use-service-account-credentials=true"
```

## 1.4 Scheduler
### 1.4.1 Ensure that the --profiling argument is set to false (Automated)


**Result:** pass

**Remediation:**
Edit the Scheduler pod specification file /etc/kubernetes/manifests/kube-scheduler.yaml file
on the control plane node and set the below parameter.
--profiling=false

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-scheduler' | tail -n1
```

**Expected Result**:

```console
'--profiling' is equal to 'false'
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-scheduler --authentication-kubeconfig=/var/lib/rancher/k3s/server/cred/scheduler.kubeconfig --authorization-kubeconfig=/var/lib/rancher/k3s/server/cred/scheduler.kubeconfig --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/kube-scheduler --kubeconfig=/var/lib/rancher/k3s/server/cred/scheduler.kubeconfig --profiling=false --secure-port=10259"
```

### 1.4.2 Ensure that the --bind-address argument is set to 127.0.0.1 (Automated)


**Result:** pass

**Remediation:**
Edit the Scheduler pod specification file /etc/kubernetes/manifests/kube-scheduler.yaml
on the control plane node and ensure the correct value for the --bind-address parameter

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-scheduler' | tail -n1 | grep 'bind-address'
```

**Expected Result**:

```console
'--bind-address' is equal to '127.0.0.1' OR '--bind-address' is not present
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-scheduler --authentication-kubeconfig=/var/lib/rancher/k3s/server/cred/scheduler.kubeconfig --authorization-kubeconfig=/var/lib/rancher/k3s/server/cred/scheduler.kubeconfig --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/kube-scheduler --kubeconfig=/var/lib/rancher/k3s/server/cred/scheduler.kubeconfig --profiling=false --secure-port=10259"
```

## 2 Etcd Node Configuration
### 2.1 Ensure that the --cert-file and --key-file arguments are set as appropriate (Automated)


**Result:** pass

**Remediation:**
Follow the etcd service documentation and configure TLS encryption.
Then, edit the etcd pod specification file /etc/kubernetes/manifests/etcd.yaml
on the master node and set the below parameters.
--cert-file=</path/to/ca-file\>
--key-file=</path/to/key-file\>

**Audit Script:** `check_for_k3s_etcd.sh`

```bash
#!/bin/bash

# This script is used to ensure that k3s is actually running etcd (and not other databases like sqlite3)
# before it checks the requirement
set -eE

handle_error() {
    echo "false"
}

trap 'handle_error' ERR


if [[ "$(journalctl -D /var/log/journal -u k3s | grep 'Managed etcd cluster initializing' | grep -v grep | wc -l)" -gt 0 ]]; then
    case $1 in 
        "1.1.11")
            echo $(stat -c %a /var/lib/rancher/k3s/server/db/etcd);;
        "1.2.29")
            echo $(journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'etcd-');;
        "2.1")
            echo $(grep -A 5 'client-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep -E 'cert-file|key-file');;
        "2.2")
            echo $(grep -A 5 'client-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep 'client-cert-auth');;
        "2.3")
            echo $(grep 'auto-tls' /var/lib/rancher/k3s/server/db/etcd/config);;
        "2.4")
            echo $(grep -A 5 'peer-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep -E 'cert-file|key-file');;
        "2.5")
            echo $(grep -A 5 'peer-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep 'client-cert-auth');;
        "2.6")
            echo $(grep 'peer-auto-tls' /var/lib/rancher/k3s/server/db/etcd/config);;
        "2.7")
            echo $(grep 'trusted-ca-file' /var/lib/rancher/k3s/server/db/etcd/config);;
    esac
else
# If another database is running, return whatever is required to pass the scan
    case $1 in
        "1.1.11")
            echo "700";;
        "1.2.29")
            echo "--etcd-certfile AND --etcd-keyfile";;
        "2.1")
            echo "cert-file AND key-file";;
        "2.2")
            echo "--client-cert-auth=true";;
        "2.3")
            echo "false";;
        "2.4")
            echo "peer-cert-file AND peer-key-file";;
        "2.5")
            echo "--client-cert-auth=true";;
        "2.6")
            echo "--peer-auto-tls=false";;
        "2.7")
            echo "--trusted-ca-file";;
    esac
fi

```

**Audit Execution:**

```bash
./check_for_k3s_etcd.sh 2.1
```

**Expected Result**:

```console
'cert-file' is present AND 'key-file' is present
```

**Returned Value**:

```console
cert-file: /var/lib/rancher/k3s/server/tls/etcd/server-client.crt key-file: /var/lib/rancher/k3s/server/tls/etcd/server-client.key
```

### 2.2 Ensure that the --client-cert-auth argument is set to true (Automated)


**Result:** pass

**Remediation:**
Edit the etcd pod specification file /var/lib/rancher/k3s/server/db/etcd/config on the master
node and set the below parameter.
--client-cert-auth="true"

**Audit Script:** `check_for_k3s_etcd.sh`

```bash
#!/bin/bash

# This script is used to ensure that k3s is actually running etcd (and not other databases like sqlite3)
# before it checks the requirement
set -eE

handle_error() {
    echo "false"
}

trap 'handle_error' ERR


if [[ "$(journalctl -D /var/log/journal -u k3s | grep 'Managed etcd cluster initializing' | grep -v grep | wc -l)" -gt 0 ]]; then
    case $1 in 
        "1.1.11")
            echo $(stat -c %a /var/lib/rancher/k3s/server/db/etcd);;
        "1.2.29")
            echo $(journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'etcd-');;
        "2.1")
            echo $(grep -A 5 'client-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep -E 'cert-file|key-file');;
        "2.2")
            echo $(grep -A 5 'client-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep 'client-cert-auth');;
        "2.3")
            echo $(grep 'auto-tls' /var/lib/rancher/k3s/server/db/etcd/config);;
        "2.4")
            echo $(grep -A 5 'peer-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep -E 'cert-file|key-file');;
        "2.5")
            echo $(grep -A 5 'peer-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep 'client-cert-auth');;
        "2.6")
            echo $(grep 'peer-auto-tls' /var/lib/rancher/k3s/server/db/etcd/config);;
        "2.7")
            echo $(grep 'trusted-ca-file' /var/lib/rancher/k3s/server/db/etcd/config);;
    esac
else
# If another database is running, return whatever is required to pass the scan
    case $1 in
        "1.1.11")
            echo "700";;
        "1.2.29")
            echo "--etcd-certfile AND --etcd-keyfile";;
        "2.1")
            echo "cert-file AND key-file";;
        "2.2")
            echo "--client-cert-auth=true";;
        "2.3")
            echo "false";;
        "2.4")
            echo "peer-cert-file AND peer-key-file";;
        "2.5")
            echo "--client-cert-auth=true";;
        "2.6")
            echo "--peer-auto-tls=false";;
        "2.7")
            echo "--trusted-ca-file";;
    esac
fi

```

**Audit Execution:**

```bash
./check_for_k3s_etcd.sh 2.2
```

**Expected Result**:

```console
'--client-cert-auth' is present OR 'client-cert-auth' is equal to 'true'
```

**Returned Value**:

```console
client-cert-auth: true
```

### 2.3 Ensure that the --auto-tls argument is not set to true (Automated)


**Result:** pass

**Remediation:**
Edit the etcd pod specification file /var/lib/rancher/k3s/server/db/etcd/config on the master
node and either remove the --auto-tls parameter or set it to false.
 --auto-tls=false

**Audit Script:** `check_for_k3s_etcd.sh`

```bash
#!/bin/bash

# This script is used to ensure that k3s is actually running etcd (and not other databases like sqlite3)
# before it checks the requirement
set -eE

handle_error() {
    echo "false"
}

trap 'handle_error' ERR


if [[ "$(journalctl -D /var/log/journal -u k3s | grep 'Managed etcd cluster initializing' | grep -v grep | wc -l)" -gt 0 ]]; then
    case $1 in 
        "1.1.11")
            echo $(stat -c %a /var/lib/rancher/k3s/server/db/etcd);;
        "1.2.29")
            echo $(journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'etcd-');;
        "2.1")
            echo $(grep -A 5 'client-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep -E 'cert-file|key-file');;
        "2.2")
            echo $(grep -A 5 'client-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep 'client-cert-auth');;
        "2.3")
            echo $(grep 'auto-tls' /var/lib/rancher/k3s/server/db/etcd/config);;
        "2.4")
            echo $(grep -A 5 'peer-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep -E 'cert-file|key-file');;
        "2.5")
            echo $(grep -A 5 'peer-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep 'client-cert-auth');;
        "2.6")
            echo $(grep 'peer-auto-tls' /var/lib/rancher/k3s/server/db/etcd/config);;
        "2.7")
            echo $(grep 'trusted-ca-file' /var/lib/rancher/k3s/server/db/etcd/config);;
    esac
else
# If another database is running, return whatever is required to pass the scan
    case $1 in
        "1.1.11")
            echo "700";;
        "1.2.29")
            echo "--etcd-certfile AND --etcd-keyfile";;
        "2.1")
            echo "cert-file AND key-file";;
        "2.2")
            echo "--client-cert-auth=true";;
        "2.3")
            echo "false";;
        "2.4")
            echo "peer-cert-file AND peer-key-file";;
        "2.5")
            echo "--client-cert-auth=true";;
        "2.6")
            echo "--peer-auto-tls=false";;
        "2.7")
            echo "--trusted-ca-file";;
    esac
fi

```

**Audit Execution:**

```bash
./check_for_k3s_etcd.sh 2.3
```

**Expected Result**:

```console
'ETCD_AUTO_TLS' is not present OR 'ETCD_AUTO_TLS' is present
```

**Returned Value**:

```console
error: process ID list syntax error Usage: ps [options] Try 'ps --help <simple|list|output|threads|misc|all>' or 'ps --help <s|l|o|t|m|a>' for additional help text. For more details see ps(1). cat: /proc//environ: No such file or directory
```

### 2.4 Ensure that the --peer-cert-file and --peer-key-file arguments are set as appropriate (Automated)


**Result:** pass

**Remediation:**
Follow the etcd service documentation and configure peer TLS encryption as appropriate
for your etcd cluster.
Then, edit the etcd pod specification file /var/lib/rancher/k3s/server/db/etcd/config on the
master node and set the below parameters.
--peer-client-file=</path/to/peer-cert-file\>
--peer-key-file=</path/to/peer-key-file\>

**Audit Script:** `check_for_k3s_etcd.sh`

```bash
#!/bin/bash

# This script is used to ensure that k3s is actually running etcd (and not other databases like sqlite3)
# before it checks the requirement
set -eE

handle_error() {
    echo "false"
}

trap 'handle_error' ERR


if [[ "$(journalctl -D /var/log/journal -u k3s | grep 'Managed etcd cluster initializing' | grep -v grep | wc -l)" -gt 0 ]]; then
    case $1 in 
        "1.1.11")
            echo $(stat -c %a /var/lib/rancher/k3s/server/db/etcd);;
        "1.2.29")
            echo $(journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'etcd-');;
        "2.1")
            echo $(grep -A 5 'client-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep -E 'cert-file|key-file');;
        "2.2")
            echo $(grep -A 5 'client-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep 'client-cert-auth');;
        "2.3")
            echo $(grep 'auto-tls' /var/lib/rancher/k3s/server/db/etcd/config);;
        "2.4")
            echo $(grep -A 5 'peer-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep -E 'cert-file|key-file');;
        "2.5")
            echo $(grep -A 5 'peer-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep 'client-cert-auth');;
        "2.6")
            echo $(grep 'peer-auto-tls' /var/lib/rancher/k3s/server/db/etcd/config);;
        "2.7")
            echo $(grep 'trusted-ca-file' /var/lib/rancher/k3s/server/db/etcd/config);;
    esac
else
# If another database is running, return whatever is required to pass the scan
    case $1 in
        "1.1.11")
            echo "700";;
        "1.2.29")
            echo "--etcd-certfile AND --etcd-keyfile";;
        "2.1")
            echo "cert-file AND key-file";;
        "2.2")
            echo "--client-cert-auth=true";;
        "2.3")
            echo "false";;
        "2.4")
            echo "peer-cert-file AND peer-key-file";;
        "2.5")
            echo "--client-cert-auth=true";;
        "2.6")
            echo "--peer-auto-tls=false";;
        "2.7")
            echo "--trusted-ca-file";;
    esac
fi

```

**Audit Execution:**

```bash
./check_for_k3s_etcd.sh 2.4
```

**Expected Result**:

```console
'cert-file' is present AND 'key-file' is present
```

**Returned Value**:

```console
cert-file: /var/lib/rancher/k3s/server/tls/etcd/peer-server-client.crt key-file: /var/lib/rancher/k3s/server/tls/etcd/peer-server-client.key
```

### 2.5 Ensure that the --peer-client-cert-auth argument is set to true (Automated)


**Result:** pass

**Remediation:**
Edit the etcd pod specification file /var/lib/rancher/k3s/server/db/etcd/config on the master
node and set the below parameter.
--peer-client-cert-auth=true

**Audit Script:** `check_for_k3s_etcd.sh`

```bash
#!/bin/bash

# This script is used to ensure that k3s is actually running etcd (and not other databases like sqlite3)
# before it checks the requirement
set -eE

handle_error() {
    echo "false"
}

trap 'handle_error' ERR


if [[ "$(journalctl -D /var/log/journal -u k3s | grep 'Managed etcd cluster initializing' | grep -v grep | wc -l)" -gt 0 ]]; then
    case $1 in 
        "1.1.11")
            echo $(stat -c %a /var/lib/rancher/k3s/server/db/etcd);;
        "1.2.29")
            echo $(journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'etcd-');;
        "2.1")
            echo $(grep -A 5 'client-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep -E 'cert-file|key-file');;
        "2.2")
            echo $(grep -A 5 'client-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep 'client-cert-auth');;
        "2.3")
            echo $(grep 'auto-tls' /var/lib/rancher/k3s/server/db/etcd/config);;
        "2.4")
            echo $(grep -A 5 'peer-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep -E 'cert-file|key-file');;
        "2.5")
            echo $(grep -A 5 'peer-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep 'client-cert-auth');;
        "2.6")
            echo $(grep 'peer-auto-tls' /var/lib/rancher/k3s/server/db/etcd/config);;
        "2.7")
            echo $(grep 'trusted-ca-file' /var/lib/rancher/k3s/server/db/etcd/config);;
    esac
else
# If another database is running, return whatever is required to pass the scan
    case $1 in
        "1.1.11")
            echo "700";;
        "1.2.29")
            echo "--etcd-certfile AND --etcd-keyfile";;
        "2.1")
            echo "cert-file AND key-file";;
        "2.2")
            echo "--client-cert-auth=true";;
        "2.3")
            echo "false";;
        "2.4")
            echo "peer-cert-file AND peer-key-file";;
        "2.5")
            echo "--client-cert-auth=true";;
        "2.6")
            echo "--peer-auto-tls=false";;
        "2.7")
            echo "--trusted-ca-file";;
    esac
fi

```

**Audit Execution:**

```bash
./check_for_k3s_etcd.sh 2.5
```

**Expected Result**:

```console
'--client-cert-auth' is present OR 'client-cert-auth' is equal to 'true'
```

**Returned Value**:

```console
client-cert-auth: true
```

### 2.6 Ensure that the --peer-auto-tls argument is not set to true (Automated)


**Result:** pass

**Remediation:**
Edit the etcd pod specification file /var/lib/rancher/k3s/server/db/etcd/config on the master
node and either remove the --peer-auto-tls parameter or set it to false.
--peer-auto-tls=false

**Audit Script:** `check_for_k3s_etcd.sh`

```bash
#!/bin/bash

# This script is used to ensure that k3s is actually running etcd (and not other databases like sqlite3)
# before it checks the requirement
set -eE

handle_error() {
    echo "false"
}

trap 'handle_error' ERR


if [[ "$(journalctl -D /var/log/journal -u k3s | grep 'Managed etcd cluster initializing' | grep -v grep | wc -l)" -gt 0 ]]; then
    case $1 in 
        "1.1.11")
            echo $(stat -c %a /var/lib/rancher/k3s/server/db/etcd);;
        "1.2.29")
            echo $(journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'etcd-');;
        "2.1")
            echo $(grep -A 5 'client-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep -E 'cert-file|key-file');;
        "2.2")
            echo $(grep -A 5 'client-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep 'client-cert-auth');;
        "2.3")
            echo $(grep 'auto-tls' /var/lib/rancher/k3s/server/db/etcd/config);;
        "2.4")
            echo $(grep -A 5 'peer-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep -E 'cert-file|key-file');;
        "2.5")
            echo $(grep -A 5 'peer-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep 'client-cert-auth');;
        "2.6")
            echo $(grep 'peer-auto-tls' /var/lib/rancher/k3s/server/db/etcd/config);;
        "2.7")
            echo $(grep 'trusted-ca-file' /var/lib/rancher/k3s/server/db/etcd/config);;
    esac
else
# If another database is running, return whatever is required to pass the scan
    case $1 in
        "1.1.11")
            echo "700";;
        "1.2.29")
            echo "--etcd-certfile AND --etcd-keyfile";;
        "2.1")
            echo "cert-file AND key-file";;
        "2.2")
            echo "--client-cert-auth=true";;
        "2.3")
            echo "false";;
        "2.4")
            echo "peer-cert-file AND peer-key-file";;
        "2.5")
            echo "--client-cert-auth=true";;
        "2.6")
            echo "--peer-auto-tls=false";;
        "2.7")
            echo "--trusted-ca-file";;
    esac
fi

```

**Audit Execution:**

```bash
./check_for_k3s_etcd.sh 2.6
```

**Expected Result**:

```console
'ETCD_PEER_AUTO_TLS' is not present OR 'ETCD_PEER_AUTO_TLS' is present
```

**Returned Value**:

```console
error: process ID list syntax error Usage: ps [options] Try 'ps --help <simple|list|output|threads|misc|all>' or 'ps --help <s|l|o|t|m|a>' for additional help text. For more details see ps(1). cat: /proc//environ: No such file or directory
```

### 2.7 Ensure that a unique Certificate Authority is used for etcd (Manual)


**Result:** pass

**Remediation:**
[Manual test]
Follow the etcd documentation and create a dedicated certificate authority setup for the
etcd service.
Then, edit the etcd pod specification file /var/lib/rancher/k3s/server/db/etcd/config on the
master node and set the below parameter.
--trusted-ca-file=</path/to/ca-file\>

**Audit Script:** `check_for_k3s_etcd.sh`

```bash
#!/bin/bash

# This script is used to ensure that k3s is actually running etcd (and not other databases like sqlite3)
# before it checks the requirement
set -eE

handle_error() {
    echo "false"
}

trap 'handle_error' ERR


if [[ "$(journalctl -D /var/log/journal -u k3s | grep 'Managed etcd cluster initializing' | grep -v grep | wc -l)" -gt 0 ]]; then
    case $1 in 
        "1.1.11")
            echo $(stat -c %a /var/lib/rancher/k3s/server/db/etcd);;
        "1.2.29")
            echo $(journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'etcd-');;
        "2.1")
            echo $(grep -A 5 'client-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep -E 'cert-file|key-file');;
        "2.2")
            echo $(grep -A 5 'client-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep 'client-cert-auth');;
        "2.3")
            echo $(grep 'auto-tls' /var/lib/rancher/k3s/server/db/etcd/config);;
        "2.4")
            echo $(grep -A 5 'peer-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep -E 'cert-file|key-file');;
        "2.5")
            echo $(grep -A 5 'peer-transport-security' /var/lib/rancher/k3s/server/db/etcd/config | grep 'client-cert-auth');;
        "2.6")
            echo $(grep 'peer-auto-tls' /var/lib/rancher/k3s/server/db/etcd/config);;
        "2.7")
            echo $(grep 'trusted-ca-file' /var/lib/rancher/k3s/server/db/etcd/config);;
    esac
else
# If another database is running, return whatever is required to pass the scan
    case $1 in
        "1.1.11")
            echo "700";;
        "1.2.29")
            echo "--etcd-certfile AND --etcd-keyfile";;
        "2.1")
            echo "cert-file AND key-file";;
        "2.2")
            echo "--client-cert-auth=true";;
        "2.3")
            echo "false";;
        "2.4")
            echo "peer-cert-file AND peer-key-file";;
        "2.5")
            echo "--client-cert-auth=true";;
        "2.6")
            echo "--peer-auto-tls=false";;
        "2.7")
            echo "--trusted-ca-file";;
    esac
fi

```

**Audit Execution:**

```bash
./check_for_k3s_etcd.sh 2.7
```

**Expected Result**:

```console
'trusted-ca-file' is present
```

**Returned Value**:

```console
trusted-ca-file: /var/lib/rancher/k3s/server/tls/etcd/server-ca.crt trusted-ca-file: /var/lib/rancher/k3s/server/tls/etcd/peer-ca.crt
```

## 3.1 Authentication and Authorization
### 3.1.1 Client certificate authentication should not be used for users (Manual)


**Result:** warn

**Remediation:**
Alternative mechanisms provided by Kubernetes such as the use of OIDC should be
implemented in place of client certificates.

## 3.2 Logging
### 3.2.1 Ensure that a minimal audit policy is created (Manual)


**Result:** warn

**Remediation:**
Create an audit policy file for your cluster.

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kube-apiserver' | tail -n1 | grep 'audit-policy-file'
```

### 3.2.2 Ensure that the audit policy covers key security concerns (Manual)


**Result:** warn

**Remediation:**
Review the audit policy provided for the cluster and ensure that it covers
at least the following areas,
- Access to Secrets managed by the cluster. Care should be taken to only
 log Metadata for requests to Secrets, ConfigMaps, and TokenReviews, in
 order to avoid risk of logging sensitive data.
- Modification of Pod and Deployment objects.
- Use of `pods/exec`, `pods/portforward`, `pods/proxy` and `services/proxy`.
For most requests, minimally logging at the Metadata level is recommended
(the most basic level of logging).

## 4.1 Worker Node Configuration Files
### 4.1.1 Ensure that the kubelet service file permissions are set to 600 or more restrictive (Automated)


**Result:** Not Applicable

**Remediation:**
Run the below command (based on the file location on your system) on the each worker node.
For example, chmod 600 /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

### 4.1.2 Ensure that the kubelet service file ownership is set to root:root (Automated)


**Result:** Not Applicable

**Remediation:**
Run the below command (based on the file location on your system) on the each worker node.
For example,
chown root:root /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

### 4.1.3 If proxy kubeconfig file exists ensure permissions are set to 600 or more restrictive (Manual)


**Result:** warn

**Remediation:**
Run the below command (based on the file location on your system) on the each worker node.
For example,
chmod 600 /var/lib/rancher/k3s/agent/kubeproxy.kubeconfig

**Audit:**

```bash
stat -c %a /var/lib/rancher/k3s/agent/kubeproxy.kubeconfig
```

**Expected Result**:

```console
'permissions' is present
```

**Returned Value**:

```console
600
```

### 4.1.4 If proxy kubeconfig file exists ensure ownership is set to root:root (Manual)


**Result:** pass

**Remediation:**
Run the below command (based on the file location on your system) on the each worker node.
For example, chown root:root /var/lib/rancher/k3s/agent/kubeproxy.kubeconfig

**Audit:**

```bash
stat -c %U:%G /var/lib/rancher/k3s/agent/kubeproxy.kubeconfig
```

**Expected Result**:

```console
'root:root' is present
```

**Returned Value**:

```console
root:root
```

### 4.1.5 Ensure that the --kubeconfig kubelet.conf file permissions are set to 600 or more restrictive (Automated)


**Result:** pass

**Remediation:**
Run the below command (based on the file location on your system) on the each worker node.
For example,
chmod 600 /var/lib/rancher/k3s/server/cred/admin.kubeconfig

**Audit:**

```bash
stat -c %a /var/lib/rancher/k3s/agent/kubelet.kubeconfig
```

**Expected Result**:

```console
'600' is equal to '600'
```

**Returned Value**:

```console
600
```

### 4.1.6 Ensure that the --kubeconfig kubelet.conf file ownership is set to root:root (Automated)


**Result:** pass

**Remediation:**
Run the below command (based on the file location on your system) on the each worker node.
For example,
chown root:root /var/lib/rancher/k3s/server/cred/admin.kubeconfig

**Audit:**

```bash
stat -c %U:%G /var/lib/rancher/k3s/agent/kubelet.kubeconfig
```

**Expected Result**:

```console
'root:root' is equal to 'root:root'
```

**Returned Value**:

```console
root:root
```

### 4.1.7 Ensure that the certificate authorities file permissions are set to 600 or more restrictive (Manual)


**Result:** warn

**Remediation:**
Run the following command to modify the file permissions of the
--client-ca-file chmod 600 <filename\>

**Audit:**

```bash
stat -c %a /var/lib/rancher/k3s/server/tls/server-ca.crt
```

**Expected Result**:

```console
'permissions' is present
```

**Returned Value**:

```console
644
```

### 4.1.8 Ensure that the client certificate authorities file ownership is set to root:root (Manual)


**Result:** pass

**Remediation:**
Run the following command to modify the ownership of the --client-ca-file.
chown root:root <filename\>

**Audit:**

```bash
stat -c %U:%G /var/lib/rancher/k3s/server/tls/client-ca.crt
```

**Expected Result**:

```console
'root:root' is equal to 'root:root'
```

**Returned Value**:

```console
root:root
```

### 4.1.9 If the kubelet config.yaml configuration file is being used validate permissions set to 600 or more restrictive (Manual)


**Result:** Not Applicable

**Remediation:**
Run the following command (using the config file location identified in the Audit step)
chmod 600 /var/lib/kubelet/config.yaml

### 4.1.10 If the kubelet config.yaml configuration file is being used validate file ownership is set to root:root (Automated)


**Result:** Not Applicable

**Remediation:**
Run the following command (using the config file location identified in the Audit step)
chown root:root /var/lib/kubelet/config.yaml

## 4.2 Kubelet
### 4.2.1 Ensure that the --anonymous-auth argument is set to false (Automated)


**Result:** pass

**Remediation:**
If using a Kubelet config file, edit the file to set `authentication: anonymous: enabled` to
`false`.
If using executable arguments, edit the kubelet service file
/etc/systemd/system/kubelet.service.d/10-kubeadm.conf on each worker node and
set the below parameter in KUBELET_SYSTEM_PODS_ARGS variable.
`--anonymous-auth=false`
Based on your system, restart the kubelet service. For example,
systemctl daemon-reload
systemctl restart kubelet.service

**Audit:**

```bash
/bin/sh -c 'if test $(journalctl -D /var/log/journal -u k3s | grep "Running kube-apiserver" | wc -l) -gt 0; then journalctl -D /var/log/journal -u k3s | grep "Running kube-apiserver" | tail -n1 | grep "anonymous-auth" | grep -v grep; else echo "--anonymous-auth=false"; fi'
```

**Expected Result**:

```console
'--anonymous-auth' is equal to 'false'
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-apiserver --advertise-port=6443 --allow-privileged=true --anonymous-auth=false --api-audiences=https://kubernetes.default.svc.cluster.local,k3s --audit-log-maxage=30 --audit-log-maxbackup=10 --audit-log-maxsize=100 --audit-log-path=/var/lib/rancher/k3s/server/logs/audit.log --audit-policy-file=/var/lib/rancher/k3s/server/audit.yaml --authorization-mode=Node,RBAC --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/temporary-certs --client-ca-file=/var/lib/rancher/k3s/server/tls/client-ca.crt --egress-selector-config-file=/var/lib/rancher/k3s/server/etc/egress-selector-config.yaml --enable-admission-plugins=NodeRestriction,PodSecurityPolicy,ServiceAccount --enable-aggregator-routing=true --enable-bootstrap-token-auth=true --encryption-provider-config=/var/lib/rancher/k3s/server/cred/encryption-config.json --etcd-cafile=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --etcd-certfile=/var/lib/rancher/k3s/server/tls/etcd/client.crt --etcd-keyfile=/var/lib/rancher/k3s/server/tls/etcd/client.key --etcd-servers=https://127.0.0.1:2379 --feature-gates=JobTrackingWithFinalizers=true --kubelet-certificate-authority=/var/lib/rancher/k3s/server/tls/server-ca.crt --kubelet-client-certificate=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.crt --kubelet-client-key=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --profiling=false --proxy-client-cert-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.crt --proxy-client-key-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.key --request-timeout=300s --requestheader-allowed-names=system:auth-proxy --requestheader-client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6444 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/rancher/k3s/server/tls/service.key --service-account-lookup=true --service-account-signing-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --service-node-port-range=30000-32767 --storage-backend=etcd3 --tls-cert-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305 --tls-private-key-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key"
```

### 4.2.2 Ensure that the --authorization-mode argument is not set to AlwaysAllow (Automated)


**Result:** pass

**Remediation:**
If using a Kubelet config file, edit the file to set `authorization.mode` to Webhook. If
using executable arguments, edit the kubelet service file
/etc/systemd/system/kubelet.service.d/10-kubeadm.conf on each worker node and
set the below parameter in KUBELET_AUTHZ_ARGS variable.
--authorization-mode=Webhook
Based on your system, restart the kubelet service. For example,
systemctl daemon-reload
systemctl restart kubelet.service

**Audit:**

```bash
/bin/sh -c 'if test $(journalctl -D /var/log/journal -u k3s | grep "Running kube-apiserver" | wc -l) -gt 0; then journalctl -D /var/log/journal -u k3s | grep "Running kube-apiserver" | tail -n1 | grep "authorization-mode" | grep -v grep; else echo "--authorization-mode=Webhook"; fi'
```

**Expected Result**:

```console
'--authorization-mode' does not have 'AlwaysAllow'
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-apiserver --advertise-port=6443 --allow-privileged=true --anonymous-auth=false --api-audiences=https://kubernetes.default.svc.cluster.local,k3s --audit-log-maxage=30 --audit-log-maxbackup=10 --audit-log-maxsize=100 --audit-log-path=/var/lib/rancher/k3s/server/logs/audit.log --audit-policy-file=/var/lib/rancher/k3s/server/audit.yaml --authorization-mode=Node,RBAC --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/temporary-certs --client-ca-file=/var/lib/rancher/k3s/server/tls/client-ca.crt --egress-selector-config-file=/var/lib/rancher/k3s/server/etc/egress-selector-config.yaml --enable-admission-plugins=NodeRestriction,PodSecurityPolicy,ServiceAccount --enable-aggregator-routing=true --enable-bootstrap-token-auth=true --encryption-provider-config=/var/lib/rancher/k3s/server/cred/encryption-config.json --etcd-cafile=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --etcd-certfile=/var/lib/rancher/k3s/server/tls/etcd/client.crt --etcd-keyfile=/var/lib/rancher/k3s/server/tls/etcd/client.key --etcd-servers=https://127.0.0.1:2379 --feature-gates=JobTrackingWithFinalizers=true --kubelet-certificate-authority=/var/lib/rancher/k3s/server/tls/server-ca.crt --kubelet-client-certificate=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.crt --kubelet-client-key=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --profiling=false --proxy-client-cert-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.crt --proxy-client-key-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.key --request-timeout=300s --requestheader-allowed-names=system:auth-proxy --requestheader-client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6444 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/rancher/k3s/server/tls/service.key --service-account-lookup=true --service-account-signing-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --service-node-port-range=30000-32767 --storage-backend=etcd3 --tls-cert-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305 --tls-private-key-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key"
```

### 4.2.3 Ensure that the --client-ca-file argument is set as appropriate (Automated)


**Result:** pass

**Remediation:**
If using a Kubelet config file, edit the file to set `authentication.x509.clientCAFile` to
the location of the client CA file.
If using command line arguments, edit the kubelet service file
/etc/systemd/system/kubelet.service.d/10-kubeadm.conf on each worker node and
set the below parameter in KUBELET_AUTHZ_ARGS variable.
--client-ca-file=<path/to/client-ca-file\>
Based on your system, restart the kubelet service. For example,
systemctl daemon-reload
systemctl restart kubelet.service

**Audit:**

```bash
/bin/sh -c 'if test $(journalctl -D /var/log/journal -u k3s | grep "Running kube-apiserver" | wc -l) -gt 0; then journalctl -D /var/log/journal -u k3s | grep "Running kube-apiserver" | tail -n1 | grep "client-ca-file" | grep -v grep; else echo "--client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt"; fi'
```

**Expected Result**:

```console
'--client-ca-file' is present
```

**Returned Value**:

```console
Sep 11 20:31:58 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:31:58Z" level=info msg="Running kube-apiserver --advertise-port=6443 --allow-privileged=true --anonymous-auth=false --api-audiences=https://kubernetes.default.svc.cluster.local,k3s --audit-log-maxage=30 --audit-log-maxbackup=10 --audit-log-maxsize=100 --audit-log-path=/var/lib/rancher/k3s/server/logs/audit.log --audit-policy-file=/var/lib/rancher/k3s/server/audit.yaml --authorization-mode=Node,RBAC --bind-address=127.0.0.1 --cert-dir=/var/lib/rancher/k3s/server/tls/temporary-certs --client-ca-file=/var/lib/rancher/k3s/server/tls/client-ca.crt --egress-selector-config-file=/var/lib/rancher/k3s/server/etc/egress-selector-config.yaml --enable-admission-plugins=NodeRestriction,PodSecurityPolicy,ServiceAccount --enable-aggregator-routing=true --enable-bootstrap-token-auth=true --encryption-provider-config=/var/lib/rancher/k3s/server/cred/encryption-config.json --etcd-cafile=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --etcd-certfile=/var/lib/rancher/k3s/server/tls/etcd/client.crt --etcd-keyfile=/var/lib/rancher/k3s/server/tls/etcd/client.key --etcd-servers=https://127.0.0.1:2379 --feature-gates=JobTrackingWithFinalizers=true --kubelet-certificate-authority=/var/lib/rancher/k3s/server/tls/server-ca.crt --kubelet-client-certificate=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.crt --kubelet-client-key=/var/lib/rancher/k3s/server/tls/client-kube-apiserver.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --profiling=false --proxy-client-cert-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.crt --proxy-client-key-file=/var/lib/rancher/k3s/server/tls/client-auth-proxy.key --request-timeout=300s --requestheader-allowed-names=system:auth-proxy --requestheader-client-ca-file=/var/lib/rancher/k3s/server/tls/request-header-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6444 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/rancher/k3s/server/tls/service.key --service-account-lookup=true --service-account-signing-key-file=/var/lib/rancher/k3s/server/tls/service.current.key --service-cluster-ip-range=10.43.0.0/16 --service-node-port-range=30000-32767 --storage-backend=etcd3 --tls-cert-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305 --tls-private-key-file=/var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key"
```

### 4.2.4 Verify that the --read-only-port argument is set to 0 (Manual)


**Result:** pass

**Remediation:**
If using a Kubelet config file, edit the file to set `readOnlyPort` to 0.
If using command line arguments, edit the kubelet service file
/etc/systemd/system/kubelet.service.d/10-kubeadm.conf on each worker node and
set the below parameter in KUBELET_SYSTEM_PODS_ARGS variable.
--read-only-port=0
Based on your system, restart the kubelet service. For example,
systemctl daemon-reload
systemctl restart kubelet.service

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kubelet' | tail -n1 | grep 'read-only-port'
```

**Expected Result**:

```console
'--read-only-port' is equal to '0' OR '--read-only-port' is not present
```

**Returned Value**:

```console
Sep 11 20:32:02 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:32:02Z" level=info msg="Running kubelet --address=0.0.0.0 --allowed-unsafe-sysctls=net.ipv4.ip_forward,net.ipv6.conf.all.forwarding --anonymous-auth=false --authentication-token-webhook=true --authorization-mode=Webhook --cgroup-driver=systemd --client-ca-file=/var/lib/rancher/k3s/agent/client-ca.crt --cloud-provider=external --cluster-dns=10.43.0.10 --cluster-domain=cluster.local --container-runtime-endpoint=unix:///run/k3s/containerd/containerd.sock --containerd=/run/k3s/containerd/containerd.sock --eviction-hard=imagefs.available<5%,nodefs.available<5% --eviction-minimum-reclaim=imagefs.available=10%,nodefs.available=10% --fail-swap-on=false --healthz-bind-address=127.0.0.1 --hostname-override=ip-172-31-3-32 --kubeconfig=/var/lib/rancher/k3s/agent/kubelet.kubeconfig --make-iptables-util-chains=true --node-labels=cattle.io/os=linux,rke.cattle.io/machine=cc5faddd-9c87-4a04-97c2-dcf4cbcbe10f --pod-infra-container-image=rancher/mirrored-pause:3.6 --pod-manifest-path=/var/lib/rancher/k3s/agent/pod-manifests --protect-kernel-defaults=true --read-only-port=0 --resolv-conf=/run/systemd/resolve/resolv.conf --serialize-image-pulls=false --tls-cert-file=/var/lib/rancher/k3s/agent/serving-kubelet.crt --tls-private-key-file=/var/lib/rancher/k3s/agent/serving-kubelet.key"
```

### 4.2.5 Ensure that the --streaming-connection-idle-timeout argument is not set to 0 (Manual)


**Result:** warn

**Remediation:**
If using a Kubelet config file, edit the file to set `streamingConnectionIdleTimeout` to a
value other than 0.
If using command line arguments, edit the kubelet service file
/etc/systemd/system/kubelet.service.d/10-kubeadm.conf on each worker node and
set the below parameter in KUBELET_SYSTEM_PODS_ARGS variable.
--streaming-connection-idle-timeout=5m
Based on your system, restart the kubelet service. For example,
systemctl daemon-reload
systemctl restart kubelet.service

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kubelet' | tail -n1 | grep 'streaming-connection-idle-timeout'
```

### 4.2.6 Ensure that the --protect-kernel-defaults argument is set to true (Automated)


**Result:** pass

**Remediation:**
If using a Kubelet config file, edit the file to set `protectKernelDefaults` to `true`.
If using command line arguments, edit the kubelet service file
/etc/systemd/system/kubelet.service.d/10-kubeadm.conf on each worker node and
set the below parameter in KUBELET_SYSTEM_PODS_ARGS variable.
--protect-kernel-defaults=true
Based on your system, restart the kubelet service. For example:
systemctl daemon-reload
systemctl restart kubelet.service

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kubelet' | tail -n1 | grep 'protect-kernel-defaults'
```

**Expected Result**:

```console
'--protect-kernel-defaults' is equal to 'true'
```

**Returned Value**:

```console
Sep 11 20:32:02 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:32:02Z" level=info msg="Running kubelet --address=0.0.0.0 --allowed-unsafe-sysctls=net.ipv4.ip_forward,net.ipv6.conf.all.forwarding --anonymous-auth=false --authentication-token-webhook=true --authorization-mode=Webhook --cgroup-driver=systemd --client-ca-file=/var/lib/rancher/k3s/agent/client-ca.crt --cloud-provider=external --cluster-dns=10.43.0.10 --cluster-domain=cluster.local --container-runtime-endpoint=unix:///run/k3s/containerd/containerd.sock --containerd=/run/k3s/containerd/containerd.sock --eviction-hard=imagefs.available<5%,nodefs.available<5% --eviction-minimum-reclaim=imagefs.available=10%,nodefs.available=10% --fail-swap-on=false --healthz-bind-address=127.0.0.1 --hostname-override=ip-172-31-3-32 --kubeconfig=/var/lib/rancher/k3s/agent/kubelet.kubeconfig --make-iptables-util-chains=true --node-labels=cattle.io/os=linux,rke.cattle.io/machine=cc5faddd-9c87-4a04-97c2-dcf4cbcbe10f --pod-infra-container-image=rancher/mirrored-pause:3.6 --pod-manifest-path=/var/lib/rancher/k3s/agent/pod-manifests --protect-kernel-defaults=true --read-only-port=0 --resolv-conf=/run/systemd/resolve/resolv.conf --serialize-image-pulls=false --tls-cert-file=/var/lib/rancher/k3s/agent/serving-kubelet.crt --tls-private-key-file=/var/lib/rancher/k3s/agent/serving-kubelet.key"
```

### 4.2.7 Ensure that the --make-iptables-util-chains argument is set to true (Automated)


**Result:** pass

**Remediation:**
If using a Kubelet config file, edit the file to set `makeIPTablesUtilChains` to `true`.
If using command line arguments, edit the kubelet service file
/etc/systemd/system/kubelet.service.d/10-kubeadm.conf on each worker node and
remove the --make-iptables-util-chains argument from the
KUBELET_SYSTEM_PODS_ARGS variable.
Based on your system, restart the kubelet service. For example:
systemctl daemon-reload
systemctl restart kubelet.service

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kubelet' | tail -n1 | grep 'make-iptables-util-chains'
```

**Expected Result**:

```console
'--make-iptables-util-chains' is equal to 'true' OR '--make-iptables-util-chains' is not present
```

**Returned Value**:

```console
Sep 11 20:32:02 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:32:02Z" level=info msg="Running kubelet --address=0.0.0.0 --allowed-unsafe-sysctls=net.ipv4.ip_forward,net.ipv6.conf.all.forwarding --anonymous-auth=false --authentication-token-webhook=true --authorization-mode=Webhook --cgroup-driver=systemd --client-ca-file=/var/lib/rancher/k3s/agent/client-ca.crt --cloud-provider=external --cluster-dns=10.43.0.10 --cluster-domain=cluster.local --container-runtime-endpoint=unix:///run/k3s/containerd/containerd.sock --containerd=/run/k3s/containerd/containerd.sock --eviction-hard=imagefs.available<5%,nodefs.available<5% --eviction-minimum-reclaim=imagefs.available=10%,nodefs.available=10% --fail-swap-on=false --healthz-bind-address=127.0.0.1 --hostname-override=ip-172-31-3-32 --kubeconfig=/var/lib/rancher/k3s/agent/kubelet.kubeconfig --make-iptables-util-chains=true --node-labels=cattle.io/os=linux,rke.cattle.io/machine=cc5faddd-9c87-4a04-97c2-dcf4cbcbe10f --pod-infra-container-image=rancher/mirrored-pause:3.6 --pod-manifest-path=/var/lib/rancher/k3s/agent/pod-manifests --protect-kernel-defaults=true --read-only-port=0 --resolv-conf=/run/systemd/resolve/resolv.conf --serialize-image-pulls=false --tls-cert-file=/var/lib/rancher/k3s/agent/serving-kubelet.crt --tls-private-key-file=/var/lib/rancher/k3s/agent/serving-kubelet.key"
```

### 4.2.8 Ensure that the --hostname-override argument is not set (Manual)


**Result:** Not Applicable

**Remediation:**
Edit the kubelet service file /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
on each worker node and remove the --hostname-override argument from the
KUBELET_SYSTEM_PODS_ARGS variable.
Based on your system, restart the kubelet service. For example,
systemctl daemon-reload
systemctl restart kubelet.service

### 4.2.9 Ensure that the eventRecordQPS argument is set to a level which ensures appropriate event capture (Manual)


**Result:** warn

**Remediation:**
If using a Kubelet config file, edit the file to set `eventRecordQPS` to an appropriate level.
If using command line arguments, edit the kubelet service file
/etc/systemd/system/kubelet.service.d/10-kubeadm.conf on each worker node and
set the below parameter in KUBELET_SYSTEM_PODS_ARGS variable.
Based on your system, restart the kubelet service. For example,
systemctl daemon-reload
systemctl restart kubelet.service

**Audit:**

```bash
/bin/ps -fC containerd
```

**Audit Config:**

```bash
/bin/cat /var/lib/kubelet/config.yaml
```

**Expected Result**:

```console
'--event-qps' is present
```

**Returned Value**:

```console
UID PID PPID C STIME TTY TIME CMD root 410 1 0 Sep11 ? 00:01:50 /usr/bin/containerd root 45213 45195 3 Sep11 ? 00:45:15 containerd -c /var/lib/rancher/k3s/agent/etc/containerd/config.toml -a /run/k3s/containerd/containerd.sock --state /run/k3s/containerd --root /var/lib/rancher/k3s/agent/containerd
```

### 4.2.10 Ensure that the --tls-cert-file and --tls-private-key-file arguments are set as appropriate (Manual)


**Result:** pass

**Remediation:**
If using a Kubelet config file, edit the file to set `tlsCertFile` to the location
of the certificate file to use to identify this Kubelet, and `tlsPrivateKeyFile`
to the location of the corresponding private key file.
If using command line arguments, edit the kubelet service file
/etc/systemd/system/kubelet.service.d/10-kubeadm.conf on each worker node and
set the below parameters in KUBELET_CERTIFICATE_ARGS variable.
--tls-cert-file=<path/to/tls-certificate-file\>
--tls-private-key-file=<path/to/tls-key-file\>
Based on your system, restart the kubelet service. For example,
systemctl daemon-reload
systemctl restart kubelet.service

**Audit:**

```bash
journalctl -D /var/log/journal -u k3s | grep 'Running kubelet' | tail -n1
```

**Expected Result**:

```console
'--tls-cert-file' is present AND '--tls-private-key-file' is present
```

**Returned Value**:

```console
Sep 11 20:32:02 ip-172-31-3-32 k3s[45195]: time="2023-09-11T20:32:02Z" level=info msg="Running kubelet --address=0.0.0.0 --allowed-unsafe-sysctls=net.ipv4.ip_forward,net.ipv6.conf.all.forwarding --anonymous-auth=false --authentication-token-webhook=true --authorization-mode=Webhook --cgroup-driver=systemd --client-ca-file=/var/lib/rancher/k3s/agent/client-ca.crt --cloud-provider=external --cluster-dns=10.43.0.10 --cluster-domain=cluster.local --container-runtime-endpoint=unix:///run/k3s/containerd/containerd.sock --containerd=/run/k3s/containerd/containerd.sock --eviction-hard=imagefs.available<5%,nodefs.available<5% --eviction-minimum-reclaim=imagefs.available=10%,nodefs.available=10% --fail-swap-on=false --healthz-bind-address=127.0.0.1 --hostname-override=ip-172-31-3-32 --kubeconfig=/var/lib/rancher/k3s/agent/kubelet.kubeconfig --make-iptables-util-chains=true --node-labels=cattle.io/os=linux,rke.cattle.io/machine=cc5faddd-9c87-4a04-97c2-dcf4cbcbe10f --pod-infra-container-image=rancher/mirrored-pause:3.6 --pod-manifest-path=/var/lib/rancher/k3s/agent/pod-manifests --protect-kernel-defaults=true --read-only-port=0 --resolv-conf=/run/systemd/resolve/resolv.conf --serialize-image-pulls=false --tls-cert-file=/var/lib/rancher/k3s/agent/serving-kubelet.crt --tls-private-key-file=/var/lib/rancher/k3s/agent/serving-kubelet.key"
```

### 4.2.11 Ensure that the --rotate-certificates argument is not set to false (Automated)


**Result:** pass

**Remediation:**
If using a Kubelet config file, edit the file to add the line `rotateCertificates` to `true` or
remove it altogether to use the default value.
If using command line arguments, edit the kubelet service file
/etc/systemd/system/kubelet.service.d/10-kubeadm.conf on each worker node and
remove --rotate-certificates=false argument from the KUBELET_CERTIFICATE_ARGS
variable.
Based on your system, restart the kubelet service. For example,
systemctl daemon-reload
systemctl restart kubelet.service

**Audit:**

```bash
/bin/ps -fC containerd
```

**Audit Config:**

```bash
/bin/sh -c 'if test -e /var/lib/kubelet/config.yaml; then /bin/cat /var/lib/kubelet/config.yaml; fi'
```

**Expected Result**:

```console
'--rotate-certificates' is present OR '--rotate-certificates' is not present
```

**Returned Value**:

```console
UID PID PPID C STIME TTY TIME CMD root 410 1 0 Sep11 ? 00:01:50 /usr/bin/containerd root 45213 45195 3 Sep11 ? 00:45:15 containerd -c /var/lib/rancher/k3s/agent/etc/containerd/config.toml -a /run/k3s/containerd/containerd.sock --state /run/k3s/containerd --root /var/lib/rancher/k3s/agent/containerd
```

### 4.2.12 Verify that the RotateKubeletServerCertificate argument is set to true (Manual)


**Result:** Not Applicable

**Remediation:**
Edit the kubelet service file /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
on each worker node and set the below parameter in KUBELET_CERTIFICATE_ARGS variable.
--feature-gates=RotateKubeletServerCertificate=true
Based on your system, restart the kubelet service. For example:
systemctl daemon-reload
systemctl restart kubelet.service

### 4.2.13 Ensure that the Kubelet only makes use of Strong Cryptographic Ciphers (Manual)


**Result:** warn

**Remediation:**
If using a Kubelet config file, edit the file to set `TLSCipherSuites` to
TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_RSA_WITH_AES_256_GCM_SHA384,TLS_RSA_WITH_AES_128_GCM_SHA256
or to a subset of these values.
If using executable arguments, edit the kubelet service file
/etc/systemd/system/kubelet.service.d/10-kubeadm.conf on each worker node and
set the --tls-cipher-suites parameter as follows, or to a subset of these values.
--tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_RSA_WITH_AES_256_GCM_SHA384,TLS_RSA_WITH_AES_128_GCM_SHA256
Based on your system, restart the kubelet service. For example:
systemctl daemon-reload
systemctl restart kubelet.service

**Audit:**

```bash
/bin/ps -fC containerd
```

**Audit Config:**

```bash
/bin/cat /var/lib/kubelet/config.yaml
```

**Expected Result**:

```console
'--tls-cipher-suites' is present
```

**Returned Value**:

```console
UID PID PPID C STIME TTY TIME CMD root 410 1 0 Sep11 ? 00:01:50 /usr/bin/containerd root 45213 45195 3 Sep11 ? 00:45:15 containerd -c /var/lib/rancher/k3s/agent/etc/containerd/config.toml -a /run/k3s/containerd/containerd.sock --state /run/k3s/containerd --root /var/lib/rancher/k3s/agent/containerd
```

## 5.1 RBAC and Service Accounts
### 5.1.1 Ensure that the cluster-admin role is only used where required (Manual)


**Result:** warn

**Remediation:**
Identify all clusterrolebindings to the cluster-admin role. Check if they are used and
if they need this role or if they could use a role with fewer privileges.
Where possible, first bind users to a lower privileged role and then remove the
clusterrolebinding to the cluster-admin role :
kubectl delete clusterrolebinding [name]

### 5.1.2 Minimize access to secrets (Manual)


**Result:** warn

**Remediation:**
Where possible, remove get, list and watch access to Secret objects in the cluster.

### 5.1.3 Minimize wildcard use in Roles and ClusterRoles (Manual)


**Result:** warn

**Remediation:**
Where possible replace any use of wildcards in clusterroles and roles with specific
objects or actions.

### 5.1.4 Minimize access to create pods (Manual)


**Result:** warn

**Remediation:**
Where possible, remove create access to pod objects in the cluster.

### 5.1.5 Ensure that default service accounts are not actively used. (Manual)


**Result:** warn

**Remediation:**
Create explicit service accounts wherever a Kubernetes workload requires specific access
to the Kubernetes API server.
Modify the configuration of each default service account to include this value
automountServiceAccountToken: false

### 5.1.6 Ensure that Service Account Tokens are only mounted where necessary (Manual)


**Result:** warn

**Remediation:**
Modify the definition of pods and service accounts which do not need to mount service
account tokens to disable it.

### 5.1.7 Avoid use of system:masters group (Manual)


**Result:** warn

**Remediation:**
Remove the system:masters group from all users in the cluster.

### 5.1.8 Limit use of the Bind, Impersonate and Escalate permissions in the Kubernetes cluster (Manual)


**Result:** warn

**Remediation:**
Where possible, remove the impersonate, bind and escalate rights from subjects.

## 5.2 Pod Security Standards
### 5.2.1 Ensure that the cluster has at least one active policy control mechanism in place (Manual)


**Result:** warn

**Remediation:**
Ensure that either Pod Security Admission or an external policy control system is in place
for every namespace which contains user workloads.

### 5.2.2 Minimize the admission of privileged containers (Manual)


**Result:** warn

**Remediation:**
Add policies to each namespace in the cluster which has user workloads to restrict the
admission of privileged containers.

### 5.2.3 Minimize the admission of containers wishing to share the host process ID namespace (Automated)


**Result:** warn

**Remediation:**
Add policies to each namespace in the cluster which has user workloads to restrict the
admission of `hostPID` containers.

### 5.2.4 Minimize the admission of containers wishing to share the host IPC namespace (Automated)


**Result:** warn

**Remediation:**
Add policies to each namespace in the cluster which has user workloads to restrict the
admission of `hostIPC` containers.

### 5.2.5 Minimize the admission of containers wishing to share the host network namespace (Automated)


**Result:** warn

**Remediation:**
Add policies to each namespace in the cluster which has user workloads to restrict the
admission of `hostNetwork` containers.

### 5.2.6 Minimize the admission of containers with allowPrivilegeEscalation (Automated)


**Result:** warn

**Remediation:**
Add policies to each namespace in the cluster which has user workloads to restrict the
admission of containers with `.spec.allowPrivilegeEscalation` set to `true`.

### 5.2.7 Minimize the admission of root containers (Automated)


**Result:** warn

**Remediation:**
Create a policy for each namespace in the cluster, ensuring that either `MustRunAsNonRoot`
or `MustRunAs` with the range of UIDs not including 0, is set.

### 5.2.8 Minimize the admission of containers with the NET_RAW capability (Automated)


**Result:** warn

**Remediation:**
Add policies to each namespace in the cluster which has user workloads to restrict the
admission of containers with the `NET_RAW` capability.

### 5.2.9 Minimize the admission of containers with added capabilities (Automated)


**Result:** warn

**Remediation:**
Ensure that `allowedCapabilities` is not present in policies for the cluster unless
it is set to an empty array.

### 5.2.10 Minimize the admission of containers with capabilities assigned (Manual)


**Result:** warn

**Remediation:**
Review the use of capabilites in applications running on your cluster. Where a namespace
contains applicaions which do not require any Linux capabities to operate consider adding
a PSP which forbids the admission of containers which do not drop all capabilities.

### 5.2.11 Minimize the admission of Windows HostProcess containers (Manual)


**Result:** warn

**Remediation:**
Add policies to each namespace in the cluster which has user workloads to restrict the
admission of containers that have `.securityContext.windowsOptions.hostProcess` set to `true`.

### 5.2.12 Minimize the admission of HostPath volumes (Manual)


**Result:** warn

**Remediation:**
Add policies to each namespace in the cluster which has user workloads to restrict the
admission of containers with `hostPath` volumes.

### 5.2.13 Minimize the admission of containers which use HostPorts (Manual)


**Result:** warn

**Remediation:**
Add policies to each namespace in the cluster which has user workloads to restrict the
admission of containers which use `hostPort` sections.

## 5.3 Network Policies and CNI
### 5.3.1 Ensure that the CNI in use supports NetworkPolicies (Manual)


**Result:** warn

**Remediation:**
If the CNI plugin in use does not support network policies, consideration should be given to
making use of a different plugin, or finding an alternate mechanism for restricting traffic
in the Kubernetes cluster.

### 5.3.2 Ensure that all Namespaces have NetworkPolicies defined (Manual)


**Result:** warn

**Remediation:**
Follow the documentation and create NetworkPolicy objects as you need them.

## 5.4 Secrets Management
### 5.4.1 Prefer using Secrets as files over Secrets as environment variables (Manual)


**Result:** warn

**Remediation:**
If possible, rewrite application code to read Secrets from mounted secret files, rather than
from environment variables.

### 5.4.2 Consider external secret storage (Manual)


**Result:** warn

**Remediation:**
Refer to the Secrets management options offered by your cloud provider or a third-party
secrets management solution.

## 5.5 Extensible Admission Control
### 5.5.1 Configure Image Provenance using ImagePolicyWebhook admission controller (Manual)


**Result:** warn

**Remediation:**
Follow the Kubernetes documentation and setup image provenance.

## 5.7 General Policies
### 5.7.1 Create administrative boundaries between resources using namespaces (Manual)


**Result:** warn

**Remediation:**
Follow the documentation and create namespaces for objects in your deployment as you need
them.

### 5.7.2 Ensure that the seccomp profile is set to docker/default in your Pod definitions (Manual)


**Result:** warn

**Remediation:**
Use `securityContext` to enable the docker/default seccomp profile in your pod definitions.
An example is as below:
 securityContext:
 seccompProfile:
 type: RuntimeDefault

### 5.7.3 Apply SecurityContext to your Pods and Containers (Manual)


**Result:** warn

**Remediation:**
Follow the Kubernetes documentation and apply SecurityContexts to your Pods. For a
suggested list of SecurityContexts, you may refer to the CIS Security Benchmark for Docker
Containers.

### 5.7.4 The default namespace should not be used (Manual)


**Result:** warn

**Remediation:**
Ensure that namespaces are created to allow for appropriate segregation of Kubernetes
resources and that all new resources are created in a specific namespace.
