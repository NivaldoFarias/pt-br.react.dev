---
title: "React Labs: View Transitions, Activity, e mais"
author: Ricky Hanlon
date: 2025/04/23
description: Em posts do React Labs, escrevemos sobre projetos em pesquisa e desenvolvimento ativo. Neste post, compartilhamos dois novos recursos experimentais que estão prontos para serem testados hoje, e atualizações sobre outras áreas em que estamos trabalhando agora.
---

23 de abril de 2025 por [Ricky Hanlon](https://twitter.com/rickhanlonii)

---

<Intro>

Em posts do React Labs, escrevemos sobre projetos em pesquisa e desenvolvimento ativo. Neste post, compartilhamos dois novos recursos experimentais que estão prontos para serem testados hoje, e atualizações sobre outras áreas em que estamos trabalhando agora.

</Intro>


<Note>

O React Conf 2025 está agendado para 7 e 8 de outubro em Henderson, Nevada!

Estamos procurando palestrantes para nos ajudar a criar talks sobre os recursos abordados neste post. Se você tem interesse em palestrar no ReactConf, [por favor, aplique aqui](https://forms.reform.app/react-conf/call-for-speakers/) (não é necessária proposta de talk).

Para mais informações sobre ingressos, streaming gratuito, patrocínio e mais, veja [o site do React Conf](https://conf.react.dev).

</Note>

Hoje, estamos animados em lançar a documentação para dois novos recursos experimentais que estão prontos para teste:

- [View Transitions](#view-transitions)
- [Activity](#activity)

Também estamos compartilhando atualizações sobre novos recursos em desenvolvimento:
- [React Performance Tracks](#react-performance-tracks)
- [Compiler IDE Extension](#compiler-ide-extension)
- [Automatic Effect Dependencies](#automatic-effect-dependencies)
- [Fragment Refs](#fragment-refs)
- [Concurrent Stores](#concurrent-stores)

---

# Novos Recursos Experimentais {/*new-experimental-features*/}

View Transitions e Activity estão agora prontos para teste em `react@experimental`. Esses recursos foram testados em produção e são estáveis, mas a API final ainda pode mudar à medida que incorporamos feedback.

Você pode testá-los atualizando os pacotes do React para a versão experimental mais recente:

- `react@experimental`
- `react-dom@experimental`

Leia mais para aprender como usar esses recursos em seu aplicativo, ou confira os docs recém-publicados:

- [`<ViewTransition>`](/reference/react/ViewTransition): Um componente que permite ativar uma animação para uma Transição.
- [`addTransitionType`](/reference/react/addTransitionType): Uma função que permite especificar a causa de uma Transição.
- [`<Activity>`](/reference/react/Activity): Um componente que permite ocultar e mostrar partes da UI.

## View Transitions {/*view-transitions*/}

React View Transitions é um novo recurso experimental que facilita a adição de animações a transições de UI em seu aplicativo. Por baixo dos panos, essas animações usam a nova API [`startViewTransition`](https://developer.mozilla.org/en-US/docs/Web/API/Document/startViewTransition) disponível na maioria dos navegadores modernos.

Para ativar a animação de um elemento, envolva-o no novo componente `<ViewTransition>`:

```js
// "o quê" animar.
<ViewTransition>
  <div>anime-me</div>
</ViewTransition>
```

Este novo componente permite definir declarativamente "o quê" animar quando uma animação é ativada.

Você pode definir "quando" animar usando um destes três gatilhos para uma View Transition:

```js
// "quando" animar.

// Transitions
startTransition(() => setState(...));

// Deferred Values
const deferred = useDeferredValue(value);

// Suspense
<Suspense fallback={<Fallback />}>
  <div>Loading...</div>
</Suspense>
```

Por padrão, essas animações usam as [animações CSS padrão para View Transitions](https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API/Using#customizing_your_animations) aplicadas (tipicamente um cross-fade suave). Você pode usar [pseudo-seletores de view transition](https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API/Using#the_view_transition_pseudo-element_tree) para definir "como" a animação é executada. Por exemplo, você pode usar `*` para alterar a animação padrão para todas as transições:

```
// "como" animar.
::view-transition-old(*) {
  animation: 300ms ease-out fade-out;
}
::view-transition-new(*) {
  animation: 300ms ease-in fade-in;
}
```

Quando o DOM é atualizado devido a um gatilho de animação — como `startTransition`, `useDeferredValue`, ou um fallback de `Suspense` mudando para conteúdo — o React usará [heurísticas declarativas](/reference/react/ViewTransition#viewtransition) para determinar automaticamente quais componentes `<ViewTransition>` ativar para a animação. O navegador então executará a animação definida em CSS.

Se você está familiarizado com a View Transition API do navegador e quer saber como o React a suporta, confira [Como funciona `<ViewTransition>`](/reference/react/ViewTransition#how-does-viewtransition-work) na documentação.

Neste post, vamos dar uma olhada em alguns exemplos de como usar View Transitions.

Começaremos com este app, que não anima nenhuma das seguintes interações:
- Clicar em um vídeo para ver os detalhes.
- Clicar em "back" para voltar ao feed.
- Digitar na lista para filtrar os vídeos.

<Sandpack>

```js src/App.js active
import {unstable_ViewTransition as ViewTransition} from 'react'; import Details from './Details'; import Home from './Home'; import {useRouter} from './router';

export default function App() {
  const {url} = useRouter();
  
  // Use ViewTransition para animar entre páginas.
  // Nenhum CSS adicional necessário por padrão.
  return (
    <ViewTransition>
      {url === '/' ? <Home /> : <Details />}
    </ViewTransition>
  );
}
```

```js src/Details.js
import { fetchVideo, fetchVideoDetails } from "./data";
import { Thumbnail, VideoControls } from "./Videos";
import { useRouter } from "./router";
import Layout from "./Layout";
import { use, Suspense } from "react";
import { ChevronLeft } from "./Icons";

function VideoInfo({ id }) {
  const details = use(fetchVideoDetails(id));
  return (
    <>
      <p className="info-title">{details.title}</p>
      <p className="info-description">{details.description}</p>
    </>
  );
}

function VideoInfoFallback() {
  return (
    <>
      <div className="fallback title"></div>
      <div className="fallback description"></div>
    </>
  );
}

export default function Details() {
  const { url, navigateBack } = useRouter();
  const videoId = url.split("/").pop();
  const video = use(fetchVideo(videoId));

  return (
    <Layout
      heading={
        <div
          className="fit back"
          onClick={() => {
            navigateBack("/");
          }}
        >
          <ChevronLeft /> Back
        </div>
      }
    >
      <div className="details">
        <Thumbnail video={video} large>
          <VideoControls />
        </Thumbnail>
        <Suspense fallback={<VideoInfoFallback />}>
          <VideoInfo id={video.id} />
        </Suspense>
      </div>
    </Layout>
  );
}

```

```js src/Home.js
import { Video } from "./Videos";
import Layout from "./Layout";
import { fetchVideos } from "./data";
import { useId, useState, use } from "react";
import { IconSearch } from "./Icons";

function SearchInput({ value, onChange }) {
  const id = useId();
  return (
    <form className="search" onSubmit={(e) => e.preventDefault()}>
      <label htmlFor={id} className="sr-only">
        Search
      </label>
      <div className="search-input">
        <div className="search-icon">
          <IconSearch />
        </div>
        <input
          type="text"
          id={id}
          placeholder="Search"
          value={value}
          onChange={(e) => onChange(e.target.value)}
        />
      </div>
    </form>
  );
}

function filterVideos(videos, query) {
  const keywords = query
    .toLowerCase()
    .split(" ")
    .filter((s) => s !== "");
  if (keywords.length === 0) {
    return videos;
  }
  return videos.filter((video) => {
    const words = (video.title + " " + video.description)
      .toLowerCase()
      .split(" ");
    return keywords.every((kw) => words.some((w) => w.includes(kw)));
  });
}

export default function Home() {
  const videos = use(fetchVideos());
  const count = videos.length;
  const [searchText, setSearchText] = useState("");
  const foundVideos = filterVideos(videos, searchText);
  return (
    <Layout heading={<div className="fit">{count} Videos</div>}>
      <SearchInput value={searchText} onChange={setSearchText} />
      <div className="video-list">
        {foundVideos.length === 0 && (
          <div className="no-results">No results</div>
        )}
        <div className="videos">
          {foundVideos.map((video) => (
            <Video key={video.id} video={video} />
          ))}
        </div>
      </div>
    </Layout>
  );
}

```

```js src/Icons.js
export function ChevronLeft() {
  return (
    <svg
      className="chevron-left"
      xmlns="http://www.w3.org/2000/svg"
      width="20"
      height="20"
      viewBox="0 0 20 20">
      <g fill="none" fillRule="evenodd" transform="translate(-446 -398)">
        <path
          fill="currentColor"
          fillRule="nonzero"
          d="M95.8838835,240.366117 C95.3957281,239.877961 94.6042719,239.877961 94.1161165,240.366117 C93.6279612,240.854272 93.6279612,241.645728 94.1161165,242.133883 L98.6161165,246.633883 C99.1042719,247.122039 99.8957281,247.122039 100.383883,246.633883 L104.883883,242.133883 C105.372039,241.645728 105.372039,240.854272 104.883883,240.366117 C104.395728,239.877961 103.604272,239.877961 103.116117,240.366117 L99.5,243.982233 L95.8838835,240.366117 Z"
          transform="translate(356.5 164.5)"
        />
        <polygon points="446 418 466 418 466 398 446 398" />
      </g>
    </svg>
  );
}

export function PauseIcon() {
  return (
    <svg
      className="control-icon"
      style={{padding: '4px'}}
      width="100"
      height="100"
      viewBox="0 0 512 512"
      fill="none"
      xmlns="http://www.w3.org/2000/svg">
      <path
        fillRule="evenodd"
        clipRule="evenodd"
        d="M256 0C114.617 0 0 114.615 0 256s114.617 256 256 256 256-114.615 256-256S397.383 0 256 0zm-32 320c0 8.836-7.164 16-16 16h-32c-8.836 0-16-7.164-16-16V192c0-8.836 7.164-16 16-16h32c8.836 0 16 7.164 16 16v128zm128 0c0 8.836-7.164 16-16 16h-32c-8.836 0-16-7.164-16-16V192c0-8.836 7.164-16 16-16h32c8.836 0 16 7.164 16 16v128z"
        fill="currentColor"
      />
    </svg>
  );
}

export function PlayIcon() {
  return (
    <svg
      className="control-icon"
      width="100"
      height="100"
      viewBox="0 0 72 72"
      fill="none"
      xmlns="http://www.w3.org/2000/svg">
      <path
        fillRule="evenodd"
        clipRule="evenodd"
        d="M36 69C54.2254 69 69 54.2254 69 36C69 17.7746 54.2254 3 36 3C17.7746 3 3 17.7746 3 36C3 54.2254 17.7746 69 36 69ZM52.1716 38.6337L28.4366 51.5801C26.4374 52.6705 24 51.2235 24 48.9464V23.0536C24 20.7764 26.4374 19.3295 28.4366 20.4199L52.1716 33.3663C54.2562 34.5034 54.2562 37.4966 52.1716 38.6337Z"
        fill="currentColor"
      />
    </svg>
  );
}
export function Heart({liked, animate}) {
  return (
    <>
      <svg
        className="absolute overflow-visible"
        viewBox="0 0 24 24"
        fill="none"
        xmlns="http://www.w3.org/2000/svg">
        <circle
          className={`circle ${liked ? 'liked' : ''} ${animate ? 'animate' : ''}`}
          cx="12"
          cy="12"
          r="11.5"
          fill="transparent"
          strokeWidth="0"
          stroke="currentColor"
        />
      </svg>

      <svg
        className={`heart ${liked ? 'liked' : ''} ${animate ? 'animate' : ''}`}
        viewBox="0 0 24 24"
        fill="none"
        xmlns="http://www.w3.org/2000/svg">
        {liked ? (
          <path
            d="M12 23a.496.496 0 0 1-.26-.074C7.023 19.973 0 13.743 0 8.