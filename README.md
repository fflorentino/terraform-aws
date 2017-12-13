# terraform-aws
Deploy to AWS with Terraform and Ansible

<strong>Setup</strong>
O primeiro passo será criamos um usuário dentro de nossa conta na AWS, para utilizar com o Terraform e Ansible.

Este passo permitirá que o Terraform consiga utilizar os recursos da API da AWS para criar a infraestrutura e para

que o Ansible possa configurar nossas instâncias EC2.

Crie um usuário que tenha poderes para administrar recursos como <strong>VPC, S3, RDS, LC/ASG, ELB e Route53,</strong>

faço o download da <strong>Access Key</strong> e <strong>Secret Key</strong>

Após realizar a criação do usuário e efetuar o download das chaves, crie uma instância EC2 com Centos 7.

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
 
 <strong> Em construço </strong>
