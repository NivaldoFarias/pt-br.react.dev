---
title: "Built-in React Hooks"
---

<Intro>

*Hooks* permitem que você use diferentes funcionalidades do React em seus componentes. Você pode usar os Hooks incorporados ou combiná-los para construir os seus próprios. Esta página lista todos os Hooks incorporados ao React.

</Intro>

---

## State Hooks {/*state-hooks*/}

O *estado* permite que um componente ["lembre" informações como entrada do usuário.](/learn/state-a-components-memory) Por exemplo, um componente de formulário pode usar o estado para armazenar o valor de entrada, enquanto um componente de galeria de imagens pode usar o estado para armazenar o índice da imagem selecionada.

Para adicionar estado a um componente, use um destes Hooks:

* [`useState`](/reference/react/useState) declara uma variável de estado que você pode atualizar diretamente.
* [`useReducer`](/reference/react/useReducer) declara uma variável de estado com a lógica de atualização dentro de uma [função reducer.](/learn/extracting-state-logic-into-a-reducer)

```js
function ImageGallery() {
  const [index, setIndex] = useState(0);
  // ...
```

---

## Context Hooks {/*context-hooks*/}

O *contexto* permite que um componente [receba informações de pais distantes sem passá-las como props.](/learn/passing-props-to-a-component) Por exemplo, o componente de nível superior do seu aplicativo pode passar o tema atual da UI para todos os componentes abaixo, não importa a profundidade.

* [`useContext`](/reference/react/useContext) lê e se inscreve em um contexto.

```js
function Button() {
  const theme = useContext(ThemeContext);
  // ...
```

---

## Ref Hooks {/*ref-hooks*/}

*Refs* permitem que um componente [mantenha alguma informação que não é usada para renderização,](/learn/referencing-values-with-refs) como um nó DOM ou um ID de tempo limite. Diferente do estado, atualizar uma ref não re-renderiza o seu componente. Refs são uma "saída de emergência" do paradigma React. Elas são úteis quando você precisa trabalhar com sistemas que não são React, como as APIs do navegador.

* [`useRef`](/reference/react/useRef) declara uma ref. Você pode guardar qualquer valor nela, mas com mais frequência é usada para guardar um nó DOM.
* [`useImperativeHandle`](/reference/react/useImperativeHandle) permite que você personalize a ref exposta pelo seu componente. Isto é raramente usado.

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

Effects são uma "saída de emergência" do paradigma React. Não use Effects para orquestrar o fluxo de dados do seu aplicativo. Se você não estiver interagindo com um sistema externo, [você pode não precisar de um Effect.](/learn/you-might-not-need-an-effect)

Há duas variações raramente usadas de `useEffect` com diferenças no tempo de execução:

* [`useLayoutEffect`](/reference/react/useLayoutEffect) dispara antes do navegador redesenhar a tela. Você pode medir o layout aqui.
* [`useInsertionEffect`](/reference/react/useInsertionEffect) dispara antes que React faça mudanças no DOM. Bibliotecas podem inserir CSS dinâmico aqui.

---

## Performance Hooks {/*performance-hooks*/}

Uma forma comum de otimizar o desempenho de re-renderização é ignorar o trabalho desnecessário. Por exemplo, você pode dizer para o React reutilizar um cálculo em cache ou ignorar uma re-renderização se os dados não mudaram desde a renderização anterior.

Para ignorar cálculos e re-renderizações desnecessárias, use um destes Hooks:

- [`useMemo`](/reference/react/useMemo) permite que você faça cache do resultado de um cálculo dispendioso.
- [`useCallback`](/reference/react/useCallback) permite que você faça cache de uma definição de função antes de passá-la para um componente otimizado.

```js
function TodoList({ todos, tab, theme }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  // ...
}
```

Às vezes, você não pode ignorar a re-renderização porque a tela realmente precisa atualizar. Nesse caso, você pode melhorar o desempenho separando atualizações de bloqueio que devem ser síncronas (como digitar em uma entrada) de atualizações não bloqueadas que não precisam bloquear a interface do usuário (como atualizar um gráfico).

Para priorizar a renderização, use um desses Hooks:

- [`useTransition`](/reference/react/useTransition) permite que você marque uma transição de estado como não bloqueante e permita que outras atualizações a interrompam.
- [`useDeferredValue`](/reference/react/useDeferredValue) permite que você adie a atualização de uma parte não crítica da UI e deixe que outras partes atualizem primeiro.

---

## Other Hooks {/*other-hooks*/}

Esses Hooks são principalmente úteis para autores de bibliotecas e não são comumente usados no código do aplicativo.

- [`useDebugValue`](/reference/react/useDebugValue) permite que você personalize o rótulo que o React DevTools exibe para seu Hook personalizado.
- [`useId`](/reference/react/useId) permite que um componente associe um ID único a si mesmo. Normalmente usado com APIs de acessibilidade.
- [`useSyncExternalStore`](/reference/react/useSyncExternalStore) permite que um componente se inscreva em um armazenamento externo.
* [`useActionState`](/reference/react/useActionState) allows you to manage state of actions.

---

## Your own Hooks {/*your-own-hooks*/}

Você também pode [definir seus próprios Hooks customizados](/learn/reusing-logic-with-custom-hooks#extracting-your-own-custom-hook-from-a-component) como funções JavaScript.