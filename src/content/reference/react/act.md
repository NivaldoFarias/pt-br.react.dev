---
title: act
---

<Intro>

`act` é uma função auxiliar de teste para aplicar atualizações React pendentes antes de fazer asserções.

```js
await act(async actFn)
```

</Intro>

Para preparar um componente para asserções, envolva o código que o renderiza e executa as atualizações dentro de uma chamada `await act()`. Isso faz com que seu teste seja executado de forma mais próxima de como o React funciona no navegador.

<Note>
Você pode achar o uso do `act()` diretamente um pouco verboso demais. Para evitar alguma burocracia, você pode usar uma biblioteca como [React Testing Library](https://testing-library.com/docs/react-testing-library/intro), cujas funções auxiliares são encapsuladas com `act()`.
</Note>


<InlineToc />

---

## Referência {/*reference*/}

### `await act(async actFn)` {/*await-act-async-actfn*/}

Ao escrever testes de UI, tarefas como renderização, eventos de usuário ou busca de dados podem ser consideradas como "unidades" de interação com uma interface de usuário. O React fornece uma função auxiliar chamada `act()` que garante que todas as atualizações relacionadas a essas "unidades" tenham sido processadas e aplicadas ao DOM antes de você fazer qualquer asserção.

O nome `act` vem do padrão [Arrange-Act-Assert](https://wiki.c2.com/?ArrangeActAssert).

```js {2,4}
it ('renders with button disabled', async () => {
  await act(async () => {
    root.render(<TestComponent />)
  });
  expect(container.querySelector('button')).toBeDisabled();
});
```

<Note>

Recomendamos usar `act` com `await` e uma função `async`. Embora a versão síncrona funcione em muitos casos, ela não funciona em todos os casos e, devido à forma como o React agenda as atualizações internamente, é difícil prever quando você pode usar a versão síncrona.

Vamos descontinuar e remover a versão síncrona no futuro.

</Note>

#### Parâmetros {/*parameters*/}

*   `async actFn`: Uma função assíncrona envolvendo renderizações ou interações para componentes sendo testados. Quaisquer atualizações acionadas dentro da `actFn` são adicionadas a uma fila interna de ação, que são, então, esvaziadas em conjunto para processar e aplicar quaisquer alterações no DOM. Como é assíncrona, o React também executará qualquer código que atravesse um limite assíncrono e esvaziará quaisquer atualizações agendadas.

#### Retorna {/*returns*/}

`act` não retorna nada.

## Uso {/*usage*/}

Ao testar um componente, você pode usar `act` para fazer asserções sobre sua saída.

Por exemplo, digamos que temos este componente `Counter`, os exemplos de uso abaixo mostram como testá-lo:

```js
function Counter() {
  const [count, setCount] = useState(0);
  const handleClick = () => {
    setCount(prev => prev + 1);
  }

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  }, [count]);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={handleClick}>
        Click me
      </button>
    </div>
  )
}
```

### Renderizando componentes em testes {/*rendering-components-in-tests*/}

Para testar a saída de renderização de um componente, envolva a renderização dentro de `act()`:

```js  {10,12}
import {act} from 'react';
import ReactDOMClient from 'react-dom/client';
import Counter from './Counter';

it('can render and update a counter', async () => {
  container = document.createElement('div');
  document.body.appendChild(container);
  
  // ✅ Render the component inside act().
  await act(() => {
    ReactDOMClient.createRoot(container).render(<Counter />);
  });
  
  const button = container.querySelector('button');
  const label = container.querySelector('p');
  expect(label.textContent).toBe('You clicked 0 times');
  expect(document.title).toBe('You clicked 0 times');
});
```

Aqui, criamos um contêiner, anexamos a ele ao documento e renderizamos o componente `Counter` dentro de `act()`. Isso garante que o componente seja renderizado e seus efeitos sejam aplicados antes de fazer as asserções.

Usar `act` garante que todas as atualizações foram aplicadas antes de fazermos as asserções.

### Despachando eventos em testes {/*dispatching-events-in-tests*/}

Para testar eventos, envolva o envio de eventos dentro de `act()`:

```js {14,16}
import {act} from 'react';
import ReactDOMClient from 'react-dom/client';
import Counter from './Counter';

it.only('can render and update a counter', async () => {
  const container = document.createElement('div');
  document.body.appendChild(container);
  
  await act( async () => {
    ReactDOMClient.createRoot(container).render(<Counter />);
  });
  
  // ✅ Dispatch the event inside act().
  await act(async () => {
    button.dispatchEvent(new MouseEvent('click', { bubbles: true }));
  });

  const button = container.querySelector('button');
  const label = container.querySelector('p');
  expect(label.textContent).toBe('You clicked 1 times');
  expect(document.title).toBe('You clicked 1 times');
});
```

Aqui, renderizamos o componente com `act` e, em seguida, despachamos o evento dentro de outro `act()`. Isso garante que todas as atualizações do evento sejam aplicadas antes de fazer as asserções.

<Pitfall>

Não se esqueça que despachar eventos DOM só funciona quando o contêiner DOM é adicionado ao documento. Você pode usar uma biblioteca como [React Testing Library](https://testing-library.com/docs/react-testing-library/intro) para reduzir o código boilerplate.

</Pitfall>

## Solução de problemas {/*troubleshooting*/}

### Estou recebendo um erro: "O ambiente de teste atual não está configurado para suportar act"(...) " {/*error-the-current-testing-environment-is-not-configured-to-support-act*/}

Usar `act` requer definir `global.IS_REACT_ACT_ENVIRONMENT=true` no seu ambiente de teste. Isso é para garantir que `act` seja usado somente no ambiente correto.

Se você não definir o global, você verá um erro como este:

<ConsoleBlock level="error">

Warning: The current testing environment is not configured to support act(...)

</ConsoleBlock>

Para corrigir, adicione isso ao seu arquivo de configuração global para testes React:

```js
global.IS_REACT_ACT_ENVIRONMENT=true
```

<Note>

Em frameworks de teste como [React Testing Library](https://testing-library.com/docs/react-testing-library/intro), `IS_REACT_ACT_ENVIRONMENT` já está definido para você.

</Note>