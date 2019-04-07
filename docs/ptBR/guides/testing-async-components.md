## Testando Comportamento Assíncrono

Para simplificar o teste, Vue Test Utils aplica atualizações ao DOM _sincronamente_. Todavia, existem algumas técnicas que você precisa estar ciente disso quando testar um componente com comportamento assíncrono tais como funções de callback ou promessas.

Um dos comportamentos assíncronos mais comuns é a chamada de API e ações Vuex. Os seguintes exemplos mostram como testar um método que faz uma chamada a uma API. Este exemplo usa o Jest para executar o teste e mimetizar a biblioteca HTTP `axios`. Mais sobre a mimetização manual do Jest pode ser encontrado [aqui](https://jestjs.io/docs/en/manual-mocks#content).

A implementação de um mímico para o `axios` se parece com isto:

```js
export default {
  get: () => Promise.resolve({ data: 'value' })
}
```

O componente abaixo faz uma chamada a uma API quando o botão é clicado, e então atribui a resposta a `value`.

```html
<template>
  <button @click="fetchResults" />
</template>

<script>
  import axios from 'axios'

  export default {
    data() {
      return {
        value: null
      }
    },

    methods: {
      async fetchResults() {
        const response = await axios.get('mock/service')
        this.value = response.data
      }
    }
  }
</script>
```

Um teste pode ser escrito como este:

```js
import { shallowMount } from '@vue/test-utils'
import Foo from './Foo'
jest.mock('axios')

it('fetches async when a button is clicked', () => {
  const wrapper = shallowMount(Foo)
  wrapper.find('button').trigger('click')
  expect(wrapper.vm.value).toBe('value')
})
```

Este teste, atualmente, falha porque a asserção é chamada antes da promessa em `fetchResults` ter sido resolvida. Muitas bibliotecas de testes unitários provêem uma função de callback para permitir que o executor saiba quando o teste está completo. Tanto o Jest quanto o Mocha usam `done`. Podemos usar `done` em combinação com `$nextTick` ou `setTimeout` para assegurar que quaisquer promessas tenham sido cumpridas antes que a asserção seja feita.

```js
it('fetches async when a button is clicked', done => {
  const wrapper = shallowMount(Foo)
  wrapper.find('button').trigger('click')
  wrapper.vm.$nextTick(() => {
    expect(wrapper.vm.value).toBe('value')
    done()
  })
})
```

A razão de `setTimeout` permitir que o teste passe é que a fila de microtarefas onde as promessas são processadas rodam antes da fila de tarefas propriamente dita, onde as funções de callback de `setTimeout` são processadas. Isto significa no momento em que o callback de `setTimeout` roda, qualquer promessa na fila de microtarefa ainda será executada. `$nextTick` por outro lado agenda uma microtarefa, mas uma vez que a fila de microtarefas é processada em forma de fila (FIFO, first-in-first-out) que também garante que a promessa tenha sido cumprida no tempo em que a asserção é feita. Veja [aqui](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/) para uma explicação mais detalhada.

Uma outra solução é usar uma função assíncrona e o pacote npm `flush-promises`. Ele descarrega todos os tratadores de promessas pendentes. Você pode usar `await` ao chamar `flushPromises` para descarregar as promessas pendentes e melhorar a legibilidade de seus testes.

O teste atualizado se parece com isto:

```js
import { shallowMount } from '@vue/test-utils'
import flushPromises from 'flush-promises'
import Foo from './Foo'
jest.mock('axios')

it('fetches async when a button is clicked', async () => {
  const wrapper = shallowMount(Foo)
  wrapper.find('button').trigger('click')
  await flushPromises()
  expect(wrapper.vm.value).toBe('value')
})
```

Esta mesma técnica pode ser aplicda ações Vuex, que retornam uma promessa por padrão.
