---
title: experimental_taintObjectReference
---

<Wip>

**Esta API é experimental e ainda não está disponível em uma versão estável do React.**

Você pode experimentá-la atualizando os pacotes React para a versão experimental mais recente:

- `react@experimental`
- `react-dom@experimental`
- `eslint-plugin-react-hooks@experimental`

As versões experimentais do React podem conter erros. Não as use em produção.

Esta API está disponível apenas dentro de Componentes Servidor React.

</Wip>


<Intro>

`taintObjectReference` permite que você impeça que uma instância de objeto específico seja passada para um Componente Cliente, como um objeto `user`.

```js
experimental_taintObjectReference(message, object);
```

Para impedir a passagem de uma chave, hash ou token, consulte [`taintUniqueValue`](/reference/react/experimental_taintUniqueValue).

</Intro>

<InlineToc />

---

## Referência {/*reference*/}

### `taintObjectReference(message, object)` {/*taintobjectreference*/}

Chame `taintObjectReference` com um objeto para registrá-lo no React como algo que não deve ser permitido ser passado para o Cliente como está:

```js
import {experimental_taintObjectReference} from 'react';

experimental_taintObjectReference(
  'Não passe TODAS as variáveis de ambiente para o cliente.',
  process.env
);
```

[Veja mais exemplos abaixo.](#usage)

#### Parâmetros {/*parameters*/}

* `message`: A mensagem que você deseja exibir se o objeto for passado para um Componente Cliente. Essa mensagem será exibida como parte do Erro que será lançado se o objeto for passado para um Componente Cliente.

* `object`: O objeto a ser contaminado. Funções e instâncias de classe podem ser passadas para `taintObjectReference` como `object`. Funções e classes já estão bloqueadas de serem passadas para Componentes Cliente, mas a mensagem de erro padrão do React será substituída pelo que você definiu em `message`. Quando uma instância específica de um Typed Array é passada para `taintObjectReference` como `object`, quaisquer outras cópias do Typed Array não serão contaminadas.

#### Retorna {/*returns*/}

`experimental_taintObjectReference` retorna `undefined`.

#### Ressalvas {/*caveats*/}

- Recriar ou clonar um objeto contaminado cria um novo objeto não contaminado que pode conter dados confidenciais. Por exemplo, se você tiver um objeto `user` contaminado, `const userInfo = {name: user.name, ssn: user.ssn}` ou `{...user}` criará novos objetos que não estão contaminados. `taintObjectReference` só protege contra erros simples quando o objeto é passado para um Componente Cliente inalterado.

<Pitfall>

**Não confie apenas em contaminar para segurança.** Contaminar um objeto não impede o vazamento de todos os valores derivados possíveis. Por exemplo, o clone de um objeto contaminado criará um novo objeto não contaminado. Usar dados de um objeto contaminado (por exemplo, `{secret: taintedObj.secret}`) criará um novo valor ou objeto que não está contaminado. Contaminar é uma camada de proteção; um aplicativo seguro terá várias camadas de proteção, APIs bem projetadas e padrões de isolamento.

</Pitfall>

---

## Uso {/*usage*/}

### Impedir que dados do usuário atinjam o cliente intencionalmente {/*prevent-user-data-from-unintentionally-reaching-the-client*/}

Um Componente Cliente nunca deve aceitar objetos que contenham dados sensíveis. Idealmente, as funções de busca de dados não devem expor dados que o usuário atual não deveria ter acesso. Às vezes, erros acontecem durante a refatoração. Para proteger contra esses erros que acontecem no futuro, podemos "contaminar" o objeto do usuário em nossa API de dados.

```js
import {experimental_taintObjectReference} from 'react';

export async function getUser(id) {
  const user = await db`SELECT * FROM users WHERE id = ${id}`;
  experimental_taintObjectReference(
    'Não passe o objeto de usuário inteiro para o cliente. ' +
      'Em vez disso, escolha as propriedades específicas que você precisa para este caso de uso.',
    user,
  );
  return user;
}
```

Agora, sempre que alguém tentar passar este objeto para um Componente Cliente, um erro será lançado com a mensagem de erro passada.

<DeepDive>

#### Protegendo contra vazamentos na busca de dados {/*protecting-against-leaks-in-data-fetching*/}

Se você estiver executando um ambiente de Componentes Servidor que tenha acesso a dados confidenciais, você deve ter cuidado para não passar objetos diretamente:

```js
// api.js
export async function getUser(id) {
  const user = await db`SELECT * FROM users WHERE id = ${id}`;
  return user;
}
```

```js
import { getUser } from 'api.js';
import { InfoCard } from 'components.js';

export async function Profile(props) {
  const user = await getUser(props.userId);
  // NÃO FAÇA ISSO
  return <InfoCard user={user} />;
}
```

```js
// components.js
"use client";

export async function InfoCard({ user }) {
  return <div>{user.name}</div>;
}
```

Idealmente, `getUser` não deve expor dados que o usuário atual não deveria ter acesso. Para evitar a passagem do objeto `user` para um Componente Cliente no futuro, podemos "contaminar" o objeto do usuário:

```js
// api.js
import {experimental_taintObjectReference} from 'react';

export async function getUser(id) {
  const user = await db`SELECT * FROM users WHERE id = ${id}`;
  experimental_taintObjectReference(
    'Não passe o objeto de usuário inteiro para o cliente. ' +
      'Em vez disso, escolha as propriedades específicas que você precisa para este caso de uso.',
    user,
  );
  return user;
}
```

Agora, se alguém tentar passar o objeto `user` para um Componente Cliente, um erro será lançado com a mensagem de erro passada.

</DeepDive>
```