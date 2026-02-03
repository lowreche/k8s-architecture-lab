# k8s-architecture-lab
Lab Pratico FIAP: Arquitetura Kubernetes

# 🚀 Laboratório Prático: Explorando a Arquitetura Kubernetes (EKS & Kubeadm)
Este laboratório foi desenhado para a disciplina de Cloud Architecture & DevOps da Pós-Tech FIAP. O objetivo é demonstrar na prática como os componentes do Control Plane e dos Workers interagem, utilizando a experiência real em ambientes críticos como AWS EKS e Azure AKS.

📋 Pré-requisitos
Conta no AWS Academy ou acesso a um console AWS.

AWS CLI e kubectl configurados.

Lens IDE (opcional, mas recomendado para troubleshooting visual).

# 🏗️ Passo 1: Provisionamento do Cluster (EKS)
Para ganhar tempo na aula de 2 horas, utilizaremos o eksctl para criar um cluster gerenciado.

# Criação do cluster com 2 nodes workers t3.medium
eksctl create cluster \
  --name fiap-lab-architecture \
  --region us-east-1 \
  --nodegroup-name standard-nodes \
  --node-type t3.medium \
  --nodes 2 \
  --with-oidc
  
Dica do Professor: Em cenários corporativos de alta segurança, como os que gerenciei na PwC, utilizaríamos VPC Endpoints para garantir que o tráfego do Control Plane não trafegue pela internet pública.

# 🔍 Passo 2: Investigando os Componentes (O "Cérebro")
O Kubernetes trabalha para manter o Desired State. Vamos verificar quem está no controle.

# Verificando os componentes do Control Plane (Namespace: kube-system)
kubectl get pods -n kube-system
O que observar:

CoreDNS: Responsável pela resolução de nomes interna (ex: http://user-service).

kube-proxy: Verifique se ele está rodando em todos os nós; ele gerencia as regras de rede (Iptables/IPVS).

AWS Node (CNI): O plugin de rede que atribui IPs da VPC diretamente aos Pods.

# 🚢 Passo 3: Deploy e a "Jornada da Requisição"
Vamos subir uma aplicação Nginx e observar o fluxo: Ingress -> Service -> Pod.

YAML:

# nginx-lab.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP  # Virtual IP (VIP) interno
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      
Aplique o arquivo:

kubectl apply -f nginx-lab.yaml

# 🛠️ Passo 4: Troubleshooting e Auto-healing

Simularemos uma falha para ver o Controller Manager e o Scheduler em ação.

Abra o Lens para monitorar os logs em tempo real (técnica que utilizei para destravar pipelines críticas).

Delete um Pod manualmente:

kubectl delete pod [NOME_DO_POD]
Observe: O Kubernetes detecta que o estado atual (2 pods) é diferente do desejado (3 pods) e cria um novo instantaneamente.

# ⚖️ Passo 5: Escalabilidade e Add-ons
Para habilitar o monitoramento de recursos, precisamos de um Add-on.

# Instalando o Metrics Server (Add-on essencial)
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Verificando o consumo (Pode levar 1 min para coletar)
kubectl top nodes
kubectl top pods
