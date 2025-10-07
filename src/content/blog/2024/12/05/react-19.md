---
title: "React v19"
author: The React Team
date: 2024/12/05
description: React 19 já está disponível no npm! Neste post, daremos uma visão geral dos novos recursos do React 19 e como você pode adotá-los.
---

05 de Dezembro de 2024 por [The React Team](/community/team)

---
<Note>

### React 19 está estável! {/*react-19-is-now-stable*/}

Adições desde que este post foi originalmente compartilhado com o React 19 RC em abril:

- **Pré-aquecimento para árvores suspensas**: veja [Melhorias no Suspense](/blog/2024/04/25/react-19-upgrade-guide#improvements-to-suspense).
- **APIs Estáticas do React DOM**: veja [Novas APIs Estáticas do React DOM](#new-react-dom-static-apis).

_A data deste post foi atualizada para refletir a data de lançamento estável._

</Note>

<Intro>

React v19 já está disponível no npm!

</Intro>

Em nosso [Guia de Atualização do React 19](/blog/2024/04/25/react-19-upgrade-guide), compartilhamos instruções passo a passo para atualizar seu aplicativo para o React 19. Neste post, daremos uma visão geral dos novos recursos do React 19 e como você pode adotá-los.

- [Novidades no React 19](#whats-new-in-react-19)
- [Melhorias no React 19](#improvements-in-react-19)
- [Como atualizar](#how-to-upgrade)

Para uma lista de alterações que podem quebrar a compatibilidade, veja o [Guia de Atualização](/blog/2024/04/25/react-19-upgrade-guide).

---

## Novidades no React 19 {/*whats-new-in-react-19*/}

### Actions {/*actions*/}

Um caso de uso comum em aplicativos React é realizar uma mutação de dados e, em seguida, atualizar o estado em resposta. Por exemplo, quando um usuário envia um formulário para alterar seu nome, você fará uma solicitação de API e, em seguida, lidará com a resposta. No passado, você precisaria lidar manualmente com estados pendentes, erros, atualizações otimistas e solicitações sequenciais.

Por exemplo, você poderia lidar com o estado pendente e de erro em `useState`:

```js
// Antes das Actions
function UpdateName({}) {
  const [name, setName] = useState("");
  const [error, setError] = useState(null);
  const [isPending, setIsPending] = useState(false);

  const handleSubmit = async () => {
    setIsPending(true);
    const error = await updateName(name);
    setIsPending(false);
    if (error) {
      setError(error);
      return;
    } 
    redirect("/path");
  };

  return (
    <div>
      <input value={name} onChange={(event) => setName(event.target.value)} />
      <button onClick={handleSubmit} disabled={isPending}>
        Atualizar
      </button>
      {error && <p>{error}</p>}
    </div>
  );
}
```

No React 19, adicionamos suporte para usar funções assíncronas em transições para lidar automaticamente com estados pendentes, erros, formulários e atualizações otimistas.

Por exemplo, você pode usar `useTransition` para lidar com o estado pendente para você:

```js
// Usando o estado pendente das Actions
function UpdateName({}) {
  const [name, setName] = useState("");
  const [error, setError] = useState(null);
  const [isPending, startTransition] = useTransition();

  const handleSubmit = () => {
    startTransition(async () => {
      const error = await updateName(name);
      if (error) {
        setError(error);
        return;
      } 
      redirect("/path");
    })
  };

  return (
    <div>
      <input value={name} onChange={(event) => setName(event.target.value)} />
      <button onClick={handleSubmit} disabled={isPending}>
        Atualizar
      </button>
      {error && <p>{error}</p>}
    </div>
  );
}
```

A transição assíncrona definirá imediatamente o estado `isPending` como `true`, fará a(s) solicitação(ões) assíncrona(s) e definirá `isPending` como `false` após quaisquer transições. Isso permite que você mantenha a interface do usuário atual responsiva e interativa enquanto os dados estão mudando.

<Note>

#### Por convenção, funções que usam transições assíncronas são chamadas de "Actions". {/*by-convention-functions-that-use-async-transitions-are-called-actions*/}

As Actions gerenciam automaticamente o envio de dados para você:

- **Estado pendente**: As Actions fornecem um estado pendente que começa no início de uma solicitação e é redefinido automaticamente quando a atualização de estado final é confirmada.
- **Atualizações otimistas**: As Actions suportam o novo hook [`useOptimistic`](#new-hook-optimistic-updates) para que você possa mostrar feedback instantâneo aos usuários enquanto as solicitações estão sendo enviadas.
- **Tratamento de erros**: As Actions fornecem tratamento de erros para que você possa exibir Error Boundaries quando uma solicitação falhar e reverter atualizações otimistas para seu valor original automaticamente.
- **Formulários**: Elementos `<form>` agora suportam a passagem de funções para as props `action` e `formAction`. Passar funções para as props `action` usam Actions por padrão e redefinem o formulário automaticamente após o envio.

</Note>

Com base nas Actions, o React 19 introduz [`useOptimistic`](#new-hook-optimistic-updates) para gerenciar atualizações otimistas e um novo hook [`React.useActionState`](#new-hook-useactionstate) para lidar com casos comuns para Actions. Em `react-dom`, estamos adicionando [`<form>` Actions](#form-actions) para gerenciar formulários automaticamente e [`useFormStatus`](#new-hook-useformstatus) para suportar os casos comuns para Actions em formulários.

No React 19, o exemplo acima pode ser simplificado para:

```js
// Usando <form> Actions e useActionState
function ChangeName({ name, setName }) {
  const [error, submitAction, isPending] = useActionState(
    async (previousState, formData) => {
      const error = await updateName(formData.get("name"));
      if (error) {
        return error;
      }
      redirect("/path");
      return null;
    },
    null,
  );

  return (
    <form action={submitAction}>
      <input type="text" name="name" />
      <button type="submit" disabled={isPending}>Atualizar</button>
      {error && <p>{error}</p>}
    </form>
  );
}
```

Na próxima seção, detalharemos cada um dos novos recursos de Action no React 19.

### Novo hook: `useActionState` {/*new-hook-useactionstate*/}

Para facilitar os casos comuns para Actions, adicionamos um novo hook chamado `useActionState`:

```js
const [error, submitAction, isPending] = useActionState(
  async (previousState, newName) => {
    const error = await updateName(newName);
    if (error) {
      // Você pode retornar qualquer resultado da action.
      // Aqui, retornamos apenas o erro.
      return error;
    }

    // Lidar com o sucesso
    return null;
  },
  null,
);
```

`useActionState` aceita uma função (a "Action") e retorna uma Action encapsulada para chamar. Isso funciona porque as Actions se compõem. Quando a Action encapsulada é chamada, `useActionState` retornará o último resultado da Action como `data` e o estado pendente da Action como `pending`.

<Note>

`React.useActionState` foi anteriormente chamado `ReactDOM.useFormState` nos lançamentos Canary, mas o renomeamos e descontinuamos `useFormState`.

Veja [#28491](https://github.com/facebook/react/pull/28491) para mais informações.

</Note>

Para mais informações, veja a documentação de [`useActionState`](/reference/react/useActionState).

### React DOM: `<form>` Actions {/*form-actions*/}

As Actions também são integradas com os novos recursos `<form>` do React 19 para `react-dom`. Adicionamos suporte para passar funções como props `action` e `formAction` dos elementos `<form>`, `<input>` e `<button>` para enviar formulários automaticamente com Actions:

```js [[1,1,"actionFunction"]]
<form action={actionFunction}>
```

Quando uma Action de `<form>` for bem-sucedida, o React redefinirá automaticamente o formulário para componentes não controlados. Se você precisar redefinir o `<form>` manualmente, poderá chamar a nova API do React DOM `requestFormReset`.

Para mais informações, veja a documentação do `react-dom` para [`<form>`](/reference/react-dom/components/form), [`<input>`](/reference/react-dom/components/input) e `<button>`.

### React DOM: Novo hook: `useFormStatus` {/*new-hook-useformstatus*/}

Em sistemas de design, é comum escrever componentes de design que precisam de acesso a informações sobre o `<form>` em que estão, sem precisar passar props para o componente. Isso pode ser feito via Context, mas para facilitar o caso comum, adicionamos um novo hook `useFormStatus`:

```js [[1, 4, "pending"], [1, 5, "pending"]]
import {useFormStatus} from 'react-dom';

function DesignButton() {
  const {pending} = useFormStatus();
  return <button type="submit" disabled={pending} />
}
```

`useFormStatus` lê o status do `<form>` pai como se o formulário fosse um provedor de Context.

Para mais informações, veja a documentação do `react-dom` para [`useFormStatus`](/reference/react-dom/hooks/useFormStatus).

### Novo hook: `useOptimistic` {/*new-hook-optimistic-updates*/}

Outro padrão comum de UI ao realizar uma mutação de dados é mostrar o estado final otimisticamente enquanto a solicitação assíncrona está em andamento. No React 19, adicionamos um novo hook chamado `useOptimistic` para facilitar isso:

```js {2,6,13,19}
function ChangeName({currentName, onUpdateName}) {
  const [optimisticName, setOptimisticName] = useOptimistic(currentName);

  const submitAction = async formData => {
    const newName = formData.get("name");
    setOptimisticName(newName);
    const updatedName = await updateName(newName);
    onUpdateName(updatedName);
  };

  return (
    <form action={submitAction}>
      <p>Seu nome é: {optimisticName}</p>
      <p>
        <label>Mudar Nome:</label>
        <input
          type="text"
          name="name"
          disabled={currentName !== optimisticName}
        />
      </p>
    </form>
  );
}
```

O hook `useOptimistic` renderizará imediatamente o `optimisticName` enquanto a solicitação `updateName` estiver em andamento. Quando a atualização terminar ou ocorrer um erro, o React voltará automaticamente para o valor `currentName`.

Para mais informações, veja a documentação de [`useOptimistic`](/reference/react/useOptimistic).

### Nova API: `use` {/*new-feature-use*/}

No React 19, estamos introduzindo uma nova API para ler recursos em renderização: `use`.

Por exemplo, você pode ler uma promise com `use`, e o React suspenderá até que a promise seja resolvida:

```js {1,5}
import {use} from 'react';

function Comments({commentsPromise}) {
  // `use` suspenderá até que a promise seja resolvida.
  const comments = use(commentsPromise);
  return comments.map(comment => <p key={comment.id}>{comment}</p>);
}

function Page({commentsPromise}) {
  // Quando `use` suspender em Comments,
  // este boundary de Suspense será mostrado.
  return (
    <Suspense fallback={<div>Carregando...</div>}>
      <Comments commentsPromise={commentsPromise} />
    </Suspense>
  )
}
```

<Note>

#### `use` não suporta promises criadas em renderização. {/*use-does-not-support-promises-created-in-render*/}

Se você tentar passar uma promise criada em renderização para `use`, o React avisará:

<ConsoleBlockMulti>

<ConsoleLogLine level="error">

Um componente foi suspenso por uma promise não cacheada. Criar promises dentro de um Componente Cliente ou hook ainda não é suportado, exceto através de uma biblioteca ou framework compatível com Suspense.

</ConsoleLogLine>

</ConsoleBlockMulti>

Para corrigir, você precisa passar uma promise de uma biblioteca ou framework com suporte a Suspense que suporte cache para promises. No futuro, planejamos lançar recursos para facilitar o cache de promises em renderização.

</Note>

Você também pode ler o contexto com `use`, permitindo que você leia o Contexto condicionalmente, como após retornos antecipados:

```js {1,11}
import {use} from 'react';
import ThemeContext from './ThemeContext'

function Heading({children}) {
  if (children == null) {
    return null;
  }
  
  // Isso não funcionaria com useContext
  // por causa do retorno antecipado.
  const theme = use(ThemeContext);
  return (
    <h1 style={{color: theme.color}}>
      {children}
    </h1>
  );
}
```

A API `use` só pode ser chamada em renderização, semelhante aos hooks. Diferente dos hooks, `use` pode ser chamado condicionalmente. No futuro, planejamos suportar mais maneiras de consumir recursos em renderização com `use`.

Para mais informações, veja a documentação de [`use`](/reference/react/use).

## Novas APIs Estáticas do React DOM {/*new-react-dom-static-apis*/}

Adicionamos duas novas APIs a `react-dom/static` para geração de sites estáticos:
- [`prerender`](/reference/react-dom/static/prerender)
- [`prerenderToNodeStream`](/reference/react-dom/static/prerenderToNodeStream)

Essas novas APIs aprimoram `renderToString` esperando os dados carregarem para a geração de HTML estático. Elas são projetadas para funcionar com ambientes de streaming como Node.js Streams e Web Streams. Por exemplo, em um ambiente de Web Stream, você pode pré-renderizar uma árvore React para HTML estático com `prerender`:

```js
import { prerender } from 'react-dom/static';

async function handler(request) {
  const {prelude} = await prerender(<App />, {
    bootstrapScripts: ['/main.js']
  });
  return new Response(prelude, {
    headers: { 'content-type': 'text/html' },
  });
}
```

As APIs de pré-renderização esperarão todos os dados carregarem antes de retornar o stream de HTML estático. Os streams podem ser convertidos em strings ou enviados com uma resposta de streaming. Eles não suportam streaming de conteúdo enquanto ele carrega, o que é suportado pelas APIs existentes de [renderização de servidor do React DOM](/reference/react-dom/server).

Para mais informações, veja [APIs Estáticas do React DOM](/reference/react-dom/static).

## Componentes de Servidor React {/*react-server-components*/}

### Componentes de Servidor {/*server-components*/}

Componentes de Servidor são uma nova opção que permite renderizar componentes antecipadamente, antes do bundling, em um ambiente separado do seu aplicativo cliente ou servidor SSR. Este ambiente separado é o "servidor" nos Componentes de Servidor React. Os Componentes de Servidor podem ser executados uma vez no tempo de compilação no seu servidor de CI, ou podem ser executados para cada solicitação usando um servidor web.

O React 19 inclui todos os recursos de Componentes de Servidor React incluídos do canal Canary. Isso significa que bibliotecas que são distribuídas com Componentes de Servidor agora podem ter como alvo o React 19 como uma dependência peer com uma condição de exportação `react-server` [export condition](https://github.com/reactjs/rfcs/blob/main/text/0227-server-module-conventions.md#react-server-conditional-exports) para uso em frameworks que suportam a [Arquitetura Full-stack React](/learn/start-a-new-react-project#which-features-make-up-the-react-teams-full-stack-architecture-vision).


<Note>

#### Como eu crio suporte para Componentes de Servidor? {/*how-do-i-build-support-for-server-components*/}

Embora os Componentes de Servidor React no React 19 sejam estáveis e não quebrem entre versões menores, as APIs subjacentes usadas para implementar um bundler ou framework de Componentes de Servidor React não seguem semver e podem quebrar entre versões menores no React 19.x.

Para