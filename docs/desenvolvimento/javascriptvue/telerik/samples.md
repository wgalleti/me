# Exemplos de Uso

## Customizar coluna

Existe casos onde precisamos adicionar templates a uma coluna. No caso do kendo-grid, vamos usar o atributo `template` do kendo-column.

**OBS:** Estou usando o vuetify

Exemplo:

![Screenshot](/img/grid_sample.PNG)

Código:

```javascript hl_lines="9 27 28 29"
<template>
    <kendo-grid :data-source="dataSource"
                :pageable="false"
                :sortable="true"
                ref="grid">
        <kendo-grid-column field="status_display"
                            title="Status"
                            :width="120"
                            :template="status_display_template"></kendo-grid-column>
        <kendo-grid-column field="descricao"
                            title="Anotação"></kendo-grid-column>
    </kendo-grid>
</template>

<script>
export default {
  computed: {
    dataSource () {
      return new kendo.data.DataSource({
        transport: {
          read: {
            url: 'minhaapi'
          }
        }
      })
    },
    status_display_template () {
      return `<div class="chip #= status_display === 'Pendente' ? 'red' : 'green' # white--text"><span class="chip__content">#= status_display #</span></div>`
    }
  }
}
</script>
```

## Icones em comandos

Exemplo:

![Screenshot](/img/grid_sample1.PNG)

Código:

```javascript hl_lines="13 14 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 57 58 59 60"
<template>
    <kendo-grid :data-source="dataSource"
                :pageable="false"
                :sortable="true"
                v-on:databound="gridBound"
                ref="grid">
        <kendo-grid-column field="status_display"
                            title="Status"
                            :width="120"
                            :template="status_display_template"></kendo-grid-column>
        <kendo-grid-column field="descricao"
                            title="Anotação"></kendo-grid-column>
        <kendo-grid-column :command="resolverCmd" :width="70"></kendo-grid-column>
        <kendo-grid-column :command="cancelarCmd" :width="70"></kendo-grid-column>
    </kendo-grid>
</template>

<script>
export default {
  computed: {
    cancelarCmd () {
      return {
        name: 'cancelar',
        text: ' ',
        className: 'k-button-icon',
        click (e) {
          e.preventDefault()
          console.log(e)
        }
      }
    },
    resolverCmd () {
      return {
        name: 'resolver',
        text: ' ',
        className: 'k-button-icon',
        click (e) {
          e.preventDefault()
          console.log(e)
        }
      }
    },      
    dataSource () {
      return new kendo.data.DataSource({
        transport: {
          read: {
            url: 'minhaapi'
          }
        }
      })
    },
    status_display_template () {
      return `<div class="chip #= status_display === 'Pendente' ? 'red' : 'green' # white--text"><span class="chip__content">#= status_display #</span></div>`
    }
  },
  methods: {
    gridBound (e) {
      $('.k-grid-resolver').addClass('k-icon').addClass(' k-i-check')
      $('.k-grid-cancelar').addClass('k-icon').addClass(' k-i-close-outline')
    }
  }
}
</script>
```