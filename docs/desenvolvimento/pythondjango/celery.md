# Usando celery com projetos Django

Nesse documento, vou tentar explicar como utilizar o Celery + Celery Beat com Django e Redis.

## Instalação

> Utilizei pipenv

```
pipenv install celery redis django-celery-beat django-celery-results
```

Instale do redis server e redis client [link](https://redis.io/)


## Configuração

No `settings.py` do seu projeto, vamos adicionar as seguintes linhas

```python
INSTALLED_APPS = [
    ...
    'django_celery_beat',
    'django_celery_results',
]

# Celery Setup
BROKER_URL = 'redis://localhost:6379'
CELERY_RESULT_BACKEND = 'redis://localhost:6379'
CELERY_ACCEPT_CONTENT = ['application/json']
CELERY_TASK_SERIALIZER = 'json'
CELERY_RESULT_SERIALIZER = 'json'
CELERY_TIMEZONE = 'America/Cuiaba'
CELERY_BEAT_SCHEDULE = {}
```

Vamos criar um arquivo chamado `celery.py` no mesmo diretório do seu `settings.py`.

```python
from __future__ import absolute_import
import os
from celery import Celery
import settings

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'config.settings')
app = Celery('<project-name>')
app.config_from_object('django.conf:settings')
app.autodiscover_tasks(lambda: settings.INSTALLED_APPS)

@app.task(bind=True)
def debug_task(self):
    print('Request: {0!r}'.format(self.request))
```

E para que o celery inicie junto a sua aplicação, vamos adicioná-lo ao `__init__.py`.

```python
from __future__ import absolute_import, unicode_literals

from .celery import app as celery_app

__all__ = ('celery_app',)
```

## Tarefas

Agora com tudo configurado, vamos criar dentro dos nossos apps as tarefas que necessitamos. Para isso, crie o arquivo `tasks.py` dentro do app.

```python
from celery.task import task


@task(name="sum_two_numbers")
def add(x, y):
    return x + y
```

Para executar agendar a execução da tarefa, vamos ao utilizar o shell_plus, uma funcionalidade do [django-extensions](https://github.com/django-extensions/django-extensions).

```bash
python manage.py shell_plus
> from app.tasks import add
> add.delay(5, 5)
<AsyncResult: 747226b7-ef82-452c-a0d6-fbb09ec305e2>
```

Com isso o agendamento da tarefa foi efetuado com sucesso. Para que ela seja executada, precisamos levantar o worker que ira executar o trabalho.

```bash
celery -A <project-name> worker -l info
```

PS: Caso esteja no windows, algumas coisas são estranhas, então, precisamos passar um pool para que as coisas voltem ao normal (hahahahahha). Para isso o comando será:

```bash
celery -A <project-name> worker -l info -P eventlet
```

Se tudo deu certo, o resultado será algo do tipo

```bash
celery -A config worker -l info -P eventlet

 -------------- celery@CBAINFNB010669 v4.2.1 (windowlicker)
---- **** -----
--- * ***  * -- Windows-10-10.0.17134-SP0 2018-08-15 07:44:02
-- * - **** ---
- ** ---------- [config]
- ** ---------- .> app:         celery2:0x1d34debbe10
- ** ---------- .> transport:   redis://localhost:6379//
- ** ---------- .> results:     redis://localhost:6379/
- *** --- * --- .> concurrency: 4 (eventlet)
-- ******* ---- .> task events: OFF (enable -E to monitor tasks in this worker)
--- ***** -----
 -------------- [queues]
                .> celery           exchange=celery(direct) key=celery


[tasks]
  . config.celery.debug_task

[2018-08-15 07:44:02,247: INFO/MainProcess] Connected to redis://localhost:6379//
[2018-08-15 07:44:02,260: INFO/MainProcess] mingle: searching for neighbors
[2018-08-15 07:44:03,313: INFO/MainProcess] mingle: all alone
[2018-08-15 07:44:03,380: INFO/MainProcess] celery@CBAINFNB010669 ready.
[2018-08-15 07:44:03,380: INFO/MainProcess] pidbox: Connected to redis://localhost:6379//.
[2018-08-15 07:47:19,141: INFO/MainProcess] Received task: sum_two_numbers[739026d6-7f78-4500-8862-6b234831252b]
[2018-08-15 07:47:19,148: INFO/MainProcess] Task sum_two_numbers[739026d6-7f78-4500-8862-6b234831252b] succeeded in 0.0s: 10
```

## Tarefas periódicas

Agora, caso você precise agendar tarefas para ser executadas em determinados horarios do dia, ou dia do mês ou até mesmo a cada minuto podemos criar `periodic_tasks`.

Seguindo nosso exemplo, o arquivo `tasks.py`

```python
from celery.schedules import crontab
from celery.task import task, periodic_task


@task(name="sum_two_numbers")
def add(x, y):
    return x + y


@periodic_task(run_every=(crontab(minute='*/1')), name="periodic_sum_two_numbers", ignore_result=True)
def some_task():
    return 10 + 20
```

Importante lembrar que o `celery-beat` não vai executar a tarefa, e sim, enviar a mesma ao worker, ou seja, ele só vai avisar que tem a tarefa para ser executada.

Então para isso, ele precisa de um serviço parecido com o worker, que podemos levantar com o comando.

```bash
celery -A <project-name> beat -l info
```

Se tudo der certo, o resultado será algo do tipo:

```bash
celery -A config beat -l info
celery beat v4.2.1 (windowlicker) is starting.
__    -    ... __   -        _
LocalTime -> 2018-08-15 07:54:59
Configuration ->
    . broker -> redis://localhost:6379//
    . loader -> celery.loaders.app.AppLoader
    . scheduler -> celery.beat.PersistentScheduler
    . db -> celerybeat-schedule
    . logfile -> [stderr]@%INFO
    . maxinterval -> 5.00 minutes (300s)
[2018-08-15 07:54:59,675: INFO/MainProcess] beat: Starting...
[2018-08-15 07:55:01,073: INFO/MainProcess] Scheduler: Sending due task periodic_sum_two_numbers (periodic_sum_two_numbers)

(.venv) E:\gruposcheffer\gitlab\gsCotton\fazenda\api>celery -A config beat -l info
celery beat v4.2.1 (windowlicker) is starting.
__    -    ... __   -        _
LocalTime -> 2018-08-15 07:55:11
Configuration ->
    . broker -> redis://localhost:6379//
    . loader -> celery.loaders.app.AppLoader
    . scheduler -> celery.beat.PersistentScheduler
    . db -> celerybeat-schedule
    . logfile -> [stderr]@%INFO
    . maxinterval -> 5.00 minutes (300s)
[2018-08-15 07:55:11,137: INFO/MainProcess] beat: Starting...
[2018-08-15 07:56:01,043: INFO/MainProcess] Scheduler: Sending due task periodic_sum_two_numbers (periodic_sum_two_numbers)
[2018-08-15 07:57:00,000: INFO/MainProcess] Scheduler: Sending due task periodic_sum_two_numbers (periodic_sum_two_numbers)
```

E o resultado do worker será:

```bash
[2018-08-15 07:56:02,188: INFO/MainProcess] Received task: periodic_sum_two_numbers[bf0a2c6e-f68c-4746-bc1d-c3b50c9b6dc1]
[2018-08-15 07:56:02,189: INFO/MainProcess] Task periodic_sum_two_numbers[bf0a2c6e-f68c-4746-bc1d-c3b50c9b6dc1] succeeded in 0.0s: 30
[2018-08-15 07:57:00,011: INFO/MainProcess] Received task: periodic_sum_two_numbers[32f5928f-9acf-445d-ae15-bc37c1592496]
[2018-08-15 07:57:00,013: INFO/MainProcess] Task periodic_sum_two_numbers[32f5928f-9acf-445d-ae15-bc37c1592496] succeeded in 0.01599999999598367s: 30
```