# Load Balancing e Alta disponibilidade.

## Conceitos

Antes de começarmos, vamos passar por alguns conceitos básicos:

 - Balanceadores de carga (Load Balancers) encaminham o tráfego para um ou mais provedores de serviço. Isso significa que você pode ter diversas  instâncias EC2 servindo o mesmo tipo de conteúdo, sendo representadas por um único serviço.

 - Alta disponibilidade é o fato do seu sistema permanecer operacional por um longo período de tempo. Isso significa que o seu sistema é mais resiliente a falhas.

 - Separação de serviços refere-se a garantir que, se você tiver multiplos sistemas dependentes uns dos outros, eles existam separadamente e compartilhem menos modos de falha.

 - Artefatos sao subprodutos tangíveis do desenvolvimento dos nossos sistemas. No nosso exemplo, nosso artefato será o pacote Wordpress modificado.
 - Artifacts are tangible byproducts of our systems development. In this case, our artifact will be the modified Wordpress package that's special to only you.

Agora vamos começar!

## O que vamos fazer

Nesse exercício prático nós iremos:

 - Criar um Balanceador Elastic
 - Reinstalar o Wordpress, porém usando um banco de dados externo
 - Criar um artefato do seu Wordpress modificado e então enviá-lo para o S3
 - Criar outra instância usando o artefato como base



## Criando um Elastic Load Balancer

### 1.) Crie o balanceador de carga

Vá à *Services* > *EC2* e então procure por *Load Balancers*. Clique em *Create Load Balancer*

![Image][2-1-1-create]


### 2.) Selecione o tipo do balanceador

Na próxima página, certifique-se de selecionar *Classic Load Balancer*.

![Image][2-1-2-classic]


### 3.) Nomeie o seu ELB

Coloque *meu_nome*-elb em *Load Balancer Name* (substitua *meu_nome* po um nome a sua escolha). Em *Create ELB Inside*, selecione *My Default VPC*.

![Image][2-1-3-lbname]


### 4.) Configure o protocolo do balanceador

Nesse caso, iremos deixar as configurações como estão - mas clique em *Load Balancer Protocol* e você verá algumas opções - balanceadores são muito poderosos!

![Image][2-1-4-listeners]


### 5.) Crie security groups para o ELB

Na próxima página, escolha a opção *Create a new Security Group*. Observe que é similar aos security groups que fizemos anteriormente - semelhante a um "firewall" no qual você pode configurar o tipo de tráfego que é permitido.

![Image][2-1-5-secgroups]

Nós deixaremos as configurações padrão por agora, exceto o campo "Source", no qual você deve escolher "My IP".

### 6.) Pule a próxima página pois não temos configuração de SSL

### 7.) Configure os health checks

Na próxima página, nós iremos configurar os *health checks*. Health checks verificam se a instância está "saudável", ou seja, se está aceitando tráfego ou se não está recebendo erros. Observe as dicas dadas em cada uma das opções.


![Image][2-1-6-healthchecks]

Agora iremos escolher *TCP* como protocolo Ping, usando a porta *80*. Isso significa que o balanceador irá checar a porta 80 de cada uma das instâncias e marcá-la como "saudável" se estas responderem de forma correta, sem erros.


### 8.) Configure as instâncias para encaminhar tráfego

Na página de instâncias, selecione a instância que você criou no módulo anterior. Isso irá habilitar o encaminhamento de qualquer tráfego que chegar ao seu balanceador para a sua instância.


![Image][2-1-7-instances]


### 9.) Adicione tags

Como de costume, vamos adicionar uma tag. Ponha uma chave chamada *Name* e associe um valor *meu_nome*-elb (substitua *meu_nome* por um nome de sua escolha).

![Image][2-1-8-tags]


### 10.) Finalizando!

Revise todas as configurações e clique em *Create*.

![Image][2-1-9-review]


### 11.) Veja os detalhes do ELB

Clique no link que redireciona para o seu ELB. Assim como na instância, você verá um painel com as propriedades do seu ELB. Anote o *DNS Name*, usaremos depois.


![Image][2-1-10-details]


## Reinstale o Wordpress usando RDS

Como o Wordpress mantem os dados de URLs no banco de dados, nós precisamos reinstalá-lo. Se sua sessao SSH ainda estiver ativa então prossiga - caso contrário, inicie uma sessão SSH (se precisar revisite as instruções aqui).

### 12.) Resete a configuração do Wordpress

Usando o seu terminal ou Putty, vá até o diretório do Wordpress e então remova o arquivo *wp-config.php*:

```
cd /var/www/html/
sudo rm wp-config.php
```

### 13.) Reconfigure o banco de dados do Wordpress

Agora abra o seu navegador e cole o *DNS Name* que você anotou no passo 11. Você irá ver uma página de instalação. Prossiga com a instalação, mas quando aparecer um painel para inserção dos detalhes do banco de dados insira os seguintes valores:

```
Database Name: devopsgirlsdb
Database User: devopsgirls
Database Password: devopsgirlsrds
Database Host: rds.devopsgirls.internal
Database Prefix: firstname_
```

Exemplo:

![Image][2-1-11-wpsql]


Certifique-se de substituir 'firstname_' pelo seu primeiro nome. Por exemplo, 'Banana Smith' deve colocar 'banana_'.


### 14.) Finalizando a instalação

Finalize a instalação, até que você chegará a uma página de Blog. Agora que o Wordpress está funcionando nós podemos criar o nosso artefato.

## Criando o artefato

### 15.) Crie uma cópia do diretório do Wordpress

Se a instalação ocorreu bem, então você irá criar uma cópia do seu diretório de configuração, dessa forma não precisaremos reinstalá-lo e seguir todo o processo anterior novamente. Nós podemos fazer isso via terminal ou utilizando Putty. Execute os seguintes comandos:


```
cd /var/www/html/
sudo tar cvfz ~/primeironome.ultimonome-wordpress.tgz .
```

Altere os valores *primeironome* e *ultimonome* com os seus dados.
*cd /var/www/html* altera o diretório atual para */var/www/html* - Onde está instalado o Wordpress. Então, na segunda linha, criamos um arquivo comprimido *tar* - primeironome.ultimonome-wordpress.tgz - contendo todos os arquivos e pastas dentro do diretório atual.


### 16.) Envie o arquivo para o S3

S3 é um serviço de armazenamento de objetos - basicamente permite que você armazene arquivos para um diretório e você pode compartilhá-lo tanto com a sua conta (privado) ou com o mundo (público). Nós faremos isso executando os seguintes comandos - um para configurar o tamanho máximo de upload, e outro para de fato copiar o arquivo para o S3 (*s3 cp*).


```
aws configure set default.s3.multipart_threshold 64MB
aws s3 cp ~/firstname.lastname-wordpress.tgz s3://devopsgirls-training-br-coaches/firstname.lastname-wordpress.tgz --no-sign-request
```

### 17.) Confirme se o arquivo existe usando o console web:

No navegador, vá em *Services > S3*. Clique no bucket chamado *devopsgirls-training-br-coaches*. Se tudo ocorreu bem, o seu arquivo deve aparecer aqui!

![Image][2-1-12-s3]


## Crie uma segunda instância

Recaptulando: Nós temos agora um balanceador e um artefato. Agora nós queremos ter múltiplas instâncias para que o nosso serviço continue rodando caso uma das instâncias caia.

### 18.) Crie a segunda instância

Vá em *Services > EC2* através do console web. Como no primeiro módulo, nós vamos criar uma nova instância. Clique em *Launch Instance*.


![Image][2-1-13-launchinstance]


### 19.) Insira os detalhes da instância

No passo 1, escolha *Amazon Linux AMI*. Em *Instance Type*, selectione *t2.micro*.

![Image][2-1-14-ami]


### 20.) Insira o *User Data*

Você pode pensar em *User Data* como scripts que você quer que rode quando sua instância EC2 inicia. Nós podemos usar User Data para declarar o que nós queremos fazer - nesse caso, vamos declarar os mesmos comandos que usamos quando instalamos o Wordpress pela primeira vez - exceto pelo fato de que não precisamos fazer login e rodar manualmente.

Na aba *Advanced Details* em *Step 3: Configure Instance Details*, cole os seguintes comandos abaixo em *User Data*:

```
#!/bin/bash
yum install -y mysql php php-mysql httpd
aws configure set default.s3.multipart_threshold 64MB
aws s3 cp s3://devopsgirls-training-br-coaches/primeironome.ultimonome-wordpress.tgz /var/www/wordpress.tgz --no-sign-request
tar xvfz /var/www/wordpress.tgz -C /var/www/html/
chown -R apache /var/www/html/
service httpd start
```

Lembrando que você deve substituir primeironome.ultimonome pelos seus dados (o mesmo que você colocou anteriormente). Sua configuração deve estar como na imagem abaixo:

![Image][2-1-15-userdata]


### 21.) Continue a configuração da instância

Para o resto da configuração da instância, especifique o seguinte:

```
 - Storage: Defaults
 - Tags:
     Key:Name
     Value: meunome-wordpress2
```

![Image][2-1-16-tags]


### 22.) Modifique a configuração de security group

Como sua instância deve receber trágego somente do seu balanceador, você pode especificar o seu balanceador como origem. Isso fará com que sua instância seja mais segura reduzindo o tipo de tráfego que ela pode receber.

Adicionalmente, você não precisa mais especificar o acesso SSH porque todas as configurações foram feitas através do seu User Data. Para a configuração de security group, escolha *Create a new security group* e especifique o seguinte:

```
 -  Type: HTTP
 -  Source: your ELB
```

Deve ficar assim:

![Image][2-1-17-elbsg]


### 23.) Inicie sua instância e vá a página de Load Balancers

Clique em *Launch instance* para iniciar a sua instância. Agora, volte para a sessão de Balanceadores ( *Services > EC2 > Load Balancers* ). Selecione o balanceador que você criou anteriormente e clique em *Instances* no painel inferior.

![Image][2-1-18-instances]


### 24.) Adicione a sua nova instância no balanceador

Clique em *Edit Instances* e adicione a nova instância que você acabou de criar (meunome-wordpress-2). Clique em *Save*.

![Image][2-1-19-addinstance]


### 25.) Confira se tudo está funcionando corretamente

Vá na aba *Description* do seu balanceador. Se tudo ocorreu bem, deve aparecer "*2 of 2 instances in service*". Voce pode testar da seguinte forma:

 - Recarregue a página do Wordpress no seu navegador

 - No console AWS (*Services > EC2*), clique com o botão direito em uma das suas instâncias e selecione *Stop Instance*.

 - Se tudo ocorreu bem, você deve continuar conseguindo acessar o seu site ao recarregar a página.

### 26.) E... Pronto!

![Image][2-1-20-blog]

Parabéns! Agora você tem um serviço com maior disponibilidade rodando!






[2-1-1-create]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-1-create.png
[2-1-10-details]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-10-details.png
[2-1-11-wpsql]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-11-wpsql.png
[2-1-12-s3]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-12-s3.png
[2-1-13-launchinstance]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-13-launchinstance.png
[2-1-14-ami]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-14-ami.png
[2-1-15-userdata]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-15-userdata.png
[2-1-16-tags]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-16-tags.png
[2-1-17-elbsg]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-17-elbsg.png
[2-1-18-instances]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-18-instances.png
[2-1-19-addinstance]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-19-addinstance.png
[2-1-2-classic]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-2-classic.png
[2-1-20-blog]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-20-blog.png
[2-1-3-lbname]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-3-lbname.png
[2-1-4-listeners]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-4-listeners.png
[2-1-5-secgroups]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-5-secgroups.png
[2-1-6-healthchecks]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-6-healthchecks.png
[2-1-7-instances]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-7-instances.png
[2-1-8-tags]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-8-tags.png
[2-1-9-review]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-9-review.png
