---
title: experimental_taintUniqueValue
---

<Wip>

**Esta API é experimental e ainda não está disponível em uma versão estável do React.**

Você pode testá-la atualizando os pacotes do React para a versão experimental mais recente:

- `react@experimental`
- `react-dom@experimental`
- `eslint-plugin-react-hooks@experimental`

As versões experimentais do React podem conter erros. Não as use em produção.

Esta API só está disponível dentro de [Componentes do Servidor React](/reference/rsc/use-client).

</Wip>

<Intro>

`taintUniqueValue` permite que você impeça que valores exclusivos sejam passados para Componentes do Cliente, como senhas, chaves ou tokens.

```js
taintUniqueValue(errMessage, lifetime, value)
```

Para impedir a passagem de um objeto contendo dados confidenciais, consulte [`taintObjectReference`](/reference/react/experimental_taintObjectReference).

</Intro>

<InlineToc />

---

## Referência {/*reference*/}

### `taintUniqueValue(message, lifetime, value)` {/*taintuniquevalue*/}

Chame `taintUniqueValue` com uma senha, token, chave ou hash para registrá-lo com o React como algo que não deve ser permitido que seja passado para o Cliente como está:

```js
import {experimental_taintUniqueValue} from 'react';

experimental_taintUniqueValue(
  'Não passe chaves secretas para o cliente.',
  process,
  process.env.SECRET_KEY
);
```

[Veja mais exemplos abaixo.](#usage)

#### Parâmetros {/*parameters*/}

* `message`: A mensagem que você deseja exibir se `value` for passado para um Componente do Cliente. Esta mensagem será exibida como parte do Erro que será lançado se `value` for passado para um Componente do Cliente.

* `lifetime`: Qualquer objeto que indique por quanto tempo `value` deve ser contaminado. `value` será bloqueado de ser enviado a qualquer Componente do Cliente enquanto este objeto ainda existir. Por exemplo, passar `globalThis` bloqueia o valor pela duração de um aplicativo. `lifetime` é tipicamente um objeto cujas propriedades contêm `value`.

* `value`: Uma string, bigint ou TypedArray. `value` deve ser uma sequência exclusiva de caracteres ou bytes com alta entropia, como um token criptográfico, chave privada, hash ou uma senha longa. `value` será bloqueado de ser enviado a qualquer Componente do Cliente.

#### Retorna {/*returns*/}

`experimental_taintUniqueValue` retorna `undefined`.

#### Ressalvas {/*caveats*/}

* Derivar novos valores de valores contaminados pode comprometer a proteção contra contaminação. Novos valores criados por meio de conversão de valores contaminados para letras maiúsculas, concatenação de valores de string contaminadas em uma string maior, conversão de valores contaminados para base64, substring de valores contaminados e outras transformações semelhantes não são contaminados, a menos que você chame explicitamente `taintUniqueValue` nesses valores recém-criados.
* Não use `taintUniqueValue` para proteger valores de baixa entropia, como códigos PIN ou números de telefone. Se qualquer valor em uma requisição for controlado por um invasor, ele poderá inferir qual valor está contaminado enumerando todos os valores possíveis do segredo.

---

## Uso {/*usage*/}

### Impedir que um token seja passado para Componentes do Cliente {/*prevent-a-token-from-being-passed-to-client-components*/}

Para garantir que informações confidenciais, como senhas, tokens de sessão ou outros valores exclusivos, não sejam acidentalmente passadas para Componentes do Cliente, a função `taintUniqueValue` fornece uma camada de proteção. Quando um valor é contaminado, qualquer tentativa de passá-lo para um Componente do Cliente resultará em um erro.

O argumento `lifetime` define a duração pela qual o valor permanece contaminado. Para valores que devem permanecer contaminados indefinidamente, objetos como [`globalThis`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/globalThis) ou `process` podem servir como argumento `lifetime`. Esses objetos têm um tempo de vida que abrange toda a duração da execução do seu aplicativo.

```js
import {experimental_taintUniqueValue} from 'react';

experimental_taintUniqueValue(
  'Não passe uma senha de usuário para o cliente.',
  globalThis,
  process.env.SECRET_KEY
);
```

Se o tempo de vida do valor contaminado estiver relacionado a um objeto, o `lifetime` deve ser o objeto que encapsula o valor. Isso garante que o valor contaminado permaneça protegido durante a vida útil do objeto encapsulador.

```js
import {experimental_taintUniqueValue} from 'react';

export async function getUser(id) {
  const user = await db`SELECT * FROM users WHERE id = ${id}`;
  experimental_taintUniqueValue(
    'Não passe um token de sessão do usuário para o cliente.',
    user,
    user.session.token
  );
  return user;
}
```

Neste exemplo, o objeto `user` serve como argumento `lifetime`. Se este objeto for armazenado em um cache global ou for acessível por outra requisição, o token de sessão permanecerá contaminado.

<Pitfall>

**Não confie apenas na contaminação para segurança.** Contaminar um valor não bloqueia todos os valores derivados possíveis. Por exemplo, a criação de um novo valor por meio da conversão de uma string contaminada para maiúsculas não contaminará o novo valor.

```js
import {experimental_taintUniqueValue} from 'react';

const password = 'correct horse battery staple';

experimental_taintUniqueValue(
  'Não passe a senha para o cliente.',
  globalThis,
  password
);

const uppercasePassword = password.toUpperCase() // `uppercasePassword` não está contaminado
```

Neste exemplo, a constante `password` está contaminada. Em seguida, `password` é usado para criar um novo valor `uppercasePassword` chamando o método `toUpperCase` em `password`. O recém-criado `uppercasePassword` não está contaminado.

Outras formas semelhantes de derivar novos valores de valores contaminados, como concatená-los em uma string maior, convertê-los para base64 ou retornar uma substring, criam valores não contaminados.

A contaminação protege apenas contra erros simples, como passar explicitamente valores secretos para o cliente. Os erros em chamar `taintUniqueValue`, como usar um armazenamento global fora do React, sem o objeto lifetime correspondente, podem fazer com que o valor contaminado se torne não contaminado. A contaminação é uma camada de proteção; um aplicativo seguro terá múltiplas camadas de proteção, APIs bem projetadas e padrões de isolamento.

</Pitfall>

<DeepDive>

#### Usando `server-only` e `taintUniqueValue` para evitar vazamento de segredos {/*using-server-only-and-taintuniquevalue-to-prevent-leaking-secrets*/}

Se você estiver executando um ambiente de Componentes do Servidor que tem acesso a chaves privadas ou senhas, como senhas de banco de dados, você deve ter cuidado para não passá-las para um Componente do Cliente.

```js
export async function Dashboard(props) {
  // NÃO FAÇA ISSO
  return <Overview password={process.env.API_PASSWORD} />;
}
```

```js
"use client";

import {useEffect} from '...'

export async function Overview({ password }) {
  useEffect(() => {
    const headers = { Authorization: password };
    fetch(url, { headers }).then(...);
  }, [password]);
  ...
}
```

Este exemplo vazaria o token secreto da API para o cliente. Se este token de API puder ser usado para acessar dados que este usuário em particular não deveria ter acesso, isso pode levar a uma violação de dados.

[comment]: <> (TODO: Link to `server-only` docs once they are written)

Idealmente, segredos como esse são abstraídos em um único arquivo auxiliar que só pode ser importado por utilitários de dados confiáveis no servidor. O auxiliar pode até ser marcado com [`server-only`](https://www.npmjs.com/package/server-only) para garantir que este arquivo não seja importado no cliente.

```js
import "server-only";

export function fetchAPI(url) {
  const headers = { Authorization: process.env.API_PASSWORD };
  return fetch(url, { headers });
}
```

Às vezes, erros acontecem durante a refatoração e nem todos os seus colegas podem saber sobre isso.
Para se proteger contra esses erros acontecendo no futuro, podemos "contaminar" a senha real:

```js
import "server-only";
import {experimental_taintUniqueValue} from 'react';

experimental_taintUniqueValue(
  'Não passe a senha do token da API para o cliente. ' +
    'Em vez disso, faça todas as buscas no servidor.',
  process,
  process.env.API_PASSWORD
);
```

Agora, sempre que alguém tentar passar esta senha para um Componente do Cliente, ou enviar a senha para um Componente do Cliente com uma Função do Servidor, um erro será lançado com a mensagem que você definiu ao chamar `taintUniqueValue`.

</DeepDive>

---
```