---
title: prefetchDNS
---

<Intro>

`prefetchDNS` permite que você procure ansiosamente o IP de um servidor do qual você espera carregar recursos.

```js
prefetchDNS("https://example.com");
```

</Intro>

<InlineToc />

---

## Referência {/*reference*/}

### `prefetchDNS(href)` {/*prefetchdns*/}

Para procurar um host, chame a função `prefetchDNS` de `react-dom`.

```js
import { prefetchDNS } from 'react-dom';

function AppRoot() {
  prefetchDNS("https://example.com");
  // ...
}

```

[Veja mais exemplos abaixo.](#usage)

A função prefetchDNS fornece ao navegador uma dica de que ele deve procurar o endereço IP de um determinado servidor. Se o navegador optar por fazê-lo, isso pode acelerar o carregamento de recursos desse servidor.

#### Parâmetros {/*parameters*/}

* `href`: uma string. A URL do servidor ao qual você deseja se conectar.

#### Retorna {/*returns*/}

`prefetchDNS` não retorna nada.

#### Ressalvas {/*caveats*/}

* Múltiplas chamadas para `prefetchDNS` com o mesmo servidor têm o mesmo efeito de uma única chamada.
* No navegador, você pode chamar `prefetchDNS` em qualquer situação: ao renderizar um componente, em um Effect, em um manipulador de eventos (event handler) etc.
* Em renderização do lado do servidor (server-side rendering) ou ao renderizar Componentes de Servidor (Server Components), `prefetchDNS` só tem efeito se você o chamar durante a renderização de um componente ou em um contexto assíncrono originário da renderização de um componente. Quaisquer outras chamadas serão ignoradas.
* Se você souber quais recursos específicos precisará, poderá chamar [outras funções](/reference/react-dom/#resource-preloading-apis) em vez disso, que começarão a carregar os recursos imediatamente.
* Não há benefício em prefetch o mesmo servidor do qual a própria página da web é hospedada, porque ele já foi procurado quando a dica seria dada.
* Em comparação com [`preconnect`](/reference/react-dom/preconnect), `prefetchDNS` pode ser melhor se você estiver se conectando especulativamente a um grande número de domínios, caso em que a sobrecarga de preconexões pode ser maior do que o benefício.

---

## Uso {/*usage*/}

### Prefetching DNS ao renderizar {/*prefetching-dns-when-rendering*/}

Chame `prefetchDNS` ao renderizar um componente se você souber que seus filhos carregarão recursos externos desse host.

```js
import { prefetchDNS } from 'react-dom';

function AppRoot() {
  prefetchDNS("https://example.com");
  return ...;
}
```

### Prefetching DNS em um manipulador de eventos (event handler) {/*prefetching-dns-in-an-event-handler*/}

Chame `prefetchDNS` em um manipulador de eventos (event handler) antes de transitar para uma página ou estado onde recursos externos serão necessários. Isso inicia o processo antes do que se você o chamasse durante a renderização da nova página ou estado.

```js
import { prefetchDNS } from 'react-dom';

function CallToAction() {
  const onClick = () => {
    prefetchDNS('http://example.com');
    startWizard();
  }
  return (
    <button onClick={onClick}>Start Wizard</button>
  );
}
```
```