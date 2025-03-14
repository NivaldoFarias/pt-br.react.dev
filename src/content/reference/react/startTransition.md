---
title: startTransition
---

<Intro>

`startTransition` te permite renderizar uma parte da UI em background.

```js
startTransition(action)
```

</Intro>

<InlineToc />

---

## Referência {/*reference*/}

### `startTransition(action)` {/*starttransition*/}

A função `startTransition` permite que você marque uma atualização de estado como uma Transição.

```js {7,9}
import { startTransition } from 'react';

function TabContainer() {
  const [tab, setTab] = useState('about');

  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);
    });
  }
  // ...
}
```

[Veja mais exemplos abaixo.](#usage)

#### Parâmetros {/*parameters*/}

* `action`: Uma função que atualiza algum estado chamando uma ou mais funções [`set`](/reference/react/useState#setstate). React chama `action` imediatamente sem parâmetros e marca todas as atualizações de estado agendadas de forma síncrona durante a chamada da função `action` como Transições. Quaisquer chamadas assíncronas aguardadas no `action` serão incluídas na transição, mas atualmente requerem encapsular quaisquer funções `set` após o `await` em um `startTransition` adicional (veja [Solução de problemas](/reference/react/useTransition#react-doesnt-treat-my-state-update-after-await-as-a-transition)). Atualizações de estado marcadas como Transições serão [não bloqueantes](#marking-a-state-update-as-a-non-blocking-transition) e [não exibirão indicadores de carregamento indesejados.](/reference/react/useTransition#preventing-unwanted-loading-indicators).

#### Retorna {/*returns*/}

`startTransition` não retorna nada.

#### Ressalvas {/*caveats*/}

* `startTransition` não fornece uma maneira de rastrear se uma Transição está pendente. Para mostrar um indicador pendente enquanto a Transição está em andamento, você precisa de [`useTransition`](/reference/react/useTransition) em vez disso.

* Você pode encapsular uma atualização em uma Transição somente se tiver acesso à função `set` desse estado. Se você deseja iniciar uma Transição em resposta a alguma prop ou um valor de retorno de um Hook personalizado, tente [`useDeferredValue`](/reference/react/useDeferredValue) em vez disso.

* A função que você passa para `startTransition` é chamada imediatamente, marcando todas as atualizações de estado que ocorrem enquanto ela é executada como Transições. Se você tentar executar atualizações de estado em um `setTimeout`, por exemplo, elas não serão marcadas como Transições.

* Você deve encapsular quaisquer atualizações de estado após quaisquer requisições assíncronas em outro `startTransition` para marcá-las como Transições. Esta é uma limitação conhecida que corrigiremos no futuro (veja [Solução de problemas](/reference/react/useTransition#react-doesnt-treat-my-state-update-after-await-as-a-transition)).

* Uma atualização de estado marcada como uma Transição será interrompida por outras atualizações de estado. Por exemplo, se você atualizar um componente de gráfico dentro de uma Transição, mas, em seguida, começar a digitar em uma entrada enquanto o gráfico estiver no meio da re-renderização, React reiniciará o trabalho de renderização no componente de gráfico após manipular a atualização de estado da entrada.

* Atualizações de transição não podem ser usadas para controlar entradas de texto.

* Se houver várias Transições em andamento, React atualmente as agrupa. Esta é uma limitação que pode ser removida em uma versão futura.

---

## Uso {/*usage*/}

### Marcando uma atualização de estado como uma Transição não bloqueante {/*marking-a-state-update-as-a-non-blocking-transition*/}

Você pode marcar uma atualização de estado como uma *Transição* encapsulando-a em uma chamada `startTransition`:

```js {7,9}
import { startTransition } from 'react';

function TabContainer() {
  const [tab, setTab] = useState('about');

  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);
    });
  }
  // ...
}
```

Transições permitem manter as atualizações da interface do usuário responsivas, mesmo em dispositivos lentos.

Com uma Transição, sua UI permanece responsiva no meio de uma re-renderização. Por exemplo, se o usuário clicar em uma aba, mas, em seguida, mudar de ideia e clicar em outra aba, ele pode fazer isso sem esperar que a primeira re-renderização termine.

<Note>

`startTransition` é muito semelhante a [`useTransition`](/reference/react/useTransition), exceto que ele não fornece a flag `isPending` para rastrear se uma Transição está em andamento. Você pode chamar `startTransition` quando `useTransition` não estiver disponível. Por exemplo, `startTransition` funciona fora de componentes, como de uma biblioteca de dados.

[Aprenda sobre Transições e veja exemplos na página `useTransition`.](/reference/react/useTransition)

</Note>