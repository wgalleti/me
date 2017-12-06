# Criando Projeto django e enviando para o HEROKU

Vamos criar um projeto simples para ser utilizado com o heroku.

## Preparação do ambiente

!!! note
    Utilizando python 3.6.2

1. Crie o diretorio para o projeto e entre no mesmo `mkdir %nomeprojeto & cd $_`
2. Criar o ambiente virtual `python -m venv .venv`
3. Ativar ambiente `source .venv/bin/activate`
4. Instalar dependencias `pip install django dj-database-url dj-static python-decouple gunicorn psycopg2`

### Detalhando as dependências
* django: MAGIC
* dj-database-url: Módulo para converter url em dicionario de conexão
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

Para aumentar a segurança, e remover desse arquivo dados de senha e chave, vamos utilizar o python-decouple.  
Para isso vamos criar o arquivo `.env` na raiz do projeto.

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
DEBUG = config('DEBUG', default=False, cast=bool)
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
```

É importante também configurar onde ficarão os arquivos estático

```python

# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/1.11/howto/static-files/

STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
```

Se tudo estiver correto, podemos rodar as migrações e iniciar nosso projeto `python manage.py migrate` e `python manage.py runserver 0.0.0.0:8000`

!!! note
    Se estiver no Unix, podemos criar um alias para facilitar a vida `alias manage='python $VIRTUAL_ENV/../manage.py'` e usar o comando `manage migrate` e `manage runserver`
    No windows, crie um arquivo manage.bat em .venv\scritps com o seguinte conteúdo `@python "%VIRTUAL_ENV%\..\manage.py" %*` 



### Configurações do Heroku:

Feito isso, vamos adicionar algumas estruturas do HEROKU:

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
python-3.6.2
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

Agora, vamos importar o restante dos arquivos

```
git add .
git commit -m "Import project"
```

### Criando projeto no Heroku

Agora, vamos ao Heroku.

Nessa etapa, precisamos criar o app, configurar as variaveis, e enviar os arquivos.

Antes de começar, instale o heroku client para utilizar os comandos.

!!! note
    Caso tenha duvidas, siga https://devcenter.heroku.com/articles/heroku-cli


Vamos criar e configurar o app:

```bash
heroku create %nomeprojeto --buildpack heroku/python
heroku addons:create heroku-postgresql:hobby-dev
heroku pg:promote
heroku plugins:install heroku-config
heroku config:push
```

Feito isso, vamos enviar o projeto

```
git push heroku master
```

Caso, tudo correto, projeto no ar!!!

### Conclusão

Para finalizar, precisamos rodar as migrações e criar o usuario admin:

```
heroku run python manage.py migrate
heroku run python manage.py createsuperuser
```

## Agradecimento

Feito isso, só utilizar eu app e desfrutar das funcionalidades do heroku! Valeu!

Agradecimentos especiais:

* Henrique Bastos (https://welcometothedjango.com.br/)
* Alessandro Folk
* Doc do Heroku
* Doc do Django