---
title: APIs do React DOM do cliente
---

<Intro>

As APIs `react-dom/client` permitem renderizar componentes React no cliente (no navegador). Essas APIs são normalmente usadas no nível superior do seu aplicativo para inicializar sua árvore React. Um [framework](/learn/start-a-new-react-project#production-grade-react-frameworks) pode chamá-las por você. A maioria de seus componentes não precisa importá-las ou usá-las.

</Intro>

---

## APIs do cliente {/*client-apis*/}

* [`createRoot`](/reference/react-dom/client/createRoot) permite criar uma raiz para exibir componentes React dentro de um nó DOM do navegador.
* [`hydrateRoot`](/reference/react-dom/client/hydrateRoot) permite exibir componentes React dentro de um nó DOM do navegador cujo conteúdo HTML foi gerado anteriormente por [`react-dom/server`.](/reference/react-dom/server)

---

## Suporte do navegador {/*browser-support*/}

React suporta todos os navegadores populares, incluindo o Internet Explorer 9 e superior. Alguns polyfills são necessários para navegadores mais antigos, como IE 9 e IE 10.