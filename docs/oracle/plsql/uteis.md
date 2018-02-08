## Exportando dados com spool

Vamos criar um script sql e um bat para exportar dados de uma determinada consulta para CSV (Grandes Volumes)

Primeiro, vamos preparar a consulta:

### Preparando a consulta

Vou usar o seguinte comando de exemplo:

```sql
SELECT A.CODEMP,
       A.CODFIL,
       A.NUMOCP,
       A.CODPRO,
       A.CODFAM,
       A.PREFOR,
       A.QTDFOR,
       A.QTDCAN,
       A.QTDABE,
       A.QTDREC,
       A.QTDPED,
       A.VLRFIN,
       A.PREUNI,
       A.CODMOE,
       A.SITIPO,
       TO_CHAR(A.DATENT,'DD/MM/YYYY') DATENT,
       TO_CHAR(A.DATGER,'DD/MM/YYYY') DATGER,
       A.PERDSC,
       A.VLRDSC,
       A.CODDEP,
       A.CODCCU,
       A.CODEMP,
       A.CODPRO,
       B.DESPRO,
       C.CODFOR
  FROM SAPIENS.E420IPO A,
       SAPIENS.E075PRO B,
       SAPIENS.E420OCP C
WHERE C.CODEMP = A.CODEMP
  AND C.CODFIL = A.CODFIL
  AND C.NUMOCP = A.NUMOCP
  AND B.CODEMP = A.CODEMP
  AND B.CODPRO = A.CODPRO
  AND A.DATGER BETWEEN '01/02/2017' AND '10/02/2017'
  AND A.SITIPO IN (1, 2, 3, 8, 9)
```

Primeiro, precisamos concatenar as colunas, transformandoas em uma única coluna

```sql
SELECT A.CODEMP||';'||
       A.CODFIL||';'||
       A.NUMOCP||';'||
       A.CODPRO||';'||
       A.CODFAM||';'||
       A.PREFOR||';'||
       A.QTDFOR||';'||
       A.QTDCAN||';'||
       A.QTDABE||';'||
       A.QTDREC||';'||
       A.QTDPED||';'||
       A.VLRFIN||';'||
       A.PREUNI||';'||
       A.CODMOE||';'||
       A.SITIPO||';'||
       TO_CHAR(A.DATENT,'DD/MM/YYYY')||';'||
       TO_CHAR(A.DATGER,'DD/MM/YYYY')||';'||
       A.PERDSC||';'||
       A.VLRDSC||';'||
       A.CODDEP||';'||
       A.CODCCU||';'||
       A.CODEMP||';'||
       A.CODPRO||';'||
       B.DESPRO||';'||
       C.CODFOR
  FROM ...
```

Agora vamos criar uma consulta ficticia para gerar os nomes das colunas

```sql
SELECT 'CODEMP;CODFIL;NUMOCP;CODPRO;CODFAM;PREFOR;QTDFOR;QTDCAN;QTDABE;QTDREC;QTDPED;VLRFIN;PREUNI;CODMOE;SITIPO;DATENT;DATGER;PERDSC;VLRDSC;CODDEP;CODCCU;CODEMP;CODPRO;DESPRO;CODFOR' FROM DUAL
```

Vamos juntar as duas consultas e definir alguns parametros para o SQL em um arquivo com o nome `consulta.sql`

```sql
set colsep ';'
set echo off
set feedback off
set linesize 1000
set pagesize 0
set sqlprompt ''
set trimspool on
set headsep off

spool EXPORT.CSV

SELECT 'CODEMP;CODFIL;NUMOCP;CODPRO;CODFAM;PREFOR;QTDFOR;QTDCAN;QTDABE;QTDREC;QTDPED;VLRFIN;PREUNI;CODMOE;SITIPO;DATENT;DATGER;PERDSC;VLRDSC;CODDEP;CODCCU;CODEMP;CODPRO;DESPRO;CODFOR' FROM DUAL
UNION ALL
SELECT A.CODEMP||';'||
       A.CODFIL||';'||
       A.NUMOCP||';'||
       A.CODPRO||';'||
       A.CODFAM||';'||
       A.PREFOR||';'||
       A.QTDFOR||';'||
       A.QTDCAN||';'||
       A.QTDABE||';'||
       A.QTDREC||';'||
       A.QTDPED||';'||
       A.VLRFIN||';'||
       A.PREUNI||';'||
       A.CODMOE||';'||
       A.SITIPO||';'||
       TO_CHAR(A.DATENT,'DD/MM/YYYY')||';'||
       TO_CHAR(A.DATGER,'DD/MM/YYYY')||';'||
       A.PERDSC||';'||
       A.VLRDSC||';'||
       A.CODDEP||';'||
       A.CODCCU||';'||
       A.CODEMP||';'||
       A.CODPRO||';'||
       B.DESPRO||';'||
       C.CODFOR
  FROM SAPIENS.E420IPO A,
       SAPIENS.E075PRO B,
       SAPIENS.E420OCP C
WHERE C.CODEMP = A.CODEMP
  AND C.CODFIL = A.CODFIL
  AND C.NUMOCP = A.NUMOCP
  AND B.CODEMP = A.CODEMP
  AND B.CODPRO = A.CODPRO
  AND A.DATGER BETWEEN '01/02/2017' AND '10/02/2017'
  AND A.SITIPO IN (1, 2, 3, 8, 9);

spool off
exit
```

Agora, basta chamar o arquivo para ser executado pelo `sqlplus`:

```bash
sqlplus usuario/senha@tns @consulta
```

Para facilitar, crie um arquivo `.bat` para executar a ação.