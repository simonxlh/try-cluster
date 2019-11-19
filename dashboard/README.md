部署在master上，参考 https://blog.csdn.net/tiger435/article/details/73650147

kubectl apply -f kubernetes-dashboard.yaml

k8s 1.16.*使用 recommend部署
https://github.com/kubernetes/dashboard/issues/4410#issuecomment-541316837
kubectl apply -f recommend.yaml
kubectl create -f dashboard-admin.yaml

## get token key
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')


eyJhbGciOiJSUzI1NiIsImtpZCI6ImZXdE54OGhCZVRaY1NOTHF6V1ZnejZaaVB3TUllV0kzZzVyQ3VvemI3d0EifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLWJiYnI5Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIyNjUxY2U4NC1mNmY2LTQwZGItYThmNi1kMzNlZmJmNzlmZWEiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.DgWvPpCflfhE6wJTx9b2Ymb3sgXCHDOFd0TleSuueVBuCSz4CgLIfdo7uwC32k4sGjoIapUIIsyTFezib4V9g2ZhW6i3iteSkNNezO7C9DypUMmoc2NdBDgPE-Htl5s80udll_aup0C_e8Q8Dl5yYdoxxQNKULBLLm3uR-o0BvnDS3xtCslaVSEON7vyTfIx8-R9-e8IAK03t5idJkSu1oSPpfheEoSTIju3X_0mBQ8KSunvFmjOflVSeDYstJZP7T52w0eqxEBsYF5rJsU9W8hibn0s29G7xTiotzCVD39TXyjqRDNpt3PMNKZTAxv6FKdu5pDmfIe00WfVV2D0FA

## get kubeconfig
http://www.simlinux.com/2017/09/07/k8s-cfssl-install-cert.html
https://jimmysong.io/kubernetes-handbook/practice/create-tls-and-secret-key.html

install  CFSSL
curl -s -L -o /bin/cfssl https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
curl -s -L -o /bin/cfssljson https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
curl -s -L -o /bin/cfssl-certinfo https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
chmod +x /bin/cfssl*

mkdir /opt/ssl
cd /opt/ssl
cfssl print-defaults config > ca-config.json
cfssl print-defaults csr > ca-csr.json


cat > kubernetes-csr.json <<EOF
{
    "CN": "kubernetes",
    "hosts": [
      "127.0.0.1",
      "192.168.56.101",
      "192.168.56.104",
      "192.168.56.102",
      "10.96.0.1",
      "kubernetes",
      "kubernetes.default",
      "kubernetes.default.svc",
      "kubernetes.default.svc.cluster",
      "kubernetes.default.svc.cluster.local"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "BeiJing",
            "L": "BeiJing",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
EOF


cat > admin-csr.json <<EOF
{
  "CN": "admin",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "system:masters",
      "OU": "System"
    }
  ]
}
EOF

cat > kube-proxy-csr.json <<EOF
{
  "CN": "system:kube-proxy",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
EOF


### 创建kubeconfig文件 -test push 

 kubectl config set-cluster dashboard --certificate-authority=/etc/kubernetes/pki/ca.crt --server="https://192.168.56.101:6443" --embed-certs=true --kubeconfig=/root/def-ns-dashboard.conf

kubectl config set-credentials def-ns-dashboard-user --kubeconfig=/root/def-ns-dashboard.conf --token=eyJhbGciOiJSUzI1NiIsImtpZCI6ImZXdE54OGhCZVRaY1NOTHF6V1ZnejZaaVB3TUllV0kzZzVyQ3VvemI3d0EifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLWJiYnI5Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIyNjUxY2U4NC1mNmY2LTQwZGItYThmNi1kMzNlZmJmNzlmZWEiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.DgWvPpCflfhE6wJTx9b2Ymb3sgXCHDOFd0TleSuueVBuCSz4CgLIfdo7uwC32k4sGjoIapUIIsyTFezib4V9g2ZhW6i3iteSkNNezO7C9DypUMmoc2NdBDgPE-Htl5s80udll_aup0C_e8Q8Dl5yYdoxxQNKULBLLm3uR-o0BvnDS3xtCslaVSEON7vyTfIx8-R9-e8IAK03t5idJkSu1oSPpfheEoSTIju3X_0mBQ8KSunvFmjOflVSeDYstJZP7T52w0eqxEBsYF5rJsU9W8hibn0s29G7xTiotzCVD39TXyjqRDNpt3PMNKZTAxv6FKdu5pDmfIe00WfVV2D0FA

kubectl config set-context def-ns-dashboard-user@dashboard --cluster=dashboard --user=def-ns-dashboard-user --kubeconfig=/root/def-ns-dashboard.conf



#### delete namespace stay terminating soon
kubectl get namespace $(namespace) -o json > tmp.json
vi tmp.json  (then delete spec content)
open new shell then
kubectl proxy
curl -k -H "Content-Type: application/json" -X PUT --data-binary @tmp.json http://127.0.0.1:8001/api/v1/namespaces/$(namespace)/finalize


