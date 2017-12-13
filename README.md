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
...
$ python --version
...

