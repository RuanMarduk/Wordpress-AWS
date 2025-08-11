# WordPress HA na AWS com ASG, ALB, RDS, EFS e NAT

## Arquitetura

<p align="center">
  <img width="1054" height="466" alt="image" src="https://github.com/user-attachments/assets/c8e9626c-f443-490c-ba4d-42f177c521c7">
</p>

Infraestrutura projetada para hospedar WordPress com alta disponibilidade:

- **VPC** com 2 sub-redes públicas e 4 privadas
- **Auto Scaling Group** para EC2 rodando Docker + WordPress
- **Application Load Balancer** distribuindo tráfego HTTP/HTTPS
- **RDS MySQL** para dados persistentes
- **EFS** compartilhado para uploads do WordPress
- **NAT Gateway** para acesso outbound de EC2 privadas

---


## 1. Criar VPC e Sub-redes

- 2 sub-redes públicas (ALB + NAT)
- 4 sub-redes privadas (EC2)
- Route tables separadas:
     - Públicas → IGW
     - Privadas → NAT Gateway

### 2. Criar Security Groups

- **SG-ALB**: inbound 80/443 de 0.0.0.0/0; outbound liberado
- **SG-EC2**: inbound 80 do SG-ALB, 22 do bastion; outbound liberado
- **SG-RDS**: inbound 3306 do SG-EC2
- **SG-EFS**: inbound 2049 do SG-EC2

### 3. Criar EFS

- Montado apenas nas privadas necessárias (private-03, private-04)
- Associar SG-EFS
- Teste: `sudo mount -t efs fs-XXXX:/ /mnt/efs`
- Verificar: `df -h | grep efs`

### 4. Criar RDS

- Engine: MySQL 8.0
- DB name: `name`
- Usuário: `admin`
- Acesso privado (sem IP público)
- SG-RDS associado
- Teste: `mysql -h <endpoint> -u admin -p`

### 5. Criar Launch Template

- AMI Amazon Linux 2023
- `user-data` contendo instalação do Docker, configuração do EFS e variáveis do WordPress:

```bash
yum update -y
yum install -y docker amazon-efs-utils
systemctl enable --now docker
mkdir -p /mnt/efs
echo "fs-XXXX.efs.us-east-2.amazonaws.com:/ /mnt/efs efs _netdev,tls 0 0" >> /etc/fstab
mount -a
docker run -d --name wordpress -p 80:80 \
  -e WORDPRESS_DB_HOST=<endpoint-rds> \
  -e WORDPRESS_DB_NAME=name \
  -e WORDPRESS_DB_USER=admin \
  -e WORDPRESS_DB_PASSWORD=<senha> \
  -v /mnt/efs:/var/www/html \
  wordpress
```

### 6. Criar Auto Scaling Group

- Sub-redes privadas
- Target Group vinculado ao ALB

### 7. Criar ALB

- Listener HTTP (80) → Target Group
- Health Check para `/` com códigos `200,302`

### 8. Criar NAT Gateway

- Sub-rede pública
- Associar Elastic IP
- Alterar route tables das privadas para enviar 0.0.0.0/0 → NAT

---

## Testes

### EFS

```bash
df -h | grep efs
cat /mnt/efs/teste-efs.txt
```

### RDS

```bash
mysql -h <endpoint-rds> -u admin -p
SHOW DATABASES;
USE databasewp;
SHOW TABLES;
```

### ALB

- Acessar o DNS público e verificar tela inicial do WordPress

### Conectividade

```bash
curl -I https://google.com
```

<p align="center">
  <img width="1025" height="566" alt="Opera Instantâneo_2025-08-11_135127_wp-load-1962812018 us-east-2 elb amazonaws com" src="https://github.com/user-attachments/assets/bd53c1e8-6d95-4eb6-b5d5-d26e9558c8d1">
</p>




