---
title: "Descontinuação do Create React App"
author: Matt Carroll and Ricky Hanlon
date: 2025/02/14
description: Hoje, estamos descontinuando o Create React App para novos aplicativos e incentivando os aplicativos existentes a migrarem para um framework ou para uma ferramenta de build como Vite, Parcel ou RSBuild. Também estamos fornecendo documentação para quando um framework não for adequado para o seu projeto, você quiser criar seu próprio framework ou apenas quiser aprender como o React funciona construindo um aplicativo React do zero.
---

14 de fevereiro de 2025 por [Matt Carroll](https://twitter.com/mattcarrollcode) e [Ricky Hanlon](https://bsky.app/profile/ricky.fm)

---

<Intro>

Hoje, estamos descontinuando o [Create React App](https://create-react-app.dev/) para novos aplicativos e incentivando os aplicativos existentes a migrarem para um [framework](#how-to-migrate-to-a-framework), ou para [migrarem para uma ferramenta de build](#how-to-migrate-to-a-build-tool) como Vite, Parcel ou RSBuild.

Também estamos fornecendo documentação para quando um framework não for adequado para o seu projeto, você quiser criar seu próprio framework ou apenas quiser aprender como o React funciona [construindo um aplicativo React do zero](/learn/build-a-react-app-from-scratch).

</Intro>

-----

Quando lançamos o Create React App em 2016, não havia uma maneira clara de criar um novo aplicativo React.

Para criar um aplicativo React, você precisava instalar várias ferramentas e configurá-las manualmente para suportar recursos básicos como JSX, linting e hot reloading. Isso era muito complicado de fazer corretamente, então a [comunidade](https://github.com/react-boilerplate/react-boilerplate) [criou](https://github.com/kriasoft/react-starter-kit) [templates (boilerplates)](https://github.com/petehunt/react-boilerplate) para [configurações comuns](https://github.com/gaearon/react-hot-boilerplate) [e](https://github.com/erikras/react-redux-universal-hot-example) [outras](https://github.com/gaearon/react-hot-boilerplate). No entanto, os templates eram difíceis de atualizar e a fragmentação dificultava o lançamento de novos recursos pelo React.

O Create React App resolveu esses problemas combinando várias ferramentas em uma única configuração recomendada. Isso permitiu que os aplicativos tivessem uma maneira simples de atualizar para novos recursos de ferramentas e permitiu que a equipe do React implantasse alterações de ferramentas não triviais (suporte ao Fast Refresh, regras de linting para Hooks do React) para o público mais amplo possível.

Este modelo se tornou tão popular que existe uma categoria inteira de ferramentas funcionando dessa maneira hoje.

## Descontinuação do Create React App {/*deprecating-create-react-app*/}

Embora o Create React App facilite o início, [existem várias limitações](#limitations-of-build-tools) que dificultam a criação de aplicativos de produção de alto desempenho. Em princípio, poderíamos resolver esses problemas essencialmente evoluindo-o para um [framework](#why-we-recommend-frameworks).

No entanto, como o Create React App atualmente não tem mantenedores ativos e já existem muitos frameworks que resolvem esses problemas, decidimos descontinuar o Create React App.

A partir de hoje, se você instalar um novo aplicativo, verá um aviso de descontinuação:

<ConsoleBlockMulti>
<ConsoleLogLine level="error">

create-react-app está descontinuado.
{'\n\n'}
Você pode encontrar uma lista de frameworks React atualizados em react.dev
Para mais informações, veja: react.dev/link/cra
{'\n\n'}
Esta mensagem de erro só será exibida uma vez por instalação.

</ConsoleLogLine>
</ConsoleBlockMulti>

Também adicionamos um aviso de descontinuação ao [site](https://create-react-app.dev/) e ao [repositório](https://github.com/facebook/create-react-app) do Create React App no GitHub. O Create React App continuará funcionando em modo de manutenção e publicamos uma nova versão do Create React App para funcionar com o React 19.

## Como Migrar para um Framework {/*how-to-migrate-to-a-framework*/}
Recomendamos [criar novos aplicativos React](/learn/creating-a-react-app) com um framework. Todos os frameworks que recomendamos suportam renderização do lado do cliente ([CSR](https://developer.mozilla.org/en-US/docs/Glossary/CSR)) e aplicativos de página única ([SPA](https://developer.mozilla.org/en-US/docs/Glossary/SPA)), e podem ser implantados em um CDN ou serviço de hospedagem estática sem um servidor.

Para aplicativos existentes, estes guias ajudarão você a migrar para uma SPA apenas do lado do cliente:

* [Guia de migração do Next.js do Create React App](https://nextjs.org/docs/app/building-your-application/upgrading/from-create-react-app)
* [Guia de adoção de framework do React Router](https://reactrouter.com/upgrading/component-routes).
* [Guia de migração do Expo webpack para Expo Router](https://docs.expo.dev/router/migrate/from-expo-webpack/)

## Como Migrar para uma Ferramenta de Build {/*how-to-migrate-to-a-build-tool*/}

Se seu aplicativo tiver restrições incomuns, ou se você preferir resolver esses problemas criando seu próprio framework, ou se você apenas quiser aprender como o React funciona do zero, você pode criar sua própria configuração personalizada com React usando Vite, Parcel ou Rsbuild.

Para aplicativos existentes, estes guias ajudarão você a migrar para uma ferramenta de build:

* [Guia de migração do Vite para Create React App](https://www.robinwieruch.de/vite-create-react-app/)
* [Guia de migração do Parcel para Create React App](https://parceljs.org/migration/cra/)
* [Guia de migração do Rsbuild para Create React App](https://rsbuild.dev/guide/migration/cra)

Para ajudar a começar com Vite, Parcel ou Rsbuild, adicionamos nova documentação para [Construindo um Aplicativo React do Zero](/learn/build-a-react-app-from-scratch).

<DeepDive>

#### Preciso de um framework? {/*do-i-need-a-framework*/}

A maioria dos aplicativos se beneficiaria de um framework, mas existem casos válidos para construir um aplicativo React do zero. Uma boa regra geral é que, se seu aplicativo precisar de roteamento, você provavelmente se beneficiará de um framework.

Assim como o Svelte tem o Sveltekit, o Vue tem o Nuxt e o Solid tem o SolidStart, [o React recomenda o uso de um framework](#why-we-recommend-frameworks) que integre totalmente o roteamento em recursos como data fetching e code-splitting de fábrica. Isso evita o incômodo de ter que escrever suas próprias configurações complexas e, essencialmente, construir um framework você mesmo.

No entanto, você sempre pode [construir um aplicativo React do zero](/learn/build-a-react-app-from-scratch) usando uma ferramenta de build como Vite, Parcel ou Rsbuild.

</DeepDive>

Continue lendo para saber mais sobre as [limitações das ferramentas de build](#limitations-of-build-tools) e [por que recomendamos frameworks](#why-we-recommend-frameworks).

## Limitações das Ferramentas de Build {/*limitations-of-build-tools*/}

O Create React App e ferramentas de build semelhantes facilitam o início da construção de um aplicativo React. Após executar `npx create-react-app my-app`, você obtém um aplicativo React totalmente configurado com um servidor de desenvolvimento, linting e um build de produção.

Por exemplo, se você estiver criando uma ferramenta de administração interna, pode começar com uma página de destino:

```js
export default function App() {
  return (
    <div>
      <h1>Bem-vindo à Ferramenta de Administração!</h1>
    </div>
  )
}
```

Isso permite que você comece imediatamente a codificar em React com recursos como JSX, regras de linting padrão e um bundler para executar tanto em desenvolvimento quanto em produção. No entanto, essa configuração não possui as ferramentas necessárias para criar um aplicativo de produção real.

A maioria dos aplicativos de produção precisa de soluções para problemas como roteamento, data fetching e code splitting.

### Roteamento {/*routing*/}

O Create React App não inclui uma solução de roteamento específica. Se você está apenas começando, uma opção é usar `useState` para alternar entre rotas. Mas fazer isso significa que você não pode compartilhar links para seu aplicativo - cada link iria para a mesma página - e a estruturação do seu aplicativo se torna difícil ao longo do tempo:

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

É por isso que a maioria dos aplicativos que usam o Create React App resolvem adicionando roteamento com uma biblioteca de roteamento como [React Router](https://reactrouter.com/) ou [Tanstack Router](https://tanstack.com/router/latest). Com uma biblioteca de roteamento, você pode adicionar rotas adicionais ao aplicativo, o que fornece opiniões sobre a estrutura do seu aplicativo e permite que você comece a compartilhar links para rotas. Por exemplo, com o React Router você pode definir rotas:

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

Com essa alteração, você pode compartilhar um link para `/dashboard` e o aplicativo navegará para a página do painel. Depois de ter uma biblioteca de roteamento, você pode adicionar recursos adicionais como rotas aninhadas, guards de rota e transições de rota, que são difíceis de implementar sem uma biblioteca de roteamento.

Há um trade-off sendo feito aqui: a biblioteca de roteamento adiciona complexidade ao aplicativo, mas também adiciona recursos que são difíceis de implementar sem ela.

### Data Fetching {/*data-fetching*/}

Outro problema comum no Create React App é o data fetching. O Create React App não inclui uma solução específica de data fetching. Se você está apenas começando, uma opção comum é usar `fetch` em um efeito para carregar dados.

Mas fazer isso significa que os dados são buscados após a renderização do componente, o que pode causar "network waterfalls" (cascatas de rede). As cascatas de rede são causadas pela busca de dados quando seu aplicativo renderiza em vez de em paralelo enquanto o código está sendo baixado:

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

Buscar em um efeito significa que o usuário tem que esperar mais para ver o conteúdo, mesmo que os dados pudessem ter sido buscados mais cedo. Para resolver isso, você pode usar uma biblioteca de data fetching como [React Query](https://react-query.tanstack.com/), [SWR](https://swr.vercel.app/), [Apollo](https://www.apollographql.com/docs/react) ou [Relay](https://relay.dev/) que fornecem opções para pré-buscar dados para que a solicitação seja iniciada antes que o componente seja renderizado.

Essas bibliotecas funcionam melhor quando integradas ao seu padrão de "loader" de roteamento para especificar dependências de dados no nível da rota, o que permite que o roteador otimize suas buscas de dados:

```js
export async function loader() {
  const response = await fetch(`/api/data`);
  const data = await response.json();
  return data;
}

// ✅ Buscando dados em paralelo enquanto o código está sendo baixado
export default function Dashboard({loaderData}) {
  return (
    <div>
      {loaderData.map(item => <div key={item.id}>{item.name}</div>)}
    </div>
  )
}
```

Na carga inicial, o roteador pode buscar os dados imediatamente antes que a rota seja renderizada. À medida que o usuário navega pelo aplicativo, o roteador é capaz de buscar tanto os dados quanto a rota ao mesmo tempo, paralelizando as buscas. Isso reduz o tempo necessário para ver o conteúdo na tela e pode melhorar a experiência do usuário.

No entanto, isso requer a configuração correta dos loaders em seu aplicativo e troca complexidade por desempenho.

### Code Splitting {/*code-splitting*/}

Outro problema comum no Create React App é o [code splitting](https://www.patterns.dev/vanilla/bundle-splitting). O Create React App não inclui uma solução específica de code splitting. Se você está apenas começando, pode nem considerar o code splitting.

Isso significa que seu aplicativo é enviado como um único bundle:

```txt
- bundle.js    75kb
```

Mas para um desempenho ideal, você deve "dividir" seu código em bundles separados para que o usuário precise baixar apenas o que ele precisa. Isso diminui o tempo que o usuário precisa esperar para carregar seu aplicativo, baixando apenas o código que ele precisa para ver a página em que está.

```txt
- core.js      25kb
- home.js      25kb
- dashboard.js 25kb
```

Uma maneira de fazer code-splitting é com `React.lazy`. No entanto, isso significa que o código não é buscado até que o componente seja renderizado, o que pode causar cascatas de rede. Uma solução mais otimizada é usar um recurso do roteador que busca o código em paralelo enquanto o código está sendo baixado. Por exemplo, o React Router fornece uma opção `lazy` para especificar que uma rota deve ser dividida em código e otimizar quando ela é carregada:

```js
import Home from './Home';
import Dashboard from './Dashboard';

// ✅ As rotas são baixadas antes da renderização
const router = createBrowserRouter([
  {path: '/', lazy: () => import('./Home')},
  {path: '/dashboard', lazy: () => import('Dashboard')}
]);
```

O code-splitting otimizado é complicado de acertar, e é fácil cometer erros que podem fazer com que o usuário baixe mais código do que precisa. Funciona melhor quando integrado com suas soluções de roteador e carregamento de dados para maximizar o cache, paralelizar buscas e suportar padrões de ["import on interaction"](https://www.patterns.dev/vanilla/import-on-interaction).

### E Mais... {/*and-more*/}

Estes são apenas alguns exemplos das limitações do Create React App.

Depois de integrar roteamento, data fetching e code splitting, você agora também precisa considerar estados pendentes, interrupções de navegação, mensagens de erro para o usuário e revalidação dos dados. Existem categorias inteiras de problemas que os usuários precisam resolver, como:

<div style={{display: 'flex', width: '100%', justifyContent: 'space-around'}}>
  <ul>
    <li>Acessibilidade</li>
    <li>Carregamento de assets</li>
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
    <li>Melhoria progressiva</li>
    <li>Renderização do lado do servidor</li>
    <li>Geração de site estático</li>
    <li>Streaming</li>
  </ul>
</div>

Todos esses trabalham juntos para criar a [sequência de carregamento](https://www.patterns.dev/vanilla/loading-sequence) mais otimizada.

Resolver cada um desses problemas individualmente no Create React App pode ser difícil, pois cada problema está interconectado com os outros e pode exigir profundo conhecimento em áreas problemáticas com as quais os usuários podem não estar familiarizados. Para resolver esses problemas, os usuários acabam construindo suas próprias soluções personalizadas sobre o Create React App, que era o problema que o Create React App originalmente tentava resolver.

## Por que Recomendamos Frameworks {/*why-we-recommend-frameworks*/}

Embora você possa resolver todas essas peças sozinho em uma ferramenta de build como Create React App, Vite ou Parcel, é difícil fazer isso bem. Assim como o Create React App integrou várias ferramentas de build, você precisa de uma ferramenta para integrar todos esses recursos para fornecer a melhor experiência aos usuários.

Essa categoria de ferramentas que integra ferramentas de build, renderização, roteamento, data fetching e code splitting é conhecida como "frameworks" - ou se você preferir chamar o próprio React de framework, pode chamá-los de "metaframeworks".

Frameworks impõem algumas opiniões sobre a estruturação do seu aplicativo para fornecer uma experiência de usuário muito melhor, da mesma forma que as ferramentas de build impõem algumas opiniões para facilitar o uso das ferramentas. É por isso que começamos a recomendar frameworks como [Next.js](https://nextjs.org/), [React Router](https://reactrouter.com/) e [Expo](https://expo.dev/) para novos projetos.

Frameworks fornecem a mesma experiência de início que o Create React App, mas também fornecem soluções para problemas que os usuários precisam resolver de qualquer maneira em aplicativos de produção