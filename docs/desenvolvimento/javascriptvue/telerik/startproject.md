## Criar projeto usando VueCli + Telerik KendoUI

### Globais

Instale o `vue-cli` e o `yarn`

```bash
npm i -g vue-cli yarn
```

### Criando o projeto

```bash
vue init webpack telerikDemo

? Project name telerikdemo
? Project description Demo de utilização do KendoUI com vuejs
? Author William Galleti <william.galleti@gmail.com>
? Vue build standalone
? Install vue-router? Yes
? Use ESLint to lint your code? Yes
? Pick an ESLint preset Standard
? Set up unit tests No
? Setup e2e tests with Nightwatch? No
? Should we run `npm install` for you after the project has been created? (recommended) no

   vue-cli · Generated "telerikDemo".

# Project initialization finished!
# ========================

To get started:

  cd telerikDemo
  npm install (or if using yarn: yarn)
  npm run lint -- --fix (or for yarn: yarn run lint --fix)
  npm run dev

Documentation can be found at https://vuejs-templates.github.io/webpack
```

Rode a instalação das dependências:

````bash
yarn install
````

Antes de continuar, vamos efetuar algumas mudanças na estrutura do `webpack` e do `eslint` para podermos utilizar o jquery e o kendoui.

Entre na pasta do projeto `cd telerikDemo`  
Acesse o arquivo `build/webpack.dev.conf.js` e `build/webpack.prod.conf.js` e adicione no array `plugins` o seguinte objeto:

```javascript hl_lines="3 4 5 6 7 8 9"
const devWebpackConfig = merge(baseWebpackConfig, {
  ...
  plugins: [
    new webpack.ProvidePlugin({
      jQuery: 'jquery',
      $: 'jquery',
      jquery: 'jquery'
    })
  ],
  ...
```

Agora, acesse o arquivo `.eslintrc.js` e adicione o jquery no atributo `env` e crie o atributo `globals` adicionando o atributo `kendo` 

````javascript hl_lines="5 7 8 9"
module.exports = {
  ...
  env: {
    browser: true,
    jquery: true
  },
  globals: {
    kendo: false
  },
  ...
}
````

Feito isso finalizamos a parte de setup do webpack e eslint, agora vamos para as instalações e configurações

### Instalando as dependencias e configurando

* jquery
* kendo-ui
* kendo-theme-default
* kendo-theme-material
* kendo-datasource-vue-wrapper
* kendo-grid-vue-wrapper

```bash
yarn add jquery @progress/kendo-ui, @progress/kendo-theme-default @progress/kendo-theme-material @progress/kendo-datasource-vue-wrapper @progress/kendo-grid-vue-wrapper 
```

Feito isso, vamos criar uma pasta `src/plugins` e dentro dela, criar o arquivo `kendo.js`

Nele vamos adicionar as importações e inicialização do kendo no vue

````javascript
import Vue from 'vue'
import '@progress/kendo-ui'
import '@progress/kendo-theme-default/dist/all.css'
import '@progress/kendo-theme-material/dist/all.css'
import { KendoDataSource, KendoDataSourceInstaller } from '@progress/kendo-datasource-vue-wrapper'
import { KendoGrid, KendoGridInstaller } from '@progress/kendo-grid-vue-wrapper'

Vue.use(KendoDataSourceInstaller)
Vue.use(KendoGridInstaller)

Vue.component(KendoDataSource)
Vue.component(KendoGrid)
````

Agora no nosso `src/main.js`, vamos importar nosso `kendo.js`

```javascript hl_lines="4"
import Vue from 'vue'
import App from './App'
import router from './router'
import '@/plugins/kendo'

Vue.config.productionTip = false

/* eslint-disable no-new */
new Vue({
  el: '#app',
  router,
  components: { App },
  template: '<App/>'
})
```

### Internacionalização

Para traduzir para `pt-br`, edite o arquivo `src/plugins/kendo.js` e importar o cultures referente ao idioma e definir na váriavel `kendo.culture` o idioma.

````javascript hl_lines="5 9"
import Vue from 'vue'
import '@progress/kendo-ui'
import '@progress/kendo-theme-default/dist/all.css'
import '@progress/kendo-theme-material/dist/all.css'
import '@progress/kendo-ui/js/cultures/kendo.culture.pt-BR'
import { KendoDataSource, KendoDataSourceInstaller } from '@progress/kendo-datasource-vue-wrapper'
import { KendoGrid, KendoGridInstaller } from '@progress/kendo-grid-vue-wrapper'

kendo.culture('pt-BR')

Vue.use(KendoDataSourceInstaller)
Vue.use(KendoGridInstaller)

Vue.component(KendoDataSource)
Vue.component(KendoGrid)

````