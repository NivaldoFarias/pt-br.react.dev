---
title: createRef
---

<Pitfall>

`createRef` é usado principalmente para [componentes de classe.](/reference/react/Component) Componentes de função geralmente dependem de [`useRef`](/reference/react/useRef) em vez disso.

</Pitfall>

<Intro>

`createRef` cria um objeto [ref](/learn/referencing-values-with-refs) que pode conter valor arbitrário.

```js
class MyInput extends Component {
  inputRef = createRef();
  // ...
}
```

</Intro>

<InlineToc />

---

## Referência {/*reference*/}

### `createRef()` {/*createref*/}

Chame `createRef` para declarar uma [ref](/learn/referencing-values-with-refs) dentro de um [componente de classe.](/reference/react/Component)

```js
import { createRef, Component } from 'react';

class MyComponent extends Component {
  intervalRef = createRef();
  inputRef = createRef();
  // ...
```

[Veja mais exemplos abaixo.](#usage)

#### Parâmetros {/*parameters*/}

`createRef` não recebe parâmetros.

#### Retorna {/*returns*/}

`createRef` retorna um objeto com uma única propriedade:

* `current`: Inicialmente, é definido como `null`. Você pode mais tarde defini-lo para outra coisa. Se você passar o objeto ref para o React como um atributo `ref` para um nó JSX, o React definirá sua propriedade `current`.

#### Ressalvas {/*caveats*/}

* `createRef` sempre retorna um objeto *diferente*. É equivalente a escrever `{ current: null }` por conta própria.
* Em um componente de função, você provavelmente vai querer [`useRef`](/reference/react/useRef) em vez disso, que sempre retorna o mesmo objeto.
* `const ref = useRef()` é equivalente a `const [ref, _] = useState(() => createRef(null))`.

---

## Uso {/*usage*/}

### Declarando uma ref em um componente de classe {/*declaring-a-ref-in-a-class-component*/}

Para declarar uma ref dentro de um [componente de classe,](/reference/react/Component) chame `createRef` e atribua seu resultado a um campo de classe:

```js {4}
import { Component, createRef } from 'react';

class Form extends Component {
  inputRef = createRef();

  // ...
}
```

Se você agora passar `ref={this.inputRef}` para um `<input>` no seu JSX, o React preencherá `this.inputRef.current` com o nó DOM do input.  Por exemplo, aqui está como criar um botão que foca no input:

<Sandpack>

```js
import { Component, createRef } from 'react';

export default class Form extends Component {
  inputRef = createRef();

  handleClick = () => {
    this.inputRef.current.focus();
  }

  render() {
    return (
      <>
        <input ref={this.inputRef} />
        <button onClick={this.handleClick}>
          Focus the input
        </button>
      </>
    );
  }
}
```

</Sandpack>

<Pitfall>

`createRef` é usado principalmente para [componentes de classe.](/reference/react/Component) Componentes de função geralmente dependem de [`useRef`](/reference/react/useRef) em vez disso.

</Pitfall>

---

## Alternativas {/*alternatives*/}

### Migrando de uma classe com `createRef` para uma função com `useRef` {/*migrating-from-a-class-with-createref-to-a-function-with-useref*/}

Recomendamos usar componentes de função em vez de [componentes de classe](/reference/react/Component) em código novo. Se você tem alguns componentes de classe existentes usando `createRef`, aqui está como você pode convertê-los. Este é o código original:

<Sandpack>

```js
import { Component, createRef } from 'react';

export default class Form extends Component {
  inputRef = createRef();

  handleClick = () => {
    this.inputRef.current.focus();
  }

  render() {
    return (
      <>
        <input ref={this.inputRef} />
        <button onClick={this.handleClick}>
          Focus the input
        </button>
      </>
    );
  }
}
```

</Sandpack>

Quando você [converte este componente de uma classe para uma função,](/reference/react/Component#alternatives) substitua chamadas para `createRef` com chamadas para [`useRef`:](/reference/react/useRef)

<Sandpack>

```js
import { useRef } from 'react';

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```

</Sandpack>
```