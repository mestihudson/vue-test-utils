## Testando Componentes de Arquivo-Simples com Mocha + webpack

> Um exemplo de projeto para essa configuração está disponível em [GitHub](https://github.com/vuejs/vue-test-utils-mocha-webpack-example).

Uma outra estratégia para teste de SFCs é compilar todos os nossos testes via webpack e então rodá-los em um executor de testes. A vantagem dessa abordagem é que ela bos dá completo completo suporte a todas as funcionalidades do webpack e do `vue-loader`, tanto que nós não temos que fazer flexibilizações em nosso código-fonte.

Você pode tecnicamente usar qualquer executor de testes que você quiser e manualmente amarrar as coisas todas juntas, mas nós encontramos [`mocha-webpack`](https://github.com/zinserjan/mocha-webpack) que provê uma experiência simplificada para essa tarefa em particular.

### Configurando `mocha-webpack`

Nós vamos supor que você está iniciando com uma configuração que já inclui o webpack, vue-loader e Babel devidamente estabelecida - por exemplo, o modelo `webpack-simple` gerado pelo `vue-cli`.

A primeira coisa a fazer é instalar as dependências de teste:

```bash
npm install --save-dev @vue/test-utils mocha mocha-webpack
```

Depois nós precisamos definir um script de teste em nosso `package.json`.

```json
// package.json
{
  "scripts": {
    "test": "mocha-webpack --webpack-config webpack.config.js --require test/setup.js test/**/*.spec.js"
  }
}
```

Algumas coisas a notar aqui:

- A flag `--webpack-config` especifica o arquivo de configuração do webpack a ser utilizado para os testes. Em muitos casos, isto seria identico a configuração que você usa para o projeto atual com um pequeno adendo. Nós vamos falar sobre isto mais tarde.

- A flag `--require` assegura que o arquivo `test/setup.js` seja executado antes de qualquer teste, de tal forma que nós passamos configurar o ambiente global para que nossos testes possam rodar nele.

- O argumento final é um punhado de arquivos de testes a serem incluídos no pacote de teste.

### Configuração Extra do webpack

#### Externalizando Dependências do NPM

Em nossos testes nós iremos fatalmente importar uma série de NPM dependências - algumas destes módulos pode ser escritos para uso sem navegador em mente e simplesmente não pode ser empacotado pelo webpack. Uma outra consideração é que a externalização de dependências melhora deveras a velocidade de carga dos testes. Nós podemos externalizar todas as dependências do NPM com o `webpack-node-externals`:

```js
// webpack.config.js
const nodeExternals = require('webpack-node-externals')

module.exports = {
  // ...
  externals: [nodeExternals()]
}
```

#### Mapeamento de Fontes

Mapeamento de fontes precisam estar em linha para serem captados pelo `mocha-webpack`. A configuração recomendada é:

```js
module.exports = {
  // ...
  devtool: 'inline-cheap-module-source-map'
}
```

Se depurar via IDE, é também recomendado adicionar o seguinte:

```js
module.exports = {
  // ...
  output: {
    // ...
    // use caminhos absolutos nos mapeamentos de fontes (importante para depuração via IDE)
    devtoolModuleFilenameTemplate: '[absolute-resource-path]',
    devtoolFallbackModuleFilenameTemplate: '[absolute-resource-path]?[hash]'
  }
}
```

### Configurando o Ambiente do Navegador

Vue Test Utils requer um ambiente de nevegador para rodar. Nós podemos simulá-lo no Node pelo uso do `jsdom-global`:

```bash
npm install --save-dev jsdom jsdom-global
```

Então em `test/setup.js`:

```js
require('jsdom-global')()
```

Isto adiciona o ambiente do navegador ao Node, tanto que o Vue Test Utils possa rodar corretamente.

### Usando uma Biblioteca de Asserções

[Chai](http://chaijs.com/) is a popular assertion library that is commonly used alongside Mocha. You may also want to check out [Sinon](http://sinonjs.org/) for creating spies and stubs.
[Chai](http://chaijs.com/) é uma biblioteca de asserções popular que é comumente usada junto com o Mocha. Você pode também querer dar uma olhada em [Sinon](http://sinonjs.org/) para criar espiões e dublês.

Alternativamente você pode usar `expect`, que agora é parte do Jest, e expõe [exatamente a mesma API](https://jestjs.io/docs/en/expect#content) na documentação do Jest.

Nós vamos usar `expect` aqui e fazê-lo globalmente disponível tanto que nós não tenhamos que importá-lo em todos os testes:

```bash
npm install --save-dev expect
```

Then in `test/setup.js`:
Então em `test/setup.js`:

```js
require('jsdom-global')()

global.expect = require('expect')
```

### Otimizando o Babel para Testess

Note que você está usando o `babel-loader` para tratar JavaScript. Você já deve ter configurado o Babel se você está usando sua app via arquivo `.babelrc`. Aqui `babel-loader` irá usar, automaticamente, o mesmo arquivo de configuração.

One thing to note is that if you are using Node 6+, which already supports the majority of ES2015 features, you can configure a separate Babel [env option](https://babeljs.io/docs/usage/babelrc/#env-option) that only transpiles features that are not already supported in the Node version you are using (e.g. `stage-2` or flow syntax support, etc.).
Uma coisa a notar é que se você está usando o Node 6+, que já suporta a maioria das funcionalidades do ES2015, você pode configurar um Babel diferente através da [opção env](https://babeljs.io/docs/usage/babelrc/#env-option) que somente transpila funcionalidades que ainda não são suportadas na versão do Node que você está usando (por exemplo, `stage-2` ou suporte à sintaxe de fluxo, etc.).

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
import { shallowMount } from '@vue/test-utils'
import Counter from '../src/Counter.vue'

describe('Counter.vue', () => {
  it('increments count when button is clicked', () => {
    const wrapper = shallowMount(Counter)
    wrapper.find('button').trigger('click')
    expect(wrapper.find('div').text()).toMatch('1')
  })
})
```

E agora nós podemos executar o teste:

```
npm run test
```

Uhuuuuu, nós conseguimos executar nossos testes!

### Cobertura

To setup code coverage to `mocha-webpack`, follow [the `mocha-webpack` code coverage guide](https://github.com/zinserjan/mocha-webpack/blob/master/docs/guides/code-coverage.md).
Para configurar a cobertura de código para o `mocha-webpack`, siga [o guia de cobertura de código do `mocha-webpack`](https://github.com/zinserjan/mocha-webpack/blob/master/docs/guides/code-coverage.md).

### Recursos

- [Exemplo de projeto para essa configuração](https://github.com/vuejs/vue-test-utils-mocha-webpack-example)
- [Mocha](https://mochajs.org/)
- [mocha-webpack](http://zinserjan.github.io/mocha-webpack/)
- [Chai](http://chaijs.com/)
- [Sinon](http://sinonjs.org/)
- [jest/expect](https://jestjs.io/docs/en/expect#content)
