# k8s-consul
Deploy consul server cluster in kubernetes

## Prerequrements

'''
 kubernetes 1.12.x, Recommend use 1.13.2,dont use 1.13.0、1.13.1，there was a kube-proxy bug in those version

 consul 1.4.0

 cfssl and cfssljson
'''

## Usage

Clone repo

'''bash
git clone https://github.com/cuishuaigit/k8s-consul.git 

cd k8s-consul 
'''

### Generate TLS Certificates

RPC communication between each Consul member will be encrypted using TLS.

'''bash 
cd ssl

cfssl gencert -initca ca-csr.json | cfssljson -bare ca 

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=default consul-csr.json | cfssljson -bare consul 

'''

### Generate the Consul Gossip Entcryption key 

'''bash
GOSSIP_ENCRYPTION_KEY=$(consul keygen)

'''

### Create Consul Secret 

'''bash
kubectl create secret generic consul \
			--from-literal="gossip-encryption-key=${GOSSIP_ENCRYPTION_KEY}" \
			--from-file=ca.pem \
			--from-file=consul.pem \
			--from-file=consul-key.pem \
			-n consul

'''

### Create namesapce and rbac secret and configmap consul(headless service and statefulset)

'''bash
cd k8s-consul 

kubectl create -f consul-namespace.yaml -f consul-rbac.yaml 

kubectl create secret generic consul \
              --from-literal="gossip-encryption-key=${GOSSIP_ENCRYPTION_KEY}" \
              --from-file=ca.pem \
              --from-file=consul.pem \
              --from-file=consul-key.pem \
              -n consul 

kubctl create -f consul-configmap.yaml 

kubectl create -f consul.yaml
'''

### Vertification

'''bash
kubectl get pods -n consul
kubectl get statefulset -n consul 
'''

### Accessing web UI 

'''
I use ambassador to expose consul ui,but in consul.yaml i have annotated. just like:

  annotations:
    getambassador.io/config: |
      ---
      apiVersion: ambassador/v0
      kind: Mapping
      name: consul-mapping
      prefix: /
      service: consul:8500/ui


hwo to use ambassador you can learn it by https://www.getambassador.io/docs

and you can  learn it by my blog https://www.cnblogs.com/cuishuai/p/9806007.html
'''
