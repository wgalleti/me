## Vamos conectar ao dominio `williamgalleti.com`

## Instalação

```python
pip install django-python3-ldap
```

## Settings

Edite o arquivo settings.py e:
* Adicione em **INSTALLED_APPS** `'django_python3_ldap',`
* Adicione as Seguintes variáveis
```python
# Ldap Autentication
LDAP_AUTH_URL = 'ldap://192.168.0.10'
LDAP_AUTH_USE_TLS = False
LDAP_AUTH_SEARCH_BASE = 'dc=williamgalleti,dc=com'
LDAP_AUTH_OBJECT_CLASS = 'person'
LDAP_AUTH_USER_FIELDS = {
    'username': 'sAMAccountName',
    'first_name': 'givenName',
    'last_name': 'sn',
    'email': 'mail',
}
LDAP_AUTH_USER_LOOKUP_FIELDS = ('username',)
LDAP_AUTH_CLEAN_USER_DATA = 'django_python3_ldap.utils.clean_user_data'
LDAP_AUTH_SYNC_USER_RELATIONS = 'django_python3_ldap.utils.sync_user_relations'
LDAP_AUTH_FORMAT_SEARCH_FILTERS = 'django_python3_ldap.utils.format_search_filters'
LDAP_AUTH_FORMAT_USERNAME = 'django_python3_ldap.utils.format_username_active_directory'
LDAP_AUTH_ACTIVE_DIRECTORY_DOMAIN = 'WILLIAMGALLETI'
LDAP_AUTH_CONNECTION_USERNAME = 'usuario'
```
* Ajustar o django para encontrar a autenticação pelo LDAP. Adicione a tupla:
```
# Autenticações
AUTHENTICATION_BACKENDS = (
    'django.contrib.auth.backends.ModelBackend',
    'django_python3_ldap.auth.LDAPBackend',
)
```

## Conclusão

Caso as informações de conexão acima estiverem corretas, basta logar com o usuário do LDAP na sua aplicação

*OBS:* Caso tentar logar no admin, vai receber um aviso de falta de permissão. Quando o ldap recebe o usuário, ele vem sem permissões. Para promover o usuário, utilize o comando `python manage.py ldap_promote NOMEDOUSUARIO`
*OBS2:* Caso queira receber todos os usuários (Não recomendo), execute o comando `python manage.py ldap_sync_users`

## Referências

https://github.com/etianen/django-python3-ldap
