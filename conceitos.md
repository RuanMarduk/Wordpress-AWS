## 1. VPC (Virtual Private Cloud)

Rede virtual isolada na AWS onde são alocados todos os recursos do projeto.
Permite definir faixas de IP (CIDR), sub-redes, tabelas de rotas, gateways e políticas de segurança.

## 2. CIDR (Classless Inter-Domain Routing)

Notação que define o intervalo de endereços IP de uma rede.
Exemplo: 10.0.0.0/16 → IPs de 10.0.0.0 a 10.0.255.255.

## 3. Subnets

Divisões lógicas da VPC, associadas a zonas de disponibilidade (AZs).

Públicas → com rota para Internet Gateway (usadas pelo Load Balancer).

Privadas → sem rota direta para a Internet, acessam via NAT Gateway (usadas pelas EC2 do WordPress).

## 4. Internet Gateway (IGW)

Dispositivo virtual que conecta a VPC à Internet, permitindo tráfego público de entrada e saída.

## 5. NAT Gateway

Gateway que permite que instâncias em sub-redes privadas acessem a Internet apenas para saída, sem ficarem acessíveis de fora.

## 6. Route Table

Tabela que define o destino do tráfego de rede dentro da VPC, associando sub-redes a gateways ou outros destinos.

## 7. Security Group (SG)

Firewall virtual que controla tráfego de entrada (Inbound) e saída (Outbound) de recursos.
As regras podem ser baseadas em portas, protocolos e origem/destino (IPs ou outros SGs).

## 8. Elastic IP (EIP)

Endereço IP público estático associado a um recurso na AWS (como NAT Gateway).

## 9. Amazon EC2 (Elastic Compute Cloud)

Serviço de máquinas virtuais escaláveis na AWS.
No projeto, usado para hospedar containers Docker do WordPress.

## 10. Launch Template

Modelo de configuração para criação de instâncias EC2, definindo:

AMI (sistema operacional)

Tipo da instância

Security Groups

User data (scripts de inicialização)

## 11. User Data

Script que roda automaticamente na inicialização da EC2, usado para instalar e configurar software.
No projeto, foi usado para:

Instalar docker e amazon-efs-utils

Montar o EFS

Subir o container WordPress

## 12. Amazon EFS (Elastic File System)

Sistema de arquivos compartilhado entre múltiplas instâncias, garantindo persistência e sincronização dos arquivos do WordPress.

## 13. Amazon RDS (Relational Database Service)

Banco de dados gerenciado.
No projeto, usamos MySQL para armazenar dados do WordPress de forma separada das instâncias.

## 14. Docker

Plataforma de containers que empacota aplicações com todas as dependências necessárias.
Aqui, foi usado para rodar o WordPress em EC2.

## 15. Application Load Balancer (ALB)

Balanceador de carga que distribui tráfego HTTP/HTTPS entre várias instâncias, garantindo alta disponibilidade.

## 16. Target Group

Grupo de recursos (EC2, IPs ou Lambda) que recebem tráfego do Load Balancer.
Possui Health Checks para monitorar se os alvos estão saudáveis.

## 17. Health Check

Teste periódico feito pelo Target Group para verificar se a aplicação está respondendo.
Se falhar, o recurso é marcado como Unhealthy e não recebe tráfego.

## 18. Auto Scaling Group (ASG)

Serviço que ajusta automaticamente a quantidade de instâncias EC2 de acordo com demanda ou falha, usando um Launch Template como base.

## 19. Alta Disponibilidade

Arquitetura projetada para minimizar tempo de inatividade, usando múltiplas AZs, balanceamento de carga e auto scaling.

## 20. Fluxo Resumido do Projeto

Usuário acessa o ALB pelo DNS público.

ALB encaminha a requisição para uma instância no Target Group.

EC2 com Docker atende via container WordPress.

Arquivos do WordPress ficam no EFS (compartilhado entre instâncias).

Dados são salvos no RDS MySQL.

ASG garante que sempre haja pelo menos 2 instâncias saudáveis.
