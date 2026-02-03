# k8s-architecture-lab
Lab Pratico FIAP: Arquitetura Kubernetes

# 🚀 Lab: Arquitetura Kubernetes & Estratégias Cloud (FIAP)
Este repositório contém o material prático e o guia de execução para a live de Arquitetura Cloud e DevOps da Pós Tech FIAP. O laboratório foi otimizado para execução dentro do ambiente AWS Academy (Learner Lab).

🏗️ A Arquitetura do Lab O objetivo é demonstrar o provisionamento de um cluster gerenciado (Amazon EKS), a integração com a rede da AWS via VPC CNI e as capacidades de Self-healing e Escalabilidade do Kubernetes.

🛠️ Passo a Passo para Execução (AWS CloudShell)

# 1. Preparação e Clone do Projeto
Abra o AWS CloudShell no console da AWS. Primeiro, vamos instalar as ferramentas necessárias e baixar este repositório:

Instalar o eksctl

curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
Clonar o repositório do laboratório
Bash

git clone https://github.com/lowreche/k8s-architecture-lab.git
cd k8s-architecture-lab

# 2. Provisionamento do Cluster (Ajuste para AWS Academy)
Devido às restrições de permissão do IAM no Academy, utilizaremos um arquivo de configuração para reaproveitar a LabRole existente.

Crie o arquivo cluster.yaml:

YAML: (Substitua o número da conta abaixo pelo numero da sua conta do AWS Academy)

cat <<EOF > cluster.yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig
metadata:
  name: fiap-lab-architecture
  region: us-east-1
  version: "1.31"
iam:
  serviceRoleARN: "arn:aws:iam::924877704926:role/LabRole"
managedNodeGroups:
  - name: standard-nodes
    instanceType: t3.medium
    minSize: 2
    maxSize: 2
    desiredCapacity: 2
    iam:
      instanceRoleARN: "arn:aws:iam::924877704926:role/LabRole"
EOF

# Execute a criação:

eksctl create cluster -f cluster.yaml

# 3. Deploy da Aplicação e Ajuste de Rede
Após o cluster estar no status READY, vamos aplicar nosso manifesto do Nginx e liberar o acesso externo.

Aplicar Manifesto:

kubectl apply -f lab/nginx-lab.yaml

Liberar porta 80 no Security Group dos Nodes

SG_ID=$(aws ec2 describe-instances --filters "Name=tag:eks:nodegroup-name,Values=standard-nodes" --query "Reservations[0].Instances[0].SecurityGroups[0].GroupId" --output text)
aws ec2 authorize-security-group-ingress --group-id $SG_ID --protocol tcp --port 80 --cidr 0.0.0.0/0

# 🧪 Validando a Operação 🌐 Acesso Externo Para obter a URL pública da sua aplicação:

kubectl get svc nginx-service
Copie o endereço em EXTERNAL-IP e cole no navegador (utilize http://).

🩹 Self-Healing (Resiliência) 

Delete um Pod e veja o Kubernetes recriá-lo em segundos para manter o estado desejado:

kubectl get pods
kubectl delete pod [NOME_DO_POD]
kubectl get pods -w

📈 Escala Horizontal Simule uma alta demanda escalando as réplicas:

kubectl scale deployment nginx-deployment --replicas=10
kubectl get pods

# ❓ FAQ de Troubleshooting:
Erro 403 Forbidden no navegador? Você provavelmente acessou o IP do API Server (Porta 443) em vez do LoadBalancer (Porta 80). Verifique o kubectl get svc.

Time-out ao acessar a URL? Certifique-se de que executou o comando de authorize-security-group-ingress.

O que é o CNI? No EKS, é o AWS VPC CNI, que permite que cada Pod tenha um IP real da sua VPC, facilitando a comunicação com outros serviços AWS.
