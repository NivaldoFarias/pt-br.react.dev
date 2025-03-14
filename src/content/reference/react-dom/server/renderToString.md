---
title: renderToString
---

<Pitfall>

`renderToString` não suporta streaming ou esperar por dados. [Veja as alternativas.](#alternatives)

</Pitfall>

<Intro>

`renderToString` renderiza uma árvore React para uma string HTML.

```js
const html = renderToString(reactNode, options?)
```

</Intro>

<InlineToc />

---

## Referência {/*reference*/}

### `renderToString(reactNode, options?)` {/*rendertostring*/}

No servidor, chame `renderToString` para renderizar seu app para HTML.

```js
import { renderToString } from 'react-dom/server';

const html = renderToString(<App />);
```

No cliente, chame [`hydrateRoot`](/reference/react-dom/client/hydrateRoot) para tornar o HTML gerado pelo servidor interativo.

[Veja mais exemplos abaixo.](#usage)

#### Parâmetros {/*parameters*/}

* `reactNode`: Um nó React que você quer renderizar para HTML. Por exemplo, um nó JSX como `<App />`.

* **opcional** `options`: Um objeto para renderização no servidor.
  * **opcional** `identifierPrefix`: Um prefixo de string que o React usa para IDs gerados por [`useId`.](/reference/react/useId) Útil para evitar conflitos ao usar múltiplas roots na mesma página. Deve ser o mesmo prefixo que foi passado para [`hydrateRoot`.](/reference/react-dom/client/hydrateRoot#parameters)

#### Retorna {/*returns*/}

Uma string HTML.

#### Ressalvas {/*caveats*/}

* `renderToString` tem suporte limitado ao Suspense. Se um componente suspender, `renderToString` imediatamente envia seu fallback como HTML.

* `renderToString` funciona no navegador, mas usá-lo no código do cliente [não é recomendado.](#removing-rendertostring-from-the-client-code)

---

## Uso {/*usage*/}

### Renderizando uma árvore React como HTML para uma string {/*rendering-a-react-tree-as-html-to-a-string*/}

Chame `renderToString` para renderizar seu app para uma string HTML que você pode enviar com sua resposta do servidor:

```js {5-6}
import { renderToString } from 'react-dom/server';

// A sintaxe do manipulador de rota depende do seu framework de backend
app.use('/', (request, response) => {
  const html = renderToString(<App />);
  response.send(html);
});
```

Isso produzirá a saída HTML inicial não interativa de seus componentes React. No cliente, você precisará chamar [`hydrateRoot`](/reference/react-dom/client/hydrateRoot)para *hidratar* esse HTML gerado pelo servidor e torná-lo interativo.

<Pitfall>

`renderToString` não suporta streaming ou esperar por dados. [Veja as alternativas.](#alternatives)

</Pitfall>

---

## Alternativas {/*alternatives*/}

### Migrando de `renderToString` para uma renderização com streaming no servidor {/*migrating-from-rendertostring-to-a-streaming-method-on-the-server*/}

`renderToString` retorna uma string imediatamente, então não suporta streaming de conteúdo conforme ele carrega.

Quando possível, recomendamos o uso dessas alternativas com todos os recursos:

* Se você usa Node.js, use [`renderToPipeableStream`.](/reference/react-dom/server/renderToPipeableStream)
* Se você usa Deno ou um tempo de execução de borda moderno com [Web Streams](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API), use [`renderToReadableStream`.](/reference/react-dom/server/renderToReadableStream)

Você pode continuar usando `renderToString` se seu ambiente de servidor não suportar streams.

---

### Migrando de `renderToString` para um prerender estático no servidor {/*migrating-from-rendertostring-to-a-static-prerender-on-the-server*/}

`renderToString` retorna uma string imediatamente, então não suporta esperar por dados para carregar para a geração de HTML estático.

Recomendamos o uso dessas alternativas com todos os recursos:

* Se você usa Node.js, use [`prerenderToNodeStream`.](/reference/react-dom/static/prerenderToNodeStream)
* Se você usa Deno ou um tempo de execução de borda moderno com [Web Streams](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API), use [`prerender`.](/reference/react-dom/static/prerender)

Você pode continuar usando `renderToString` se seu ambiente de geração de site estático não suportar streams.

---

### Removendo `renderToString` do código do cliente {/*removing-rendertostring-from-the-client-code*/}

Às vezes, `renderToString` é usado no cliente para converter algum componente para HTML.

```js {1-2}
// 🚩 Desnecessário: usando renderToString no cliente
import { renderToString } from 'react-dom/server';

const html = renderToString(<MyIcon />);
console.log(html); // Por exemplo, "<svg>...</svg>"
```

Importar `react-dom/server` **no cliente** aumenta desnecessariamente o tamanho do seu bundle e deve ser evitado. Se você precisar renderizar algum componente para HTML no navegador, use [`createRoot`](/reference/react-dom/client/createRoot) e leia HTML do DOM:

```js
import { createRoot } from 'react-dom/client';
import { flushSync } from 'react-dom';

const div = document.createElement('div');
const root = createRoot(div);
flushSync(() => {
  root.render(<MyIcon />);
});
console.log(div.innerHTML); // Por exemplo, "<svg>...</svg>"
```

A chamada [`flushSync`](/reference/react-dom/flushSync) é necessária para que o DOM seja atualizado antes de ler sua propriedade [`innerHTML`](https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML).

---

## Solução de problemas {/*troubleshooting*/}

### Quando um componente suspende, o HTML sempre contém um fallback {/*when-a-component-suspends-the-html-always-contains-a-fallback*/}

`renderToString` não tem suporte total para Suspense.

Se algum componente suspender (por exemplo, porque é definido com [`lazy`](/reference/react/lazy) ou busca dados), `renderToString` não esperará que seu conteúdo seja resolvido. Em vez disso, `renderToString` encontrará o limite [`<Suspense>`](/reference/react/Suspense) mais próximo acima dele e renderizará sua prop `fallback` no HTML. O conteúdo não aparecerá até que o código do cliente carregue.

Para resolver isso, use uma das [soluções de streaming recomendadas.](#alternatives) Para renderização do lado do servidor, elas podem fazer streaming do conteúdo em pedaços conforme ele é resolvido no servidor, para que o usuário veja a página sendo progressivamente preenchida antes que o código do cliente carregue. Para geração de site estático, elas podem esperar que todo o conteúdo seja resolvido antes de gerar o HTML estático.