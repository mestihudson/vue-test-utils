## Iniciando

### Setup (Configuração)

Para ter um rápida noção do uso do Vue Test Utils, clone nosso repositório demostrativo com setup (configuração) básica e instale as dependências:

```bash
git clone https://github.com/vuejs/vue-test-utils-getting-started
cd vue-test-utils-getting-started
npm install
```

Você verá que o projeto inclui um component simples, `counter.js`:

```js
// counter.js

export default {
  template: `
    <div>
      <span class="count">{{ count }}</span>
      <button @click="increment">Increment</button>
    </div>
  `,

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
```

### Montando Componentes

Vue Test Utils testa componentes Vue ao montá-los em isolamento, 'mockando' as entradas necessárias (propriedades, injeções e eventos de usuário) e assertindo as saídas (resultados renderizados, eventos personalizados emitidos).

Componentes Montados são retornados dentro de um [Embrulho (Wrapper)](../api/wrapper/), que expõe muitos métodos convenientes para manipular, examinar e consultar a instância subjacente do componente Vue.

Você pode criar um embrulho usando o método `mount`. Vamos criar um arquivo chamado `test.js`:

```js
// test.js

// Importe o método `mount()` dos utilitários de teste
// e o componente que você quer testar
import { mount } from '@vue/test-utils'
import Counter from './counter'

// Agora monte o componente e você terá o embrulho
const wrapper = mount(Counter)

// Você pode acessar a instância atual do Vue via `wrapper.vm`
const vm = wrapper.vm

// Para inspecionar o embrulho mais profundamente basta logá-lo para o console
// e a sua aventura com o Vue Test Utils começa
console.log(wrapper)
```

### Teste HTML renderizado pela saída do componente

Agora que nós temos o embrulho, a primeira coisa que nós podemos fazer é verificar que o HTML rederizado pela saída do componente casa com o que é esperado.

```js
import { mount } from '@vue/test-utils'
import Counter from './counter'

describe('Counter', () => {
  // Agora monte o componente e você tem o embrulho
  const wrapper = mount(Counter)

  it('renders the correct markup', () => {
    expect(wrapper.html()).toContain('<span class="count">0</span>')
  })

  // it's also easy to check for the existence of elements
  // também é fácil checar a existência dos elementos
  it('has a button', () => {
    expect(wrapper.contains('button')).toBe(true)
  })
})
```

Agora execute os testes com `npm test`. Você deve ver os testes passando.

### Simulando Interação de Usuário

Nosso contador (counter) deve incrementar a conta (count) quando o usuário clicar no botão. Para simular o comportamento, nós precisamos primeiro localizar o botão com `wrapper.find()`, que retorna um embrulho para o elemento botão (button). Nós podemos então simular o clique ao chamar `.trigger()` no botão do embrulho.

```js
it('button click should increment the count', () => {
  expect(wrapper.vm.count).toBe(0)
  const button = wrapper.find('button')
  button.trigger('click')
  expect(wrapper.vm.count).toBe(1)
})
```

### E quanto a `nextTick`?

Vue criar lotes de atualizações do DOM pendentes e os aplica assincronamente para evitar novas renderizações desnecessárias. Esse é o porquê de, na prática, geralmente nós usarmos `Vue.nextTick` para esperar até que o Vue desempenhe a atualização efetiva do DOM depois que disparamos alguma mudança de estado.

Para simplificar o uso, Vue Test Utils, aplica todas as atualizações sincronamente tanto que você não precisa usar `Vue.nextTick` para esperar por atualizações em seus testes.

_Nota: `nextTick` ainda é necessário quando você precisa explicitamente avançar o laço de evento, para operações tais como chamadas assíncronas ou resolução de promessas (promises)._

Se você ainda precisar usar `nextTick` em seus arquivos de teste, esteja ciente que qualquer erro é disparado dentro dele não pode ser capturado pelo seu executor de testes (test runner) posto que ele usa promessas (promises) internamente. Existem dois jeitos para corrigir isso: ou você pode configurar o método de callback `done` como um tratador de erro global do Vue no início do teste, ou você pode chamar `nextTick` sem nenhum argumento e retorná-lo como uma promessa.

```js
// isso não será capturado
it('will time out', done => {
  Vue.nextTick(() => {
    expect(true).toBe(false)
    done()
  })
})

// os dois testes seguintes irão trabalhar como esperado
it('will catch the error using done', done => {
  Vue.config.errorHandler = done
  Vue.nextTick(() => {
    expect(true).toBe(false)
    done()
  })
})

it('will catch the error using a promise', () => {
  return Vue.nextTick().then(function() {
    expect(true).toBe(false)
  })
})
```

### O que vem depois

- Integrar o Vue Test Utils em seu projeto ao [escolher um executor de testes](./choosing-a-test-runner.md).
- Aprenda mais sobre [técnicas comuns quando se escreve testes](./common-tips.md).
