# Ubuntu + Nginx + PHP + PHP-fpm + Oracle

Primeiro vale lembrar que esse é mais um guia do que um tutorial!

!!! note ""
    Revisado e testado em 2017-12-06

## Preparando servidor

### Criando usuário

!!! note ""
    Revisado e testado em 2017-12-06

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

LDConfig

```bash
sudo vi /etc/ld.so.conf.d/oracle.conf
```
Adicione o camino `/usr/lib/oracle/12.2/client64/lib` e salve o arquivo. Após isso, recarregue as configurações com `sudo ldconfig`

Adicione os tns no arquivo tnsnames.ora e feito!

### Backend

Dependencias

```bash
sudo add-apt-repository ppa:ondrej/php
sudo apt-get install software-properties-common
sudo apt-get update
```


#### PHP 5.6

Instalando o PHP

```bash
sudo apt-get install php5.6 php5.6-dev php5.6-dev php5.6-fpm
```

Vamos deixar o serviço do `fpm` para inicializar junto com o SO

```bash

```

Agora vamos instalar o oci8

!!! note "Recomentação da pecl"
    Use the OCI8 extension to access Oracle Database. Use 'pecl install oci8' to install for PHP 7. Use 'pecl install oci8-2.0.12' to install for PHP 5.2 - PHP 5.6. Use  pecl install oci8-1.4.10' to install for PHP 4.3.9 - PHP 5.1. The OCI8 extension can be linked with Oracle client libraries from Oracle Database 12, 11, or 10.2. These libraries are found in your database installation, or in the free Oracle Instant Client from http://www.oracle.com/technetwork/database/features/instant-client/.  
    Oracle's standard cross-version connectivity applies. For example, PHP OCI8 linked with Instant Client 11.2 can connect to Oracle Database 9.2 onward. See Oracle's note "Oracle Client / Server Interoperability Support" (ID 207303.1) for details.

```bash
sudo pecl download oci8-1.4.10
tar zxvf oci8-1.4.10.tgz
cd oci8-1.4.10
phpize
./configure --with-oci8=instantclient,$ORACLE_HOME/lib
sudo make install
```

Caso tudo correto, vamos habilitar a lib no `php-cli` e  `php-fpm`

Primeiro, vamos criar a configuração da lib
```bash
cd /etc/php/5.6/mods-available
sudo vi oci.ini
```

Dentro do arquivo, informe o conteudo:
```
extension=oci8.so
```

Salve o arquivo e vamos ativar as extensões

##### PHP-Cli

```bash
cd /etc/php/5.6/cli/conf.d/
sudo ln -s /etc/php/5.6/mods-available/oci.ini 20-oci.ini
```

Teste com o comando para verificar se a extensão esta ativa

```bash
php -i | grep oci8
```

##### PHP-Fpm

```bash
cd /etc/php/5.6/fpm/conf.d/
sudo ln -s /etc/php/5.6/mods-available/oci.ini 20-oci.ini
```

#### nginx

Vamos instalar o servidor nginx:

```bash
sudo apt-get install nginx
```

Após instalado, acesse o diretorio `cd /etc/nginx/sites-avaliable`

Vamos criar a configuração `touch php.conf`


##### PHP 

Importante ter o dns configurado para utilização de nomes (exemplo: php.seudominio.com)

```bash
# php.conf

server {
    listen 80;
    server_name php.seudominio.com;

    root /home/deploy/projects/php;

    index index.php index.html;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php5.6-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;                
    }
}
```

##### CODEIGNITER 

Importante ter o dns configurado para utilização de nomes (exemplo: php.seudominio.com)

```bash
# php.conf

server {
    listen 80;
    server_name php.seudominio.com;

    root /home/deploy/projects/php;

    index index.php index.html;

    location / {
		try_files $uri $uri/ /index.php;
	
	    location = /index.php {

			fastcgi_pass unix:/var/run/php/php5.6-fpm.sock;
			fastcgi_param SCRIPT_FILENAME /home/deploy/projects/php$fastcgi_script_name;
			include fastcgi_params;
		}
	}

}
```

Feito isso vamos ativar o site e reiniciar o nginx

```bash
sudo ln -s /etc/nginx/sites-avaliable/php.conf /etc/nginx/sites-enable/
sudo /etc/init.d/nginx restart
```

Para testar, abra o navegador e acesse o endereço `php.seudominio.com`


## Links

* https://www.digitalocean.com/community/tutorials/como-instalar-linux-nginx-mysql-php-pilha-lemp-no-ubuntu-16-04-pt
* https://askubuntu.com/questions/927026/install-php-fpm-5-6-on-ubuntu-xenial-16-04
* https://rosemberg.net.br/pt/instalando-pdo-oracle-e-oci8-do-php7-no-ubuntumint-oracle-11-2/