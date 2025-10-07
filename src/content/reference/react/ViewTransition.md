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

`<ViewTransition>` permite animar elementos que são atualizados dentro de uma [Transição](/reference/react/useTransition).


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

- `enter`: Se um `<ViewTransition>` é inserido nesta Transição, então ele será ativado.
- `exit`: Se um `<ViewTransition>` é removido nesta Transição, então ele será ativado.
- `update`: Se um `<ViewTransition>` tem alguma mutação no DOM dentro dele que o React está realizando (como uma prop mudando) ou se a própria fronteira do `<ViewTransition>` muda de tamanho ou posição devido a um irmão imediato. Se houver `<ViewTransition>` aninhados, a mutação se aplica a eles e não ao pai.
- `share`: Se um `<ViewTransition>` nomeado está dentro de uma subárvore removida e outro `<ViewTransition>` nomeado com o mesmo nome faz parte de uma subárvore inserida na mesma Transição, eles formam uma Transição de Elemento Compartilhado e animam do elemento removido para o inserido.

Por padrão, `<ViewTransition>` anima com um fade cruzado suave (a transição de visualização padrão do navegador). Você pode personalizar a animação fornecendo uma [Classe de Transição de Visualização](#view-transition-class) para o componente `<ViewTransition>`. Você pode personalizar animações para cada tipo de gatilho (veja [Estilizando Transições de Visualização](#styling-view-transitions)).

<DeepDive>

#### Como funciona o `<ViewTransition>`? {/*how-does-viewtransition-work*/}

Por baixo dos panos, o React aplica `view-transition-name` aos estilos inline do nó DOM mais próximo aninhado dentro do componente `<ViewTransition>`. Se houver múltiplos nós DOM irmãos como `<ViewTransition><div /><div /></ViewTransition>`, o React adiciona um sufixo ao nome para tornar cada um único, mas conceitualmente eles fazem parte do mesmo. O React não aplica isso de forma antecipada, mas apenas no momento em que a fronteira deve participar de uma animação.

O React chama automaticamente `startViewTransition` nos bastidores, então você nunca deve fazer isso sozinho. Na verdade, se você tiver algo mais na página executando uma ViewTransition, o React a interromperá. Portanto, é recomendado que você use o próprio React para coordenar essas transições. Se você usou outras formas de acionar ViewTransitions no passado, recomendamos que migre para a forma integrada.

Se houver outras ViewTransitions do React já em execução, o React esperará que elas terminem antes de iniciar a próxima. No entanto, é importante notar que se houver várias atualizações ocorrendo enquanto a primeira está em execução, todas elas serão agrupadas em uma única transição. Se você iniciar A->B. Então, nesse meio tempo, você recebe uma atualização para ir para C e depois para D. Quando a primeira animação A->B terminar, a próxima animará de B->D.

O ciclo de vida `getSnapshotBeforeUpdate` será chamado antes de `startViewTransition` e alguns `view-transition-name` serão atualizados ao mesmo tempo.

Em seguida, o React chama `startViewTransition`. Dentro do `updateCallback`, o React:

- Aplica suas mutações ao DOM e invoca `useInsertionEffects`.
- Espera que as fontes carreguem.
- Chama `componentDidMount`, `componentDidUpdate`, `useLayoutEffect` e refs.
- Espera que qualquer Navegação pendente termine.
- Em seguida, o React medirá quaisquer alterações no layout para ver quais fronteiras precisarão ser animadas.

Após a resolução da Promise `ready` de `startViewTransition`, o React reverterá o `view-transition-name`. Em seguida, o React invocará os callbacks `onEnter`, `onExit`, `onUpdate` e `onShare` para permitir o controle programático manual sobre as Animações. Isso ocorrerá após as animações padrão integradas já terem sido computadas.

Se um `flushSync` ocorrer no meio desta sequência, o React pulará a Transição, pois depende de ser capaz de concluir de forma síncrona.

Após a resolução da Promise `finished` de `startViewTransition`, o React invocará `useEffect`. Isso evita que essas animações interfiram no desempenho. No entanto, isso não é uma garantia, pois se outro `setState` ocorrer enquanto a animação estiver em execução, ele ainda terá que invocar o `useEffect` mais cedo para preservar as garantias sequenciais.

</DeepDive>

#### Props {/*props*/}

Por padrão, `<ViewTransition>` anima com um fade cruzado suave. Você pode personalizar a animação ou especificar uma transição de elemento compartilhado com estas props:

* **opcional** `enter`: Uma string ou objeto. A [Classe de Transição de Visualização](#view-transition-class) a ser aplicada quando `enter` é ativado.
* **opcional** `exit`: Uma string ou objeto. A [Classe de Transição de Visualização](#view-transition-class) a ser aplicada quando `exit` é ativado.
* **opcional** `update`: Uma string ou objeto. A [Classe de Transição de Visualização](#view-transition-class) a ser aplicada quando uma atualização é ativada.
* **opcional** `share`: Uma string ou objeto. A [Classe de Transição de Visualização](#view-transition-class) a ser aplicada quando um elemento compartilhado é ativado.
* **opcional** `default`: Uma string ou objeto. A [Classe de Transição de Visualização](#view-transition-class) usada quando nenhuma outra prop de ativação correspondente é encontrada.
* **opcional** `name`: Uma string ou objeto. O nome da Transição de Visualização usado para transições de elementos compartilhados. Se não for fornecido, o React usará um nome exclusivo para cada Transição de Visualização para evitar animações inesperadas.

#### Callbacks {/*events*/}

Esses callbacks permitem ajustar a animação imperativamente usando as APIs [animate](https://developer.mozilla.org/en-US/docs/Web/API/Element/animate):

* **opcional** `onEnter`: Uma função. O React chama `onEnter` após uma animação "enter".
* **opcional** `onExit`: Uma função. O React chama `onExit` após uma animação "exit".
* **opcional** `onShare`: Uma função. O React chama `onShare` após uma animação "share".
* **opcional** `onUpdate`: Uma função. O React chama `onUpdate` após uma animação "update".

Cada callback recebe como argumentos:
- `element`: O elemento DOM que foi animado.
- `types`: Os [Tipos de Transição](/reference/react/addTransitionType) incluídos na animação.

### Classe de Transição de Visualização {/*view-transition-class*/}

A Classe de Transição de Visualização é o(s) nome(s) da(s) classe(s) CSS aplicada(s) pelo React durante a transição quando o `<ViewTransition>` é ativado. Pode ser uma string ou um objeto.
- `string`: a `class` adicionada aos elementos filhos quando ativada. Se `'none'` for fornecido, nenhuma classe será adicionada.
- `object`: a classe adicionada aos elementos filhos será a chave correspondente ao tipo de Transição de Visualização adicionado com `addTransitionType`. O objeto também pode especificar um `default` a ser usado se nenhum tipo correspondente for encontrado.

O valor `'none'` pode ser usado para impedir que uma Transição de Visualização seja ativada para um gatilho específico.

### Estilizando Transições de Visualização {/*styling-view-transitions*/}

<Note>

Em muitos exemplos iniciais de Transições de Visualização na web, você verá o uso de [`view-transition-name`](https://developer.mozilla.org/en-US/docs/Web/CSS/view-transition-name) e, em seguida, a estilização usando seletores `::view-transition-...(meu-nome)`. Não recomendamos isso para estilização. Em vez disso, normalmente recomendamos o uso de uma Classe de Transição de Visualização.

</Note>

Para personalizar a animação de um `<ViewTransition>`, você pode fornecer uma Classe de Transição de Visualização a uma das props de ativação. A Classe de Transição de Visualização é um nome de classe CSS que o React aplica aos elementos filhos quando a Transição de Visualização é ativada.

Por exemplo, para personalizar uma animação de "enter", forneça um nome de classe à prop `enter`:


```js
<ViewTransition enter="slide-in">
```

Quando o `<ViewTransition>` ativar uma animação "enter", o React adicionará o nome de classe `slide-in`. Em seguida, você pode referenciar essa classe usando [pseudo-seletores de transição de visualização](https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API#pseudo-elements) para criar animações reutilizáveis:

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

- Por padrão, as atualizações `setState` ocorrem imediatamente e não ativam `<ViewTransition>`, apenas atualizações envolvidas em uma [Transição](/reference/react/useTransition). Você também pode usar [`<Suspense>`](/reference/react/Suspense) para optar por uma Transição para [revelar conteúdo](/reference/react/Suspense#revealing-content-together-at-once).
- `<ViewTransition>` cria uma imagem que pode ser movida, dimensionada e com fade cruzado. Ao contrário das Animações de Layout que você pode ter visto no React Native ou Motion, isso significa que nem todo Elemento individual dentro dele anima sua posição. Isso pode levar a um melhor desempenho e a uma animação mais contínua e suave em comparação com a animação de cada peça individual. No entanto, também pode perder a continuidade em coisas que deveriam estar se movendo sozinhas. Portanto, você pode ter que adicionar mais `<ViewTransition>` fronteiras manualmente como resultado.
- Muitos usuários podem preferir não ter animações na página. O React não desabilita automaticamente as animações para este caso. Recomendamos o uso da consulta de mídia `@media (prefers-reduced-motion)` para desativar animações ou atenuá-las com base na preferência do usuário. No futuro, bibliotecas CSS podem ter isso integrado em seus presets.
- Atualmente, `<ViewTransition>` funciona apenas no DOM. Estamos trabalhando para adicionar suporte para React Native e outras plataformas.

---

## Uso {/*usage*/}

### Animando um elemento na entrada/saída {/*animating-an-element-on-enter*/}

Transições de Entrada/Saída disparam quando um `<ViewTransition>` é adicionado ou removido por um componente em uma transição:

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

Quando `setShow` é chamado, `show` muda para `true` e o componente `Child` é renderizado. Quando `setShow` é chamado dentro de `startTransition`, e `Child` renderiza um `ViewTransition` antes de quaisquer outros nós DOM, uma animação `enter` é acionada.

Quando `show` volta para `false`, uma animação `exit` é acionada.

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

`<ViewTransition>` só é ativado se for colocado antes de qualquer nó DOM. Se `Child` fosse assim, nenhuma animação seria acionada:

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

Normalmente, não recomendamos atribuir um nome a um `<ViewTransition>` e, em vez disso, deixamos o React atribuir um nome automático. A razão pela qual você pode querer atribuir um nome é animar entre componentes completamente diferentes quando uma árvore é desmontada e outra é montada ao mesmo tempo. Para preservar a continuidade.

```js