# Servidor Web Algodoeira

!!! note ""
    Revisado e testado em 2018-08-22

## Preparando servidor

Para essa instalação, vamos utilizar o ubuntu 18.04 server.

### Atualizando

Como root, atualize os pacotes já instalados.

- `add-apt-repository universe`
- `apt update`
- `apt upgrade`

### Criando usuário

Durante a instalação do ubuntu, criar usuário `deploy`

... VIDEO de instalação

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

Com o usuário `deploy` vamos configurar o git. Para isso vamos criar a chave ssh e configurar o usuário e email global

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

### Monitor de Rede

Para instalar o netdata, instale os seguintes pacotes:

```bash
sudo apt install zlib1g-dev uuid-dev libmnl-dev gcc make git autoconf autoconf-archive autogen automake pkg-config curl

# download it - the directory 'netdata' will be created
git clone https://github.com/firehol/netdata.git --depth=1
cd netdata

# run script with root privileges to build, install, start netdata
./netdata-installer.sh
```

### Nginx

Vamos instalar o servidor nginx:

```bash
sudo apt install nginx
```

Vamos dar uma turbinada no nginx, para evitar alguns problemas mais adiante.

No arquivo `/etc/nginx/nginx.conf` edite:

```bash
...
worker_processes 4; # Quantidade de nucleos da maquina
pid /var/run/nginx.pid; # Mudar o caminho do pid para multiplos servers
...
```

No arquivo `/etc/init.d/nginx` adicione:
```bash
NGINX_PID=/var/run/nginx.pid
```

### Oracle

!!! note "Obs"
    Essa foi a forma mais rapida de instalar e configurar o oracle no ubuntu!!!

**Referência** https://help.ubuntu.com/community/Oracle%20Instant%20Client

Baixe os rpm's do site http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html e de permissão de execução nos mesmos.

*PS*: basic, sqlplus, devel

Para efetuar a instalação, vamos usar o alien

```bash
sudo apt install alien libaio1

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

### MySQL

Vamos instalar o MySQL

```bash
sudo apt install mysql-server libmysqlclient-dev
sudo mysql_secure_installation
```

Após efetuar a configuração de segurança, vamos habilitar a conexão remota. Para isso edite o arquivo `/etc/mysql/mysql.conf.d/mysqld.cnf`
```
...
# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
bind-address            = 0.0.0.0
...
```

Reinicie o serviço `sudo /etc/init.d/mysql restart`

Agora, vamos criar um usuário para não utilizar o root:

```bash
sudo mysql -u root
> CREATE USER 'deploy'@'%' IDENTIFIED BY 'S3nh@2018';
> GRANT ALL PRIVILEGES ON * . * TO 'deploy'@'%';
```

Para facilitar a administração do banco, vamos instalar o phpMyAdmin.

Primeiro, vamos instalar e configurar algumas coisas do php =(

```
sudo apt update && sudo apt install php-fpm php-mysql
```

Agora vamos editar o nginx para entender o php.

No arquivo `/etc/nginx/sites-enabled/default` deixe dessa forma

```nginx
server {
    listen 80 default_server;

    root /var/www/html;
    index index.php index.html index.htm index.nginx-debian.html;
    server_name _;

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
    }
}
```

Agora vamos instalar o phpmyadmin `sudo apt update && sudo apt install phpmyadmin`

- Durante a instalação, ele irá pedir qual webserver você deseja selecionar para configurar automáticamente. Não selecione nenhum, e pressione <OK>
- Informe a senha para o administrador do phpadmin

Após isso, vamos criar o simbolik link `sudo ln -s /usr/share/phpmyadmin /var/www/html/phpmyadmin`

Feito isso, reinicie os serviços do nginx `sudo service nginx reload` e estara tudo funcionando


### Frontend

Vamos instalar o node 6 e npm para depois adicionar as libs de frontend

```bash
curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
sudo apt install nodejs

~$ node -v
v4.2.6
~$ npm -v
3.5.2
sudo npm install -g npm
```

### Backend

Dependencias

```bash
sudo apt install build-essential libssl-dev libffi-dev python3-dev
```

Instalando o pyenv

```bash
sudo apt install -y make build-essential libssl-dev zlib1g-dev libbz2-dev \
libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev \
xz-utils tk-dev

git clone https://github.com/pyenv/pyenv.git ~/.pyenv
echo '' >> ~/.profile
echo '# PYENV' >> ~/.profile
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.profile
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.profile
echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.profile
```

Vamos instalar o python 3.6.6

```bash
pyenv install 3.6.6
pyenv global 3.6.6
python -V
```

#### uWsgi e PipEnv

*PS: Caso esteja com virtualenv ativo, desative o mesmo `deactive`*

Instale o uwsgi

```
pip install uwsgi pipenv
```


#### Supervisor

Para não ter a necessidade de ficar levantando o uwsgi e nginx toda vez vamos automatizar essa tarefa com o supervisor.

Para instalar e configurar

```bash
sudo apt install supervisor
```

#### Redis

Rode o comando `sudo apt install redis-server redis-tools`

Após isso, teste da seginte forma

```bash
redis-cli
127.0.0.1:6379> ping
PONG
127.0.0.1:6379>
```

#### Certbot

Para colocar o certificado, vamos utilizar o CertBot

```bash
sudo apt install -y software-properties-common
sudo add-apt-repository -y ppa:certbot/certbot
sudo apt update
sudo apt install -y python-certbot-nginx 
sudo certbot --nginx
sudo certbot renew --dry-run
```

#### Nmon

Vamos instalar o monitor de hardware

```bash
sudo apt install nmon
echo 'export NMON=cmd' >> ~/.profile
source ~/.profile
```

#### VNC

Para facilitar alguns acessos remotos, vamos instalar o VNC

```bash
sudo apt install xfce4 xfce4-goodies tightvncserver
```

### Fontes

- https://devanswers.co/installing-phpmyadmin-nginx-ubuntu-18-04/
- https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-vnc-on-ubuntu-18-04
