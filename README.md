# terraform-aws
Deploy to AWS with Terraform and Ansible

Qual a finalidade deste respositório?
```
Criarmos automaticamente um ambiente dentro da AWS, para que possamos realizar o deploy de um web app.
Seja ele qual for, você pode adaptar este repositório para o seu projeto e assim realizar de maneira estratégica
todos os passos para o deploy de seu app de uma maneira fácil e rápida.
Executando apenas um comando.
Serviços na AWS:
VPC, S3, RDS, EC2, LC/ASG, ELB, Route53 (Criação dos registros)
```


<strong>Qual a grande vantagem em utiizar o Terraform?</strong>

Você pode começar a versionar sua infraestrutura na nuvem, como versiona os códigos de seu app.
Uma vez versionada toda a sua infraestrutura, o Terraform pode implementá-la ou verificar a difenrença
entre o ambiente atual.

Outro ponto importante, você pode parar de resolver merda.
Ou seja, se um recurso começa a apresentar muitos problemas, ao invés de perder horas tentando resolver este problema,
você pode substituí-lo.
lol

![alt text](https://user-images.githubusercontent.com/33163017/33994084-4a213930-e0c0-11e7-9a4c-eee433ca6e74.png)

<blockquote>
Na imagem ilustrativa acima, temos um servidor Linux, e nele instalado o Terraform e Ansible.

Após a configurações do mesmo será necessário apenas efetuar  dois comandos.
```
$ terraform plan 
$ terraform apply
```
O terraform plan, irá planejar toda a execução da nossa configuração e verificar se tudo poderia ser criado, caso sim, ele vai dar um resumo de tudo que será criado e você executará o "terraform apply" para efetivamente criar todos esses recurso.

Após isso, tudo que configuramos estará disponível dentro da AWS.
O Ansible é utilizado para realizar as configurações na instância que hospedará o app.
</blockquote>

<h2>O que vou executar neste tutorial?</h2>
<blockquote>
<p>
Eu vou subir um blog com Angular.JS.
Ele será hospedado em uma EC2, sendo servido pelo Nginx com SSL.
</p>
<p>
Essa configuração terá um Load Balance para balanceamento de cargo e Auto Scaling.
Também será efetuado a criação deste dominio e ao final será possvel acessa-lo,
sem efetuar nada manual na AWS.
</p>
 
Isso pode ser utilizado por equipes de implantaço para realizar migraço de todo um sistema para a AWS.
</blockquote>

<strong>Setup</strong>

O primeiro passo será criamos um usuário dentro de nossa conta na AWS, para utilizar com o Terraform e Ansible.

Este passo permitirá que o Terraform consiga utilizar os recursos da API da AWS para criar a infraestrutura e para

que o Ansible possa configurar nossas instâncias EC2.

Crie um usuário que tenha poderes para administrar recursos como <strong>VPC, S3, RDS, LC/ASG, ELB e Route53,</strong>

faço o download da <strong>Access Key</strong> e <strong>Secret Key</strong>

Após realizar a criação do usuário você pode criar no VirtualBox, ou caso utilize sua própria máquina para efetuar o deploy.
Eu utilizei um CentOs 7 para tal.

Dentro desta instância vamos checar se o Pyhton esta instalado
```
$ python --version
```
Caso o Python não esteja instalado você pode executar um: 
```
yum -y install python
```
Eu pressuponho que você esteja como "root" e dentro do "~" do mesmo.
Agora vamos executar o download do PIP
```
curl -O https://bootstrap.pypa.io/get-pip.py
```
Realizamos a instalação:
```
python get-pip.py
```
Agora com o PIP instalado vamoz realizar o download da CLI da AWS
```
pip install awscli
```
Com a CLI da AWS instalada vamos realizar a configuração da mesma.
```
aws configure
```
Ele vai pedir algumas informações como listado abaixo, preencha com suas informaçes da AWS

AWS Access Key ID:

AWS Secret Key:

Default Region:

Default output format: (Deixe como padrão)

Agora vamos realizar a criação de um profile para utilização destas credenciais.

Isso nos ajuda quando estamos trabalhando com multiplos ambientes de trabalhos.
```
vim ~/.aws/credentials
```
```
[blogdeploy]
aws_access_key_id = YOUR ACCESS KEY
aws_secret_access_key = YOUR SECRET KEY
```
Assim nos meus scripts do Terraform, vamos conseguir informar qual o ambiente estamos trabalhando,

e neste caso sera o blogdeploy.

Agora vamos exportar isso para que o sitema estenda esta alteração.
```
export AWS_DESFAULT_PROFILE=blogdeploy
```
O Terraform nos permite utilizar as keys geradas pela AWS, porem neste caso vamos utilizar 

uma key criada por nós mesmo, para isso vamos utilizar o ssh-keygen
```
ssh-keygen
```
A única coisa que inseri foi o caminho com o nome da chave que eu gostaria

/root/.ssh/terraform

O resto mantive como padrão e também não criei um passpharese.

Eu consigo ver a chave criada

Adicionamos esta nova chave
```
ssh-agent bash
ssh-add ~/.ssh/terraform
```
Agora vamos realizar o download do Terraform
```
wget https://releases.hashicorp.com/terraform/0.7.10/terraform_0.7.10_linux_amd64.zip
```
Realizaremos a criação de um diretório chamado terraform, assim centralizamos tudo referente ao mesmo dentro deste diretório
```
mkdir ~/terraform
```
Vamos extrair o conteúdo do zip para a pasta que acabamos de criar:
```
unzip terraform_0.7.10_linux_amd64.zip -d ~/terraform
```
Feito isso vamos exportar dentro do nosso PATH o caminho para o script do terraform
```
export PATH=$PATH:~/terraform
```
Estamos terminando quase nossa sessão de setup, agora vamos realizar a instalação do Ansible
```
 yum install -y ansible
 ```
<h2>Projeto Passo-a-Passo<h2>

IMAGEMINFRAESTRUTURAFINAL

<H2>Directory and File Setup</h2>

Nesta etapa vamos começar a construir nosso projeto, vamos criar um diretório que será o padrão
para os nosso script, será nosso diretório de trabalho.

Eu vou criar no "~" do root, um diretório chamado: <strong>deployblog</strong>
```
mkdir deployblog
```
Agora acessamos este diretório e vamos criar três arquivos:

- main.tf

- variables.tf

- terraform.tfvars
```
cd deployblog
touch main.tf ; touch variables.tf ; touch terraform.tfvars
```
Feito isso, vamos editar o arquivo main.tf
```
vim main.tf
```
Aqui será definido nosso provedor de Cloud, e também variáveis referentes a acces key e região na AWS
```hcl
provider "aws" {
    region = "${var.aws_region}"
    profile = "${var.aws_profile}"
}
```
Agora vamos editar o arquivo variables.tf, onde colocaremos os valores das variáveis acima
```
vim varibales.tf
```
Adicione o seguinte conteúdo:
```hcl
variable "aws_region" {}
variable "aws_profile" {}
```
Editaremos agora o arquivo terraform.tfvars
```
vim terraform.tfvars
```
Copie o conteúdo abaixo:
```hcl
aws_profile = "blogdeploy"
aws_region = "sa-east-1"
```
<strong>Explicação</strong>
<blockquote>
 <p>
 Realizamos a criação de três aquivos, main.tf | variables.tf | terraform.tfvars
 </p>
 <p>
 O arquivo <strong>main.tf:</strong> neste arquivo vamos realizar toda a configuraço na AWS,
 como criação de VPC, e todos os recursos e serviços que vamos utilizar
 </p>
 <p>
 O arquivo <strong>variables.tf:</strong> neste arquivo vamos dar nomes as variáveis que serão inseridas,
 ao longo da codificaço do main.tf
 </p>
 <p>
 O arquivo <strong>terraform.tfvars:</strong> neste arquivo vamos inserir os valores das variáveis do arquivo
 variables.tf, isso para que no nosso código não tenham senhas, ou configuraçes de variáveis expostas.
 </p>
 </blockquote>
 
 <h2> Esboço do Terraform main.tf</h2>
 COLOCAR AINDA

<h2>Construindo nossa infraestrutura</h2>

Vamos editar nosso arquivo main.tf e começar a montar nossa infraestrutura, vamos seguir o roteiro acima, 
da lógica das criações.

Estes arquivos estão dentro do nosso diretório de trabalho como foram criados anteriormente.

Como este diretório esta no home do meu root.
```
vim deployblog/main.tf
```
Vamos colocar neste arquivo o conteúdo abaixo:
```hcl
#Criando a VPC
resource "aws_vpc" "vpc" {
  cidr_block = "10.1.0.0/16"
}
#Criando Internet Gateway
resource "aws_internet_gateway" "internet_gateway" {
  vpc_id = "${aws_vpc.vpc.id}"
}
# Create Route Tables
#Public Route Table
resource "aws_route_table" "public" {
  vpc_id = "${aws_vpc.vpc.id}"
  route {
        cidr_block = "0.0.0.0/0"
	gateway_id = "${aws_internet_gateway.internet_gateway.id}"
	}
  tags {
	Name = "public"
  }
}
#Private Route Table
resource "aws_default_route_table" "private" {
  default_route_table_id = "${aws_vpc.vpc.default_route_table_id}"
  tags {
    Name = "private"
  }
}
#Criando as subnets
#Public Subnet
resource "aws_subnet" "public" {
  vpc_id = "${aws_vpc.vpc.id}"
  cidr_block = "10.1.1.0/24"
  map_public_ip_on_launch = true
  availability_zone = "sa-east-1a"
  tags {
    Name = "public"
  }
}
#Private Subnet1
resource "aws_subnet" "private1" {
  vpc_id = "${aws_vpc.vpc.id}"
  cidr_block = "10.1.2.0/24"
  map_public_ip_on_launch = false
  availability_zone = "sa-east-1c"
  tags {
    Name = "private1"
  }
}
#Private Subnet2
resource "aws_subnet" "private2" {
  vpc_id = "${aws_vpc.vpc.id}"
  cidr_block = "10.1.3.0/24"
  map_public_ip_on_launch = false
  availability_zone = "sa-east-1c"
  tags {
    Name = "private2"
  }
}
#Private RDS1
resource "aws_subnet" "rds1" {
  vpc_id = "${aws_vpc.vpc.id}"
  cidr_block = "10.1.4.0/24"
  map_public_ip_on_launch = false
  availability_zone = "sa-east-1c" 
  tags {
    Name = "rds1"
  }
}
#Private RDS2
resource "aws_subnet" "rds2" {
  vpc_id = "${aws_vpc.vpc.id}"
  cidr_block = "10.1.5.0/24"
  map_public_ip_on_launch = false
  availability_zone = "sa-east-1c"
  tags {
    Name = "rds2"
  }
}
#Private RDS3
resource "aws_subnet" "rds3" {
  vpc_id = "${aws_vpc.vpc.id}"
  cidr_block = "10.1.6.0/24"
  map_public_ip_on_launch = false
  availability_zone = "sa-east-1c"
  tags {
    Name = "rds3"
  }
}
```
<strong>Explicação: </strong>
<blockquote>
Nesta etapa realizamos a criação dos requisitos bsicos de rede,
como VPC, Intenternet Gateway, Tabelas de Roteamento, Subnet pública e privadas.
</blockquote>
Nos precisaremos associar nossa subnet privada com nossa public route table
para que o nosso load balance fique acessível pela Web.

Editaremos novamente o arquivo main.tf.
```
vim deployblog/main.tf
```
Insira este conteúdo
```hcl
#Associação de Subnets
#Associando a subnet publica a nossa tabela de roteamento publica
resource "aws_route_table_association" "public_assoc" {
  subnet_id = "${aws_subnet.public.id}"
  route_table_id = "${aws_route_table.public.id}"
}
#Associando a subnet private1 a nossa table de roteamento publica
#Isso é necessário para que o load balance seja acessivel pelo web
resource "aws_route_table_association" "private1_assoc" {
  subnet_id = "${aws_subnet.private1.id}"
  route_table_id = "${aws_route_table.public.id}"
}
#Associando a subnet private2
resource "aws_route_table_association" "private2_assoc" {
  subnet_id = "${aws_subnet.private2.id}"
  route_table_id = "${aws_route_table.public.id}"
}
#Agora nos vamos criar um grupo de subnet para nosso RDS
resource "aws_db_subnet_group" "rds_subnetgroup" {
  name = "rds_subnetgroup"
  subnet_ids = ["${aws_subnet.rds1.id}", "${aws_subnet.rds2.id}", "${aws_subnet.rds3.id}"]
  tags {
    Name = "rds_sng"
  }
}
#Agora vamos realizar a criação dos Security Groups
#Public Security Group
resource "aws_security_group" "public" {
  name = "sg_public"
  description = "Usado pelas instancias publicas e instancias privadas para acesso do load balancer"
  vpc_id = "${aws_vpc.vpc.id}"
 #Roles
 #SSH
  ingress {
    from_port 	= 22
    to_port 	= 22
    protocol 	= "tcp"
    cidr_blocks = ["${var.localip}"]
  }
  #HTTP 
  ingress {
    from_port 	= 80
    to_port 	= 80
    protocol 	= "tcp"
    cidr_blocks	= ["0.0.0.0/0"]
  }
  #Outbound 
  egress {
    from_port	= 0
    to_port 	= 0
    protocol	= "-1"
    cidr_blocks	= ["0.0.0.0/0"]
  }
}
#Private Security Group
resource "aws_security_group" "private" {
  name        = "sg_private"
  description = "Used for private instances"
  vpc_id      = "${aws_vpc.vpc.id}"
  
#Acesso para os outros security groups
  ingress {
    from_port    = 0
    to_port      = 0
    protocol     = "-1"
    cidr_blocks  = ["10.1.0.0/16"]
  }
  egress {
    from_port    = 0
    to_port      = 0
    protocol     = "-1"
    cidr_blocks  = ["0.0.0.0/0"]
  }
}
#RDS Security Group
resource "aws_security_group" "RDS" {
  name= "sg_rds"
  description = "Used for DB instances"
  vpc_id      = "${aws_vpc.vpc.id}"
# SQL access from public/private security group
  
ingress {
    from_port        = 3306
    to_port          = 3306
    protocol         = "tcp"
    security_groups  = ["${aws_security_group.public.id}", "${aws_security_group.private.id}"]
  }
}
```
<strong>Explicação:</strong>
<blockquote>
Acredito que o código assim esteja bem comentado,
onde realizo a criação de mais alguns recursos dentro da AWS.
</blockquote>

<h2>Criando a instância do RDS</h2>

Agora vamos começar a realizar a criação de recursos computacionais.

Vamos editar novamente o arquivo main.tf e realizar estas configurações.
```
vim deployblog/main.tf
```
```hcl
#Criando nosso RDS
resource "aws_db_instance" "db" {
  allocated_storage	= 10
  engine		= "mysql"
  engine_version	= "5.6.27"
  instance_class	= "${var.db_instance_class}"
  name			= "${var.dbname}"
  username		= "${var.dbuser}"
  password		= "${var.dbpassword}"
  db_subnet_group_name  = "${aws_db_subnet_group.rds_subnetgroup.name}"
  vpc_security_group_ids = ["${aws_security_group.RDS.id}"]
}
```

Nos adicionamos no main.tf algumas variáveis que não estavam presentes em nosso arquivo variables.tf e terraform.tfvars

Primeiro vamos editar o arquivo variables.tf
```
vim variables.tf
```
Vamos adicionar abaixo das outras variáveis o seguinte conteúdo:
```hcl
variable "localip" {}
variable "db_instance_class" {}
variable "dbname" {}
variable "dbuser" {}
variable "dbpassword" {}
```
Agora vamos alterar o terraform.tfvars e colocar os valores destas variáveis.
```
vim deployblog/terraform.tfvars
```
Eu coloquei o seguinte conteúdo, aqui você pode alterar para os valores desejados
```hcl
db_instance_class  = "db.t1.micro"
dbname             = "deployblogdb"
dbuser             = "deployblog"
dbpass             = "deployblogpass"
```

<h2>Criando nosso key pair</h2>

Neste passo nos vamos criar nosso key pair para ser usado na AWS.
Obs: Esse nosso key pair será a chave que geramos pelo ssh-keygen no inicío deste tutorial.

Para criarmos este recurso editaremos novamente o arquivo main.tf
```
vim deployblog/main.tf
```
O último registro que efetuamos neste arquivo foi a criação de nosso RDS.

Vamos adicionar o conteúdo abaixo após esta entrada do RDS.
```hcl
#Criando o Key Pair
resource "aws_key_pair" "auth" {
  key_name  ="${var.key_name}"
  public_key = "${file(var.public_key_path)}"
}
```
Assim como fizemos após a criação da configuração para subir uma instância RDS, precisaremos editar os arquivos
variables.tf e terraform.tfvars adicionando estas novas variáveis.
```
vim deployblog/variables.tf
```
```hcl
variable "key_name" {}
variable "public_key_path" {}
```
Agora vamos editar o arquivo terraform.tfvars.
```
deployblog/terraform.tfvars
```
```hcl
key_name             = "terraform"
public_key_path      = "/root/.ssh/terraform.pub"
```
<blockquote>
Obs: public_key_path é o caminho da chave gerada pelo ssh, nos passos inicias.
</blockquote>

<h2>Criando nosso bucket no S3</h2>

Nesta etapa criaremos nosso bucket e algumas outras coisas na nossa Infraetsrutura que serão apresentadas ainda, como nosso damain name.

Antes de começar já vamos realizar a confiuração da variável Domain Name.

Para isso vamos editar o arquivo variables.tf
```
vim deployblog/variables.tf
```
Ao final do arquivo adicione:
```hcl
variable "domain_name" {}
```
Como adicionamos uma variável no variables.tf já sabemos que devemos setar seu cocnteúdo no arquivo terraform.tfvars.

Vamos edita-lo.
```
vim deployblog/terraform.tfvars
```
```hcl
domain_name        = "fabiomaranflorentino"
```
Agora partiremos para a criação do nosso S3.

A primeira coisa que vamos fazer e realizar a criação de um role para ser atachada as máquinas que realizarão acesso ao S3.

Isso é muito útil quando nossas máquinas trabalham com serviços que podem ser integrados.

Pois assim eu não preciso de um usuário de cada serviço, eu apenas defino na role os serviços que esta máquina acessam, outra vantagem: eu subo imagens a partir desta máquina e ela ja vem configurada. :)

Vamos lá, lembra do nosso conograma? que a primeira coisa a fazer seria esta?

Pois é parece que não mas vamos fazer isto agora e esta será uma das primeiras coisas que nosso terraform script irá executar. :)

Vamos editar o arquivo main.tf.

Vamos deixar esta parte logo abaixo de onde definimos nosso provedor e antes da criação da VPC.

 <strong> Em construção </strong>
 
