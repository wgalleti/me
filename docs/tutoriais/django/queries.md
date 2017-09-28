# Django Queries

O que aprendi fazendo queries no Django?

## Filtros

| Expressão | Resultado |
| - | - |
| `a__exact=value` | Combinação exata |
| `a__exact=None` | IS NULL |
| `x__iexact=value` | Combinação string ignorando maiúscula e minúscula |
| `a__isnull=True` | IS NULL | 
| `a__isnull=False` | IS NOT NULL |
| `a__contains=value` | Contém na string |
| `a__icontains=value` | Contém na string ignorando maiúscula e minúscula |
| `a__contains=value` | Combinação exata |
| `a__exact=value` | Combinação exata |

