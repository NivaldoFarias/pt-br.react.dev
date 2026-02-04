---
title: "Descontinuação do Create React App"
author: Matt Carroll e Ricky Hanlon
date: 2025/02/14
description: Hoje, estamos descontinuando o Create React App para novos aplicativos e incentivando os aplicativos existentes a migrar para um framework ou para uma ferramenta de build como Vite, Parcel ou RSBuild. Também estamos fornecendo documentação para quando um framework não é adequado para seu projeto, você deseja criar seu próprio framework ou apenas deseja aprender como o React funciona construindo um aplicativo React do zero.
---

14 de fevereiro de 2025 por [Matt Carroll](https://twitter.com/mattcarrollcode) e [Ricky Hanlon](https://bsky.app/profile/ricky.fm)

---

<Intro>

Hoje, estamos descontinuando o [Create React App](https://create-react-app.dev/) para novos aplicativos e incentivando os aplicativos existentes a migrar para um [framework](#how-to-migrate-to-a-framework), ou para [migrar para uma ferramenta de build](#how-to-migrate-to-a-build-tool) como Vite, Parcel ou RSBuild.

Também estamos fornecendo documentação para quando um framework não é adequado para seu projeto, você deseja criar seu próprio framework ou apenas deseja aprender como o React funciona [construindo um aplicativo React do zero](/learn/build-a-react-

## Por que recomendamos frameworks {/*why-we-recommend-frameworks*/}

Embora você pudesse resolver todas essas partes sozinho em uma ferramenta de build como Create React App, Vite ou Parcel, é difícil fazer isso bem. Assim como o Create React App integrou várias ferramentas de build, você precisa de uma ferramenta para integrar todos esses recursos e oferecer a melhor experiência aos usuários.

Essa categoria de ferramentas que integra ferramentas de build, renderização, roteamento, busca de dados e divisão de código é conhecida como "frameworks" — ou, se você preferir chamar o próprio React de framework, pode chamá-los de "metaframeworks".

Frameworks impõem algumas opiniões sobre a estruturação do seu aplicativo para fornecer uma experiência de usuário muito melhor, da mesma forma que as ferramentas de build impõem algumas opiniões para facilitar o uso das ferramentas. É por isso que começamos a recomendar frameworks como [Next.js](https://nextjs.org/), [React Router](https://reactrouter.com/) e [Expo](https://expo.dev/) para novos projetos.

Frameworks fornecem a mesma experiência de início rápido que o Create React App, mas também oferecem soluções para problemas que os usuários precisam resolver de qualquer maneira em aplicativos de produção reais.

<DeepDive>

#### A renderização do lado do servidor é opcional {/*server-rendering-is-optional*/}

Os frameworks que recomendamos oferecem a opção de criar um aplicativo de [renderização do lado do cliente (CSR)](https://developer.mozilla.org/pt-BR/docs/Glossary/CSR).

Em alguns casos, CSR é a escolha certa para uma página, mas muitas vezes não é. Mesmo que a maior parte do seu aplicativo seja do lado do cliente, muitas vezes há páginas individuais que podem se beneficiar de recursos de renderização do lado do servidor, como [geração de site estático (SSG)](https://developer.mozilla.org/pt-BR/docs/Glossary/SSG) ou [renderização do lado do servidor (SSR)](https://developer.mozilla.org/pt-BR/docs/Glossary/SSR), por exemplo, uma página de Termos de Serviço ou documentação.

A renderização do lado do servidor geralmente envia menos JavaScript para o cliente e um documento HTML completo, o que produz uma [Primeira Pintura de Conteúdo (FCP)](https://web.dev/articles/fcp) mais rápida, reduzindo o [Tempo Total de Bloqueio (TBD)](https://web.dev/articles/tbt), o que também pode diminuir o [Tempo de Interação para a Próxima Pintura (INP)](https://web.dev/articles/inp). É por isso que a [equipe do Chrome tem incentivado](https://web.dev/articles/rendering-on-the-web) os desenvolvedores a considerar a renderização estática ou do lado do servidor em vez de uma abordagem totalmente do lado do cliente para alcançar o melhor desempenho possível.

Existem compromissos no uso de um servidor, e ele nem sempre é a melhor opção para todas as páginas. Gerar páginas no servidor incorre em custos adicionais e leva tempo para gerar, o que pode aumentar o [Tempo para o Primeiro Byte (TTFB)](https://web.dev/articles/ttfb). Os aplicativos com melhor desempenho conseguem escolher a estratégia de renderização correta em uma base por página, com base nos compromissos de cada estratégia.

Frameworks oferecem a opção de usar um servidor em qualquer página, se você quiser, mas não o forçam a usar um servidor. Isso permite que você escolha a estratégia de renderização correta para cada página do seu aplicativo.

#### E os Server Components? {/*server-components*/}

Os frameworks que recomendamos também incluem suporte para React Server Components.

Server Components ajudam a resolver esses problemas movendo o roteamento e a busca de dados para o servidor, e permitindo que a divisão de código seja feita para componentes do cliente com base nos dados que você renderiza, em vez de apenas na rota renderizada, e reduzindo a quantidade de JavaScript enviado para a melhor [sequência de carregamento](https://www.patterns.dev/vanilla/loading-sequence) possível.

Server Components não exigem um servidor. Eles podem ser executados no momento da compilação no seu servidor de CI para criar um aplicativo de site estático (SSG), em tempo de execução em um servidor web para um aplicativo renderizado do lado do servidor (SSR).

Veja [Introdução aos React Server Components com tamanho zero de bundle](/blog/2020/12/21/data-fetching-with-react-server-components) e [a documentação](/reference/rsc/server-components) para mais informações.

</DeepDive>

<Note>

#### A renderização do lado do servidor não é apenas para SEO {/*server-rendering-is-not-just-for-seo*/}

Um mal-entendido comum é que a renderização do lado do servidor é apenas para [SEO](https://developer.mozilla.org/pt-BR/docs/Glossary/SEO).

Embora a renderização do lado do servidor possa melhorar o SEO, ela também melhora o desempenho, reduzindo a quantidade de JavaScript que o usuário precisa baixar e analisar antes que ele possa ver o conteúdo na tela.

É por isso que a equipe do Chrome [tem incentivado](https://web.dev/articles/rendering-on-the-web) os desenvolvedores a considerar a renderização estática ou do lado do servidor em vez de uma abordagem totalmente do lado do cliente para alcançar o melhor desempenho possível.

</Note>

---

_Agradecemos a [Dan Abramov](https://bsky.app/profile/danabra.mov) por criar o Create React App, e a [Joe Haddad](https://github.com/Timer), [Ian Schmitz](https://github.com/ianschmitz), [Brody McKee](https://github.com/mrmckeb) e [muitos outros](https://github.com/facebook/create-react-app/graphs/contributors) por manterem o Create React App ao longo dos anos. Agradecemos a [Brooks Lybrand](https://bsky.app/profile/brookslybrand.bsky.social), [Dan Abramov](https://bsky.app/profile/danabra.mov), [Devon Govett](https://bsky.app/profile/devongovett.bsky.social), [Eli White](https://x.com/Eli_White), [Jack Herrington](https://bsky.app/profile/jherr.dev), [Joe Savona](https://x.com/en_JS), [Lauren Tan](https://bsky.app/profile/no.lol), [Lee Robinson](https://x.com/leeerob), [Mark Erikson](https://bsky.app/profile/acemarke.dev), [Ryan Florence](https://x.com/ryanflorence), [Sophie Alpert](https://bsky.app/profile/sophiebits.com), [Tanner Linsley](https://bsky.app/profile/tannerlinsley.com) e [Theo Browne](https://x.com/theo) por revisarem e fornecerem feedback sobre esta postagem._