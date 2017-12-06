# Ubuntu + Nginx + uwsgi + supervisor para Django

Primeiro vale lembrar que esse é mais um guia do que um tutorial!

!!! note ""
    Revisado e testado em 2017-09-28

## Agradecimentos

Antes de mais nada, agradecer a galera que me ajudaram muito!!! 

Valeu moçada!!!

* Alessandro Folk
* Rodrigo Rodrigues (grupo ngMasters)
* Romulo (grupo ngMasters)

## Preparando servidor

### Criando usuário

```bash
useradd -m -d /home/deploy -G adm,www-data,sudo -s /bin/bash deploy
passwd deploy
```

Para confirmar as pemissões, verifique o arquivo `/etc/passwd`

```bash
deploy:x:1001:1001::/home/deploy:/bin/bash
```

Verifique se o arquivo /home/deploy/.profile está com as seguintes informações

```bash hl_lines="11 12 13 14 15 16 17"
# ~/.profile: executed by the command interpreter for login shells.
# This file is not read by bash(1), if ~/.bash_profile or ~/.bash_login
# exists.
# see /usr/share/doc/bash/examples/startup-files for examples.
# the files are located in the bash-doc package.

# the default umask is set in /etc/profile; for setting the umask
# for ssh logins, install and configure the libpam-umask package.
#umask 022

# if running bash
if [ -n "$BASH_VERSION" ]; then
    # include .bashrc if it exists
    if [ -f "$HOME/.bashrc" ]; then
        . "$HOME/.bashrc"
    fi
fi

# set PATH so it includes user's private bin directories
PATH="$HOME/bin:$HOME/.local/bin:$PATH"
```

### Rede

!!! note "Troque o usuário"
    As configurações abaixo poderão ser feitas com usuário **deploy** pois o mesmo foi criado como adminitrador!!!

Adicionando dns e dominio de busca

```bash
sudo nano /etc/resolvconf/resolv.conf.d/tail

-- Add
nameserver 192.168.0.16
search gruposcheffer.com
```

### Git

Primeiro vamo configurar o git. Para isso vamos criar a chave ssh e configurar o usuário e email global


Vamos Gerar a Chave

```bash
ssh-keygen -t rsa -C "email@dominio.com"

Generating public/private rsa key pair.
Enter file in which to save the key (/home/deploy/.ssh/id_rsa):[Enter]
Created directory '/home/deploy/.ssh'.
Enter passphrase (empty for no passphrase):[Senha]
Enter same passphrase again:[Repete a Senha]
Your identification has been saved in /home/deploy/.ssh/id_rsa.
Your public key has been saved in /home/deploy/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:***************************** ti@gruposcheffer.com.br
```

Aplique a chave no seu servidor de versões (gitlab, ou github)

Configurando usuário global

```bash
git config --global user.name "Seu Nome"
git config --global user.email "email@dominio.com"
```

### Oracle

!!! note "Obs"
    Essa foi a forma mais rapida de instalar e configurar o oracle no ubuntu!!!

**Referência** https://help.ubuntu.com/community/Oracle%20Instant%20Client

Baixe os rpm's do site http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html e de permissão de execução nos mesmos.

*PS*: basic, sqlplus, devel

Para efetuar a instalação, vamos usar o alien

```bash
sudo apt-get install alien libaio1 libaio1:i386

sudo alien -i oracle-instantclient12.2-basic-12.2.0.1.0-1.x86_64.rpm
sudo alien -i oracle-instantclient12.2-devel-12.2.0.1.0-1.x86_64.rpm
sudo alien -i oracle-instantclient12.2-sqlplus-12.2.0.1.0-1.x86_64.rpm

 echo 'export LD_LIBRARY_PATH=/usr/lib/oracle/12.2/client64/lib/${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}' >> ~/.profile

 sudo vi /etc/ld.so.conf.d/oracle.conf && sudo chmod o+r /etc/ld.so.conf.d/oracle.conf

# Add This on /etc/ld.so.conf.d/oracle.conf
 /usr/lib/oracle/12.2/client64/lib/
sudo ldconfig

echo 'export ORACLE_HOME=/usr/lib/oracle/12.2/client64' >> ~/.profile
echo 'export TNS_ADMIN=$ORACLE_HOME/network/admin' >> ~/.profile
echo 'export PATH=$PATH:$ORACLE_HOME/bin' >> ~/.profile

source ~/.profile

mkdir -p $TNS_ADMIN
sudo touch $TNS_ADMIN/tnsnames.ora
sudo chmod o+r $TNS_ADMIN/tnsnames.ora
```

Adicione os tns no arquivo tnsnames.ora e feito!

### Frontend

Vamos instalar o node 6 e npm para depois adicionar as libs de frontend

```bash
sudo apt install python-software-properties
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt-get install nodejs

~$ node -v
v4.2.6
~$ npm -v
3.5.2
```

Agora vamos preparar algumas libs para o frontend (compilação)

```bash
sudo npm i -g bower gulp yarn @angular/cli

~$ bower -v
1.8.2
~$ yarn -v
1.1.0
~$ gulp -v
[11:08:52] CLI version 3.9.1
```

### Backend

Dependencias

```bash
sudo apt-get install build-essential libssl-dev libffi-dev python3-dev
```

Instalando o pyenv

```bash
sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev \
libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev \
xz-utils tk-dev

git clone https://github.com/pyenv/pyenv.git ~/.pyenv
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.profile
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.profile
echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.profile
```

Vamos instalar o python 3.6.2

```bash
pyenv install 3.6.2
pyenv global 3.6.2
python -V
```

#### uWsgi

*PS: Caso esteja com virtualenv ativo, desative o mesmo `deactive`*

Instale o uwsgi

```
pip install uwsgi
```

Precisamos configurar um arquivo de uwsgi para cada projeto.

Nesse exemplo vamos utilizar as seguintes propriedades:

* socket: Arquivo que será gerado o socket para comunicação 
* master: Ativar Processos master
* wsgi-file: Arquivo uwsgi do projeto django
* processes: Quantidade de Processos
* enable-threads: Ativar Threads
* threads: Quantidades de threads por processo
* max_requests: Maximo de requisições
* harakiri: 
* stats: status do socket
* stats-http: mostra status no http
* uid: usuário do SO
* gid: grupo do SO
* chmod-socket: permissão do Socket
* chdir: diretorio raiz do projeto
* home: diretorio do virutalenv
* logger: arquivo de log
* vacuum: Limpar ao fechar
* ignore-write-errors: Ignorar escrita de erros
* disable-write-exception: Desativar escrita de exceções

Nosso arquivo de exemplo que ficara no projeto1 (*raiz do projeto/uwsgi.ini*), ficará com a seguinte estrutura:

```ini
[uwsgi]
socket = projeto1.sock
master = true
wsgi-file = projeto1/wsgi.py
processes = 2
enable-threads = true
threads = 10
max_requests = 100
harakiri = 60
stats = 127.0.0.1:9191
stats-http = true
uid = www-data
gid = www-data
chmod-socket = 666
chdir = /home/deploy/projects/projeto1
home = /home/deploy/projects/projeto1/.venv/
logger = file:/home/deploy/projects/projeto1/logs/uwsgi.log
vacuum = true
ignore-write-errors = true
disable-write-exception = true
```

Importante lembra que em `stats` a porta não pode repetir para quando configurar uma outra aplicação

**uwsgi_params**

Na documentação oficial, precisamos definir alguns parametros para integrar com nginx https://github.com/nginx/nginx/blob/master/conf/uwsgi_params

Vamos criar o arquivo uwsgi_params.conf na raiz do projeto com o seguinte conteúdo:

```bash
uwsgi_param  QUERY_STRING       $query_string;
uwsgi_param  REQUEST_METHOD     $request_method;
uwsgi_param  CONTENT_TYPE       $content_type;
uwsgi_param  CONTENT_LENGTH     $content_length;

uwsgi_param  REQUEST_URI        $request_uri;
uwsgi_param  PATH_INFO          $document_uri;
uwsgi_param  DOCUMENT_ROOT      $document_root;
uwsgi_param  SERVER_PROTOCOL    $server_protocol;
uwsgi_param  REQUEST_SCHEME     $scheme;
uwsgi_param  HTTPS              $https if_not_empty;

uwsgi_param  REMOTE_ADDR        $remote_addr;
uwsgi_param  REMOTE_PORT        $remote_port;
uwsgi_param  SERVER_PORT        $server_port;
uwsgi_param  SERVER_NAME        $server_name;
```

Para testar se esta tudo funcionando, vamos rodar o seguinte comando:

```
cd %diretorio_projeto%
uwsgi --ini uwsgi.ini
```

#### nginx

Vamos instalar o servidor nginx:

```bash
sudo apt-get install nginx
```

Após instalado, acesse o diretorio `cd /etc/nginx/sites-avaliable`

Vamos criar a configuração `touch projeto1.conf`

* Servidor web para frontend na porta 80
* Servidor web para backend mapeando as rotas /statics e / utilizando socket para integrar com uwsgi

Importante ter o dns configurado para utilização de nomes (exemplo: projeto1.seudominio.com)

```bash
# projeto1.conf

# Frontend server
server {
    # porta
    listen 80;
    # nome DNS
    server_name projeto1.seudominio.com;
    # caminho raiz
    root /home/deploy/projects/projeto1/frontend/dist;
    # index file
    index index.html;
    # locais
    location / {
        try_files $uri $uri/ =404;
    }
}

# Backend server
server {
    # porta
    listen 8000;
    # nome DNS
    server_name projeto1.seudominio.com;
    # charset
    charset utf-8;
    # configs
    client_body_in_file_only clean;
    client_body_buffer_size 32M;
    sendfile on;
    keepalive_timeout 120;
    send_timeout 300s;
    client_max_body_size 300M;

    # local /static
    location /static {
        alias /home/deploy/projects/projeto1/static;
    }

    # local /
    location / {
        # socket
        uwsgi_pass projeto1;
        # configs
        proxy_read_timeout 40s;
        proxy_send_timeout 40s;
        uwsgi_ignore_client_abort on;
        # params
        include     /home/deploy/projects/projeto1/uwsgi_params;
    }
}
# Reader socket
upstream projeto1 {
    server unix:///home/deploy/projects/projeto1/projeto1.sock;
}
```

Feito isso vamos ativar o site e reiniciar o nginx

```bash
sudo ln -s /etc/nginx/sites-avaliable/projeto1.conf /etc/nginx/sites-enable/
sudo /etc/init.d/nginx restart
```

Para testar, abra o navegador e acesse o endereço `projeto1.seudominio.com:8000` e você ira recever um erro 502 Bad Gateway

Isso ocorre porque o socket do uwsgi não está em execução. Vamos no projeto e rodar o mesmo.

```bash
cd raiz_projeto
uwsgi --ini uwsgi.ini
```

Acesse novamente e o site estara funcionando!

#### Supervisor

Para não ter a necessidade de ficar levantando o uwsgi e nginx toda vez vamos automatizar essa tarefa com o supervisor.

Para instalar e configurar

```bash
sudo apt-get install supervisor
sudo touch /etc/supervisor/conf.d/projeto1.conf
```

Dentro do arquivo, vamos colocar a seguinte estrutura

```bash
[program:projeto1]
user = deploy
command = /home/deploy/.pyenv/shims/uwsgi --ini /home/deploy/projects/projeto1/uwsgi.ini
autostart = true
environment=LD_LIBRARY_PATH='/usr/lib/oracle/12.2/client64/lib/${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}',ORACLE_HOME='/usr/lib/oracle/12.2/client64/',TNS_ADMIN='$ORACLE_HOME/network/admin'

[program:nginx]
command = /usr/sbin/nginx
user = root
autostart = true
```

Finalizado, vamos instalar o programa no supervisor

```bash
sudo supervisorctl reread
sudo supervisorctl start all
```
Feito isso, sitema configurado e pronto!

## Links

* http://uwsgi-docs.readthedocs.io/en/latest/
* https://docs.djangoproject.com/pt-br/1.11/howto/deployment/wsgi/uwsgi/
* https://www.digitalocean.com/community/tutorials/how-to-serve-django-applications-with-uwsgi-and-nginx-on-ubuntu-16-04