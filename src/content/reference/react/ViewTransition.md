---
title: <ViewTransition>
version: experimental
---
```
<Experimental>

**Esta API é experimental e ainda não está disponível em uma versão estável do React.**

Você pode experimentá-la atualizando os pacotes do React para a versão experimental mais recente:

- `react@experimental`
- `react-dom@experimental`
- `eslint-plugin-react-hooks@experimental`

As versões experimentais do React podem conter erros. Não as use em produção.

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

Envolva elementos em `<ViewTransition>` para animá-los quando forem atualizados dentro de uma [Transição](/reference/react/useTransition). O React usa as seguintes heurísticas para determinar se uma Transição de Visualização é ativada para uma animação:

- `enter`: Se um `ViewTransition` em si for inserido nesta Transição, então isso será ativado.
- `exit`: Se um `ViewTransition` em si for excluído nesta Transição, então isso será ativado.
- `update`: Se um `ViewTransition` tiver alguma mutação DOM dentro dele que o React está fazendo (como uma alteração de prop) ou se o próprio limite `ViewTransition` mudar de tamanho ou posição devido a um irmão imediato. Se houver `ViewTransition` aninhados, a mutação se aplica a eles e não ao pai.
- `share`: Se um `ViewTransition` nomeado estiver dentro de uma subárvore excluída e outro `ViewTransition` nomeado com o mesmo nome fizer parte de uma subárvore inserida na mesma Transição, eles formam uma Transição de Elemento Compartilhado e ela anima do excluído para o inserido.

Por padrão, `<ViewTransition>` anima com um cross-fade suave (a transição de visualização padrão do navegador). Você pode personalizar a animação fornecendo uma [Classe de Transição de Visualização](#view-transition-class) ao componente `<ViewTransition>`. Você pode personalizar animações para cada tipo de gatilho (consulte [Estilizando Transições de Visualização](#styling-view-transitions)).

<DeepDive>

#### Como `<ViewTransition>` funciona? {/*how-does-viewtransition-work*/}

Nos bastidores, o React aplica `view-transition-name` aos estilos embutidos do nó DOM mais próximo aninhado dentro do componente `<ViewTransition>`. Se houver vários nós DOM irmãos como `<ViewTransition><div /><div /></ViewTransition>`, o React adiciona um sufixo ao nome para tornar cada um único, mas conceitualmente eles fazem parte do mesmo. O React não os aplica ansiosamente, mas apenas no momento em que esse limite deve participar de uma animação.

O React chama automaticamente `startViewTransition` por trás das cenas, então você nunca deve fazer isso sozinho. Na verdade, se você tiver outra coisa na página executando um ViewTransition, o React o interromperá. Portanto, é recomendável que você use o próprio React para coordená-los. Se você tinha outras maneiras de acionar ViewTransitions no passado, recomendamos que você migre para a maneira integrada.

Se houver outros React ViewTransitions já em execução, o React esperará que eles terminem antes de iniciar o próximo. No entanto, é importante ressaltar que, se houver várias atualizações acontecendo enquanto a primeira estiver em execução, todas elas serão agrupadas em uma só. Se você iniciar A->B. Enquanto isso, você recebe uma atualização para ir para C e depois D. Quando a primeira animação A->B terminar, a próxima animará de B->D.

O ciclo de vida `getSnapshotBeforeUpdate` será chamado antes de `startViewTransition` e algum `view-transition-name` será atualizado ao mesmo tempo.

Então, o React chama `startViewTransition`. Dentro do `updateCallback`, o React irá:

- Aplicar suas mutações ao DOM e invocar useInsertionEffects.
- Aguardar o carregamento das fontes.
- Chamar componentDidMount, componentDidUpdate, useLayoutEffect e refs.
- Aguardar a conclusão de qualquer Navegação pendente.
- Então, o React medirá quaisquer alterações no layout para ver quais limites precisarão animar.

Depois que a Promise pronta do `startViewTransition` for resolvida, o React reverterá o `view-transition-name`. Então, o React invocará os callbacks `onEnter`, `onExit`, `onUpdate` e `onShare` para permitir o controle programático manual sobre as Animações. Isso será depois que as integradas padrão já tiverem sido computadas.

Se um `flushSync` acontecer no meio dessa sequência, o React pulará a Transição, pois ela depende da capacidade de ser concluída de forma síncrona.

Depois que a Promise finalizada do `startViewTransition` for resolvida, o React invocará `useEffect`. Isso impede que eles interfiram no desempenho da Animação. No entanto, isso não é uma garantia, pois se outro `setState` acontecer enquanto a Animação estiver em execução, ele ainda terá que invocar o `useEffect` mais cedo para preservar as garantias sequenciais.

</DeepDive>

#### Props {/*props*/}

Por padrão, `<ViewTransition>` anima com um cross-fade suave. Você pode personalizar a animação ou especificar uma transição de elemento compartilhado com estas props:

* **opcional** `enter`: Uma string ou objeto. A [Classe de Transição de Visualização](#view-transition-class) a ser aplicada quando a entrada for ativada.
* **opcional** `exit`: Uma string ou objeto. A [Classe de Transição de Visualização](#view-transition-class) a ser aplicada quando a saída for ativada.
* **opcional** `update`: Uma string ou objeto. A [Classe de Transição de Visualização](#view-transition-class) a ser aplicada quando uma atualização for ativada.
* **opcional** `share`: Uma string ou objeto. A [Classe de Transição de Visualização](#view-transition-class) a ser aplicada quando um elemento compartilhado for ativado.
* **opcional** `default`: Uma string ou objeto. A [Classe de Transição de Visualização](#view-transition-class) usada quando nenhuma outra prop de ativação correspondente é encontrada.
* **opcional** `name`: Uma string ou objeto. O nome da Transição de Visualização usada para transições de elementos compartilhados. Se não for fornecido, o React usará um nome exclusivo para cada Transição de Visualização para evitar animações inesperadas.

#### Callback {/*events*/}

Esses callbacks permitem que você ajuste a animação de forma imperativa usando as APIs [animate](https://developer.mozilla.org/en-US/docs/Web/API/Element/animate):

* **opcional** `onEnter`: Uma função. O React chama `onEnter` após uma animação de "entrada".
* **opcional** `onExit`: Uma função. O React chama `onExit` após uma animação de "saída".
* **opcional** `onShare`: Uma função. O React chama `onShare` após uma animação de "compartilhamento".
* **opcional** `onUpdate`: Uma função. O React chama `onUpdate` após uma animação de "atualização".

Cada callback recebe como argumentos:
- `element`: O elemento DOM que foi animado.
- `types`: Os [Tipos de Transição](/reference/react/addTransitionType) incluídos na animação.

### Classe de Transição de Visualização {/*view-transition-class*/}

A Classe de Transição de Visualização é o(s) nome(s) da classe CSS aplicado(s) pelo React durante a transição quando o ViewTransition é ativado. Pode ser uma string ou um objeto.
- `string`: a `class` adicionada aos elementos filhos quando ativada. Se `'none'` for fornecido, nenhuma classe será adicionada.
- `object`: a classe adicionada aos elementos filhos será a chave correspondente ao tipo de Transição de Visualização adicionada com `addTransitionType`. O objeto também pode especificar um `default` para usar se nenhum tipo correspondente for encontrado.

O valor `'none'` pode ser usado para impedir que uma Transição de Visualização seja ativada para um gatilho específico.

### Estilizando Transições de Visualização {/*styling-view-transitions*/}

<Note>

Em muitos exemplos iniciais de Transições de Visualização pela web, você terá visto o uso de um [`view-transition-name`](https://developer.mozilla.org/en-US/docs/Web/CSS/view-transition-name) e, em seguida, estilizado-o usando seletores `::view-transition-...(meu-nome)`. Não recomendamos isso para estilização. Em vez disso, normalmente recomendamos o uso de uma Classe de Transição de Visualização.

</Note>

Para personalizar a animação para um `<ViewTransition>`, você pode fornecer uma Classe de Transição de Visualização a uma das props de ativação. A Classe de Transição de Visualização é um nome de classe CSS que o React aplica aos elementos filhos quando o ViewTransition é ativado.

Por exemplo, para personalizar uma animação de "entrada", forneça um nome de classe à prop `enter`:

```js
<ViewTransition enter="slide-in">
```

Quando o `<ViewTransition>` ativa uma animação de "entrada", o React adicionará o nome da classe `slide-in`. Então, você pode se referir a esta classe usando [seletores de pseudo-elementos de transição de visualização](https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API#pseudo-elements) para construir animações reutilizáveis:

```css
::view-transition-group(.slide-in) {
  
}
::view-transition-old(.slide-in) {

}
::view-transition-new(.slide-in) {

}
```
No futuro, as bibliotecas CSS podem adicionar animações integradas usando Classes de Transição de Visualização para tornar isso mais fácil de usar.

#### Ressalvas {/*caveats*/}

- Por padrão, as atualizações de `setState` são imediatas e não ativam `<ViewTransition>`, apenas as atualizações encapsuladas em uma [Transição](/reference/react/useTransition). Você também pode usar [`<Suspense>`](/reference/react/Suspense) para participar de uma Transição para [revelar conteúdo](/reference/react/Suspense#revealing-content-together-at-once).
- `<ViewTransition>` cria uma imagem que pode ser movida, dimensionada e com cross-fade. Ao contrário das Animações de Layout que você pode ter visto no React Native ou Motion, isso significa que nem todos os Elementos individuais dentro dele animam sua posição. Isso pode levar a um melhor desempenho e uma animação mais contínua e suave em comparação com a animação de cada peça individual. No entanto, também pode perder a continuidade em coisas que devem estar se movendo sozinhas. Portanto, pode ser necessário adicionar mais limites `<ViewTransition>` manualmente como resultado.
- Muitos usuários podem preferir não ter animações na página. O React não desativa automaticamente as animações para este caso. Recomendamos o uso da consulta de mídia `@media (prefers-reduced-motion)` para desativar as animações ou reduzi-las com base na preferência do usuário. No futuro, as bibliotecas CSS podem ter isso integrado em seus predefinições.
- Atualmente, `<ViewTransition>` só funciona no DOM. Estamos trabalhando para adicionar suporte para React Native e outras plataformas.

---

## Uso {/*usage*/}

### Animando um elemento na entrada/saída {/*animating-an-element-on-enter*/}

As Transições de Entrada/Saída são acionadas quando um `<ViewTransition>` é adicionado ou removido por um componente em uma transição:

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

Quando `setShow` é chamado, `show` muda para `true` e o componente `Child` é renderizado. Quando `setShow` é chamado dentro de `startTransition`, e `Child` renderiza um `ViewTransition` antes de quaisquer outros nós DOM, uma animação de `enter` é acionada.

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

`<ViewTransition>` só é ativado se for colocado antes de qualquer nó DOM. Se `Child` tivesse esta aparência, nenhuma animação seria acionada:

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
### Animando um elemento compartilhado {/*animating-a-shared-element*/}

Normalmente, não recomendamos atribuir um nome a um `<ViewTransition>` e, em vez disso, deixar o React atribuir um nome automático. A razão pela qual você pode querer atribuir um nome é animar entre componentes completamente diferentes quando uma árvore é desmontada e outra árvore é montada ao mesmo tempo. Para preservar a continuidade.

```js
<ViewTransition name={UNIQUE_NAME}>
  <Child />
</ViewTransition>
```

Quando uma árvore é desmontada e outra é montada, se houver um par em que o mesmo nome exista na árvore de desmontagem e na árvore de montagem, eles acionam a animação de "compartilhamento" em ambos. Ele anima do lado da desmontagem para o lado da montagem.

Ao contrário de uma animação de saída/entrada, isso pode estar profundamente dentro da árvore excluída/montada. Se um `<ViewTransition>` também for elegível para saída/entrada, a animação de "compartilhamento" terá precedência.

Se a Transição primeiro desmontar um lado e, em seguida, levar a um fallback `<Suspense>` sendo mostrado antes que o novo nome seja eventualmente montado, nenhuma transição de elemento compartilhado acontecerá.

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

Isso não acontece se a mesma instância do Componente mudar de posição, o que aciona uma "atualização". Aqueles animam independentemente se uma posição estiver fora da janela de visualização.

Atualmente, há uma peculiaridade em que, se um `<ViewTransition>` desmontado profundamente aninhado estiver dentro da janela de visualização, mas o lado montado não estiver dentro da janela de visualização, o lado desmontado anima como sua própria animação de "saída", mesmo que esteja profundamente aninhado em vez de como parte da animação pai.

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
Em vez disso, qualquer `<ViewTransition>` pai faria o cross-fade. Se não houver `<ViewTransition>` pai, não haverá animação nesse caso.

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
  background-color: #