---
title: Funções do Servidor
---

<RSC>

Funções do Servidor são para uso em [Componentes do Servidor React](/learn/start-a-new-react-project#bleeding-edge-react-frameworks).

**Observação:** Até setembro de 2024, nos referíamos a todas as Funções do Servidor como "Ações do Servidor". Se uma Função do Servidor for passada para uma propriedade `action` ou chamada de dentro de uma ação ,ela é uma Ação do Servidor, mas nem todas as Funções do Servidor são Ações do Servidor. A nomenclatura nesta documentação foi atualizada para refletir que as Funções do Servidor podem ser usadas para múltiplos propósitos.

</RSC>

<Intro>

Funções do Servidor permitem que os Componentes do Cliente chamem funções assíncronas executadas no servidor.

</Intro>

<InlineToc />

<Note>

#### Como crio suporte para Funções do Servidor? {/*how-do-i-build-support-for-server-functions*/}

Embora as Funções do Servidor no React 19 sejam estáveis e não quebrem entre versões secundárias, as APIs subjacentes usadas para implementar Funções do Servidor em um bundler ou framework de Componentes do Servidor React não seguem a semver e podem quebrar entre secundárias no React 19.x.

Para suportar Funções do Servidor como um bundler ou framework, recomendamos fixar em uma versão específica do React ou usar o release Canary. Continuaremos trabalhando com bundlers e frameworks para estabilizar as APIs usadas para implementar Funções do Servidor no futuro.

</Note>

Quando uma Função do Servidor é definida com a diretiva [`"use server"`](/reference/rsc/use-server), seu framework criará automaticamente uma referência à função do servidor e passará essa referência para o Componente do Cliente. Quando essa função é chamada no cliente, o React enviará uma requisição ao servidor para executar a função e retornar o resultado.

Funções do Servidor podem ser criadas em Componentes do Servidor e passadas como props para Componentes do Cliente, ou podem ser importadas e usadas em Componentes do Cliente.

## Uso {/*usage*/}

### Criando uma Função do Servidor a partir de um Componente do Servidor {/*creating-a-server-function-from-a-server-component*/}

Componentes do Servidor podem definir Funções do Servidor com a diretiva `"use server"`:

```js [[2, 7, "'use server'"], [1, 5, "createNoteAction"], [1, 12, "createNoteAction"]]
// Componente do Servidor
import Button from './Button';

function EmptyNote () {
  async function createNoteAction() {
    // Função do Servidor
    'use server';
    
    await db.notes.create();
  }

  return <Button onClick={createNoteAction}/>;
}
```

Quando o React renderiza a função `EmptyNote` do Servidor, ele criará uma referência à função `createNoteAction` e passará essa referência para o Componente do Cliente `Button`. Quando o botão for clicado, o React enviará uma requisição ao servidor para executar a função `createNoteAction` com a referência fornecida:

```js {5}
"use client";

export default function Button({onClick}) { 
  console.log(onClick); 
  // {$$typeof: Symbol.for("react.server.reference"), $$id: 'createNoteAction'}
  return <button onClick={() => onClick()}>Criar nota vazia</button>
}
```

Para saber mais, consulte a documentação para [`"use server"`](/reference/rsc/use-server).


### Importando Funções do Servidor de Componentes do Cliente {/*importing-server-functions-from-client-components*/}

Componentes do Cliente podem importar Funções do Servidor de arquivos que usam a diretiva `"use server"`:

```js [[1, 3, "createNote"]]
"use server";

export async function createNote() {
  await db.notes.create();
}

```

Quando o bundler cria o Componente do Cliente `EmptyNote`, ele criará uma referência à função `createNote` no bundle. Quando o `button` for clicado, o React environmentará uma requisição ao servidor para  executar a função `createNote` usando a referência fornecida:

```js [[1, 2, "createNote"], [1, 5, "createNote"], [1, 7, "createNote"]]
"use client";
import {createNote} from './actions';

function EmptyNote() {
  console.log(createNote);
  // {$$typeof: Symbol.for("react.server.reference"), $$id: 'createNote'}
  <button onClick={() => createNote()} />
}
```

Para saber mais, consulte a documentação para [`"use server"`](/reference/rsc/use-server).

### Funções do Servidor com Ações {/*server-functions-with-actions*/}

Funções do Servidor podem ser chamadas de Ações no cliente:

```js [[1, 3, "updateName"]]
"use server";

export async function updateName(name) {
  if (!name) {
    return {error: 'Name is required'};
  }
  await db.users.updateName(name);
}
```

```js [[1, 3, "updateName"], [1, 13, "updateName"], [2, 11, "submitAction"],  [2, 23, "submitAction"]]
"use client";

import {updateName} from './actions';

function UpdateName() {
  const [name, setName] = useState('');
  const [error, setError] = useState(null);

  const [isPending, startTransition] = useTransition();

  const submitAction = async () => {
    startTransition(async () => {
      const {error} = await updateName(name);
      if (!error) {
        setError(error);
      } else {
        setName('');
      }
    })
  }
  
  return (
    <form action={submitAction}>
      <input type="text" name="name" disabled={isPending}/>
      {state.error && <span>Falha: {state.error}</span>}
    </form>
  )
}
```

Isso permite que você acesse o estado `isPending` da Função do Servidor, encapsulando-a em uma Ação no cliente.

Para saber mais, consulte a documentação para [Chamando uma Função do Servidor fora de `<form>`](/reference/rsc/use-server#calling-a-server-function-outside-of-form)

### Funções do Servidor com Form Actions {/*using-server-functions-with-form-actions*/}

Funções do Servidor funcionam com os novos recursos de Form no React 19.

Você pode passar uma Função do Servidor para um Form para automaticamente submeter o formulário ao servidor:


```js [[1, 3, "updateName"], [1, 7, "updateName"]]
"use client";

import {updateName} from './actions';

function UpdateName() {
  return (
    <form action={updateName}>
      <input type="text" name="name" />
    </form>
  )
}
```

Quando a submissão do Form tiver sucesso, o React irá automaticamente redefinir o form. Você pode adicionar `useActionState` para acessar o estado pendente, última resposta, ou para suportar aprimoramento progressivo.

Para saber mais, consulte a documentação para [Funções do Servidor em Forms](/reference/rsc/use-server#server-functions-in-forms).

### Funções do Servidor com `useActionState` {/*server-functions-with-use-action-state*/}

Você pode chamar Funções do Servidor com `useActionState` para o caso comum em que você só precisa acessar o estado pendente da ação e a última resposta retornada:

```js [[1, 3, "updateName"], [1, 6, "updateName"], [2, 6, "submitAction"], [2, 9, "submitAction"]]
"use client";

import {updateName} from './actions';

function UpdateName() {
  const [state, submitAction, isPending] = useActionState(updateName, {error: null});

  return (
    <form action={submitAction}>
      <input type="text" name="name" disabled={isPending}/>
      {state.error && <span>Falha: {state.error}</span>}
    </form>
  );
}
```

Ao usar `useActionState` com Funções do Servidor, o React também irá automaticamente reproduzir as submissões do formulário que foram inseridas antes que a hidratação finalize. Isso significa que os usuários podem interagir com seu app mesmo antes do app ter hidratado.

Para saber mais, consulte a documentação para [`useActionState`](/reference/react-dom/hooks/useFormState).

### Aprimoramento progressivo com `useActionState` {/*progressive-enhancement-with-useactionstate*/}

Funções do Servidor também suportam o aprimoramento progressivo com o terceiro argumento de `useActionState`.

```js [[1, 3, "updateName"], [1, 6, "updateName"], [2, 6, "/name/update"], [3, 6, "submitAction"], [3, 9, "submitAction"]]
"use client";

import {updateName} from './actions';

function UpdateName() {
  const [, submitAction] = useActionState(updateName, null, `/name/update`);

  return (
    <form action={submitAction}>
      ...
    </form>
  );
}
```

Quando o <CodeStep step={2}>permalink</CodeStep> for fornecido para `useActionState`, o React irá redirecionar para a URL fornecida se o formulário for submetido antes que o pacote JavaScript seja carregado.

Para saber mais, consulte a documentação para [`useActionState`](/reference/react-dom/hooks/useFormState).
```