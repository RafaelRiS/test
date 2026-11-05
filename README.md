# test

## Brute Force (Ou Força Bruta)

Tenta descobrir senhas ou credenciais testando várias combinações até encontrar a correta.

## Auditoria (ftp)

Primeiro verificamos a comunicação com o ping da outra rede/ computador

Exemplo:

``` bash
ping -c 3 192.168.56.101
```

|Parâmetro|Significado|
|---|---|
|`ping`|Ferramenta utilizada para verificar a conectividade entre duas máquinas na rede.|
|`-c 3`|Envia exatamente 3 pacotes de teste (ICMP).|
|`192.168.56.101`|Endereço IP da máquina de destino (Metasploitable 2).|

Se a resposta aos 3 pacotes for respondida com sucesso então podemos ir para o próximo passo.

### NMap

Agora usamos o NMap para verif

Exemplo:

``` bash
nmap -sV -p 21,22,80,445,139 192.168.56.101
```

Aqui ==-p== é para especificar as portas que queremos que o nmap escaneie
Portas 21,22,80,445 e 139 são portas muito comuns para FTP, SSH, HTTP, SMB e NetBIOS/SMB.

|Parâmetro|Significado|
|---|---|
|`nmap`|Ferramenta utilizada para escaneamento de redes.|
|`-sV`|Identifica as versões dos serviços que estão rodando nas portas abertas.|
|`-p`|Especifica quais portas serão escaneadas.|
|`21`|Porta utilizada pelo serviço FTP.|
|`22`|Porta utilizada pelo serviço SSH.|
|`80`|Porta utilizada pelo serviço HTTP (servidor web).|
|`445`|Porta utilizada pelo serviço SMB.|
|`139`|Porta utilizada pelo serviço NetBIOS/SMB.|
|`192.168.56.101`|Endereço IP da máquina de destino (Metasploitable 2).|

O Parâmetro ==🟡-sV== Tenta identificar a versão do serviço que está rodando em cada porta.

E quando rodamos esse comando, e o scan termina, o nmap retorna para nós uma saída indicando se há portas abertas ou não, e as versões dos serviços.

Se a porta 21 aparecer como aberta por exemplo, a gente vai saber que o FTP está Habilitado e está rodando em uma versão antiga por ser Vulnerável.

![Comando nmap](Imagens/Comando2%20nmap.png)

E para saber por exemplo se realmente a porta 21 (FTP) está aberta podemos usar o seguinte comando:

```bash
ftp 192.168.56.101
```


| Parâmetro        | Significado                                        |
| ---------------- | -------------------------------------------------- |
| `ftp`            | Inicia um cliente para conexão com servidores FTP. |
| `192.168.56.101` | Endereço IP do Metasploitable 2.                   |

Se a resposta for "Connected to 192.168.56.101 220" então significa realmente que está aberta e que está aceitando conexão.
![[Imagens/Resposta ftp.png]]

Pronto, agora sabemos que o serviço está realmente ativo, está aberta a porta e que está vulnerável e conseguimos saber que podemos começar diretamente com o FTP (Meduza), porém, não sabemos qual é o login e nem a senha.

Para isso, para tentar descobrir vamos tentar o ataque de força bruta utilizando a ferramenta ==Medusa== e para isso vamos precisar ==🔵**criar duas listas, uma com os possíveis nomes de usuário e outra com senhas comuns**==

Vamos criar primeiro um arquivo chamado "users.tx" no seguinte diretório:

```bash
echo -e "user\nmsfadmin\nadmin\nroot" > users.txt
```

|Parâmetro|Significado|
|---|---|
|`echo`|Exibe ou escreve um texto.|
|`-e`|Habilita a interpretação de caracteres especiais, como `\n`.|
|`"user\nmsfadmin\nadmin\nroot"`|Texto que será escrito no arquivo, com cada usuário em uma linha.|
|`\n`|Representa uma quebra de linha.|
|`>`|Redireciona a saída para um arquivo, criando-o ou sobrescrevendo seu conteúdo.|
|`users.txt`|Nome do arquivo que receberá a lista de usuários.|

E vamos criar também o de senhas no seguinte diretório:

```bash
echo -e "123456\npassword\nqwerty\nmsfadmin" > pass.txt
```

|Parâmetro|Significado|
|---|---|
|`echo`|Exibe ou escreve um texto.|
|`-e`|Habilita a interpretação de caracteres especiais.|
|`"123456\npassword\nqwerty\nmsfadmin"`|Texto contendo quatro senhas, separadas por quebras de linha.|
|`\n`|Representa uma quebra de linha.|
|`>`|Redireciona a saída para um arquivo, criando-o ou sobrescrevendo seu conteúdo.|
|`pass.txt`|Nome do arquivo que receberá a lista de senhas.|

e enfim vamos executar o medusa com o seguinte comando:

```bash
medusa -h 192.168.56.101 -U users.txt -P pass.txt -M ftp -t 6
```

|Parâmetro|Significado|
|---|---|
|`-h`|Endereço IP do alvo|
|`-U`|Arquivo com lista de usuários|
|`-P`|Arquivo com lista de senhas|
|`-M ftp`|Módulo de autenticação FTP|
|`-t 6`|Número de tarefas paralelas|

E ele vai começar a tentar as combinações possíveis e assim que for acertando ele vai retornando "ACCOUNT FOUND" e "Success".

E então sabendo o Usuário e Senha, podemos conectar novamente via FTP:

```bash
ftp 192.168.56.101
```

|Parâmetro|Significado|
|---|---|
|`ftp`|Inicia um cliente para conexão com servidores FTP.|
|`192.168.56.101`|Endereço IP do Metasploitable 2.|

Colocamos login e senha e esses são os significados das respostas:

|Mensagem|Significado|
|---|---|
|`Connected to 192.168.56.101`|Conexão estabelecida com o servidor.|
|`220 (vsFTPd 2.3.4)`|O servidor está utilizando o vsFTPd versão 2.3.4.|
|`Name`|Solicitação do nome de usuário.|
|`331 Please specify the password`|O servidor está solicitando a senha.|
|`230 Login successful`|Sucesso Autenticação realizada.|
|`Remote system type is UNIX`|O servidor remoto utiliza um sistema UNIX.|
|`Using binary mode`|Transferências configuradas em modo binário.|

-------------------------------------------------------------------------

## Auditoria 2 (Formulário de login)

Esse tipo de ataque é extremamente comum e se não for mitigado ele pode levar ao comprometimento total do sistema especialmente se o painel acessado for de administração, gerenciamento de banco de dados ou mesmo controle de infraestrutura.

O Cenário que vamos simular é através do dam vunerable web que é o DVWA, que basicamente vai rodar em um servidor com o ip que estamos utilizando na máquina 2 (Metasploitable), não no Kalilinux, mas esse processo vamos fazer no Kali.

E esse sistema (DVWA) é interessante porque ele vai apresentar para a gente um formulário web normal que nem a gente vê no dia a dia, onde o usuário vai digitar o login a senha e vai clicar em login, porém nosso objetivo será entender como esse formulário se comporta tecnicamente por trás do navegador (Se o desenvolvedor esqueceu alguma dica, deixou alguma brecha/ vulnerabilidade por trás desse formulário) e vamos também automatizar as tentativas de login utilizando a ferramenta MEDUZA que vai simular essas interações, vamos descobrir se alguma das credenciais que a gente vai utilizar são fracas, se o sistema possui algumas credenciais fracas que nós vamos passar e se elas vão ser aceitas e vamos analisar o que isso significa para evitar que aconteça em aplicações reais que nós estejamos gerenciando.

![[Imagens/Metasploitable2 HTTP.png]]

![[Imagens/DVWA.png]]

E abrimos o INSPECIONAR (A parte do desenvolvedor, e vamos abrir na abinha 'Network') e aqui a gente consegue ver tudo que o navegador está enviando e recebendo durante a interação com o Servidor/ backend:

![[Imagens/Inspecionar do DVWA.png]]

Clicamos na requisição POST e na abinha 'Request' onde podemos ver quais os nomes dos parâmetros que o Backend espera receber:

![[Imagens/DVWA Parametros de requisicao.png]]

e é o formato que vamos usar no MEDUSA e a mensagem de erro nesse caso 'Login failed' é a frase que o MEDUSA vai usar como indicador de falhas. Ou seja, se a resposta não contiver essa frase, o MEDUSA vai entender que o login foi bem sucedido.

## Vamos partir agora para nossa criação de um Wordlist

Fornecemos os dois arquivos ao MEDUZA um com os nomes de usuários possíveis e o outro com senhas possíveis que são chamados de WordList que são basicamente simples arquivos .txt onde cada linha vai representar uma tentativa e para fazer isso vamos fazer o seguinte:

Criar os dois arquivos com o comando

```bash
echo -e "user\nmsfadmin\nadmin\nroot" > users.txt
```

e

```bash
echo -e "123456\npassword\nqwerty\nmsfadmin" > pass.txt
```

Ou podemos criar com os nomes users_http.txt e pass_http.txt.

Podemos pegar Wordlists mais bem elaboradas com milhares e até milhões de entradas.

E então usamos o seguinte comando:

```bash
medusa -h 192.168.56.101 -U users.txt -P pass.txt -M http \
-m PAGE: '/dvwa/login.php' \
-m FORM: 'username=^USER^&password=^PASS^&Login=Login' \
-m 'FAIL=Login failed' -t 6
```

Temos que tomar cuidado com as Aspas pois:

' - Aspas simples o shell interpreta como parte do mesmo comando ainda.
" - Aspas duplas o shell/ bash interpreta como separador de comando.

------------------------------------------------------------------------

## Auditoria 3 (Enumeração SMB e Password Spraying)

#### O que é Serviço SMB ?

Serviço SMB significa Server Massage Block. É um protocolo da rede da Microsoft, utilizado para compartilhar arquivos, pastas, impressoras, também para realizar a autenticação de usuários e também para a comunicação entre máquinas Windows e também Linux via SAMBA.

## O que é Samba?

O Samba é um software que permite compartilhar arquivos, pastas e impressoras entre computadores pela rede.

Ele permite, por exemplo, que:

- Um computador Linux compartilhe arquivos com Windows.
    
- Um usuário acesse pastas compartilhadas em outro computador.
    
- Administradores gerenciem permissões de acesso a arquivos pela rede.

## Portas utilizadas

|Porta|Serviço|
|---|---|
|139|NetBIOS Session Service|
|445|SMB diretamente sobre TCP|

## Por onde começamos?

A Primeira Etapa vai ser a ENUMERAÇÃO DE USUÁRIOS utilizando o ==enum4Linux==. Então, imagine que queremos entrar em um prédio com várias portas, mas nenhuma delas tem identificação. O que a gente faz? A gente vai tentar abrir todas as portas? Não.

Primeiro a gente vai tentar descobrir quais portas realmente existem e quais levam a lugares que vão ser úteis para gente.

Com ataques Cibernéticos a lógica é a mesma; Se a gente começar tentando senhas em contas que nem existem, o sistema pode gerar alertas, bloquear IP ou acionar um administrador, além disso, a gente vai estar desperdiçando tentativa e tempo. Por isso, antes de qualquer ataque, a gente precisa confirmar quem são os usuários reais do sistema, que isso é chamado de enumeração de usuários.

 Para isso nós vamos usar uma ferramenta chamada ==enum4linux== que vai interagir com o protocolo SMB, que é usado para o compartilhamento entre máquinas Windows e Linux. 
 
 Se o serviço SMB estiver mal configurado como frequentemente ocorre em servidores legados ou ambientes de testes, a gente vai conseguir então extrair informações interessantes para o nosso ataque como por exemplo:

- Nomes de contas locais reais; 
- Políticas de Senha;
- Grupos de Usuários
- E Etc.

## Começando o Teste

Então para ver se isso funciona bora para o Terminal digitar o Seguinte comando:

```bash
enum4linux -a 192.168.56.101 | tee enum4_output.txt
```

E para vermos o arquivo que foi criado é só digitarmos o seguinte comando:

```bash
less enum4_output.txt
```

## ==🔵Less é um programa que podemos utilizar para ler textos.==

| Elemento           | Significado                                                                    |
| ------------------ | ------------------------------------------------------------------------------ |
| `enum4linux`       | Executa a ferramenta Enum4linux.                                               |
| `-a`               | Solicita uma enumeração abrangente, reunindo várias categorias de informações. |
| `192.168.56.101`   | Endereço IP da máquina que será analisada.                                     |
| \|                 | Pipe: direciona a saída de um comando para outro.                              |
| `tee`              | Exibe a saída na tela e também a grava em um arquivo.                          |
| `enum4_output.txt` | Nome do arquivo onde os resultados serão salvos.                               |
E

|Elemento|Significado|
|---|---|
|`less`|Programa utilizado para visualizar arquivos de texto.|
|`enum4_output.txt`|Arquivo que você deseja consultar.|
## O que o parâmetro `-a` procura?

Uma enumeração abrangente pode coletar informações como:

- Usuários: contas existentes no sistema.
    
- Grupos: grupos cadastrados.
    
- Compartilhamentos: pastas disponibilizadas pela rede.
    
- Informações do domínio: dados relacionados ao domínio Windows, quando disponíveis.
    
- Informações do sistema: detalhes sobre a máquina e o serviço SMB.
    
- Políticas de senha: informações sobre políticas de autenticação, quando expostas.
    

A quantidade de informações obtidas depende das permissões e da configuração do servidor.

## Entendendo o `tee`

Imagine que você execute apenas:

bash

```
enum4linux -a 192.168.56.101
```

Os resultados aparecem no terminal, mas não são automaticamente salvos em um arquivo.

Agora, com:

bash

```
| tee enum4_output.txt
```

Você consegue duas coisas simultaneamente:

Terminal

Exibe os resultados

enum4_output.txt

Salva os resultados

Uma vantagem do `tee` é permitir acompanhar a execução e guardar os resultados para análise posterior.

# ==🟢Dica: se quiser acrescentar resultados ao arquivo sem apagar o conteúdo anterior, utilize `tee -a`.==

## Resumindo

O comando inteiro significa:

"Execute uma enumeração abrangente de informações SMB/Samba no endereço IP `192.168.56.101`, mostre os resultados no terminal e salve uma cópia no arquivo `enum4_output.txt`."

Esse processo é chamado de enumeração, uma etapa importante do reconhecimento em um pentest autorizado.

E o que obtemos através dessa execução desse código é varias informações de texto mas o que nos interessa é essa parte:

![[Imagens/Rid.png]]

Os nomes de users!

Esses nomes são das contas reais do sistema remoto, e esse número rid que está aparecendo como por exemplo:

user:[user] rid:[0xbba]

O rid:[0xbba] é o identificador relativo do usuário no sistema. 

E SEMPRE QUE VERMOS NOMES COM INTERROGAÇÃO (?) [num] OU NÚMEROS GENÉRICOS COMO [48293] POR EXEMPLO, SIGNIFICA QUE SÃO BONS CANDIDATOS PARA ATAQUES.

E agora temos nossa base para ataques.

## Agora podemos dar seguimento ao ''Ataque''

Agora vamos criar uma WordList de Usuários com o comando:

```bash
echo -e "user\nmsfadmin\nservice" > smb_users.txt
```

E assim como casos REAIS, quanto mais certeiro for o levantamento de nomes de usuários que esteja correto, menor vai ser o RUIDO, maior a chance de SUCESSO e menor a chance de ser detectado.

E agora temos que criar a WordList de Senhas e é onde entra o PASSWORD SPRAYING;

 Então, ao contrário do Brute Force, que vai tentar muitas senhas em um só usuário, o PASSWORD SPRAYING por outro lado vai testar poucas senhas, mas em muitos usuários. 
 
 Isso é muito mais difícil de detectar, porque a maioria dos sistemas de segurança disparam alertas apenas após várias tentativas de senhas no mesmo usuário.

E a nossa sacada aqui vai ser usar senhas que realmente aparecem no mundo real em milhares de contas. Então a gente não vai testar várias vezes as senhas no mesmo usuário, mas em diferentes usuários.

E para isso, o que nós vamos fazer é:

Criar o arquivo senhas_spray.txt com o comando

```bash
echo -e "password\n123456\nWelcome123\nmsfadmin" > senhas_spray.txt
```

> [!Check] O Ideal no mundo real é fazermos essa lista de senhas (WordList) com base na política de senhas da empresa ou com dados de vazamentos anteriores!

E agora a gente vai fazer o ataque utilizando o MEDUSA!

## Hora de realizar o 'Ataque'

Vamos realizar o ataque com a Ferramenta MEDUSA que é uma das mais confiáveis e rápidas para ataques de autenticação.

Vamos utilizar o comando:

```bash
medusa -h 192.168.56.101 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50
```

|Parâmetro|Significado|
|---|---|
|`medusa`|Executa a ferramenta Medusa.|
|`-h`|Define o endereço IP do alvo.|
|`192.168.56.101`|IP da máquina analisada.|
|`-U`|Especifica um arquivo com nomes de usuários.|
|`smb_users.txt`|Arquivo que contém os usuários.|
|`-P`|Especifica um arquivo com senhas.|
|`senhas_spray.txt`|Arquivo que contém as senhas.|
|`-M smbnt`|Seleciona o módulo SMBNT, relacionado à autenticação SMB/NTLM.|
|`-t 2`|Define duas tarefas simultâneas por host.|
|`-T 50`|Define o limite de tarefas simultâneas globais.|

Se retornar alguma linha que contenha ''ACCOUNT FOUND'' encontramos o login e senha que vai estar na mesma linha e dependendo vai até responder se conseguiu conexão com acesso ADMIN que possivelmente é porque o usuário em questão tem esse privilégio.

E agora vamos verificar se realmente obtivemos essa conexão de fato com o seguinte comando:

```bash
smbclient -L //192.168.56.101 -U msfadmin
```

| Elemento           | Significado                                         |
| ------------------ | --------------------------------------------------- |
| `smbclient`        | Cliente utilizado para acessar serviços SMB/Samba.  |
| `-L`               | Lista os compartilhamentos disponíveis no servidor. |
| `//192.168.56.101` | Endereço IP do servidor SMB.                        |
| `-U msfadmin`      | Informa o usuário utilizado na autenticação.        |


![[Imagens/SMB Access.png]]

Prontinho, tivemos acesso.
