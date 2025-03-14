# Guia de Estilo Universal

Este documento descreve as regras que devem ser aplicadas para **todos** os idiomas.
Quando estiver se referindo ao próprio `React`, use `o React`.

## IDs dos Títulos

Todos os títulos possuem IDs explícitos como abaixo:

```md
## Tente React {#try-react}
```

**Não** traduza estes IDs! Eles são usado para navegação e quebrarão se o documento for um link externo, como:

```md
Veja a [seção iniciando](/getting-started#try-react) para mais informações.
```

✅ FAÇA:

```md
## Tente React {#try-react}
```

❌ NÃO FAÇA:

```md
## Tente React {#tente-react}
```

Isto quebraria o link acima.

## Texto em Blocos de Código

Mantenha o texto em blocos de código sem tradução, exceto para os comentários. Você pode optar por traduzir o texto em strings, mas tenha cuidado para não traduzir strings que se refiram ao código!

Exemplo:

```js
// Example
const element = <h1>Hello, world</h1>;
ReactDOM.render(element, document.getElementById('root'));
```

✅ FAÇA:

```js
// Exemplo
const element = <h1>Hello, world</h1>;
ReactDOM.render(element, document.getElementById('root'));
```

✅ PERMITIDO:

```js
// Exemplo
const element = <h1>Olá mundo</h1>;
ReactDOM.render(element, document.getElementById('root'));
```

❌ NÃO FAÇA:

```js
// Exemplo
const element = <h1>Olá mundo</h1>;
// "root" se refere a um ID de elemento.
// NÃO TRADUZA
ReactDOM.render(element, document.getElementById('raiz'));
```

❌ DEFINITIVAMENTE NÃO FAÇA:

```js
// Exemplo
const elemento = <h1>Olá mundo</h1>;
ReactDOM.renderizar(elemento, documento.obterElementoPorId('raiz'));
```

## Links Externos

Se um link externo se referir a um artigo no [MDN] or [Wikipedia] e se houver uma versão traduzida em seu idioma em uma qualidade decente, opte por usar a versão traduzida.

[mdn]: https://developer.mozilla.org/pt-BR/
[wikipedia]: https://pt.wikipedia.org/wiki/Wikipédia:Página_principal

Exemplo:

```md
React elements are [immutable](https://en.wikipedia.org/wiki/Immutable_object).
```

✅ OK:

```md
Elementos React são [imutáveis](https://pt.wikipedia.org/wiki/Objeto_imutável).
```

Para links que não possuem tradução (Stack Overflow, vídeos do YouTube, etc.), simplesmente use o link original.

## Traduções Comuns

Sugestões de palavras e termos:

| Palavra/Termo original | Sugestão                               |
| ---------------------- | -------------------------------------- |
| assertion              | asserção                               |
| at the top level       | na raiz                                |
| browser                | navegador                              |
| bubbling               | propagar                               |
| bug                    | erro                                   |
| caveats                | ressalvas                              |
| class component        | componente de classe                   |
| class                  | classe                                 |
| client                 | cliente                                |
| client-side            | lado do cliente                        |
| container              | contêiner                              |
| context                | contexto                               |
| controlled component   | componente controlado                  |
| debugging              | depuração                              |
| DOM node               | nó do DOM                              |
| event handler          | manipulador de eventos (event handler) |
| function component     | componente de função                   |
| handler                | manipulador                            |
| helper function        | função auxiliar                        |
| high-order components  | componente de alta-ordem               |
| key                    | chave                                  |
| library                | biblioteca                             |
| lowercase              | minúscula(s) / caixa baixa             |
| package                | pacote                                 |
| React element          | Elemento React                         |
| React fragment         | Fragmento React                        |
| render                 | renderizar (verb), renderizado (noun)  |
| server                 | servidor                               |
| server-side            | lado do servidor                       |
| siblings               | irmãos                                 |
| stateful component     | componente com estado                  |
| stateful logic         | lógica com estado                      |
| to assert              | afirmar                                |
| to wrap                | encapsular                             |
| troubleshooting        | solução de problemas                   |
| uncontrolled component | componente não controlado              |
| uppercase              | maiúscula(s) / caixa alta              |

## Conteúdo que não deve ser traduzido

- array
- arrow function
- bind
- bundle
- bundler
- callback
- camelCase
- DOM
- event listener
- framework
- hook
- log
- mock
- portal
- props
- ref
- release
- script
- single-page-apps
- state
- string
- string literal
- subscribe
- subscription
- template literal
- timestamps
- UI
- watcher
- widgets
- wrapper
---
title: renderToStaticMarkup
---

<Intro>

`renderToStaticMarkup` renderiza uma árvore React não interativa para uma HTML string.

```js
const html = renderToStaticMarkup(reactNode, options?)
```

</Intro>

<InlineToc />

---

## Referência {/*reference*/}

### `renderToStaticMarkup(reactNode, options?)` {/*rendertostaticmarkup*/}

No servidor, chame `renderToStaticMarkup` para renderizar o seu app para HTML.

```js
import { renderToStaticMarkup } from 'react-dom/server';

const html = renderToStaticMarkup(<Page />);
```

Isso produzirá uma saída HTML não interativa de seus componentes React.

[Veja mais exemplos abaixo.](#usage)

#### Parâmetros {/*parameters*/}

* `reactNode`: Um React node que você quer renderizar para HTML. Por exemplo, um nó JSX como `<Page />`.
* **opcional** `options`: Um objeto para renderização no servidor.
  * **opcional** `identifierPrefix`: Um prefixo string que o React usa para IDs gerados por [`useId`.](/reference/react/useId) Útil para evitar conflitos ao usar múltiplos roots na mesma página.

#### Retorna {/*returns*/}

Uma HTML string.

#### Ressalvas {/*caveats*/}

* A saída de `renderToStaticMarkup` não pode ser hidratada.

* `renderToStaticMarkup` possui suporte limitado ao Suspense. Se um componente suspender, `renderToStaticMarkup` imediatamente envia seu fallback como HTML.

* `renderToStaticMarkup` funciona no navegador, mas não é recomendado usá-lo no código do cliente. Se você precisa renderizar um componente para HTML no navegador, [obtenha o HTML renderizando-o em um nó DOM.](/reference/react-dom/server/renderToString#removing-rendertostring-from-the-client-code)

---

## Uso {/*usage*/}

### Renderizando uma árvore React não interativa como HTML para uma string {/*rendering-a-non-interactive-react-tree-as-html-to-a-string*/}

Chame `renderToStaticMarkup` para renderizar o seu app para uma HTML string que você pode enviar com sua resposta do servidor:

```js {5-6}
import { renderToStaticMarkup } from 'react-dom/server';

// A sintaxe do manipulador de rota depende do seu framework de backend
app.use('/', (request, response) => {
  const html = renderToStaticMarkup(<Page />);
  response.send(html);
});
```

Isso produzirá a saída HTML inicial não interativa de seus componentes React.

<Pitfall>

Este método renderiza **HTML não interativo que não pode ser hidratado.** Isso é útil caso você queira usar o React como um gerador de página estática simples, ou se você estiver renderizando conteúdo completamente estático como e-mails.

Aplicativos interativos devem usar [`renderToString`](/reference/react-dom/server/renderToString) no servidor e [`hydrateRoot`](/reference/react-dom/client/hydrateRoot) no cliente.

</Pitfall>