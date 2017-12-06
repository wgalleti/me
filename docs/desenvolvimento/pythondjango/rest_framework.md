# Usando Django Rest Framework

!!! note
    Usando python 3.5.2

## Instalação

Com seu projeto funcionando, vamos instalar os pacotes

`pip install django-rest-auth djangorestframework django-filter`

### Detalhando as dependências
* **django-cors-headers**: Módulo para permitir acesso via cors (cross domain origin)
* **django-filter**: Módulo para utilizar filtros no django
* **django-rest-auth**: Módulo para autenticação via REST
* **djangorestframework**: Módulo para desenvolver apis REST

### Configurando o projeto

Vamos adicionar o `'rest_framework'`, `'rest_framework.authtoken'`, `'rest_auth'` ao **INSTALLED_APPS**, e criamos o a seguinte variavel no `settings.py`

Agora, vamos definir algumas configurações padrões, como autenticação, parser de class, formatação e paginação

```python
REST_FRAMEWORK = {
    'DEFAULT_FILTER_BACKENDS': (
        'django_filters.rest_framework.DjangoFilterBackend',
    ),
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework.authentication.TokenAuthentication',
    ),
    'DEFAULT_PARSER_CLASSES': (
        'rest_framework.parsers.JSONParser',
    ),
    'COERCE_DECIMAL_TO_STRING': False,
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.LimitOffsetPagination',
}
```

### CorsHeaders
Como vamos transformar nosso projeto em uma API RestFull, precisamos configurar as permissões de CORS (*cross domain origins*) e para isso vamos utilizar a biblioteca `cors-headers`

Primeiro, vamos instalar o módulo `pip install django-cors-headers`

Adicionando o `'corsheaders'` ao **INSTALLED_APPS**, e criamos o a seguinte variavel

```python
# Cors Headers
CORS_ORIGIN_ALLOW_ALL = True
```

!!! note
    Ou você pode configurar os hosts que deseja permitir a conexão. [Veja a Documentação oficial](https://github.com/ottoyiu/django-cors-headers)

Em `MIDDLEWARE`, antes de `'django.middleware.common.CommonMiddleware'`, adicione `'corsheaders.middleware.CorsMiddleware',`

Feito isso, só utilizar as facilidades do `django-rest-framework`

## Exemplo de uso

Vamos criar um exemplo de api para um crud da tabela de `pessoa`

Nesse exemplo, vamos usar o cliente para consumir o restfull o [postman](https://www.getpostman.com/)

### Modelo

Vamos criar a classe `pessoa`

```python
class Pessoa(models.Model):
    """Modelo de Pessoas"""

    nome = models.CharField(max_length=50)
    email = models.EmailField(unique=True)
    telefone = models.CharField(max_length=15)
    tem_whats = models.BooleanField(default=False)

    def __str__(self):
        return self.nome

    class Meta:
        ordering = ('nome', 'email')
```

Criamos e aplicamos as migrações

```bash
(.venv) wgalleti@mpGalleti api (master) $ python manage.py makemigrations
Migrations for 'core':
  intranet/core/migrations/0002_pessoa.py
    - Create model Pessoa

(.venv) wgalleti@mpGalleti api (master) $ python manage.py migrate
Operations to perform:
  Apply all migrations: core
Running migrations:
  Applying core.0002_pessoa... OK
```

### Serializer
Feito isso, vamos necessitar de uma classe para serializar esses dados, então adicionaremos o arquivo `serializers.py` no seu `app` e nele vamos criar a classe para serializar pessoas

```python
from rest_framework import serializers

from intranet.core.models import Pessoa
        
class PessoaSerializer(serializers.ModelSerializer):
    """Serializer para modelo de Pessoas"""
    
    class Meta:
        model = Pessoa
        fields = '__all__'
```

Nessa classe, estamos dizendo que ela herda de `serializer.ModelSerializer`, que vamos usar o model `Pessoa` e vamos utilizar todos os campos

### View

Feito isso, vamos criar o nosso view set, dentro de `views.py`

```python
from rest_framework import viewsets, permissions, response
from rest_framework.filters import DjangoFilterBackend

from intranet.core.serializers import *
    
class PessoaViewSet(viewsets.ModelViewSet):
    """ViewSet para modelo de Pessoa"""
    
    serializer_class = PessoaSerializer
    queryset = Pessoa.objects.all()
    filter_backends = (DjangoFilterBackend,)
    filter_fields = ('nome', 'email')
    permission_classes = (permissions.DjangoModelPermissions,)
```

Nessa classe, estamos dizendo que ela herda de `viewset.ModelViewSet`, que vamos utilizar o serializer_class a classe que criamos anteriomente `PessoaSerializer` a nossa pesquisa será através o queryset `Pessoa.objects.all()`.

Definimos que o método de filtro é o `DjangoFilterBackend` para podermos interagir com a url. Exemplo: Acessar http://url/api/pessoa/?nome=William

Definimos que os campos que poderão ser filtrados são **nome** e **email**

E por ultimo, definimos que as permissões seguirão as regras das Permissões de Modelo do Django `DjangoModelPermissions`

### Url
Feito isso, basta registrar a url para usar!

No nosso `url.py` vamos efetuar as seguinte configurações

```python
from django.conf.urls import url, include

from rest_framework import routers

from intranet.core.views import *

router = routers.DefaultRouter(trailing_slash=True)
router.register(r'pessoa', PessoaViewSet)

urlpatterns = [
    url(r'^api/', include(router.urls)),
]
```

Primeiro, criamos uma variavel `router` recebendo informações de `routers.DefaultRouter`. Ali definimos `trailing_slash=True` para utilização em frameworks como *angularjs*, *vue* para que os mesmos forçem a preencher a `/` no final do metodo

Após isso, registramos na url `pessoa` o nosso viewSet

Agora em nossas rotas mesmo, adicionamos as rotas que nosso `router` irá gerar!

### Acessando

Para testar tudo, vamos rodar nosso servidor `python manage.py runserver` e verificar a url http://localhost:8000/core/api

Se aparecer a seguinte tela, é porque está tudo funcionando

![img](rest_01.png)

!!! note
    Dessa forma, você tera acesso aos metodos GET, POST, PUT, DELETE para manipular os arquivos


Valeu!