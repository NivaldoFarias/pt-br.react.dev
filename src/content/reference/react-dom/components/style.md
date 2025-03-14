```
---
style: "<style>"
---

<Intro>

O [componente `<style>` do navegador embutido](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/style) permite que você adicione folhas de estilo CSS inline ao seu documento.

```js
<style>{` p { color: red; } `}</style>
```

</Intro>

<InlineToc />

---

## Referência {/*reference*/}

### `<style>` {/*style*/}

Para adicionar estilos inline ao seu documento, renderize o [componente `<style>` do navegador embutido](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/style). Você pode renderizar `<style>` de qualquer componente, e o React irá [em certos casos](#special-rendering-behavior) colocar o elemento DOM correspondente no cabeçalho do documento e de-duplicar estilos idênticos.

```js
<style>{` p { color: red; } `}</style>
```

[Veja mais exemplos abaixo.](#usage)

#### Props {/*props*/}

`<style>` suporta todas as [props de elementos comuns](/reference/react-dom/components/common#props).

* `children`: uma string, obrigatória. O conteúdo da folha de estilo.
* `precedence`: uma string. Diz ao React onde classificar o nó DOM `<style>` em relação a outros no `<head>` do documento, o que determina qual folha de estilo pode substituir a outra. React irá inferir que os valores de precedência que ele descobrir primeiro são "menores" e os valores de precedência que ele descobrir mais tarde são "maiores". Muitos sistemas de estilo podem funcionar bem usando um único valor de precedência porque as regras de estilo são atômicas. Folhas de estilo com a mesma precedência combinam-se, sejam tags `<link>` ou inline `<style>`, ou carregadas usando funções [`preinit`](/reference/react-dom/preinit).
* `href`: uma string. Permite que o React [de-duplique estilos](#special-rendering-behavior) que tenham o mesmo `href`.
* `media`: uma string. Restringe a folha de estilo a certa [media query](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_media_queries/Using_media_queries).
* `nonce`: uma string. Um [nonce](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/nonce) criptográfico para permitir o recurso ao usar uma Content Security Policy estrita.
* `title`: uma string. Especifica o nome de uma [folha de estilo alternativa](https://developer.mozilla.org/en-US/docs/Web/CSS/Alternative_style_sheets).

Props que **não são recomendadas** para uso com o React:

* `blocking`: uma string. Se definido como `"render"`, instrui o navegador a não renderizar a página até que a folha de estilo seja carregada. React fornece controle mais granular usando Suspense.

#### Comportamento de renderização especial {/*special-rendering-behavior*/}

O React pode mover componentes `<style>` para o `<head>` do documento, de-duplicar folhas de estilo idênticas e [suspender](/reference/react/Suspense) enquanto a folha de estilo está carregando.

Para optar por esse comportamento, forneça as props `href` e `precedence`. O React de-duplicará estilos se eles tiverem o mesmo `href`. A prop `precedence` diz ao React onde classificar o nó DOM `<style>` em relação a outros no `<head>` do documento, o que determina qual folha de estilo pode substituir a outra.

Esse tratamento especial vem com duas ressalvas:

* O React irá ignorar as mudanças nas props depois que o estilo tiver sido renderizado. (O React emitirá um aviso no desenvolvimento se isso acontecer.)
* O React pode deixar o estilo no DOM, mesmo depois que o componente que o renderizou tiver sido desmontado.

---

## Uso {/*usage*/}

### Renderizando uma folha de estilo CSS inline {/*rendering-an-inline-css-stylesheet*/}

Se um componente depende de certos estilos CSS para ser exibido corretamente, você pode renderizar uma folha de estilo inline dentro do componente.

A prop `href` deve identificar unicamente a folha de estilo, porque o React irá de-duplicar folhas de estilo que têm o mesmo `href`.
Se você fornecer uma prop `precedence`, o React irá reordenar as folhas de estilo inline com base na ordem em que esses valores aparecem na árvore de componentes.

As folhas de estilo inline não acionarão limites Suspense enquanto estiverem carregando.
Mesmo que carreguem recursos assíncronos como fontes ou imagens.

<SandpackWithHTMLOutput>

```js src/App.js active
import ShowRenderedHTML from './ShowRenderedHTML.js';
import { useId } from 'react';

function PieChart({data, colors}) {
  const id = useId();
  const stylesheet = colors.map((color, index) =>
    `#${id} .color-${index}: \{ color: "${color}"; \}`
  ).join();
  return (
    <>
      <style href={"PieChart-" + JSON.stringify(colors)} precedence="medium">
        {stylesheet}
      </style>
      <svg id={id}>
        …
      </svg>
    </>
  );
}

export default function App() {
  return (
    <ShowRenderedHTML>
      <PieChart data="..." colors={['red', 'green', 'blue']} />
    </ShowRenderedHTML>
  );
}
```

</SandpackWithHTMLOutput>
```