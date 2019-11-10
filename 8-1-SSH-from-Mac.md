# Acesso SSH via Mac

### 1.) Abra o seu terminal

Abra o **Finder**. Vá para **Application** > **Utilities** e depois abra o app chamado **Terminal**

### 2.) Verifique onde se encontra o seu par de chaves

Por padrão, seu par de chaves fica localizado na pasta *Downloads*. Confirme se ele se encontra lá, executando o seguinte comando:

```
ls ~/Downloads/[NOME DO MEU KEYPAIR].pem
```

Certifique-se de substituir *[NOME DO MEU KEYPAIR]* pelo nome que você deu ao seu par de chaves. Por exemplo, se eu gerei um par de chaves chamado `banana-smith-keypair`, então o comando que irei executar será: 

```
ls ~/Downloads/banana-smith-keypair.pem
```

### 3.) Altere o dono do arquivo

Lembre-se: este arquivo é necessário para que você consiga fazer login na sua instância! Portanto, ele precisa estar acessível somente por você. Nós iremos fazer isso executando o seguinte comando:

```
chmod 600 ~/Downloads/[NOME DO MEU KEYPAIR].pem
```

Novamente, você precisa substituir *[NOME DO MEU KEYPAIR]* pelo nome que você deu ao arquivo. Portanto, para `banana-smith-keypair`, o comando será:

```
chmod 600 ~/Downloads/banana-smith-keypair.pem
```

### 4.) Copie o IP ou URL da sua instância

Lembra de quando anotamos o *Public IP* da sua instância EC2? Agora nós iremos usá-lo. Basicamente nós iremos conectar ao IP da instância usando o arquivo contendo o par de chaves que nós criamos. Copie para a sua área de transferência ou anote em algum lugar, por exemplo Notepad.


### 5.) Login!

E finalmente, nós iremos conectar na instância. Nós faremos isso ao executar o seguinte comando:

```
ssh ec2-user@[MY IP ADDRESS] -i ~/Downloads/[NOME DO MEU KEYPAIR].pem
```

Assumindo que temos um *Public IP* com valor `22.22.22.22` e um par de chaves chamado `banana-smith-keypair`,o comando deverá ser:


```
ssh ec2-user@22.22.22.22 -i ~/Downloads/banana-smith-keypair.pem
```


