## Testando Componentes de Arquivo-Simples com Karma

> Um exemplo de projeto para essa configuração está disponível no [GitHub](https://github.com/eddyerburgh/vue-test-utils-karma-example).

Karma é um executor de testes que inicia navegadores, roda testes, e relata seus resultados como saída. Nós vamos usar o framework Mocha framework para escrever os testes. Iremos usar a biblioteca Chai para as asserções dos testes.

### Configurando o Mocha

Nós vamos supor que você está iniciando com uma configuração que já inclui o webpack, vue-loader e Babel devidamente estabelecida - por exemplo, o modelo `webpack-simple` gerado pelo `vue-cli`.

A primeira coisa a fazer é instalar as dependências de teste:

```bash
npm install --save-dev @vue/test-utils karma karma-chrome-launcher karma-mocha karma-sourcemap-loader karma-spec-reporter karma-webpack mocha
```

Depois precisaremos definir um script `test` em nosso `package.json`.

```json
// package.json
{
  "scripts": {
    "test": "karma start --single-run"
  }
}
```

- A opção `--single-run` diz ao Karma para executar o conjunto de testes apenas uma vez.

### Configuração do Karma

Crie um arquivo `karma.conf.js` na pasta raiz do projeto:

```js
// karma.conf.js

var webpackConfig = require('./webpack.config.js')

module.exports = function(config) {
  config.set({
    frameworks: ['mocha'],

    files: ['test/**/*.spec.js'],

    preprocessors: {
      '**/*.spec.js': ['webpack', 'sourcemap']
    },

    webpack: webpackConfig,

    reporters: ['spec'],

    browsers: ['Chrome']
  })
}
```

Este arquivo é usado para configurar o Karma.

Precisamos pré-processar nossos arquivos com o webpack, para tanto, adicionamos o citado como um, e incluímos nossa configuração referente ao mesmo. Podemos usar o arquivo de configuração do webpack na base do projeto sem mudar nada.

Em nossa configuração, executamos os testes no Chrome. Para adicionar navegadores extra, veja [a sessão de Navegadores na documentação do Karma](http://karma-runner.github.io/2.0/config/browsers.html).

### Pegando uma Biblioteca de Asserção

[Chai](http://chaijs.com/) é uma biblioteca de asserção popular que é comumente utilizada junto ao Mocha. Você pode querer dar uma olhada também em [Sinon](http://sinonjs.org/) para criar espiões e dublês.

Nós podemos instalar o plugin `karma-chai` para usar o `chai` em nossos testes.

```bash
npm install --save-dev karma-chai
```

### Adicionando um teste

Crie um arquivo em `src` chamado `Counter.vue`:

```html
<template>
  <div>
    {{ count }}
    <button @click="increment">Increment</button>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        count: 0
      }
    },

    methods: {
      increment() {
        this.count++
      }
    }
  }
</script>
```

E crie um arquivo de teste chamado `test/Counter.spec.js` com o seguinte código:

```js
import { expect } from 'chai'
import { shallowMount } from '@vue/test-utils'
import Counter from '../src/Counter.vue'

describe('Counter.vue', () => {
  it('increments count when button is clicked', () => {
    const wrapper = shallowMount(Counter)
    wrapper.find('button').trigger('click')
    expect(wrapper.find('div').text()).contains('1')
  })
})
```

E agora podemos executar os testes:

```
npm run test
```

Uhuuuu, nossos testes estão rodando!

### Cobertura

Para configurar a cobertura de código para o Karma, precisamos usar o plugin `karma-coverage`.

Por padrão, `karma-coverage` não usa mapas de código-fonte para mapear os relatórios de cobertura. Então nós precisamos usar o plugin `babel-plugin-istanbul` para assegurar que a cobertura é corretamente mapeada.

Instale `karma-coverage`, `babel-plugin-istanbul`, e `cross-env`:

```
npm install --save-dev karma-coverage cross-env
```

Nós vamos usar `cross-env` para configurar a variável de ambiente `BABEL_ENV`. Desta maneira nós podemos usar o `babel-plugin-istanbul` quando estamos compilando nossos testes - não queremos incluir o `babel-plugin-istanbul` quando estamos compilando nosso código para produção:

```
npm install --save-dev babel-plugin-istanbul
```

Atualize seu arquivo `.babelrc` para usar o `babel-plugin-istanbul` quando a variável de ambiente `BABEL_ENV` tiver sido configurada para teste:

```json
{
  "presets": [["env", { "modules": false }], "stage-3"],
  "env": {
    "test": {
      "plugins": ["istanbul"]
    }
  }
}
```

Agora atualize o arquivo `karma.conf.js` para usar a cobertura. Adicione `coverage` ao array/opção de `reporters`, e adicione um campo `coverageReporter`:

```js
// karma.conf.js

module.exports = function(config) {
  config.set({
    // ...

    reporters: ['spec', 'coverage'],

    coverageReporter: {
      dir: './coverage',
      reporters: [{ type: 'lcov', subdir: '.' }, { type: 'text-summary' }]
    }
  })
}
```

E atualize o script `test` para configurar a variável de ambiente `BABEL_ENV`:

```json
// package.json
{
  "scripts": {
    "test": "cross-env BABEL_ENV=test karma start --single-run"
  }
}
```

### Recursos

- [Exemplo de projeto para esta configuração](https://github.com/eddyerburgh/vue-test-utils-karma-example)
- [Karma](http://karma-runner.github.io/)
- [Mocha](https://mochajs.org/)
- [Chai](http://chaijs.com/)
- [Sinon](http://sinonjs.org/)
