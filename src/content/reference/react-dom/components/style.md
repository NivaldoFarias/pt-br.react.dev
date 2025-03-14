---
style: "<style>"
---

<Intro>

O [componente nativo do navegador `<style>`](https://developer.mozilla.org/pt-BR/docs/Web/HTML/Element/style) permite que você adicione folhas de estilo CSS embutidas ao seu documento.

```js
<style>{` p { color: red; } `}</style>
```

</Intro>

<InlineToc />

---

## Referência {/*reference*/}

### `<style>` {/*style*/}

Para adicionar estilos embutidos ao seu documento, renderize o [componente nativo do navegador `<style>`](https://developer.mozilla.org/pt-BR/docs/Web/HTML/Element/style). Você pode renderizar `<style>` de qualquer componente e o React, [em certos casos](#special-rendering-behavior), colocará o elemento DOM correspondente no cabeçalho do documento e removerá estilos idênticos duplicados.

```js
<style>{` p { color: red; } `}</style>
```

[Veja mais exemplos abaixo.](#usage)

#### Props {/*props*/}

`<style>` suporta todas as [props de elementos comuns.](/reference/react-dom/components/common#props)

* `children`: uma string, obrigatória. O conteúdo da folha de estilo.
* `precedence`: uma string. Diz ao React onde classificar o nó DOM `<style>` em relação a outros no `<head>` do documento, o que determina qual folha de estilo pode substituir a outra. O React inferirá que os valores de precedência que ele descobrir primeiro são "menores" e os valores de precedência que ele descobrir depois são "maiores". Muitos sistemas de estilo podem funcionar bem usando um único valor de precedência porque as regras de estilo são atômicas. As folhas de estilo com a mesma precedência se juntam, sejam elas tags `<link>` ou `<style>` embutidas ou carregadas usando funções [`preinit`](/reference/react-dom/preinit).
* `href`: uma string. Permite que o React [remova estilos duplicados](#special-rendering-behavior) que possuem o mesmo `href`.
* `media`: uma string. Restringe a folha de estilo a uma determinada [media query](https://developer.mozilla.org/pt-BR/docs/Web/CSS/CSS_media_queries/Using_media_queries).
* `nonce`: uma string. Um [nonce](https://developer.mozilla.org/pt-BR/docs/Web/HTML/Global_attributes/nonce) criptográfico para permitir o recurso ao usar uma Content Security Policy estrita.
* `title`: uma string. Especifica o nome de uma [folha de estilo alternativa](https://developer.mozilla.org/pt-BR/docs/Web/CSS/Alternative_style_sheets).

Props que **não são recomendadas** para uso com o React:

* `blocking`: uma string. Se definida como `"render"`, instrui o navegador a não renderizar a página até que a folha de estilo seja carregada. O React fornece um controle mais preciso usando Suspense.

#### Comportamento especial de renderização {/*special-rendering-behavior*/}

O React pode mover componentes `<style>` para o `<head>` do documento, remover folhas de estilo idênticas duplicadas e [suspender](/reference/react/Suspense) enquanto a folha de estilo estiver carregando.

Para participar desse comportamento, forneça as props `href` e `precedence`. O React removerá estilos duplicados se eles tiverem o mesmo `href`. A prop precedence diz ao React onde classificar o nó DOM `<style>` em relação a outros no `<head>` do documento, o que determina qual folha de estilo pode substituir a outra.

Este tratamento especial vem com duas ressalvas:

* O React ignorará as alterações nas props depois que o estilo for renderizado. (O React emitirá um aviso em desenvolvimento se isso acontecer.)
* O React pode deixar o estilo no DOM mesmo depois que o componente que o renderizou foi desmontado.

---

## Usage {/*usage*/}

### Renderizando uma folha de estilo CSS embutida {/*rendering-an-inline-css-stylesheet*/}

Se um componente depende de determinados estilos CSS para serem exibidos corretamente, você pode renderizar uma folha de estilo embutida dentro do componente.

A prop `href` deve identificar exclusivamente a folha de estilo, porque o React removerá as folhas de estilo duplicadas que possuem o mesmo `href`.
Se você fornecer uma prop `precedence`, o React reordenará as folhas de estilo embutidas com base na ordem em que esses valores aparecem na árvore de componentes.

As folhas de estilo embutidas não acionarão limites de Suspense enquanto estiverem carregando.
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