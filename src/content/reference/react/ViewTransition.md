---
title: >-
  # Guia de Estilo Universal


  Este documento descreve as regras que devem ser aplicadas para **todos** os
  idiomas.

  Quando estiver se referindo ao próprio `React`, use `o React`.


  ## IDs dos Títulos


  Todos os títulos possuem IDs explícitos como abaixo:


  ```md

  ## Tente React {#try-react}

  ```


  **Não** traduza estes IDs! Eles são usado para navegação e quebrarão se o
  documento for um link externo, como:


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


  Mantenha o texto em blocos de código sem tradução, exceto para os comentários.
  Você pode optar por traduzir o texto em strings, mas tenha cuidado para não
  traduzir strings que se refiram ao código!


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


  Se um link externo se referir a um artigo no [MDN] or [Wikipedia] e se houver
  uma versão traduzida em seu idioma em uma qualidade decente, opte por usar a
  versão traduzida.


  [mdn]: https://developer.mozilla.org/pt-BR/

  [wikipedia]: https://pt.wikipedia.org/wiki/Wikipédia:Página_principal


  Exemplo:


  ```md

  React elements are
  [immutable](https://en.wikipedia.org/wiki/Immutable_object).

  ```


  ✅ OK:


  ```md

  Elementos React são
  [imutáveis](https://pt.wikipedia.org/wiki/Objeto_imutável).

  ```


  Para links que não possuem tradução (Stack Overflow, vídeos do YouTube, etc.),
  simplesmente use o link original.


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
version: experimental
---
<Experimental>

**Esta API é experimental e ainda não está disponível em uma versão estável do React.**

Você pode experimentá-la atualizando os pacotes do React para a versão experimental mais recente:

- `react@experimental`
- `react-dom@experimental`
- `eslint-plugin-react-hooks@experimental`

As versões experimentais do React podem conter erros. Não as use em produção.

</Experimental>

<Intro>

`<ViewTransition>` permite animar elementos que são atualizados dentro de uma Transition.

```js
import {unstable_ViewTransition as ViewTransition} from 'react';

<ViewTransition>
  <div>...</div>
</ViewTransition>
```

</Intro>

<InlineToc />

---

## Referência {/*reference*/}

### `<ViewTransition>` {/*viewtransition*/}

Envolva elementos em `<ViewTransition>` para animá-los quando forem atualizados dentro de uma [Transition](/reference/react/useTransition). O React usa as seguintes heurísticas para determinar se uma View Transition é ativada para uma animação:

- `enter`: Se um `ViewTransition` em si for inserido nesta Transition, então isso será ativado.
- `exit`: Se um `ViewTransition` em si for excluído nesta Transition, então isso será ativado.
- `update`: Se um `ViewTransition` tiver alguma mutação do DOM dentro dele que o React está fazendo (como uma alteração de prop) ou se o próprio limite do `ViewTransition` mudar de tamanho ou posição devido a um irmão imediato. Se houver `ViewTransition` aninhados, a mutação se aplica a eles e não ao pai.
- `share`: Se um `ViewTransition` nomeado estiver dentro de uma subárvore excluída e outro `ViewTransition` nomeado com o mesmo nome fizer parte de uma subárvore inserida na mesma Transition, eles formam uma Shared Element Transition, e ela anima do excluído para o inserido.

Por padrão, `<ViewTransition>` anima com um cross-fade suave (a transição de visualização padrão do navegador). Você pode personalizar a animação fornecendo uma [View Transition Class](#view-transition-class) ao componente `<ViewTransition>`. Você pode personalizar animações para cada tipo de gatilho (consulte [Estilizando View Transitions](#styling-view-transitions)).

<DeepDive>

#### Como `<ViewTransition>` funciona? {/*how-does-viewtransition-work*/}

Por baixo dos panos, o React aplica `view-transition-name` aos estilos inline do nó DOM mais próximo aninhado dentro do componente `<ViewTransition>`. Se houver vários nós DOM irmãos como `<ViewTransition><div /><div /></ViewTransition>`, o React adiciona um sufixo ao nome para tornar cada um único, mas conceitualmente eles fazem parte do mesmo. O React não os aplica ansiosamente, mas apenas no momento em que esse limite deve participar de uma animação.

O React chama automaticamente `startViewTransition` por trás das cenas, então você nunca deve fazer isso sozinho. Na verdade, se você tiver outra coisa na página executando um ViewTransition, o React o interromperá. Portanto, é recomendável que você use o próprio React para coordená-los. Se você tinha outras maneiras de acionar ViewTransitions no passado, recomendamos que você migre para a maneira integrada.

Se houver outras React ViewTransitions já em execução, o React esperará que elas terminem antes de iniciar a próxima. No entanto, é importante ressaltar que, se houver várias atualizações ocorrendo enquanto a primeira estiver em execução, todas elas serão agrupadas em uma. Se você iniciar A->B. Então, enquanto isso, você recebe uma atualização para ir para C e depois D. Quando a primeira animação A->B terminar, a próxima animará de B->D.

O ciclo de vida `getSnapshotBeforeUpdate` será chamado antes de `startViewTransition` e algum `view-transition-name` será atualizado ao mesmo tempo.

Então, o React chama `startViewTransition`. Dentro do `updateCallback`, o React irá:

- Aplicar suas mutações ao DOM e invocar useInsertionEffects.
- Aguardar o carregamento das fontes.
- Chamar componentDidMount, componentDidUpdate, useLayoutEffect e refs.
- Aguardar a conclusão de qualquer Navegação pendente.
- Então, o React medirá quaisquer alterações no layout para ver quais limites precisarão animar.

Depois que a Promise pronta do `startViewTransition` for resolvida, o React reverterá o `view-transition-name`. Então, o React invocará os callbacks `onEnter`, `onExit`, `onUpdate` e `onShare` para permitir o controle programático manual sobre as Animações. Isso será depois que as animações padrão integradas já tiverem sido computadas.

Se um `flushSync` acontecer no meio dessa sequência, o React pulará a Transition, pois ela depende da capacidade de ser concluída de forma síncrona.

Depois que a Promise finalizada do `startViewTransition` for resolvida, o React invocará `useEffect`. Isso impede que eles interfiram no desempenho da Animação. No entanto, isso não é uma garantia, porque se outro `setState` acontecer enquanto a Animação estiver em execução, ele ainda terá que invocar o `useEffect` mais cedo para preservar as garantias sequenciais.

</DeepDive>

#### Props {/*props*/}

Por padrão, `<ViewTransition>` anima com um cross-fade suave. Você pode personalizar a animação ou especificar uma transição de elemento compartilhado com estas props:

* **opcional** `enter`: Uma string ou objeto. A [View Transition Class](#view-transition-class) a ser aplicada quando a entrada for ativada.
* **opcional** `exit`: Uma string ou objeto. A [View Transition Class](#view-transition-class) a ser aplicada quando a saída for ativada.
* **opcional** `update`: Uma string ou objeto. A [View Transition Class](#view-transition-class) a ser aplicada quando uma atualização for ativada.
* **opcional** `share`: Uma string ou objeto. A [View Transition Class](#view-transition-class) a ser aplicada quando um elemento compartilhado for ativado.
* **opcional** `default`: Uma string ou objeto. A [View Transition Class](#view-transition-class) usada quando nenhuma outra prop de ativação correspondente é encontrada.
* **opcional** `name`: Uma string ou objeto. O nome da View Transition usado para transições de elementos compartilhados. Se não for fornecido, o React usará um nome exclusivo para cada View Transition para evitar animações inesperadas.

#### Callback {/*events*/}

Esses callbacks permitem que você ajuste a animação de forma imperativa usando as APIs [animate](https://developer.mozilla.org/en-US/docs/Web/API/Element/animate):

* **opcional** `onEnter`: Uma função. O React chama `onEnter` após uma animação de "entrada".
* **opcional** `onExit`: Uma função. O React chama `onExit` após uma animação de "saída".
* **opcional** `onShare`: Uma função. O React chama `onShare` após uma animação de "compartilhamento".
* **opcional** `onUpdate`: Uma função. O React chama `onUpdate` após uma animação de "atualização".

Cada callback recebe como argumentos:
- `element`: O elemento DOM que foi animado.
- `types`: Os [Tipos de Transição](/reference/react/addTransitionType) incluídos na animação.

### View Transition Class {/*view-transition-class*/}

A View Transition Class é o(s) nome(s) da classe CSS aplicado(s) pelo React durante a transição quando o ViewTransition é ativado. Pode ser uma string ou um objeto.
- `string`: a `class` adicionada aos elementos filhos quando ativada. Se `'none'` for fornecido, nenhuma classe será adicionada.
- `object`: a classe adicionada aos elementos filhos será a chave correspondente ao tipo de View Transition adicionado com `addTransitionType`. O objeto também pode especificar um `default` para usar se nenhum tipo correspondente for encontrado.

O valor `'none'` pode ser usado para impedir que uma View Transition seja ativada para um gatilho específico.

### Estilizando View Transitions {/*styling-view-transitions*/}

<Note>

Em muitos exemplos iniciais de View Transitions pela web, você terá visto o uso de um [`view-transition-name`](https://developer.mozilla.org/en-US/docs/Web/CSS/view-transition-name) e, em seguida, estilizado-o usando seletores `::view-transition-...(my-name)`. Não recomendamos isso para estilização. Em vez disso, normalmente recomendamos o uso de uma View Transition Class.

</Note>

Para personalizar a animação de um `<ViewTransition>`, você pode fornecer uma View Transition Class a uma das props de ativação. A View Transition Class é um nome de classe CSS que o React aplica aos elementos filhos quando o ViewTransition é ativado.

Por exemplo, para personalizar uma animação de "entrada", forneça um nome de classe à prop `enter`:

```js
<ViewTransition enter="slide-in">
```

Quando o `<ViewTransition>` ativa uma animação de "entrada", o React adicionará o nome da classe `slide-in`. Então, você pode se referir a esta classe usando [seletores de pseudo-classe de transição de visualização](https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API#pseudo-elements) para construir animações reutilizáveis:

```css
::view-transition-group(.slide-in) {
  
}
::view-transition-old(.slide-in) {

}
::view-transition-new(.slide-in) {

}
```

No futuro, as bibliotecas CSS podem adicionar animações integradas usando View Transition Classes para tornar isso mais fácil de usar.

#### Ressalvas {/*caveats*/}

- Por padrão, as atualizações de `setState` são imediatas e não ativam `<ViewTransition>`, apenas atualizações encapsuladas em uma [Transition](/reference/react/useTransition). Você também pode usar [`<Suspense>`](/reference/react/Suspense) para optar por uma Transition para [revelar conteúdo](/reference/react/Suspense#revealing-content-together-at-once).
- `<ViewTransition>` cria uma imagem que pode ser movida, dimensionada e cross-faded. Ao contrário das Animações de Layout que você pode ter visto no React Native ou Motion, isso significa que nem todos os Elementos individuais dentro dele animam sua posição. Isso pode levar a um melhor desempenho e uma sensação mais contínua, animação suave em comparação com a animação de cada peça individual. No entanto, também pode perder a continuidade em coisas que devem estar se movendo sozinhas. Então, você pode ter que adicionar mais limites `<ViewTransition>` manualmente como resultado.
- Muitos usuários podem preferir não ter animações na página. O React não desativa automaticamente as animações para este caso. Recomendamos o uso da consulta de mídia `@media (prefers-reduced-motion)` para desativar as animações ou reduzi-las com base na preferência do usuário. No futuro, as bibliotecas CSS podem ter isso integrado em seus predefinições.
- Atualmente, `<ViewTransition>` só funciona no DOM. Estamos trabalhando para adicionar suporte para React Native e outras plataformas.

---



## Uso {/*uso*/}

### Animando um elemento ao entrar/sair {/*animando-um-elemento-ao-entrar*/}

As transições de entrada/saída são acionadas quando um `<ViewTransition>` é adicionado ou removido por um componente em uma transição:

```js
function Child() {
  return <ViewTransition>Oi</ViewTransition>
}

function Parent() {
  const [show, setShow] = useState();
  if (show) {
    return <Child />;
  }
  return null;
}
```

Quando `setShow` é chamado, `show` muda para `true` e o componente `Child` é renderizado. Quando `setShow` é chamado dentro de `startTransition`, e `Child` renderiza um `ViewTransition` antes de quaisquer outros nós DOM, uma animação de `entrada` é acionada.

Quando `show` muda de volta para `false`, uma animação de `saída` é acionada.

<Sandpack>

```js src/Video.js hidden
function Thumbnail({ video, children }) {
  return (
    <div
      aria-hidden="true"
      tabIndex={-1}
      className={`thumbnail ${video.image}`}
    />
  );
}

export function Video({ video }) {
  return (
    <div className="video">
      <div
        className="link"
      >
        <Thumbnail video={video}></Thumbnail>

        <div className="info">
          <div className="video-title">{video.title}</div>
          <div className="video-description">{video.description}</div>
        </div>
      </div>
    </div>
  );
}
```

```js
import {
  unstable_ViewTransition as ViewTransition,
  useState,
  startTransition
} from 'react';
import {Video} from "./Video";
import videos from "./data"

function Item() {
  return (
    <ViewTransition>
      <Video video={videos[0]}/>
    </ViewTransition>
  );
}

export default function Component() {
  const [showItem, setShowItem] = useState(false);
  return (
    <>
      <button
        onClick={() => {
          startTransition(() => {
            setShowItem((prev) => !prev);
          });
        }}
      >{showItem ? '➖' : '➕'}</button>

      {showItem ? <Item /> : null}
    </>
  );
}
```

```js src/data.js hidden
export default [
  {
    id: '1',
    title: 'First video',
    description: 'Video description',
    image: 'blue',
  }
]
```

```css
#root {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: 200px;
}
button {
  border: none;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f0f8ff;
  color: white;
  font-size: 20px;
  cursor: pointer;
  transition: background-color 0.3s, border 0.3s;
}
button:hover {
  border: 2px solid #ccc;
  background-color: #e0e8ff;
}
.thumbnail {
  position: relative;
  aspect-ratio: 16 / 9;
  display: flex;
  overflow: hidden;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 0.5rem;
  outline-offset: 2px;
  width: 8rem;
  vertical-align: middle;
  background-color: #ffffff;
  background-size: cover;
  user-select: none;
}
.thumbnail.blue {
  background-image: conic-gradient(at top right, #c76a15, #087ea4, #2b3491);
}
.video {
  display: flex;
  flex-direction: row;
  gap: 0.75rem;
  align-items: center;
  margin-top: 1em;
}
.video .link {
  display: flex;
  flex-direction: row;
  flex: 1 1 0;
  gap: 0.125rem;
  outline-offset: 4px;
  cursor: pointer;
}
.video .info {
  display: flex;
  flex-direction: column;
  justify-content: center;
  margin-left: 8px;
  gap: 0.125rem;
}
.video .info:hover {
  text-decoration: underline;
}
.video-title {
  font-size: 15px;
  line-height: 1.25;
  font-weight: 700;
  color: #23272f;
}
.video-description {
  color: #5e687e;
  font-size: 13px;
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest"
  }
}
```

</Sandpack>

<Pitfall>

`<ViewTransition>` só ativa se for colocado antes de qualquer nó DOM. Se `Child` em vez disso se parecesse com isto, nenhuma animação seria acionada:

```js [3, 5]
function Component() {
  return (
    <div>
      <ViewTransition>Oi</ViewTransition>
    </div>
  );
}
```

</Pitfall>

---
### Animando um elemento compartilhado {/*animando-um-elemento-compartilhado*/}

Normalmente, não recomendamos atribuir um nome a um `<ViewTransition>` e, em vez disso, deixar o React atribuir um nome automático. A razão pela qual você pode querer atribuir um nome é para animar entre componentes completamente diferentes quando uma árvore é desmontada e outra árvore é montada ao mesmo tempo. Para preservar a continuidade.

```js
<ViewTransition name={UNIQUE_NAME}>
  <Child />
</ViewTransition>
```

Quando uma árvore é desmontada e outra é montada, se houver um par em que o mesmo nome exista na árvore de desmontagem e na árvore de montagem, eles acionam a animação "compartilhar" em ambos. Ele anima do lado da desmontagem para o lado da montagem.

Ao contrário de uma animação de saída/entrada, isso pode estar profundamente dentro da árvore excluída/montada. Se um `<ViewTransition>` também for elegível para saída/entrada, a animação "compartilhar" terá precedência.

Se a Transição primeiro desmontar um lado e, em seguida, levar a um fallback `<Suspense>` sendo mostrado antes que, eventualmente, o novo nome seja montado, então nenhuma transição de elemento compartilhado acontece.

<Sandpack>

```js
import {
  unstable_ViewTransition as ViewTransition,
  useState,
  startTransition
} from "react";
import {Video, Thumbnail, FullscreenVideo} from "./Video";
import videos from "./data";

export default function Component() {
  const [fullscreen, setFullscreen] = useState(false);
  if (fullscreen) {
    return <FullscreenVideo
      video={videos[0]}
      onExit={() => startTransition(() => setFullscreen(false))}
    />
  }
  return <Video
    video={videos[0]}
    onClick={() => startTransition(() => setFullscreen(true))}
  />
}

```

```js src/Video.js
import {unstable_ViewTransition as ViewTransition} from "react";

const THUMBNAIL_NAME = "video-thumbnail"

export function Thumbnail({ video, children }) {
  return (
    <ViewTransition name={THUMBNAIL_NAME}>
      <div
        aria-hidden="true"
        tabIndex={-1}
        className={`thumbnail ${video.image}`}
      />
    </ViewTransition>
  );
}

export function Video({ video, onClick }) {
  return (
    <div className="video">
      <div className="link" onClick={onClick}>
        <Thumbnail video={video} />
        <div className="info">
          <div className="video-title">{video.title}</div>
          <div className="video-description">{video.description}</div>
        </div>
      </div>
    </div>
  );
}

export function FullscreenVideo({video, onExit}) {
  return (
    <div className="fullscreenLayout">
      <ViewTransition name={THUMBNAIL_NAME}>
        <div
          aria-hidden="true"
          tabIndex={-1}
          className={`thumbnail ${video.image} fullscreen`}
        />
        <button
          className="close-button"
          onClick={onExit}
        >
          ✖
        </button>
      </ViewTransition>
    </div>
  );
}
```

```js src/data.js hidden
export default [
  {
    id: '1',
    title: 'First video',
    description: 'Video description',
    image: 'blue',
  }
]
```

```css
#root {
  display: flex;
  flex-direction: column;
  align-items: center;
  height: 300px;
}
button {
  border: none;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f0f8ff;
  color: white;
  font-size: 20px;
  cursor: pointer;
  transition: background-color 0.3s, border 0.3s;
}
button:hover {
  border: 2px solid #ccc;
  background-color: #e0e8ff;
}
.thumbnail {
  position: relative;
  aspect-ratio: 16 / 9;
  display: flex;
  overflow: hidden;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 0.5rem;
  outline-offset: 2px;
  width: 8rem;
  vertical-align: middle;
  background-color: #ffffff;
  background-size: cover;
  user-select: none;
}
.thumbnail.blue {
  background-image: conic-gradient(at top right, #c76a15, #087ea4, #2b3491);
}
.thumbnail.red {
  background-image: conic-gradient(at top right, #c76a15, #a6423a, #2b3491);
}
.thumbnail.fullscreen {
  height: 100%;
  width: 100%;
}
.video {
  display: flex;
  flex-direction: row;
  gap: 0.75rem;
  align-items: center;
  margin-top: 1em;
}
.video .link {
  display: flex;
  flex-direction: row;
  flex: 1 1 0;
  gap: 0.125rem;
  outline-offset: 4px;
  cursor: pointer;
}
.video .info {
  display: flex;
  flex-direction: column;
  justify-content: center;
  margin-left: 8px;
  gap: 0.125rem;
}
.video .info:hover {
  text-decoration: underline;
}
.video-title {
  font-size: 15px;
  line-height: 1.25;
  font-weight: 700;
  color: #23272f;
}
.video-description {
  color: #5e687e;
  font-size: 13px;
}
.fullscreenLayout {
  position: relative;
  height: 100%;
  width: 100%;
}
.close-button {
  position: absolute;
  top: 10px;
  right: 10px;
  color: black;
}
@keyframes progress-animation {
  from {
    width: 0;
  }
  to {
    width: 100%;
  }
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest"
  }
}
```

</Sandpack>

<Note>

Se qualquer um dos lados montados ou desmontados de um par estiver fora da janela de visualização, nenhum par é formado. Isso garante que ele não entre ou saia da janela de visualização quando algo é rolado. Em vez disso, é tratado como uma entrada/saída regular por si só.

Isso não acontece se a mesma instância do Componente mudar de posição, o que aciona uma "atualização". Aqueles animam independentemente de uma posição estar fora da janela de visualização.

Atualmente, há uma peculiaridade em que, se um `<ViewTransition>` profundamente aninhado e desmontado estiver dentro da janela de visualização, mas o lado montado não estiver dentro da janela de visualização, o lado desmontado anima como sua própria animação de "saída", mesmo que esteja profundamente aninhado em vez de como parte da animação pai.

</Note>

<Pitfall>

É importante que haja apenas uma coisa com o mesmo nome montada por vez em todo o aplicativo. Portanto, é importante usar namespaces exclusivos para o nome para evitar conflitos. Para garantir que você possa fazer isso, convém adicionar uma constante em um módulo separado que você importa.

```js
export const MY_NAME = "my-globally-unique-name";
import {MY_NAME} from './shared-name';
...
<ViewTransition name={MY_NAME}>
```

</Pitfall>

---


### Animando a reordenação de itens em uma lista {/*animando-reorder-of-items-in-a-list*/}

```js
items.map(item => <Component key={item.id} item={item} />)
```

Ao reordenar uma lista, sem atualizar o conteúdo, a animação de "atualização" é acionada em cada `<ViewTransition>` na lista, se eles estiverem fora de um nó DOM. Semelhante às animações de entrada/saída.

Isso significa que isso acionará a animação neste `<ViewTransition>`:

```js
function Component() {
  return <ViewTransition><div>...</div></ViewTransition>;
}
```

<Sandpack>

```js src/Video.js hidden
function Thumbnail({ video }) {
  return (
    <div
      aria-hidden="true"
      tabIndex={-1}
      className={`thumbnail ${video.image}`}
    />
  );
}

export function Video({ video }) {
  return (
    <div className="video">
      <div className="link">
        <Thumbnail video={video}></Thumbnail>
        <div className="info">
          <div className="video-title">{video.title}</div>
          <div className="video-description">{video.description}</div>
        </div>
      </div>
    </div>
  );
}
```

```js
import {
  unstable_ViewTransition as ViewTransition,
  useState,
  startTransition
} from "react";
import {Video} from "./Video";
import videos from "./data";

export default function Component() {
  const [orderedVideos, setOrderedVideos] = useState(videos);
  const reorder = () => {
    startTransition(() => {
      setOrderedVideos((prev) => {
        return [...prev.sort(() => Math.random() - 0.5)];
      });
    });
  };
  return (
    <>
      <button onClick={reorder}>🎲</button>
      <div className="listContainer">
        {orderedVideos.map((video, i) => {
          return (
            <ViewTransition key={video.title}>
              <Video video={video} />
            </ViewTransition>
          );
        })}
      </div>
    </>
  );
}
  

```

```js src/data.js hidden
export default [
  {
    id: '1',
    title: 'First video',
    description: 'Video description',
    image: 'blue',
  },
  {
    id: '2',
    title: 'Second video',
    description: 'Video description',
    image: 'red',
  },
  {
    id: '3',
    title: 'Third video',
    description: 'Video description',
    image: 'green',
  },
  {
    id: '4',
    title: 'Fourth video',
    description: 'Video description',
    image: 'purple',
  }
]
```

```css
#root {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: 150px;
}
button {
  border: none;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f0f8ff;
  color: white;
  font-size: 20px;
  cursor: pointer;
  transition: background-color 0.3s, border 0.3s;
}
button:hover {
  border: 2px solid #ccc;
  background-color: #e0e8ff;
}
.thumbnail {
  position: relative;
  aspect-ratio: 16 / 9;
  display: flex;
  overflow: hidden;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 0.5rem;
  outline-offset: 2px;
  width: 8rem;
  vertical-align: middle;
  background-color: #ffffff;
  background-size: cover;
  user-select: none;
}
.thumbnail.blue {
  background-image: conic-gradient(at top right, #c76a15, #087ea4, #2b3491);
}
.thumbnail.red {
  background-image: conic-gradient(at top right, #c76a15, #a6423a, #2b3491);
}
.thumbnail.green {
  background-image: conic-gradient(at top right, #c76a15, #388f7f, #2b3491);
}
.thumbnail.purple {
  background-image: conic-gradient(at top right, #c76a15, #575fb7, #2b3491);
}
.video {
  display: flex;
  flex-direction: row;
  gap: 0.75rem;
  align-items: center;
  margin-top: 1em;
}
.video .link {
  display: flex;
  flex-direction: row;
  flex: 1 1 0;
  gap: 0.125rem;
  outline-offset: 4px;
}
.video .info {
  display: flex;
  flex-direction: column;
  justify-content: center;
  margin-left: 8px;
  gap: 0.125rem;
}
.video .info:hover {
  text-decoration: underline;
}
.video-title {
  font-size: 15px;
  line-height: 1.25;
  font-weight: 700;
  color: #23272f;
}
.video-description {
  color: #5e687e;
  font-size: 13px;
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest"
  }
}
```

</Sandpack>

No entanto, isso não animaria cada item individualmente:

```js
function Component() {
  return <div><ViewTransition>...</ViewTransition></div>;
}
```

Em vez disso, qualquer `<ViewTransition>` pai faria um cross-fade. Se não houver nenhum `<ViewTransition>` pai, então não haverá animação nesse caso.

<Sandpack>

```js src/Video.js hidden
function Thumbnail({ video }) {
  return (
    <div
      aria-hidden="true"
      tabIndex={-1}
      className={`thumbnail ${video.image}`}
    />
  );
}

export function Video({ video }) {
  return (
    <div className="video">
      <div className="link">
        <Thumbnail video={video}></Thumbnail>
        <div className="info">
          <div className="video-title">{video.title}</div>
          <div className="video-description">{video.description}</div>
        </div>
      </div>
    </div>
  );
}
```

```js
import {
  unstable_ViewTransition as ViewTransition,
  useState,
  startTransition
} from "react";
import {Video} from "./Video";
import videos from "./data";

export default function Component() {
  const [orderedVideos, setOrderedVideos] = useState(videos);
  const reorder = () => {
    startTransition(() => {
      setOrderedVideos((prev) => {
        return [...prev.sort(() => Math.random() - 0.5)];
      });
    });
  };
  return (
    <>
      <button onClick={reorder}>🎲</button>
      <ViewTransition>
        <div className="listContainer">
          {orderedVideos.map((video, i) => {
            return <Video video={video} key={video.title} />;
          })}
        </div>
      </ViewTransition>
    </>
  );
}
  

```

```js src/data.js hidden
export default [
  {
    id: '1',
    title: 'First video',
    description: 'Video description',
    image: 'blue',
  },
  {
    id: '2',
    title: 'Second video',
    description: 'Video description',
    image: 'red',
  },
  {
    id: '3',
    title: 'Third video',
    description: 'Video description',
    image: 'green',
  },
  {
    id: '4',
    title: 'Fourth video',
    description: 'Video description',
    image: 'purple',
  }
]
```

```css
#root {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: 150px;
}
button {
  border: none;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f0f8ff;
  color: white;
  font-size: 20px;
  cursor: pointer;
  transition: background-color 0.3s, border 0.3s;
}
button:hover {
  border: 2px solid #ccc;
  background-color: #e0e8ff;
}
.thumbnail {
  position: relative;
  aspect-ratio: 16 / 9;
  display: flex;
  overflow: hidden;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 0.5rem;
  outline-offset: 2px;
  width: 8rem;
  vertical-align: middle;
  background-color: #ffffff;
  background-size: cover;
  user-select: none;
}
.thumbnail.blue {
  background-image: conic-gradient(at top right, #c76a15, #087ea4, #2b3491);
}
.thumbnail.red {
  background-image: conic-gradient(at top right, #c76a15, #a6423a, #2b3491);
}
.thumbnail.green {
  background-image: conic-gradient(at top right, #c76a15, #388f7f, #2b3491);
}
.thumbnail.purple {
  background-image: conic-gradient(at top right, #c76a15, #575fb7, #2b3491);
}
.video {
  display: flex;
  flex-direction: row;
  gap: 0.75rem;
  align-items: center;
  margin-top: 1em;
}
.video .link {
  display: flex;
  flex-direction: row;
  flex: 1 1 0;
  gap: 0.125rem;
  outline-offset: 4px;
}
.video .info {
  display: flex;
  flex-direction: column;
  justify-content: center;
  margin-left: 8px;
  gap: 0.125rem;
}
.video .info:hover {
  text-decoration: underline;
}
.video-title {
  font-size: 15px;
  line-height: 1.25;
  font-weight: 700;
  color: #23272f;
}
.video-description {
  color: #5e687e;
  font-size: 13px;
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest"
  }
}
```

</Sandpack>

Isso significa que você pode querer evitar elementos wrapper em listas onde você deseja permitir que o Componente controle sua própria animação de reordenação:

```
items.map(item => <div><Component key={item.id} item={item} /></div>)
```

A regra acima também se aplica se um dos itens for atualizado para redimensionar, o que então faz com que os irmãos sejam redimensionados, ele também animará seu irmão `<ViewTransition>`, mas somente se forem irmãos imediatos.

Isso significa que, durante uma atualização, que causa muito re-layout, ele não anima individualmente cada `<ViewTransition>` na página. Isso levaria a muitas animações ruidosas que distraem da mudança real. Portanto, o React é mais conservador sobre quando uma animação individual é acionada.

<Pitfall>

É importante usar chaves corretamente para preservar a identidade ao reordenar listas. Pode parecer que você pode usar "name", transições de elementos compartilhados, para animar reordenações, mas isso não seria acionado se um lado estivesse fora da viewport. Para animar uma reordenação, você geralmente deseja mostrar que ela foi para uma posição fora da viewport.

</Pitfall>

---


### Animando a partir do conteúdo do Suspense {/*animating-from-suspense-content*/}

Assim como qualquer Transição, o React aguarda dados e novos CSS (`<link rel="stylesheet" precedence="...">`) antes de executar a animação. Além disso, as ViewTransitions também aguardam até 500ms para que novas fontes carreguem antes de iniciar a animação, a fim de evitar que elas pisquem mais tarde. Pela mesma razão, uma imagem encapsulada em ViewTransition aguardará o carregamento da imagem.

Se estiver dentro de uma nova instância de limite Suspense, o fallback será exibido primeiro. Após o carregamento completo do limite Suspense, ele aciona o `<ViewTransition>` para animar a revelação do conteúdo.

Atualmente, isso só acontece para a Transição do lado do cliente. No futuro, isso também animará o limite Suspense para streaming SSR quando o conteúdo do servidor suspender durante o carregamento inicial.

Há duas maneiras de animar os limites Suspense, dependendo de onde você coloca o `<ViewTransition>`:

Atualização:

```
<ViewTransition>
  <Suspense fallback={<A />}>
    <B />
  </Suspense>
</ViewTransition>
```

Nesse cenário, quando o conteúdo vai de A para B, ele será tratado como uma "atualização" e aplicará essa classe, se apropriado. Tanto A quanto B receberão o mesmo view-transition-name e, portanto, estão agindo como um cross-fade por padrão.

<Sandpack>

```js src/Video.js hidden
function Thumbnail({ video, children }) {
  return (
    <div
      aria-hidden="true"
      tabIndex={-1}
      className={`thumbnail ${video.image}`}
    />
  );
}

export function Video({ video }) {
  return (
    <div className="video">
      <div className="link">
        <Thumbnail video={video}></Thumbnail>
        <div className="info">
          <div className="video-title">{video.title}</div>
          <div className="video-description">{video.description}</div>
        </div>
      </div>
    </div>
  );
}

export function VideoPlaceholder() {
  const video = {image: "loading"}
  return (
    <div className="video">
      <div className="link">
        <Thumbnail video={video}></Thumbnail>
        <div className="info">
          <div className="video-title loading" />
          <div className="video-description loading" />
        </div>
      </div>
    </div>
  );
}
```

```js
import {
  unstable_ViewTransition as ViewTransition,
  useState,
  startTransition,
  Suspense
} from 'react';
import {Video, VideoPlaceholder} from "./Video";
import {useLazyVideoData} from "./data"

function LazyVideo() {
  const video = useLazyVideoData();
  return (
    <Video video={video}/>
  );
}

export default function Component() {
  const [showItem, setShowItem] = useState(false);
  return (
    <>
      <button
        onClick={() => {
          startTransition(() => {
            setShowItem((prev) => !prev);
          });
        }}
      >{showItem ? '➖' : '➕'}</button>
      {showItem ? (
        <ViewTransition>
          <Suspense fallback={<VideoPlaceholder />}>
            <LazyVideo />
          </Suspense>
        </ViewTransition>
      ) : null}
    </>
  );
}
```

```js src/data.js hidden
import {use} from "react";

let cache = null;

function fetchVideo() {
  if (!cache) {
    cache = new Promise((resolve) => {
      setTimeout(() => {
        resolve({
          id: '1',
          title: 'First video',
          description: 'Video description',
          image: 'blue',
        });
      }, 1000);
    });
  }
  return cache;
}

export function useLazyVideoData() {
  return use(fetchVideo());
}
```

```css
#root {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: 200px;
}
button {
  border: none;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f0f8ff;
  color: white;
  font-size: 20px;
  cursor: pointer;
  transition: background-color 0.3s, border 0.3s;
}
button:hover {
  border: 2px solid #ccc;
  background-color: #e0e8ff;
}
.thumbnail {
  position: relative;
  aspect-ratio: 16 / 9;
  display: flex;
  overflow: hidden;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 0.5rem;
  outline-offset: 2px;
  width: 8rem;
  vertical-align: middle;
  background-color: #ffffff;
  background-size: cover;
  user-select: none;
}
.thumbnail.blue {
  background-image: conic-gradient(at top right, #c76a15, #087ea4, #2b3491);
}
.loading {
  background-image: linear-gradient(90deg, rgba(173, 216, 230, 0.3) 25%, rgba(135, 206, 250, 0.5) 50%, rgba(173, 216, 230, 0.3) 75%);
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
}
@keyframes shimmer {
  0% {
    background-position: -200% 0;
  }
  100% {
    background-position: 200% 0;
  }
}
.video {
  display: flex;
  flex-direction: row;
  gap: 0.75rem;
  align-items: center;
  margin-top: 1em;
}
.video .link {
  display: flex;
  flex-direction: row;
  flex: 1 1 0;
  gap: 0.125rem;
  outline-offset: 4px;
  cursor: pointer;
}
.video .info {
  display: flex;
  flex-direction: column;
  justify-content: center;
  margin-left: 8px;
  gap: 0.125rem;
}
.video .info:hover {
  text-decoration: underline;
}
.video-title {
  font-size: 15px;
  line-height: 1.25;
  font-weight: 700;
  color: #23272f;
}
.video-title.loading {
  height: 20px;
  width: 80px;
  border-radius: 0.5rem;
}
.video-description {
  color: #5e687e;
  font-size: 13px;
  border-radius: 0.5rem;
}
.video-description.loading {
  height: 15px;
  width: 100px;
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest"
  }
}
```

</Sandpack>

Entrada/Saída:

```
<Suspense fallback={<ViewTransition><A /></ViewTransition>}>
  <ViewTransition><B /></ViewTransition>
</Suspense>
```

Nesse cenário, essas são duas instâncias separadas de ViewTransition, cada uma com seu próprio `view-transition-name`. Isso será tratado como uma "saída" de `<A>` e uma "entrada" de `<B>`.

Você pode obter efeitos diferentes dependendo de onde você escolher colocar o limite `<ViewTransition>`.

---
### Desativando uma animação {/*opting-out-of-an-animation*/}

Às vezes, você está encapsulando um componente existente grande, como uma página inteira, e deseja animar algumas atualizações, como alterar o tema. No entanto, você não quer que ele aceite todas as atualizações dentro da página inteira para cross-fade quando estiverem atualizando. Especialmente se você estiver adicionando mais animações incrementalmente.

Você pode usar a classe "none" para desativar uma animação. Ao encapsular seus filhos em "none", você pode desabilitar as animações para atualizações neles, enquanto o pai ainda é acionado.

```js
<ViewTransition>
  <div className={theme}>
    <ViewTransition update="none">
      {children}
    </ViewTransition>
  </div>
</ViewTransition>
```

Isso só animará se o tema mudar e não se apenas os filhos forem atualizados. Os filhos ainda podem aceitar novamente com seu próprio `<ViewTransition>`, mas pelo menos é manual novamente.

---


### Personalizando animações {/*customizing-animations*/}

Por padrão, `<ViewTransition>` inclui o cross-fade padrão do navegador.

Para personalizar animações, você pode fornecer props para o componente `<ViewTransition>` para especificar quais animações usar, com base em como o `<ViewTransition>` é ativado.

Por exemplo, podemos diminuir a velocidade da animação de cross fade padrão:

```js
<ViewTransition default="slow-fade">
  <Video />
</ViewTransition>
```

E definir slow-fade em CSS usando classes de transição de visualização:

```css
::view-transition-old(.slow-fade) {
    animation-duration: 500ms;
}

::view-transition-new(.slow-fade) {
    animation-duration: 500ms;
}
```

<Sandpack>

```js src/Video.js hidden
function Thumbnail({ video, children }) {
  return (
    <div
      aria-hidden="true"
      tabIndex={-1}
      className={`thumbnail ${video.image}`}
    />
  );
}

export function Video({ video }) {
  return (
    <div className="video">
      <div
        className="link"
      >
        <Thumbnail video={video}></Thumbnail>

        <div className="info">
          <div className="video-title">{video.title}</div>
          <div className="video-description">{video.description}</div>
        </div>
      </div>
    </div>
  );
}
```

```js
import {
  unstable_ViewTransition as ViewTransition,
  useState,
  startTransition
} from 'react';
import {Video} from "./Video";
import videos from "./data"

function Item() {
  return (
    <ViewTransition default="slow-fade">
      <Video video={videos[0]}/>
    </ViewTransition>
  );
}

export default function Component() {
  const [showItem, setShowItem] = useState(false);
  return (
    <>
      <button
        onClick={() => {
          startTransition(() => {
            setShowItem((prev) => !prev);
          });
        }}
      >{showItem ? '➖' : '➕'}</button>

      {showItem ? <Item /> : null}
    </>
  );
}
```

```js src/data.js hidden
export default [
  {
    id: '1',
    title: 'First video',
    description: 'Video description',
    image: 'blue',
  }
]
```


```css
::view-transition-old(.slow-fade) {
    animation-duration: 500ms;
}

::view-transition-new(.slow-fade) {
    animation-duration: 500ms;
}

#root {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: 200px;
}
button {
  border: none;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f0f8ff;
  color: white;
  font-size: 20px;
  cursor: pointer;
  transition: background-color 0.3s, border 0.3s;
}
button:hover {
  border: 2px solid #ccc;
  background-color: #e0e8ff;
}
.thumbnail {
  position: relative;
  aspect-ratio: 16 / 9;
  display: flex;
  overflow: hidden;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 0.5rem;
  outline-offset: 2px;
  width: 8rem;
  vertical-align: middle;
  background-color: #ffffff;
  background-size: cover;
  user-select: none;
}
.thumbnail.blue {
  background-image: conic-gradient(at top right, #c76a15, #087ea4, #2b3491);
}
.video {
  display: flex;
  flex-direction: row;
  gap: 0.75rem;
  align-items: center;
  margin-top: 1em;
}
.video .link {
  display: flex;
  flex-direction: row;
  flex: 1 1 0;
  gap: 0.125rem;
  outline-offset: 4px;
  cursor: pointer;
}
.video .info {
  display: flex;
  flex-direction: column;
  justify-content: center;
  margin-left: 8px;
  gap: 0.125rem;
}
.video .info:hover {
  text-decoration: underline;
}
.video-title {
  font-size: 15px;
  line-height: 1.25;
  font-weight: 700;
  color: #23272f;
}
.video-description {
  color: #5e687e;
  font-size: 13px;
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest"
  }
}
```

</Sandpack>

Além de definir o `default`, você também pode fornecer configurações para as animações `enter`, `exit`, `update` e `share`.

<Sandpack>

```js src/Video.js hidden
function Thumbnail({ video, children }) {
  return (
    <div
      aria-hidden="true"
      tabIndex={-1}
      className={`thumbnail ${video.image}`}
    />
  );
}

export function Video({ video }) {
  return (
    <div className="video">
      <div
        className="link"
      >
        <Thumbnail video={video}></Thumbnail>

        <div className="info">
          <div className="video-title">{video.title}</div>
          <div className="video-description">{video.description}</div>
        </div>
      </div>
    </div>
  );
}
```

```js
import {
  unstable_ViewTransition as ViewTransition,
  useState,
  startTransition
} from 'react';
import {Video} from "./Video";
import videos from "./data"

function Item() {
  return (
    <ViewTransition enter="slide-in" exit="slide-out">
      <Video video={videos[0]}/>
    </ViewTransition>
  );
}

export default function Component() {
  const [showItem, setShowItem] = useState(false);
  return (
    <>
      <button
        onClick={() => {
          startTransition(() => {
            setShowItem((prev) => !prev);
          });
        }}
      >{showItem ? '➖' : '➕'}</button>

      {showItem ? <Item /> : null}
    </>
  );
}
```

```js src/data.js hidden
export default [
  {
    id: '1',
    title: 'First video',
    description: 'Video description',
    image: 'blue',
  }
]
```


```css
::view-transition-old(.slide-in) {
  animation-name: slideOutRight;
  animation-duration: 500ms;
  animation-timing-function: ease-in-out;
}

::view-transition-new(.slide-in) {
  animation-name: slideInRight;
  animation-duration: 500ms;
  animation-timing-function: ease-in-out;
}

::view-transition-old(.slide-out) {
  animation-name: slideOutLeft;
  animation-duration: 500ms;
  animation-timing-function: ease-in-out;
}

::view-transition-new(.slide-out) {
  animation-name: slideInLeft;
  animation-duration: 500ms;
  animation-timing-function: ease-in-out;
}

@keyframes slideOutLeft {
  from {
    transform: translateX(0);
    opacity: 1;
  }
  to {
    transform: translateX(-100%);
    opacity: 0;
  }
}

@keyframes slideInLeft {
  from {
    transform: translateX(-100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

@keyframes slideOutRight {
  from {
    transform: translateX(0);
    opacity: 1;
  }
  to {
    transform: translateX(100%);
    opacity: 0;
  }
}

@keyframes slideInRight {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

@keyframes slideInRight {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

#root {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: 200px;
}
button {
  border: none;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f0f8ff;
  color: white;
  font-size: 20px;
  cursor: pointer;
  transition: background-color 0.3s, border 0.3s;
}
button:hover {
  border: 2px solid #ccc;
  background-color: #e0e8ff;
}
.thumbnail {
  position: relative;
  aspect-ratio: 16 / 9;
  display: flex;
  overflow: hidden;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 0.5rem;
  outline-offset: 2px;
  width: 8rem;
  vertical-align: middle;
  background-color: #ffffff;
  background-size: cover;
  user-select: none;
}
.thumbnail.blue {
  background-image: conic-gradient(at top right, #c76a15, #087ea4, #2b3491);
}
.video {
  display: flex;
  flex-direction: row;
  gap: 0.75rem;
  align-items: center;
  margin-top: 1em;
}
.video .link {
  display: flex;
  flex-direction: row;
  flex: 1 1 0;
  gap: 0.125rem;
  outline-offset: 4px;
  cursor: pointer;
}
.video .info {
  display: flex;
  flex-direction: column;
  justify-content: center;
  margin-left: 8px;
  gap: 0.125rem;
}
.video .info:hover {
  text-decoration: underline;
}
.video-title {
  font-size: 15px;
  line-height: 1.25;
  font-weight: 700;
  color: #23272f;
}
.video-description {
  color: #5e687e;
  font-size: 13px;
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest"
  }
}
```

</Sandpack>


### Personalizando animações com tipos {/*customizing-animations-with-types*/}

Você pode usar a API [`addTransitionType`](/reference/react/addTransitionType) para adicionar um nome de classe aos elementos filhos quando um tipo de transição específico é ativado para um gatilho de ativação específico. Isso permite que você personalize a animação para cada tipo de transição.

Por exemplo, para personalizar a animação para todas as navegações para frente e para trás:

```js
<ViewTransition default={{
  'navigation-back': 'slide-right',
  'navigation-forward': 'slide-left',
 }}>
  <div>...</div>
</ViewTransition>
 
// no seu roteador:
startTransition(() => {
  addTransitionType('navigation-' + navigationType);
});
```

Quando o ViewTransition ativa uma animação "navigation-back", o React adicionará o nome da classe "slide-right". Quando o ViewTransition ativa uma animação "navigation-forward", o React adicionará o nome da classe "slide-left".

No futuro, roteadores e outras bibliotecas podem adicionar suporte para tipos e estilos de transição de visualização padrão.

<Sandpack>

```js src/Video.js hidden
function Thumbnail({ video, children }) {
  return (
    <div
      aria-hidden="true"
      tabIndex={-1}
      className={`thumbnail ${video.image}`}
    />
  );
}

export function Video({ video }) {
  return (
    <div className="video">
      <div
        className="link"
      >
        <Thumbnail video={video}></Thumbnail>
        <div className="info">
          <div className="video-title">{video.title}</div>
          <div className="video-description">{video.description}</div>
        </div>
      </div>
    </div>
  );
}
```

```js
import {
  unstable_ViewTransition as ViewTransition,
  unstable_addTransitionType as addTransitionType,
  useState,
  startTransition,
} from "react";
import {Video} from "./Video";
import videos from "./data"

function Item() {
  return (
    <ViewTransition enter={
        {
          "add-video-back": "slide-in-back",
          "add-video-forward": "slide-in-forward"
        }
      }
      exit={
        {
          "remove-video-back": "slide-in-forward",
          "remove-video-forward": "slide-in-back"
        }
      }>
      <Video video={videos[0]}/>
    </ViewTransition>
  );
}

export default function Component() {
  const [showItem, setShowItem] = useState(false);
  return (
    <>
      <div className="button-container">
        <button
          onClick={() => {
            startTransition(() => {
              if (showItem) {
                addTransitionType("remove-video-back")
              } else {
                addTransitionType("add-video-back")
              }
              setShowItem((prev) => !prev);
            });
          }}
        >⬅️</button>
        <button
          onClick={() => {
            startTransition(() => {
              if (showItem) {
                addTransitionType("remove-video-forward")
              } else {
                addTransitionType("add-video-forward")
              }
              setShowItem((prev) => !prev);
            });
          }}
        >➡️</button>
      </div>
      {showItem ? <Item /> : null}
    </>
  );
}
```

```js src/data.js hidden
export default [
  {
    id: '1',
    title: 'First video',
    description: 'Video description',
    image: 'blue',
  }
]
```


```css
::view-transition-old(.slide-in-back) {
  animation-name: slideOutRight;
  animation-duration: 500ms;
  animation-timing-function: ease-in-out;
}

::view-transition-new(.slide-in-back) {
  animation-name: slideInRight;
  animation-duration: 500ms;
  animation-timing-function: ease-in-out;
}

::view-transition-old(.slide-out-back) {
  animation-name: slideOutLeft;
  animation-duration: 500ms;
  animation-timing-function: ease-in-out;
}

::view-transition-new(.slide-out-back) {
  animation-name: slideInLeft;
  animation-duration: 500ms;
  animation-timing-function: ease-in-out;
}

::view-transition-old(.slide-in-forward) {
  animation-name: slideOutLeft;
  animation-duration: 500ms;
  animation-timing-function: ease-in-out;
}

::view-transition-new(.slide-in-forward) {
  animation-name: slideInLeft;
  animation-duration: 500ms;
  animation-timing-function: ease-in-out;
}

::view-transition-old(.slide-out-forward) {
  animation-name: slideOutRight;
  animation-duration: 500ms;
  animation-timing-function: ease-in-out;
}

::view-transition-new(.slide-out-forward) {
  animation-name: slideInRight;
  animation-duration: 500ms;
  animation-timing-function: ease-in-out;
}

@keyframes slideOutLeft {
  from {
    transform: translateX(0);
    opacity: 1;
  }
  to {
    transform: translateX(-100%);
    opacity: 0;
  }
}

@keyframes slideInLeft {
  from {
    transform: translateX(-100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

@keyframes slideOutRight {
  from {
    transform: translateX(0);
    opacity: 1;
  }
  to {
    transform: translateX(100%);
    opacity: 0;
  }
}

@keyframes slideInRight {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

@keyframes slideInRight {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

#root {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: 200px;
}
button {
  border: none;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f0f8ff;
  color: white;
  font-size: 20px;
  cursor: pointer;
  transition: background-color 0.3s, border 0.3s;
}
button:hover {
  border: 2px solid #ccc;
  background-color: #e0e8ff;
}
.button-container {
  display: flex;
}
.thumbnail {
  position: relative;
  aspect-ratio: 16 / 9;
  display: flex;
  overflow: hidden;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 0.5rem;
  outline-offset: 2px;
  width: 8rem;
  vertical-align: middle;
  background-color: #ffffff;
  background-size: cover;
  user-select: none;
}
.thumbnail.blue {
  background-image: conic-gradient(at top right, #c76a15, #087ea4, #2b3491);
}
.video {
  display: flex;
  flex-direction: row;
  gap: 0.75rem;
  align-items: center;
  margin-top: 1em;
}
.video .link {
  display: flex;
  flex-direction: row;
  flex: 1 1 0;
  gap: 0.125rem;
  outline-offset: 4px;
  cursor: pointer;
}
.video .info {
  display: flex;
  flex-direction: column;
  justify-content: center;
  margin-left: 8px;
  gap: 0.125rem;
}
.video .info:hover {
  text-decoration: underline;
}
.video-title {
  font-size: 15px;
  line-height: 1.25;
  font-weight: 700;
  color: #23272f;
}
.video-description {
  color: #5e687e;
  font-size: 13px;
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest"
  }
}
```

</Sandpack>

### Construindo roteadores habilitados para View Transition {/*building-view-transition-enabled-routers*/}

O React aguarda a conclusão de qualquer Navegação pendente para garantir que a restauração da rolagem ocorra dentro da animação. Se a Navegação estiver bloqueada no React, seu roteador deve desbloquear em `useLayoutEffect`, pois `useEffect` levaria a um impasse.

Se um `startTransition` for iniciado a partir do evento popstate legado, como durante uma navegação "back", ele deve ser concluído de forma síncrona para garantir que a rolagem e a restauração do formulário funcionem corretamente. Isso está em conflito com a execução de uma animação View Transition. Portanto, o React ignorará as animações do popstate. Portanto, as animações não serão executadas para o botão voltar. Você pode corrigir isso atualizando seu roteador para usar a API de Navegação.

---


## Solução de problemas {/*troubleshooting*/}

### Meu `<ViewTransition>` não está ativando {/*my-viewtransition-is-not-activating*/}

`<ViewTransition>` só ativa se for colocado antes de qualquer nó DOM:

```js [3, 5]
function Component() {
  return (
    <div>
      <ViewTransition>Oi</ViewTransition>
    </div>
  );
}
```

Para corrigir, certifique-se de que o `<ViewTransition>` vem antes de quaisquer outros nós DOM:

```js [3, 5] 
function Component() {
  return (
    <ViewTransition>
      <div>Oi</div>
    </ViewTransition>
  );
}
```

### Estou recebendo o erro "There are two `<ViewTransition name=%s>` components with the same name mounted at the same time." {/*two-viewtransition-with-same-name*/}

Este erro ocorre quando dois componentes `<ViewTransition>` com o mesmo `name` são montados ao mesmo tempo:

```js [3]
function Item() {
  // 🚩 Todos os itens receberão o mesmo "name".
  return <ViewTransition name="item">...</ViewTransition>;
}

function ItemList({items}) {
  return (
    <>
      {item.map(item => <Item key={item.id} />)}
    </>
  );
}
```

Isso fará com que a Transição de Visualização gere um erro. Em desenvolvimento, o React detecta esse problema para expô-lo e registra dois erros:

<ConsoleBlockMulti>
<ConsoleLogLine level="error">

There are two `<ViewTransition name=%s>` components with the same name mounted at the same time. This is not supported and will cause View Transitions to error. Try to use a more unique name e.g. by using a namespace prefix and adding the id of an item to the name.
{'    '}at Item
{'    '}at ItemList

</ConsoleLogLine>

<ConsoleLogLine level="error">

The existing `<ViewTransition name=%s>` duplicate has this stack trace.
{'    '}at Item
{'    '}at ItemList

</ConsoleLogLine>
</ConsoleBlockMulti>

Para corrigir, certifique-se de que haja apenas um `<ViewTransition>` com o mesmo nome montado por vez em todo o aplicativo, garantindo que o `name` seja único ou adicionando um `id` ao nome:

```js [3]
function Item({id}) {
  // ✅ Todos os itens receberão o mesmo "name".
  return <ViewTransition name={`item-${id}`}>...</ViewTransition>;
}

function ItemList({items}) {
  return (
    <>
      {item.map(item => <Item key={item.id} item={item} />)}
    </>
  );
}
```