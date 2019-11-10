# Configurações de inicialização e grupos de Auto Scaling 

## Conceitos

Antes de começarmos, vamos visitar alguns conceitos básicos:

 - *Configurações de inicialização são um modelo de como você deseja criar as suas instâncias*. Isso significa que você pode configurá-las da mesmo forma que você configuraria uma instância EC2, com a diferença de que, neste caso, você não está de fato criando-a e sim criando um *plano* para elas. 

 - *Grupos de Autoscaling decretam quantas instâncias você deseja que estejam ativas*. Há vários mecanismos para se fazer isso - atravésde intervenção manual ou checagens de sistema, mas combinado com Configurações de Inicialização, você pode automatizar esse processo. 

Agora vamos começar!

## O que iremos fazer

Neste exercício prático, nós iremos:

    - Criar uma configuração de inicialização

    - Especificar o User Data como fizemos no EC2 anteriormente

    - Criar um Grupo de Autoscaling

    - Configurar o grupo de Autoscaling para atribuir instâncias ao ELB

    - Testar o scaling
 

## Criando uma configuração de inicialização


### 1.) Criando a configuração de inicialização

Va a *Services > EC2* e procure por *Launch Configurations* na sessão a esquerda. Clique em *Create Launch Configuration*.

![Image][3-1-createlcfg]


### 2.) Defina o tamanho da configuração e AMI

Nós vamos configurar a configuração de inicialização da mesma forma que configuramos a nossa instância anteriormente - com os seguintes parâmetros:


```
 - AMI: Amazon Linux AMI
 - Instance Type: t2.micro
```

![Image][3-1-2-instancetype]


### 3.) Configure os detalhes da configuração de inicialização

Nós vamos definir os detalhes da nossa configuração. Nomeie-a como *meunome-launchconfig* (lembre-se de trocar *meunome* pelo seu nome).

![Image][3-1-3-iamrole]

Note que esse papel IAM permite somente acesso de leitura aos conteúdos do bucket S3 onde você colocou o seu pacote Wordpress (e nada mais).

### 4.) Defina detalhes avançados

No mesmo painel, clique em *Advanced Details*. No campo *User Data*, cole os seguintes comandos:

```
#!/bin/bash
yum install -y mysql php php-mysql httpd
aws s3 cp s3://devopsgirls-training/primeironome.ultimonome-wordpress.tgz /var/www/wordpress.tgz --no-sign-request
tar xvfz /var/www/wordpress.tgz -C /var/www/html/
chown -R apache /var/www/html
service httpd start
```

**Certifique-se** de que você modificou  `primeironome.ultimonome-wordpress.tgz` pelo seu nome, ou pelo nome que você definiu anteriormente. Modifique também o nome do bucket S3 de acordo com a sua conta.

![Image][3-1-4-userdata]


### 5.) Desabilite o endereço de IP público

No mesmo painel, selecione *Do not assign public IP*. Isso é importante em termos de segurança e separa as instânvias da web pública - todas as interações serâo feitas através do balanceador.

Vá para a próxima página.

### 6.) Adicione um storage (armazenamento)

Nós não precisamos modificar o armazenamento, então deixe tudo em *Add Storage* com os valores padrão por agora.

![Image][3-1-5-storage]


### 7.) Security Groups

Nós iremos fazer o mesmo que fizemos anteriormente com as instâncias EC2. Em *Protocol*, selecione *HTTP* - então defina *Source* como *"Custon IP"*. Se você começar a digitar o nome do balanceador que você criou anteriormente, ele deve aparecer na lista.

![Image][3-1-6-secgroups]


O ponto positivo disso é que qualquer instância que nós criarmos receberá tráfego HTTP *somente do seu balanceador*.

### 8.) Verifique a sua configuração, selecione o par de chaves SSH

Revise a sua configuração de inicialização. Verifique se tudo está correto e crique em *Create Launch Configuration*. 

![Image][3-1-7-reviewlc]


Assim como quando criamos as instâncias EC2, será pedido para que você especifique um par de chaves. Selecione o que você criou anteriormente.

## Criando um grupo de Autoscaling

### 9.) Crie um grupo de Autoscaling para a sua configuração de inicialização

Uma vez que você clicou em *Create Launch Configuration*, você deverá ver um botão chamado *Create an Autoscaling Group using this Launch Configuration*. Clique nele. 

![Image][3-1-8-createsgh]


### 10.) Configure os detalhes do grupo de Autoscaling

No próximo painel, especifique de GroupName como *meunome-asg* (substitua *meunome* pelo seu nome). Configure *Group Size* para *2* instâncias.

![Image][3-1-9-asgname]


### 11.) Selecione Network e Subnets

Para este exercício, iremos usar a VPC marcada como *default*. Se você clicar em *Subnet*, você terá de duas a três opções para escolher. Selecione *all of them* como destino.


![Image][3-1-10-subnets]


Subnetes (subredes) basicamente define quais Availability Zones (zonas de disponibilidade) suas instâncias ficarão. Pense em Availability Zones como se fossem datacenters - diferentes localidades no Brasil onde sua instância irá ficar. 
Subnets essentially set which Availability Zones to deply your instances under. Think of Availability Zones as datacenters - separate locations in Sydney where your instance will live. Isso protege seu serviço contra interrupções, se, por exemplo, um datacenter em um local do Brasil ficar inativo.

### 12.) Defina o Balanceador

Se você clicar em *Advanced Details* no mesmo painel, você verá uma caixa de seleção chamada *Load Balancing*. Isso irá permitir que você registre o servidor como *live* e atribuí-lo ao balanceador criado anteriormente. 


![Image][3-1-11-elb]


Um conjunto adicional de opções irá aparecer. Se você clicar na caixa *Classic Load Balancers*, você poderá selecionar o balanceador que voce criou.

### 13.)Mantenha as scaling pocilies (políticas de escalonamento) como padrão

Nós não vamos mexer nas Scaling Policies agora, então escolha *Keep this group at its initial size*. Só lembre-se que você pode usar *policies* para monitorar o uso de memória ou CPU das suas instâncias - por exemplo: se o uso de CPU ultrapassar 70%, você pode configurar uma policy para adicionar mais instâncias automaticamente. 


### 14.) Pule a parte de notificações

Nós não iremos enviar notificações agora. Pense nisso como se fosse uma maneira de notificar alguem que um determinado evento ocorreu - por exemplo, uma instância foi deletada ou iniciada.


### 15.) Configure as tags

Como sempre fazemos, é importante adicionar tags para as futuras instâncias assim que forem criadas. Adicione uma chave *Name* e adicione o seu nome como valor.

![Image][3-1-12-tags]


### 16.) Revise e finalize!

Você encontrará uma sessão onde poderá revisar as informações inseridas. Clique em *Create Auto Scaling Group*.

![Image][3-1-13-review]


## Testando o seu grupo de Autoscaling: Substituindo instâncias

Agora tudo está pronto! Nós temos um *modelo* para criar instâncias e temos um *contador* de instâncias a serem criadas, então vamos ver se está tudo funcionando!

### 17.) Verifique o seu balanceador novamente

Va em *Services > EC2 > Load Balancers*. Selecione o balanceador que você criou antes (*meunome-elb*), então procure pela descrição dele.

![Image][3-1-14-elbinstances]


### 18.) Use o DNS para acessar o seu balanceador

Copie o nome de DNS (semelhante ao mostrado abaixo) e cole em outra aba do seu navegador. Você está vendo agora uma de suas 4 instâncias.


![Image][3-1-15-accesslb]


### 19.) Mate suas instâncias

Va em *Services > EC2 > Instances*. No campo de pesquisa acima digite o seu nome - assim você irá pesquisar todas as instâncias que tiverem o seu nome como tag.

Para cada uma das suas 4 instâncias: *Clique com botão direito > Instance State > Terminate*.

![Image][3-1-16-killec2]


### 20.) Verifique o seu ELB

No seu navegador, recarrege a aba com o nome de DNS do seu balanceador. Você deverá receber um erro.


### 21.) Observe as novas instâncias serem criadas 

No console AWS, verifique em *Services > EC2 > Instances* a cada minuto. Você irá ver novas instâncias com o seu nome serem criadas.

![Image][3-1-17-instancereplace]



## Testando o seu grupo de Autoscaling: Redefina o numero de instâncias

Agora que sabemos que as instâncias são iniciadas automaticamente, nós podemos ver o que acontece se mudarmos o numero de instâncias no grupo de Autoscaling.

### 22.) Modifique o seu grupo de Autoscaling

Va em *Services > EC2 > Autoscaling Group*. Procure pelo grupo de Autoscaling que você criou.

Após selecionar o grupo de Autoscaling que você criou, vá no painel inferior e clique em *Edit*. Defina *Desired Instances* como 3 e *Max Instances* como 3. Clique em *Save*.

![Image][3-1-18-setasg]


### 23.) Observe as novas instâncias serem criadas

No console da AWS, verifique em *Services > EC2 > Instances* a cada minuto. Você irá ver as novas instâncias com o seu nome serem criadas.

Parabéns! Você acabou de automatizar o serviço de scaling!


## Infrastructure as code

Agora que você fez tudo para construir um *scaling group*, está na hora de implantar tudo isso como código. Infrastructure-as-code é um conceito muito importante para DevOps - isso assegura que sua infraestrutura é replicável e colaborativa.

### 24.) Download o Cloudformation template


Download o cloudformation template [here](https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/devopsgirls-wordpress.yaml). Você pode abri-lo com o seu editor de texto (Notepad, gedit etc).

### 25.) Observe o template

Perceba que tudo que você fez pode ser declarado linha por linha no documento. Eu sei que parece confuso, mas não se preocupe! É só a declaração de coisa que você quer construir.

### 26.) Vá para a sessao de Cloudformation

Vá em *Services > Cloudformation*. Clique em *Create Stack*.

Isso irá te levar apra uma página onde você pode fazer upload do seu arquivo de template. Clique em *Next*.

![Image][3-1-19-upload]


### 27.) Defina o Stack Name e os parâmetros

Defina o parâmetro `DevopsGirlsUser`. Utilize o seu `primeironome.ultimonome` para isso. 

Altere `WordpressS3Bucket` para o nome do seu bucket do S3 - por exemplo: `devopsgirls-training-2` ou `devopsgirls-training-3` dependendo da sua conta. 

### 28.) Revise e clique em IAM resources

Clique nas demais caixas de diálogo até que você chegará na sessão *Review*. Na próxima caixa de texto onde diz "*I acknowledge that AWS CloudFormation might create IAM resources*", marque as caixas.


![Image][3-1-20-iam]

Clique em *Create*.

### 29.) Observe o seu stack ser criado

Vá em *Services > Cloudformation* e procure pelo nome do stack que você definiu anteriormente. Embaixo, na caixa de diálogo clique em *Events*. Recarregue a página de tempos em tempos e observe o seu ambiente ser criado!

Parabéns! Você acabou de implantar todo o ambiente utilizando código!


[3-1-10-subnets]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-10-subnets.png
[3-1-11-elb]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-11-elb.png
[3-1-12-tags]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-12-tags.png
[3-1-13-review]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-13-review.png
[3-1-14-elbinstances]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-14-elbinstances.png
[3-1-15-accesslb]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-15-accesslb.png
[3-1-16-killec2]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-16-killec2.png
[3-1-17-instancereplace]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-17-instancereplace.png
[3-1-18-setasg]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-18-setasg.png
[3-1-19-upload]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-19-upload.png
[3-1-18-iam]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-20-iam.png
[3-1-20-iam]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-20-iam.png
[3-1-2-instancetype]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-2-instancetype.png
[3-1-3-iamrole]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-3-iamrole.png
[3-1-4-userdata]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-4-userdata.png
[3-1-5-storage]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-5-storage.png
[3-1-6-secgroups]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-6-secgroups.png
[3-1-7-reviewlc]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-7-reviewlc.png
[3-1-8-createsgh]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-8-createsgh.png
[3-1-9-asgname]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-9-asgname.png
[3-1-ASG]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-ASG.png
[3-1-createlcfg]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-createlcfg.png
