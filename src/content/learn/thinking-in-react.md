---
title: Pensando em React
---

<Intro>

O React pode mudar a forma como você pensa sobre os designs que você vê e os apps que você constrói. Quando você constrói uma interface de usuário com o React, você primeiro irá dividi-la em pedaços chamados de *componentes*. Então, você descreverá os diferentes estados visuais para cada um de seus componentes. Finalmente, você conectará seus componentes para que os dados fluam por eles. Neste tutorial, guiaremos você pelo processo de pensamento de construção de uma tabela de dados de produtos pesquisável com o React.

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

Para implementar uma UI em React, você normalmente seguirá os mesmos cinco passos.

## Passo 1: Divida a UI em uma hierarquia de componentes {/*step-1-break-the-ui-into-a-component-hierarchy*/}

Comece desenhando caixas em torno de cada componente e subcomponente no mockup e nomeando-os. Se você trabalhar com um designer, ele pode já ter nomeado esses componentes em sua ferramenta de design. Pergunte a ele!

Dependendo da sua formação, você pode pensar em dividir um design em componentes de diferentes maneiras:

*   **Programação** -- use as mesmas técnicas para decidir se você deve criar uma nova função ou objeto. Uma dessas técnicas é o [princípio da responsabilidade única](https://en.wikipedia.org/wiki/Single_responsibility_principle), ou seja, um componente idealmente deve fazer apenas uma coisa. Se ele acabar crescendo, ele deve ser decomposto em subcomponentes menores.
*   **CSS** -- considere para o que você criaria seletores de classe. (No entanto, os componentes são um pouco menos granulares.)
*   **Design** -- considere como você organizaria as camadas do design.

Se o seu JSON for bem estruturado, você frequentemente descobrirá que ele mapeia naturalmente para a estrutura de componentes da sua UI. Isso ocorre porque a UI e os modelos de dados geralmente têm a mesma arquitetura de informação -- ou seja, a mesma forma. Separe a sua UI em componentes, onde cada componente corresponde a uma parte do seu modelo de dados.

Existem cinco componentes nesta tela:

<FullWidth>

<CodeDiagram flip>

<img src="/images/docs/s_thinking-in-react_ui_outline.png" width="500" style={{margin: '0 auto'}} />

1.  `FilterableProductTable` (cinza) contém todo o aplicativo.
2.  `SearchBar` (azul) recebe a entrada do usuário.
3.  `ProductTable` (lavanda) exibe e filtra a lista de acordo com a entrada do usuário.
4.  `ProductCategoryRow` (verde) exibe um cabeçalho para cada categoria.
5.  `ProductRow` (amarelo) exibe uma linha para cada produto.

</CodeDiagram>

</FullWidth>

Se você olhar para `ProductTable` (lavanda), você verá que o cabeçalho da tabela (contendo os rótulos "Name" e "Price") não é um componente próprio. Essa é uma questão de preferência, e você pode seguir qualquer caminho. Para este exemplo, ele faz parte do `ProductTable` porque aparece dentro da lista do `ProductTable`. No entanto, se este cabeçalho crescer para ser complexo (por exemplo, se você adicionar ordenação), você pode movê-lo para seu próprio componente `ProductTableHeader`.

Agora que você identificou os componentes no mockup, organize-os em uma hierarquia. Os componentes que aparecem dentro de outro componente no mockup devem aparecer como um filho na hierarquia:

*   `FilterableProductTable`
    *   `SearchBar`
    *   `ProductTable`
        *   `ProductCategoryRow`
        *   `ProductRow`

## Passo 2: Crie uma versão estática no React {/*step-2-build-a-static-version-in-react*/}

Agora que você tem a sua hierarquia de componentes, é hora de implementar o seu app. A abordagem mais direta é criar uma versão que renderiza a UI a partir do seu modelo de dados sem adicionar nenhuma interatividade... ainda! Muitas vezes, é mais fácil construir a versão estática primeiro e adicionar a interatividade mais tarde. Construir uma versão estática requer muita digitação e nenhum pensamento, mas adicionar interatividade requer muito pensamento e pouca digitação.

Para construir uma versão estática do seu app que renderiza o seu modelo de dados, você vai querer construir [componentes](/learn/your-first-component) que reutilizam outros componentes e passam dados usando [props.](/learn/passing-props-to-a-component) As props são uma maneira de passar dados do pai para o filho. (Se você está familiarizado com o conceito de [state](/learn/state-a-components-memory), não use state para construir esta versão estática. State é reservado apenas para interatividade, ou seja, dados que mudam com o tempo. Como esta é uma versão estática do app, você não precisa dela.)

Você pode construir "de cima para baixo", começando com a construção dos componentes mais acima na hierarquia (como `FilterableProductTable`) ou "de baixo para cima" trabalhando a partir de componentes mais abaixo (como `ProductRow`). Em exemplos mais simples, geralmente é mais fácil ir de cima para baixo, e em projetos maiores, é mais fácil ir de baixo para cima.

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

(Se este código parece intimidador, passe pelo [Início Rápido](/learn/) primeiro!)

Depois de construir seus componentes, você terá uma biblioteca de componentes reutilizáveis que renderizam seu modelo de dados. Como este é um app estático, os componentes retornarão apenas JSX. O componente no topo da hierarquia (`FilterableProductTable`) usará seu modelo de dados como uma prop. Isso é chamado de _fluxo de dados unidirecional_ porque os dados fluem de cima para baixo, do componente de nível superior para os da parte inferior da árvore.

<Pitfall>

Neste ponto, você não deve usar nenhum valor de state. Isso é para o próximo passo!

</Pitfall>

## Passo 3: Encontre a representação mínima, mas completa, do state da UI {/*step-3-find-the-minimal-but-complete-representation-of-ui-state*/}

Para tornar a UI interativa, você precisa permitir que os usuários alterem o seu modelo de dados subjacente. Você usará o *state* para isso.

Pense no state como o conjunto mínimo de dados em mudança que seu aplicativo precisa lembrar. O princípio mais importante para estruturar o state é mantê-lo [DRY (Não se repita).](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) Descubra a representação mínima absoluta do state que seu aplicativo precisa e calcule todo o resto sob demanda. Por exemplo, se você estiver construindo uma lista de compras, poderá armazenar os itens como um array no state. Se você também quiser exibir o número de itens na lista, não armazene o número de itens como outro valor de state -- em vez disso, leia o comprimento do seu array.

Agora pense em todas as partes dos dados neste aplicativo de exemplo:

1.  A lista original de produtos
2.  O texto de pesquisa que o usuário inseriu
3.  O valor da caixa de seleção
4.  A lista filtrada de produtos

Quais dessas são state? Identifique aquelas que não são:

*   Ele **permanece inalterado** ao longo do tempo? Se sim, não é state.
*   Ele é **passado de um pai** via props? Se sim, não é state.
*   **Você pode calculá-lo** com base no state ou props existentes em seu componente? Se sim, *definitivamente* não é state!

O que sobra é provavelmente o state.

Vamos analisá-los um por um novamente:

1.  A lista original de produtos é **passada como props, então não é state.**
2.  O texto de pesquisa parece ser state, pois muda ao longo do tempo e não pode ser calculado a partir de nada.
3.  O valor da caixa de seleção parece ser state, pois muda ao longo do tempo e não pode ser calculado a partir de nada.
4.  A lista filtrada de produtos **não é state, pois pode ser calculada** pegando a lista original de produtos e filtrando-a de acordo com o texto de pesquisa e o valor da caixa de seleção.

Isso significa que apenas o texto de pesquisa e o valor da caixa de seleção são state! Bom trabalho!

<DeepDive>

#### Props vs State {/*props-vs-state*/}

Existem dois tipos de dados "modelo" no React: props e state. Os dois são muito diferentes:

*   [**Props** são como argumentos que você passa](/learn/passing-props-to-a-component) para uma função. Eles permitem que um componente pai passe dados para um componente filho e personalize sua aparência. Por exemplo, um `Form` pode passar uma prop `color` para um `Button`.
*   [**State** é como a memória de um componente.](/learn/state-a-components-memory) Ele permite que um componente mantenha o controle de algumas informações e as altere em resposta a interações. Por exemplo, um `Button` pode manter o controle do state `isHovered`.

Props e state são diferentes, mas eles trabalham juntos. Um componente pai costuma manter algumas informações no state (para que possa alterá-las) e *passá-las* para os componentes filhos como suas props. É ok se a diferença ainda parecer confusa na primeira leitura. Leva um pouco de prática para realmente fixar!

</DeepDive>

## Passo 4: Identifique onde seu state deve ficar {/*step-4-identify-where-your-state-should-live*/}

Depois de identificar os dados mínimos de state do seu app, você precisa identificar qual componente é responsável por alterar esse state, ou *possui* o state. Lembre-se: o React usa o fluxo de dados unidirecional, passando os dados pela hierarquia de componentes do componente pai para o componente filho. Pode não ser imediatamente claro qual componente deve possuir qual state. Isso pode ser desafiador se você é novo nesse conceito, mas você pode descobrir isso seguindo estas etapas!

Para cada parte do state no seu aplicativo:

1.  Identifique *todos* os componentes que renderizam algo com base nesse state.
2.  Encontre o componente pai comum mais próximo deles -- um componente acima de todos eles na hierarquia.
3.  Decida onde o state deve ficar:
    1.  Frequentemente, você pode colocar o state diretamente em seu pai comum.
    2.  Você também pode colocar o state em algum componente acima do seu pai comum.
    3.  Se você não conseguir encontrar um componente onde faça sentido possuir o state, crie um novo componente apenas para armazenar o state e adicione-o em algum lugar na hierarquia acima do componente pai comum.

No passo anterior, você encontrou duas partes de state neste aplicativo: o texto de entrada de pesquisa e o valor da caixa de seleção. Neste exemplo, eles sempre aparecem juntos, então faz sentido colocá-los no mesmo lugar.

Agora vamos executar nossa estratégia para eles:

1.  **Identifique os componentes que usam state:**
    *   `ProductTable` precisa filtrar a lista de produtos com base nesse state (texto de pesquisa e valor da caixa de seleção).
    *   `SearchBar` precisa exibir esse state (texto de pesquisa e valor da caixa de seleção).
2.  **Encontre seu pai comum:** O primeiro componente pai que ambos os componentes compartilham é `FilterableProductTable`.
3.  **Decida onde o state fica**: Manteremos o texto do filtro e os valores de state marcados em `FilterableProductTable`.

Então os valores de state ficarão em `FilterableProductTable`.

Adicione o state ao componente com o [`useState()` Hook.](/reference/react/useState) Hooks são funções especiais que permitem que você "se conecte" ao React. Adicione duas variáveis de state no topo de `FilterableProductTable` e especifique seu state inicial:

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

Você pode começar a ver como seu aplicativo se comportará. Edite o valor inicial de `filterText` de `useState('')` para `useState('fruit')` no código sandbox abaixo. Você verá o texto de entrada de pesquisa e a tabela serem atualizados:

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

Observe que a edição do formulário ainda não funciona. Há um erro de console no sandbox acima explicando o porquê:

<ConsoleBlock level="error">

Você forneceu uma prop \`value\` para um campo de formulário sem um manipulador \`onChange\`. Isso renderizará um campo somente leitura.

</ConsoleBlock>

No sandbox acima, `ProductTable` e `SearchBar` lêem as props `filterText` e `inStockOnly` para renderizar a tabela, a entrada e a caixa de seleção. Por exemplo, aqui está como `SearchBar` preenche o valor de entrada:

```js {1,6}
function SearchBar({ filterText, inStockOnly }) {
  return (
    <form>
      <input 
        type="text" 
        value={filterText} 
        placeholder="Search..."/>
```

No entanto, você ainda não adicionou nenhum código para responder às ações do usuário, como digitar. Esta será sua etapa final.

## Passo 5: Adicione o fluxo de dados inverso {/*step-5-add-inverse-data-flow*/}

Atualmente, seu app renderiza corretamente com as props e o state fluindo pela hierarquia. Mas para alterar o state de acordo com a entrada do usuário, você precisará oferecer suporte aos dados fluindo no outro sentido: os componentes do formulário no fundo da hierarquia precisam atualizar o state em `FilterableProductTable`.

O React torna esse fluxo de dados explícito, mas requer um pouco mais de digitação do que a ligação de dados bidirecional. Se você tentar digitar ou marcar a caixa no exemplo acima, você verá que o React ignora sua entrada. Isso é intencional. Ao escrever `<input value={filterText} />`, você definiu a prop `value` do `input` para sempre ser igual ao state `filterText` passado de `FilterableProductTable`. Como o state `filterText` nunca é definido, a entrada nunca muda.

Você quer fazer com que, sempre que o usuário alterar as entradas do formulário, o state seja atualizado para refletir essas alterações. O state pertence a `FilterableProductTable`, então somente ele pode chamar `setFilterText` e `setInStockOnly`. Para permitir que `SearchBar` atualize o state de `FilterableProductTable`, você precisa passar essas funções para `SearchBar`:

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

Dentro do `SearchBar`, você adicionará os manipuladores de eventos `onChange` e definirá o state pai a partir deles:

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

Agora o aplicativo funciona totalmente!

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

Você pode aprender tudo sobre como lidar com eventos e atualizar o state na seção [Adicionando Interatividade](/learn/adding-interactivity).

## Para onde ir a partir daqui {/*where-to-go-from-here*/}

Esta foi uma breve introdução sobre como pensar em construir componentes e aplicativos com o React. Você pode [iniciar um projeto React](/learn/installation) agora ou [mergulhar mais fundo em toda a sintaxe](/learn/describing-the-ui) usada neste tutorial.
```