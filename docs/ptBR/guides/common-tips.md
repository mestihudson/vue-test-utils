## Dicas Comuns

### Sabendo O Quê Testar

Para componentes de UI (Interface de Usuário), nós não recomendamos que se almeje pela cobertura completa, pois isso leva a uma preocupação demasiada com detalhes de implementação interna dos componentes e poderia resultar em testes frágeis.

Ao invés disso, nós recomendamos escrever testes que verifiquem a interface pública dos componentes, e trate seus detalhes internos como uma caixa preta. Um caso de teste simples verificaria que alguma entrada (interação do usuário ou mudança de propriedades) proveu ao componente resultados em sua saída esperada (renderizar resultado ou emitir eventos personalizados).

Por exemplo, para o componente `Counter`, que incrementa o mostrador do contador em 1 cada vez que o botão é clicado, seu caso de teste simularia o clique e verificaria que a saída rederizada foi incrementada em 1. O teste não se importa como o `Counter` incrementa o valor, ele somente se preocupa com a entrada e a saída.

O benefício desse método é que a interface pública do seu componente permanesce a mesma, seu teste irá passar não importando como internamente a implementação dele muda com o tempo.

Esse tópico é discutido com mais detalhes em uma [grande apresentação de Matt O'Connell](https://www.youtube.com/watch?v=OIpfWTThrK8).

### Renderização Raza

Em testes unitários, nós tipicamente queremos focar no componente sendo testado como uma unidade isolada e evitar indiretamente verificar o comportamento de seus componentes filhos.

Além disso, para componentes que contêm muitos filhos, a renderização da árvore inteira pode ser realmente grande. Renderizar, repetidamente, todos os componentes filhos poderia tornar nossos testes lentos.

Vue Test Utils permite que você monte um componente sem renderizar seus componentes filhos (por dublá-los) com o métodos `shallowMount`:

```js
import { shallowMount } from '@vue/test-utils'

const wrapper = shallowMount(Component)
wrapper.vm // a instância já montada do componente Vue
```

### Verificando Eventos Emitidos

Cada embrulho montado automaticamente registra todos os eventos emitidos pela instância Vue subjacente. Você pode recuperar os eventos registrados usando o método `wrapper.emmited()`:

```js
wrapper.vm.$emit('foo')
wrapper.vm.$emit('foo', 123)

/*
`wrapper.emitted()` retorna o seguinte objeto:
{
  foo: [[], [123]]
}
*/
```

Você pode então fazer verificações baseadas nesses dados:

```js
// verifica se o evento foi emitido
expect(wrapper.emitted().foo).toBeTruthy()

// verifica quantas vezes o evento foi emitido
expect(wrapper.emitted().foo.length).toBe(2)

// verifica o conteúdo relacionado ao evento
expect(wrapper.emitted().foo[1]).toEqual([123])
```

Você pode também obter uma lista (Array) de eventos pela ordem em que foram emitidos chamando o método [`wrapper.emittedByOrder()`](../api/wrapper/emittedByOrder.md).

### Emitindo Eventos do Componente Filho

Você pode emitir um evento personalizado a partir do componente filho acessando a sua instância.

**Componente sob teste**

```html
<template>
  <div>
    <child-component @custom="onCustom" />
    <p v-if="emitted">Emitted!</p>
  </div>
</template>

<script>
  import ChildComponent from './ChildComponent'

  export default {
    name: 'ParentComponent',
    components: { ChildComponent },
    data() {
      return {
        emitted: false
      }
    },
    methods: {
      onCustom() {
        this.emitted = true
      }
    }
  }
</script>
```

**Teste**

```js
import { shallowMount } from '@vue/test-utils'
import ParentComponent from '@/components/ParentComponent'
import ChildComponent from '@/components/ChildComponent'

describe('ParentComponent', () => {
  it("displays 'Emitted!' when custom event is emitted", () => {
    const wrapper = shallowMount(ParentComponent)
    wrapper.find(ChildComponent).vm.$emit('custom')
    expect(wrapper.html()).toContain('Emitted!')
  })
})
```

### Manipulando Estado do Componente

Você pode diretamente manipular o estado do componente usando os métodos `setData` ou `setProps` no embrulho:

```js
wrapper.setData({ count: 10 })

wrapper.setProps({ foo: 'bar' })
```

### Mimetizando (Mocking) Propriedades

Você pode passar propriedades para o componente usando a opção `propsData` do Vue:

```js
import { mount } from '@vue/test-utils'

mount(Component, {
  propsData: {
    aProp: 'some value'
  }
})
```

Você pode também atualizar as propriedades de um componente já montado com o método `wrapper.setProps({})`.

_Para uma lista completa de opções, favor ver a [sessão opções de montagem](../api/options.md) da documentação._

### Aplicando Plugins e Mixins Globais

Alguns componentes podem confiar em funcionalidades injetadas por um plugin ou mixin global, por exemplo `vuex` e `vue-router`.

Se você está escrevendo testes para componentes em uma aplicação específica, você pode configurar os mesmos plugins e mixins globais uma vez na entrada/construção de seus testes. Mas em alguns casos, por exemplo ao testar um componente genérico específico que pode ser compartilhado através de diferentes aplicações, é melhor testar seus componentes em uma configuração mais isolada, sem poluir o construtor global do `Vue`. Nós podemos usar o método [`createLocalVue`](../api/createLocalVue.md) para conseguir isso:

```js
import { createLocalVue } from '@vue/test-utils'

// cria um contrutor extendido do `Vue`
const localVue = createLocalVue()

// instala plugins como de costume
localVue.use(MyPlugin)

// passa o construtor extendido `localVue` para as opções de montagem
mount(Component, {
  localVue
})
```

**Note que alguns plugins, tal como Vue Router, adicionar propriedades que são somente-leitura para o contrutor global do Vue. Isso torna impossível reinstalar o plugin no construtor extendido `localVue`, ou adicionar mímicos (mocks) para essas propriedades.

### Mimetizando Injeções

Uma outra estratégia para propriedades injetadas é simplesmente mimetizá-los. Você pode fazer isso com a opção `mocks`:

```js
import { mount } from '@vue/test-utils'

const $route = {
  path: '/',
  hash: '',
  params: { id: '123' },
  query: { q: 'hello' }
}

mount(Component, {
  mocks: {
    // adiciona um objeto mímico `$route` à instancia do Vue
    // antes de montar o componente
    $route
  }
})
```

### Dublando componentes

Você pode sobrepor componentes que são registrados globalmente ou localmente pelo uso da opção `stubs`:

```js
import { mount } from '@vue/test-utils'

mount(Component, {
  // Irá resolver globally-registered-component com
  // um dublê vazio
  stubs: ['globally-registered-component']
})
```

### Lidando com Roteamento

Uma vez que roteamento, por definição, tem haver com a estrutura geral da aplicação e envolve múltiplos componentes, é melhor testado via testes de integração ou de ponta-a-ponta. Para componentes individuais que confiam nas funcionalidades de `vue-router`, você pode mimetizá-los por usar técnicas mencionadas acima.

### Detectando estilos

Seu teste pode somente detectar estilos diretamente definidos (inline) quando executados em `jsdom`.
