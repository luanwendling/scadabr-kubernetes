# ScadaBR 1.2 no Kubernetes (Production Ready)

Este repositório documenta a migração e execução do **ScadaBR 1.2** em um **cluster Kubernetes**, utilizando boas práticas de produção:

- StatefulSet para MySQL
- Volumes persistentes com Longhorn
- ConfigMaps para configuração
- Escala vertical segura
- Readiness/Liveness Probes
- Rolling Update sem downtime

---

## 🧠 Arquitetura

Kubernetes Cluster
├── Control Plane
└── Worker (sminidash-k8s)
    ├── MySQL 5.7 (StatefulSet + Longhorn)
    └── ScadaBR 1.2 (Deployment + Longhorn)

---

## 📁 Estrutura do projeto

.
├── namespace.yaml
├── mysql
│   ├── mysql-configmap.yaml
│   ├── mysql-service.yaml
│   └── mysql-statefulset.yaml
└── scadabr
    ├── scadabr-deployment.yaml
    ├── scadabr-env-configmap.yaml
    ├── scadabr-jvm-configmap.yaml
    ├── scadabr-pvc.yaml
    └── scadabr-service.yaml

---

## 🚀 Deploy do ambiente

### Criar namespace
kubectl apply -f namespace.yaml

### Subir MySQL
kubectl apply -f mysql/

### Subir ScadaBR
kubectl apply -f scadabr/

---

## 🌐 Acesso

http://<IP_DO_NODE>:30080/ScadaBR

Usuário: admin  
Senha: admin

---

## ⚙️ Configurações

### env.properties
Gerenciado via ConfigMap (`scadabr-env-configmap.yaml`).

### JVM
Gerenciado via ConfigMap (`scadabr-jvm-configmap.yaml`).

---

## 🫀 Health Checks

Readiness e Liveness configurados no endpoint:

/ScadaBR/index.jsp

---

## 🔄 Rolling Update

Configurado com:
- maxUnavailable: 0
- maxSurge: 1

---

## 📈 Escala vertical

ScadaBR escala verticalmente:

requests:
  memory: 1Gi
  cpu: 500m

limits:
  memory: 2Gi
  cpu: 2

---

## 💾 Persistência

- MySQL: Longhorn PVC
- Uploads ScadaBR: Longhorn PVC

---

## 📊 Observabilidade

kubectl top pod -n scada-minidash  
kubectl top node

---

## 🏁 Status

✔ Produção-ready  
✔ Zero downtime  
✔ Persistente  
✔ Escalável verticalmente  

---

## 📄 Licença

Uso educacional e de infraestrutura.  
ScadaBR é open-source mantido por seus autores.

# scadabr-kubernetes
# scadabr-kubernetes
