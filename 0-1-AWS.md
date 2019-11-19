# Crie seu ambiente na AWS Account

Acesse https://aws.amazon.com/pt/free/ e clique em "Crie sua conta gratuita". 
Selecione a região "São Paulo" no menu superior direito.

## Crie uma instancia RDS

Você precisará criar uma instância RDS. Para isso, acesse RDS no menu de serviços e clique em "Create database"

No formulário, selecione "Easy Create", "MySQL" e "Free tier". Você não será cobrada por isso.

Preencha as seguintes credenciais:

DB instance identifier: devopsgirlsdb
Master username: devopsgirls
Master password: devopsgirlsrds!

Clique em "Create Database" e espere a database ficar pronta.

## Crie uma Zona hosteada interna (Route 53)

## Create Internal DNS Hosted Zone (Route 53)

You'll need to create a Private Hosted Zone for Route 53. Go to *Services > Route 53*, then click on *Create Hosted Zone*. You'll want to set the following:

```
Domain Name: devopsgirls.internal
VPC: [Your Default VPC]
```

## Crie um bucket no S3

Você também vai precisar de um bucket no S3. Vá até *Serviços > S3*, então crie um novo bucket com um nome único. Anote este nome pois usaremos durante todo o bootcamp. Agora no menu do bucket, selecione *Propriedades*, então vá em *Permissões*. Selecione *Add Bucket Policy* e coloque a permissão abaixo. Lembre-se de trocar <meu-bucket> pelo nome do seu bucket! 


```

	"Version": "2008-10-17",
	"Id": "1",
	"Statement": [
		{
			"Sid": "VPC Only access",
			"Effect": "Allow",
			"Principal": {
				"AWS": "*"
			},
			"Action": [
				"s3:GetObject",
				"s3:PutObject"
			],
			"Resource": [
				"arn:aws:s3:::<meu bucket>",
				"arn:aws:s3:::<meu bucket>/*"
			],
			"Condition": {
				"StringEquals": {
					"aws:sourceVpc": "vpc-cad8d0a8"
				}
			}
		}
	]
}
```


## Create an S3 VPC Endpoint

Você também vai precisar criar um endpoint interno para acesso ao S3. Faça o seguinte:

1.) Vá até Serviços > VPC > Endpoints

2.) Selecione *Criar Endpoint*

3.) Na parte de seleção das route tables, selecione todas que estiverem disponíveis.

4.) Na configuração do Endpoint, configure o seguinte:

```
VPC: [your default VPC]
Service: com.amazonaws.ap-southeast-2.s3
Policy:

{
    "Statement": [
        {
            "Action": "*",
            "Effect": "Allow",
            "Resource": "*",
            "Principal": "*"
        }
    ]
}

```

Após esses passos, o seu ambiente está pronto!
