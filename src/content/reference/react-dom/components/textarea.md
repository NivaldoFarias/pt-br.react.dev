---
title: "<textarea>"
---

<Intro>

O [componente `<textarea>` integrado do navegador](https://developer.mozilla.org/pt-BR/docs/Web/HTML/Element/textarea) permite que você renderize uma entrada de texto de várias linhas.

```js
<textarea />
```

</Intro>

<InlineToc />

---

## Referência {/*reference*/}

### ``<textarea>`` {/*textarea*/}

Para exibir uma área de texto, renderize o componente [integrado do navegador `textarea`](https://developer.mozilla.org/pt-BR/docs/Web/HTML/Element/textarea).

```js
<textarea name="postContent" />
```

[Veja mais exemplos abaixo.](#usage)

#### Props {/*props*/}

`<textarea>` suporta todas as [props comuns de elementos.](/reference/react-dom/components/common#props)

Você pode [tornar uma área de texto controlada](#controlling-a-text-area-with-a-state-variable) passando uma prop `value`:

* `value`: Uma string. Controla o texto dentro da área de texto.

Quando você passa `value`, você também deve passar um manipulador `onChange` que atualiza o valor passado.

Se seu `<textarea>` é não controlado, você pode passar a prop `defaultValue` em vez disso:

* `defaultValue`: Uma string. Especifica [o valor inicial](#providing-an-initial-value-for-a-text-area) para uma área de texto.

Estas props `<textarea>` são relevantes tanto para áreas de texto controladas quanto para não controladas:

* [`autoComplete`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea#autocomplete): Either `'on'` or `'off'`. Especifica o comportamento de preenchimento automático.
* [`autoFocus`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea#autofocus): Um booleano. Se `true`, o React irá focar no elemento ao montar.
* `children`: `<textarea>` não aceita filhos. Para definir o valor inicial, use `defaultValue`.
* [`cols`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea#cols): Um número. Especifica a largura padrão em média de caracteres. O padrão é `20`.
* [`disabled`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea#disabled): Um booleano. Se `true`, a entrada não será interativa e aparecerá esmaecida.
* [`form`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea#form): Uma string. Especifica o `id` do `<form>` ao qual essa entrada pertence. Se omitido, é o formulário pai mais próximo.
* [`maxLength`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea#maxlength): Um número. Especifica o comprimento máximo do texto.
* [`minLength`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea#minlength): Um número. Especifica o comprimento mínimo do texto.
* [`name`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#name): Uma string. Especifica o nome para esta entrada que é [enviada com o formulário.](#reading-the-textarea-value-when-submitting-a-form)
* `onChange`: Um manipulador de [`Event`](/reference/react-dom/components/common#event-handler) function. Obrigatório para [áreas de texto controladas.](#controlling-a-text-area-with-a-state-variable) Dispara imediatamente quando o valor da entrada é alterado pelo usuário (por exemplo, é disparado a cada pressionamento de tecla). Comporta-se como o evento `input` do navegador.
* `onChangeCapture`: Uma versão de `onChange` que dispara na [fase de captura.](/learn/responding-to-events#capture-phase-events)
* [`onInput`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/input_event): Um manipulador de [`Event`](/reference/react-dom/components/common#event-handler) function. Dispara imediatamente quando o valor é alterado pelo usuário. Por razões históricas, no React é idiomático usar `onChange` no lugar que funciona de forma similar.
* `onInputCapture`: Uma versão de `onInput` que dispara na [fase de captura.](/learn/responding-to-events#capture-phase-events)
* [`onInvalid`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/invalid_event): Um manipulador de [`Event`](/reference/react-dom/components/common#event-handler) function. Dispara se uma entrada falha na validação ao enviar o formulário. Diferente do evento `invalid` integrado, o evento `onInvalid` do React é propagado.
* `onInvalidCapture`: Uma versão de `onInvalid` que dispara na [fase de captura.](/learn/responding-to-events#capture-phase-events)
* [`onSelect`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLTextAreaElement/select_event): Um manipulador de [`Event`](/reference/react-dom/components/common#event-handler) function. Dispara após a seleção dentro do `<textarea>` mudar. React estende o evento `onSelect` para também disparar para seleção vazia e em edições (o que pode afetar a seleção).
* `onSelectCapture`: Uma versão de `onSelect` que dispara na [fase de captura.](/learn/responding-to-events#capture-phase-events)
* [`placeholder`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea#placeholder): Uma string. Exibido em uma cor esmaecida quando o valor da área de texto está vazio.
* [`readOnly`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea#readonly): Um booleano. Se `true`, a área de texto não é editável pelo usuário.
* [`required`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea#required): Um booleano. Se `true`, o valor deve ser fornecido para o envio do formulário.
* [`rows`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea#rows): Um número. Especifica a altura padrão em média de caracteres. O padrão é `2`.
* [`wrap`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea#wrap): Either `'hard'`, `'soft'`, or `'off'`. Especifica como o texto deve ser ajustado ao enviar um formulário.

#### Ressalvas {/*caveats*/}

- Passar filhos como `<textarea>algo</textarea>` não é permitido. [Use `defaultValue` para o conteúdo inicial.](#providing-an-initial-value-for-a-text-area)
- Se uma área de texto recebe uma prop `value` string, ela será [tratada como controlada.](#controlling-a-text-area-with-a-state-variable)
- Uma área de texto não pode ser controlada e não controlada ao mesmo tempo.
- Uma área de texto não pode alternar entre ser controlada ou não controlada ao longo de sua vida útil.
- Toda área de texto controlada precisa de um manipulador de evento `onChange` que atualiza de forma síncrona seu valor de backup.

---

## Uso {/*usage*/}

### Exibindo uma área de texto {/*displaying-a-text-area*/}

Renderize `<textarea>` para exibir uma área de texto. Você pode especificar seu tamanho padrão com os atributos [`rows`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea#rows) e [`cols`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea#cols), mas por padrão o usuário poderá redimensioná-lo. Para desabilitar o redimensionamento, você pode especificar `resize: none` no CSS.

<Sandpack>

```js
export default function NewPost() {
  return (
    <label>
      Escreva sua publicação:
      <textarea name="postContent" rows={4} cols={40} />
    </label>
  );
}
```

```css
input { margin-left: 5px; }
textarea { margin-top: 10px; }
label { margin: 10px; }
label, textarea { display: block; }
```

</Sandpack>

---

### Fornecendo um rótulo para uma área de texto {/*providing-a-label-for-a-text-area*/}

Normalmente, você colocará cada `<textarea>` dentro de uma tag [`<label>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/label). Isso informa ao navegador que este rótulo está associado a essa área de texto. Quando o usuário clica no rótulo, o navegador focará na área de texto. Também é essencial para acessibilidade: um leitor de tela anunciará a legenda do rótulo quando o usuário focar na área de texto.

Se você não puder aninhar `<textarea>` em um `<label>`, associe-os passando o mesmo ID para `<textarea id>` e [`<label htmlFor>`.](https://developer.mozilla.org/en-US/docs/Web/API/HTMLLabelElement/htmlFor) Para evitar conflitos entre instâncias de um componente, gere tal ID com [`useId`.](/reference/react/useId)

<Sandpack>

```js
import { useId } from 'react';

export default function Form() {
  const postTextAreaId = useId();
  return (
    <>
      <label htmlFor={postTextAreaId}>
        Escreva sua publicação:
      </label>
      <textarea
        id={postTextAreaId}
        name="postContent"
        rows={4}
        cols={40}
      />
    </>
  );
}
```

```css
input { margin: 5px; }
```

</Sandpack>

---

### Fornecendo um valor inicial para uma área de texto {/*providing-an-initial-value-for-a-text-area*/}

Você pode opcionalmente especificar o valor inicial para a área de texto. Passe-o como a string `defaultValue`.

<Sandpack>

```js
export default function EditPost() {
  return (
    <label>
      Edite sua publicação:
      <textarea
        name="postContent"
        defaultValue="Eu realmente gostei de andar de bicicleta ontem!"
        rows={4}
        cols={40}
      />
    </label>
  );
}
```

```css
input { margin-left: 5px; }
textarea { margin-top: 10px; }
label { margin: 10px; }
label, textarea { display: block; }
```

</Sandpack>

<Pitfall>

Ao contrário do HTML, passar texto inicial como `<textarea>Algum conteúdo</textarea>` não é suportado.

</Pitfall>

---

### Lendo o valor da área de texto ao enviar um formulário {/*reading-the-textarea-value-when-submitting-a-form*/}

Adicione um [`<form>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form) ao redor de seu textarea com um [`<button type="submit">`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/button) dentro. Ele chamará seu manipulador de evento `<form onSubmit>`. Por padrão, o navegador enviará os dados do formulário para a URL atual e atualizará a página. Você pode substituir esse comportamento chamando `e.preventDefault()`. Leia os dados do formulário com [`new FormData(e.target)`](https://developer.mozilla.org/en-US/docs/Web/API/FormData).
<Sandpack>

```js
export default function EditPost() {
  function handleSubmit(e) {
    // Impede que o navegador recarregue a página
    e.preventDefault();

    // Leia os dados do formulário
    const form = e.target;
    const formData = new FormData(form);

    // Você pode passar formData como um corpo de busca diretamente:
    fetch('/some-api', { method: form.method, body: formData });

    // Ou você pode trabalhar com ele como um objeto simples:
    const formJson = Object.fromEntries(formData.entries());
    console.log(formJson);
  }

  return (
    <form method="post" onSubmit={handleSubmit}>
      <label>
        Título da publicação: <input name="postTitle" defaultValue="Biking" />
      </label>
      <label>
        Edite sua publicação:
        <textarea
          name="postContent"
          defaultValue="Eu realmente gostei de andar de bicicleta ontem!"
          rows={4}
          cols={40}
        />
      </label>
      <hr />
      <button type="reset">Redefinir edições</button>
      <button type="submit">Salvar publicação</button>
    </form>
  );
}
```

```css
label { display: block; }
input { margin: 5px; }
```

</Sandpack>

<Note>

Dê um `name` ao seu `<textarea>`, por exemplo `<textarea name="postContent" />`. O `name` que você especificou será usado como uma chave nos dados do formulário, por exemplo, `{ postContent: "Sua publicação" }`.

</Note>

<Pitfall>

Por padrão, *qualquer* `<button>` dentro de um `<form>` irá enviá-lo. Isso pode ser surpreendente! Se você tem seu próprio componente `Button` React customizado, considere retornar [`<button type="button">`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/button) em vez de `<button>`. Então, para ser explícito, use `<button type="submit">` para botões que *são* para enviar o formulário.

</Pitfall>

---

### Controlando uma área de texto com uma variável de estado {/*controlling-a-text-area-with-a-state-variable*/}

Uma área de texto como `<textarea />` é *não controlada.* Mesmo se você [passar um valor inicial](#providing-an-initial-value-for-a-text-area) como `<textarea defaultValue="Texto inicial" />`, seu JSX apenas especifica o valor inicial, não o valor agora.

**Para renderizar uma área de texto _controlada_, passe a prop `value` para ela.** O React forçará a área de texto a sempre ter o `value` que você passou. Normalmente, você controlará uma área de texto declarando uma [variável de state:](/reference/react/useState)

```js {2,6,7}
function NewPost() {
  const [postContent, setPostContent] = useState(''); // Declare a state variable...
  // ...
  return (
    <textarea
      value={postContent} // ...force the input's value to match the state variable...
      onChange={e => setPostContent(e.target.value)} // ... and update the state variable on any edits!
    />
  );
}
```

Isso é útil se você deseja renderizar novamente alguma parte da UI em resposta a cada pressionamento de tecla.

<Sandpack>

```js
import { useState } from 'react';
import MarkdownPreview from './MarkdownPreview.js';

export default function MarkdownEditor() {
  const [postContent, setPostContent] = useState('_Olá,_ **Markdown**!');
  return (
    <>
      <label>
        Digite algum markdown:
        <textarea
          value={postContent}
          onChange={e => setPostContent(e.target.value)}
        />
      </label>
      <hr />
      <MarkdownPreview markdown={postContent} />
    </>
  );
}
```

```js src/MarkdownPreview.js
import { Remarkable } from 'remarkable';

const md = new Remarkable();

export default function MarkdownPreview({ markdown }) {
  const renderedHTML = md.render(markdown);
  return <div dangerouslySetInnerHTML={{__html: renderedHTML}} />;
}
```

```json package.json
{
  "dependencies": {
    "react": "latest",
    "react-dom": "latest",
    "react-scripts": "latest",
    "remarkable": "2.0.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```css
textarea { display: block; margin-top: 5px; margin-bottom: 10px; }
```

</Sandpack>

<Pitfall>

**Se você passar `value` sem `onChange`, será impossível digitar na área de texto.** Quando você controla uma área de texto passando algum `value` para ela, você a *força* a sempre ter o valor que você passou. Então, se você passar uma variável de estado como um `value`, mas esquecer de atualizar essa variável de estado de forma síncrona durante o manipulador de evento `onChange`, o React reverterá a área de texto após cada pressionamento de tecla de volta para o `value` que você especificou.

</Pitfall>

---

## Solução de problemas {/*troubleshooting*/}

### Minha área de texto não atualiza quando eu digito nela {/*my-text-area-doesnt-update-when-i-type-into-it*/}

Se você renderizar uma área de texto com `value` mas sem `onChange`, você verá um erro no console:

```js
// 🔴 Bug: controlled text area with no onChange handler
<textarea value={something} />
```

<ConsoleBlock level="error">

Você forneceu uma prop `value` a um campo de formulário sem um manipulador `onChange`. Isso renderizará um campo somente leitura. Se o campo deve ser mutável, use `defaultValue`. Caso contrário, defina `onChange` ou `readOnly`.

</ConsoleBlock>

Como a mensagem de erro sugere, se você só queria [especificar o valor *inicial*,](#providing-an-initial-value-for-a-text-area) passe `defaultValue` em vez disso:

```js
// ✅ Good: uncontrolled text area with an initial value
<textarea defaultValue={something} />
```

Se você quer [controlar esta área de texto com uma variável de estado,](#controlling-a-text-area-with-a-state-variable) especifique um manipulador `onChange`:

```js
// ✅ Good: controlled text area with onChange
<textarea value={something} onChange={e => setSomething(e.target.value)} />
```

Se o valor é intencionalmente somente leitura, adicione uma prop `readOnly` para suprimir o erro:

```js
// ✅ Good: readonly controlled text area without on change
<textarea value={something} readOnly={true} />
```

---

### Meu cursor da área de texto pula para o começo em cada pressionamento de tecla {/*my-text-area-caret-jumps-to-the-beginning-on-every-keystroke*/}

Se você [controlar uma área de texto,](#controlling-a-text-area-with-a-state-variable) você deve atualizar sua variável de estado para o valor da área de texto do DOM durante `onChange`.

Você não pode atualizá-lo para algo diferente de `e.target.value`:

```js
function handleChange(e) {
  // 🔴 Bug: updating an input to something other than e.target.value
  setFirstName(e.target.value.toUpperCase());
}
```

Você também não pode atualizá-lo de forma assíncrona:

```js
function handleChange(e) {
  // 🔴 Bug: updating an input asynchronously
  setTimeout(() => {
    setFirstName(e.target.value);
  }, 100);
}
```

Para corrigir seu código, atualize-o de forma síncrona para `e.target.value`:

```js
function handleChange(e) {
  // ✅ Updating a controlled input to e.target.value synchronously
  setFirstName(e.target.value);
}
```

Se isso não corrigir o problema, é possível que a área de texto seja removida e readicionada do DOM a cada pressionamento de tecla. Isso pode acontecer se você estiver [redefinindo o estado](/learn/preserving-and-resetting-state) acidentalmente a cada re-renderização. Por exemplo, isso pode acontecer se a área de texto ou um de seus pais sempre receber um atributo `key` diferente, ou se você aninhar definições de componentes (o que não é permitido no React e causa a remontagem do componente "interno" a cada renderização).

---

### Eu estou recebendo um erro: "Um componente está mudando uma entrada não controlada para ser controlada" {/*im-getting-an-error-a-component-is-changing-an-uncontrolled-input-to-be-controlled*/}

Se você fornecer um `value` para o componente, ele deve permanecer uma string ao longo de sua vida útil.

Você não pode passar `value={undefined}` primeiro e depois passar `value="alguma string"` porque o React não saberá se você quer que o componente seja não controlado ou controlado. Um componente controlado sempre deve receber um `value` de string, não `null` ou `undefined`.

Se seu `value` está vindo de uma API ou de uma variável de estado, ele pode ser inicializado para `null` ou `undefined`. Nesse caso, ou defina-o para uma string vazia (`''`) inicialmente, ou passe `value={someValue ?? ''}` para garantir que `value` é uma string.
```