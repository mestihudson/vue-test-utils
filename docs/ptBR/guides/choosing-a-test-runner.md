## Escolhendo um executor de testes

Um executor de testes é um programa que roda testes.

Existem muitos executores de testes muito populares para JavaScript, e Vue Test Utils trabalha com todos eles. Ele é agnóstico ao executor de testes.

Existem umas poucas coisas a considerar quando se escolhe um executor de testes, no entanto: conjunto de funcionalidades, desempenho, e suporte a pre-compilação de arquivo-simples de componente (SFC, Single-File Component). Depois de cuidadosamente comparar as bibliotecas existentes, aqui estão os dois executores de testes que nós recomendamos.

- [Jest](https://jestjs.io/docs/en/getting-started#content) é um executor de testes com mais funcionalidades. Ele requer menos configuração, configura o JSDOM por padrão, provê uma gama de asserssões de fábrica, e tem uma linha de comando que possui uma grande experiência de usuário. Todavia, você irá necessitar de um pré-processador capaz de importar componentes SFC em seus testes. Nós criamos o pré-processador `vue-jest` que pode tratar muitas das funcionalidades comuns de um SFC, mas, atualmente, não possui 100% de paridade funcional com o `vue-loader`.

- [mocha-webpack](https://github.com/zinserjan/mocha-webpack) é um embrulho em volta do webpack + Mocha, mas com uma interface mais próxima (a more streamlined interface) e modo de observação (watch mode). Os benefícios de sua configuração é que nós temos um completo suporte ao SFC via webpack + `vue-loader`, mas ele requer mais configuração desde o começo.

### Ambiente do Navegador

Vue Test Utils confia no ambiente do navegador. Tecnicamente você pode rodá-lo em um navegador real, mas não é recomendado devido a complexidade de executar navegadores reais em diferentes plataformas. Ao invés disso, nós recomendamos rodar os teste no Node com um ambiete de navegador virtual usando [JSDOM](https://github.com/tmpvar/jsdom).

Jest, o executor de testes, configura o JSDOM automaticamente. Para outros executores de testes, você pode manualmente configurar o JSDOM para os testes usando [jsdom-global](https://github.com/rstacruz/jsdom-global) na entrada para seus testes:

```bash
npm install --save-dev jsdom jsdom-global
```

---

```js
// na configuração/entrada dos testes
require('jsdom-global')()
```

### Testando Componentes de Arquivo-Simples (SFC, Single-File Components)

Componentes de Arquivo-Simples do Vue (SFCs) requerem pré-compilação antes de eles poderem ser rodados no Node ou no navegador. Existem duas formas recomendadas para desempenhar essa compilação: com um pré-processador Jest, ou por diretamente usar o webpack.

O pré-processador `vue-jest` dá suporte as funcionalidades básicas de um SFC mas, atualmente, não trata blocos de estilo ou blocos personalizados, que são suportados apenas no `vue-loader`. Se você confia nessas funcionalidades ou outras configurações específicas do webpack, você precisará usar uma configuração baseada em webpack + `vue-loader`.

Leia os seguintes guias para configurações diversas:

- [Testando Componentes de Arquivo-Simples com Jest](./testing-single-file-components-with-jest.md)
- [Testando Componentes de Arquivo-Simples com Mocha + webpack](./testing-single-file-components-with-mocha-webpack.md)

### Resources
### Recursos

- [Comparação de desempenho de executor de testes](https://github.com/eddyerburgh/vue-unit-test-perf-comparison)
- [Exemplo de projeto com Jest](https://github.com/vuejs/vue-test-utils-jest-example)
- [Exemplo de projeto com Mocha](https://github.com/vuejs/vue-test-utils-mocha-webpack-example)
- [Exemplo de projeto com tape](https://github.com/eddyerburgh/vue-test-utils-tape-example)
- [Exemplo de projeto com AVA](https://github.com/eddyerburgh/vue-test-utils-ava-example)
- [tyu - Delightful web testing by egoist](https://github.com/egoist/tyu)
- [tyu - Testes web deleitáveis por egoist](https://github.com/egoist/tyu)
