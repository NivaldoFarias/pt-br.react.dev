---
title: "Descontinuando Create React App"
author: Matt Carroll and Ricky Hanlon
date: 2025/02/14
description: Hoje, estamos descontinuando o Create React App para novos
  aplicativos e incentivando os aplicativos existentes a migrar para um
  framework ou para uma ferramenta de build como Vite, Parcel ou RSBuild. Também
  estamos fornecendo documentação para quando um framework não for adequado para
  o seu projeto, você quiser construir seu próprio framework ou apenas quiser
  aprender como o React funciona construindo um aplicativo React do zero.
---
February 14, 2025 por [Matt Carroll](https://twitter.com/mattcarrollcode) e [Ricky Hanlon](https://bsky.app/profile/ricky.fm)

---

<Intro>

Hoje, estamos **descontinuando** o [Create React App](https://create-react-app.dev/) para novos aplicativos e incentivando os aplicativos existentes a migrarem para um [framework](#how-to-migrate-to-a-framework) ou a [migrarem para uma ferramenta de build](#how-to-migrate-to-a-build-tool) como Vite, Parcel ou RSBuild.

Também estamos fornecendo documentação para quando um framework não for adequado para seu projeto, você quiser construir seu próprio framework ou apenas quiser aprender como o React funciona [construindo um aplicativo React do zero](/learn/build-a-react-app-from-scratch).

</Intro>

-----

Quando lançamos o Create React App em 2016, não havia uma maneira clara de construir um novo aplicativo React.

Para criar um aplicativo React, você tinha que instalar um monte de ferramentas e conectá-las para suportar recursos básicos como JSX, linting e hot reloading. Isso era muito complicado de fazer corretamente, então a [comunidade](https://github.com/react-boilerplate/react-boilerplate) [criou](https://github.com/kriasoft/react-starter-kit) [boilerplates](https://github.com/petehunt/react-boilerplate) para [configurações](https://github.com/gaearon/react-hot-boilerplate) [comuns](https://github.com/erikras/react-redux-universal-hot-example). No entanto, os boilerplates eram difíceis de atualizar e a fragmentação dificultava o lançamento de novos recursos pelo React.

O Create React App resolveu esses problemas combinando várias ferramentas em uma única configuração recomendada. Isso permitiu que os aplicativos tivessem uma maneira simples de atualizar para novos recursos de ferramentas e permitiu que a equipe do React implantasse alterações de ferramentas não triviais (suporte Fast Refresh, regras de lint de React Hooks) para o público mais amplo possível.

Esse modelo se tornou tão popular que existe toda uma categoria de ferramentas que trabalham dessa forma hoje.

## **Descontinuando** Create React App {/*deprecating-create-react-app*/}

Embora o Create React App facilite o início, [existem várias limitações](#limitations-of-build-tools) que dificultam a criação de aplicativos de produção de alto desempenho. Em princípio, poderíamos resolver esses problemas essencialmente evoluindo-o para um [framework](#why-we-recommend-frameworks).

No entanto, como o Create React App atualmente não possui mantenedores ativos e existem muitos frameworks existentes que já resolvem esses problemas, decidimos **descontinuar** o Create React App.

A partir de hoje, se você instalar um novo aplicativo, verá um aviso de **descontinuação**:

<ConsoleBlockMulti>
<ConsoleLogLine level="error">

create-react-app is deprecated.
{'\n\n'}
You can find a list of up-to-date React frameworks on react.dev
For more info see: react.dev/link/cra
{'\n\n'}
This error message will only be shown once per install.

</ConsoleLogLine>
</ConsoleBlockMulti>

Também adicionamos um aviso de **descontinuação** ao [site](https://create-react-app.dev/) e ao [repositório](https://github.com/facebook/create-react-app) do GitHub do Create React App. O Create React App continuará funcionando no modo de manutenção e publicamos uma nova versão do Create React App para funcionar com o React 19.

## Como Migrar para um Framework {/*how-to-migrate-to-a-framework*/}

Recomendamos [criar novos aplicativos React](/learn/creating-a-react-app) com um framework. Todos os frameworks que recomendamos suportam a renderização do lado do cliente ([CSR](https://developer.mozilla.org/en-US/docs/Glossary/CSR)) e aplicativos de página única ([SPA](https://developer.mozilla.org/en-US/docs/Glossary/SPA)) e podem ser implantados em uma CDN ou serviço de hospedagem estática sem um servidor.

Para aplicativos existentes, estes guias ajudarão você a migrar para um SPA somente cliente:

*   [Guia de migração do Create React App do Next.js](https://nextjs.org/docs/app/building-your-application/upgrading/from-create-react-app)
*   [Guia de adoção de framework do React Router](https://reactrouter.com/upgrading/component-routes).
*   [Guia de migração do Expo webpack para Expo Router](https://docs.expo.dev/router/migrate/from-expo-webpack/)

## Como Migrar para uma Ferramenta de Build {/*how-to-migrate-to-a-build-tool*/}

Se seu aplicativo tiver restrições incomuns ou você preferir resolver esses problemas construindo seu próprio framework, ou apenas quiser aprender como o react funciona do zero, você pode criar sua própria configuração personalizada com React usando Vite, Parcel ou Rsbuild.

Para aplicativos existentes, estes guias ajudarão você a migrar para uma ferramenta de build:

*   [Guia de migração do Vite Create React App](https://www.robinwieruch.de/vite-create-react-app/)
*   [Guia de migração do Parcel Create React App](https://parceljs.org/migration/cra/)
*   [Guia de migração do Rsbuild Create React App](https://rsbuild.dev/guide/migration/cra)

Para ajudar a começar com Vite, Parcel ou Rsbuild, adicionamos novos documentos para [Construindo um Aplicativo React do Zero](/learn/build-a-react-app-from-scratch).

<DeepDive>

#### Preciso de um framework? {/*do-i-need-a-framework*/}

A maioria dos aplicativos se beneficiaria de um framework, mas existem casos válidos para construir um aplicativo React do zero. Uma boa regra geral é que, se seu aplicativo precisar de roteamento, você provavelmente se beneficiará de um framework.

Assim como Svelte tem Sveltekit, Vue tem Nuxt e Solid tem SolidStart, [o React recomenda o uso de um framework](#why-we-recommend-frameworks) que integra totalmente o roteamento em recursos como busca de dados e divisão de código pronta para uso. Isso evita a dor de precisar escrever suas próprias configurações complexas e essencialmente construir um framework você mesmo.

No entanto, você sempre pode [construir um aplicativo React do zero](/learn/build-a-react-app-from-scratch) usando uma ferramenta de build como Vite, Parcel ou Rsbuild.

</DeepDive>

Continue lendo para saber mais sobre as [limitações das ferramentas de build](#limitations-of-build-tools) e [por que recomendamos frameworks](#why-we-recommend-frameworks).

## Limitações das Ferramentas de Build {/*limitations-of-build-tools*/}

O Create React App e as ferramentas de build como ele facilitam o início da construção de um aplicativo React. Depois de executar `npx create-react-app my-app`, você obtém um aplicativo React totalmente configurado com um servidor de desenvolvimento, linting e uma build de produção.

Por exemplo, se você estiver construindo uma ferramenta de administração interna, pode começar com uma página de destino:

```js
export default function App() {
  return (
    <div>
      <h1>Bem-vindo à Ferramenta de Administração!</h1>
    </div>
  )
}
```

Isso permite que você comece a codificar imediatamente em React com recursos como JSX, regras de linting padrão e um bundler para executar tanto no desenvolvimento quanto na produção. No entanto, essa configuração está perdendo as ferramentas necessárias para construir um aplicativo de produção real.

A maioria dos aplicativos de produção precisa de soluções para problemas como roteamento, busca de dados e divisão de código.

### Roteamento {/*routing*/}

O Create React App não inclui uma solução de roteamento específica. Se você está apenas começando, uma opção é usar `useState` para alternar entre as rotas. Mas fazer isso significa que você não pode compartilhar links para seu aplicativo - cada link iria para a mesma página - e estruturar seu aplicativo se torna difícil com o tempo:

```js
import {useState} from 'react';

import Home from './Home';
import Dashboard from './Dashboard';

export default function App() {
  // ❌ O roteamento no estado não cria URLs
  const [route, setRoute] = useState('home');
  return (
    <div>
      {route === 'home' && <Home />}
      {route === 'dashboard' && <Dashboard />}
    </div>
  )
}
```

É por isso que a maioria dos aplicativos que usam o Create React App resolvem adicionar roteamento com uma biblioteca de roteamento como [React Router](https://reactrouter.com/) ou [Tanstack Router](https://tanstack.com/router/latest). Com uma biblioteca de roteamento, você pode adicionar rotas adicionais ao aplicativo, o que fornece opiniões sobre a estrutura do seu aplicativo e permite que você comece a compartilhar links para rotas. Por exemplo, com o React Router, você pode definir rotas:

```js
import {RouterProvider, createBrowserRouter} from 'react-router';

import Home from './Home';
import Dashboard from './Dashboard';

// ✅ Cada rota tem sua própria URL
const router = createBrowserRouter([
  {path: '/', element: <Home />},
  {path: '/dashboard', element: <Dashboard />}
]);

export default function App() {
  return (
    <RouterProvider value={router} />
  )
}
```

Com essa alteração, você pode compartilhar um link para `/dashboard` e o aplicativo navegará para a página do painel. Depois de ter uma biblioteca de roteamento, você pode adicionar recursos adicionais como rotas aninhadas, guardas de rota e transições de rota, que são difíceis de implementar sem uma biblioteca de roteamento.

Há uma troca sendo feita aqui: a biblioteca de roteamento adiciona complexidade ao aplicativo, mas também adiciona recursos que são difíceis de implementar sem ela.

### Busca de Dados {/*data-fetching*/}

Outro problema comum no Create React App é a busca de dados. O Create React App não inclui uma solução específica de busca de dados. Se você está apenas começando, uma opção comum é usar `fetch` em um efeito para carregar dados.

Mas fazer isso significa que os dados são buscados após a renderização do componente, o que pode causar cascatas de rede. As cascatas de rede são causadas pela busca de dados quando seu aplicativo renderiza em vez de em paralelo enquanto o código está sendo baixado:

```js
export default function Dashboard() {
  const [data, setData] = useState(null);

  // ❌ Buscar dados em um componente causa cascatas de rede
  useEffect(() => {
    fetch('/api/data')
      .then(response => response.json())
      .then(data => setData(data));
  }, []);

  return (
    <div>
      {data.map(item => <div key={item.id}>{item.name}</div>)}
    </div>
  )
}
```

Buscar em um efeito significa que o usuário tem que esperar mais tempo para ver o conteúdo, mesmo que os dados pudessem ter sido buscados antes. Para resolver isso, você pode usar uma biblioteca de busca de dados como [React Query](https://react-query.tanstack.com/), [SWR](https://swr.vercel.app/), [Apollo](https://www.apollographql.com/docs/react) ou [Relay](https://relay.dev/) que fornecem opções para pré-buscar dados para que a solicitação seja iniciada antes da renderização do componente.

Essas bibliotecas funcionam melhor quando integradas ao seu padrão de "carregador" de roteamento para especificar dependências de dados no nível da rota, o que permite que o roteador otimize suas buscas de dados:

```js
export async function loader() {
  const response = await fetch(`/api/data`);
  const data = await response.json();
  return data;
}

// ✅ Buscar dados em paralelo enquanto o código está sendo baixado
export default function Dashboard({loaderData}) {
  return (
    <div>
      {loaderData.map(item => <div key={item.id}>{item.name}</div>)}
    </div>
  )
}
```

No carregamento inicial, o roteador pode buscar os dados imediatamente antes da renderização da rota. À medida que o usuário navega pelo aplicativo, o roteador é capaz de buscar os dados e a rota ao mesmo tempo, paralelizando as buscas. Isso reduz o tempo necessário para ver o conteúdo na tela e pode melhorar a experiência do usuário.

No entanto, isso requer a configuração correta dos carregadores em seu aplicativo e troca a complexidade pelo desempenho.

### Divisão de Código {/*code-splitting*/}

Outro problema comum no Create React App é a [divisão de código](https://www.patterns.dev/vanilla/bundle-splitting). O Create React App não inclui uma solução específica de divisão de código. Se você está apenas começando, pode não considerar a divisão de código.

Isso significa que seu aplicativo é enviado como um único bundle:

```txt
- bundle.js    75kb
```

Mas, para um desempenho ideal, você deve "dividir" seu código em bundles separados para que o usuário só precise baixar o que precisa. Isso diminui o tempo que o usuário precisa esperar para carregar seu aplicativo, baixando apenas o código necessário para ver a página em que está.

```txt
- core.js      25kb
- home.js      25kb
- dashboard.js 25kb
```

Uma maneira de fazer a divisão de código é com `React.lazy`. No entanto, isso significa que o código não é buscado até que o componente renderize, o que pode causar cascatas de rede. Uma solução mais otimizada é usar um recurso de roteador que busca o código em paralelo enquanto o código está sendo baixado. Por exemplo, o React Router fornece uma opção `lazy` para especificar que uma rota deve ser dividida em código e otimizar quando ela é carregada:

```js
import Home from './Home';
import Dashboard from './Dashboard';

// ✅ As rotas são baixadas antes da renderização
const router = createBrowserRouter([
  {path: '/', lazy: () => import('./Home')},
  {path: '/dashboard', lazy: () => import('Dashboard')}
]);
```

A divisão de código otimizada é complicada de acertar e é fácil cometer erros que podem fazer com que o usuário baixe mais código do que precisa. Funciona melhor quando integrado ao seu roteador e soluções de carregamento de dados para maximizar o cache, paralelizar as buscas e suportar padrões de ["importação por interação"](https://www.patterns.dev/vanilla/import-on-interaction).

### E mais... {/*and-more*/}

Estes são apenas alguns exemplos das limitações do Create React App.

Depois de integrar roteamento, busca de dados e divisão de código, você também precisa considerar estados pendentes, interrupções de navegação, mensagens de erro para o usuário e revalidação dos dados. Existem categorias inteiras de problemas que os usuários precisam resolver, como:

<div style={{display: 'flex', width: '100%', justifyContent: 'space-around'}}>
  <ul>
    <li>Acessibilidade</li>
    <li>Carregamento de ativos</li>
    <li>Autenticação</li>
    <li>Cache</li>
  </ul>
  <ul>
    <li>Tratamento de erros</li>
    <li>Mutação de dados</li>
    <li>Navegações</li>
    <li>Atualizações otimistas</li>
  </ul>
  <ul>
    <li>Aprimoramento progressivo</li>
    <li>Renderização do lado do servidor</li>
    <li>Geração de site estático</li>
    <li>Streaming</li>
  </ul>
</div>

Todos esses trabalham juntos para criar a [sequência de carregamento](https://www.patterns.dev/vanilla/loading-sequence) mais otimizada.

Resolver cada um desses problemas individualmente no Create React App pode ser difícil, pois cada problema está interconectado com os outros e pode exigir profundo conhecimento em áreas problemáticas com as quais os usuários podem não estar familiarizados. Para resolver esses problemas, os usuários acabam construindo suas próprias soluções sob medida em cima do Create React App, que era o problema que o Create React App originalmente tentou resolver.

## Por que Recomendamos Frameworks {/*why-we-recommend-frameworks*/}

Embora você pudesse resolver todas essas peças sozinho em uma ferramenta de build como Create React App, Vite ou Parcel, é difícil fazer bem. Assim como quando o próprio Create React App integrou várias ferramentas de build, você precisa de uma ferramenta para integrar todos esses recursos para fornecer a melhor experiência aos usuários.

Essa categoria de ferramentas que integra ferramentas de build, renderização, roteamento, busca de dados e divisão de código é conhecida como "frameworks" - ou, se você preferir chamar o próprio React de framework, você pode chamá-los de "metaframeworks".

Os frameworks impõem algumas opiniões sobre como estruturar seu aplicativo para fornecer uma experiência de usuário muito melhor, da mesma forma que as ferramentas de build impõem algumas opiniões para facilitar as ferramentas. É por isso que começamos a recomendar frameworks como [Next.js](https://nextjs.org/), [React Router](https://reactrouter.com/) e [Expo](https://expo.dev/) para novos projetos.

Os frameworks fornecem a mesma experiência de introdução que o Create React App, mas também fornecem soluções para problemas que os usuários precisam resolver de qualquer maneira em aplicativos de produção reais.

<DeepDive>

#### Renderização do servidor é opcional {/*server-rendering-is-optional*/}

Os frameworks que recomendamos todos fornecem a opção de criar um aplicativo [renderizado no lado do cliente (CSR)](https://developer.mozilla.org/en-US/docs/Glossary/CSR).

Em alguns casos, o CSR é a escolha certa para uma página, mas muitas vezes não é. Mesmo que a maior parte do seu aplicativo seja do lado do cliente, muitas vezes existem páginas individuais que podem se beneficiar de recursos de renderização do servidor, como [geração de site estático (SSG)](https://developer.mozilla.org/en-US/docs/Glossary/SSG) ou [renderização do lado do servidor (SSR)](https://developer.mozilla.org/en-US/docs/Glossary/SSR), por exemplo, uma página de Termos de Serviço ou documentação.

A renderização do servidor geralmente envia menos JavaScript para o cliente e um documento HTML completo, o que produz um [First Contentful Paint (FCP)](https://web.dev/articles/fcp) mais rápido, reduzindo o [Total Blocking Time (TBD)](https://web.dev/articles/tbt), o que também pode diminuir o [Interaction to Next Paint (INP)](https://web.dev/articles/inp). É por isso que a [equipe do Chrome incentivou](https://web.dev/articles/rendering-on-the-web) os desenvolvedores a considerar a renderização estática ou do lado do servidor em vez de uma abordagem totalmente do lado do cliente para obter o melhor desempenho possível.

Existem compensações ao usar um servidor, e nem sempre é a melhor opção para todas as páginas. A geração de páginas no servidor incorre em custos adicionais e leva tempo para gerar, o que pode aumentar o [Time to First Byte (TTFB)](https://web.dev/articles/ttfb). Os aplicativos com melhor desempenho são capazes de escolher a estratégia de renderização certa por página, com base nas compensações de cada estratégia.

Os frameworks fornecem a opção de usar um servidor em qualquer página, se você quiser, mas não o forçam a usar um servidor. Isso permite que você escolha a estratégia de renderização certa para cada página em seu aplicativo.

#### E quanto aos Server Components {/*server-components*/}

Os frameworks que recomendamos também incluem suporte para React Server Components.

Os Server Components ajudam a resolver esses problemas movendo o roteamento e a busca de dados para o servidor e permitindo que a divisão de código seja feita para componentes do cliente com base nos dados que você renderiza, em vez apenas da rota renderizada, e reduzindo a quantidade de JavaScript enviado para obter a melhor [sequência de carregamento](https://www.patterns.dev/vanilla/loading-sequence) possível.

Os Server Components não exigem um servidor. Eles podem ser executados no momento da build no seu servidor CI para criar um aplicativo gerado por site estático (SSG), no tempo de execução em um servidor web para um aplicativo renderizado no lado do servidor (SSR).

Consulte [Apresentando React Server Components de tamanho zero do bundle](/blog/2020/12/21/data-fetching-with-react-server-components) e [os documentos](/reference/rsc/server-components) para obter mais informações.

</DeepDive>

<Note>

#### Renderização do servidor não é apenas para SEO {/*server-rendering-is-not-just-for-seo*/}

Um mal-entendido comum é que a renderização do servidor é apenas para [SEO](https://developer.mozilla.org/en-US/docs/Glossary/SEO).

Embora a renderização do servidor possa melhorar o SEO, ela também melhora o desempenho, reduzindo a quantidade de JavaScript que o usuário precisa baixar e analisar antes de poder ver o conteúdo na tela.

É por isso que a equipe do Chrome [incentivou](https://web.dev/articles/rendering-on-the-web) os desenvolvedores a considerar a renderização estática ou do lado do servidor em vez de uma abordagem totalmente do lado do cliente para obter o melhor desempenho possível.

</Note>

---

_Obrigado a [Dan Abramov](https://bsky.app/profile/danabra.mov) por criar o Create React App e [Joe Haddad](https://github.com/Timer), [Ian Schmitz](https://github.com/ianschmitz), [Brody McKee](https://github.com/mrmckeb) e [muitos outros](https://github.com/facebook/create-react-app/graphs/contributors) por manter o Create React App ao longo dos anos. Obrigado a [Brooks Lybrand](https://bsky.app/profile/brookslybrand.bsky.social), [Dan Abramov](https://bsky.app/profile/danabra.mov), [Devon Govett](https://bsky.app/profile/devongovett.bsky.social), [Eli White](https://x.com/Eli_White), [Jack Herrington](https://bsky.app/profile/jherr.dev), [Joe Savona](https://x.com/en_JS), [Lauren Tan](https://bsky.app/profile/no.lol), [Lee Robinson](https://x.com/leeerob), [Mark Erikson](https://bsky.app/profile/acemarke.dev), [Ryan Florence](https://x.com/ryanflorence), [Sophie Alpert](https://bsky.app/profile/sophiebits.com), [Tanner Linsley](https://bsky.app/profile/tannerlinsley.com) e [Theo Browne](https://x.com/theo) por revisar e fornecer feedback sobre esta postagem._