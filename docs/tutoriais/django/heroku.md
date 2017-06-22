# Criando Projeto django e enviando para o HEROKU

Vamos criar um projeto simples para ser utilizado com o heroku.

## Preparação do ambiente

!!! note
    Utilizando python 3.5.2

1. Crie o diretorio para o projeto e entre no mesmo `mkdir %nomeprojeto & cd $_`
2. Criar o ambiente virtual `python -m venv .venv`
3. Ativar ambiente `source .venv/bin/activate`
4. Atualizar pip `pip install --upgrade pip`
5. Instalar dependencias `pip install django dj-database-url dj-static python-decouple gunicorn psycopg2`

### Detalhando as dependências
* django: Framework para desenvolvimento web
* dj-database-url: Módulo para converter string de conexão em url
* dj-static: Módulo para servir arquivos estáticos
* python-decouple: Módulo que ajuda na parte de seguraça da aplicação (utilizar parametros via arquivo .env)
* gunicorn: Servidor web
* psycopg2: Módulo de acesso ao postgres

## Inicando o projeto

Após o ambiente preparado, vamos iniciar o projeto.  
Primeiro, vamos salvar os módulos que nosso projeto necessita:  
`pip freeze > requirements.txt`

Após isso, vamos criar nosso projeto:  
`django-admin startproject %nomeprojeto .`

!!! note
    Utilizo o ponto para não criar subpasta no projeto!!!


### Configurações do Projeto:

Primeiro, vamos ao arquivo `settings.py`. Nele, iremos informar as chaves do projeto, conexão de banco de dados, caminhos, etc.

Para aumentar a segurança, e remover desse arquivo dados de senha e chave, vamos utilizar o decouple. Para isso vamos criar o arquivo `.env` na raiz do projeto.

Nele vamos informar os seguintes itens:

```
SECRET_KEY=***********************************(Copiar do seu settings.py)
DEBUG=True
```

Agora dentro do nosso `setting.py`, vamos efetuar os imports do decouple e dj-database-url, logo após o `import os`:

```python
from decouple import config
from dj_database_url import parse as dburl
```

Agora vamos configurar nosso `SECRET_KEY` e `DEBUG` para buscar as informações do arquivo `.env`

```python
# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = config('SECRET_KEY')

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = DEBUG = config('DEBUG', default=False, cast=bool)
```

Em desenvolvimento, vamos usar o `sqlite`, então para configurá-lo, vamos criar uma variável, passando o valor padrão para caso não houver informações de conexão no arquivo `.env`

```python
# Database
# https://docs.djangoproject.com/en/1.10/ref/settings/#databases
default_db_url = 'sqlite:///' + os.path.join(BASE_DIR, 'db.sqlite3')
```

A nossa string de conexão, irá ficar dessa forma:

```python
DATABASES = {
    'default': config('DATABASE_URL', default=default_db_url, cast=dburl)
}
```

Agora vamos efetuar algumas outras configurações importantes como hosts permitidos para acessar, linguagem, timezone, local dos arquivos staticos e etc.

```python
# hosts
ALLOWED_HOSTS = [
    '*'
]
# tradução
LANGUAGE_CODE = 'pt-br'
# timestamp
TIME_ZONE = 'America/Cuiaba'
# desativa timezone
USE_TZ = False
# diretorio de arquivos staticos
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
```

Por ultimo, vamos informar as configurações de email:

```python
# Email
EMAIL_HOST = 'smtp.provedo.com.br'
EMAIL_HOST_USER = 'email@provedor.com.br'
EMAIL_HOST_PASSWORD = 'senha'
EMAIL_PORT = 587
EMAIL_USE_TLS = False
```

Se tudo estiver correto, podemos rodar as migrações e iniciar nosso projeto `python manage.py migrate` e `python manage.py runserver 0.0.0.0:8000`

Após isso, precisamos criar um usuario padrão, para podemos acessar nossa administração. Para isso, iremos utilizar o comando `python manage.py createsuperuser`:

```bash
Username (leave blank to use 'wgalleti'): admin
Email address: admin@admin.com
Password: 
Password (again): 
Superuser created successfully.
```

### Configurações do Heroku:

Feito isso, vamos adicionar algumas estruturas do HEROKU (aws):

Para isso, vamos precisar criar o app no heroku, criar os arquivos Procfile, runtime.txt.

Primeiro, precisamos informar quem vai servir os arquivos staticos no heroku, por isso utilizamos o módulo `dj-static`. Para isso, abra o arquivo `wsgi.py`:

```python
import os
from dj_static import Cling

from django.core.wsgi import get_wsgi_application

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "atividades.settings")

application = Cling(get_wsgi_application())
```

Agora, vamos criar o arquivo `Procfile`, que informa ao heroku como que as coisas vão acontecer:

```
web: gunicorn %nomeprojeto.wsgi --log-file -
```

Agora, camos criar o arquivo `runtime.txt`, que informa qual versão do python vamos utilizar:

```
python-3.5.2
```

Feito isso, vamos preparar o git.

Primeiro, iniciamos o repositório com o comando `git init` e depois vamos criar o arquivo `.gitignore`, para não enviarmos alguns arquivos.

```git
.DS_Store
.env
.idea
.venv
*.sqlite3
*pyc
__pycache__
```

Feito isso, vamos comitar os arquivos. Primeiro enviamos o .gitignore

```git
git add .gitignore
git commit -m "Add ignore config"
```

Agora vamos enviar os arquivos de configuração do heroku

```git
git add Procfile runtime.txt
git commit -m "Heroku config"
```

Agora, vamos importar o restante dos arquivos

```
git add .
git commit -m "Import project"
```

### Criando projeto no Heroku

Agora, vamos ao Heroku.

Nessa etapa, precisamos criar o app, configurar as variaveis, e enviar os arquivos.

Antes de começar, instale o heroku client para utilizar os comandos.

Vamos criar e configurar o app:

```bash
heroku create %nomeprojeto
heroku config:set SECRET_KEY='CHAVE QUE ESTA NO .env'
heroku config:set DEBUG=False
```

Feito isso, vamos enviar o projeto

```
git push heroku master
```

Pronto, projeto no ar!!!

### Conclusão

Para finalizar, precisamos informar a conexão do banco de dados. Para isso, crie a variavel `DATABASE_URL` e defina o valor que esta no site, pois o heroku criou um novo datastore para essa aplicação.

```bash
heroku config:set DATABASE_URL=postgres://...
```

Após isso, vamos rodar as migrações e criar o usuario admin:

```
heroku run python manage.py migrate
heroku run python manage.py createsuperuser
```

## Agradecimento

Feito isso, só utilizar eu app e desfrutar das funcionalidades do heroku! Valeu e até o proximo!

Agradecimentos especiais:

* Alessandro Folk
* Doc do Heroku
* Doc do Django