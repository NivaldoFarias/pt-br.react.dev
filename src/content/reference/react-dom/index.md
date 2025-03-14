---
title: React DOM APIs
---

<Intro>

O pacote `react-dom` contém métodos que são suportados apenas para aplicações web (que são executadas no ambiente DOM do navegador). Eles não são suportados para o React Native.

</Intro>

---

## APIs {/*apis*/}

Essas APIs podem ser importadas em seus componentes. Elas são raramente usadas:

* [`createPortal`](/reference/react-dom/createPortal) permite renderizar componentes filhos em uma parte diferente da árvore DOM.
* [`flushSync`](/reference/react-dom/flushSync) permite forçar o React a atualizar o state e atualizar o DOM sincronamente.

## Resource Preloading APIs {/*resource-preloading-apis*/}

Estas APIs podem ser usadas para tornar apps mais rápidas ao pré-carregar recursos como `scripts`, `stylesheets`, e fontes assim que souber que você precisa delas, por exemplo, antes de navegar para outra página onde os recursos serão usados.

[React-based frameworks](/learn/start-a-new-react-project) frequentemente lidam com o carregamento de recursos para você, então você pode não precisar chamar essas APIs sozinho. Consulte a documentação do seu framework para detalhes.

* [`prefetchDNS`](/reference/react-dom/prefetchDNS) permite que você pré-busque o endereço IP de um nome de domínio DNS que você espera se conectar.
* [`preconnect`](/reference/react-dom/preconnect) permite que você se conecte a um servidor do qual você espera solicitar recursos, mesmo que você ainda não saiba quais recursos você vai precisar.
* [`preload`](/reference/react-dom/preload) permite que você busque uma `stylesheet`, fonte, imagem ou `script` externo que você espera usar.
* [`preloadModule`](/reference/react-dom/preloadModule) permite que você busque um módulo ESM que você espera usar.
* [`preinit`](/reference/react-dom/preinit) permite que você busque e avalie um `script` externo ou busque e insira uma `stylesheet`.
* [`preinitModule`](/reference/react-dom/preinitModule) permite que você busque e avalie um módulo ESM.

---

## Entry points {/*entry-points*/}

O pacote `react-dom` fornece dois pontos de entrada adicionais:

* [`react-dom/client`](/reference/react-dom/client) contém APIs para renderizar componentes do React no cliente (no navegador).
* [`react-dom/server`](/reference/react-dom/server) contém APIs para renderizar componentes do React no servidor.

---

## APIs Removidas {/*removed-apis*/}

Essas APIs foram removidas no React 19:

* [`findDOMNode`](https://18.react.dev/reference/react-dom/findDOMNode): veja [alternativas](https://18.react.dev/reference/react-dom/findDOMNode#alternatives).
* [`hydrate`](https://18.react.dev/reference/react-dom/hydrate): use [`hydrateRoot`](/reference/react-dom/client/hydrateRoot) em vez disso.
* [`render`](https://18.react.dev/reference/react-dom/render): use [`createRoot`](/reference/react-dom/client/createRoot) em vez disso.
* [`unmountComponentAtNode`](/reference/react-dom/unmountComponentAtNode): use [`root.unmount()`](/reference/react-dom/client/createRoot#root-unmount) em vez disso.
* [`renderToNodeStream`](https://18.react.dev/reference/react-dom/server/renderToNodeStream): use as APIs de [`react-dom/server`](/reference/react-dom/server) em vez disso.
* [`renderToStaticNodeStream`](https://18.react.dev/reference/react-dom/server/renderToStaticNodeStream): use as APIs de [`react-dom/server`](/reference/react-dom/server) em vez disso.