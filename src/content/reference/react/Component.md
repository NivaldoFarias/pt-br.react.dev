---
title: Component
---

<Pitfall>

Recomendamos definir componentes como funções ao invés de classes. [Veja como migrar.](#alternatives)

</Pitfall>

<Intro>

`Component` é a classe base para os componentes React definidos como [classes JavaScript.](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes) Componentes de classe ainda são suportados pelo React, mas não recomendamos usá-los em código novo.

```js
class Greeting extends Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}
```

</Intro>

<InlineToc />

---

## Referência {/*reference*/}

### `Component` {/*component*/}

Para definir um componente React como uma classe, estenda a classe `Component` embutida e defina um método [`render`](#render):

```js
import { Component } from 'react';

class Greeting extends Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}
```

Somente o método `render` é obrigatório, outros métodos são opcionais.

[Veja mais exemplos abaixo.](#usage)

---

### `context` {/*context*/}

O [contexto](/learn/passing-data-deeply-with-context) de um componente de classe está disponível como `this.context`. Ele só está disponível se você especificar *qual* contexto você deseja receber usando [`static contextType`](#static-contexttype).

Um componente de classe pode ler apenas um contexto por vez.

```js {2,5}
class Button extends Component {
  static contextType = ThemeContext;

  render() {
    const theme = this.context;
    const className = 'button-' + theme;
    return (
      <button className={className}>
        {this.props.children}
      </button>
    );
  }
}

```

<Note>

Ler `this.context` em componentes de classe é equivalente a [`useContext`](/reference/react/useContext) em componentes de função.

[Veja como migrar.](#migrating-a-component-with-context-from-a-class-to-a-function)

</Note>

---

### `props` {/*props*/}

As props passadas para um componente de classe estão disponíveis como `this.props`.

```js {3}
class Greeting extends Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}

<Greeting name="Taylor" />
```

<Note>

Ler `this.props` em componentes de classe é equivalente a [declarar props](/learn/passing-props-to-a-component#step-2-read-props-inside-the-child-component) em componentes de função.

[Veja como migrar.](#migrating-a-simple-component-from-a-class-to-a-function)

</Note>

---

### `state` {/*state*/}

O estado de um componente de classe está disponível como `this.state`. O campo `state` deve ser um objeto. Não muta o estado diretamente. Se você deseja alterar o estado, chame `setState` com o novo estado.

```js {2-4,7-9,18}
class Counter extends Component {
  state = {
    age: 42,
  };

  handleAgeChange = () => {
    this.setState({
      age: this.state.age + 1 
    });
  };

  render() {
    return (
      <>
        <button onClick={this.handleAgeChange}>
        Increment age
        </button>
        <p>You are {this.state.age}.</p>
      </>
    );
  }
}
```

<Note>

Definir `state` em componentes de classe é equivalente a chamar [`useState`](/reference/react/useState) em componentes de função.

[Veja como migrar.](#migrating-a-component-with-state-from-a-class-to-a-function)

</Note>

---

### `constructor(props)` {/*constructor*/}

O [constructor](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/constructor) é executado antes que seu componente de classe *monte* (seja adicionado na tela). Normalmente, um construtor é usado apenas para dois propósitos no React. Ele permite que você declare o estado e [faça o bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_objects/Function/bind) de seus métodos de classe à instância da classe:

```js {2-6}
class Counter extends Component {
  constructor(props) {
    super(props);
    this.state = { counter: 0 };
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    // ...
  }
```

Se você usar a sintaxe JavaScript moderna, os construtores raramente são necessários. Em vez disso, você pode reescrever este código acima usando a [sintaxe de campo de classe pública](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Public_class_fields), que é suportada tanto por navegadores modernos quanto por ferramentas como [Babel:](https://babeljs.io/)

```js {2,4}
class Counter extends Component {
  state = { counter: 0 };

  handleClick = () => {
    // ...
  }
```

Um construtor não deve conter quaisquer efeitos colaterais ou inscrições.

#### Parâmetros {/*constructor-parameters*/}

* `props`: As props iniciais do componente.

#### Retorna {/*constructor-returns*/}

`constructor` não deve retornar nada.

#### Ressalvas {/*constructor-caveats*/}

* Não execute quaisquer efeitos colaterais ou inscrições no construtor. Em vez disso, use [`componentDidMount`](#componentdidmount) para isso.

* Dentro de um construtor, você precisa chamar `super(props)` antes de qualquer outra instrução. Se você não fizer isso, `this.props` será `undefined` enquanto o construtor for executado, o que pode ser confuso e causar erros.

* Construtor é o único lugar onde você pode atribuir [`this.state`](#state) diretamente. Em todos os outros métodos, você precisa usar [`this.setState()`](#setstate) em vez disso. Não chame `setState` no contrutor.

* Quando você usa [renderização do servidor,](/reference/react-dom/server) o construtor também será executado no servidor, seguido pelo método [`render`](#render). No entanto, métodos de ciclo de vida como `componentDidMount` ou `componentWillUnmount` não serão executados no servidor.

* Quando o [Modo Estrito](/reference/react/StrictMode) estiver ativado, o React chamará `constructor` duas vezes no desenvolvimento e, em seguida, descartará uma das instâncias. Isso ajuda você a perceber os efeitos colaterais acidentais que precisam ser movidos para fora do `constructor`.

<Note>

Não há um equivalente exato para `constructor` em componentes de função. Para declarar o estado em um componente de função, chame [`useState`.](/reference/react/useState) Para evitar o recálculo do estado inicial, [passe uma função para `useState`.](/reference/react/useState#avoiding-recreating-the-initial-state)

</Note>

---

### `componentDidCatch(error, info)` {/*componentdidcatch*/}

Se você definir `componentDidCatch`, o React o chamará quando algum componente filho (incluindo filhos distantes) lançar um erro durante a renderização. Isso permite que você registre esse erro em um serviço de relatório de erros na produção.

Normalmente, ele é usado em conjunto com [`static getDerivedStateFromError`](#static-getderivedstatefromerror), que permite que você atualize o estado em resposta a um erro e exiba uma mensagem de erro para o usuário. Um componente com esses métodos é chamado de *contorno de erro*.

[Veja um exemplo.](#catching-rendering-errors-with-an-error-boundary)

#### Parâmetros {/*componentdidcatch-parameters*/}

* `error`: O erro que foi lançado. Na prática, geralmente será uma instância de [`Error`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error), mas isso não é garantido porque o JavaScript permite [`throw`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/throw) qualquer valor, incluindo strings ou até mesmo `null`.

* `info`: Um objeto contendo informações adicionais sobre o erro. Seu campo `componentStack` contém um rastreamento de pilha com o componente que lançou, bem como os nomes e localizações de origem de todos os seus componentes pai. Na produção, os nomes dos componentes serão minificados. Se você configurar relatórios de erros de produção, poderá decodificar a pilha de componentes usando sourcemaps da mesma forma que faria para pilhas de erros JavaScript regulares.

#### Retorna {/*componentdidcatch-returns*/}

`componentDidCatch` não deve retornar nada.

#### Ressalvas {/*componentdidcatch-caveats*/}

* No passado, era comum chamar `setState` dentro de `componentDidCatch` para atualizar a interface do usuário e exibir a mensagem de erro de fallback. Isso é obsoleto em favor da definição de [`static getDerivedStateFromError`.](#static-getderivedstatefromerror)

* As versões de produção e desenvolvimento do React são ligeiramente diferentes na forma como `componentDidCatch` lida com erros. No desenvolvimento, os erros serão propagados para `window`, o que significa que qualquer `window.onerror` ou `window.addEventListener('error', callback)` interceptará os erros que foram capturados por `componentDidCatch`. Na produção, em vez disso, os erros não se propagarão, o que significa que qualquer manipulador de erros ancestral receberá apenas erros que não foram explicitamente capturados por `componentDidCatch`.

<Note>

Ainda não há um equivalente direto para `componentDidCatch` em componentes de função. Se você quiser evitar a criação de componentes de classe, escreva um único componente `ErrorBoundary` como o acima e use-o em todo o seu aplicativo. Alternativamente, você pode usar o pacote [`react-error-boundary`](https://github.com/bvaughn/react-error-boundary), que faz isso por você.

</Note>

---

### `componentDidMount()` {/*componentdidmount*/}

Se você definir o método `componentDidMount`, o React o chamará quando seu componente for adicionado *(montado)* à tela. Este é um lugar comum para iniciar a busca de dados, configurar assinaturas ou manipular os nós do DOM.

Se você implementar `componentDidMount`, geralmente precisará implementar outros métodos de ciclo de vida para evitar erros. Por exemplo, se `componentDidMount` ler algum estado ou props, você também precisará implementar [`componentDidUpdate`](#componentdidupdate) para lidar com suas alterações e [`componentWillUnmount`](#componentwillunmount) para limpar o que `componentDidMount` estava fazendo.

```js {6-8}
class ChatRoom extends Component {
  state = {
    serverUrl: 'https://localhost:1234'
  };

  componentDidMount() {
    this.setupConnection();
  }

  componentDidUpdate(prevProps, prevState) {
    if (
      this.props.roomId !== prevProps.roomId ||
      this.state.serverUrl !== prevState.serverUrl
    ) {
      this.destroyConnection();
      this.setupConnection();
    }
  }

  componentWillUnmount() {
    this.destroyConnection();
  }

  // ...
}
```

[Veja mais exemplos.](#adding-lifecycle-methods-to-a-class-component)

#### Parâmetros {/*componentdidmount-parameters*/}

`componentDidMount` não aceita nenhum parâmetro.

#### Retorna {/*componentdidmount-returns*/}

`componentDidMount` não deve retornar nada.

#### Ressalvas {/*componentdidmount-caveats*/}

- Quando o [Modo Estrito](/reference/react/StrictMode) estiver ativado, no desenvolvimento o React chamará `componentDidMount`, em seguida, chamará imediatamente [`componentWillUnmount`,](#componentwillunmount) e em seguida chamará `componentDidMount` novamente. Isso ajuda você a perceber se você esqueceu de implementar `componentWillUnmount` ou se sua lógica não "espelha" totalmente o que o `componentDidMount` faz.

- Embora você possa chamar [`setState`](#setstate) imediatamente em `componentDidMount`, é melhor evitar isso quando você puder. Ele acionará uma renderização extra, mas ela acontecerá antes que o navegador atualize a tela. Isso garante que, embora o [`render`](#render) seja chamado duas vezes neste caso, o usuário não verá o estado intermediário. Use este padrão com cautela, pois ele geralmente causa problemas de desempenho. Na maioria dos casos, você deve ser capaz de atribuir o estado inicial no [`constructor`](#constructor) em vez disso. No entanto, pode ser necessário para casos como modais e dicas de ferramentas quando você precisar medir um nó do DOM antes de renderizar algo que dependa de seu tamanho ou posição.

<Note>

Para muitos casos de uso, definir `componentDidMount`, `componentDidUpdate` e `componentWillUnmount` juntos em componentes de classe é equivalente a chamar [`useEffect`](/reference/react/useEffect) em componentes de função. Nos raros casos em que é importante que o código seja executado antes da pintura do navegador, [`useLayoutEffect`](/reference/react/useLayoutEffect) é uma correspondência mais próxima.

[Veja como migrar.](#migrating-a-component-with-lifecycle-methods-from-a-class-to-a-function)

</Note>

---

### `componentDidUpdate(prevProps, prevState, snapshot?)` {/*componentdidupdate*/}

Se você definir o método `componentDidUpdate`, o React o chamará imediatamente após seu componente ser renderizado novamente com props ou estado atualizados.  Este método não é chamado para a renderização inicial.

Você pode usá-lo para manipular o DOM após uma atualização. Este também é um lugar comum para fazer requisições de rede, desde que você compare as props atuais com as props anteriores (por exemplo, uma requisição de rede pode não ser necessária se as props não foram alteradas). Normalmente, você o usaria em conjunto com [`componentDidMount`](#componentdidmount) e [`componentWillUnmount`:](#componentwillunmount)

```js {10-18}
class ChatRoom extends Component {
  state = {
    serverUrl: 'https://localhost:1234'
  };

  componentDidMount() {
    this.setupConnection();
  }

  componentDidUpdate(prevProps, prevState) {
    if (
      this.props.roomId !== prevProps.roomId ||
      this.state.serverUrl !== prevState.serverUrl
    ) {
      this.destroyConnection();
      this.setupConnection();
    }
  }

  componentWillUnmount() {
    this.destroyConnection();
  }

  // ...
}
```

[Veja mais exemplos.](#adding-lifecycle-methods-to-a-class-component)


#### Parâmetros {/*componentdidupdate-parameters*/}

* `prevProps`: Props antes da atualização. Compare `prevProps` com [`this.props`](#props) para determinar o que mudou.

* `prevState`: Estado antes da atualização. Compare `prevState` com [`this.state`](#state) para determinar o que mudou.

* `snapshot`: Se você implementou [`getSnapshotBeforeUpdate`](#getsnapshotbeforeupdate), `snapshot` conterá o valor que você retornou desse método. Caso contrário, será `undefined`.

#### Retorna {/*componentdidupdate-returns*/}

`componentDidUpdate` não deve retornar nada.

#### Ressalvas {/*componentdidupdate-caveats*/}

- `componentDidUpdate` não será chamado se [`shouldComponentUpdate`](#shouldcomponentupdate) for definido e retornar `false`.

- A lógica dentro de `componentDidUpdate` geralmente deve ser encapsulada em condições que comparam `this.props` com `prevProps` e `this.state` com `prevState`. Caso contrário, há o risco de criar loops infinitos.

- Embora você possa chamar [`setState`](#setstate) imediatamente em `componentDidUpdate`, é melhor evitar isso quando você puder. Ele acionará uma renderização extra, mas ela acontecerá antes que o navegador atualize a tela. Isso garante que, embora o [`render`](#render) seja chamado duas vezes neste caso, o usuário não verá o estado intermediário. Este padrão geralmente causa problemas de desempenho, mas pode ser necessário para casos raros, como modais e dicas de ferramentas, quando você precisa medir um nó do DOM antes de renderizar algo que dependa de seu tamanho ou posição.

<Note>

Para muitos casos de uso, definir `componentDidMount`, `componentDidUpdate` e `componentWillUnmount` juntos em componentes de classe é equivalente a chamar [`useEffect`](/reference/react/useEffect) em componentes de função. Nos raros casos em que é importante que o código seja executado antes da pintura do navegador, [`useLayoutEffect`](/reference/react/useLayoutEffect) é uma correspondência mais próxima.

[Veja como migrar.](#migrating-a-component-with-lifecycle-methods-from-a-class-to-a-function)

</Note>
---

### `componentWillMount()` {/*componentwillmount*/}

<Deprecated>

Esta API foi renomeada de `componentWillMount` para [`UNSAFE_componentWillMount`.](#unsafe_componentwillmount) O nome antigo foi descontinuado. Em uma versão principal futura do React, apenas o novo nome funcionará.

Execute o [`rename-unsafe-lifecycles` codemod](https://github.com/reactjs/react-codemod#rename-unsafe-lifecycles) para atualizar automaticamente seus componentes.

</Deprecated>

---

### `componentWillReceiveProps(nextProps)` {/*componentwillreceiveprops*/}

<Deprecated>

Esta API foi renomeada de `componentWillReceiveProps` para [`UNSAFE_componentWillReceiveProps`.](#unsafe_componentwillreceiveprops) O nome antigo foi descontinuado. Em uma versão principal futura do React, apenas o novo nome funcionará.

Execute o [`rename-unsafe-lifecycles` codemod](https://github.com/reactjs/react-codemod#rename-unsafe-lifecycles) para atualizar automaticamente seus componentes.

</Deprecated>

---

### `componentWillUpdate(nextProps, nextState)` {/*componentwillupdate*/}

<Deprecated>

Esta API foi renomeada de `componentWillUpdate` para [`UNSAFE_componentWillUpdate`.](#unsafe_componentwillupdate) O nome antigo foi descontinuado. Em uma versão principal futura do React, apenas o novo nome funcionará.

Execute o [`rename-unsafe-lifecycles` codemod](https://github.com/reactjs/react-codemod#rename-unsafe-lifecycles) para atualizar automaticamente seus componentes.

</Deprecated>

---

### `componentWillUnmount()` {/*componentwillunmount*/}

Se você definir o método `componentWillUnmount`, o React o chamará antes que seu componente seja removido *(desmontado)* da tela. Este é um lugar comum para cancelar a busca de dados ou remover assinaturas.

A lógica dentro de `componentWillUnmount` deve "espelhar" a lógica dentro do [`componentDidMount`.](#componentdidmount) Por exemplo, se `componentDidMount` configura uma assinatura, `componentWillUnmount` deve limpar essa assinatura. Se a lógica de limpeza em seu `componentWillUnmount` ler algumas props ou estado, você geralmente também precisará implementar [`componentDidUpdate`](#componentdidupdate) para limpar recursos (como assinaturas) correspondentes às props e estado antigos.

```js {20-22}
class ChatRoom extends Component {
  state = {
    serverUrl: 'https://localhost:1234'
  };

  componentDidMount() {
    this.setupConnection();
  }

  componentDidUpdate(prevProps, prevState) {
    if (
      this.props.roomId !== prevProps.roomId ||
      this.state.serverUrl !== prevState.serverUrl
    ) {
      this.destroyConnection();
      this.setupConnection();
    }
  }

  componentWillUnmount() {
    this.destroyConnection();
  }

  // ...
}
```

[Veja mais exemplos.](#adding-lifecycle-methods-to-a-class-component)

#### Parâmetros {/*componentwillunmount-parameters*/}

`componentWillUnmount` não aceita nenhum parâmetro.

#### Retorna {/*componentwillunmount-returns*/}

`componentWillUnmount` não deve retornar nada.

#### Ressalvas {/*componentwillunmount-caveats*/}
``````
- Quando o [Modo Strict](/reference/react/StrictMode) estiver ativado, no desenvolvimento o React chamará [`componentDidMount`](#componentdidmount), então chamará imediatamente `componentWillUnmount` e, em seguida, chamará `componentDidMount` novamente. Isso ajuda a perceber se você esqueceu de implementar `componentWillUnmount` ou se sua lógica não "espelha" totalmente o que `componentDidMount` faz.

<Note>

Para muitos casos de uso, definir `componentDidMount`, `componentDidUpdate` e `componentWillUnmount` juntos em componentes de classe é equivalente a chamar [`useEffect`](/reference/react/useEffect) em componentes de função. Nos raros casos em que é importante que o código seja executado antes da renderização do navegador, [`useLayoutEffect`](/reference/react/useLayoutEffect) é uma correspondência mais próxima.

[Veja como migrar.](#migrating-a-component-with-lifecycle-methods-from-a-class-to-a-function)

</Note>

---

### `forceUpdate(callback?)` {/*forceupdate*/}

Força um componente a renderizar novamente.

Normalmente, isso não é necessário. Se o método [`render`](#render) de seu componente lê apenas de [`this.props`](#props), [`this.state`](#state), ou [`this.context,](#context) ele renderizará novamente automaticamente quando você chamar [`setState`](#setstate) dentro de seu componente ou de um de seus pais. Entretanto, se o método `render` de seu componente ler diretamente de uma fonte de dados externa, você deve dizer ao React para atualizar a interface do usuário quando essa fonte de dados mudar. É isso que `forceUpdate `permite que você faça.

Tente evitar todos os usos de `forceUpdate` e leia apenas de `this.props` e `this.state` em `render`.

#### Parâmetros {/*forceupdate-parameters*/}

* **opcional** `callback` Se especificado, o React chamará o `callback` que você forneceu após a confirmação da atualização.

#### Retorna {/*forceupdate-returns*/}

`forceUpdate` não retorna nada.

#### Ressalvas {/*forceupdate-caveats*/}

- Se você chamar `forceUpdate`, o React irá renderizar novamente sem chamar [`shouldComponentUpdate`.](#shouldcomponentupdate)

<Note>

Ler uma fonte de dados externa e forçar os componentes de classe a renderizar novamente em resposta a suas mudanças com `forceUpdate` foi substituído por [`useSyncExternalStore`](/reference/react/useSyncExternalStore) em componentes de função.

</Note>

---

### `getSnapshotBeforeUpdate(prevProps, prevState)` {/*getsnapshotbeforeupdate*/}

Se você implementar `getSnapshotBeforeUpdate`, React irá chamá-lo imediatamente antes do React atualizar o DOM. Ele permite que seu componente capture algumas informações do DOM (por exemplo, posição da rolagem) antes que ele seja potencialmente alterado. Qualquer valor retornado por este método do ciclo de vida será passado como um parâmetro para [`componentDidUpdate`.](#componentdidupdate)

Por exemplo, você pode usá-lo em uma interface do usuário como um tópico de bate-papo que precisa preservar sua posição de rolagem durante as atualizações:

```js {7-15,17}
class ScrollingList extends React.Component {
  constructor(props) {
    super(props);
    this.listRef = React.createRef();
  }

  getSnapshotBeforeUpdate(prevProps, prevState) {
    // Estamos adicionando novos itens à lista?
    // Capture a posição de rolagem para que possamos ajustar a rolagem mais tarde.
    if (prevProps.list.length < this.props.list.length) {
      const list = this.listRef.current;
      return list.scrollHeight - list.scrollTop;
    }
    return null;
  }

  componentDidUpdate(prevProps, prevState, snapshot) {
    // Se tivermos um valor de snapshot, acabamos de adicionar novos itens.
    // Ajuste a rolagem para que esses novos itens não empurrem os antigos para fora da visualização.
    // (o snapshot aqui é o valor retornado de getSnapshotBeforeUpdate)
    if (snapshot !== null) {
      const list = this.listRef.current;
      list.scrollTop = list.scrollHeight - snapshot;
    }
  }

  render() {
    return (
      <div ref={this.listRef}>{/* ...contents... */}</div>
    );
  }
}
```

No exemplo acima, é importante ler a propriedade `scrollHeight` diretamente em `getSnapshotBeforeUpdate`. Não é seguro lê-la em [`render`](#render), [`UNSAFE_componentWillReceiveProps`](#unsafe_componentwillreceiveprops), ou [`UNSAFE_componentWillUpdate`](#unsafe_componentwillupdate) porque há uma lacuna de tempo potencial entre a chamada desses métodos e a atualização do React no DOM.

#### Parâmetros {/*getsnapshotbeforeupdate-parameters*/}

* `prevProps`: Props antes da atualização. Compare `prevProps` com [`this.props`](#props) para determinar o que mudou.

* `prevState`: State antes da atualização. Compare `prevState` com [`this.state`](#state) para determinar o que mudou.

#### Retorna {/*getsnapshotbeforeupdate-returns*/}

Você deve retornar um valor de snapshot de qualquer tipo que desejar, ou `null`. O valor que você retornou será passado como o terceiro argumento para [`componentDidUpdate`.](#componentdidupdate)

#### Ressalvas {/*getsnapshotbeforeupdate-caveats*/}

- `getSnapshotBeforeUpdate` não será chamado se [`shouldComponentUpdate`](#shouldcomponentupdate) estiver definido e retornar `false`.

<Note>

No momento, não existe um equivalente de `getSnapshotBeforeUpdate` para componentes de função. Este caso de uso é muito incomum, mas se você precisar dele, por enquanto terá que escrever um componente de classe.

</Note>

---

### `render()` {/*render*/}

O método `render` é o único método obrigatório em um componente de classe.

O método `render` deve especificar o que você quer que apareça na tela, por exemplo:

```js {4-6}
import { Component } from 'react';

class Greeting extends Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}
```

O React pode chamar `render` a qualquer momento, então você não deve presumir que ele é executado em um momento específico. Normalmente, o método `render` deve retornar um trecho de [JSX](/learn/writing-markup-with-jsx), mas alguns [outros tipos de retorno](#render-returns) (como strings) são suportados. Para calcular o JSX retornado, o método `render` pode ler [`this.props`](#props), [`this.state`](#state), e [`this.context`](#context).

Você deve escrever o método `render` como uma função pura, o que significa que ele deve retornar o mesmo resultado se props, state e context forem os mesmos. Ele também não deve conter efeitos colaterais (como configurar assinaturas) ou interagir com as APIs do navegador. Efeitos colaterais devem acontecer em manipuladores de eventos ou métodos como [`componentDidMount`.](#componentdidmount)

#### Parâmetros {/*render-parameters*/}

`render` não recebe nenhum parâmetro.

#### Retorna {/*render-returns*/}

`render` pode retornar qualquer nó React válido. Isso inclui elementos React como `<div />`, strings, números, [portais](/reference/react-dom/createPortal), nós vazios (`null`, `undefined`, `true` e `false`) e arrays de nós React.

#### Ressalvas {/*render-caveats*/}

- `render` deve ser escrito como uma função pura de props, state e context. Ele não deve ter efeitos colaterais.

- `render` não será chamado se [`shouldComponentUpdate`](#shouldcomponentupdate) estiver definido e retornar `false`.

- Quando o [Modo Strict](/reference/react/StrictMode) estiver ativado, o React chamará `render` duas vezes no desenvolvimento e, em seguida, descartará um dos resultados. Isso ajuda você a perceber os efeitos colaterais acidentais que precisam ser movidos para fora do método `render`.

- Não há uma correspondência um a um entre a chamada `render` e a subsequente chamada `componentDidMount` ou `componentDidUpdate`. Alguns dos resultados da chamada `render` podem ser descartados pelo React quando for benéfico.

---

### `setState(nextState, callback?)` {/*setstate*/}

Chame `setState` para atualizar o state do seu componente React.

```js {8-10}
class Form extends Component {
  state = {
    name: 'Taylor',
  };

  handleNameChange = (e) => {
    const newName = e.target.value;
    this.setState({
      name: newName
    });
  }

  render() {
    return (
      <>
        <input value={this.state.name} onChange={this.handleNameChange} />
        <p>Hello, {this.state.name}.</p>
      </>
    );
  }
}
```

`setState` enfileira alterações no state do componente. Ele diz ao React que este componente e seus filhos precisam renderizar novamente com o novo state. Esta é a principal forma de atualizar a interface do usuário em resposta às interações.

<Pitfall>

Chamar `setState` **não** muda o state atual no código já em execução:

```js {6}
function handleClick() {
  console.log(this.state.name); // "Taylor"
  this.setState({
    name: 'Robin'
  });
  console.log(this.state.name); // Ainda "Taylor"!
}
```

Ele só afeta o que `this.state` retornará a partir da *próxima* renderização.

</Pitfall>

Você também pode passar uma função para `setState`. Isso permite que você atualize o state com base no state anterior:

```js {2-6}
  handleIncreaseAge = () => {
    this.setState(prevState => {
      return {
        age: prevState.age + 1
      };
    });
  }
```

Você não precisa fazer isso, mas é útil se você quiser atualizar o state várias vezes durante o mesmo evento.

#### Parâmetros {/*setstate-parameters*/}

* `nextState`: Um objeto ou uma função.
  * Se você passar um objeto como `nextState`, ele será superficialmente mesclado em `this.state`.
  * Se você passar uma função como `nextState`, ela será tratada como uma _função atualizadora_. Ela deve ser pura, deve aceitar o state e as props pendentes como argumentos e deve retornar o objeto a ser superficialmente mesclado em `this.state`. O React colocará sua função atualizadora em uma fila e renderizará novamente seu componente. Durante a próxima renderização, o React calculará o próximo state aplicando todos os atualizadores enfileirados ao state anterior.

* **opcional** `callback`: Se especificado, o React chamará o `callback` que você forneceu após a confirmação da atualização.

#### Retorna {/*setstate-returns*/}

`setState` não retorna nada.

#### Ressalvas {/*setstate-caveats*/}

- Pense em `setState` como um *pedido* em vez de um comando imediato para atualizar o componente. Quando vários componentes atualizam seu state em resposta a um evento, o React agrupará suas atualizações e renderizará novamente em conjunto em uma única passagem no final do evento. No raro caso de você precisar forçar uma atualização de state específica a ser aplicada de forma síncrona, você pode encapsulá-la em [`flushSync`,](/reference/react-dom/flushSync) mas isso pode prejudicar o desempenho.

- `setState` não atualiza `this.state` imediatamente. Isso torna a leitura de `this.state` logo após chamar `setState` uma armadilha potencial. Em vez disso, use [`componentDidUpdate`](#componentdidupdate) ou o argumento `callback` de setState, os quais garantem que serão disparados após a aplicação da atualização. Se você precisar definir o state com base no state anterior, você pode passar uma função para `nextState` como descrito acima.

<Note>

Chamar `setState` em componentes de classe é semelhante a chamar uma função [`set`](/reference/react/useState#setstate) em componentes de função.

[Veja como migrar.](#migrating-a-component-with-state-from-a-class-to-a-function)

</Note>

---

### `shouldComponentUpdate(nextProps, nextState, nextContext)` {/*shouldcomponentupdate*/}

Se você definir `shouldComponentUpdate`, o React o chamará para determinar se uma nova renderização pode ser ignorada.

Se você tiver certeza de que quer escrevê-lo manualmente, você pode comparar `this.props` com `nextProps` e `this.state` com `nextState` e retornar `false` para dizer ao React que a atualização pode ser ignorada.

```js {6-18}
class Rectangle extends Component {
  state = {
    isHovered: false
  };

  shouldComponentUpdate(nextProps, nextState) {
    if (
      nextProps.position.x === this.props.position.x &&
      nextProps.position.y === this.props.position.y &&
      nextProps.size.width === this.props.size.width &&
      nextProps.size.height === this.props.size.height &&
      nextState.isHovered === this.state.isHovered
    ) {
      // Nada mudou, então uma nova renderização é desnecessária
      return false;
    }
    return true;
  }

  // ...
}

```

O React chama `shouldComponentUpdate` antes de renderizar quando novas props ou state estão sendo recebidos. O padrão é `true`. Este método não é chamado para a renderização inicial ou quando [`forceUpdate`](#forceupdate) é usado.

#### Parâmetros {/*shouldcomponentupdate-parameters*/}

- `nextProps`: As próximas props que o componente está prestes a renderizar. Compare `nextProps` com [`this.props`](#props) para determinar o que mudou.
- `nextState`: O próximo state com o qual o componente está prestes a renderizar. Compare `nextState` com [`this.state`](#props) para determinar o que mudou.
- `nextContext`: O próximo context com o qual o componente está prestes a renderizar. Compare `nextContext` com [`this.context`](#context) para determinar o que mudou. Disponível apenas se você especificar [`static contextType`](#static-contexttype).

#### Retorna {/*shouldcomponentupdate-returns*/}

Retorne `true` se você quiser que o componente renderize novamente. Esse é o comportamento padrão.

Retorne `false` para dizer ao React que a renderização novamente pode ser ignorada.

#### Ressalvas {/*shouldcomponentupdate-caveats*/}

- Este método *só* existe como uma otimização de desempenho. Se seu componente quebrar sem ele, conserte-o primeiro.

- Considere usar [`PureComponent`](/reference/react/PureComponent) em vez de escrever `shouldComponentUpdate` manualmente. `PureComponent` compara superficialmente props e state, e reduz a chance de você pular uma atualização necessária.

- Não recomendamos fazer verificações de igualdade profunda ou usar `JSON.stringify` em `shouldComponentUpdate`. Isso torna o desempenho imprevisível e dependente da estrutura de dados de cada prop e state. Na melhor das hipóteses, você corre o risco de introduzir interrupções de vários segundos em sua aplicação e, na pior das hipóteses, você corre o risco de travá-la.

- Retornar `false` não impede que os componentes filhos renderizem novamente quando o *state deles* muda.

- Retornar `false` não *garante* que o componente não renderizará novamente. O React usará o valor de retorno como dica, mas ainda pode escolher renderizar novamente seu componente se for lógico fazê-lo por outros motivos.

<Note>

Otimizar componentes de classe com `shouldComponentUpdate` é semelhante a otimizar componentes de função com [`memo`.](/reference/react/memo) Os componentes de função também oferecem uma otimização mais granular com [`useMemo`.](/reference/react/useMemo)

</Note>

---

### `UNSAFE_componentWillMount()` {/*unsafe_componentwillmount*/}

Se você definir `UNSAFE_componentWillMount`, o React o chamará imediatamente após o [`constructor`.](#constructor) Ele só existe por razões históricas e não deve ser usado em nenhum código novo. Em vez disso, use uma das alternativas:

- Para inicializar o state, declare [`state`](#state) como um campo de classe ou defina `this.state` dentro do [`constructor`.](#constructor)
- Se você precisar executar um efeito colateral ou configurar uma assinatura, mova essa lógica para [`componentDidMount`](#componentdidmount) em vez disso.

[Veja exemplos de como migrar de ciclos de vida não seguros.](https://legacy.reactjs.org/blog/2018/03/27/update-on-async-rendering.html#examples)

#### Parâmetros {/*unsafe_componentwillmount-parameters*/}

`UNSAFE_componentWillMount` não recebe nenhum parâmetro.

#### Retorna {/*unsafe_componentwillmount-returns*/}

`UNSAFE_componentWillMount` não deve retornar nada.

#### Ressalvas {/*unsafe_componentwillmount-caveats*/}

- `UNSAFE_componentWillMount` não será chamado se o componente implementar [`static getDerivedStateFromProps`](#static-getderivedstatefromprops) ou [`getSnapshotBeforeUpdate`.](#getsnapshotbeforeupdate)

- Apesar de sua nomenclatura, `UNSAFE_componentWillMount` não garante que o componente *será* montado se seu aplicativo usa recursos modernos do React, como [`Suspense`.](/reference/react/Suspense) Se uma tentativa de renderização for suspensa (por exemplo, porque o código de algum componente filho ainda não foi carregado), o React descartará a árvore em andamento e tentará construir o componente do zero durante a próxima tentativa. É por isso que este método é "não seguro". O código que depende da montagem (como adicionar uma assinatura) deve ir para [`componentDidMount`.](#componentdidmount)

- `UNSAFE_componentWillMount` é o único método do ciclo de vida que é executado durante a [renderização do servidor.](/reference/react-dom/server) Para todos os propósitos práticos, ele é idêntico a [`constructor,](#constructor) então você deve usar o `constructor` para este tipo de lógica em vez disso.

<Note>

Chamar [`setState`](#setstate) dentro de `UNSAFE_componentWillMount` em um componente de classe para inicializar o state é equivalente a passar esse state como o state inicial para [`useState`](/reference/react/useState) em um componente de função.

</Note>

---

### `UNSAFE_componentWillReceiveProps(nextProps, nextContext)` {/*unsafe_componentwillreceiveprops*/}

Se você definir `UNSAFE_componentWillReceiveProps`, o React o chamará quando o componente receber novas props. Ele só existe por razões históricas e não deve ser usado em nenhum código novo. Em vez disso, use uma das alternativas:

- Se você precisar **executar um efeito colateral** (por exemplo, buscar dados, executar uma animação ou reinicializar uma assinatura) em resposta a alterações de prop, mova essa lógica para [`componentDidUpdate`](#componentdidupdate) em vez disso.
- Se você precisar **evitar o recálculo de alguns dados somente quando uma prop muda,** use um [auxiliar de memorização](https://legacy.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#what-about-memoization) em vez disso.
- Se você precisar **"resetar" algum state quando uma prop mudar,** considere fazer um componente [totalmente controlado](https://legacy.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#recommendation-fully-controlled-component) ou [totalmente não controlado com uma chave](https://legacy.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#recommendation-fully-uncontrolled-component-with-a-key) em vez disso.
- Se você precisar **"ajustar" algum state quando uma prop mudar,** verifique se você pode calcular todas as informações necessárias apenas a partir das props durante a renderização. Caso contrário, use [`static getDerivedStateFromProps`](/reference/react/Component#static-getderivedstatefromprops) em vez disso.

[Veja exemplos de como migrar de ciclos de vida não seguros.](https://legacy.reactjs.org/blog/2018/03/27/update-on-async-rendering.html#updating-state-based-on-props)

#### Parâmetros {/*unsafe_componentwillreceiveprops-parameters*/}- `nextProps`: As próximas props que o componente está prestes a receber do seu componente pai. Compare `nextProps` com [`this.props`](#props) para determinar o que mudou.
- `nextContext`: O próximo contexto que o componente está prestes a receber do provedor mais próximo. Compare `nextContext` com [`this.context`](#context) para determinar o que mudou. Disponível apenas se você especificar [`static contextType`](#static-contexttype).

#### Retorna {/*unsafe_componentwillreceiveprops-returns*/}

`UNSAFE_componentWillReceiveProps` não deve retornar nada.

#### Ressalvas {/*unsafe_componentwillreceiveprops-caveats*/}

- `UNSAFE_componentWillReceiveProps` não será chamado se o componente implementar [`static getDerivedStateFromProps`](#static-getderivedstatefromprops) ou [`getSnapshotBeforeUpdate`.](#getsnapshotbeforeupdate)

- Apesar da sua nomenclatura, `UNSAFE_componentWillReceiveProps` não garante que o componente *receberá* essas props se o seu aplicativo usar recursos modernos do React, como [`Suspense`.](/reference/react/Suspense) Se uma tentativa de renderização for suspensa (por exemplo, porque o código de algum componente filho ainda não foi carregado), o React descartará a árvore em andamento e tentará construir o componente do zero durante a próxima tentativa. No momento da próxima tentativa de renderização, as props podem ser diferentes. É por isso que este método é "inseguro". O código que deve ser executado apenas para atualizações confirmadas (como redefinir uma assinatura) deve ir para [`componentDidUpdate`.](#componentdidupdate)

- `UNSAFE_componentWillReceiveProps` não significa que o componente recebeu props *diferentes* da última vez. Você precisa comparar `nextProps` e `this.props` sozinho para verificar se algo mudou.

- O React não chama `UNSAFE_componentWillReceiveProps` com as props iniciais durante a montagem. Ele só chama esse método se algumas das props do componente forem atualizadas. Por exemplo, chamar [`setState`](#setstate) geralmente não aciona `UNSAFE_componentWillReceiveProps` dentro do mesmo componente.

<Note>

Chamar [`setState`](#setstate) dentro de `UNSAFE_componentWillReceiveProps` em um componente de classe para "ajustar" o estado é equivalente a [chamar a função `set` de `useState` durante a renderização](/reference/react/useState#storing-information-from-previous-renders) em um componente de função.

</Note>

---

### `UNSAFE_componentWillUpdate(nextProps, nextState)` {/*unsafe_componentwillupdate*/}

Se você definir `UNSAFE_componentWillUpdate`, o React o chamará antes de renderizar com as novas props ou estado. Ele só existe por razões históricas e não deve ser usado em nenhum código novo. Em vez disso, use uma das alternativas:

- Se você precisar executar um efeito colateral (por exemplo, buscar dados, executar uma animação ou reinicializar uma assinatura) em resposta a alterações de prop ou estado, mova essa lógica para [`componentDidUpdate`](#componentdidupdate).
- Se você precisar ler algumas informações do DOM (por exemplo, para salvar a posição atual da rolagem) para que possa usá-las em [`componentDidUpdate`](#componentdidupdate) mais tarde, leia-o dentro de [`getSnapshotBeforeUpdate`](#getsnapshotbeforeupdate).

[Veja exemplos de migração de ciclos de vida inseguros.](https://legacy.reactjs.org/blog/2018/03/27/update-on-async-rendering.html#examples)

#### Parâmetros {/*unsafe_componentwillupdate-parameters*/}

- `nextProps`: As próximas props que o componente está prestes a renderizar com. Compare `nextProps` com [`this.props`](#props) para determinar o que mudou.
- `nextState`: O próximo estado que o componente está prestes a renderizar com. Compare `nextState` com [`this.state`](#state) para determinar o que mudou.

#### Retorna {/*unsafe_componentwillupdate-returns*/}

`UNSAFE_componentWillUpdate` não deve retornar nada.

#### Ressalvas {/*unsafe_componentwillupdate-caveats*/}

- `UNSAFE_componentWillUpdate` não será chamado se [`shouldComponentUpdate`](#shouldcomponentupdate) for definido e retornar `false`.

- `UNSAFE_componentWillUpdate` não será chamado se o componente implementar [`static getDerivedStateFromProps`](#static-getderivedstatefromprops) ou [`getSnapshotBeforeUpdate`.](#getsnapshotbeforeupdate)

- Não é suportado chamar [`setState`](#setstate) (ou qualquer método que leve à chamada de `setState`, como despachar uma ação do Redux) durante `componentWillUpdate`.

- Apesar da sua nomenclatura, `UNSAFE_componentWillUpdate` não garante que o componente *será* atualizado se seu aplicativo usar recursos modernos do React como [`Suspense`.](/reference/react/Suspense) Se uma tentativa de renderização for suspensa (por exemplo, porque o código de algum componente filho ainda não foi carregado), o React descartará a árvore em andamento e tentará construir o componente do zero durante a próxima tentativa. No momento da próxima tentativa de renderização, as props e o estado podem ser diferentes. É por isso que este método é "inseguro". O código que deve ser executado apenas para atualizações confirmadas (como redefinir uma assinatura) deve ir para [`componentDidUpdate`.](#componentdidupdate)

- `UNSAFE_componentWillUpdate` não significa que o componente recebeu props ou estado *diferentes* da última vez. Você precisa comparar `nextProps` com `this.props` e `nextState` com `this.state` sozinho para verificar se algo mudou.

- O React não chama `UNSAFE_componentWillUpdate` com props e estado iniciais durante a montagem.

<Note>

Não existe um equivalente direto para `UNSAFE_componentWillUpdate` em componentes de função.

</Note>

---

### `static contextType` {/*static-contexttype*/}

Se você quiser ler [`this.context`](#context-instance-field) do seu componente de classe, deve especificar qual contexto ele precisa ler. O contexto que você especifica como `static contextType` deve ser um valor criado anteriormente por [`createContext`.](/reference/react/createContext)

```js {2}
class Button extends Component {
  static contextType = ThemeContext;

  render() {
    const theme = this.context;
    const className = 'button-' + theme;
    return (
      <button className={className}>
        {this.props.children}
      </button>
    );
  }
}
```

<Note>

A leitura de `this.context` em componentes de classe é equivalente a [`useContext`](/reference/react/useContext) em componentes funcionais.

[Veja como migrar.](#migrating-a-component-with-context-from-a-class-to-a-function)

</Note>

---

### `static defaultProps` {/*static-defaultprops*/}

Você pode definir `static defaultProps` para definir as props padrão para a classe. Elas serão usadas para props `undefined` ou ausentes, mas não para props `null`.

Por exemplo, aqui está como você define que a prop `color` deve ter como padrão `'blue'`:

```js {2-4}
class Button extends Component {
  static defaultProps = {
    color: 'blue'
  };

  render() {
    return <button className={this.props.color}>click me</button>;
  }
}
```

Se a prop `color` não for fornecida ou for `undefined`, ela será definida por padrão como `'blue'`:

```js
<>
  {/* this.props.color é "blue" */}
  <Button />

  {/* this.props.color é "blue" */}
  <Button color={undefined} />

  {/* this.props.color é null */}
  <Button color={null} />

  {/* this.props.color é "red" */}
  <Button color="red" />
</>
```

<Note>

A definição de `defaultProps` em componentes de classe é semelhante ao uso de [valores padrão](/learn/passing-props-to-a-component#specifying-a-default-value-for-a-prop) em componentes de função.

</Note>

---

### `static getDerivedStateFromError(error)` {/*static-getderivedstatefromerror*/}

Se você definir `static getDerivedStateFromError`, o React o chamará quando um componente filho (incluindo filhos distantes) lançar um erro durante a renderização. Isso permite que você exiba uma mensagem de erro em vez de limpar a UI.

Normalmente, ele é usado em conjunto com [`componentDidCatch`](#componentdidcatch), que permite que você envie o relatório de erro para algum serviço de análise. Um componente com esses métodos é chamado de *limite de erro*.

[Veja um exemplo.](#catching-rendering-errors-with-an-error-boundary)

#### Parâmetros {/*static-getderivedstatefromerror-parameters*/}

* `error`: O erro que foi lançado. Na prática, geralmente será uma instância de [`Error`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error), mas isso não é garantido, porque o JavaScript permite [`throw`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/throw) qualquer valor, incluindo strings ou até mesmo `null`.

#### Retorna {/*static-getderivedstatefromerror-returns*/}

`static getDerivedStateFromError` deve retornar o estado dizendo ao componente para exibir a mensagem de erro.

#### Ressalvas {/*static-getderivedstatefromerror-caveats*/}

* `static getDerivedStateFromError` deve ser uma função pura. Se você quiser executar um efeito colateral (por exemplo, para chamar um serviço de análise), você também precisa implementar [`componentDidCatch`.](#componentdidcatch)

<Note>

Ainda não existe um equivalente direto para `static getDerivedStateFromError` em componentes de função. Se você quiser evitar a criação de componentes de classe, escreva um único componente `ErrorBoundary` como acima e use-o em todo o seu aplicativo. Como alternativa, use o pacote [`react-error-boundary`](https://github.com/bvaughn/react-error-boundary), que faz isso.

</Note>

---

### `static getDerivedStateFromProps(props, state)` {/*static-getderivedstatefromprops*/}

Se você definir `static getDerivedStateFromProps`, o React o chamará imediatamente antes de chamar [`render,](#render) tanto na montagem inicial quanto nas atualizações subsequentes. Ele deve retornar um objeto para atualizar o estado ou `null` para não atualizar nada.

Este método existe para [casos de uso raros](https://legacy.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#when-to-use-derived-state), onde o estado depende de mudanças nas props ao longo do tempo. Por exemplo, este componente `Form` redefine o estado `email` quando a prop `userID` muda:

```js {7-18}
class Form extends Component {
  state = {
    email: this.props.defaultEmail,
    prevUserID: this.props.userID
  };

  static getDerivedStateFromProps(props, state) {
    // Sempre que o usuário atual mudar,
    // Redefina qualquer parte do estado que esteja vinculada a esse usuário.
    // Neste exemplo simples, é apenas o email.
    if (props.userID !== state.prevUserID) {
      return {
        prevUserID: props.userID,
        email: props.defaultEmail
      };
    }
    return null;
  }

  // ...
}
```

Observe que esse padrão exige que você mantenha um valor anterior da prop (como `userID`) no estado (como `prevUserID`).

<Pitfall>

Derivar o estado leva a um código verbose e dificulta a reflexão sobre seus componentes. [Certifique-se de estar familiarizado com alternativas mais simples:](https://legacy.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html)

- Se você precisar **executar um efeito colateral** (por exemplo, busca de dados ou uma animação) em resposta a uma alteração nas props, use o método [`componentDidUpdate`](#componentdidupdate).
- Se você quiser **recomputar alguns dados apenas quando uma prop muda**, [use um auxiliar de memorização em vez disso.](https://legacy.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#what-about-memoization)
- Se você quiser **"redefinir" algum estado quando uma prop muda**, considere tornar um componente [totalmente controlado](https://legacy.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#recommendation-fully-controlled-component) ou [totalmente não controlado com uma chave](https://legacy.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#recommendation-fully-uncontrolled-component-with-a-key).

</Pitfall>

#### Parâmetros {/*static-getderivedstatefromprops-parameters*/}

- `props`: As próximas props que o componente está prestes a renderizar com.
- `state`: O próximo estado que o componente está prestes a renderizar com.

#### Retorna {/*static-getderivedstatefromprops-returns*/}

`static getDerivedStateFromProps` retorna um objeto para atualizar o estado ou `null` para não atualizar nada.

#### Ressalvas {/*static-getderivedstatefromprops-caveats*/}

- Este método é acionado em *cada* renderização, independentemente da causa. Isso é diferente de [`UNSAFE_componentWillReceiveProps`](#unsafe_cmoponentwillreceiveprops), que só é acionado quando o pai causa uma nova renderização e não como resultado de um `setState` local.

- Este método não tem acesso à instância do componente. Se você quiser, é possível reutilizar algum código entre `static getDerivedStateFromProps` e os outros métodos de classe, extraindo funções puras das props e do estado do componente fora da definição da classe.

<Note>

A implementação de `static getDerivedStateFromProps` em um componente de classe é equivalente a [chamar a função `set` de `useState` durante a renderização](/reference/react/useState#storing-information-from-previous-renders) em um componente de função.

</Note>

---

## Uso {/*usage*/}

### Definindo um componente de classe {/*defining-a-class-component*/}

Para definir um componente React como uma classe, estenda a classe `Component` integrada e defina um método [`render`:](#render)

```js
import { Component } from 'react';

class Greeting extends Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}
```

O React chamará seu método [`render`](#render) sempre que precisar descobrir o que exibir na tela. Normalmente, você retornará algum [JSX](/learn/writing-markup-with-jsx) dele. Seu método `render` deve ser uma [função pura:](https://en.wikipedia.org/wiki/Pure_function) ele deve apenas calcular o JSX.

Semelhante a [componentes de função,](/learn/your-first-component#defining-a-component) um componente de classe pode [receber informações por props](/learn/your-first-component#defining-a-component) do seu componente pai. No entanto, a sintaxe para ler as props é diferente. Por exemplo, se o componente pai renderizar `<Greeting name="Taylor" />`, você poderá ler a prop `name` de [`this.props`](#props), como `this.props.name`:

<Sandpack>

```js
import { Component } from 'react';

class Greeting extends Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}

export default function App() {
  return (
    <>
      <Greeting name="Sara" />
      <Greeting name="Cahal" />
      <Greeting name="Edite" />
    </>
  );
}
```

</Sandpack>

Observe que Hooks (funções começando com `use`, como [`useState`](/reference/react/useState)) não são suportados dentro de componentes de classe.

<Pitfall>

Recomendamos definir componentes como funções em vez de classes. [Veja como migrar.](#migrating-a-simple-component-from-a-class-to-a-function)

</Pitfall>

---

### Adicionando estado a um componente de classe {/*adding-state-to-a-class-component*/}

Para adicionar [estado](/learn/state-a-components-memory) a uma classe, atribua um objeto a uma propriedade chamada [`state`](#state). Para atualizar o estado, chame [`this.setState`](#setstate).

<Sandpack>

```js
import { Component } from 'react';

export default class Counter extends Component {
  state = {
    name: 'Taylor',
    age: 42,
  };

  handleNameChange = (e) => {
    this.setState({
      name: e.target.value
    });
  }

  handleAgeChange = () => {
    this.setState({
      age: this.state.age + 1 
    });
  };

  render() {
    return (
      <>
        <input
          value={this.state.name}
          onChange={this.handleNameChange}
        />
        <button onClick={this.handleAgeChange}>
          Increment age
        </button>
        <p>Hello, {this.state.name}. You are {this.state.age}.</p>
      </>
    );
  }
}
```

```css
button { display: block; margin-top: 10px; }
```

</Sandpack>

<Pitfall>

Recomendamos definir componentes como funções em vez de classes. [Veja como migrar.](#migrating-a-component-with-state-from-a-class-to-a-function)

</Pitfall>

---

### Adicionando métodos de ciclo de vida a um componente de classe {/*adding-lifecycle-methods-to-a-class-component*/}

Existem alguns métodos especiais que você pode definir em sua classe.

Se você definir o método [`componentDidMount`](#componentdidmount), o React o chamará quando seu componente for adicionado *(montado)* à tela. O React chamará [`componentDidUpdate`](#componentdidupdate) após seu componente renderizar novamente devido a props ou estados alterados. O React chamará [`componentWillUnmount`](#componentwillunmount) depois que seu componente tiver sido removido *(desmontado)* da tela.

Se você implementar `componentDidMount`, geralmente precisará implementar todos os três ciclos de vida para evitar erros. Por exemplo, se `componentDidMount` lê algum estado ou props, você também deve implementar `componentDidUpdate` para lidar com suas alterações e `componentWillUnmount` para limpar o que `componentDidMount` estava fazendo.

Por exemplo, este componente `ChatRoom` mantém uma conexão de bate-papo sincronizada com props e estado:

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [show, setShow] = useState(false);
  return (
    <>
      <label>
        Escolha a sala de chat:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">geral</option>
          <option value="travel">viagem</option>
          <option value="music">música</option>
        </select>
      </label>
      <button onClick={() => setShow(!show)}>
        {show ? 'Fechar bate-papo' : 'Abrir bate-papo'}
      </button>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}
```

```js src/ChatRoom.js active
import { Component } from 'react';
import { createConnection } from './chat.js';

export default class ChatRoom extends Component {
  state = {
    serverUrl: 'https://localhost:1234'
  };

  componentDidMount() {
    this.setupConnection();
  }

  componentDidUpdate(prevProps, prevState) {
    if (
      this.props.roomId !== prevProps.roomId ||
      this.state.serverUrl !== prevState.serverUrl
    ) {
      this.destroyConnection();
      this.setupConnection();
    }
  }
```
```js
  componentWillUnmount() {
    this.destroyConnection();
  }

  setupConnection() {
    this.connection = createConnection(
      this.state.serverUrl,
      this.props.roomId
    );
    this.connection.connect();    
  }

  destroyConnection() {
    this.connection.disconnect();
    this.connection = null;
  }

  render() {
    return (
      <>
        <label>
          URL do servidor:{' '}
          <input
            value={this.state.serverUrl}
            onChange={e => {
              this.setState({
                serverUrl: e.target.value
              });
            }}
          />
        </label>
        <h1>Bem-vindo(a) ao {this.props.roomId} sala!</h1>
      </>
    );
  }
}
```

```js src/chat.js
export function createConnection(serverUrl, roomId) {
  // A real implementation would actually connect to the server
  return {
    connect() {
      console.log('✅ Conectando à sala "' + roomId + '" em ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Desconectado da sala "' + roomId + '" em ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

Observe que, em desenvolvimento, quando o [Strict Mode](/reference/react/StrictMode) está ativado, o React irá chamar `componentDidMount`, chamar imediatamente `componentWillUnmount` e, em seguida, chamar `componentDidMount` novamente. Isso ajuda você a notar se você se esqueceu de implementar `componentWillUnmount` ou se sua lógica não "espelha" totalmente o que `componentDidMount` faz.

<Pitfall>

Recomendamos definir componentes como funções em vez de classes. [Veja como migrar.](#migrating-a-component-with-lifecycle-methods-from-a-class-to-a-function)

</Pitfall>

---

### Capturando erros de renderização com um limite de erro {/*catching-rendering-errors-with-an-error-boundary*/}

Por padrão, se seu aplicativo lançar um erro durante a renderização, o React removerá sua UI da tela. Para evitar isso, você pode encapsular uma parte da sua UI em um *limite de erro*. Um limite de erro é um componente especial que permite exibir alguma UI de fallback em vez da parte que travou -- por exemplo, uma mensagem de erro.

Para implementar um componente de limite de erro, você precisa fornecer [`static getDerivedStateFromError`](#static-getderivedstatefromerror), que permite que você atualize o estado em resposta a um erro e exiba uma mensagem de erro ao usuário. Você também pode implementar opcionalmente [`componentDidCatch`](#componentdidcatch) para adicionar alguma lógica extra, por exemplo, para registrar o erro em um serviço de análise.

```js {7-10,12-19}
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // Update state so the next render will show the fallback UI.
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    // Example "componentStack":
    //   in ComponentThatThrows (created by App)
    //   in ErrorBoundary (created by App)
    //   in div (created by App)
    //   in App
    logErrorToMyService(error, info.componentStack);
  }

  render() {
    if (this.state.hasError) {
      // You can render any custom fallback UI
      return this.props.fallback;
    }

    return this.props.children;
  }
}
```

Então, você pode envolver uma parte da árvore de componentes com ele:

```js {1,3}
<ErrorBoundary fallback={<p>Algo deu errado</p>}>
  <Profile />
</ErrorBoundary>
```

Se `Profile` ou seu componente filho lançar um erro, `ErrorBoundary` "capturará" esse erro, exibirá uma UI de fallback com a mensagem de erro que você forneceu e enviará um relatório de erro de produção para seu serviço de relatório de erros.

Você não precisa envolver todos os componentes em um limite de erro separado. Quando você pensa na [granularidade dos limites de erro,](https://www.brandondail.com/posts/fault-tolerance-react) considere onde faz sentido exibir uma mensagem de erro. Por exemplo, em um aplicativo de mensagens, faz sentido colocar um limite de erro em torno da lista de conversas. Também faz sentido colocar um em cada mensagem individual. No entanto, não faria sentido colocar um limite em torno de cada avatar.

<Note>

Atualmente, não há como escrever um limite de erro como um componente de função. No entanto, você não precisa escrever a classe de limite de erro sozinho. Por exemplo, você pode usar [`react-error-boundary`](https://github.com/bvaughn/react-error-boundary) em vez disso.

</Note>

---

## Alternativas {/*alternatives*/}

### Migrando um componente simples de uma classe para uma função {/*migrating-a-simple-component-from-a-class-to-a-function*/}

Normalmente, você [define componentes como funções](/learn/your-first-component#defining-a-component) em vez disso.

Por exemplo, suponha que você esteja convertendo este componente de classe `Greeting` em uma função:

<Sandpack>

```js
import { Component } from 'react';

class Greeting extends Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}

export default function App() {
  return (
    <>
      <Greeting name="Sara" />
      <Greeting name="Cahal" />
      <Greeting name="Edite" />
    </>
  );
}
```

</Sandpack>

Defina uma função chamada `Greeting`. É aqui que você moverá o corpo da sua função `render`.

```js
function Greeting() {
  // ... move the code from the render method here ...
}
```

Em vez de `this.props.name`, defina a prop `name` [usando a sintaxe de desestruturação](/learn/passing-props-to-a-component) e leia-a diretamente:

```js
function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>;
}
```

Aqui está um exemplo completo:

<Sandpack>

```js
function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>;
}

export default function App() {
  return (
    <>
      <Greeting name="Sara" />
      <Greeting name="Cahal" />
      <Greeting name="Edite" />
    </>
  );
}
```

</Sandpack>

---

### Migrando um componente com estado de uma classe para uma função {/*migrating-a-component-with-state-from-a-class-to-a-function*/}

Suponha que você esteja convertendo este componente de classe `Counter` em uma função:

<Sandpack>

```js
import { Component } from 'react';

export default class Counter extends Component {
  state = {
    name: 'Taylor',
    age: 42,
  };

  handleNameChange = (e) => {
    this.setState({
      name: e.target.value
    });
  }

  handleAgeChange = (e) => {
    this.setState({
      age: this.state.age + 1 
    });
  };

  render() {
    return (
      <>
        <input
          value={this.state.name}
          onChange={this.handleNameChange}
        />
        <button onClick={this.handleAgeChange}>
          Increment age
        </button>
        <p>Hello, {this.state.name}. You are {this.state.age}.</p>
      </>
    );
  }
}
```

```css
button { display: block; margin-top: 10px; }
```

</Sandpack>

Comece declarando uma função com as [variáveis de estado](/reference/react/useState#adding-state-to-a-component) necessárias:

```js {4-5}
import { useState } from 'react';

function Counter() {
  const [name, setName] = useState('Taylor');
  const [age, setAge] = useState(42);
  // ...
```

Em seguida, converta os manipuladores de eventos:

```js {5-7,9-11}
function Counter() {
  const [name, setName] = useState('Taylor');
  const [age, setAge] = useState(42);

  function handleNameChange(e) {
    setName(e.target.value);
  }

  function handleAgeChange() {
    setAge(age + 1);
  }
  // ...
```

Finalmente, substitua todas as referências começando com `this` pelas variáveis e funções que você definiu em seu componente. Por exemplo, substitua `this.state.age` por `age` e substitua `this.handleNameChange` por `handleNameChange`.

Aqui está um componente totalmente convertido:

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [name, setName] = useState('Taylor');
  const [age, setAge] = useState(42);

  function handleNameChange(e) {
    setName(e.target.value);
  }

  function handleAgeChange() {
    setAge(age + 1);
  }

  return (
    <>
      <input
        value={name}
        onChange={handleNameChange}
      />
      <button onClick={handleAgeChange}>
        Increment age
      </button>
      <p>Hello, {name}. You are {age}.</p>
    </>
  )
}
```

```css
button { display: block; margin-top: 10px; }
```

</Sandpack>

---

### Migrando um componente com métodos de ciclo de vida de uma classe para uma função {/*migrating-a-component-with-lifecycle-methods-from-a-class-to-a-function*/}

Suponha que você esteja convertendo este componente de classe `ChatRoom` com métodos de ciclo de vida em uma função:

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [show, setShow] = useState(false);
  return (
    <>
      <label>
        Choose the chat room:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <button onClick={() => setShow(!show)}>
        {show ? 'Close chat' : 'Open chat'}
      </button>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}
```

```js src/ChatRoom.js active
import { Component } from 'react';
import { createConnection } from './chat.js';

export default class ChatRoom extends Component {
  state = {
    serverUrl: 'https://localhost:1234'
  };

  componentDidMount() {
    this.setupConnection();
  }

  componentDidUpdate(prevProps, prevState) {
    if (
      this.props.roomId !== prevProps.roomId ||
      this.state.serverUrl !== prevState.serverUrl
    ) {
      this.destroyConnection();
      this.setupConnection();
    }
  }

  componentWillUnmount() {
    this.destroyConnection();
  }

  setupConnection() {
    this.connection = createConnection(
      this.state.serverUrl,
      this.props.roomId
    );
    this.connection.connect();    
  }

  destroyConnection() {
    this.connection.disconnect();
    this.connection = null;
  }

  render() {
    return (
      <>
        <label>
          URL do servidor:{' '}
          <input
            value={this.state.serverUrl}
            onChange={e => {
              this.setState({
                serverUrl: e.target.value
              });
            }}
          />
        </label>
        <h1>Bem-vindo(a) ao {this.props.roomId} sala!</h1>
      </>
    );
  }
}
```

```js src/chat.js
export function createConnection(serverUrl, roomId) {
  // A real implementation would actually connect to the server
  return {
    connect() {
      console.log('✅ Conectando à sala "' + roomId + '" em ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Desconectado da sala "' + roomId + '" em ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

Primeiro, verifique se seu [`componentWillUnmount`](#componentwillunmount) faz o oposto de [`componentDidMount`.](#componentdidmount) No exemplo acima, isso é verdade: ele desconecta a conexão que `componentDidMount` configura. Se essa lógica estiver faltando, adicione-a primeiro.

Em seguida, verifique se seu método [`componentDidUpdate`](#componentdidupdate) lida com as alterações em quaisquer props e estado que você está usando em `componentDidMount`. No exemplo acima, `componentDidMount` chama `setupConnection`, que lê `this.state.serverUrl` e `this.props.roomId`. É por isso que `componentDidUpdate` verifica se `this.state.serverUrl` e `this.props.roomId` foram alterados e redefine a conexão se foram. Se sua lógica `componentDidUpdate` estiver faltando ou não lidar com as alterações em todas as props e estado relevantes, corrija isso primeiro.

No exemplo acima, a lógica dentro dos métodos de ciclo de vida conecta o componente a um sistema fora do React (um servidor de bate-papo). Para conectar um componente a um sistema externo, [descreva essa lógica como um único Effect:](/reference/react/useEffect#connecting-to-an-external-system)

```js {6-12}
import { useState, useEffect } from 'react';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]);

  // ...
}
```

Essa chamada [`useEffect`](/reference/react/useEffect) é equivalente à lógica nos métodos de ciclo de vida acima. Se seus métodos de ciclo de vida fazem várias coisas não relacionadas, [divida-os em vários Effects independentes.](/learn/removing-effect-dependencies#is-your-effect-doing-several-unrelated-things) Aqui está um exemplo completo com o qual você pode brincar:

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [show, setShow] = useState(false);
  return (
    <>
      <label>
        Escolha a sala de bate-papo:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">geral</option>
          <option value="travel">viagem</option>
          <option value="music">música</option>
        </select>
      </label>
      <button onClick={() => setShow(!show)}>
        {show ? 'Fechar bate-papo' : 'Abrir bate-papo'}
      </button>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}
```

```js src/ChatRoom.js active
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId, serverUrl]);

  return (
    <>
      <label>
        URL do servidor:{' '}
        <input
          value={serverUrl}
          onChange={e => setServerUrl(e.target.value)}
        />
      </label>
      <h1>Bem-vindo(a) ao {roomId} sala!</h1>
    </>
  );
}
```

```js src/chat.js
export function createConnection(serverUrl, roomId) {
  // A real implementation would actually connect to the server
  return {
    connect() {
      console.log('✅ Conectando à sala "' + roomId + '" em ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Desconectado da sala "' + roomId + '" em ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

<Note>

Se seu componente não sincroniza com nenhum sistema externo, [você pode não precisar de um Effect.](/learn/you-might-not-need-an-effect)

</Note>

---

### Migrando um componente com contexto de uma classe para uma função {/*migrating-a-component-with-context-from-a-class-to-a-function*/}

Neste exemplo, os componentes de classe `Panel` e `Button` leem [contexto](/learn/passing-data-deeply-with-context) de [`this.context`:](#context)

<Sandpack>

```js
import { createContext, Component } from 'react';

const ThemeContext = createContext(null);

class Panel extends Component {
  static contextType = ThemeContext;

  render() {
    const theme = this.context;
    const className = 'panel-' + theme;
    return (
      <section className={className}>
        <h1>{this.props.title}</h1>
        {this.props.children}
      </section>
    );    
  }
}

class Button extends Component {
  static contextType = ThemeContext;

  render() {
    const theme = this.context;
    const className = 'button-' + theme;
    return (
      <button className={className}>
        {this.props.children}
      </button>
    );
  }
}

function Form() {
  return (
    <Panel title="Welcome">
      <Button>Sign up</Button>
      <Button>Log in</Button>
    </Panel>
  );
}

export default function MyApp() {
  return (
    <ThemeContext.Provider value="dark">
      <Form />
    </ThemeContext.Provider>
  )
}
```

```css
.panel-light,
.panel-dark {
  border: 1px solid black;
  border-radius: 4px;
  padding: 20px;
}
.panel-light {
  color: #222;
  background: #fff;
}

.panel-dark {
  color: #fff;
  background: rgb(23, 32, 42);
}

.button-light,
.button-dark {
  border: 1px solid #777;
  padding: 5px;
  margin-right: 10px;
  margin-top: 10px;
}

.button-dark {
  background: #222;
  color: #fff;
}

.button-light {
  background: #fff;
  color: #222;
}
```

</Sandpack>

Quando você os converte em componentes de função, substitua `this.context` por chamadas [`useContext`](/reference/react/useContext):

<Sandpack>

```js
import { createContext, useContext } from 'react';

const ThemeContext = createContext(null);

function Panel({ title, children }) {
  const theme = useContext(ThemeContext);
  const className = 'panel-' + theme;
  return (
    <section className={className}>
      <h1>{title}</h1>
      {children}
    </section>
  )
}

function Button({ children }) {
  const theme = useContext(ThemeContext);
  const className = 'button-' + theme;
  return (
    <button className={className}>
      {children}
    </button>
  );
}

function Form() {
  return (
    <Panel title="Welcome">
      <Button>Sign up</Button>
      <Button>Log in</Button>
    </Panel>
  );
}

export default function MyApp() {
  return (
    <ThemeContext.Provider value="dark">
      <Form />
    </ThemeContext.Provider>
  )
}
```

```css
.panel-light,
.panel-dark {
  border: 1px solid black;
  border-radius: 4px;
  padding: 20px;
}
.panel-light {
  color: #222;
  background: #fff;
}

.panel-dark {
  color: #fff;
  background: rgb(23, 32, 42);
}

.button-light,
.button-dark {
  border: 1px solid #777;
  padding: 5px;
  margin-right: 10px;
  margin-top: 10px;
}

.button-dark {
  background: #222;
  color: #fff;
}

.button-light {
  background: #fff;
  color: #222;
}
```

</Sandpack>
