---
title: "'use server'"
titleForTitleTag: "'use server' directive"
---

<RSC>

`'use server'` é para usar com [usando React Server Components](/learn/start-a-new-react-project#bleeding-edge-react-frameworks).

</RSC>

<Intro>

`'use server'` marca funções do lado do servidor que podem ser chamadas do código do lado do cliente.

</Intro>

<InlineToc />

---

## Referência {/*reference*/}

### `'use server'` {/*use-server*/}

Adicione `'use server'` no topo do corpo de uma função assíncrona para marcar a função como chamável pelo cliente. Nós chamamos essas funções de [_Server Functions_](/reference/rsc/server-functions).

```js {2}
async function addToCart(data) {
  'use server';
  // ...
}
```

Ao chamar uma Server Function no cliente, ela fará uma requisição de rede para o servidor que inclui uma cópia serializada de quaisquer argumentos passados. Se a Server Function retornar um valor, esse valor será serializado e retornado para o cliente.

Em vez de marcar individualmente funções com `'use server'`, você pode adicionar a diretiva ao topo de um arquivo para marcar todas as exportações dentro desse arquivo como Server Functions que podem ser usadas em qualquer lugar, inclusive importadas no código do cliente.

#### Ressalvas {/*caveats*/}
* `'use server'` deve estar no início de sua função ou módulo; acima de qualquer outro código, incluindo imports (comentários acima das diretivas são OK). Elas devem ser escritas com aspas simples ou duplas, não crases.
* `'use server'` só pode ser usado em arquivos do lado do servidor. As Server Functions resultantes podem ser passadas para Client Components por meio de props. Veja os [tipos suportados para serialização](#serializable-parameters-and-return-values).
* Para importar Server Functions do [código do cliente](/reference/rsc/use-client), a diretiva deve ser usada em nível de módulo.
* Como as chamadas de rede subjacentes são sempre assíncronas, `'use server'` só pode ser usado em funções assíncronas.
* Sempre trate os argumentos para Server Functions como entrada não confiável e autorize quaisquer mutações. Veja [considerações de segurança](#security).
* Server Functions devem ser chamadas em uma [Transition](/reference/react/useTransition). Server Functions passadas para [`<form action>`](/reference/react-dom/components/form#props) ou [`formAction`](/reference/react-dom/components/input#props) serão automaticamente chamadas em uma transição.
* Server Functions são projetadas para mutações que atualizam o estado do lado do servidor; elas não são recomendadas para busca de dados. Consequentemente, frameworks que implementam Server Functions tipicamente processam uma ação por vez e não têm uma maneira de cachear o valor de retorno.

### Considerações de segurança {/*security*/}

Os argumentos para Server Functions são totalmente controlados pelo cliente. Por segurança, sempre trate-os como entrada não confiável e certifique-se de validar e escapar os argumentos conforme apropriado.

Em qualquer Server Function, certifique-se de validar se o usuário logado tem permissão para executar essa ação.

<Wip>

Para evitar o envio de dados sensíveis de um Server Function, existem APIs de taint experimentais para impedir que valores e objetos exclusivos sejam passados para o código do cliente.

Veja [experimental_taintUniqueValue](/reference/react/experimental_taintUniqueValue) e [experimental_taintObjectReference](/reference/react/experimental_taintObjectReference).

</Wip>

### Argumentos e valores de retorno serializáveis {/*serializable-parameters-and-return-values*/}

Como o código do cliente chama a Server Function pela rede, quaisquer argumentos passados precisarão ser serializáveis.

Aqui estão os tipos suportados para argumentos de Server Function:

* Primitivos
	* [string](https://developer.mozilla.org/en-US/docs/Glossary/String)
	* [number](https://developer.mozilla.org/en-US/docs/Glossary/Number)
	* [bigint](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt)
	* [boolean](https://developer.mozilla.org/en-US/docs/Glossary/Boolean)
	* [undefined](https://developer.mozilla.org/en-US/docs/Glossary/Undefined)
	* [null](https://developer.mozilla.org/en-US/docs/Glossary/Null)
	* [symbol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol), apenas símbolos registrados no registro global de Symbol via [`Symbol.for`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/for)
* Iteráveis contendo valores serializáveis
	* [String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)
	* [Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)
	* [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)
	* [Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set)
	* [TypedArray](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray) e [ArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer)
* [Date](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date)
* Instâncias de [FormData](https://developer.mozilla.org/en-US/docs/Web/API/FormData)
* [Objetos](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object) simples: aqueles criados com [inicializadores de objeto](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer), com propriedades serializáveis
* Funções que são Server Functions
* [Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)

Notavelmente, estes não são suportados:
* Elementos React, ou [JSX](/learn/writing-markup-with-jsx)
* Funções, incluindo funções de componente ou qualquer outra função que não seja uma Server Function
* [Classes](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/Classes_in_JavaScript)
* Objetos que são instâncias de qualquer classe (além das embutidas mencionadas) ou objetos com [um protótipo nulo](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object#null-prototype_objects)
* Símbolos não registrados globalmente, ex. `Symbol('my new symbol')`
* Eventos de event handlers

Valores de retorno serializáveis suportados são os mesmos que [props serializáveis](/reference/rsc/use-client#passing-props-from-server-to-client-components) para um Client Component de limite.

## Uso {/*usage*/}

### Server Functions em forms {/*server-functions-in-forms*/}

O caso de uso mais comum de Server Functions será chamar funções que mutam dados. No navegador, o [elemento HTML form](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form) é a abordagem tradicional para um usuário enviar uma mutação. Com React Server Components, React apresenta suporte de primeira classe para Server Functions como Actions em [forms](/reference/react-dom/components/form).

Aqui está um form que permite que um usuário solicite um nome de usuário.

```js [[1, 3, "formData"]]
// App.js

async function requestUsername(formData) {
  'use server';
  const username = formData.get('username');
  // ...
}

export default function App() {
  return (
    <form action={requestUsername}>
      <input type="text" name="username" />
      <button type="submit">Request</button>
    </form>
  );
}
```

Neste exemplo, `requestUsername` é uma Server Function passada para um `<form>`. Quando um usuário envia este form, existe uma requisição de rede para a server function `requestUsername`. Ao chamar uma Server Function em um form, React fornecerá o <CodeStep step={1}>[FormData](https://developer.mozilla.org/en-US/docs/Web/API/FormData)</CodeStep> do form como o primeiro argumento para a Server Function.

Ao passar uma Server Function para a form `action`, React pode [aprimorar progressivamente](https://developer.mozilla.org/en-US/docs/Glossary/Progressive_Enhancement) o form. Isso significa que os forms podem ser enviados antes que o bundle JavaScript seja carregado.

#### Manipulando valores de retorno em forms {/*handling-return-values*/}

No form de requisição de nome de usuário, pode haver a chance de que um nome de usuário não esteja disponível. `requestUsername` deve nos dizer se falha ou não.

Para atualizar a UI com base no resultado de uma Server Function enquanto suporta o aprimoramento progressivo, use [`useActionState`](/reference/react/useActionState).

```js
// requestUsername.js
'use server';

export default async function requestUsername(formData) {
  const username = formData.get('username');
  if (canRequest(username)) {
    // ...
    return 'successful';
  }
  return 'failed';
}
```

```js {4,8}, [[2, 2, "'use client'"]]
// UsernameForm.js
'use client';

import { useActionState } from 'react';
import requestUsername from './requestUsername';

function UsernameForm() {
  const [state, action] = useActionState(requestUsername, null, 'n/a');

  return (
    <>
      <form action={action}>
        <input type="text" name="username" />
        <button type="submit">Request</button>
      </form>
      <p>Última requisição de submissão retornou: {state}</p>
    </>
  );
}
```

Observe que, como a maioria dos Hooks, `useActionState` só pode ser chamado em <CodeStep step={1}>[código do cliente](/reference/rsc/use-client)</CodeStep>.

### Chamando uma Server Function fora de `<form>` {/*calling-a-server-function-outside-of-form*/}

Server Functions são endpoints de servidor expostos e podem ser chamados em qualquer lugar no código do cliente.

Ao usar uma Server Function fora de um [form](/reference/react-dom/components/form), chame a Server Function em uma [Transition](/reference/react/useTransition), que permite que você exiba um indicador de carregamento, mostre [atualizações de estado otimistas](/reference/react/useOptimistic) e lide com erros inesperados. Forms automaticamente encapsularão Server Functions em transitions.

```js {9-12}
import incrementLike from './actions';
import { useState, useTransition } from 'react';

function LikeButton() {
  const [isPending, startTransition] = useTransition();
  const [likeCount, setLikeCount] = useState(0);

  const onClick = () => {
    startTransition(async () => {
      const currentCount = await incrementLike();
      setLikeCount(currentCount);
    });
  };

  return (
    <>
      <p>Total de Curtidas: {likeCount}</p>
      <button onClick={onClick} disabled={isPending}>Curtir</button>;
    </>
  );
}
```

```js
// actions.js
'use server';

let likeCount = 0;
export default async function incrementLike() {
  likeCount++;
  return likeCount;
}
```

Para ler um valor de retorno de Server Function, você precisará de `await` na promise retornada.
```