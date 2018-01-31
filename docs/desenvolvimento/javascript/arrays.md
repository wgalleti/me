# Dicas uteis para manipulação de arrays

## Distinct

```typescript
constant data = [{
        "COD_EMPR": 2,
        "UNIDADE": "Rafaela",
        "DATA": "2017-08-13",
        "QTD_FARDOS": 258,
        "PESO_FARDOS": 53859.54
    },{
        "COD_EMPR": 5,
        "UNIDADE": "Tres Lagoas",
        "DATA": "2017-08-13",
        "QTD_FARDOS": 659,
        "PESO_FARDOS": 149127.67
    },{
        "COD_EMPR": 2,
        "UNIDADE": "Rafaela",
        "DATA": "2017-08-14",
        "QTD_FARDOS": 460,
        "PESO_FARDOS": 99821.9
    },{
        "COD_EMPR": 5,
        "UNIDADE": "Tres Lagoas",
        "DATA": "2017-08-14",
        "QTD_FARDOS": 848,
        "PESO_FARDOS": 191857.24
}];

const unidades = data.map(item => {
	return item.UNIDADE;
}).filter((v, i, a) => a.indexOf(v) === i);
```

