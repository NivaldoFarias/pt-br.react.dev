---
title: cloneElement
---

<Pitfall>

Usar `cloneElement` não é comum e pode levar a um código frágil. [Veja alternativas comuns.](#alternatives)

</Pitfall>

<Intro>

`cloneElement` permite que você crie um novo Elemento React usando outro elemento como ponto de partida.

```js
const clonedElement = cloneElement(element, props, ...children)
```

</Intro>

<InlineToc />

---

## Referência {/*reference*/}

### `cloneElement(element, props, ...children)` {/*cloneelement*/}

Chame `cloneElement` para criar um Elemento React com base no `element`, mas com `props` e `children` diferentes:

```js
import { cloneElement } from 'react';

// ...
const clonedElement = cloneElement(
  <Row title="Cabbage">
    Hello
  </Row>,
  { isHighlighted: true },
  'Goodbye'
);

console.log(clonedElement); // <Row title="Cabbage" isHighlighted={true}>Goodbye</Row>
```

[Veja mais exemplos abaixo.](#usage)

#### Parâmetros {/*parameters*/}

* `element`: O argumento `element` deve ser um Elemento React válido. Por exemplo, pode ser um nó JSX como `<Something />`, o resultado de chamar [`createElement`](/reference/react/createElement), ou o resultado de outra chamada `cloneElement`.

* `props`: O argumento `props` deve ser um objeto ou `null`. Se você passar `null`, o elemento clonado irá reter todas as `element.props` originais. Caso contrário, para cada prop no objeto `props`, o elemento retornado "preferirá" o valor de `props` ao valor de `element.props`. O restante das props serão preenchidas a partir das `element.props` originais. Se você passar `props.key` ou `props.ref`, eles substituirão os originais.

* **opcional** `...children`: Zero ou mais nós filhos. Eles podem ser quaisquer nós React, incluindo Elementos React, strings, números, [portais](/reference/react-dom/createPortal), nós vazios (`null`, `undefined`, `true`, e `false`), e arrays de nós React. Se você não passar nenhum argumento `...children`, o `element.props.children` original será preservado.

#### Retorna {/*returns*/}

`cloneElement` retorna um objeto de Elemento React com algumas propriedades:

* `type`: Igual a `element.type`.
* `props`: O resultado da mesclagem rasa de `element.props` com as `props` que você passou para substituir.
* `ref`: O `element.ref` original, a menos que seja substituído por `props.ref`.
* `key`: O `element.key` original, a menos que seja substituído por `props.key`.

Normalmente, você retornará o elemento do seu componente ou o tornará um filho de outro elemento. Embora você possa ler as propriedades do elemento, é melhor tratar cada elemento como opaco depois de criado e apenas renderizá-lo.

#### Ressalvas {/*caveats*/}

* Clonar um elemento **não modifica o elemento original.**

* Você só deve **passar filhos como vários argumentos para `cloneElement` se eles forem todos estaticamente conhecidos**, como `cloneElement(element, null, child1, child2, child3)`. Se seus filhos forem dinâmicos, passe o array inteiro como o terceiro argumento: `cloneElement(element, null, listItems)`. Isso garante que o React [avise você sobre a falta de `key`s](/learn/rendering-lists#keeping-list-items-in-order-with-key) para quaisquer listas dinâmicas. Para listas estáticas isso não é necessário porque elas nunca reordenam.

* `cloneElement` dificulta o rastreamento do fluxo de dados, então **tente as [alternativas](#alternatives) em vez disso.**

---

## Uso {/*usage*/}

### Substituindo props de um elemento {/*overriding-props-of-an-element*/}

Para substituir as props de algum <CodeStep step={1}>Elemento React</CodeStep>, passe-o para `cloneElement` com as <CodeStep step={2}>props que você deseja substituir</CodeStep>:

```js [[1, 5, "<Row title=\\"Cabbage\\" />"], [2, 6, "{ isHighlighted: true }"], [3, 4, "clonedElement"]]
import { cloneElement } from 'react';

// ...
const clonedElement = cloneElement(
  <Row title="Cabbage" />,
  { isHighlighted: true }
);
```

Aqui, o <CodeStep step={3}>elemento clonado</CodeStep> resultante será `<Row title="Cabbage" isHighlighted={true} />`.

**Vamos analisar um exemplo para ver quando isso é útil.**

Imagine um componente `List` que renderiza seus [`children`](/learn/passing-props-to-a-component#passing-jsx-as-children) como uma lista de linhas selecionáveis com um botão "Next" que muda qual linha é selecionada. O componente `List` precisa renderizar a `Row` selecionada de forma diferente, então ele clona cada filho `<Row>` que recebeu e adiciona uma prop extra `isHighlighted: true` ou `isHighlighted: false`:

```js {6-8}
export default function List({ children }) {
  const [selectedIndex, setSelectedIndex] = useState(0);
  return (
    <div className="List">
      {Children.map(children, (child, index) =>
        cloneElement(child, {
          isHighlighted: index === selectedIndex 
        })
      )}
```

Digamos que o JSX original recebido por `List` se parece com isto:

```js {2-4}
<List>
  <Row title="Cabbage" />
  <Row title="Garlic" />
  <Row title="Apple" />
</List>
```

Ao clonar seus filhos, o `List` pode passar informações extras para cada `Row` dentro. O resultado se parece com isto:

```js {4,8,12}
<List>
  <Row
    title="Cabbage"
    isHighlighted={true} 
  />
  <Row
    title="Garlic"
    isHighlighted={false} 
  />
  <Row
    title="Apple"
    isHighlighted={false} 
  />
</List>
```

Observe como pressionar "Next" atualiza o estado da `List` e destaca uma linha diferente:

<Sandpack>

```js
import List from './List.js';
import Row from './Row.js';
import { products } from './data.js';

export default function App() {
  return (
    <List>
      {products.map(product =>
        <Row
          key={product.id}
          title={product.title} 
        />
      )}
    </List>
  );
}
```

```js src/List.js active
import { Children, cloneElement, useState } from 'react';

export default function List({ children }) {
  const [selectedIndex, setSelectedIndex] = useState(0);
  return (
    <div className="List">
      {Children.map(children, (child, index) =>
        cloneElement(child, {
          isHighlighted: index === selectedIndex 
        })
      )}
      <hr />
      <button onClick={() => {
        setSelectedIndex(i =>
          (i + 1) % Children.count(children)
        );
      }}>
        Next
      </button>
    </div>
  );
}
```

```js src/Row.js
export default function Row({ title, isHighlighted }) {
  return (
    <div className={[
      'Row',
      isHighlighted ? 'RowHighlighted' : ''
    ].join(' ')}>
      {title}
    </div>
  );
}
```

```js src/data.js
export const products = [
  { title: 'Cabbage', id: 1 },
  { title: 'Garlic', id: 2 },
  { title: 'Apple', id: 3 },
];
```

```css
.List {
  display: flex;
  flex-direction: column;
  border: 2px solid grey;
  padding: 5px;
}

.Row {
  border: 2px dashed black;
  padding: 5px;
  margin: 5px;
}

.RowHighlighted {
  background: #ffa;
}

button {
  height: 40px;
  font-size: 20px;
}
```

</Sandpack>

Para resumir, o `List` clonou os elementos `<Row />` que recebeu e adicionou uma prop extra a eles.

<Pitfall>

Clonar filhos dificulta dizer como os dados fluem através de seu app. Tente uma das [alternativas.](#alternatives)

</Pitfall>

---

## Alternativas {/*alternatives*/}

### Passando dados com uma render prop {/*passing-data-with-a-render-prop*/}

Em vez de usar `cloneElement`, considere aceitar uma *render prop* como `renderItem`. Aqui, `List` recebe `renderItem` como uma prop. `List` chama `renderItem` para cada item e passa `isHighlighted` como um argumento:

```js {1,7}
export default function List({ items, renderItem }) {
  const [selectedIndex, setSelectedIndex] = useState(0);
  return (
    <div className="List">
      {items.map((item, index) => {
        const isHighlighted = index === selectedIndex;
        return renderItem(item, isHighlighted);
      })}
```

A prop `renderItem` é chamada de "render prop" porque é uma prop que especifica como renderizar algo. Por exemplo, você pode passar uma implementação `renderItem` que renderiza um `<Row>` com o valor `isHighlighted` fornecido:

```js {3,7}
<List
  items={products}
  renderItem={(product, isHighlighted) =>
    <Row
      key={product.id}
      title={product.title}
      isHighlighted={isHighlighted}
    />
  }
/>
```

O resultado final é o mesmo de `cloneElement`:

```js {4,8,12}
<List>
  <Row
    title="Cabbage"
    isHighlighted={true} 
  />
  <Row
    title="Garlic"
    isHighlighted={false} 
  />
  <Row
    title="Apple"
    isHighlighted={false} 
  />
</List>
```

No entanto, você pode rastrear claramente de onde vem o valor de `isHighlighted`.

<Sandpack>

```js
import List from './List.js';
import Row from './Row.js';
import { products } from './data.js';

export default function App() {
  return (
    <List
      items={products}
      renderItem={(product, isHighlighted) =>
        <Row
          key={product.id}
          title={product.title}
          isHighlighted={isHighlighted}
        />
      }
    />
  );
}
```

```js src/List.js active
import { useState } from 'react';

export default function List({ items, renderItem }) {
  const [selectedIndex, setSelectedIndex] = useState(0);
  return (
    <div className="List">
      {items.map((item, index) => {
        const isHighlighted = index === selectedIndex;
        return renderItem(item, isHighlighted);
      })}
      <hr />
      <button onClick={() => {
        setSelectedIndex(i =>
          (i + 1) % items.length
        );
      }}>
        Next
      </button>
    </div>
  );
}
```

```js src/Row.js
export default function Row({ title, isHighlighted }) {
  return (
    <div className={[
      'Row',
      isHighlighted ? 'RowHighlighted' : ''
    ].join(' ')}>
      {title}
    </div>
  );
}
```

```js src/data.js
export const products = [
  { title: 'Cabbage', id: 1 },
  { title: 'Garlic', id: 2 },
  { title: 'Apple', id: 3 },
];
```

```css
.List {
  display: flex;
  flex-direction: column;
  border: 2px solid grey;
  padding: 5px;
}

.Row {
  border: 2px dashed black;
  padding: 5px;
  margin: 5px;
}

.RowHighlighted {
  background: #ffa;
}

button {
  height: 40px;
  font-size: 20px;
}
```

</Sandpack>

Este padrão é preferível a `cloneElement` porque é mais explícito.

---

### Passando dados por meio de contexto {/*passing-data-through-context*/}

Outra alternativa para `cloneElement` é [passar dados por meio de contexto.](/learn/passing-data-deeply-with-context)

Por exemplo, você pode chamar [`createContext`](/reference/react/createContext) para definir um `HighlightContext`:

```js
export const HighlightContext = createContext(false);
```

Seu componente `List` pode encapsular cada item que ele renderiza em um provedor `HighlightContext`:

```js {8,10}
export default function List({ items, renderItem }) {
  const [selectedIndex, setSelectedIndex] = useState(0);
  return (
    <div className="List">
      {items.map((item, index) => {
        const isHighlighted = index === selectedIndex;
        return (
          <HighlightContext.Provider key={item.id} value={isHighlighted}>
            {renderItem(item)}
          </HighlightContext.Provider>
        );
      })}
```

Com esta abordagem, `Row` não precisa receber uma prop `isHighlighted` de forma alguma. Em vez disso, ele lê o contexto:

```js src/Row.js {2}
export default function Row({ title }) {
  const isHighlighted = useContext(HighlightContext);
  // ...
```

Isso permite que o componente de chamada não saiba ou se preocupe em passar `isHighlighted` para `<Row>`:

```js {4}
<List
  items={products}
  renderItem={product =>
    <Row title={product.title} />
  }
/>
```

Em vez disso, `List` e `Row` coordenam a lógica de destaque por meio do contexto.

<Sandpack>

```js
import List from './List.js';
import Row from './Row.js';
import { products } from './data.js';

export default function App() {
  return (
    <List
      items={products}
      renderItem={(product) =>
        <Row title={product.title} />
      }
    />
  );
}
```

```js src/List.js active
import { useState } from 'react';
import { HighlightContext } from './HighlightContext.js';

export default function List({ items, renderItem }) {
  const [selectedIndex, setSelectedIndex] = useState(0);
  return (
    <div className="List">
      {items.map((item, index) => {
        const isHighlighted = index === selectedIndex;
        return (
          <HighlightContext.Provider
            key={item.id}
            value={isHighlighted}
          >
            {renderItem(item)}
          </HighlightContext.Provider>
        );
      })}
      <hr />
      <button onClick={() => {
        setSelectedIndex(i =>
          (i + 1) % items.length
        );
      }}>
        Next
      </button>
    </div>
  );
}
```

```js src/Row.js
import { useContext } from 'react';
import { HighlightContext } from './HighlightContext.js';

export default function Row({ title }) {
  const isHighlighted = useContext(HighlightContext);
  return (
    <div className={[
      'Row',
      isHighlighted ? 'RowHighlighted' : ''
    ].join(' ')}>
      {title}
    </div>
  );
}
```

```js src/HighlightContext.js
import { createContext } from 'react';

export const HighlightContext = createContext(false);
```

```js src/data.js
export const products = [
  { title: 'Cabbage', id: 1 },
  { title: 'Garlic', id: 2 },
  { title: 'Apple', id: 3 },
];
```

```css
.List {
  display: flex;
  flex-direction: column;
  border: 2px solid grey;
  padding: 5px;
}

.Row {
  border: 2px dashed black;
  padding: 5px;
  margin: 5px;
}

.RowHighlighted {
  background: #ffa;
}

button {
  height: 40px;
  font-size: 20px;
}
```

</Sandpack>

[Saiba mais sobre como passar dados por meio de contexto.](/reference/react/useContext#passing-data-deeply-into-the-tree)

---

### Extraindo a lógica em um Hook personalizado {/*extracting-logic-into-a-custom-hook*/}

Outra abordagem que você pode tentar é extrair a lógica "não visual" em seu próprio Hook e usar as informações retornadas pelo seu Hook para decidir o que renderizar. Por exemplo, você pode escrever um Hook personalizado `useList` assim:

```js
import { useState } from 'react';

export default function useList(items) {
  const [selectedIndex, setSelectedIndex] = useState(0);

  function onNext() {
    setSelectedIndex(i =>
      (i + 1) % items.length
    );
  }

  const selected = items[selectedIndex];
  return [selected, onNext];
}
```

Então você pode usá-lo assim:

```js {2,9,13}
export default function App() {
  const [selected, onNext] = useList(products);
  return (
    <div className="List">
      {products.map(product =>
        <Row
          key={product.id}
          title={product.title}
          isHighlighted={selected === product}
        />
      )}
      <hr />
      <button onClick={onNext}>
        Next
      </button>
    </div>
  );
}
```

O fluxo de dados é explícito, mas o estado está dentro do Hook personalizado `useList` que você pode usar de qualquer componente:

<Sandpack>

```js
import Row from './Row.js';
import useList from './useList.js';
import { products } from './data.js';

export default function App() {
  const [selected, onNext] = useList(products);
  return (
    <div className="List">
      {products.map(product =>
        <Row
          key={product.id}
          title={product.title}
          isHighlighted={selected === product}
        />
      )}
      <hr />
      <button onClick={onNext}>
        Next
      </button>
    </div>
  );
}
```

```js src/useList.js
import { useState } from 'react';

export default function useList(items) {
  const [selectedIndex, setSelectedIndex] = useState(0);

  function onNext() {
    setSelectedIndex(i =>
      (i + 1) % items.length
    );
  }

  const selected = items[selectedIndex];
  return [selected, onNext];
}
```

```js src/Row.js
export default function Row({ title, isHighlighted }) {
  return (
    <div className={[
      'Row',
      isHighlighted ? 'RowHighlighted' : ''
    ].join(' ')}>
      {title}
    </div>
  );
}
```

```js src/data.js
export const products = [
  { title: 'Cabbage', id: 1 },
  { title: 'Garlic', id: 2 },
  { title: 'Apple', id: 3 },
];
```

```css
.List {
  display: flex;
  flex-direction: column;
  border: 2px solid grey;
  padding: 5px;
}

.Row {
  border: 2px dashed black;
  padding: 5px;
  margin: 5px;
}

.RowHighlighted {
  background: #ffa;
}

button {
  height: 40px;
  font-size: 20px;
}
```

</Sandpack>

Esta abordagem é particularmente útil se você deseja reutilizar esta lógica entre diferentes componentes.
```