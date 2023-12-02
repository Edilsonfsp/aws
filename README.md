# Curso AWS Academy Cloud Architecting 
## Escola SENAI Santo André - Projeto conclusão do curso.
### Autor: Edilson F Souza
[![NPM](https://img.shields.io/npm/l/react)](https://github.com/Edilsonfsp/aws/blob/main/LICENSE)
# Sobre o Projeto
## Visão geral do projeto
Este projeto oferece a você uma oportunidade para demonstrar as habilidades de design de soluções que você desenvolverá ao longo deste curso. Seu objetivo será projetar e implantar uma solução para o caso a seguir.
### No final deste projeto, você deverá ser capaz de aplicar os princípios de projeto arquitetônico que aprendeu neste curso para:
- Implantar uma aplicação PHP executada em uma instância do Amazon Elastic Compute Cloud (Amazon EC2)
- Criar uma instância de banco de dados que a aplicação PHP possa consultar
- Criar um banco de dados MySQL a partir de um arquivo de despejo de linguagem de consulta estruturada (SQL)
- Atualizar parâmetros da aplicação em um AWS Systems Manager Parameter Store
## Entregas para Projeto Capstone
- Oferecer hospedagem segura do banco de dados MySQL
- Conceder acesso seguro a um usuário administrativo
- Conceder acesso anônimo a usuários da Web
- Executar o site em uma instância t2.micro do EC2 e conceder acesso SSH (Secure Shell) aos administradores
- Fornecer alta disponibilidade ao site por meio de um balanceador de carga
- Armazenar informações de conexão do banco de dados no AWS Systems Manager Parameter Store
- Oferecer escalabilidade automática que usa um modelo de execução
## Arquitetura atual do projeto
![Projeto inicial!](assets/imgs/capstonestart.jpg "Arquitetura inicial do projeto")
# Passo a passo
### 1 - Criação do Application Load Balancer ( Nome: CapStone-ELB )
> Visite o [Link da Documentação da AWS](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/application-load-balancer-getting-started.html "A referência para consultar como criar um load balancer.") caso seja necessário.
  - ***1.1 Criar um Applicattion Load Balancer*** ( Nome: CapStone-ELB )
      - Scheme: Internet-facing
      - IP address Type: IPV4
      - VPC: Escolha a VPC Example VPC
      - Mapping: Escolha Public Subnet 1 e Public Subnet 2
      - Security groups: Escolha ALBSG
      - Listeners and routing: Siga o passo 1.1.1
        - ***1.1.1 Criar um Target Group a partir do Launch Template Example-LT*** ( Nome: CapStone-TG )
            - Default action: Clique no link Create target Group (Uma nova janela será aberta)
            - Choose a target type: Instances
            - Target group name: CapStone-TG
            - Protocol: Http Port: 80
            - IP address type: IPv4
            - VPC: Example VPC
            - Protocol version: HTTP1
            - Health check protocol: HTTP
            - Health check path: /
            - Clique em Next
            - Depois clique em Create target group
            > Volte para a janela anterior ( Onde esta criando o load balancer e continue a criação )
      - Em default action clique no icone para atualizar 
      - Forwart to: Escolha CapStone-TG
      - Clique em Create load balancer       
### 2 - Criar um Auto Scaling Groups ( Nome: CapStone-ASG )
  - Clique no menu sanduiche no lado superior esquerdo caso necessario
  - Cliqe em Auto Scaling Groups
  - Clique em Createauto scaling group
  - Name: CapStone-ASG
  - Launche Template: Escolha Example-LT
  - Clique em Next
  - VPC: Escolha Example VPC
  - Availability Zones and subnets: Escolha Private Subnet 1 e Private Subnet 2
  - Clique em Next
  - Load balancing: Escolha Attach to an existing Load balancing
  - Attach to an existing load balancer: Mantenha selecionando Choose from your load balancer target groups
  - Existing load balancer target groups: Escolha CapStone-TG
  - EC2 health checks: Marque Turno on Elastic Loada Balancing health checks
  - Health check grace period: 30
  - Additional setting: Marque Enable group metrics collection within CloudWatch
  - Clique em next
  - Desired capacity: 2
  - Scaling limits: Min 2 e Max 2
  - clique em next
  - clique em next
  - clique em Create Auto Scalling group
  - Volte a janela do Target groups e atualize a tela
  - Verifique o Total de targets e a saúde das instancias. 
      - Clique no menu sanduiche no lado superior esquerdo caso necessario.
      - Clique em Instances.
      - Verifique se as instancias foram iniciadas.
### 3 - Teste a aplicação utilizando o endereço de DNS do Load Balancer CapStone-ELB, abra em uma nova guia do navegador.
  - O Load Balancer está funcionando? sim 
  - O load balancer encontrou as instancias com a aplicação? não
  - Qual erro foi apresentado? ***502 Bad Gateway***
> ***Troubleshooting:*** Para corrigir este erro será necessário criar um NAT para a comunicação das instâncias com o Load Balancer.
### 4 - Criar um NAT ( nome: CapStone-NAT )
  - Em services -> clique Networking & Content Delivery: Clique em VPC
  - Na lateral esquerda clique em NAT Gateways:
  - Clique em Create NAT gateway
  - Name: CapStone-NAT
  - Subnet: Public Subnet 1
  - Connectivity type: Public
  - Elastic IP allocation ID: Clique em Allocate Elastic IP
  - Clique em Create NAT gateway
  - Repita o passo 3.
      - O load balancer funcionou? não
> ***Troubleshooting:*** O problema ainda persiste pois é preciso configurar uma rota na tabela de rotas das subredes privadas.
### 5 - Aualizar a tabela de rotas privada
  - Em services -> clique Networking & Content Delivery: Clique em VPC
  - Clique em route tables
  - Selecione Private route table 1
  - Na parte de baixo da tela selecione a guia routes
  - Clique em edit routes
  - Clique em add route
  - Destination: 0.0.0.0/0
  - Target: Selecione Nat Gateway e depois CapStone-NAT
> Aguarde alguns instantes e verifique o endereço do load balancer no navegador, e depois volte novamente à página do Target group para ver a saúde das instâncias.
    - A aplicação funcionou? sim
    - Você consegue executar uma query? não é possível. Apresenta um erro ***"Connection error: No such file or directory"***
> ***Troubleshooting:*** Este erro acontece por que ainda não foi criado o RDS.
### 6 - Criar um banco de dados MySQL
> Clique em services -> Clique em Database -> clique em RDS
  - ***6.1 - Criar um Subnet groups.*** Nome: CapStone-SubNetGroups
      - No menu esquerdo clique em Subnet Groups
      - Clique em create DB subnet group
          - Name: CapStone-SubNetGroups
          - Description: Subnets para o banco de dados capstonedb.
          - VPC: Example VPC
          - Availability Zones: Escolha as duas primeiras
          - Subnets: Escolha as subnets privadas 10.0.2.0/23 e 10.0.4.0/23
      - Clique em create
  - ***6.2 - Criar o banco de dados MySQL***
      - no menu esquerdo clique em Databases
      - Clique em create database
      - Choose a database creation method: Standard create
      - Engine options: Escolha MySQL
      - Templates: Dev Test
      - Availability and durability: Mult-AZ DB Instance
      - DB instance identifier: capstonedb ( tudo minusculo )
      - Master username: admin
      - Master password: 12345678
      - Confirm master password: 12345678
      - DB instance class: Burstable classes
      - Storage type: General Purpose SSD
      - Allocated storage: 20 GB
      - Virtual private cloud (VPC): Escolha Example VPC
      - DB subnet group: CapStone-SubNetGroups
      - Existing VPC security groups: Example-DB
      - Database authentication: Marque Password authentication
      - Demarque todos os monitoramentos.
      - Expanda Additional configuration
      - Initial database name: capstonedb
      - Clique em create database
> Aguarde alguns instantes e na faixa verde que aparece clique em View connection details e anote as informações que serão usadas na próxima etapa.
  - Os seguintes parâmetros serão usados pela aplicação PHP para se conectar ao banco de dados:
      - /example/endpoint - String: capstonedb.c3i3qriyqk1z.us-east-1.rds.amazonaws.com ( exemplo )
      - /example/username - String: admin
      - /example/password - String: 12345678
      - /example/database - String: capstonedb
> Aguarde alguns instantes e verifique o endereço do load balancer no navegador.
  - Com o banco de dados criado você consegue executar um query? não.
  - Ainda apresenta um erro ***"Connection error: No such file or directory"***
> ***Troubleshooting:*** Este erro acontece por que ainda não foi configurado o acesso ao banco de dados nas variáveis do Sistems Managers.
### 7 - Configurando o Parameter Store no Systems Manager
> Obs: Esses valores de parâmetro diferenciam maiúsculas de minúsculas.
  - ***7.1 - Criando os parametros do endpoint***
      - Clique em services -> Clique em Management & Governance -> Clique em Systems Manager
      - No menu  a esquerda clique em Parameter Store
      - Clique em create parameter 
      - Name: /example/endpoint 
      - Type: String
      - Value: capstonedb.c3i3qriyqk1z.us-east-1.rds.amazonaws.com ( exemplo )
  - ***7.2 - Criando os parametros do nome do banco de dados***
      - Clique em services -> Clique em Management & Governance -> Clique em Systems Manager
      - No menu  a esquerda clique em Parameter Store
      - Clique em create parameter 
      - Name: /example/database 
      - Type: String
      - Value: capstonedb
  - ***7.3 - Criando os parametros do usuário do banco de dados***
      - Clique em services -> Clique em Management & Governance -> Clique em Systems Manager
      - No menu  a esquerda clique em Parameter Store
      - Clique em create parameter 
      - Name: /example/username 
      - Type: String
      - Value: admin
  - ***7.4 - Criando os parametros da senha do usuário do banco de dados***
      - Clique em services -> Clique em Management & Governance -> Clique em Systems Manager
      - No menu  a esquerda clique em Parameter Store
      - Clique em create parameter 
      - Name: /example/password 
      - Type: String
      - Value: 12345678
> Aguarde alguns instantes e verifique o endereço do load balancer no navegador
  - Com o banco de dados do system manager criado você consegue executar um query? não.
  - Ainda apresenta um erro ***"Connection error: Unknown database 'capstonedb'"***
> ***Troubleshooting:*** Este erro acontece por que ainda não foi configurado a tabela no banco de dados
### 8 - Criando um acesso seguro a nossa instancia com a aplicação.
  - Clique em services -> Clique em Compute -> Clique em EC2
  - No menu a esquerda clique em Instances.
  - Selecione uma das instancias com nome ExampleAPP
  - Anote o endereço Private IPv4 addresses: 10.0.X.X ( Esse endereço será usado para o acesso remoto seguro ) 10.0.4.150
  - Selecione a instancia com nome Bastion
  - Clique em connect no menu acima
  - Clique no botão laranja connect
  - Volte na janela do Projeto Capstone no Academy e clique em AWS Details
  - Em SSH Key clique em Show e copie o texto da chave
  - Digite o comando: ```#sudo nano labsuser.pem```
  - Botão direito colar
  - Control o (letra O)
  - Enter
  - Control X
  - Digite o comando: ```#sudo chmod 400 labsuser.pem```
  - ***8.1 - Conectando na instância da aplicação***
      - Execute o comando: ```#sudo ssh -i labsuser.pem ec2-user@10.0.4.150```
      - Caso obtenha sucesso com a conexão digite yes e depois tecle enter
      - A conexão obteve sucesso?
          - não - Leia o Troubleshooting.
          - sim - Faça o passo 9.
> ***Troubleshooting:*** Será preciso liberar o acesso a instância do Bastion no grupo de segurança da aplicação. Faça o passo 8.2 e tente a conexão novamente.
  - ***8.2 - Liberar o acesso do Bastion no grupo de segurança da instância da aplicação***
      - Clique em services -> Clique em Compute -> Clique em EC2
      - Clique em Security Groups.
      - Selecione Inventory-App
      - Clique em Inbound rules
      - Clique em Edit Inbound rules
      - Clique em Add rule
      - Type: SSH
      - Source: Escolha Bastion-SG
      - Clique: Em Save rules.
      - Volte ao passo 8.1
### 9 - Criar e verificar as tabelas do banco de dados.
> Na seção de SSH da instância da aplicação execute os próximos passos.
  - ***9.1 - Populando o banco de dados***
      - Digite o comando: ls
          - A saída deve mostrar o arquivo Countrydatadump.sql (Este arquivo será utilizado para realizar o dump da base de dados.)
      - Digite o comando:
          - ```# mysql -u admin -p --host <Cole o end point do RDS> <Cole o nome do RDS> < Countrydatadump.sql```
      - Digite a senha do usuário do RDS: 12345678 ( No caso do exemplo dado neste passo a passo )
  - ***9.2 - Verificando se a base de dado foi criada.***
      - Digite o comando: ``` # mysql -u admin -p --host <Cole o end point do RDS>```
      - Digite a senha do usuário do RDS: 12345678 ( No caso do exemplo dado neste passo a passo )
      - O prompt do mysql digite os seguintes comandos;
          - ```MySQL [(none)]> show databases;```
          - ```MySQL [(none)]> use capstonedb;```
          - ```MySQL [capstonedb]> show tables;```
          - ```MySQL [capstonedb]> select - from countrydata_final;```
          - ```MySQL [capstonedb]> exit;```
> Obs: Volte para a página da aplicação e tente realizar uma consulta. A consulta foi realizada com sucesso? sim. ***Clique em submit para enviar o seu projeto e pontuar.***
#### Obs: Obrigado por acompanhar este passo a passo. Para dúvidas e erros encontrado contate-me pelo linkedim 
