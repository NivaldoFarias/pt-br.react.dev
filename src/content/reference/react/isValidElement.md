---
title: isValidElement
---

<Intro>

`isValidElement` verifica se um valor é um Elemento React.

```js
const isElement = isValidElement(value)
```

</Intro>

<InlineToc />

---

## Referência {/*reference*/}

### `isValidElement(value)` {/*isvalidelement*/}

Chame `isValidElement(value)` para verificar se `value` é um Elemento React.

```js
import { isValidElement, createElement } from 'react';

// ✅ Elementos React
console.log(isValidElement(<p />)); // true
console.log(isValidElement(createElement('p'))); // true

// ❌ Não são Elementos React
console.log(isValidElement(25)); // false
console.log(isValidElement('Hello')); // false
console.log(isValidElement({ age: 42 })); // false
```

[Veja mais exemplos abaixo.](#usage)

#### Parâmetros {/*parameters*/}

* `value`: O `value` que você deseja verificar. Pode ser qualquer valor de qualquer tipo.

#### Retornos {/*returns*/}

`isValidElement` retorna `true` se o `value` é um Elemento React. Caso contrário, retorna `false`.

#### Ressalvas {/*caveats*/}

*   **Apenas [tags JSX](/learn/writing-markup-with-jsx) e objetos retornados por [`createElement`](/reference/react/createElement) são considerados Elementos React.** Por exemplo, embora um número como `42` seja um *nó* React válido (e possa ser retornado de um componente), ele não é um Elemento React válido. Arrays e portais criados com [`createPortal`](/reference/react-dom/createPortal) também *não* são considerados Elementos React.

---

## Uso {/*usage*/}

### Verificando se algo é um Elemento React {/*checking-if-something-is-a-react-element*/}

Chame `isValidElement` para verificar se algum valor é um *Elemento React.*

Elementos React são:

*   Valores produzidos pela escrita de uma [tag JSX](/learn/writing-markup-with-jsx)
*   Valores produzidos pela chamada de [`createElement`](/reference/react/createElement)

Para Elementos React, `isValidElement` retorna `true`:

```js
import { isValidElement, createElement } from 'react';

// ✅ Tags JSX são Elementos React
console.log(isValidElement(<p />)); // true
console.log(isValidElement(<MyComponent />)); // true

// ✅ Valores retornados por createElement são Elementos React
console.log(isValidElement(createElement('p'))); // true
console.log(isValidElement(createElement(MyComponent))); // true
```

Quaisquer outros valores, como strings, números ou objetos e arrays arbitrários, não são Elementos React.

Para eles, `isValidElement` retorna `false`:

```js
// ❌ Estes *não* são Elementos React
console.log(isValidElement(null)); // false
console.log(isValidElement(25)); // false
console.log(isValidElement('Hello')); // false
console.log(isValidElement({ age: 42 })); // false
console.log(isValidElement([<div />, <div />])); // false
console.log(isValidElement(MyComponent)); // false
```

É muito incomum precisar de `isValidElement`. É principalmente útil se você estiver chamando outra API que *somente* aceita elementos (como [`cloneElement`](/reference/react/cloneElement) faz) e você deseja evitar um erro quando seu argumento não é um Elemento React.

A menos que você tenha alguma razão muito específica para adicionar uma verificação `isValidElement`, você provavelmente não precisa dela.

<DeepDive>

#### Elementos React vs nós React {/*react-elements-vs-react-nodes*/}

Quando você escreve um componente, você pode retornar qualquer tipo de *nó React* dele:

```js
function MyComponent() {
  // ... você pode retornar qualquer nó React ...
}
```

Um nó React pode ser:

*   Um Elemento React criado como `<div />` ou `createElement('div')`
*   Um portal criado com [`createPortal`](/reference/react-dom/createPortal)
*   Uma string
*   Um número
*   `true`, `false`, `null` ou `undefined` (que não são exibidos)
*   Um array de outros nós React

**Observação: `isValidElement` verifica se o argumento é um *Elemento React,* não se é um nó React.** Por exemplo, `42` não é um Elemento React válido. No entanto, é um nó React perfeitamente válido:

```js
function MyComponent() {
  return 42; // Tudo bem retornar um número de um componente
}
```

É por isso que você não deve usar `isValidElement` como uma maneira de verificar se algo pode ser renderizado.

</DeepDive>
