---
title: renderToPipeableStream
---

<Intro>

`renderToPipeableStream` renderiza uma árvore React para um [Stream do Node.js] pipeável.(https://nodejs.org/api/stream.html)

```js
const { pipe, abort } = renderToPipeableStream(reactNode, options?)
```

</Intro>

<InlineToc />

<Note>

Esta API é específica do Node.js. Ambientes com [Web Streams,](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API) como Deno e runtimes de borda modernos, devem usar [`renderToReadableStream`](/reference/react-dom/server/renderToReadableStream) em vez disso.

</Note>

---

## Referência {/*reference*/}

### `renderToPipeableStream(reactNode, options?)` {/*rendertopipeablestream*/}

Chame `renderToPipeableStream` para renderizar sua árvore React como HTML em um [Stream do Node.js.](https://nodejs.org/api/stream.html#writable-streams)

```js
import { renderToPipeableStream } from 'react-dom/server';

const { pipe } = renderToPipeableStream(<App />, {
  bootstrapScripts: ['/main.js'],
  onShellReady() {
    response.setHeader('content-type', 'text/html');
    pipe(response);
  }
});
```

No cliente, chame [`hydrateRoot`](/reference/react-dom/client/hydrateRoot) para tornar o HTML gerado pelo servidor interativo.

[Veja mais exemplos abaixo.](#usage)

#### Parâmetros {/*parameters*/}

* `reactNode`: Um nó React que você quer renderizar em HTML. Por exemplo, um elemento JSX como `<App />`. Espera-se que ele represente o documento inteiro, então o componente `App` deve renderizar a tag `<html>`.

* **opcional** `options`: Um objeto com opções de streaming.
  * **opcional** `bootstrapScriptContent`: Se especificado, esta string será colocada em uma tag `<script>` embutida.
  * **opcional** `bootstrapScripts`: Uma array de URLs de string para as tags `<script>` a serem emitidas na página. Use isto para incluir o `<script>` que chama [`hydrateRoot`.](/reference/react-dom/client/hydrateRoot) Omita-o se você não quiser executar React no cliente.
  * **optional** `bootstrapModules`: Como `bootstrapScripts`, mas emite [`<script type="module">`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) em vez disso.
  * **opcional** `identifierPrefix`: Um prefixo de string que o React usa para IDs gerados por [`useId`.](/reference/react/useId) Útil para evitar conflitos ao usar várias roots na mesma página. Deve ser o mesmo prefixo que foi passado para [`hydrateRoot`.](/reference/react-dom/client/hydrateRoot#parameters)
  * **opcional** `namespaceURI`: Uma string com o [namespace URI](https://developer.mozilla.org/en-US/docs/Web/API/Document/createElementNS#important_namespace_uris) da root para o stream. O padrão é HTML normal. Passe `'http://www.w3.org/2000/svg'` para SVG ou `'http://www.w3.org/1998/Math/MathML'` para MathML.
  * **opcional** `nonce`: Uma string [`nonce`](http://developer.mozilla.org/en-US/docs/Web/HTML/Element/script#nonce) para permitir scripts para [`script-src` de Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/script-src).
  * **opcional** `onAllReady`: Uma callback que dispara quando toda a renderização está completa, incluindo tanto o [shell](#specifying-what-goes-into-the-shell) quanto todo [conteúdo adicional.](#streaming-more-content-as-it-loads) Você pode usar isto em vez de `onShellReady` [para rastreadores e geração estática.](#waiting-for-all-content-to-load-for-crawlers-and-static-generation) Se você começar a fazer o streaming aqui, você não terá nenhuma carga progressiva. O stream conterá o HTML final.
  * **opcional** `onError`: Uma callback que dispara sempre que há um erro no servidor, [recuperável](#recovering-from-errors-outside-the-shell) ou [não.](#recovering-from-errors-inside-the-shell) Por padrão, isto só chama `console.error`. Se você substituí-lo para [registrar relatórios de falhas,](#logging-crashes-on-the-server) certifique-se de que você ainda chama `console.error`. Você também pode usá-lo para [ajustar o código de status](#setting-the-status-code) antes que o shell seja emitido.
  * **opcional** `onShellReady`: Uma callback que dispara logo após o [shell inicial](#specifying-what-goes-into-the-shell) ter sido renderizado. Você pode [definir o código de status](#setting-the-status-code) e chamar `pipe` aqui para começar o streaming. React irá [transmitir o conteúdo adicional](#streaming-more-content-as-it-loads) após o shell junto com as tags `<script>` embutidas que substituem as loading fallbacks de HTML com o conteúdo.
  * **opcional** `onShellError`: Uma callback que dispara se houve um erro ao renderizar o shell inicial. Recebe o erro como um argumento. Nenhum byte foi emitido do stream ainda, e nem `onShellReady` nem `onAllReady` serão chamados, então você pode [enviar um shell HTML de fallback.](#recovering-from-errors-inside-the-shell)
  * **opcional** `progressiveChunkSize`: O número de bytes em um chunk. [Leia mais sobre a heurística padrão.](https://github.com/facebook/react/blob/14c2be8dac2d5482fda8a0906a31d239df8551fc/packages/react-server/src/ReactFizzServer.js#L210-L225)


#### Retorna {/*returns*/}

`renderToPipeableStream` retorna um objeto com dois métodos:

* `pipe` envia o HTML no [Writable Node.js Stream] fornecido.(https://nodejs.org/api/stream.html#writable-streams) Chame `pipe` em `onShellReady` se você quiser habilitar streaming, ou em `onAllReady` para rastreadores e geração estática.
* `abort` permite que você [encerre a renderização do servidor](#aborting-server-rendering) e renderize o restante no cliente.

---

## Uso {/*usage*/}

### Renderizando uma árvore React como HTML para um Node.js Stream {/*rendering-a-react-tree-as-html-to-a-nodejs-stream*/}

Chame `renderToPipeableStream` para renderizar sua árvore React como HTML em um [Stream do Node.js:](https://nodejs.org/api/stream.html#writable-streams)

```js [[1, 5, "<App />"], [2, 6, "['/main.js']"]]
import { renderToPipeableStream } from 'react-dom/server';

// A sintaxe do manipulador de rotas depende do seu framework de back-end
app.use('/', (request, response) => {
  const { pipe } = renderToPipeableStream(<App />, {
    bootstrapScripts: ['/main.js'],
    onShellReady() {
      response.setHeader('content-type', 'text/html');
      pipe(response);
    }
  });
});
```

Junto com o <CodeStep step={1}>componente root</CodeStep>, você precisa fornecer uma lista de <CodeStep step={2}>caminhos de `<script>` bootstrap</CodeStep>. Seu componente root deve retornar **o documento inteiro, incluindo a tag `<html>` root.**

Por exemplo, pode se parecer com isto:

```js [[1, 1, "App"]]
export default function App() {
  return (
    <html>
      <head>
        <meta charSet="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
        <link rel="stylesheet" href="/styles.css"></link>
        <title>My app</title>
      </head>
      <body>
        <Router />
      </body>
    </html>
  );
}
```

React irá injetar o [doctype](https://developer.mozilla.org/en-US/docs/Glossary/Doctype) e suas tags de <CodeStep step={2}>`<script>` bootstrap</CodeStep> no stream HTML resultante:

```html [[2, 5, "/main.js"]]
<!DOCTYPE html>
<html>
  <!-- ... HTML from your components ... -->
</html>
<script src="/main.js" async=""></script>
```

No cliente, seu script bootstrap deve [hidratar o `document` inteiro com uma chamada para `hydrateRoot`:](/reference/react-dom/client/hydrateRoot#hydrating-an-entire-document)

```js [[1, 4, "<App />"]]
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(document, <App />);
```

Isto irá anexar manipuladores de eventos ao HTML gerado pelo servidor e torná-lo interativo.

<DeepDive>

#### Lendo caminhos de assets CSS e JS da saída de build {/*reading-css-and-js-asset-paths-from-the-build-output*/}

As URLs de asset finais (como arquivos JavaScript e CSS) são frequentemente hash depois do build. Por exemplo, em vez de `styles.css` você pode acabar com `styles.123456.css`. O hashing de nomes de arquivos de asset estáticos garante que cada build distinto do mesmo asset terá um nome de arquivo diferente. Isto é útil porque permite que você habilite com segurança o caching de longo prazo para assets estáticos: um arquivo com um determinado nome nunca mudaria o conteúdo.

No entanto, se você não sabe as URLs de asset até depois do build, não há como colocá-las no código fonte. Por exemplo, codificar `"/styles.css"` em JSX como antes não funcionaria. Para mantê-las fora do seu código fonte, seu componente root pode ler os nomes de arquivos reais de um mapa passado como uma prop:

```js {1,6}
export default function App({ assetMap }) {
  return (
    <html>
      <head>
        ...
        <link rel="stylesheet" href={assetMap['styles.css']}></link>
        ...
      </head>
      ...
    </html>
  );
}
```

No servidor, renderize `<App assetMap={assetMap} />` e passe seu `assetMap` com as URLs do asset:

```js {1-5,8,9}
// Você precisaria obter este JSON de suas ferramentas de construção, por exemplo, lê-lo da saída da construção.
const assetMap = {
  'styles.css': '/styles.123456.css',
  'main.js': '/main.123456.js'
};

app.use('/', (request, response) => {
  const { pipe } = renderToPipeableStream(<App assetMap={assetMap} />, {
    bootstrapScripts: [assetMap['main.js']],
    onShellReady() {
      response.setHeader('content-type', 'text/html');
      pipe(response);
    }
  });
});
```

Como seu servidor agora está renderizando `<App assetMap={assetMap} />`, você precisa renderizá-lo com `assetMap` no cliente também para evitar erros de hidratação. Você pode serializar e passar `assetMap` para o cliente assim:

```js {9-10}
// Você precisaria obter este JSON de suas ferramentas de construção.
const assetMap = {
  'styles.css': '/styles.123456.css',
  'main.js': '/main.123456.js'
};

app.use('/', (request, response) => {
  const { pipe } = renderToPipeableStream(<App assetMap={assetMap} />, {
    // Cuidado: É seguro stringify() isto porque estes dados não são gerados pelo usuário.
    bootstrapScriptContent: `window.assetMap = ${JSON.stringify(assetMap)};`,
    bootstrapScripts: [assetMap['main.js']],
    onShellReady() {
      response.setHeader('content-type', 'text/html');
      pipe(response);
    }
  });
});
```

No exemplo acima, a opção `bootstrapScriptContent` adiciona uma tag `<script>` embutida extra que define a variável global `window.assetMap` no cliente. Isto permite que o código do cliente leia o mesmo `assetMap`:

```js {4}
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(document, <App assetMap={window.assetMap} />);
```

Tanto o cliente quanto o servidor renderizam `App` com a mesma prop `assetMap`, então não há erros de hidratação.

</DeepDive>

---

### Streaming de mais conteúdo conforme ele carrega {/*streaming-more-content-as-it-loads*/}

O streaming permite que o usuário comece a ver o conteúdo mesmo antes que todos os dados tenham carregado no servidor. Por exemplo, considere uma página de perfil que mostra uma capa, uma barra lateral com amigos e fotos, e uma lista de posts:

```js
function ProfilePage() {
  return (
    <ProfileLayout>
      <ProfileCover />
      <Sidebar>
        <Friends />
        <Photos />
      </Sidebar>
      <Posts />
    </ProfileLayout>
  );
}
```

Imagine que carregar dados para `<Posts />` leva algum tempo. Idealmente, você gostaria de mostrar o resto do conteúdo da página do perfil ao usuário sem esperar pelos posts. Para fazer isto, [envolva `Posts` em um limite `<Suspense>`:](/reference/react/Suspense#displaying-a-fallback-while-content-is-loading)

```js {9,11}
function ProfilePage() {
  return (
    <ProfileLayout>
      <ProfileCover />
      <Sidebar>
        <Friends />
        <Photos />
      </Sidebar>
      <Suspense fallback={<PostsGlimmer />}>
        <Posts />
      </Suspense>
    </ProfileLayout>
  );
}
```

Isto diz ao React para começar a transmitir o HTML antes que `Posts` carregue seus dados. React irá enviar o HTML para o fallback de carregamento (`PostsGlimmer`) primeiro, e então, quando `Posts` terminar de carregar seus dados, React irá enviar o HTML restante junto com uma tag `<script>` embutida que substitui o fallback de carregamento com aquele HTML. Da perspectiva do usuário, a página aparecerá primeiro com o `PostsGlimmer`, mais tarde substituído pelos `Posts`.

Você pode ainda [aninhas limites `<Suspense>`](/reference/react/Suspense#revealing-nested-content-as-it-loads) para criar uma sequência de carregamento mais granular:

```js {5,13}
function ProfilePage() {
  return (
    <ProfileLayout>
      <ProfileCover />
      <Suspense fallback={<BigSpinner />}>
        <Sidebar>
          <Friends />
          <Photos />
        </Sidebar>
        <Suspense fallback={<PostsGlimmer />}>
          <Posts />
        </Suspense>
      </Suspense>
    </ProfileLayout>
  );
}
```

Neste exemplo, React pode começar a transmitir a página ainda mais cedo. Somente `ProfileLayout` e `ProfileCover` devem terminar a renderização primeiro porque eles não são envolvidos em nenhum limite `<Suspense>`. No entanto, se `Sidebar`, `Friends`, ou `Photos` precisarem carregar alguns dados, React irá enviar o HTML para o fallback `BigSpinner` em vez disso. Então, conforme mais dados ficam disponíveis, mais conteúdo continuará a ser revelado até que tudo se torne visível.

O streaming não precisa esperar que o próprio React carregue no navegador, ou que seu app se torne interativo. O conteúdo HTML do servidor será progressivamente revelado antes que qualquer uma das tags `<script>` carreguem.

[Leia mais sobre como funciona o streaming HTML.](https://github.com/reactwg/react-18/discussions/37)

<Note>

**Apenas as fontes de dados habilitadas para Suspense ativarão o componente Suspense.** Elas incluem:

- Obtenção de dados com frameworks habilitados para Suspense como [Relay](https://relay.dev/docs/guided-tour/rendering/loading-states/) e [Next.js](https://nextjs.org/docs/getting-started/react-essentials)
- Código de componente lazy-loading com [`lazy`](/reference/react/lazy)
- Lendo o valor de uma Promise com [`use`](/reference/react/use)

Suspense **não** detecta quando os dados são obtidos de dentro de um Effect ou manipulador de eventos.

A forma exata como você carregaria dados no componente `Posts` acima depende do seu framework. Se você usar um framework habilitado para Suspense, você encontrará os detalhes em sua documentação de obtenção de dados.

A obtenção de dados habilitada para Suspense sem o uso de um framework com opinião ainda não é suportada. Os requisitos para implementar uma fonte de dados habilitada para Suspense são instáveis e não documentados. Uma API oficial para integrar fontes de dados com Suspense será lançada em uma versão futura do React.

</Note>

---

### Especificando o que vai no shell {/*specifying-what-goes-into-the-shell*/}

A parte do seu app fora de qualquer limite `<Suspense>` é chamada *shell:*

```js {3-5,13,14}
function ProfilePage() {
  return (
    <ProfileLayout>
      <ProfileCover />
      <Suspense fallback={<BigSpinner />}>
        <Sidebar>
          <Friends />
          <Photos />
        </Sidebar>
        <Suspense fallback={<PostsGlimmer />}>
          <Posts />
        </Suspense>
      </Suspense>
    </ProfileLayout>
  );
}
```

Ela determina o primeiro estado de carregamento que o usuário pode ver:

```js {3-5,13
<ProfileLayout>
  <ProfileCover />
  <BigSpinner />
</ProfileLayout>
```

Se você envolver o app inteiro em um limite `<Suspense>` na root, o shell só conterá aquele spinner. No entanto, essa não é uma experiência do usuário agradável porque ver um spinner grande na tela pode parecer mais lento e mais irritante do que esperar um pouco mais e ver o layout real. É por isso que geralmente você vai querer colocar os limites `<Suspense>` para que o shell pareça *mínimo mas completo*--como um esqueleto de todo o layout da página.

A callback `onShellReady` dispara quando todo o shell foi renderizado. Geralmente, você irá começar a transmitir então:

```js {3-6}
const { pipe } = renderToPipeableStream(<App />, {
  bootstrapScripts: ['/main.js'],
  onShellReady() {
    response.setHeader('content-type', 'text/html');
    pipe(response);
  }
});
```

No momento em que `onShellReady` dispara, os componentes nos limites `<Suspense>` aninhados ainda podem estar carregando dados.

---

### Registrando falhas no servidor {/*logging-crashes-on-the-server*/}

Por padrão, todos os erros no servidor são logados no console. Você pode substituir este comportamento para registrar relatórios de falhas:

```js {7-10}
const { pipe } = renderToPipeableStream(<App />, {
  bootstrapScripts: ['/main.js'],
  onShellReady() {
    response.setHeader('content-type', 'text/html');
    pipe(response);
  },
  onError(error) {
    console.error(error);
    logServerCrashReport(error);
  }
});
```

Se você fornecer uma implementação `onError` customizada, não se esqueça de também registrar erros no console como acima.

---

### Recuperando de erros dentro do shell {/*recovering-from-errors-inside-the-shell*/}

Neste exemplo, o shell contém `ProfileLayout`, `ProfileCover`, e `PostsGlimmer`:

```js {3-5,7-8}
function ProfilePage() {
  return (
    <ProfileLayout>
      <ProfileCover />
      <Suspense fallback={<PostsGlimmer />}>
        <Posts />
      </Suspense>
    </ProfileLayout>
  );
}
```

Se um erro ocorrer ao renderizar esses componentes, React não terá nenhum HTML significativo para enviar para o cliente. Substitua `onShellError` para enviar um HTML de fallback que não se baseie na renderização do servidor como último recurso:

```js {7-11}
const { pipe } = renderToPipeableStream(<App />, {
  bootstrapScripts: ['/main.js'],
  onShellReady() {
    response.setHeader('content-type', 'text/html');
    pipe(response);
  },
  onShellError(error) {
    response.statusCode = 500;
    response.setHeader('content-type', 'text/html');
    response.send('<h1>Something went wrong</h1>'); 
  },
  onError(error) {
    console.error(error);
    logServerCrashReport(error);
  }
});
```

Se houver um erro ao gerar o shell, tanto `onError` quanto `onShellError` serão disparados. Use `onError` para relatório de erros e use `onShellError` para enviar o documento HTML de fallback. Seu HTML de fallback não precisa ser uma página de erro. Em vez disso, você pode incluir um shell alternativo que renderiza seu app apenas no cliente.

---

### Recuperando de erros fora do shell {/*recovering-from-errors-outside-the-shell*/}

Neste exemplo, o componente `<Posts />` está envolvido em `<Suspense>` então ele *não* faz parte do shell:

```js {6}
function ProfilePage() {
  return (
    <ProfileLayout>
      <ProfileCover />
      <Suspense fallback={<PostsGlimmer />}>
        <Posts />
      </Suspense>
    </ProfileLayout>
  );
}
```

Se um erro acontecer no componente `Posts` ou em algum lugar dentro dele, React irá [tentar se recuperar dele:](/reference/react/Suspense#providing-a-fallback-for-server-errors-and-client-only-content)

1. Ele enviará o loading fallback para o limite `<Suspense>` mais próximo (`PostsGlimmer`) no HTML.
2. Ele vai "desistir" de tentar renderizar o conteúdo `Posts` no servidor.
3. Quando o código JavaScript carregar no cliente, React irá *tentar* a renderização de `Posts` no cliente.

Se tentar renderizar `Posts` no cliente *também* falhar, React lançará o erro no cliente. Como com todos os erros lançados durante a renderização, o [limite de erro pai mais próximo](/reference/react/Component#static-getderivedstatefromerror) determina como apresentar o erro ao usuário. Na prática, isto significa que o usuário verá um indicador de carregamento até que seja certo que o erro não será capaz de ser recuperado.

Se tentar renderizar `Posts` no cliente tiver sucesso, o loading fallback do servidor será substituído pela saída de renderização do cliente. O usuário não saberá que houve um erro de servidor. No entanto, a callback do servidor `onError` e as callbacks do cliente [`onRecoverableError`](/reference/react-dom/client/hydrateRoot#hydrateroot) serão disparadas para que você possa ser notificado sobre o erro.

---

### Definindo o código de status {/*setting-the-status-code*/}

O streaming introduz uma troca. Você quer começar a transmitir a página o mais cedo possível para que o usuário possa ver o conteúdo mais cedo. No entanto, uma vez que você começa a transmitir, você não pode mais definir o código de status da resposta.

Ao [dividir seu app](#specifying-what-goes-into-the-shell) no shell (acima de todos os limites `<Suspense>`) e o resto do conteúdo, você já resolveu parte deste problema. Se o shell apresentar erros, você receberá a callback `onShellError` que permite que você defina o código de status de erro. Caso contrário, você sabe que o app pode se recuperar no cliente, então você pode enviar "OK".

```js {4}
const { pipe } = renderToPipeableStream(<App />, {
  bootstrapScripts: ['/main.js'],
  onShellReady() {
    response.statusCode = 200;
    response.setHeader('content-type', 'text/html');
    pipe(response);
  },
  onShellError(error) {
    response.statusCode = 500;
    response.setHeader('content-type', 'text/html');
    response.send('<h1>Something went wrong</h1>'); 
  },
  onError(error) {
    console.error(error);
    logServerCrashReport(error);
  }
});
```

Se um componente *fora* do shell (ou seja, dentro de um limite `<Suspense>`) lançar um erro, React não irá parar a renderização. Isto significa que a callback `onError` será disparada, mas você ainda receberá `onShellReady` em vez de `onShellError`. Isto é porque React tentará se recuperar daquele erro no cliente, [conforme descrito acima.](#recovering-from-errors-outside-the-shell)

No entanto, se você quiser, você pode usar o fato de que algo apresentou erros para definir o código de status:

```js {1,6,16}
let didError = false;

const { pipe } = renderToPipeableStream(<App />, {
  bootstrapScripts: ['/main.js'],
  onShellReady() {
    response.statusCode = didError ? 500 : 200;
    response.setHeader('content-type', 'text/html');
    pipe(response);
  },
  onShellError(error) {
    response.statusCode = 500;
    response.setHeader('content-type', 'text/html');
    response.send('<h1>Something went wrong</h1>'); 
  },
  onError(error) {
    didError = true;
    console.error(error);
    logServerCrashReport(error);
  }
});
```

Isto só irá capturar erros fora do shell que aconteceram ao gerar o conteúdo inicial do shell, então não é exaustivo. Se saber se um erro ocorreu para algum conteúdo é crítico, você pode movê-lo para o shell.

---

### Lidando com diferentes erros de diferentes maneiras {/*handling-different-errors-in-different-ways*/}

Você pode [criar suas próprias subclasses `Error`](https://javascript.info/custom-errors) e usar o operador [`instanceof`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/instanceof) para verificar qual erro foi lançado. Por exemplo, você pode definir um `NotFoundError` customizado e lançá-lo do seu componente. Então suas callbacks `onError`, `onShellReady`, e `onShellError` podem fazer algo diferente dependendo do tipo de erro:

```js {2,4-14,19,24,30}
let didError = false;
let caughtError = null;

function getStatusCode() {
  if (didError) {
    if (caughtError instanceof NotFoundError) {
      return 404;
    } else {
      return 500;
    }
  } else {
    return 200;
  }
}

const { pipe } = renderToPipeableStream(<App />, {
  bootstrapScripts: ['/main.js'],
  onShellReady() {
    response.statusCode = getStatusCode();
    response.setHeader('content-type', 'text/html');
    pipe(response);
  },
  onShellError(error) {
   response.statusCode = getStatusCode();
   response.setHeader('content-type', 'text/html');
   response.send('<h1>Something went wrong</h1>'); 
  },
  onError(error) {
    didError = true;
    caughtError = error;
    console.error(error);
    logServerCrashReport(error);
  }
});
```

Tenha em mente que uma vez que você emite o shell e começa a transmitir, você não pode mudar o código de status.

---

### Esperando que todo o conteúdo carregue para rastreadores e geração estática {/*waiting-for-all-content-to-load-for-crawlers-and-static-generation*/}

O streaming oferece uma melhor experiência do usuário porque o usuário pode ver o conteúdo conforme ele fica disponível.

No entanto, quando um rastreador visita sua página, ou se você está gerando as páginas no momento da construção, você pode querer deixar todo o conteúdo carregar primeiro e então produzir a saída HTML final em vez de revelá-la progressivamente.

Você pode esperar todo o conteúdo carregar usando a callback `onAllReady`:


```js {2,7,11,18-24}
let didError = false;
let isCrawler = // ... depende de sua estratégia de detecção de bot ...

const { pipe } = renderToPipeableStream(<App />, {
  bootstrapScripts: ['/main.js'],
  onShellReady() {
    if (!isCrawler) {
      response.statusCode = didError ? 500 : 200;
      response.setHeader('content-type', 'text/html');
      pipe(response);
    }
  },
  onShellError(error) {
    response.statusCode = 500;
    response.setHeader('content-type', 'text/html');
    response.send('<h1>Something went wrong</h1>'); 
  },
  onAllReady() {
    if (isCrawler) {
      response.statusCode = didError ? 500 : 200;
      response.setHeader('content-type', 'text/html');
      pipe(response);      
    }
  },
  onError(error) {
    didError = true;
    console.error(error);
    logServerCrashReport(error);
  }
});
```

Um visitante regular receberá um stream de conteúdo carregado progressivamente. Um rastreador receberá a saída HTML final após todos os dados carregarem. No entanto, isto também significa que o rastreador terá que esperar por *todos* os dados, alguns dos quais podem ser lentos para carregar ou apresentar erros. Dependendo do seu app, você pode escolher enviar o shell para os rastreadores também.

---

### Interrompendo a renderização do servidor {/*aborting-server-rendering*/}

Você pode forçar a renderização do servidor a "desistir" após um tempo limite:

```js {1,5-7}
const { pipe, abort } = renderToPipeableStream(<App />, {
  // ...
});

setTimeout(() => {
  abort();
}, 10000);
```

React irá descartar as loading fallbacks restantes como HTML, e tentará renderizar o resto no cliente.
```