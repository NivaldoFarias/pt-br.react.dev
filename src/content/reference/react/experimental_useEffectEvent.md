---
title: experimental_useEffectEvent
---

<Wip>

**Esta API é experimental e ainda não está disponível em uma versão estável do React.**

Você pode testá-la atualizando os pacotes do React para a versão experimental mais recente:

- `react@experimental`
- `react-dom@experimental`
- `eslint-plugin-react-hooks@experimental`

As versões experimentais do React podem conter erros. Não as use na produção.

</Wip>


<Intro>

`useEffectEvent` é um React Hook que permite extrair a lógica não reativa em um [Evento de Efeito.](/learn/separating-events-from-effects#declaring-an-effect-event)

```js
const onSomething = useEffectEvent(callback)
```

</Intro>

<InlineToc />
```