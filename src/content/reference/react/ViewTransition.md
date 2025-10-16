---
title: <ViewTransition>
version: experimental
---

<Experimental>

**Esta API é experimental e ainda não está disponível em uma versão estável do React.**

Você pode experimentá-la atualizando os pacotes do React para a versão experimental mais recente:

- `react@experimental`
- `react-dom@experimental`
- `eslint-plugin-react-hooks@experimental`

Versões experimentais do React podem conter erros. Não as utilize em produção.

</Experimental>

<Intro>

`<ViewTransition>` permite animar elementos que são atualizados dentro de uma Transição.


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

Envolva elementos em `<ViewTransition>` para animá-los quando eles são atualizados dentro de uma [Transição](/reference/react/useTransition). O React usa as seguintes heurísticas para determinar se uma Transição de Visualização é ativada para uma animação:

- `enter`: Se um `ViewTransition` é inserido nesta Transição, ele será ativado.
- `exit`: Se um `ViewTransition` é excluído nesta Transição, ele será ativado.
- `update`: Se um `ViewTransition` possui mutações no DOM dentro dele que o React está realizando (como uma mudança de prop) ou se o próprio limite do `ViewTransition` muda de tamanho ou posição devido a um irmão imediato. Se houver `ViewTransition` aninhados, a mutação se aplica a eles e não ao pai.
- `share`: Se um `ViewTransition` nomeado está dentro de uma subárvore excluída e outro `ViewTransition` nomeado com o mesmo nome faz parte de uma subárvore inserida na mesma Transição, eles formam uma Transição de Elemento Compartilhado e animam do excluído para o inserido.

Por padrão, `<ViewTransition>` anima com um fade cruzado suave (a transição de visualização padrão do navegador). Você pode personalizar a animação fornecendo uma [Classe de Transição de Visualização](#view-transition-class) ao componente `<ViewTransition>`. Você pode personalizar animações para cada tipo de gatilho (veja [Estilizando Transições de Visualização](#styling-view-transitions)).

<DeepDive>

#### Como funciona o `<ViewTransition>`? {/*how-does-viewtransition-work*/}

Por baixo dos panos, o React aplica `view-transition-name` aos estilos inline do nó DOM mais próximo aninhado dentro do componente `<ViewTransition>`. Se houver múltiplos nós DOM irmãos como `<ViewTransition><div /><div /></ViewTransition>`, o React adicionará um sufixo ao nome para tornar cada um único, mas conceitualmente eles fazem parte do mesmo. O React não aplica isso de forma antecipada, mas apenas no momento em que o limite deve participar de uma animação.

O React chama automaticamente `startViewTransition` nos bastidores, então você nunca deve fazer isso sozinho. De fato, se você tiver outra coisa na página executando uma ViewTransition, o React a interromperá. Portanto, é recomendado que você use o próprio React para coordenar essas transições. Se você tinha outras maneiras de acionar ViewTransitions no passado, recomendamos que migre para a maneira integrada.

Se outras ViewTransitions do React já estiverem em execução, o React esperará que elas terminem antes de iniciar a próxima. No entanto, é importante notar que se houver várias atualizações ocorrendo enquanto a primeira está em execução, todas elas serão agrupadas em uma única transição. Se você iniciar A->B. Então, nesse ínterim, você recebe uma atualização para ir para C e depois para D. Quando a primeira animação A->B terminar, a próxima animará de B->D.

O ciclo de vida `getSnapshotBeforeUpdate` será chamado antes de `startViewTransition` e alguns `view-transition-name` serão atualizados ao mesmo tempo.

Então o React chama `startViewTransition`. Dentro do `updateCallback`, o React:

- Aplicará suas mutações ao DOM e invocará `useInsertionEffects`.
- Esperará o carregamento das fontes.
- Chamará `componentDidMount`, `componentDidUpdate`, `useLayoutEffect` e refs.
- Esperará que qualquer Navegação pendente termine.
- Então o React medirá quaisquer alterações no layout para ver quais limites precisarão ser animados.

Após a resolução da Promise `ready` de `startViewTransition`, o React reverterá o `view-transition-name`. Em seguida, o React invocará os callbacks `onEnter`, `onExit`, `onUpdate` e `onShare` para permitir o controle programático manual das Animações. Isso ocorrerá após os padrões integrados já terem sido computados.

Se um `flushSync` ocorrer no meio desta sequência, o React pulará a Transição, pois depende de ser capaz de concluir de forma síncrona.

Após a resolução da Promise `finished` de `startViewTransition`, o React invocará `useEffect`. Isso evita que eles interfiram no desempenho da Animação. No entanto, isso não é uma garantia, pois se outro `setState` ocorrer enquanto a Animação estiver em execução, ele ainda terá que invocar o `useEffect` mais cedo para preservar as garantias sequenciais.

</DeepDive>

#### Props {/*props*/}

Por padrão, `<ViewTransition>` anima com um fade cruzado suave. Você pode personalizar a animação ou especificar uma transição de elemento compartilhado com estas props:

* **opcional** `enter`: Uma string ou objeto. A [Classe de Transição de Visualização](#view-transition-class) a ser aplicada quando `enter` é ativado.
* **opcional** `exit`: Uma string ou objeto. A [Classe de Transição de Visualização](#view-transition-class) a ser aplicada quando `exit` é ativado.
* **opcional** `update`: Uma string ou objeto. A [Classe de Transição de Visualização](#view-transition-class) a ser aplicada quando uma atualização é ativada.
* **opcional** `share`: Uma string ou objeto. A [Classe de Transição de Visualização](#view-transition-class) a ser aplicada quando um elemento compartilhado é ativado.
* **opcional** `default`: Uma string ou objeto. A [Classe de Transição de Visualização](#view-transition-class) usada quando nenhuma outra prop de ativação correspondente é encontrada.
* **opcional** `name`: Uma string ou objeto. O nome da Transição de Visualização usado para transições de elementos compartilhados. Se não for fornecido, o React usará um nome exclusivo para cada Transição de Visualização para evitar animações inesperadas.

#### Callback {/*events*/}

Esses callbacks permitem que você ajuste a animação imperativamente usando as APIs [animate](https://developer.mozilla.org/en-US/docs/Web/API/Element/animate):

* **opcional** `onEnter`: Uma função. O React chama `onEnter` após uma animação de "enter".
* **opcional** `onExit`: Uma função. O React chama `onExit` após uma animação de "exit".
* **opcional** `onShare`: Uma função. O React chama `onShare` após uma animação de "share".
* **opcional** `onUpdate`: Uma função. O React chama `onUpdate` após uma animação de "update".

Cada callback recebe como argumentos:
- `element`: O elemento DOM que foi animado.
- `types`: Os [Tipos de Transição](/reference/react/addTransitionType) incluídos na animação.

### Classe de Transição de Visualização {/*view-transition-class*/}

A Classe de Transição de Visualização é o(s) nome(s) da(s) classe(s) CSS aplicada(s) pelo React durante a transição quando o ViewTransition é ativado. Pode ser uma string ou um objeto.
- `string`: a `class` adicionada aos elementos filhos quando ativada. Se `'none'` for fornecido, nenhuma classe será adicionada.
- `object`: a classe adicionada aos elementos filhos será a chave correspondente ao tipo de Transição de Visualização adicionado com `addTransitionType`. O objeto também pode especificar um `default` a ser usado se nenhum tipo correspondente for encontrado.

O valor `'none'` pode ser usado para impedir que uma Transição de Visualização seja ativada para um gatilho específico.

### Estilizando Transições de Visualização {/*styling-view-transitions*/}

<Note>

Em muitos exemplos iniciais de Transições de Visualização na web, você verá o uso de [`view-transition-name`](https://developer.mozilla.org/en-US/docs/Web/CSS/view-transition-name) e, em seguida, a estilização usando seletores `::view-transition-...(meu-nome)`. Não recomendamos isso para estilização. Em vez disso, normalmente recomendamos o uso de uma Classe de Transição de Visualização.

</Note>

Para personalizar a animação de um `<ViewTransition>`, você pode fornecer uma Classe de Transição de Visualização a uma das props de ativação. A Classe de Transição de Visualização é um nome de classe CSS que o React aplica aos elementos filhos quando o ViewTransition é ativado.

Por exemplo, para personalizar uma animação de "enter", forneça um nome de classe à prop `enter`:


```js
<ViewTransition enter="slide-in">
```

Quando o `<ViewTransition>` ativar uma animação de "enter", o React adicionará o nome de classe `slide-in`. Em seguida, você pode referenciar essa classe usando [pseudo-seletores de transição de visualização](https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API#pseudo-elements) para criar animações reutilizáveis:

```css
::view-transition-group(.slide-in) {
  
}
::view-transition-old(.slide-in) {

}
::view-transition-new(.slide-in) {

}
```
No futuro, bibliotecas CSS podem adicionar animações integradas usando Classes de Transição de Visualização para facilitar o uso.

#### Ressalvas {/*caveats*/}

- Por padrão, `setState` atualiza imediatamente e não ativa `<ViewTransition>`, apenas atualizações envolvidas em uma [Transição](/reference/react/useTransition). Você também pode usar [`<Suspense>`](/reference/react/Suspense) para optar por uma Transição para [revelar conteúdo](/reference/react/Suspense#revealing-content-together-at-once).
- `<ViewTransition>` cria uma imagem que pode ser movida, dimensionada e com fade cruzado. Ao contrário das Animações de Layout que você pode ter visto no React Native ou Motion, isso significa que nem todo elemento individual dentro dele anima sua posição. Isso pode levar a um melhor desempenho e uma animação mais contínua e suave em comparação com a animação de cada peça individual. No entanto, também pode perder a continuidade em coisas que deveriam estar se movendo por conta própria. Portanto, você pode precisar adicionar mais `<ViewTransition>` limites manualmente como resultado.
- Muitos usuários podem preferir não ter animações na página. O React não desabilita automaticamente as animações para este caso. Recomendamos o uso da consulta de mídia `@media (prefers-reduced-motion)` para desativar animações ou atenuá-las com base na preferência do usuário. No futuro, bibliotecas CSS podem ter isso integrado em seus presets.
- Atualmente, `<ViewTransition>` funciona apenas no DOM. Estamos trabalhando para adicionar suporte para React Native e outras plataformas.

---


## Uso {/*usage*/}

### Animando um elemento ao entrar/sair {/*animating-an-element-on-enter*/}

As Transições de Entrada/Saída são acionadas quando um `<ViewTransition>` é adicionado ou removido por um componente em uma transição:

```js
function Child() {
  return <ViewTransition>Hi</ViewTransition>
}

function Parent() {
  const [show, setShow] = useState();
  if (show) {
    return <Child />;
  }
  return null;
}
```

Quando `setShow` é chamado, `show` muda para `true` e o componente `Child` é renderizado. Quando `setShow` é chamado dentro de `startTransition`, e `Child` renderiza um `ViewTransition` antes de qualquer outro nó DOM, uma animação de `enter` é acionada.

Quando `show` muda de volta para `false`, uma animação de `exit` é acionada.

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

`<ViewTransition>` só ativa se for colocado antes de qualquer nó DOM. Se `Child` fosse assim, nenhuma animação seria acionada:

```js [3, 5]
function Component() {
  return (
    <div>
      <ViewTransition>Hi</ViewTransition>
    </div>
  );
}
```

</Pitfall>

---
### Animando um elemento compartilhado {/*animating-a-shared-element*/}

Normalmente, não recomendamos atribuir um nome a um `<ViewTransition>` e, em vez disso, deixamos o React atribuir um nome automático. A razão pela qual você pode querer atribuir um nome é animar entre componentes completamente diferentes quando uma árvore é desmontada e outra árvore é montada ao mesmo tempo. Para preservar a continuidade.

```js
<ViewTransition name={UNIQUE_NAME}>
  <Child />
</ViewTransition>
```

Quando uma árvore é desmontada e outra é montada, se houver um par onde o mesmo nome existe na árvore que está sendo desmontada e na árvore que está sendo montada, eles acionam a animação "compartilhada" em ambos. Ela anima do lado que está sendo desmontado para o lado que está sendo montado.

Ao contrário de uma animação de saída/entrada, isso pode estar profundamente dentro da árvore deletada/montada. Se um `<ViewTransition>` também fosse elegível para saída/entrada, a animação "compartilhada" teria precedência.

Se a Transição primeiro desmontar um lado e depois levar a um fallback `<Suspense>` a ser exibido antes que o novo nome seja eventualmente montado, nenhuma transição de elemento compartilhado acontece.

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

Se um dos lados do par, montado ou desmontado, estiver fora da viewport, nenhum par será formado. Isso garante que ele não entre ou saia da viewport quando algo for rolado. Em vez disso, ele é tratado como uma entrada/saída regular por si só.

Isso não acontece se a mesma instância do Componente mudar de posição, o que aciona uma "atualização". Essas animam independentemente se uma posição estiver fora da viewport.

Atualmente, há uma peculiaridade onde, se um `<ViewTransition>` desmontado profundamente aninhado estiver dentro da viewport, mas o lado montado não estiver dentro da viewport, o lado desmontado anima como sua própria animação de "saída", mesmo que esteja profundamente aninhado, em vez de como parte da animação pai.

</Note>

<Pitfall>

É importante que haja apenas uma coisa com o mesmo nome montada por vez em todo o aplicativo. Portanto, é importante usar namespaces únicos para o nome para evitar conflitos. Para garantir que você possa fazer isso, talvez você queira adicionar uma constante em um módulo separado que você importe.

```js
export const MY_NAME = "my-globally-unique-name";
import {MY_NAME} from './shared-name';
...
<ViewTransition name={MY_NAME}>
```

</Pitfall>


---

### Animando a reordenação de itens em uma lista {/*animating-reorder-of-items-in-a-list*/}


```js
items.map(item => <Component key={item.id} item={item} />)
```

Ao reordenar uma lista, sem atualizar o conteúdo, a animação de "atualização" é acionada em cada `<ViewTransition>` na lista se eles estiverem fora de um nó DOM. Semelhante às animações de entrada/saída.

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
Em vez disso, qualquer `<ViewTransition>` pai faria uma transição de fade cruzado. Se não houver um `<ViewTransition>` pai, então não haverá animação nesse caso.

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

Isso significa que você pode querer evitar elementos de wrapper em listas onde deseja permitir que o Componente controle sua própria animação de reordenação:

```
items.map(item => <div><Component key={item.id} item={item} /></div>)
```

A regra acima também se aplica se um dos itens for atualizado para redimensionar, o que faz com que os irmãos redimensionem, ele também animará seu `<ViewTransition>` irmão, mas apenas se eles forem irmãos imediatos.

Isso significa que durante uma atualização, que causa muito relayout, ele não anima individualmente cada `<ViewTransition>` na página. Isso levaria a muitas animações barulhentas que distraem da mudança real. Portanto, o React é mais conservador sobre quando uma animação individual é acionada.

<Pitfall>

É importante usar chaves corretamente para preservar a identidade ao reordenar listas. Pode parecer que você poderia usar "nome", transições de elementos compartilhados, para animar reordenações, mas isso não seria acionado se um lado estivesse fora da viewport. Para animar uma reordenação, você geralmente quer mostrar que ela foi para uma posição fora da viewport.

</Pitfall>

---

### Animando a partir de conteúdo Suspense {/*animating-from-suspense-content*/}

Assim como qualquer `Transition`, o React espera por dados e por CSS novo (`<link rel="stylesheet" precedence="...">`) antes de executar a animação. Além disso, `ViewTransitions` também esperam até 500ms para que novas fontes sejam carregadas antes de iniciar a animação, para evitar que elas pisquem posteriormente. Pelo mesmo motivo, uma imagem envolvida em `ViewTransition` esperará a imagem carregar.

Se estiver dentro de uma nova instância de `Suspense`, o fallback é mostrado primeiro. Após o `Suspense` carregar completamente, ele aciona o `<ViewTransition>` para animar a revelação do conteúdo.

Atualmente, isso só acontece para `Transition` do lado do cliente. No futuro, isso também animará o `Suspense` para SSR em streaming quando o conteúdo do servidor suspender durante o carregamento inicial.

Existem duas maneiras de animar `Suspense` dependendo de onde você coloca o `<ViewTransition>`:

Atualização:

```
<ViewTransition>
  <Suspense fallback={<A />}>
    <B />
  </Suspense>
</ViewTransition>
```
Neste cenário, quando o conteúdo muda de A para B, ele será tratado como uma "atualização" e aplicará a classe apropriada. Tanto A quanto B terão o mesmo `view-transition-name` e, portanto, agirão como um cross-fade por padrão.

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

Neste cenário, estas são duas instâncias separadas de `ViewTransition`, cada uma com seu próprio `view-transition-name`. Isso será tratado como uma "saída" de `<A>` e uma "entrada" de `<B>`.

Você pode obter efeitos diferentes dependendo de onde escolher colocar o limite do `<ViewTransition>`.

---
### Optando por não participar de uma animação {/*opting-out-of-an-animation*/}

Às vezes, você está envolvendo um componente grande existente, como uma página inteira, e deseja animar algumas atualizações, como a mudança de tema. No entanto, você não quer que todas as atualizações dentro da página inteira participem do cross-fade quando elas estiverem sendo atualizadas. Especialmente se você estiver adicionando mais animações incrementalmente.

Você pode usar a classe "none" para optar por não participar de uma animação. Ao envolver seus filhos em um "none", você pode desabilitar animações para atualizações neles, enquanto o pai ainda aciona.

```js
<ViewTransition>
  <div className={theme}>
    <ViewTransition update="none">
      {children}
    </ViewTransition>
  </div>
</ViewTransition>
```

Isso só animará se o tema mudar e não se apenas os filhos forem atualizados. Os filhos ainda podem optar por participar novamente com seu próprio `<ViewTransition>`, mas pelo menos será manual novamente.

---

### Opting-out of an animation {/*opting-out-of-an-animation*/}

Às vezes, você está encapsulando um componente grande e existente, como uma página inteira, e deseja animar algumas atualizações, como a mudança de tema. No entanto, você não quer que todas as atualizações dentro da página inteira façam um cross-fade quando estiverem sendo atualizadas. Especialmente se você estiver adicionando mais animações incrementalmente.

Você pode usar a classe "none" para optar por não participar de uma animação. Ao encapsular seus filhos em um "none", você pode desativar animações para atualizações neles enquanto o pai ainda dispara.

```js
<ViewTransition>
  <div className={theme}>
    <ViewTransition update="none">
      {children}
    </ViewTransition>
  </div>
</ViewTransition>
```

Isso só animará se o tema mudar e não se apenas os filhos forem atualizados. Os filhos ainda podem optar por participar novamente com seu próprio `<ViewTransition>`, mas pelo menos é manual novamente.

---

### Customizing animations {/*customizing-animations*/}

Por padrão, `<ViewTransition>` inclui o cross-fade padrão do navegador.

Para personalizar animações, você pode fornecer props ao componente `<ViewTransition>` para especificar quais animações usar, com base em como o `<ViewTransition>` é ativado.

Por exemplo, podemos desacelerar a animação de cross-fade padrão:

```js
<ViewTransition default="slow-fade">
  <Video />
</ViewTransition>
```

E definir `slow-fade` em CSS usando classes de transição de visualização:

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

Além de definir o `default`, você também pode fornecer configurações para animações `enter`, `exit`, `update` e `share`.

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
Você pode usar a API [`addTransitionType`](/reference/react/addTransitionType) para adicionar um nome de classe aos elementos filhos quando um tipo específico de transição é ativado para um gatilho de ativação específico. Isso permite que você personalize a animação para cada tipo de transição.

Por exemplo, para personalizar a animação para todas as navegações para frente e para trás:

```js
<ViewTransition default={{
  'navigation-back': 'slide-right',
  'navigation-forward': 'slide-left',
 }}>
  <div>...</div>
</ViewTransition>
 
// no seu router:
startTransition(() => {
  addTransitionType('navigation-' + navigationType);
});
```

Quando o ViewTransition ativar uma animação "navigation-back", o React adicionará o nome de classe "slide-right". Quando o ViewTransition ativar uma animação "navigation-forward", o React adicionará o nome de classe "slide-left".

No futuro, roteadores e outras bibliotecas poderão adicionar suporte a tipos e estilos padrão de transição de visualização.

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

### Construindo roteadores com suporte a View Transition {/*building-view-transition-enabled-routers*/}

O React aguarda a conclusão de qualquer Navegação pendente para garantir que a restauração da rolagem ocorra dentro da animação. Se a Navegação for bloqueada no React, seu roteador deve desbloquear em `useLayoutEffect`, pois `useEffect` levaria a um deadlock.

Se um `startTransition` for iniciado a partir do evento popstate legado, como durante uma navegação "para trás", ele deverá ser concluído de forma síncrona para garantir que a restauração de scroll e formulário funcione corretamente. Isso entra em conflito com a execução de uma animação View Transition. Portanto, o React pulará as animações do popstate. Consequentemente, as animações não serão executadas para o botão voltar. Você pode corrigir isso atualizando seu roteador para usar a API de Navegação.

---

## Solução de Problemas {/*troubleshooting*/}

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

Para corrigir, certifique-se de que o `<ViewTransition>` venha antes de quaisquer outros nós DOM:

```js [3, 5]
function Component() {
  return (
    <ViewTransition>
      <div>Oi</div>
    </ViewTransition>
  );
}
```

### Estou recebendo o erro "Existem dois componentes `<ViewTransition name=%s>` com o mesmo nome montados ao mesmo tempo." {/*two-viewtransition-with-same-name*/}

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

Isso fará com que a Transição de Visualização gere um erro. Em desenvolvimento, o React detecta esse problema para apresentá-lo e registra dois erros:

<ConsoleBlockMulti>
<ConsoleLogLine level="error">

Existem dois componentes `<ViewTransition name=%s>` com o mesmo nome montados ao mesmo tempo. Isso não é suportado e fará com que as Transições de Visualização gerem um erro. Tente usar um nome mais exclusivo, por exemplo, usando um prefixo de namespace e adicionando o id de um item ao nome.
{'    '}at Item
{'    '}at ItemList

</ConsoleLogLine>

<ConsoleLogLine level="error">

A duplicata `<ViewTransition name=%s>` existente tem este trace de pilha.
{'    '}at Item
{'    '}at ItemList

</ConsoleLogLine>
</ConsoleBlockMulti>

Para corrigir, certifique-se de que haja apenas um `<ViewTransition>` com o mesmo nome montado ao mesmo tempo em todo o aplicativo, garantindo que o `name` seja exclusivo ou adicionando um `id` ao nome:

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