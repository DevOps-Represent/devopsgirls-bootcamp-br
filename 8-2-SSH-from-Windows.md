# Acesso SSH via Windows

Infelismente, o melhor client SSH para Windows requer um formato próprio. Então para acesso via Windows temos 2 passos:

A.) Gerar a chave

B.) Usar a chave

## Gerar a chave

### 1.) Vá a página de download do PuTTY

Abra o seguinte link no seu navegador: http://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html

### 2.) Download PuTTY

Clique com o botão direito no link da última instalação mais recente e selecione *Save As*. No momento em que este curso estava sendo escrito a última versão era  **putty-0.67-installer.msi**.

### 3.) Execute o instalador

Execute o arquivo MSI e inicie a instalação. Vários programas serão instalados mas nós só precisamos de 2: *PuTTY* e *PuTTYGen*.

### 4.) Inicie o PuTTYGen

A partir do menu *Start*, escolha **All Programs > PuTTY > PuTTYGen**

Em **Type of key to generate**, selecione **SSH-2 RSA**.


### 5.) Carreque o seu par de chaves

Clique em **Load**. Por padrão, PuTTYGen mostra somente arquivos com extensão `.ppk`. Para localizar o seu arquivo .pem, selecione a opção *All Files*.


Vá até o diretório onde você baixou o seu par de chaves, abra o arquivo e clique em *Open*.

### 6.) Salve a sua chave privada

Escolha **Save private key** par salvar a sua chave privada em um formato que o PuTTY consiga usar. PuTTYGen irá mostrar um alerta pois a chave não tem uma senha. Escolha *Yes*.

### 7.) Nomeie a sua chave

Especifíque o mesmo nome que você deu para o seu par de chaves (por exemplo, `banana-smith-keypair`)


## Acesse a instância

### 1.) Inicie o PuTTY 

A partir do menu inicial do Windows, escolha **All Programs > PuTTY > PuTTY**

### 2.) Configure o seu login

No painel *Category*, selecione **Session** e preencha os seguintes campos:


```
 Host: ec2-user@[PUBLIC IP]
 Port: 22
 Connection type: SSH
```

Certifique-se de substituir *[PUBLIC IP]* pelo *Public IP* da sua instância EC2. Por exemplo:

```
 Host: ec2-user@22.33.44.55
 Port: 22
 Connection type: SSH
```


### 3.) Selecione o seu arquivo com a chave

No painel *Category*, expanda **Connection**, expanda **SSH** e então selecione **Auth**.

Clique em **Browse**. Selecione o arquivo `.ppk` que você gerou para o seu par de chaves e então selecione **Open**.  


### 4.) Salve a sua sessão

No painel *Category*, selecione **Session**. Em **Saved Sessions**, insira um nome para a sua sessão. Selecione **Save**.

### 5.) Login!

E finalmente, está tudo pronto. Clique em **Open** para iniciar sua conexão SSH - um alerta de segurança será exibido na sua tela mas você pode ignorá-lo sem problemas (para essa sessão).