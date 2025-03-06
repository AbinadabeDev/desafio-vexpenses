# VExpenses - Terraform AWS Infrastructure

Este repositório contém código em Terraform para criar uma infraestrutura na AWS, configurada para suportar um projeto DevOps. O código define e configura os recursos da AWS, como VPC, sub-rede, instância EC2, regras de segurança e muito mais. O objetivo é fornecer uma base sólida para quem deseja entender e automatizar o processo de provisionamento de infraestrutura na nuvem com AWS e Terraform.

## Descrição do Projeto

Este código Terraform implementa a infraestrutura necessária para o projeto **VExpenses**, simulando a criação de uma instância EC2 na AWS, configuração de rede e segurança, utilizando boas práticas de organização e automação.

## Estrutura do Código

### 1. **Provedor AWS**
O código começa com a definição do provedor, especificando a região onde os recursos serão criados.

```hcl
provider "aws" {
  region = "us-east-1"
}
```
A região definida é us-east-1 (Costa Leste dos EUA), o que especifica que os recursos serão criados na AWS Norte da Virgínia.

### 2. **Variáveis de Configuração**

```hcl
variable "projeto" {
description = "Nome do projeto"
type        = string
default     = "VExpenses"
}

variable "candidato" {
description = "Nome do candidato"
type        = string
default     = "Abinadabe Oliveira"
}
```
As variáveis projeto e candidato são utilizadas para personalizar os recursos criados, como os nomes de chaves e recursos.

### 3. **Chave Privada para SSH**
A chave privada RSA é criada para permitir o acesso seguro à instância EC2 via SSH.
```hcl
resource "tls_private_key" "ec2_key" {
  algorithm = "RSA"
  rsa_bits  = 2048
}
```
A chave privada será utilizada para configurar o acesso SSH a uma instância EC2.

### 4. **Par de Chaves SSH na AWS**
O par de chaves é criado para ser associado à instância EC2, contendo a chave pública.
```hcl
resource "aws_key_pair" "ec2_key_pair" {
  key_name   = "${var.projeto}-${var.candidato}-key"
  public_key = tls_private_key.ec2_key.public_key_openssh
}
```
A chave pública gerada é associada à instância EC2.

### 5. **VPC (Virtual Private Cloud)**
Uma VPC é criada para isolar os recursos da AWS dentro de uma rede privada.
```hcl
resource "aws_vpc" "main_vpc" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true
  tags = {
    Name = "${var.projeto}-${var.candidato}-vpc"
  }
}
```
A VPC tem um bloco CIDR de 10.0.0.0/16, proporcionando até 65.534 endereços IP.

### 6. **Sub-rede na VPC**
A sub-rede é criada dentro da VPC para isolar ainda mais os recursos.
```hcl
resource "aws_subnet" "main_subnet" {
  vpc_id            = aws_vpc.main_vpc.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-east-1a"
  tags = {
    Name = "${var.projeto}-${var.candidato}-subnet"
  }
}
```
O intervalo de endereços IP para a sub-rede é definido como 10.0.1.0/24, proporcionando 256 endereços IP.

### 7. **Internet Gateway**
O Internet Gateway é configurado para permitir a comunicação entre a VPC e a internet pública.
```hcl
resource "aws_internet_gateway" "main_igw" {
  vpc_id = aws_vpc.main_vpc.id
  tags = {
    Name = "${var.projeto}-${var.candidato}-igw"
  }
}
```
### 8. **Tabela de Roteamento**
A tabela de roteamento é configurada para direcionar o tráfego da VPC para a internet via Internet Gateway.

```hcl
resource "aws_route_table" "main_route_table" {
  vpc_id = aws_vpc.main_vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main_igw.id
  }

  tags = {
    Name = "${var.projeto}-${var.candidato}-route_table"
  }
}
```

### 9. **Associação da Tabela de Roteamento**
A tabela de roteamento é associada à sub-rede.
```hcl
resource "aws_route_table_association" "main_association" {
  subnet_id      = aws_subnet.main_subnet.id
  route_table_id = aws_route_table.main_route_table.id
  tags = {
    Name = "${var.projeto}-${var.candidato}-route_table_association"
  }
}
```

### 10. **Grupo de Segurança**
O grupo de segurança é configurado para permitir conexões SSH (porta 22) de qualquer origem e todo o tráfego de saída.
```hcl
resource "aws_security_group" "main_sg" {
  name        = "${var.projeto}-${var.candidato}-sg"
  description = "Permitir SSH de qualquer lugar e todo o tráfego de saída"
  vpc_id      = aws_vpc.main_vpc.id

  ingress {
    description      = "Allow SSH from anywhere"
    from_port        = 22
    to_port          = 22
    protocol         = "tcp"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }

  egress {
    description      = "Allow all outbound traffic"
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }

  tags = {
    Name = "${var.projeto}-${var.candidato}-sg"
  }
}
```

### 11. **Instância EC2 com Debian 12**
A instância EC2 é lançada com a AMI Debian 12, tipo t2.micro e associada ao par de chaves e grupo de segurança criados.
```hcl
resource "aws_instance" "debian_ec2" {
  ami             = data.aws_ami.debian12.id
  instance_type   = "t2.micro"
  subnet_id       = aws_subnet.main_subnet.id
  key_name        = aws_key_pair.ec2_key_pair.key_name
  security_groups = [aws_security_group.main_sg.name]

  tags = {
    Name = "${var.projeto}-${var.candidato}-ec2-instance"
  }
}
```
### 12. **Outputs**
```hcl
output "private_key" {
  value     = tls_private_key.ec2_key.private_key_pem
  sensitive = true
}

output "public_key" {
  value = tls_private_key.ec2_key.public_key_openssh
}
```

