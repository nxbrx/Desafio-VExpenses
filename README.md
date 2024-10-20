# Guia de Utilização do Projeto Terraform para AWS

### 1. Criar um Usuário IAM com Permissões Necessárias
Crie um usuário IAM com as permissões descritas abaixo para provisionar a infraestrutura necessária:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:CreateKeyPair",
                "ec2:CreateTags",
                "ec2:DescribeKeyPairs",
                "ec2:DeleteKeyPair",
                "ec2:CreateVpc",
                "ec2:DeleteVpc",
                "ec2:DescribeVpcs",
                "ec2:ModifyVpcAttribute",
                "ec2:CreateSubnet",
                "ec2:DeleteSubnet",
                "ec2:DescribeSubnets",
                "ec2:CreateInternetGateway",
                "ec2:AttachInternetGateway",
                "ec2:DetachInternetGateway",
                "ec2:DeleteInternetGateway",
                "ec2:DescribeInternetGateways",
                "ec2:CreateRouteTable",
                "ec2:DeleteRouteTable",
                "ec2:DescribeRouteTables",
                "ec2:CreateRoute",
                "ec2:ReplaceRoute",
                "ec2:CreateSecurityGroup",
                "ec2:DeleteSecurityGroup",
                "ec2:AuthorizeSecurityGroupIngress",
                "ec2:RevokeSecurityGroupIngress",
                "ec2:AuthorizeSecurityGroupEgress",
                "ec2:RevokeSecurityGroupEgress",
                "ec2:RunInstances",
                "ec2:TerminateInstances",
                "ec2:DescribeInstances",
                "ec2:AllocateAddress",
                "ec2:ReleaseAddress",
                "ec2:AssociateAddress",
                "ec2:DisassociateAddress",
                "ec2:DescribeImages",
                "ec2:ImportKeyPair",
                "ec2:DescribeVpcAttribute",
                "ec2:AssociateRouteTable",
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeNetworkInterfaces",
                "ec2:DescribeTags",
                "ec2:DescribeInstanceTypes",
                "ec2:DescribeInstanceAttribute",
                "ec2:DescribeVolumes",
                "ec2:DescribeInstanceCreditSpecifications",
                "ec2:DescribeAddresses",
                "ec2:DescribeAddressesAttribute"
            ],
            "Resource": "*"
        }
    ]
}
```

### 2. Instalar o AWS CLI
Baixe e instale a AWS CLI a partir do site oficial da AWS.

### 3. Criar Access Key e Secret Key
Crie as chaves de acesso (Access Key e Secret Key) para o usuário IAM criado anteriormente.

### 4. Configurar Access Key e Secret Key via CLI
Execute o comando `aws configure` para inserir as chaves de acesso e configurar sua CLI.

### 5. Instalar o Terraform
Baixe e instale o Terraform a partir do site oficial.

### 6. Inicializar o Terraform
No diretório do projeto, execute o comando:
```bash
terraform init
```

### 7. Planejar a Infraestrutura
Para visualizar os recursos que serão criados, execute:
```bash
terraform plan
```

### 8. Aplicar a Infraestrutura
Para criar todos os recursos na AWS, execute:
```bash
terraform apply
```

### 9. Acessar a Instância EC2 com Nginx
Após a aplicação, acesse a página do Nginx no navegador com o endereço:
``` 
http://{Endereço IP público da instância EC2}
```

### 10. Excluir Infraestrutura
Para remover todos os recursos criados, execute:
```bash
terraform destroy
```

---

## Descrição Técnica da Arquitetura

O código em Terraform cria uma arquitetura de rede na AWS e uma instância EC2 com o Nginx instalado. Os principais componentes são:

1. **tls_private_key.ec2_key**: Gera uma chave privada no formato PEM, usada para acessar temporariamente a instância EC2. O formato PEM é amplamente utilizado para armazenar e transmitir chaves criptográficas.
   
2. **aws_key_pair.ec2_key_pair**: Cria uma chave pública associada à instância EC2 para acesso via SSH.

3. **aws_vpc.main_vpc**: Cria uma Virtual Private Cloud (VPC) com o escopo de rede `10.0.0.0/16`, oferecendo até 65.536 endereços IP.

4. **aws_subnet.main_subnet**: Subrede pública com acesso à Internet.

5. **aws_internet_gateway.main_igw**: Fornece acesso de entrada e saída à internet para a VPC.

6. **aws_route_table.main_route_table**: Cria uma tabela de rotas para permitir tráfego entre a subrede pública e a internet.

7. **aws_route_table_association.main_association**: Associa a subrede pública à tabela de rotas.

8. **aws_security_group.main_sg**: Define regras de segurança para permitir tráfego HTTP (porta 80) de qualquer IP.

9. **aws_ami.debian12**: Utiliza a AMI Debian 12 com virtualização HVM para provisionar a EC2.

10. **aws_instance.debian_ec2**: Cria uma instância EC2 `t2.micro` (1vCPU, 1GB RAM, 20GB de disco GP3) e instala o Nginx.

11. **aws_eip.elastic-ip-servidor-web**: Atribui um Elastic IP à instância EC2 para garantir que o IP público permaneça constante.

12. **Outputs**: O IP público da EC2 e a chave privada são outputs do Terraform, mas não são exibidos no console por questões de segurança.

---

## Arquivo Modificado
Veja o arquivo [main.tf](https://github.com/nxbrx/Desafio-VExpenses/blob/main/main.tf) modificado.

---

## Modificações Implementadas

1. **Variável "candidato"**: Atualizada para o nome `vinicius-nobre`.

2. **aws_subnet.main_subnet**: Adicionado o sufixo `public` para identificação clara.

3. **aws_route_table.main_route_table**: Adicionado o sufixo `public` para rotas de Internet Gateway (IGW).

4. **aws_security_group.main_sg**: Removida a porta 22 (SSH) para evitar falhas de segurança e modificado para liberar apenas a porta 80 (HTTP).

5. **aws_instance.debian_ec2**: Atualizada para coletar o ID do Security Group em vez do nome e alterado o tipo de disco para GP3.

6. **Instalação do Nginx**: Adicionados comandos para instalar e iniciar o serviço Nginx na instância EC2.

7. **aws_eip.elastic-ip-servidor-web**: Atribui um Elastic IP à instância EC2.

8. **Output**: O Elastic IP da instância EC2 é exibido em vez de um IP temporário.

---

## Sugestões de Melhoria

1. **Backend S3**: Armazenar o estado do Terraform em um bucket S3 para maior durabilidade.

2. **AWS Secrets Manager**: Armazenar as credenciais (Access Key e Secret Key) no AWS Secrets Manager, em vez de configurá-las localmente.

3. **Modularização**: Separar a configuração em módulos:
   - `vpc`: Criar a VPC.
   - `variables`: Definir variáveis globais.
   - `ec2`: Provisionar a instância EC2.
   - `securitygroup`: Criar o Security Group.

**OBS**: As sugestões não foram implementadas por não estarem previstas no escopo original.
