## Acesso SSH à sua instância EC2

### 1) Anote o endereço de IP público (*Public IP*) da sua instância EC2

No console Web da AWS clique em EC2

Vá em *Running Instances*

Localize e selecione a sua instância EC2

Anote o endereço de IP IPv4 público (*Public IP*) da instância. 


### 2) Altere o modo do arquivo .pem para 'read-only' (somente leitura)

Abra o seu terminal

Vá ao diretório onde se encontra o seu arquivo .pem e altere-o para 'read-only'

Execute o seguinte comando no terminal (assumindo que você salvou o arquivo em *Downloads*):


`$chmod 400 ~/Downloads/nomedachave.pem`

Exemplo:

`$chmod 400 ~/Downloads/leorentanyag.pem` 
 
### 3) Acesso SSH na sua instância usando o seu arquivo .pem

`ssh -i ~/Downloads/<keyname.pem> ec2-user@<IPv4 Public IP adress>`

Exemplo:
`ssh -i ~/Downloads/leorentanyag.pem ec2-user@13.55.214.58`
