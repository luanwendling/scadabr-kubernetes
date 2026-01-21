# ScadaBR 1.2 no Kubernetes (Production Ready)

Este repositório documenta a execução do **ScadaBR 1.2** em um **cluster Kubernetes**, utilizando boas práticas reais de produção:

- MySQL em StatefulSet
- Persistência com Longhorn
- Configuração via ConfigMaps
- Escala vertical segura
- Readiness & Liveness Probes
- Rolling Update sem downtime

---

## 🧠 Arquitetura

```
Kubernetes Cluster
├── Control Plane
└── Worker Node (sminidash-k8s)
    ├── MySQL 5.7 (StatefulSet + Longhorn)
    └── ScadaBR 1.2 (Deployment + Longhorn)
```

### Componentes
- **ScadaBR 1.2** rodando em Tomcat 9 + JDK 11
- **MySQL 5.7** como banco de dados
- **Longhorn** para volumes persistentes
- **Metrics Server** para métricas
- **ConfigMaps** para configuração da aplicação e JVM

---

## 📁 Estrutura do projeto

```
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
```

---

## 🚀 Deploy do ambiente

### 1️⃣ Criar o namespace
```bash
kubectl apply -f namespace.yaml
```

### 2️⃣ Subir o MySQL
```bash
kubectl apply -f mysql/
```

Verificar:
```bash
kubectl get pods -n scada-minidash
kubectl get pvc -n scada-minidash
```

### 3️⃣ Subir o ScadaBR
```bash
kubectl apply -f scadabr/
```

Acompanhar o rollout:
```bash
kubectl rollout status deployment scadabr -n scada-minidash
```

---

## 🌐 Acesso ao ScadaBR

Usando **NodePort**:

```
http://<IP_DO_NODE>:30080/ScadaBR
```

Credenciais padrão:
- **Usuário:** `admin`
- **Senha:** `admin`

---

## ⚙️ Configurações importantes

### 🔹 env.properties
Gerenciado via ConfigMap:

```
scadabr-env-configmap.yaml
```

Exemplos de ajustes:
```properties
main.maxthreadlimit=300
db.pool.maxActive=10
```

> Alterações exigem restart do pod.

---

### 🔹 JVM / Tomcat
Gerenciado via ConfigMap:

```
scadabr-jvm-configmap.yaml
```

Configuração aplicada:
```
-Xms512m
-Xmx1024m
-XX:+UseG1GC
```

---

## 🫀 Health Checks

### Readiness Probe
- Endpoint: `/ScadaBR/index.jsp`
- Define quando o pod está pronto para receber tráfego

### Liveness Probe
- Mesmo endpoint
- Reinicia o pod caso o Tomcat trave

Essas probes garantem:
- Inicialização segura
- Rolling update sem downtime
- Auto-recovery

---

## 🔄 Rolling Update (Zero Downtime)

Configurado com:
```yaml
maxUnavailable: 0
maxSurge: 1
```

Fluxo:
1. Novo pod sobe
2. Readiness passa
3. Pod antigo é removido

---

## 📈 Escala vertical (recomendada)

O ScadaBR **não suporta escala horizontal**.

Configuração padrão segura:
```yaml
requests:
  memory: 1Gi
  cpu: 500m
limits:
  memory: 2Gi
  cpu: 2
```

Aumentar conforme:
- número de gráficos
- data points
- usuários simultâneos

---

## 💾 Persistência de dados

| Componente | Tipo |
|-----------|------|
| MySQL | Longhorn PVC |
| Uploads ScadaBR | Longhorn PVC |

Volumes podem ser **expandidos sem downtime**.

---

## 📊 Observabilidade

Metrics Server habilitado.

Comandos úteis:
```bash
kubectl top pod -n scada-minidash
kubectl top node
```

---

## 📌 Boas práticas aplicadas

- ❌ Sem edição manual dentro do pod
- ✅ Configuração via ConfigMap
- ✅ Dados via PVC
- ✅ Imagem Docker imutável
- ✅ Rollout controlado
- ✅ Restart declarativo

---


## 📄 Licença

Este projeto é apenas para fins educacionais e de infraestrutura.  
O ScadaBR é um software open-source mantido por seus respectivos autores.

