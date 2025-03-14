---
title: Pensando em React
---

<Intro>

React pode mudar a forma como você pensa sobre os designs que você vê e os apps que você constroi. Quando você constrói uma interface de usuário com o React, você primeiro irá dividí-la em pedaços chamados de *componentes*. Então, você irá descrever os diferentes estados visuais para cada um dos seus componentes. Finalmente, você vai conectar seus componentes juntos para que os dados fluam através deles. Neste tutorial, vamos guiá-lo através do processo de pensamento de como construir uma tabela de dados de produto pesquisável com o React.

</Intro>

## Comece com o mockup {/*start-with-the-mockup*/}

Imagine que você já tem uma API JSON e um mockup de um designer.

A API JSON retorna alguns dados que se parecem com isto:

```json
[
  { category: "Fruits", price: "$1", stocked: true, name: "Apple" },
  { category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit" },
  { category: "Fruits", price: "$2", stocked: false, name: "Passionfruit" },
  { category: "Vegetables", price: "$2", stocked: true, name: "Spinach" },
  { category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin" },
  { category: "Vegetables", price: "$1", stocked: true, name: "Peas" }
]
```

O mockup se parece com isto:

<img src="/images/docs/s_thinking-in-react_ui.png" width="300" style={{margin: '0 auto'}} />

Para implementar uma UI no React, você geralmente vai seguir os mesmos cinco passos.

## Passo 1: Divida a UI em uma hierarquia de componentes {/*step-1-break-the-ui-into-a-component-hierarchy*/}

Comece desenhando caixas em torno de cada componente e subcomponente no mockup e dando nome a eles. Se você trabalha com um designer, ele já pode ter nomeado esses componentes em sua ferramenta de design. Pergunte a eles!

Dependendo da sua experiência, você pode pensar em dividir um design em componentes de diferentes maneiras:

*   **Programação** -- use as mesmas técnicas para decidir se você deve criar uma nova função ou objeto. Uma dessas técnicas é o [princípio da responsabilidade única](https://pt.wikipedia.org/wiki/Princ%C3%ADpio_da_responsabilidade_%C3%BAnica), isto é, um componente idealmente deve fazer apenas uma coisa. Se ele acabar crescendo, ele deve ser decomposto em subcomponentes menores.
*   **CSS** -- considere para o que você criaria seletores de classe. (No entanto, componentes são um pouco menos granulares.)
*   **Design** -- considere como você organizaria as camadas do design.

Se seu JSON estiver bem estruturado, você vai frequentemente descobrir que ele mapeia naturalmente para a estrutura de componentes da sua UI. Isso acontece porque as UIs e os modelos de dados geralmente têm a mesma arquitetura de informação -- isto é, a mesma forma. Separe sua UI em componentes, onde cada componente corresponde a uma parte do seu modelo de dados.

Há cinco componentes nesta tela:

<FullWidth>

<CodeDiagram flip>

<img src="/images/docs/s_thinking-in-react_ui_outline.png" width="500" style={{margin: '0 auto'}} />

1.  `FilterableProductTable` (cinza) contém o app inteiro.
2.  `SearchBar` (azul) recebe a entrada do usuário.
3.  `ProductTable` (lavanda) exibe e filtra a lista de acordo com a entrada do usuário.
4.  `ProductCategoryRow` (verde) exibe um título para cada categoria.
5.  `ProductRow` (amarelo) exibe uma linha para cada produto.

</CodeDiagram>

</FullWidth>

Se você olhar para `ProductTable` (lavanda), você vai ver que o cabeçalho da tabela (contendo os rótulos "Name" e "Price") não é um componente próprio. Isso é uma questão de preferência, e você pode ir de qualquer maneira. Para este exemplo, ele faz parte do `ProductTable` porque ele aparece dentro da lista do `ProductTable`. No entanto, se este cabeçalho crescer para ser complexo (por exemplo, se você adicionar ordenação), você pode movê-lo para seu próprio componente `ProductTableHeader`.

Agora que você identificou os componentes no mockup, organize-os em uma hierarquia. Componentes que aparecem dentro de outro componente no mockup devem aparecer como um filho na hierarquia:

*   `FilterableProductTable`
    *   `SearchBar`
    *   `ProductTable`
        *   `ProductCategoryRow`
        *   `ProductRow`

## Passo 2: Construa uma versão estática no React {/*step-2-build-a-static-version-in-react*/}

Agora que você tem a hierarquia de componentes, é hora de implementar seu app. A abordagem mais direta é construir uma versão que renderize a UI do seu modelo de dados sem adicionar qualquer interatividade... ainda! É frequentemente mais fácil construir a versão estática primeiro e adicionar interatividade depois. Construir uma versão estática requer muita digitação e pouco pensamento, enquanto adicionar interatividade requer muito pensamento e pouca digitação.

Para construir uma versão estática do seu app que renderize seu modelo de dados, você vai querer construir [componentes](/learn/your-first-component) que reutilizem outros componentes e passem dados utilizando [props.](/learn/passing-props-to-a-component) Props são uma forma de passar dados do pai para o filho. (Se você estiver familiarizado com o conceito de [state](/learn/state-a-components-memory), não use o state para construir esta versão estática. State é reservado apenas para interatividade, isto é, dados que mudam com o tempo. Como esta é uma versão estática do app, você não precisa dele.)

Você pode construir "de cima para baixo" começando com a construção dos componentes mais acima na hierarquia (como `FilterableProductTable`) ou "de baixo para cima" trabalhando com componentes mais abaixo (como `ProductRow`). Em exemplos mais simples, é normalmente mais fácil ir de cima para baixo, e em projetos maiores, é mais fácil ir de baixo para cima.

<Sandpack>

```jsx src/App.js
function ProductCategoryRow({ category }) {
  return (
    <tr>
      <th colSpan="2">
        {category}
      </th>
    </tr>
  );
}

function ProductRow({ product }) {
  const name = product.stocked ? product.name :
    <span style={{ color: 'red' }}>
      {product.name}
    </span>;

  return (
    <tr>
      <td>{name}</td>
      <td>{product.price}</td>
    </tr>
  );
}

function ProductTable({ products }) {
  const rows = [];
  let lastCategory = null;

  products.forEach((product) => {
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category} />
      );
    }
    rows.push(
      <ProductRow
        product={product}
        key={product.name} />
    );
    lastCategory = product.category;
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Price</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}

function SearchBar() {
  return (
    <form>
      <input type="text" placeholder="Search..." />
      <label>
        <input type="checkbox" />
        {' '}
        Only show products in stock
      </label>
    </form>
  );
}

function FilterableProductTable({ products }) {
  return (
    <div>
      <SearchBar />
      <ProductTable products={products} />
    </div>
  );
}

const PRODUCTS = [
  {category: "Fruits", price: "$1", stocked: true, name: "Apple"},
  {category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit"},
  {category: "Fruits", price: "$2", stocked: false, name: "Passionfruit"},
  {category: "Vegetables", price: "$2", stocked: true, name: "Spinach"},
  {category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin"},
  {category: "Vegetables", price: "$1", stocked: true, name: "Peas"}
];

export default function App() {
  return <FilterableProductTable products={PRODUCTS} />;
}
```

```css
body {
  padding: 5px
}
label {
  display: block;
  margin-top: 5px;
  margin-bottom: 5px;
}
th {
  padding-top: 10px;
}
td {
  padding: 2px;
  padding-right: 40px;
}
```

</Sandpack>

(Se este código parecer intimidador, passe pelo [Início Rápido](/learn/) primeiro!)

Depois de construir seus componentes, você terá uma biblioteca de componentes reutilizáveis que renderizam seu modelo de dados. Como este é um app estático, os componentes só vão retornar JSX. O componente no topo da hierarquia (`FilterableProductTable`) vai pegar seu modelo de dados como uma prop. Isto é chamado de _fluxo de dados unidirecional_ porque os dados fluem de cima para baixo no componente de nível superior até os da parte inferior da árvore.

<Pitfall>

Neste ponto, você não deve estar usando nenhum valor de state. Isso é para o próximo passo!

</Pitfall>

## Passo 3: Encontre a representação mínima, mas completa, de UI state {/*step-3-find-the-minimal-but-complete-representation-of-ui-state*/}

Para tornar a UI interativa, você precisa deixar os usuários mudarem seu modelo de dados subjacente. Você pode usar um *state* para isso.

Pense no state como o conjunto mínimo de dados mutáveis que seu app precisa lembrar. O princípio mais importante para estruturar o state é mantê-lo [DRY (Don't Repeat Yourself).](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself). Descubra a representação absolutamente mínima do state que seu aplicativo precisa e calcule todo o resto sob demanda. Por exemplo, se você está construindo uma lista de compras, você pode armazenar os itens como um array no state. Se você também quiser exibir o número de itens na lista, não armazene o número de itens como outro valor de state -- em vez disso, leia o comprimento do seu array.

Agora pense em todas as peças de dados neste aplicativo de exemplo:

1.  A lista original de produtos
2.  O texto de pesquisa que o usuário inseriu
3.  O valor da caixa de seleção
4.  A lista filtrada de produtos

Quais destes são state? Identifique aqueles que não são:

*   Ele **permanece inalterado** ao longo do tempo? Se sim, não é state.
*   Ele é **passado de um pai** via props? Se sim, não é state.
*   **Você pode computá-lo** baseado no state ou props existentes no seu componente? Se sim, não é *definitivamente* state!

O que sobrou é provavelmente state.

Vamos passar por eles um a um novamente:

1.  A lista original de produtos é **passada como props, então não é state.**
2.  O texto de pesquisa parece ser state, pois muda ao longo do tempo e não pode ser calculado a partir de nada.
3.  O valor da caixa de seleção parece ser state, pois muda ao longo do tempo e não pode ser calculado a partir de nada.
4.  A lista filtrada de produtos **não é state porque pode ser calculada** pegando a lista original de produtos e filtrando-a de acordo com o texto de pesquisa e o valor da caixa de seleção.

Isso significa que apenas o texto de pesquisa e o valor da caixa de seleção são state! Ótimo trabalho!

<DeepDive>

#### Props vs State {/*props-vs-state*/}

Existem dois tipos de dados "modelo" no React: props e state. Os dois são muito diferentes:

*   [**Props** são como argumentos que você passa](/learn/passing-props-to-a-component) para uma função. Elas permitem que um componente pai passe dados para um componente filho e personalize sua aparênncia. Por exemplo, um `Form` pode passar uma prop `color` para um `Button`.
*   [**State** é como a memória de um componente.](/learn/state-a-components-memory) Ele permite que um componente mantenha o controle de algumas informações e as altere em resposta a interações. Por exemplo, um `Button` pode manter o controle do state de `isHovered`.

Props e state são diferentes, mas trabalham juntos. Um componente pai frequentemente manterá algumas informações no state (para que possa alterá-las) e *passá-las para baixo* para os componentes filhos como suas props. Tudo bem se a diferença ainda parecer confusa na primeira leitura. Leva um pouco de prática para que isso realmente funcione!

</DeepDive>

## Passo 4: Identifique onde seu state deve viver {/*step-4-identify-where-your-state-should-live*/}

Depois de identificar os dados mínimos de state do seu app, você precisa identificar qual componente é responsável por mudar este state, ou *possui* o state. Lembre-se: O React usa o fluxo de dados unidirecional, passando dados pela hierarquia de componentes do componente pai para o componente filho. Pode não ser imediatamente claro qual componente deve possuir qual state. Isso pode ser desafiador, mas você pode descobrir seguindo esses passos!

Para cada parte do state no seu aplicativo:

1.  Identifique *todo* componente que renderiza algo baseado nesse state.
2.  Encontre seu componente pai comum mais próximo -- um componente acima deles todos na hierarquia.
3.  Decida onde o state deve viver:
    1.  Frequentemente, você pode colocar o state diretamente em seu pai comum.
    2.  Você também pode colocar o state em algum componente acima de seu pai comum.
    3.  Se você não conseguir encontrar um componente onde faça sentido possuir o state, crie um novo componente somente para manter o state e adicione-o em algum lugar na hierarquia acima do componente pai comum.

No passo anterior, você encontrou duas partes de state neste aplicativo: o texto de entrada de pesquisa, e o valor da caixa de seleção. Neste exemplo, elas sempre aparecem juntas, então faz sentido colocá-las no mesmo lugar.

Agora vamos rodar a nossa estratégia para elas:

1.  **Identifique os componentes que usam o state:**
    *   `ProductTable` precisa filtrar a lista de produtos com base naquele state (texto de pesquisa e valor da caixa de seleção).
    *   `SearchBar` precisa exibir esse state (texto de pesquisa e valor da caixa de seleção).
2.  **Encontre seu pai comum:** O primeiro componente pai que ambos os componentes compartilham é `FilterableProductTable`.
3.  **Decidir onde o state vive**: Vamos manter o texto do filtro e os valores de state verificados em `FilterableProductTable`.

Então os valores de state vão viver em `FilterableProductTable`.

Adicione state ao componente com o [Hook `useState()`.](/reference/react/useState) Hooks são funções especiais que permitem que você "se conecte" ao React. Adicione duas variáveis de state no topo do `FilterableProductTable` e especifique seu state inicial:

```js
function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);  
```

Então, passe `filterText` e `inStockOnly` para `ProductTable` e `SearchBar` como props:

```js
<div>
  <SearchBar 
    filterText={filterText} 
    inStockOnly={inStockOnly} />
  <ProductTable 
    products={products}
    filterText={filterText}
    inStockOnly={inStockOnly} />
</div>
```

Você pode começar a ver como seu aplicativo se comportará. Edite o valor inicial de `filterText` de `useState('')` para `useState('fruit')` no código sandbox abaixo. Você vai ver tanto o texto de entrada de pesquisa quanto a tabela atualizarem:

<Sandpack>

```jsx src/App.js
import { useState } from 'react';

function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar 
        filterText={filterText} 
        inStockOnly={inStockOnly} />
      <ProductTable 
        products={products}
        filterText={filterText}
        inStockOnly={inStockOnly} />
    </div>
  );
}

function ProductCategoryRow({ category }) {
  return (
    <tr>
      <th colSpan="2">
        {category}
      </th>
    </tr>
  );
}

function ProductRow({ product }) {
  const name = product.stocked ? product.name :
    <span style={{ color: 'red' }}>
      {product.name}
    </span>;

  return (
    <tr>
      <td>{name}</td>
      <td>{product.price}</td>
    </tr>
  );
}

function ProductTable({ products, filterText, inStockOnly }) {
  const rows = [];
  let lastCategory = null;

  products.forEach((product) => {
    if (
      product.name.toLowerCase().indexOf(
        filterText.toLowerCase()
      ) === -1
    ) {
      return;
    }
    if (inStockOnly && !product.stocked) {
      return;
    }
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category} />
      );
    }
    rows.push(
      <ProductRow
        product={product}
        key={product.name} />
    );
    lastCategory = product.category;
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Price</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}

function SearchBar({ filterText, inStockOnly }) {
  return (
    <form>
      <input 
        type="text" 
        value={filterText} 
        placeholder="Search..."/>
      <label>
        <input 
          type="checkbox" 
          checked={inStockOnly} />
        {' '}
        Only show products in stock
      </label>
    </form>
  );
}

const PRODUCTS = [
  {category: "Fruits", price: "$1", stocked: true, name: "Apple"},
  {category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit"},
  {category: "Fruits", price: "$2", stocked: false, name: "Passionfruit"},
  {category: "Vegetables", price: "$2", stocked: true, name: "Spinach"},
  {category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin"},
  {category: "Vegetables", price: "$1", stocked: true, name: "Peas"}
];

export default function App() {
  return <FilterableProductTable products={PRODUCTS} />;
}
```

```css
body {
  padding: 5px
}
label {
  display: block;
  margin-top: 5px;
  margin-bottom: 5px;
}
th {
  padding-top: 5px;
}
td {
  padding: 2px;
}
```

</Sandpack>

Observe que editar o formulário ainda não funciona. Há um erro de console no sandbox acima explicando o porquê:

<ConsoleBlock level="error">

You provided a \`value\` prop to a form field without an \`onChange\` handler. This will render a read-only field.

</ConsoleBlock>

No sandbox acima, `ProductTable` e `SearchBar` lêem as props `filterText` e `inStockOnly` para renderizar a tabela, a entrada e a caixa de seleção. Por exemplo, aqui está como a `SearchBar` preenche o valor de entrada:

```js {1,6}
function SearchBar({ filterText, inStockOnly }) {
  return (
    <form>
      <input 
        type="text" 
        value={filterText} 
        placeholder="Search..."/>
```

No entanto, você não adicionou nenhum código para responder às ações do usuário, como digitação, ainda. Este será seu passo final.

## Passo 5: Adicione fluxo de dados inverso {/*step-5-add-inverse-data-flow*/}

Atualmente seu app renderiza corretamente com as props e state fluindo pela hierarquia. Mas para alterar o state de acordo com a entrada do usuário, você precisará suportar dados fluindo no outro sentido: os componentes do formulário no fundo da hierarquia precisam atualizar o state em `FilterableProductTable`.

O React torna esse fluxo de dados explícito, mas requer um pouco mais de digitação que a ligação de dados em duas vias. Se você tentar digitar ou marcar a caixa no exemplo acima, você vai ver que o React ignora sua entrada. Isso é intencional. Ao escrever `<input value={filterText} />`, você definiu a prop `value` da `input` para sempre ser igual ao state `filterText` passado de `FilterableProductTable`. Como o state `filterText` nunca é definido, a entrada nunca muda.

Você quer fazer com que sempre que o usuário mudar as entradas do formulário, o state atualize para refletir essas mudanças. O state é propriedade do `FilterableProductTable`, então somente ele pode chamar `setFilterText` e `setInStockOnly`. Para deixar o `SearchBar` atualizar o state de `FilterableProductTable`, você precisa passar essas funções para `SearchBar`:

```js {2,3,10,11}
function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar 
        filterText={filterText} 
        inStockOnly={inStockOnly}
        onFilterTextChange={setFilterText}
        onInStockOnlyChange={setInStockOnly} />
```

Dentro do `SearchBar`, você vai adicionar os manipuladores de eventos `onChange` e definir o state pai a partir deles:

```js {4,5,13,19}
function SearchBar({
  filterText,
  inStockOnly,
  onFilterTextChange,
  onInStockOnlyChange
}) {
  return (
    <form>
      <input
        type="text"
        value={filterText}
        placeholder="Search..."
        onChange={(e) => onFilterTextChange(e.target.value)}
      />
      <label>
        <input
          type="checkbox"
          checked={inStockOnly}
          onChange={(e) => onInStockOnlyChange(e.target.checked)}
```

Agora o aplicativo funciona completamente!

<Sandpack>

```jsx src/App.js
import { useState } from 'react';

function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar 
        filterText={filterText} 
        inStockOnly={inStockOnly} 
        onFilterTextChange={setFilterText} 
        onInStockOnlyChange={setInStockOnly} />
      <ProductTable 
        products={products} 
        filterText={filterText}
        inStockOnly={inStockOnly} />
    </div>
  );
}

function ProductCategoryRow({ category }) {
  return (
    <tr>
      <th colSpan="2">
        {category}
      </th>
    </tr>
  );
}

function ProductRow({ product }) {
  const name = product.stocked ? product.name :
    <span style={{ color: 'red' }}>
      {product.name}
    </span>;

  return (
    <tr>
      <td>{name}</td>
      <td>{product.price}</td>
    </tr>
  );
}

function ProductTable({ products, filterText, inStockOnly }) {
  const rows = [];
  let lastCategory = null;

  products.forEach((product) => {
    if (
      product.name.toLowerCase().indexOf(
        filterText.toLowerCase()
      ) === -1
    ) {
      return;
    }
    if (inStockOnly && !product.stocked) {
      return;
    }
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category} />
      );
    }
    rows.push(
      <ProductRow
        product={product}
        key={product.name} />
    );
    lastCategory = product.category;
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Price</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}

function SearchBar({
  filterText,
  inStockOnly,
  onFilterTextChange,
  onInStockOnlyChange
}) {
  return (
    <form>
      <input 
        type="text" 
        value={filterText} placeholder="Search..." 
        onChange={(e) => onFilterTextChange(e.target.value)} />
      <label>
        <input 
          type="checkbox" 
          checked={inStockOnly} 
          onChange={(e) => onInStockOnlyChange(e.target.checked)} />
        {' '}
        Only show products in stock
      </label>
    </form>
  );
}

const PRODUCTS = [
  {category: "Fruits", price: "$1", stocked: true, name: "Apple"},
  {category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit"},
  {category: "Fruits", price: "$2", stocked: false, name: "Passionfruit"},
  {category: "Vegetables", price: "$2", stocked: true, name: "Spinach"},
  {category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin"},
  {category: "Vegetables", price: "$1", stocked: true, name: "Peas"}
];

export default function App() {
  return <FilterableProductTable products={PRODUCTS} />;
}
```

```css
body {
  padding: 5px
}
label {
  display: block;
  margin-top: 5px;
  margin-bottom: 5px;
}
th {
  padding: 4px;
}
td {
  padding: 2px;
}
```

</Sandpack>

Você pode aprender tudo sobre o tratamento de eventos e a atualização do state na sessão [Adicionando Interatividade](/learn/adding-interactivity).

## Para onde ir a partir daqui {/*where-to-go-from-here*/}

Esta foi uma introdução bem breve sobre como pensar em construir componentes e aplicativos com o React. Você pode [iniciar um projeto React](/learn/installation) agora ou [mergulhar mais fundo em toda a sintaxe](/learn/describing-the-ui) usada neste tutorial.