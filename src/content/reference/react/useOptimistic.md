---
title: useOptimistic
---

<Intro>

O `useOptimistic` é um React Hook que permite atualizar otimistamente a UI.

```js
  const [optimisticState, addOptimistic] = useOptimistic(state, updateFn);
```

</Intro>

<InlineToc />

---

## Referência {/*reference*/}

### `useOptimistic(state, updateFn)` {/*use*/}

`useOptimistic` é um React Hook que permite mostrar um estado diferente enquanto uma ação assíncrona está em andamento. Ele aceita algum estado como argumento e retorna uma cópia desse estado que pode ser diferente durante a duração de uma ação assíncrona, como uma requisição de rede. Você fornece uma função que recebe o estado atual e a entrada para a ação e retorna o estado otimista a ser usado enquanto a ação estiver pendente.

Este estado é chamado de estado "otimista" porque geralmente é usado para apresentar imediatamente ao usuário o resultado da execução de uma ação, mesmo que a ação realmente demore para ser concluída.

```js
import { useOptimistic } from 'react';

function AppContainer() {
  const [optimisticState, addOptimistic] = useOptimistic(
    state,
    // updateFn
    (currentState, optimisticValue) => {
      // merge and return new state
      // with optimistic value
    }
  );
}
```

[Veja mais exemplos abaixo.](#usage)

#### Parâmetros {/*parameters*/}

*   `state`: o valor a ser retornado inicialmente e sempre que nenhuma ação estiver pendente.
*   `updateFn(currentState, optimisticValue)`: uma função que recebe o estado atual e o valor otimista passado para `addOtimistic` e retorna o estado otimista resultante. Deve ser uma função pura. `updateFn` recebe dois parâmetros: o `currentState` e o `optimisticValue`. O valor de retorno será o valor mesclado de `currentState` e `optimisticValue`.

#### Retorna {/*returns*/}

*   `optimisticState`: O estado otimista resultante. É igual a `state`, a menos que uma ação esteja pendente, caso em que é igual ao valor retornado por `updateFn`.
*   `addOptimistic`: `addOptimistic` é a função de dispatch a ser chamada quando você tiver uma atualização otimista. Ele recebe um argumento, `optimisticValue`, de qualquer tipo e chamará a `updateFn` com `state` e `optimisticValue`.

---

## Uso {/*usage*/}

### Atualizando formulários otimistamente {/*optimistically-updating-with-forms*/}

O Hook `useOptimistic` fornece uma maneira de atualizar otimistamente a interface do usuário antes que uma operação em segundo plano, como uma requisição de rede, seja concluída. No contexto de formulários, essa técnica ajuda a fazer com que os apps pareçam mais responsivos. Quando um usuário envia um formulário, em vez de esperar pela resposta do servidor para refletir as alterações, a interface é imediatamente atualizada com o resultado esperado.

Por exemplo, quando um usuário digita uma mensagem no formulário e clica no botão "Enviar", o Hook `useOptimistic` permite que a mensagem apareça imediatamente na lista com um rótulo "Enviando...", mesmo antes que a mensagem seja realmente enviada para um servidor. Essa abordagem "otimista" dá a impressão de velocidade e responsividade. O formulário tenta, então, realmente enviar a mensagem em segundo plano. Assim que o servidor confirmar que a mensagem foi recebida, o rótulo "Enviando..." é removido.

<Sandpack>

```js src/App.js
import { useOptimistic, useState, useRef } from "react";
import { deliverMessage } from "./actions.js";

function Thread({ messages, sendMessage }) {
  const formRef = useRef();
  async function formAction(formData) {
    addOptimisticMessage(formData.get("message"));
    formRef.current.reset();
    await sendMessage(formData);
  }
  const [optimisticMessages, addOptimisticMessage] = useOptimistic(
    messages,
    (state, newMessage) => [
      ...state,
      {
        text: newMessage,
        sending: true
      }
    ]
  );

  return (
    <>
      {optimisticMessages.map((message, index) => (
        <div key={index}>
          {message.text}
          {!!message.sending && <small> (Enviando...)</small>}
        </div>
      ))}
      <form action={formAction} ref={formRef}>
        <input type="text" name="message" placeholder="Hello!" />
        <button type="submit">Send</button>
      </form>
    </>
  );
}

export default function App() {
  const [messages, setMessages] = useState([
    { text: "Hello there!", sending: false, key: 1 }
  ]);
  async function sendMessage(formData) {
    const sentMessage = await deliverMessage(formData.get("message"));
    setMessages((messages) => [...messages, { text: sentMessage }]);
  }
  return <Thread messages={messages} sendMessage={sendMessage} />;
}
```

```js src/actions.js
export async function deliverMessage(message) {
  await new Promise((res) => setTimeout(res, 1000));
  return message;
}
```

</Sandpack>