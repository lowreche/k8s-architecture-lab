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

# 📈 Passo 5: Elasticidade com HPA (Horizontal Pod Autoscaler)
Nesta etapa, demonstramos como o Kubernetes escala a aplicação automaticamente com base no consumo de CPU, otimizando performance e custos (FinOps).

1. Configurar limites de recursos (Necessário para o cálculo do HPA)

kubectl patch deployment nginx-deployment -p '{"spec":{"template":{"spec":{"containers":[{"name":"nginx","resources":{"requests":{"cpu":"100m"}}}]}}}}'
2. Criar a regra de Autoscaling (Mínimo 3, Máximo 10 réplicas)

kubectl autoscale deployment nginx-deployment --cpu="50%" --min=3 --max=10
3. Monitorar o escalonamento em tempo real

kubectl get hpa -w

🚀 Simulação de Carga (Stress Test)
Para ver o HPA em ação e as réplicas subindo, abra um novo terminal e execute:

kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while true; do wget -q -O- http://nginx-service; done"

❓ Dica do Professor:
Observe que o Kubernetes leva cerca de 1 a 2 minutos para coletar as métricas iniciais (status <unknown>). Após o teste de carga, o HPA levará alguns minutos para fazer o Scale Down (reduzir para 3 pods), garantindo que a aplicação esteja estável antes de remover recursos.

# ❓ FAQ de Troubleshooting:
Erro 403 Forbidden no navegador? Você provavelmente acessou o IP do API Server (Porta 443) em vez do LoadBalancer (Porta 80). Verifique o kubectl get svc.

Time-out ao acessar a URL? Certifique-se de que executou o comando de authorize-security-group-ingress.

O que é o CNI? No EKS, é o AWS VPC CNI, que permite que cada Pod tenha um IP real da sua VPC, facilitando a comunicação com outros serviços AWS.
