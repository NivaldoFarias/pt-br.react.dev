---
title: flushSync
---

<Pitfall>

Usar `flushSync` é incomum e pode prejudicar o desempenho do seu aplicativo.

</Pitfall>

<Intro>

`flushSync` permite forçar o React a *flush* quaisquer atualizações dentro do *callback* fornecido de forma síncrona. Isso garante que o DOM seja atualizado imediatamente.

```js
flushSync(callback)
```

</Intro>

<InlineToc />

---

## Referência {/*reference*/}

### `flushSync(callback)` {/*flushsync*/}

Chame `flushSync` para forçar o React a *flush* qualquer trabalho pendente e atualizar o DOM de forma síncrona.

```js
import { flushSync } from 'react-dom';

flushSync(() => {
  setSomething(123);
});
```

Na maioria das vezes, `flushSync` pode ser evitado. Use `flushSync` como último recurso.

[Veja mais exemplos abaixo.](#usage)

#### Parâmetros {/*parameters*/}

* `callback`: Uma função. O React chamará imediatamente este *callback* e *flush* quaisquer atualizações que ele contenha de forma síncrona. Ele também pode *flush* quaisquer atualizações pendentes ou Effects, ou atualizações dentro de Effects. Se uma atualização suspender como resultado desta chamada `flushSync`, os *fallbacks* podem ser exibidos novamente.

#### Retornos {/*returns*/}

`flushSync` retorna `undefined`.

#### Ressalvas {/*caveats*/}

*   `flushSync` pode prejudicar significativamente o desempenho. Use com moderação.
*   `flushSync` pode forçar limites de Suspense pendentes a mostrar seu estado `fallback`.
*   `flushSync` pode rodar Effects pendentes e aplicar de forma síncrona quaisquer atualizações que eles contenham antes de retornar.
*   `flushSync` pode *flush* atualizações fora do *callback* quando necessário para *flush* as atualizações dentro do *callback*. Por exemplo, se houver atualizações pendentes de um clique, o React poderá *flush* essas atualizações antes de *flush* as atualizações dentro do *callback*.

---

## Usage {/*usage*/}

### *Flushing* atualizações para integrações de terceiros {/*flushing-updates-for-third-party-integrations*/}

Ao integrar com código de terceiros, como APIs de navegador ou bibliotecas de UI, pode ser necessário forçar o React a *flush* atualizações. Use `flushSync` para forçar o React a *flush* quaisquer <CodeStep step={1}>atualizações de state</CodeStep> dentro do *callback* de forma síncrona:

```js [[1, 2, "setSomething(123)"]]
flushSync(() => {
  setSomething(123);
});
// Nesta linha, o DOM é atualizado.
```

Isso garante que, no momento em que a próxima linha de código for executada, o React já tenha atualizado o DOM.

**Usar `flushSync` é incomum e usá-lo com frequência pode prejudicar significativamente o desempenho do seu aplicativo.** Se o seu aplicativo usar apenas APIs do React e não se integrar a bibliotecas de terceiros, `flushSync` deve ser desnecessário.

No entanto, pode ser útil para integrar com código de terceiros, como APIs de navegador.

Algumas APIs de navegador esperam que os resultados dentro de *callbacks* sejam gravados no DOM de forma síncrona, até o final do *callback*, para que o navegador possa fazer algo com o DOM renderizado. Na maioria dos casos, o React lida com isso automaticamente. Mas, em alguns casos, pode ser necessário forçar uma atualização síncrona.

Por exemplo, a API `onbeforeprint` do navegador permite que você altere a página imediatamente antes que a caixa de diálogo de impressão seja aberta. Isso é útil para aplicar estilos de impressão personalizados que permitem que o documento seja exibido melhor para impressão. No exemplo abaixo, você usa `flushSync` dentro do *callback* `onbeforeprint` para "flushear" imediatamente o state do React para o DOM. Então, quando a caixa de diálogo de impressão é aberta, `isPrinting` exibe "yes":

<Sandpack>

```js src/App.js active
import { useState, useEffect } from 'react';
import { flushSync } from 'react-dom';

export default function PrintApp() {
  const [isPrinting, setIsPrinting] = useState(false);

  useEffect(() => {
    function handleBeforePrint() {
      flushSync(() => {
        setIsPrinting(true);
      })
    }

    function handleAfterPrint() {
      setIsPrinting(false);
    }

    window.addEventListener('beforeprint', handleBeforePrint);
    window.addEventListener('afterprint', handleAfterPrint);
    return () => {
      window.removeEventListener('beforeprint', handleBeforePrint);
      window.removeEventListener('afterprint', handleAfterPrint);
    }
  }, []);

  return (
    <>
      <h1>isPrinting: {isPrinting ? 'yes' : 'no'}</h1>
      <button onClick={() => window.print()}>
        Print
      </button>
    </>
  );
}
```

</Sandpack>

Sem `flushSync`, a caixa de diálogo de impressão exibirá `isPrinting` como "no". Isso ocorre porque o React agrupa as atualizações de forma assíncrona e a caixa de diálogo de impressão é exibida antes que o state seja atualizado.

<Pitfall>

`flushSync` pode prejudicar significativamente o desempenho e pode, inesperadamente, forçar limites de Suspense pendentes a mostrar seu estado *fallback*.

Na maioria das vezes, `flushSync` pode ser evitado, então use `flushSync` como último recurso.

</Pitfall>
```