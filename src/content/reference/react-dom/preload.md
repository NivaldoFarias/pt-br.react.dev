---
title: preload
---

<Note>

[Frameworks baseados em React](/learn/start-a-new-react-project) frequentemente tratam o carregamento de recursos para você, então talvez você não precise chamar esta API você mesmo. Consulte a documentação do seu framework para detalhes.

</Note>

<Intro>

`preload` permite que você busque ansiosamente um recurso, como uma stylesheet, fonte ou script externo, que você espera usar.

```js
preload("https://example.com/font.woff2", {as: "font"});
```

</Intro>

<InlineToc />

---

## Referência {/*reference*/}

### `preload(href, options)` {/*preload*/}

Para fazer o preload de um recurso, chame a função `preload` de `react-dom`.

```js
import { preload } from 'react-dom';

function AppRoot() {
  preload("https://example.com/font.woff2", {as: "font"});
  // ...
}

```

[Veja mais exemplos abaixo.](#usage)

A função `preload` fornece ao navegador uma dica de que ele deve começar a baixar o recurso fornecido, o que pode economizar tempo.

#### Parâmetros {/*parameters*/}

* `href`: uma string. A URL do recurso que você deseja baixar.
* `options`: um objeto. Ele contém as seguintes propriedades:
  *  `as`: uma string obrigatória. O tipo de recurso. Seus [valores possíveis](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/link#as) são `audio`, `document`, `embed`, `fetch`, `font`, `image`, `object`, `script`, `style`, `track`, `video`, `worker`.
  *  `crossOrigin`: uma string. A [política CORS](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/crossorigin) a ser usada. Seus valores possíveis são `anonymous` e `use-credentials`. É obrigatório quando `as` é definido como `"fetch"`.
  *  `referrerPolicy`: uma string. O [cabeçalho Referrer](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/link#referrerpolicy) a ser enviado durante a busca. Seus valores possíveis são `no-referrer-when-downgrade` (o padrão), `no-referrer`, `origin`, `origin-when-cross-origin` e `unsafe-url`.
  *  `integrity`: uma string. Um hash criptográfico do recurso, para [verificar sua autenticidade](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity).
  *  `type`: uma string. O tipo MIME do recurso.
  *  `nonce`: uma string. Um [nonce](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/nonce) criptográfico para permitir o recurso ao usar uma Content Security Policy estrita. 
  *  `fetchPriority`: uma string. Sugere uma prioridade relativa para buscar o recurso. Os valores possíveis são `auto` (o padrão), `high` e `low`.
  *  `imageSrcSet`: uma string. Para uso somente com `as: "image"`. Especifica o [source set da imagem](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images).
  *  `imageSizes`: uma string. Para uso somente com `as: "image"`. Especifica os [tamanhos da imagem](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images).

#### Retorna {/*returns*/}

`preload` não retorna nada.

#### Ressalvas {/*caveats*/}

* Múltiplas chamadas equivalentes para `preload` têm o mesmo efeito que uma única chamada. As chamadas para `preload` são consideradas equivalentes de acordo com as seguintes regras:
  * Duas chamadas são equivalentes se tiverem o mesmo `href`, exceto:
  * Se `as` estiver definido como `image`, duas chamadas serão equivalentes se tiverem o mesmo `href`, `imageSrcSet` e `imageSizes`.
* No navegador, você pode chamar `preload` em qualquer situação: ao renderizar um componente, em um Effect, em um event handler, e assim por diante.
* Na renderização do lado do servidor ou ao renderizar Server Components, `preload` só tem efeito se você o chamar durante a renderização de um componente ou em um contexto assíncrono originário da renderização de um componente. Quaisquer outras chamadas serão ignoradas.

---

## Uso {/*usage*/}

### Preloading ao renderizar {/*preloading-when-rendering*/}

Chame `preload` ao renderizar um componente se você souber que ele ou seus filhos usarão um recurso específico.

<Recipes titleText="Examples of preloading">

#### Preloading de um script externo {/*preloading-an-external-script*/}

```js
import { preload } from 'react-dom';

function AppRoot() {
  preload("https://example.com/script.js", {as: "script"});
  return ...;
}
```

Se você deseja que o navegador comece a executar o script imediatamente (em vez de apenas baixá-lo), use [`preinit`](/reference/react-dom/preinit`) em vez disso. Se você deseja carregar um módulo ESM, use [`preloadModule`](/reference/react-dom/preloadModule`).

<Solution />

#### Preloading de uma stylesheet {/*preloading-a-stylesheet*/}

```js
import { preload } from 'react-dom';

function AppRoot() {
  preload("https://example.com/style.css", {as: "style"});
  return ...;
}
```

Se você deseja que a stylesheet seja inserida no documento imediatamente (o que significa que o navegador começará a analisá-la imediatamente, em vez de apenas baixá-la), use [`preinit`](/reference/react-dom/preinit`) em vez disso.

<Solution />

#### Preloading de uma fonte {/*preloading-a-font*/}

```js
import { preload } from 'react-dom';

function AppRoot() {
  preload("https://example.com/style.css", {as: "style"});
  preload("https://example.com/font.woff2", {as: "font"});
  return ...;
}
```

Se você fizer o preload de uma stylesheet, é inteligente também fazer o preload de todas as fontes às quais a stylesheet se refere. Dessa forma, o navegador pode começar a baixar a fonte antes que ela tenha baixado e analisado a stylesheet.

<Solution />

#### Preloading de uma imagem {/*preloading-an-image*/}

```js
import { preload } from 'react-dom';

function AppRoot() {
  preload("/banner.png", {
    as: "image",
    imageSrcSet: "/banner512.png 512w, /banner1024.png 1024w",
    imageSizes: "(max-width: 512px) 512px, 1024px",
  });
  return ...;
}
```

Ao fazer o preload de uma imagem, as opções `imageSrcSet` e `imageSizes` ajudam o navegador a [pegar a imagem com o tamanho correto para o tamanho da tela](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images).

<Solution />

</Recipes>

### Preloading em um event handler {/*preloading-in-an-event-handler*/}

Chame `preload` em um event handler antes de fazer a transição para uma página ou state onde os recursos externos serão necessários. Isso inicia o processo mais cedo do que se você o chamar durante a renderização da nova página ou state.

```js
import { preload } from 'react-dom';

function CallToAction() {
  const onClick = () => {
    preload("https://example.com/wizardStyles.css", {as: "style"});
    startWizard();
  }
  return (
    <button onClick={onClick}>Start Wizard</button>
  );
}
```
