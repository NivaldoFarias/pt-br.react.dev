---
title: "<title>"
---

<Intro>

O [componente `<title>` nativo do navegador](https://developer.mozilla.org/pt-BR/docs/Web/HTML/Element/title) permite que você especifique o título do documento.

```js
<title>My Blog</title>
```

</Intro>

<InlineToc />

---

## Referência {/*reference*/}

### `<title>` {/*title*/}

Para especificar o título do documento, renderize o [componente `<title>` nativo do navegador](https://developer.mozilla.org/pt-BR/docs/Web/HTML/Element/title). Você pode renderizar `<title>` de qualquer componente e o React sempre colocará o elemento DOM correspondente no `head` do documento.

```js
<title>My Blog</title>
```

[Veja mais exemplos abaixo.](#usage)

#### Props {/*props*/}

`<title>` suporta todas as [props de elemento comuns.](/reference/react-dom/components/common#props)

* `children`: `<title>` aceita apenas texto como filho. Este texto se tornará o título do documento. Você também pode passar seus próprios componentes, desde que eles renderizem apenas texto.

#### Comportamento especial de renderização {/*special-rendering-behavior*/}

O React sempre colocará o elemento DOM correspondente ao componente `<title>` dentro do `<head>` do documento, independentemente de onde na árvore do React ele for renderizado. O `<head>` é o único lugar válido para `<title>` existir dentro do DOM, mas é conveniente e mantém as coisas compostas se um componente representando uma página específica puder renderizar seu próprio `<title>`.

Existem duas exceções a isso:
* Se `<title>` estiver dentro de um componente `<svg>`, então não há comportamento especial, porque nesse contexto ele não representa o título do documento, mas sim uma [anotação de acessibilidade para aquele gráfico SVG](https://developer.mozilla.org/pt-BR/docs/Web/SVG/Element/title).
* Se o `<title>` tiver uma prop [`itemProp`](https://developer.mozilla.org/pt-BR/docs/Web/HTML/Global_attributes/itemprop), não há comportamento especial, porque, nesse caso, ele não representa o título do documento, mas sim metadados sobre uma parte específica da página.

<Pitfall>

Renderize apenas um único `<title>` por vez. Se mais de um componente renderizar uma tag `<title>` ao mesmo tempo, o React colocará todos esses títulos no head do documento. Quando isso acontece, o comportamento dos navegadores e mecanismos de pesquisa é indefinido.

</Pitfall>

---

## Uso {/*usage*/}

### Definir o título do documento {/*set-the-document-title*/}

Renderize o componente `<title>` de qualquer componente com texto como seus filhos. O React colocará um nó DOM `<title>` no `<head>` do documento.

<SandpackWithHTMLOutput>

```js src/App.js active
import ShowRenderedHTML from './ShowRenderedHTML.js';

export default function ContactUsPage() {
  return (
    <ShowRenderedHTML>
      <title>My Site: Contact Us</title>
      <h1>Contact Us</h1>
      <p>Email us at support@example.com</p>
    </ShowRenderedHTML>
  );
}
```

</SandpackWithHTMLOutput>

### Usar variáveis no título {/*use-variables-in-the-title*/}

Os filhos do componente `<title>` devem ser uma única string de texto. (Ou um único número ou um único objeto com um método `toString`.) Pode não ser óbvio, mas usar chaves JSX como esta:

```js
<title>Results page {pageNumber}</title> // 🔴 Problem: This is not a single string
```

... na verdade, faz com que o componente `<title>` receba um array de dois elementos como seus filhos (a string `"Results page"` e o valor de `pageNumber`). Isso causará um erro. Em vez disso, use a interpolação de string para passar uma única string para `<title>`:

```js
<title>{`Results page ${pageNumber}`}</title>
```
```