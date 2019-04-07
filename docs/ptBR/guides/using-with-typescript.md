## Usando com TypeScript

> Um exemplo de projeto para esta configuração está disponível no [GitHub](https://github.com/vuejs/vue-test-utils-typescript-example).

TypeScript é um subconjunto de JavaScript que adiciona tipagem e classes no topo da pilha do JS padrão. Vue Test Utils inclue tipos no pacote distribuído, tanto que ele possa trabalhar bem com TypeScript.

Neste guia, nós vamos passar processo de configuração de um projeto com TypeScript usando o Jest e o Vue Test Utils a partir de uma configuração básica criada pelo Vue CLI para TypeScript.

### Adicionando TypeScript

Primeiro você precisa criar um projeto. Se você não tem o Vue CLI instalado, faça-o globalmente:

```shell
$ npm install -g @vue/cli
```

E crie um projeto ao executar:

```shell
$ vue create hello-world
```

No prompt de comando do CLI, escolha `Manually select features`, selecione TypeScript, e pressione enter. Isto irá criar um projeto com TypeScript já configurado.

::: tip NOTE
If you want a more detailed guide on setting up Vue with TypeScript, checkout the [TypeScript Vue starter guide](https://github.com/Microsoft/TypeScript-Vue-Starter).
:::

The next step is to add Jest to the project.

### Setting up Jest

Jest is a test runner developed by Facebook, aiming to deliver a battery-included unit testing solution. You can learn more about Jest on its [official documentation](https://jestjs.io/).

Install Jest and Vue Test Utils:

```bash
$ npm install --save-dev jest @vue/test-utils
```

Next define a `test:unit` script in `package.json`.

```json
// package.json
{
  // ..
  "scripts": {
    // ..
    "test:unit": "jest"
  }
  // ..
}
```

### Processing Single-File Components in Jest

To teach Jest how to process `*.vue` files, we need to install and configure the `vue-jest` preprocessor:

```bash
npm install --save-dev vue-jest
```

Next, create a `jest` block in `package.json`:

```json
{
  // ...
  "jest": {
    "moduleFileExtensions": [
      "js",
      "ts",
      "json",
      // tell Jest to handle `*.vue` files
      "vue"
    ],
    "transform": {
      // process `*.vue` files with `vue-jest`
      ".*\\.(vue)$": "vue-jest"
    },
    "testURL": "http://localhost/"
  }
}
```

### Configuring TypeScript for Jest

In order to use TypeScript files in tests, we need to set up Jest to compile TypeScript. For that we need to install `ts-jest`:

```bash
$ npm install --save-dev ts-jest
```

Next, we need to tell Jest to process TypeScript test files with `ts-jest` by adding an entry under `jest.transform` in `package.json`:

```json
{
  // ...
  "jest": {
    // ...
    "transform": {
      // ...
      // process `*.ts` files with `ts-jest`
      "^.+\\.tsx?$": "ts-jest"
    }
    // ...
  }
}
```

### Placing Test Files

By default, Jest will recursively pick up all files that have a `.spec.js` or `.test.js` extension in the entire project.

To run test files with a `.ts` extension, we need to change the `testRegex` in the config section in the `package.json` file.

Add the following to the `jest` field in `package.json`:

```json
{
  // ...
  "jest": {
    // ...
    "testRegex": "(/__tests__/.*|(\\.|/)(test|spec))\\.(jsx?|tsx?)$"
  }
}
```

Jest recommends creating a `__tests__` directory right next to the code being tested, but feel free to structure your tests as you see fit. Just beware that Jest would create a `__snapshots__` directory next to test files that performs snapshot testing.

### Writing a unit test

Now we've got the project set up, it's time to write a unit test.

Create a `src/components/__tests__/HelloWorld.spec.ts` file, and add the following code:

```js
// src/components/__tests__/HelloWorld.spec.ts
import { shallowMount } from '@vue/test-utils'
import HelloWorld from '../HelloWorld.vue'

describe('HelloWorld.vue', () => {
  test('renders props.msg when passed', () => {
    const msg = 'new message'
    const wrapper = shallowMount(HelloWorld, {
      propsData: { msg }
    })
    expect(wrapper.text()).toMatch(msg)
  })
})
```

That's all we need to do to get TypeScript and Vue Test Utils working together!

### Resources

- [Example project for this setup](https://github.com/vuejs/vue-test-utils-typescript-example)
- [Jest](https://jestjs.io/)
