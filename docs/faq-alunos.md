# ❓ FAQ - Arquitetura, Redes e Troubleshooting Kubernetes
Este guia compila as principais dúvidas técnicas surgidas durante a aula de Arquitetura Cloud e DevOps, integrando conceitos fundamentais com cenários reais de operação em AWS EKS e Azure AKS.

# 1. Redes e Conectividade
O que é um CNI (Container Network Interface)?
O CNI é um padrão e um conjunto de plugins que gerencia a rede dos containers. O Kubernetes, por design, não possui uma rede de Pods nativa; ele delega essa tarefa ao CNI. Ele é responsável por:

Atribuir um endereço IP único para cada Pod.

Garantir que todos os Pods consigam se comunicar entre si sem a necessidade de NAT (Network Address Translation).

Gerenciar a criação de interfaces de rede virtuais (veth pairs) e a propagação de rotas entre os nós.

Como funciona a rede do Kubernetes na AWS (EKS)?
No Amazon EKS, utilizamos geralmente o AWS VPC CNI. Diferente de redes "overlay" (como Flannel), este plugin atribui IPs diretamente das subnets da sua VPC para os Pods.

Integração Nativa: Os Pods são cidadãos de primeira classe na sua rede AWS.

Segurança: Você pode aplicar Security Groups diretamente nos Pods (Security Groups for Pods).

VPC Endpoints: Como vimos em aula, o uso de Endpoints permite que o tráfego entre o seu cluster e outros serviços AWS (como o API Server) ocorra de forma privada, sem exposição à internet, exigindo ajustes finos no CoreDNS.

Posso fazer comunicação entre nuvens via VPC Endpoint?
Não diretamente. O VPC Endpoint (AWS PrivateLink) serve para acessar serviços dentro da rede da AWS de forma privada. Para conectar um cluster EKS (AWS) a um AKS (Azure), as rotas comuns são:

VPN Site-to-Site ou Direct Connect/ExpressRoute: Para unir as redes (VPC e VNet).

API Gateway (Kong): Exposição de serviços via camadas de governança e segurança.

# 2. Limites e Escalabilidade
Quais são os limites de Pods e Nodes?
O limite de Pods por Node não é fixo; ele depende de:

Recursos de Hardware: CPU e Memória RAM disponíveis no Worker Node.

IPs Disponíveis: No caso do EKS, o limite é frequentemente ditado pela quantidade de IPs que o tipo de instância EC2 consegue gerenciar via interfaces de rede (ENI).

Impacto no dia a dia: Exceder esses limites causa o status Pending nos Pods, pois o Scheduler não encontrará espaço para alocá-los.

O que é o Karpenter e ele é exclusivo do EKS?
O Karpenter é um orquestrador de nós "just-in-time".

Funcionamento: Ele observa Pods que não puderam ser agendados e provisiona a instância EC2 exata (tipo e tamanho) necessária, sem depender de Node Groups estáticos.

Exclusividade: Embora seja focado em AWS, é um projeto open-source. No entanto, sua implementação e maturidade são significativamente maiores no ecossistema EKS.

# 3. Troubleshooting e Operação
Como fazer troubleshooting avançado?
Além dos comandos básicos (kubectl logs, describe), o uso de ferramentas visuais é um diferencial:

Lens IDE: Permite analisar logs em tempo real e identificar gargalos de performance ou erros de configuração de forma rápida.

Exemplo Real: Recentemente, corrigi um erro que travava uma pipeline de CI/CD analisando os eventos de erro de permissão (RBAC) diretamente pelo painel do Lens.

Por que desativar o Swap?
O Kubernetes foi desenhado para rodar com Swap desativado para garantir previsibilidade. O Kubelet precisa ter controle total sobre a memória para evitar que o uso de disco (lento) degrade a performance da aplicação e confunda as métricas de limite de memória do cluster.

🎯 Analogia para Memorizar
Nodes vs Pods: Imagine uma frota de caminhões. O Node é o caminhão (a máquina/instância). O Pod é a carga (agrupamento de containers). Você não pode colocar mais carga do que a suspensão do caminhão aguenta (limites de recursos) e cada carga precisa de uma etiqueta de rastreio única (IP via CNI).