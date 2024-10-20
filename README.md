Modos de utilização:
Step1: Criar um usuário IAM com as seguintes permissões:
´´´{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "VisualEditor0",
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
				"ec2:GetSecurityGroupsForVpc",
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
				"ec2:DisassociateRouteTable",
				"ec2:DescribeInstanceTypes",
				"ec2:DescribeTags",
				"ec2:DescribeInstanceAttribute",
				"ec2:DescribeVolumes",
				"ec2:DescribeInstanceCreditSpecifications",
				"ec2:DescribeAddresses",
				"ec2:DescribeAddresses",
				"ec2:DescribeAddressesAttribute"
			],
			"Resource": "*"
		}
	]
}´´´
Step 2: Instalar o AWS CLI
Step 3: Criar uma Acess Key e Secreet Key para o usuário criado anteriormente
Step 4: Configurar o Acess Key e Secreet Key via cli
Step 5: Instalar o Terraform
Step 6: Executar o comando ´´´terraform init´´´ para inicializar o terraform
Step 7: Executar o comando ´´´terraform plan´´´ para verificar quais são os recursos que serão adicionados
Step 8: Executar o comando ´´´terraform apply´´´ para subir toda a infraestrutura
Step 9: Para acessar a página do Nginx basta acessar 'http://{Endereço IP público da instância EC2}'
Step 10: Para excluir toda a infraestrutura basta executar o comando ´´´terraform destroy´´´


Tarefa 1
Descrição Técnica

O código escrito em Terraform cria uma arquitetura de rede e uma EC2 com o nginx instalado assim que a EC2 é provisionada.

1. tls_private_key.ec2_key -> Gera uma chave privada e codifica no formato PEM(Explicar o que é PEM). Geralmente é utilizado para facilitar a criação de ambientes temporários.
2. aws_key_pair.ec2_key_pair -> Cria uma chave de acesso pública para acessar a EC2 via SSH
3. aws_vpc.main_vpc -> Cria uma VPC(Virtual Private Cloud) com o escopo de rede 10.0.0.0/16 tendo disponíveis 65.536 ips, porém a AWS reserva alguns ips para gerenciamento interno dela
4. aws_subnet.main_subnet -> Criando uma subnet de caráter pública com acesso a internet
5. aws_internet_gateway.main_igw -> Criando um Internet Gateway para fornecer acesso de entrada e saída para internet
6. aws_route_table.main_route_table -> Criando uma route table para a subnet pública com rota 0.0.0.0/0(saída para internet) através do Internet Gateway
7. aws_route_table_association.main_association -> Dando um attach subnet na route table para que a subnet utilize das rotas das subnet
8. aws_security_group.main_sg -> Criando um security group para acessar a instância a regra para acesso da instância ta para qualquer ip para a porta 80(HTTP) e regra de saída qualquer porta para qualquer ip
9. aws_ami.debian12 -> Definindo a AMI da EC2 com o sistema operacional debiam com o modelo de virtualização HVM
10. aws_instance.debian_ec2 -> Criando uma EC2 t2.micro(1vCpu/1GB RAM), 20GB de disco do tipo GP3, na vpc, subnet, security group criada anteriomente e instalando o Nginx
11. aws_eip.elastic-ip-servidor-web -> Criando e dando um attach no elastic-ip na ec2 para que o ip público da EC2 não mude quando a instância renicie
12. Output -> Os output do terraform são o IP público e a Chave privada gerada e não mostra no console, pois, a flag sensitive = true;

Tarefa 2
Modificações:
1. candidato -> Foi alterado a variável candidato para o nome vinicius-nobre
2. aws_subnet.main_subnet -> Foi alterado o sufixo do nome da subnet para identificar de forma mais prática caso a subnet seja pública ou privada, neste caso a subnet é pública foi colocado o sufixo public 
3. aws_route_table.main_route_table -> Foi alterado o sufixo da route table para identificar se tem destino para um IGW(public) ou NATGW(private) para facilitar a identificação, neste caso foi colocado public pois uma das route da route table tem saída e entrada para internet
4. aws_route_table_association.main_association -> Foi retirado a tag, pois, não precisa colocar tag em uma ação de association no terraform, visto que o nome da route table já foi definido no resource main_route_table
5. aws_security_group.main_sg -> Foi retirado a liberação da porta 22, pois, não é necessário, nesse caso se configurando como uma falha de segurança. Foi alterado também a descrição do security group, para indicar o acesso somente a porta 80(HTTP). Retirado o ipv6_cidr_blocks, pois, só é necessário o cidr_blocks.
6. aws_instance.debian_ec2 -> Foi alterado para coletar o id do security group em vez do name, foi alterado para gp3 o tipo do disco pois ele tem um custo e uma eficiência melhor.
7. Instalação do Nginx -> Para instalar o nginx foi adicionado os comando "sudo apt install -y nginx" para instalar e o comando "sudo systemctl start nginx" e para inicializar o serviço
8. aws_eip.elastic-ip-servidor-web -> Criando e dando um attach no elastic-ip na ec2 para que o ip público da EC2 não mude quando a instância renicie
9. Foi substituido o output para exibir o elastic-ip da EC2 em vez do ip anterior.

Sugestões:
1. Colocar o backend para que o estado de terraform seja armazenado de forma durável em um S3 em vez de armazenar localmente.
2. Configurar o AWS Secreet para colocar o ACESS_KEY e SECREET_KEY no serviço da AWS em vez de configurar localmente.
3. Alterar a estrutura de arquivos do terraform para substituir de um único arquivo main.tf, para módulos, assim sendo, criando 4 módulos, vpc (Criar a VPC), variables (Com as variáveis utilizadas no código), ec2 (Com as instruções para provisionar EC2, junto com a chave de acesso), securitygroup (Criação do security Group)
4. OBS: Não foi implementado essas sugestões, pois, precisaria provisionar recursos que não estavam na lista "original".