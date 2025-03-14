---
title: renderToString
---

<Pitfall>

`renderToString` não suporta streaming nem esperar por dados. [Veja as alternativas.](#alternatives)

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

No servidor, chame `renderToString` para renderizar seu aplicativo para HTML.

```js
import { renderToString } from 'react-dom/server';

const html = renderToString(<App />);
```

No cliente, chame [`hydrateRoot`](/reference/react-dom/client/hydrateRoot) para tornar o HTML gerado pelo servidor interativo.

[Veja mais exemplos abaixo.](#usage)

#### Parâmetros {/*parameters*/}

* `reactNode`: um nó React que você deseja renderizar em HTML. Por exemplo, um nó JSX como `<App />`.

* **opcional** `options`: um objeto para renderização no servidor.
  * **opcional** `identifierPrefix`: um prefixo de string que o React usa para IDs gerados por [`useId`.](/reference/react/useId) Útil para evitar conflitos ao usar várias raízes na mesma página. Deve ser o mesmo prefixo que foi passado para [`hydrateRoot`.](/reference/react-dom/client/hydrateRoot#parameters)

#### Retorna {/*returns*/}

Uma string HTML.

#### Ressalvas {/*caveats*/}

* `renderToString` tem suporte limitado a Suspense. Se um componente suspender, `renderToString` envia imediatamente seu fallback como HTML.

* `renderToString` funciona no navegador, mas usá-lo no código do cliente [não é recomendado.](#removing-rendertostring-from-the-client-code)

---

## Uso {/*usage*/}

### Renderizando uma árvore React como HTML para uma string {/*rendering-a-react-tree-as-html-to-a-string*/}

Chame `renderToString` para renderizar seu aplicativo em uma string HTML que você pode enviar com a resposta do seu servidor:

```js {5-6}
import { renderToString } from 'react-dom/server';

// A sintaxe do manipulador da rota depende da sua framework de backend
app.use('/', (request, response) => {
  const html = renderToString(<App />);
  response.send(html);
});
```

Isso produzirá a saída HTML inicial não interativa de seus componentes React. No cliente, você precisará chamar [`hydrateRoot`](/reference/react-dom/client/hydrateRoot) para *hidratar* o HTML gerado pelo servidor e torná-lo interativo.

<Pitfall>

`renderToString` não suporta streaming ou espera por dados. [Veja as alternativas.](#alternatives)

</Pitfall>

---

## Alternativas {/*alternatives*/}

### Migrando de `renderToString` para uma renderização de streaming no servidor {/*migrating-from-rendertostring-to-a-streaming-method-on-the-server*/}

`renderToString` retorna uma string imediatamente, portanto, não suporta streaming de conteúdo conforme ele carrega.

Quando possível, recomendamos o uso dessas alternativas completas:

* Se você usa Node.js, use [`renderToPipeableStream`.](/reference/react-dom/server/renderToPipeableStream)
* Se você usa Deno ou um tempo de execução moderno com [Web Streams](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API), use [`renderToReadableStream`.](/reference/react-dom/server/renderToReadableStream)

Você pode continuar usando `renderToString` se seu ambiente de servidor não oferecer suporte a streams.

---

### Migrando de `renderToString` para um prerender estático no servidor {/*migrating-from-rendertostring-to-a-static-prerender-on-the-server*/}

`renderToString` retorna uma string imediatamente, portanto, não suporta esperar que os dados carreguem para a geração de HTML estático.

Recomendamos o uso dessas alternativas completas:

* Se você usa Node.js, use [`prerenderToNodeStream`.](/reference/react-dom/static/prerenderToNodeStream)
* Se você usa Deno ou um tempo de execução moderno com [Web Streams](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API), use [`prerender`.](/reference/react-dom/static/prerender)

Você pode continuar usando `renderToString` se seu ambiente de geração de site estático não oferecer suporte a streams.

---

### Removendo `renderToString` do código do cliente {/*removing-rendertostring-from-the-client-code*/}

Às vezes, `renderToString` é usado no cliente para converter algum componente em HTML.

```js {1-2}
// 🚩 Desnecessário: usando renderToString no cliente
import { renderToString } from 'react-dom/server';

const html = renderToString(<MyIcon />);
console.log(html); // Por exemplo, "<svg>...</svg>"
```

Importar `react-dom/server` **no cliente** aumenta desnecessariamente o tamanho do seu pacote e deve ser evitado. Se você precisar renderizar algum componente para HTML no navegador, use [`createRoot`](/reference/react-dom/client/createRoot) e leia o HTML do DOM:

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

`renderToString` não oferece suporte total a Suspense.

Se algum componente suspender (por exemplo, porque ele é definido com [`lazy`](/reference/react/lazy) ou busca dados), `renderToString` não esperará que seu conteúdo seja resolvido. Em vez disso, `renderToString` encontrará o limite [`<Suspense>`](/reference/react/Suspense) mais próximo acima dele e renderizará sua propriedade `fallback` no HTML. O conteúdo não aparecerá até que o código do cliente seja carregado.

Para resolver isso, use uma das [soluções de streaming recomendadas.](#alternatives) Para a renderização no lado do servidor, elas podem transmitir conteúdo em blocos à medida que ele é resolvido no servidor, para que o usuário veja a página sendo progressivamente preenchida antes que o código do cliente carregue. Para a geração de sites estáticos, elas podem esperar que todo o conteúdo seja resolvido antes de gerar o HTML estático.
```