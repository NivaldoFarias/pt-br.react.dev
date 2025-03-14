---
title: experimental_taintObjectReference
---

<Wip>

**Esta API é experimental e ainda não está disponível em uma versão estável do React.**

Você pode testá-la atualizando os pacotes do React para a versão experimental mais recente:

- `react@experimental`
- `react-dom@experimental`
- `eslint-plugin-react-hooks@experimental`

Versões experimentais do React podem conter **erros**. Não as use em **produção**.

Esta API só está disponível dentro de Componentes de Servidor React.

</Wip>

<Intro>

`taintObjectReference` permite que você impeça que uma instância de objeto específico seja passada a um Componente de Cliente, como um objeto `user`.

```js
experimental_taintObjectReference(message, object);
```

Para impedir a passagem de uma chave, hash ou token, consulte [`taintUniqueValue`](/reference/react/experimental_taintUniqueValue).

</Intro>

<InlineToc />

---

## Referência {/*reference*/}

### `taintObjectReference(message, object)` {/*taintobjectreference*/}

Chame `taintObjectReference` com um objeto para registrá-lo com o React como algo que não deve ser permitido que seja passado para o Cliente como está:

```js
import {experimental_taintObjectReference} from 'react';

experimental_taintObjectReference(
  'Não passe TODAS as variáveis de ambiente para o cliente.',
  process.env
);
```

[Veja mais exemplos abaixo.](#usage)

#### Parâmetros {/*parameters*/}

* `message`: A mensagem que você deseja exibir se o objeto for passado para um Componente de Cliente. Essa mensagem será exibida como parte do Erro que será lançado se o objeto for passado para um Componente de Cliente.

* `object`: O objeto a ser contaminado. Funções e instâncias de classe podem ser passadas para `taintObjectReference` como `object`. Funções e classes já são bloqueadas de serem passadas para Componentes de Cliente, mas a mensagem de erro padrão do React será substituída pelo que você definiu em `message`. Quando uma instância específica de um Typed Array é passada para `taintObjectReference` como `object`, quaisquer outras cópias do Typed Array não serão contaminadas.

#### Retorna {/*returns*/}

`experimental_taintObjectReference` retorna `undefined`.

#### Ressalvas {/*caveats*/}

- A recriação ou clonagem de um objeto contaminado cria um novo objeto não contaminado que pode conter dados sensíveis. Por exemplo, se você tiver um objeto `user` contaminado, `const userInfo = {name: user.name, ssn: user.ssn}` or `{...user}` criará novos objetos que não são contaminados. `taintObjectReference` protege apenas contra erros simples quando o objeto é passado para um Componente de Cliente inalterado.

<Pitfall>

**Não confie apenas na contaminação para segurança.** Contaminar um objeto não impede o vazamento de todos os valores derivados possíveis. Por exemplo, o clone de um objeto contaminado criará um novo objeto não contaminado. Usar dados de um objeto contaminado (por exemplo, `{secret: taintedObj.secret}`) criará um novo valor ou objeto que não está contaminado. A contaminação é uma camada de proteção; um app seguro terá múltiplas camadas de proteção, APIs bem projetadas e padrões de isolamento.

</Pitfall>

---

## Uso {/*usage*/}

### Impedir que dados do usuário cheguem ao cliente intencionalmente {/*prevent-user-data-from-unintentionally-reaching-the-client*/}

Um Componente de Cliente nunca deve aceitar objetos que carregam dados sensíveis. Idealmente, as funções de busca de dados não devem expor dados aos quais o usuário atual não deveria ter acesso. Às vezes, erros acontecem durante a refatoração. Para proteger contra esses erros acontecendo no futuro, podemos "contaminar" o objeto do usuário em nossa API de dados.

```js
import {experimental_taintObjectReference} from 'react';

export async function getUser(id) {
  const user = await db`SELECT * FROM users WHERE id = ${id}`;
  experimental_taintObjectReference(
    'Não passe o objeto inteiro do usuário para o cliente. ' +
      'Em vez disso, selecione as propriedades específicas necessárias para este caso de uso.',
    user,
  );
  return user;
}
```

Agora, sempre que alguém tentar passar este objeto para um Componente de Cliente, um erro será lançado com a mensagem de erro passada no lugar.

<DeepDive>

#### Proteger contra vazamentos na busca de dados {/*protecting-against-leaks-in-data-fetching*/}

Se você estiver executando um ambiente de Componentes de Servidor que tem acesso a dados sensíveis, você deve ter cuidado para não passar objetos diretamente:

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

Idealmente, a `getUser` não deve expor dados aos quais o usuário atual não deveria ter acesso. Para evitar a passagem do objeto `user` para um Componente de Cliente no futuro, podemos "contaminar" o objeto do usuário:

```js
// api.js
import {experimental_taintObjectReference} from 'react';

export async function getUser(id) {
  const user = await db`SELECT * FROM users WHERE id = ${id}`;
  experimental_taintObjectReference(
    'Não passe o objeto inteiro do usuário para o cliente. ' +
      'Em vez disso, selecione as propriedades específicas necessárias para este caso de uso.',
    user,
  );
  return user;
}
```

Agora, se alguém tentar passar o objeto `user` para um Componente de Cliente, um erro será lançado com a mensagem de erro passada.

</DeepDive>
```