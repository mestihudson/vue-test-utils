## Testando Componentes de Arquivo-Simples com Jest

> Um exemplo de projeto para essa configuração está disponível em [GitHub](https://github.com/vuejs/vue-test-utils-jest-example).

Jest é um executor de testes desenvolvido pelo Facebook, visando entregar uma solução para testes unitários que já venham com 'baterias inclusas', como se costuma dizer. Você pode aprender mais sobre Jest na página de sua [documentação oficial](https://jestjs.io/).

#### Configurando o Jest

Nós vamos supor que você está iniciando com uma configuração que já inclui o webpack, vue-loader e Babel devidamente estabelecida - por exemplo, o modelo `webpack-simple` gerado pelo `vue-cli`.

The first thing to do is install Jest and Vue Test Utils:
A primeira coisa a fazer é instalar o Jest e o Vue Test Utils:

```bash
$ npm install --save-dev jest @vue/test-utils
```

Depois nós precisamos definir um script de teste em nosso `package.json`.

```json
// package.json
{
  "scripts": {
    "test": "jest"
  }
}
```

### Processando Componentes de Arquivo-Simples no Jest

Para ensinar ao Jest como processar arquivos `*.vue`, nós vamos precisar instalar e configurar o pré-processador `vue-jest`:

```bash
npm install --save-dev vue-jest
```

Depois, criar um bloco `jest` em `package.json`:

```json
{
  // ...
  "jest": {
    "moduleFileExtensions": [
      "js",
      "json",
      // diz ao Jest que ele deve tratar arquivos `*.vue`
      "vue"
    ],
    "transform": {
      // processa arquivos `*.vue` com `vue-jest`
      ".*\\.(vue)$": "vue-jest"
    }
  }
}
```

> **Nota:** `vue-jest` atualmente não dá suporte a todas as funcionalidades do `vue-loader`, por exemplo, suporte a blocos personalizados e carga de estilos. Além disso, algumas funcionalidade específicas do webpack tais como code-splitting (divisão de código em vários arquivos/módulos), também não são suportadas. Para usar essas características não suportadas, você precisa usar o Mocha ao invés do Jest para rodas seus testes, e webpack para compilar seus componentes. Para começar, leia o guia em [testando SFCs com Mocha + webpack](./testing-single-file-components-with-mocha-webpack.md).

### Tratando Atalhos (Aliases) com webpack

If you use a resolve alias in the webpack config, e.g. aliasing `@` to `/src`, you need to add a matching config for Jest as well, using the `moduleNameMapper` option:
Se você usa um resolvedor de atalho na configuração do webpack, por exemplo, que leve `@` para `/src`, você precisa adicionar uma configuração de reconhecimento de padrão para o Jest também, usando a opção `moduleNameMapper`:

```json
{
  // ...
  "jest": {
    // ...
    // adiciona o suporte para que `@` seja mapeado, através de um atalho, para `src`, no código fonte
    "moduleNameMapper": {
      "^@/(.*)$": "<rootDir>/src/$1"
    }
  }
}
```

### Configurando o Babel para o Jest

<!-- todo ES modules has been supported in latest versions of Node -->

Apesar de as últimas versões do Node já suportarem muitas das funcionalidades do ES2015, você pode ainda querer usar a sintaxe do ES e as funcionalidades em estágio-x (stage-x) em seus testes. Para isso nós precisamos instalar o  `babel-jest`:

```bash
npm install --save-dev babel-jest
```

Depois, nós precisamos dizer ao Jest para processar arquivos de teste JavaScript com `babel-jest` adicionando uma entrada sob a propriedade `jest.transform` no `package.json`:

```json
{
  // ...
  "jest": {
    // ...
    "transform": {
      // ...
      // processa arquivos js com o `babel-jest`
      "^.+\\.js$": "<rootDir>/node_modules/babel-jest"
    }
    // ...
  }
}
```

> Por padrão, `babel-jest` automaticamente se configura tão logo ele seja instalado. Todavia, uma vez que nós tenhamos explicitamente adicionado uma transformação para arquivos `*.vue`, nós agora precisamos explicitamente configurá-lo para o `babel-jest` também.

Assumindo o uso do `babel-preset-env` com webpack, a configuração padrão do Babel desabilita a transpilação de módulos ES porque o webpack já sabe como tratá-los. No entanto, nós precisamos habilitá-lo para nossos testes porque os testes do Jest executam diretamente no Node.

Também, nós podemos dizer ao `babel-preset-env` para direcionar para a versão do Node que estamos usando. Isto pula funcionalidades de transpilação desnecessárias e faz nossos testes bem mais rápidos.

Para aplicar essas opções somente para os testes, ponha-os em uma configuração separada sob a `env.test` (isto será automaticamente captado pelo `babel-jest`)

Exemplo `.babelrc`:

```json
{
  "presets": [["env", { "modules": false }]],
  "env": {
    "test": {
      "presets": [["env", { "targets": { "node": "current" } }]]
    }
  }
}
```

### Colocando Arquivos de Test

Por padrão, o Jest recursivamente detectará todos os arquivos cujas extensões sejam `.spec.js` ou `.test.js` no projeto inteiro. Se isso não se encaixar em suas necessidades, é possível [mudar a propriedade `testRegex`](https://jestjs.io/docs/en/configuration#testregex-string-array-string) na sessão de configuração do arquivo `package.json`.

Jest recommends creating a `__tests__` directory right next to the code being tested, but feel free to structure your tests as you see fit. Just beware that Jest would create a `__snapshots__` directory next to test files that performs snapshot testing.
Jest recomenda criar um diretório chamado `__tests__` no mesmo local do código a ser testado, mas sinta-se livre para estruturar seus testes como você achar melhor. Apenas esteja ciente que o Jest criaria um diretório chamado `__snapshots__` para os arquivos de que executam testes daquele tipo.

### Cobertura

Jest pode ser usado para gerar relatórios de cobertura em múltiplos formatos. A seguir um exemplo simples para começar:

Extenda sua configuração `jest` (geralmente no arquivo `package.json` ou no arquivo `jest.config.js`) com a opção [`collectCoverage`](https://jestjs.io/docs/en/configuration#collectcoverage-boolean), e então adicionar a array/opção [`collectCoverageFrom`](https://jestjs.io/docs/en/configuration#collectcoveragefrom-array) para definir os arquivos para os quais a informação de cobertura deverá ser coletada.

```json
{
  "jest": {
    // ...
    "collectCoverage": true,
    "collectCoverageFrom": ["**/*.{js,vue}", "!**/node_modules/**"]
  }
}
```

This will enable coverage reports with the [default coverage reporters](https://jestjs.io/docs/en/configuration#coveragereporters-array-string). You can customise these with the `coverageReporters` option:
Isto habilitará os relatórios de cobertura com o [tipo padrão](https://jestjs.io/docs/en/configuration#coveragereporters-array-string). Você pode personalizar estas com a opção `coverageReporters`:

```json
{
  "jest": {
    // ...
    "coverageReporters": ["html", "text-summary"]
  }
}
```

Documentação adicional pode ser encontrada na [sessão de configuração correspondente do Jest](https://jestjs.io/docs/en/configuration#collectcoverage-boolean), onde você pode achar opções para cobertura de limites, diretórios de saída, etc.

### Exemplo de Especificação (Spec)

Se você está familiarizado com Jasmine, você deve se sentir em casa com a [API de asserções](https://jestjs.io/docs/en/expect#content) do Jest:

```js
import { mount } from '@vue/test-utils'
import Component from './component'

describe('Component', () => {
  test('is a Vue instance', () => {
    const wrapper = mount(Component)
    expect(wrapper.isVueInstance()).toBeTruthy()
  })
})
```

### Teste de Instantâneo (Snapshot)

Quando você monta um componente com o Vue Test Utils, você tem acesso ao elemento raiz do HTML. Isto pode ser salvo na forma de um instantâneo pelo uso do [teste de instantâneo do Jest]:

```js
test('renders correctly', () => {
  const wrapper = mount(Component)
  expect(wrapper.element).toMatchSnapshot()
})
```

Nós podemos melhorar o instantâneo salvo com um serializador personalizado:

```bash
npm install --save-dev jest-serializer-vue
```

Então configure-o no `package.json`:

```json
{
  // ...
  "jest": {
    // ...
    // serializador para instantâneos
    "snapshotSerializers": ["jest-serializer-vue"]
  }
}
```

### Recursos

- [Exemplo de projeto para esta configuração](https://github.com/vuejs/vue-test-utils-jest-example)
- [Exemplos e slides da Vue Conf 2017](https://github.com/codebryo/vue-testing-with-jest-conf17)
- [Jest](https://jestjs.io/)
- [Babel preset env](https://github.com/babel/babel-preset-env)
