---
title: "Hooks do React embutidos"
---

<Intro>

*Hooks* permitem que você use diferentes recursos do React de seus componentes. Você pode usar os Hooks embutidos ou combiná-los para construir os seus próprios. Esta página lista todos os Hooks embutidos no React.

</Intro>

---

## State Hooks {/*state-hooks*/}

*State* permite que um componente ["lembre" informações como entrada do usuário.](/learn/state-a-components-memory) Por exemplo, um componente de formulário pode usar o state para armazenar o valor da entrada, enquanto um componente de galeria de imagens pode usar o state para armazenar o índice da imagem selecionada.

Para adicionar state a um componente, use um desses Hooks:

* [`useState`](/reference/react/useState) declara uma variável de state que você pode atualizar diretamente.
* [`useReducer`](/reference/react/useReducer) declara uma variável de state com a lógica de atualização dentro de uma [função reducer.](/learn/extracting-state-logic-into-a-reducer)

```js
function ImageGallery() {
  const [index, setIndex] = useState(0);
  // ...
```

---

## Context Hooks {/*context-hooks*/}

*Context* permite que um componente [receba informações de pais distantes sem passá-las como props.](/learn/passing-props-to-a-component) Por exemplo, o componente de nível superior do seu aplicativo pode passar o tema atual da UI para todos os componentes abaixo, não importa o quão profundos eles sejam.

* [`useContext`](/reference/react/useContext) lê e se inscreve em um context.

```js
function Button() {
  const theme = useContext(ThemeContext);
  // ...
```

---

## Ref Hooks {/*ref-hooks*/}

*Refs* permitem que um componente [mantenha algumas informações que não são usadas para renderização,](/learn/referencing-values-with-refs) como um nó DOM ou um ID de timeout. Diferente do state, atualizar uma ref não re-renderiza seu componente. Refs são uma "saída de emergência" do paradigma do React. Elas são úteis quando você precisa trabalhar com sistemas que não são React, como as APIs embutidas do navegador.

* [`useRef`](/reference/react/useRef) declara uma ref. Você pode armazenar qualquer valor nela, mas com mais frequência ela é usada para armazenar um nó DOM.
* [`useImperativeHandle`](/reference/react/useImperativeHandle) permite que você personalize a ref exposta pelo seu componente. Isso é raramente usado.

```js
function Form() {
  const inputRef = useRef(null);
  // ...
```

---

## Effect Hooks {/*effect-hooks*/}

*Effects* permitem que um componente [se conecte e sincronize com sistemas externos.](/learn/synchronizing-with-effects) Isso inclui lidar com rede, DOM do navegador, animações, widgets escritos usando uma biblioteca de UI diferente e outro código que não é React.

* [`useEffect`](/reference/react/useEffect) conecta um componente a um sistema externo.

```js
function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);
  // ...
```

Effects são uma "saída de emergência" do paradigma do React. Não use Effects para orquestrar o fluxo de dados do seu aplicativo. Se você não estiver interagindo com um sistema externo, [você pode não precisar de um Effect.](/learn/you-might-not-need-an-effect)

Existem duas variações raramente usadas de `useEffect` com diferenças no tempo:

* [`useLayoutEffect`](/reference/react/useLayoutEffect) dispara antes do navegador redesenhar a tela. Você pode medir o layout aqui.
* [`useInsertionEffect`](/reference/react/useInsertionEffect) dispara antes que o React faça alterações no DOM. As bibliotecas podem inserir CSS dinâmico aqui.

---

## Performance Hooks {/*performance-hooks*/}

Uma maneira comum de otimizar o desempenho de re-renderização é pular trabalhos desnecessários. Por exemplo, você pode dizer ao React para reutilizar um cálculo em cache ou para pular um re-render se os dados não mudaram desde a renderização anterior.

Para pular cálculos e re-renderização desnecessários, use um desses Hooks:

- [`useMemo`](/reference/react/useMemo) permite que você armazene em cache o resultado de um cálculo caro.
- [`useCallback`](/reference/react/useCallback) permite que você armazene em cache uma definição de função antes de passá-la para um componente otimizado.

```js
function TodoList({ todos, tab, theme }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  // ...
}
```

Às vezes, você não pode pular o re-render porque a tela realmente precisa atualizar. Nesse caso, você pode melhorar o desempenho separando as atualizações de bloqueio que devem ser síncronas (como digitar em uma entrada) das atualizações não bloqueantes que não precisam bloquear a interface do usuário (como atualizar um gráfico).

Para priorizar a renderização, use um desses Hooks:

- [`useTransition`](/reference/react/useTransition) permite que você marque uma transição de state como não bloqueante e permita que outras atualizações a interrompam.
- [`useDeferredValue`](/reference/react/useDeferredValue) permite que você adie a atualização de uma parte não crítica da UI e deixe outras partes atualizarem primeiro.

---

## Other Hooks {/*other-hooks*/}

Esses Hooks são principalmente úteis para autores de bibliotecas e não são comumente usados no código do aplicativo.

- [`useDebugValue`](/reference/react/useDebugValue) permite que você personalize o rótulo que o React DevTools exibe para seu Hook customizado.
- [`useId`](/reference/react/useId) permite que um componente associe um ID único a si mesmo. Tipicamente usado com APIs de acessibilidade.
- [`useSyncExternalStore`](/reference/react/useSyncExternalStore) permite que um componente se inscreva em um armazenamento externo.
* [`useActionState`](/reference/react/useActionState) permite que você gerencie o state de actions.

---

## Your own Hooks {/*your-own-hooks*/}

Você também pode [definir seus próprios Hooks customizados](/learn/reusing-logic-with-custom-hooks#extracting-your-own-custom-hook-from-a-component) como funções JavaScript.
```