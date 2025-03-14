---
title: preinitModule
---

<Note>

[Frameworks baseados em React](/learn/start-a-new-react-project) frequentemente lidam com carregamento de recursos para você, então você pode não precisar chamar esta API sozinho. Consulte a documentação do seu framework para detalhes.

</Note>

<Intro>

`preinitModule` permite que você busque e avalie ansiosamente um módulo ESM.

```js
preinitModule("https://example.com/module.js", {as: "script"});
```

</Intro>

<InlineToc />

---

## Referência {/*reference*/}

### `preinitModule(href, options)` {/*preinitmodule*/}

Para preinit um módulo ESM, chame a função `preinitModule` de `react-dom`.

```js
import { preinitModule } from 'react-dom';

function AppRoot() {
  preinitModule("https://example.com/module.js", {as: "script"});
  // ...
}

```

[Veja mais exemplos abaixo.](#usage)

A função `preinitModule` fornece ao navegador uma dica de que ele deve começar a baixar e executar o módulo fornecido, o que pode economizar tempo. Módulos que você faz `preinit` são executados quando terminam o download.

#### Parâmetros {/*parameters*/}

* `href`: uma string. A URL do módulo que você deseja baixar e executar.
* `options`: um objeto. Ele contém as seguintes propriedades:
  *  `as`: uma string requerida. Deve ser `'script'`.
  *  `crossOrigin`: uma string. A [política CORS](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/crossorigin) a ser usada. Seus valores possíveis são `anonymous` e `use-credentials`.
  *  `integrity`: uma string. Um hash criptográfico do módulo, para [verificar sua autenticidade](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity).
  *  `nonce`: uma string. Um [nonce criptográfico para permitir o módulo](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/nonce) ao usar uma Content Security Policy estrita.

#### Retorna {/*returns*/}

`preinitModule` não retorna nada.

#### Ressalvas {/*caveats*/}

* Múltiplas chamadas para `preinitModule` com o mesmo `href` têm o mesmo efeito de uma única chamada.
* No navegador, você pode chamar `preinitModule` em qualquer situação: ao renderizar um componente, em um Effect, em um manipulador de eventos, e assim por diante.
* Na renderização do lado do servidor ou ao renderizar Componentes de Servidor, `preinitModule` só tem efeito se você chamá-lo ao renderizar um componente ou em um contexto assíncrono originário da renderização de um componente. Quaisquer outras chamadas serão ignoradas.

---

## Uso {/*usage*/}

### Pré-carregamento durante a renderização {/*preloading-when-rendering*/}

Chame `preinitModule` ao renderizar um componente se você souber que ele ou seus filhos usarão um módulo específico e você estiver de acordo com o módulo sendo avaliado e, portanto, entrando em vigor imediatamente após ser baixado.

```js
import { preinitModule } from 'react-dom';

function AppRoot() {
  preinitModule("https://example.com/module.js", {as: "script"});
  return ...;
}
```

Se você deseja que o navegador baixe o módulo, mas não o execute imediatamente, use [`preloadModule`](/reference/react-dom/preloadModule) em vez disso. Se você deseja preinit um script que não é um módulo ESM, use [`preinit`](/reference/react-dom/preinit).

### Pré-carregamento em um manipulador de eventos {/*preloading-in-an-event-handler*/}

Chame `preinitModule` em um manipulador de eventos antes de fazer a transição para uma página ou estado onde o módulo será necessário. Isso inicia o processo mais cedo do que se você o chamasse durante a renderização da nova página ou estado.

```js
import { preinitModule } from 'react-dom';

function CallToAction() {
  const onClick = () => {
    preinitModule("https://example.com/module.js", {as: "script"});
    startWizard();
  }
  return (
    <button onClick={onClick}>Start Wizard</button>
  );
}
```
```