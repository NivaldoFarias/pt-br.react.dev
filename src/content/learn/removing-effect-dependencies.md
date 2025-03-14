js
// Guia de Estilo Universal

Este documento descreve as regras que devem ser aplicadas para **todos** os idiomas.
Quando estiver se referindo ao próprio `React`, use `o React`.

## IDs dos Títulos

Todos os títulos possuem IDs explícitos como abaixo:

```md
## Tente React {#try-react}
```

**Não** traduza estes IDs! Eles são usado para navegação e quebrarão se o documento for um link externo, como:

```md
Veja a [seção iniciando](/getting-started#try-react) para mais informações.
```

✅ FAÇA:

```md
## Tente React {#try-react}
```

❌ NÃO FAÇA:

```md
## Tente React {#tente-react}
```

Isto quebraria o link acima.

## Texto em Blocos de Código

Mantenha o texto em blocos de código sem tradução, exceto para os comentários. Você pode optar por traduzir o texto em strings, mas tenha cuidado para não traduzir strings que se refiram ao código!

Exemplo:

```js
// Example
const element = <h1>Hello, world</h1>;
ReactDOM.render(element, document.getElementById('root'));
```

✅ FAÇA:

```js
// Exemplo
const element = <h1>Hello, world</h1>;
ReactDOM.render(element, document.getElementById('root'));
```

✅ PERMITIDO:

```js
// Exemplo
const element = <h1>Olá mundo</h1>;
ReactDOM.render(element, document.getElementById('root'));
```

❌ NÃO FAÇA:

```js
// Exemplo
const element = <h1>Olá mundo</h1>;
// "root" se refere a um ID de elemento.
// NÃO TRADUZA
ReactDOM.render(element, document.getElementById('raiz'));
```

❌ DEFINITIVAMENTE NÃO FAÇA:

```js
// Exemplo
const elemento = <h1>Olá mundo</h1>;
ReactDOM.renderizar(elemento, documento.obterElementoPorId('raiz'));
```

## Links Externos

Se um link externo se referir a um artigo no [MDN] or [Wikipedia] e se houver uma versão traduzida em seu idioma em uma qualidade decente, opte por usar a versão traduzida.

[mdn]: https://developer.mozilla.org/pt-BR/
[wikipedia]: https://pt.wikipedia.org/wiki/Wikipédia:Página_principal

Exemplo:

```md
React elements are [immutable](https://en.wikipedia.org/wiki/Immutable_object).
```

✅ OK:

```md
Elementos React são [imutáveis](https://pt.wikipedia.org/wiki/Objeto_imutável).
```

Para links que não possuem tradução (Stack Overflow, vídeos do YouTube, etc.), simplesmente use o link original.

## Traduções Comuns

Sugestões de palavras e termos:

| Palavra/Termo original | Sugestão                               |
| ---------------------- | -------------------------------------- |
| assertion              | asserção                               |
| at the top level       | na raiz                                |
| browser                | navegador                              |
| bubbling               | propagar                               |
| bug                    | erro                                   |
| caveats                | ressalvas                              |
| class component        | componente de classe                   |
| class                  | classe                                 |
| client                 | cliente                                |
| client-side            | lado do cliente                        |
| container              | contêiner                              |
| context                | contexto                               |
| controlled component   | componente controlado                  |
| debugging              | depuração                              |
| DOM node               | nó do DOM                              |
| event handler          | manipulador de eventos (event handler) |
| function component     | componente de função                   |
| handler                | manipulador                            |
| helper function        | função auxiliar                        |
| high-order components  | componente de alta-ordem               |
| key                    | chave                                  |
| library                | biblioteca                             |
| lowercase              | minúscula(s) / caixa baixa             |
| package                | pacote                                 |
| React element          | Elemento React                         |
| React fragment         | Fragmento React                        |
| render                 | renderizar (verb), renderizado (noun)  |
| server                 | servidor                               |
| server-side            | lado do servidor                       |
| siblings               | irmãos                                 |
| stateful component     | componente com estado                  |
| stateful logic         | lógica com estado                      |
| to assert              | afirmar                                |
| to wrap                | encapsular                             |
| troubleshooting        | solução de problemas                   |
| uncontrolled component | componente não controlado              |
| uppercase              | maiúscula(s) / caixa alta              |

## Conteúdo que não deve ser traduzido

- array
- arrow function
- bind
- bundle
- bundler
- callback
- camelCase
- DOM
- event listener
- framework
- hook
- log
- mock
- portal
- props
- ref
- release
- script
- single-page-apps
- state
- string
- string literal
- subscribe
- subscription
- template literal
- timestamps
- UI
- watcher
- widgets
- wrapper
```

```
PART 2 OF 2:

---
title: 'Removendo Dependências de Effect'
---

<Intro>

Ao escrever um Effect, o linter verificará se você incluiu cada valor reativo (como props e state) que o Effect lê na lista de dependências do seu Effect. Isso garante que seu Effect permaneça sincronizado com as props e o state mais recentes do seu componente. Dependências desnecessárias podem fazer com que seu Effect seja executado com muita frequência ou até mesmo criar um loop infinito. Siga este guia para revisar e remover as dependências desnecessárias de seus Effects.

</Intro>

<YouWillLearn>

- Como corrigir loops infinitos de dependência de Effect
- O que fazer quando você quiser remover uma dependência
- Como ler um valor do seu Effect sem "reagir" a ele
- Como e por que evitar dependências de objeto e função
- Por que suprimir o linter de dependência é perigoso e o que fazer em vez disso

</YouWillLearn>

## As dependências devem corresponder ao código {/*dependencies-should-match-the-code*/}

Ao escrever um Effect, você primeiro especifica como [iniciar e parar](/learn/lifecycle-of-reactive-effects#the-lifecycle-of-an-effect) o que quer que seu Effect esteja fazendo:

```js {5-7}
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  	// ...
}
```

Então, se você deixar as dependências do Effect vazias (`[]`), o linter sugerirá as dependências corretas:

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // <-- Corrija o erro aqui!
  return <h1>Bem-vindo(a) à sala {roomId}!</h1>;
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  return (
    <>
      <label>
        Escolha a sala de bate-papo:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <hr />
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

```js src/chat.js
export function createConnection(serverUrl, roomId) {
  // Uma implementação real realmente se conectaria ao servidor
  return {
    connect() {
      console.log('✅ Conectando-se à sala "' + roomId + '" em ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Desconectado da sala "' + roomId + '" em ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

Preencha-as de acordo com o que o linter diz:

```js {6}
function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ Todas as dependências declaradas
  // ...
}
```

[Effects "reagem" aos valores reativos.](/learn/lifecycle-of-reactive-effects#effects-react-to-reactive-values) Como `roomId` é um valor reativo (ele pode mudar devido a uma nova renderização), o linter verifica se você o especificou como uma dependência. Se `roomId` receber um valor diferente, o React vai re-sincronizar seu Effect. Isso garante que o bate-papo permaneça conectado à sala selecionada e "reaja" ao dropdown:

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);
  return <h1>Bem-vindo(a) à sala {roomId}!</h1>;
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  return (
    <>
      <label>
        Escolha a sala de bate-papo:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <hr />
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

```js src/chat.js
export function createConnection(serverUrl, roomId) {
  // Uma implementação real realmente se conectaria ao servidor
  return {
    connect() {
      console.log('✅ Conectando-se à sala "' + roomId + '" em ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Desconectado da sala "' + roomId + '" em ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

### Para remover uma dependência, prove que ela não é uma dependência {/*to-remove-a-dependency-prove-that-its-not-a-dependency*/}

Observe que você não pode "escolher" as dependências do seu Effect. Cada <CodeStep step={2}>valor reativo</CodeStep> usado pelo código do seu Effect deve ser declarado em sua lista de dependências. A lista de dependências é determinada pelo código circundante:

```js [[2, 3, "roomId"], [2, 5, "roomId"], [2, 8, "roomId"]]
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) { // Este é um valor reativo
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Este Effect lê esse valor reativo
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ Então você deve especificar esse valor reativo como uma dependência do seu Effect
  // ...
}
```

[Valores reativos](/learn/lifecycle-of-reactive-effects#all-variables-declared-in-the-component-body-are-reactive) incluem props e todas as variáveis e funções declaradas diretamente dentro do seu componente. Como `roomId` é um valor reativo, você não pode removê-lo da lista de dependências. O linter não permitiria:

```js {8}
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // 🔴 React Hook useEffect has a missing dependency: 'roomId'
  // ...
}
```

E o linter estaria certo! Como `roomId` pode mudar com o tempo, isso introduziria um bug em seu código.

**Para remover uma dependência, "prove" ao linter que *não é necessário* que ela seja uma dependência.** Por exemplo, você pode mover `roomId` para fora do seu componente para provar que ele não é reativo a não mudará nas novas renderizações:

```js {2,9}
const serverUrl = 'https://localhost:1234';
const roomId = 'music'; // Não é mais um valor reativo

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // ✅ Todas as dependências declaradas
  // ...
}
```

Agora que `roomId` não é um valor reativo (e não pode mudar em uma nova renderização), ele não precisa ser uma dependência:

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';
const roomId = 'music';

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []);
  return <h1>Bem-vindo(a) à sala {roomId}!</h1>;
}
```

```js src/chat.js
export function createConnection(serverUrl, roomId) {
  // Uma implementação real realmente se conectaria ao servidor
  return {
    connect() {
      console.log('✅ Conectando-se à sala "' + roomId + '" em ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Desconectado da sala "' + roomId + '" em ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

É por isso que agora você pode especificar uma lista de dependências [vazia (`[]`).](/learn/lifecycle-of-reactive-effects#what-an-effect-with-empty-dependencies-means) Seu Effect *realmente não* depende mais de nenhum valor reativo, então *realmente não* precisa ser executado novamente quando qualquer uma das props ou state do componente mudarem.

### Para mudar as dependências, mude o código {/*to-change-the-dependencies-change-the-code*/}

Você pode ter notado um padrão no seu fluxo de trabalho:

1. Primeiro, você **muda o código** do seu Effect ou como seus valores reativos são declarados.
2. Em seguida, você segue o linter e ajusta as dependências para **combinar com o código que você mudou.**
3. Se você não estiver satisfeito com a lista de dependências, você **volta para a primeira etapa** (e muda o código novamente).

A última parte é importante. **Se você quiser mudar as dependências, mude o código circundante primeiro.** Você pode pensar na lista de dependências como [uma lista de todos os valores reativos usados pelo código do seu Effect.](/learn/lifecycle-of-reactive-effects#react-verifies-that-you-specified-every-reactive-value-as-a-dependency) Você não *escolhe* o que colocar nessa lista. A lista *descreve* seu código. Para mudar a lista de dependências, mude o código.

Isso pode parecer resolver uma equação. Você pode começar com um objetivo (por exemplo, remover uma dependência) e precisa "encontrar" o código correspondente a esse objetivo. Nem todo mundo acha resolver equações divertido, e o mesmo pode ser dito sobre escrever Effects! Felizmente, existe uma lista de receitas comuns que você pode experimentar abaixo.

<Pitfall>

Se você tiver uma base de código existente, pode ter alguns Effects que suprimem o linter assim:

```js {3-4}
useEffect(() => {
  // ...
  // 🔴 Evite suprimir o linter assim:
  // eslint-ignore-next-line react-hooks/exhaustive-deps
}, []);
```

**Quando as dependências não correspondem ao código, há um risco muito alto de introduzir erros.** Ao suprimir o linter, você "mente" para o React sobre os valores dos quais seu Effect depende.

Em vez disso, use as técnicas abaixo.

</Pitfall>

<DeepDive>

#### Por que suprimir o linter de dependência é tão perigoso? {/*why-is-suppressing-the-dependency-linter-so-dangerous*/}

Suprimir o linter leva a erros muito não intuitivos que são difíceis de encontrar e corrigir. Aqui está um exemplo:

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function Timer() {
  const [count, setCount] = useState(0);
  const [increment, setIncrement] = useState(1);

  function onTick() {
	setCount(count + increment);
  }

  useEffect(() => {
    const id = setInterval(onTick, 1000);
    return () => clearInterval(id);
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, []);

  return (
    <>
      <h1>
        Contador: {count}
        <button onClick={() => setCount(0)}>Reset</button>
      </h1>
      <hr />
      <p>
        A cada segundo, incrementar por:
        <button disabled={increment === 0} onClick={() => {
          setIncrement(i => i - 1);
        }}>–</button>
        <b>{increment}</b>
        <button onClick={() => {
          setIncrement(i => i + 1);
        }}>+</button>
      </p>
    </>
  );
}
```

```css
button { margin: 10px; }
```

</Sandpack>

Digamos que você quisesse executar o Effect "somente na montagem". Você leu que [as dependências vazias (`[]`)](/learn/lifecycle-of-reactive-effects#what-an-effect-with-empty-dependencies-means) fazem isso, então você decidiu ignorar o linter e especificou à força `[]` como dependências.

Este contador deveria ser incrementado a cada segundo pela quantidade configurável com os dois botões. No entanto, como você "mentiu" para o React que este Effect não depende de nada, o React continua usando a função `onTick` da renderização inicial para sempre. [Durante essa renderização,](/learn/state-as-a-snapshot#rendering-takes-a-snapshot-in-time) `count` era `0` e `increment` era `1`. É por isso que `onTick` daquela renderização sempre chama `setCount(0 + 1)` a cada segundo, e você sempre vê `1`. Erros como esse são mais difíceis de corrigir quando se espalham por vários componentes.

Sempre há uma solução melhor do que ignorar o linter! Para corrigir este código, você precisa adicionar `onTick` à lista de dependências. (Para garantir que o intervalo seja configurado apenas uma vez, [faça de `onTick` um Effect Event.](/learn/separating-events-from-effects#reading-latest-props-and-state-with-effect-events))

**Recomendamos tratar o erro lint de dependência como um erro de compilação. Se você não o suprimir, nunca verá erros como este.** O restante desta página documenta as alternativas para este e outros casos.

</DeepDive>

## Removendo dependências desnecessárias {/*removing-unnecessary-dependencies*/}

Toda vez que você ajustar as dependências do Effect para refletir o código, olhe para a lista de dependências. Faz sentido para o Effect ser executado novamente quando alguma dessas dependências mudar? Às vezes, a resposta é "não":

* Você pode querer re-executar *diferentes partes* do seu Effect sob diferentes condições.
* Você pode querer apenas ler o *último valor* de alguma dependência em vez de "reagir" às suas mudanças.
* Uma dependência pode mudar com muita frequência *intencionalmente* porque é um objeto ou uma função.

Para encontrar a solução certa, você precisará responder a algumas perguntas sobre o seu Effect. Vamos analisá-las.

### Este código deve ser movido para um manipulador de eventos? {/*should-this-code-move-to-an-event-handler*/}

A primeira coisa que você deve pensar é se este código deve ser um Effect.

Imagine um formulário. Ao enviar, você define a variável de state `submitted` como `true`. Você precisa enviar uma solicitação POST e mostrar uma notificação. Você colocou essa lógica dentro de um Effect que "reage" ao `submitted` ser `true`:

```js {6-8}
function Form() {
  const [submitted, setSubmitted] = useState(false);

  useEffect(() => {
    if (submitted) {
      // 🔴 Evitar: lógica específica do evento dentro de um Effect
      post('/api/register');
      showNotification('Registrado com sucesso!');
    }
  }, [submitted]);

  function handleSubmit() {
    setSubmitted(true);
  }

  // ...
}
```

Mais tarde, você deseja estilizar a mensagem de notificação de acordo com o tema atual, então lê o tema atual. Como `theme` é declarado no corpo do componente, é um valor reativo, então você o adiciona como uma dependência:

```js {3,9,11}
function Form() {
  const [submitted, setSubmitted] = useState(false);
  const theme = useContext(ThemeContext);

  useEffect(() => {
    if (submitted) {
      // 🔴 Evitar: lógica específica do evento dentro de um Effect
      post('/api/register');
      showNotification('Registrado com sucesso!', theme);
    }
  }, [submitted, theme]); // ✅ Todas as dependências declaradas

  function handleSubmit() {
    setSubmitted(true);
  }  

  // ...
}
```

Ao fazer isso, você introduziu um bug. Imagine que você envia o formulário primeiro e depois alterna entre temas Escuro e Claro. O `theme` vai mudar, o Effect vai ser executado novamente e, portanto, ele exibirá a mesma notificação novamente!

**O problema aqui é que isso não deveria ser um Effect em primeiro lugar.** Você deseja enviar essa solicitação POST e mostrar a notificação em resposta ao *envio do formulário*, que é uma interação específica. Para executar algum código em resposta a uma interação em particular, coloque essa lógica diretamente no manipulador de eventos correspondente:

```js {6-7}
function Form() {
  const theme = useContext(ThemeContext);

  function handleSubmit() {
    // ✅ Bom: a lógica específica do evento é chamada de manipuladores de eventos
    post('/api/register');
    showNotification('Registrado com sucesso!', theme);
  }  

  // ...
}
```

Agora que o código está em um manipulador de eventos, ele não é reativo, portanto, será executado apenas quando o usuário enviar o formulário. Leia mais sobre [escolhendo entre manipuladores de eventos e Effects](/learn/separating-events-from-effects#reactive-values-and-reactive-logic) e [como excluir Effects desnecessários.](/learn/you-might-not-need-an-effect)

### Seu Effect está fazendo várias coisas não relacionadas? {/*is-your-effect-doing-several-unrelated-things*/}

A próxima pergunta que você deve se fazer é se seu Effect está fazendo várias coisas não relacionadas.

Imagine que você está criando um formulário de envio em que o usuário precisa escolher sua cidade e área. Você busca a lista de `cities` do servidor de acordo com o `country` selecionado para mostrá-los em um dropdown:

```js
function ShippingForm({ country }) {
  const [cities, setCities] = useState(null);
  const [city, setCity] = useState(null);

  useEffect(() => {
    let ignore = false;
    fetch(`/api/cities?country=${country}`)
      .then(response => response.json())
      .then(json => {
        if (!ignore) {
          setCities(json);
        }
      });
    return () => {
      ignore = true;
    };
  }, [country]); // ✅ Todas as dependências declaradas

  // ...
```

Este é um bom exemplo de [busca de dados em um Effect.](/learn/you-might-not-need-an-effect#fetching-data) Você está sincronizando o estado `cities` com a rede de acordo com o `country` prop. Você não pode fazer isso em um manipulador de eventos porque precisa buscar assim que `ShippingForm` for exibido e sempre que o `country` mudar (independentemente de qual interação causa a sua alteração).

Agora, digamos que você esteja adicionando uma segunda caixa de seleção para as áreas da cidade, que deve buscar as `areas` para a `city` selecionada no momento. Você pode começar adicionando uma segunda chamada `fetch` para a lista de áreas dentro do mesmo Effect:

```js {15-24,28}
function ShippingForm({ country }) {
  const [cities, setCities] = useState(null);
  const [city, setCity] = useState(null);
  const [areas, setAreas] = useState(null);

  useEffect(() => {
    let ignore = false;
    fetch(`/api/cities?country=${country}`)
      .then(response => response.json())
      .then(json => {
        if (!ignore) {
          setCities(json);
        }
      });
    // 🔴 Evitar: um único Effect sincroniza dois processos independentes
    if (city) {
      fetch(`/api/areas?city=${city}`)
        .then(response => response.json())
        .then(json => {
          if (!ignore) {
            setAreas(json);
          }
        });
    }
    return () => {
      ignore = true;
    };
  }, [country, city]); // ✅ Todas as dependências declaradas

  // ...
```

No entanto, como o Effect agora usa a variável de estado `city`, você teve que adicionar `city` à lista de dependências. Isso, por sua vez, introduziu um problema: quando o usuário seleciona uma cidade diferente, o Effect será executado novamente e chamará `fetchCities(country)`. Como resultado, você estará refazendo, desnecessariamente, a lista de cidades muitas vezes.

**O problema com este código é que você está sincronizando duas coisas diferentes não relacionadas:**

1. Você deseja sincronizar o estado `cities` com a rede com base no prop `country`.
1. Você deseja sincronizar o estado `areas` com a rede com base no estado `city`.

Divida a lógica em dois Effects, cada um dos quais reage ao prop com o qual precisa sincronizar:

```js {19-33}
function ShippingForm({ country }) {
  const [cities, setCities] = useState(null);
  useEffect(() => {
    let ignore = false;
    fetch(`/api/cities?country=${country}`)
      .then(response => response.json())
      .then(json => {
        if (!ignore) {
          setCities(json);
        }
      });
    return () => {
      ignore = true;
    };
  }, [country]); // ✅ Todas as dependências declaradas
``````js
  const [city, setCity] = useState(null);
  const [areas, setAreas] = useState(null);
  useEffect(() => {
    if (city) {
      let ignore = false;
      fetch(`/api/areas?city=${city}`)
        .then(response => response.json())
        .then(json => {
          if (!ignore) {
            setAreas(json);
          }
        });
      return () => {
        ignore = true;
      };
    }
  }, [city]); // ✅ Todas as dependências declaradas

  // ...
```

Agora, o primeiro Effect só é executado novamente se o `country` mudar, enquanto o segundo Effect é executado novamente quando o `city` muda. Você os separou por propósito: duas coisas diferentes são sincronizadas por dois Effects separados. Dois Effects separados têm duas listas de dependências separadas, então não acionarão um ao outro intencionalmente.

O código final é mais longo que o original, mas dividir esses Effects ainda é correto. [Cada Effect deve representar um processo de sincronização independente.](/learn/lifecycle-of-reactive-effects#each-effect-represents-a-separate-synchronization-process) Neste exemplo, excluir um Effect não quebra a lógica do outro Effect. Isso significa que eles *sincronizam coisas diferentes,* e é bom dividi-los. Se você está preocupado com a duplicação, pode aprimorar este código [extraindo a lógica repetitiva em um Hook personalizado.](/learn/reusing-logic-with-custom-hooks#when-to-use-custom-hooks)

### Você está lendo algum state para calcular o próximo state? {/*are-you-reading-some-state-to-calculate-the-next-state*/}

Este Effect atualiza a variável de state `messages` com um novo array criado toda vez que uma nova mensagem chega:

```js {2,6-8}
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages([...messages, receivedMessage]);
    });
    // ...
```

Ele usa a variável `messages` para [criar um novo array](/learn/updating-arrays-in-state) começando com todas as mensagens existentes e adiciona a nova mensagem no final. No entanto, como `messages` é um valor reativo lido por um Effect, ele deve ser uma dependência:

```js {7,10}
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages([...messages, receivedMessage]);
    });
    return () => connection.disconnect();
  }, [roomId, messages]); // ✅ Todas as dependências declaradas
  // ...
```

E tornar `messages` uma dependência introduz um problema.

Toda vez que você recebe uma mensagem, `setMessages()` faz o componente renderizar novamente com um novo array `messages` que inclui a mensagem recebida. No entanto, uma vez que este Effect agora depende de `messages`, isso *também* vai ressincronizar o Effect. Então, cada nova mensagem fará o chat reconectar. O usuário não gostaria disso!

Para corrigir o problema, não leia `messages` dentro do Effect. Em vez disso, passe uma [função atualizadora](/reference/react/useState#updating-state-based-on-the-previous-state) para `setMessages`:

```js {7,10}
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages(msgs => [...msgs, receivedMessage]);
    });
    return () => connection.disconnect();
  }, [roomId]); // ✅ Todas as dependências declaradas
  // ...
```

**Observe como seu Effect não lê a variável `messages`.** Você só precisa passar uma função atualizadora como `msgs => [...msgs, receivedMessage]`. O React [coloca sua função atualizadora em uma fila](/learn/queueing-a-series-of-state-updates) e fornecerá o argumento `msgs` para ela durante a próxima renderização. É por isso que o próprio Effect não precisa depender mais de `messages`. Como resultado desta correção, receber uma mensagem de chat não fará mais o chat reconectar.

### Você quer ler um valor sem "reagir" às suas mudanças? {/*do-you-want-to-read-a-value-without-reacting-to-its-changes*/}

<Wip>

Esta seção descreve uma **API experimental que ainda não foi lançada** em uma versão estável do React.

</Wip>

Suponha que você queira tocar um som quando o usuário receber uma nova mensagem, a menos que `isMuted` seja `true`:

```js {3,10-12}
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  const [isMuted, setIsMuted] = useState(false);

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages(msgs => [...msgs, receivedMessage]);
      if (!isMuted) {
        playSound();
      }
    });
    // ...
```

Como o seu Effect agora usa `isMuted` em seu código, você tem que adicioná-lo às dependências:

```js {10,15}
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  const [isMuted, setIsMuted] = useState(false);

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages(msgs => [...msgs, receivedMessage]);
      if (!isMuted) {
        playSound();
      }
    });
    return () => connection.disconnect();
  }, [roomId, isMuted]); // ✅ Todas as dependências declaradas
  // ...
```

O problema é que toda vez que `isMuted` muda (por exemplo, quando o usuário pressiona o botão "Muted"), o Effect será ressincronizado e reconectará ao chat. Essa não é a experiência do usuário desejada! (Neste exemplo, mesmo desabilitar o lint não funcionaria - se você fizer isso, `isMuted` ficaria "preso" com seu valor antigo.)

Para resolver este problema, você precisa extrair a lógica que não deve ser reativa do Effect. Você não quer que este Effect "reaja" às mudanças em `isMuted`. [Mova esta parte não reativa da lógica para um Evento de Effect:](/learn/separating-events-from-effects#declaring-an-effect-event)

```js {1,7-12,18,21}
import { useState, useEffect, useEffectEvent } from 'react';

function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  const [isMuted, setIsMuted] = useState(false);

  const onMessage = useEffectEvent(receivedMessage => {
    setMessages(msgs => [...msgs, receivedMessage]);
    if (!isMuted) {
      playSound();
    }
  });

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      onMessage(receivedMessage);
    });
    return () => connection.disconnect();
  }, [roomId]); // ✅ Todas as dependências declaradas
  // ...
```

Eventos de Effect permitem que você divida um Effect em partes reativas (que devem "reagir" a valores reativos como `roomId` e suas mudanças) e partes não reativas (que só leem seus valores mais recentes, como `onMessage` lê `isMuted`). **Agora que você lê `isMuted` dentro de um Evento de Effect, ele não precisa ser uma dependência do seu Effect.** Como resultado, o chat não vai reconectar quando você ativar e desativar a configuração "Mudo", resolvendo o problema original!

#### Encapsulando um manipulador de eventos das props {/*wrapping-an-event-handler-from-the-props*/}

Você pode se deparar com um problema semelhante quando seu componente recebe um manipulador de eventos como uma prop:

```js {1,8,11}
function ChatRoom({ roomId, onReceiveMessage }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      onReceiveMessage(receivedMessage);
    });
    return () => connection.disconnect();
  }, [roomId, onReceiveMessage]); // ✅ Todas as dependências declaradas
  // ...
```

Suponha que o componente pai passe uma função `onReceiveMessage` *diferente* a cada renderização:

```js {3-5}
<ChatRoom
  roomId={roomId}
  onReceiveMessage={receivedMessage => {
    // ...
  }}
/>
```

Como `onReceiveMessage` é uma dependência, isso faria com que o Effect fosse ressincronizado após cada re-renderização pai. Isso faria com que ele se reconectasse ao bate-papo. Para resolver isso, encapsule a chamada em um Evento de Effect:

```js {4-6,12,15}
function ChatRoom({ roomId, onReceiveMessage }) {
  const [messages, setMessages] = useState([]);

  const onMessage = useEffectEvent(receivedMessage => {
    onReceiveMessage(receivedMessage);
  });

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      onMessage(receivedMessage);
    });
    return () => connection.disconnect();
  }, [roomId]); // ✅ Todas as dependências declaradas
  // ...
```

Eventos de Effect não são reativos, então você não precisa especificá-los como dependências. Como resultado, o chat não vai mais se reconectar mesmo que o componente pai passe uma função que é diferente em cada re-renderização.

#### Separando código reativo e não reativo {/*separating-reactive-and-non-reactive-code*/}

Neste exemplo, você quer registrar uma visita toda vez que `roomId` mudar. Você quer incluir o `notificationCount` atual com cada log, mas você *não* quer uma mudança para `notificationCount` para acionar um evento de log.

A solução é novamente dividir o código não reativo em um Evento de Effect:

```js {2-4,7}
function Chat({ roomId, notificationCount }) {
  const onVisit = useEffectEvent(visitedRoomId => {
    logVisit(visitedRoomId, notificationCount);
  });

  useEffect(() => {
    onVisit(roomId);
  }, [roomId]); // ✅ Todas as dependências declaradas
  // ...
}
```

Você quer que sua lógica seja reativa em relação ao `roomId`, então você lê `roomId` dentro do seu Effect. No entanto, você não quer que uma mudança para `notificationCount` registre uma visita extra, então você lê `notificationCount` dentro do Evento de Effect. [Saiba mais sobre como ler as últimas props e state de Effects usando Eventos de Effect.](/learn/separating-events-from-effects#reading-latest-props-and-state-with-effect-events)

### Algum valor reativo muda sem intenção? {/*does-some-reactive-value-change-unintentionally*/}

Às vezes, você *quer* que seu Effect "reaja" a um determinado valor, mas esse valor muda com mais frequência do que você gostaria - e pode não refletir nenhuma mudança real da perspectiva do usuário. Por exemplo, digamos que você crie um objeto `options` no corpo do seu componente e, em seguida, leia esse objeto de dentro do seu Effect:

```js {3-6,9}
function ChatRoom({ roomId }) {
  // ...
  const options = {
    serverUrl: serverUrl,
    roomId: roomId
  };

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    // ...
```

Este objeto é declarado no corpo do componente, então é um [valor reativo.](/learn/lifecycle-of-reactive-effects#effects-react-to-reactive-values) Quando você lê um valor reativo como este dentro de um Effect, você o declara como uma dependência. Isso garante que seu Effect "reaja" às suas mudanças:

```js {3,6}
  // ...
  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // ✅ Todas as dependências declaradas
  // ...
```

É importante declará-lo como uma dependência! Isso garante, por exemplo, que se o `roomId` mudar, seu Effect vai se reconectar ao chat com as novas `options`. No entanto, também há um problema com o código acima. Para vê-lo, tente digitar na entrada no sandbox abaixo e observe o que acontece no console:

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  // Desative temporariamente o linter para demonstrar o problema
  // eslint-disable-next-line react-hooks/exhaustive-deps
  const options = {
    serverUrl: serverUrl,
    roomId: roomId
  };

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]);

  return (
    <>
      <h1>Bem-vindo ao quarto {roomId}!</h1>
      <input value={message} onChange={e => setMessage(e.target.value)} />
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('geral');
  return (
    <>
      <label>
        Escolher o chat: {' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="geral">geral</option>
          <option value="viagem">viagem</option>
          <option value="música">música</option>
        </select>
      </label>
      <hr />
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

```js src/chat.js
export function createConnection({ serverUrl, roomId }) {
  // Uma implementação real realmente se conectaria ao servidor
  return {
    connect() {
      console.log('✅ Conectando ao quarto "' + roomId + '" em ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Desconectado do quarto "' + roomId + '" em ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

No sandbox acima, a entrada só atualiza a variável de state `message`. Da perspectiva do usuário, isso não deve afetar a conexão do chat. No entanto, toda vez que você atualiza a `message`, seu componente renderiza novamente. Quando seu componente renderiza novamente, o código dentro dele é executado novamente do zero.

Um novo objeto `options` é criado do zero em cada re-renderização do componente `ChatRoom`. O React vê que o objeto `options` é um *objeto diferente* do objeto `options` criado durante a última renderização. É por isso que ele ressincroniza seu Effect (que depende das `options`), e o chat se reconecta enquanto você digita.

**Este problema só afeta objetos e funções. No JavaScript, cada objeto e função recém-criados são considerados distintos de todos os outros. Não importa que o conteúdo dentro deles possa ser o mesmo!**

```js {7-8}
// Durante a primeira renderização
const options1 = { serverUrl: 'https://localhost:1234', roomId: 'música' };

// Durante a próxima renderização
const options2 = { serverUrl: 'https://localhost:1234', roomId: 'música' };

// Estes são dois objetos diferentes!
console.log(Object.is(options1, options2)); // false
```

**Dependências de objetos e funções podem fazer com que seu Effect ressincronize com mais frequência do que você precisa.**

É por isso que, sempre que possível, você deve tentar evitar objetos e funções como dependências do seu Effect. Em vez disso, tente movê-los fora do componente, dentro do Effect ou extrair valores primitivos deles.

#### Mova objetos e funções estáticos para fora do seu componente {/*move-static-objects-and-functions-outside-your-component*/}

Se o objeto não depender de nenhuma prop e state, você pode mover esse objeto para fora do seu componente:

```js {1-4,13}
const options = {
  serverUrl: 'https://localhost:1234',
  roomId: 'música'
};

function ChatRoom() {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, []); // ✅ Todas as dependências declaradas
  // ...
```

Desta forma, você *prova* para o linter que ele não é reativo. Ele não pode mudar como resultado de um re-renderizar, então não precisa ser uma dependência. Agora, re-renderizar `ChatRoom` não vai fazer seu Effect ressincronizar.

Isso funciona para funções também:

```js {1-6,12}
function createOptions() {
  return {
    serverUrl: 'https://localhost:1234',
    roomId: 'música'
  };
}

function ChatRoom() {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const options = createOptions();
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, []); // ✅ Todas as dependências declaradas
  // ...
```

Como `createOptions` é declarado fora do seu componente, não é um valor reativo. É por isso que ele não precisa ser especificado nas dependências do seu Effect e por que ele nunca fará seu Effect ressincronizar.

#### Mova objetos e funções dinâmicos para dentro do seu Effect {/*move-dynamic-objects-and-functions-inside-your-effect*/}

Se o seu objeto depender de algum valor reativo que pode mudar como resultado de um re-renderizar, como uma prop `roomId`, você não pode puxá-lo para *fora* do seu componente. Você pode, no entanto, mover sua criação *para dentro* do código do seu Effect:

```js {7-10,11,14}
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ Todas as dependências declaradas
  // ...
```

Agora que `options` é declarado dentro do seu Effect, ele não é mais uma dependência do seu Effect. Em vez disso, o único valor reativo usado pelo seu Effect é `roomId`. Como `roomId` não é um objeto ou função, você pode ter certeza de que ele não será *intencionalmente* diferente. No JavaScript, números e strings são comparados pelo seu conteúdo:

```js {7-8}
// Durante a primeira renderização
const roomId1 = 'música';

// Durante a próxima renderização
const roomId2 = 'música';

// Estas duas strings são as mesmas!
console.log(Object.is(roomId1, roomId2)); // true
```

Graças a esta correção, o chat não se reconecta mais se você editar a entrada:

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return (
    <>
      <h1>Bem-vindo ao quarto {roomId}!</h1>
      <input value={message} onChange={e => setMessage(e.target.value)} />
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('geral');
  return (
    <>
      <label>
        Escolha o chat:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="geral">geral</option>
          <option value="viagem">viagem</option>
          <option value="música">música</option>
        </select>
      </label>
      <hr />
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

```js src/chat.js
export function createConnection({ serverUrl, roomId }) {
  // Uma implementação real realmente se conectaria ao servidor
  return {
    connect() {
      console.log('✅ Conectando ao quarto "' + roomId + '" em ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Desconectado do quarto "' + roomId + '" em ' + serverUrl);
    }
  };
}
```
html
```
```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

No entanto, ele se reconecta *quando* você altera o dropdown `roomId`, como seria de se esperar.

Isso também funciona para funções:

```js {7-12,14}
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    function createOptions() {
      return {
        serverUrl: serverUrl,
        roomId: roomId
      };
    }

    const options = createOptions();
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ Todas as dependências declaradas
  // ...
```

Você pode escrever suas próprias funções para agrupar trechos de lógica dentro de seu efeito. Contanto que você também as declare *dentro* do seu efeito, elas não são valores reativos e, portanto, não precisam ser dependências do seu efeito.

#### Leia valores primitivos de objetos {/*read-primitive-values-from-objects*/}

Às vezes, você pode receber um objeto de props:

```js {1,5,8}
function ChatRoom({ options }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // ✅ Todas as dependências declaradas
  // ...
```

O risco aqui é que o componente pai criará o objeto durante a renderização:

```js {3-6}
<ChatRoom
  roomId={roomId}
  options={{
    serverUrl: serverUrl,
    roomId: roomId
  }}
/>
```

Isso faria com que seu efeito se reconectasse toda vez que o componente pai renderizasse novamente. Para corrigir isso, leia as informações do objeto *fora* do efeito e evite ter dependências de objetos e funções:

```js {4,7-8,12}
function ChatRoom({ options }) {
  const [message, setMessage] = useState('');

  const { roomId, serverUrl } = options;
  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // ✅ Todas as dependências declaradas
  // ...
```

A lógica fica um pouco repetitiva (você lê alguns valores de um objeto fora de um efeito e, em seguida, cria um objeto com os mesmos valores dentro do efeito). Mas isso deixa muito explícito em quais informações seu efeito *realmente* depende. Se um objeto for recriado intencionalmente pelo componente pai, o bate-papo não se reconectará. No entanto, se `options.roomId` ou `options.serverUrl` forem realmente diferentes, o bate-papo se reconectará.

#### Calcule valores primitivos de funções {/*calculate-primitive-values-from-functions*/}

A mesma abordagem pode funcionar para funções. Por exemplo, suponha que o componente pai passe uma função:

```js {3-8}
<ChatRoom
  roomId={roomId}
  getOptions={() => {
    return {
      serverUrl: serverUrl,
      roomId: roomId
    };
  }}
/>
```

Para evitar torná-la uma dependência (e fazê-la se reconectar nas re-renderizações), chame-a fora do efeito. Isso fornece os valores `roomId` e `serverUrl` que não são objetos e que você pode ler de dentro do seu efeito:

```js {1,4}
function ChatRoom({ getOptions }) {
  const [message, setMessage] = useState('');

  const { roomId, serverUrl } = getOptions();
  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // ✅ Todas as dependências declaradas
  // ...
```

Isso só funciona para funções [puras](/learn/keeping-components-pure) porque elas são seguras para serem chamadas durante a renderização. Se sua função for um manipulador de eventos, mas você não quiser que suas alterações ressincronizem seu efeito, [envolva-o em um Evento de Efeito em vez disso.](#do-you-want-to-read-a-value-without-reacting-to-its-changes)

<Recap>

- As dependências devem sempre corresponder ao código.
- Quando você não estiver satisfeito com suas dependências, o que você precisa editar é o código.
- Suprimir o linter leva a erros muito confusos, e você deve sempre evitá-lo.
- Para remover uma dependência, você precisa "provar" ao linter que ela não é necessária.
- Se algum código deve ser executado em resposta a uma interação específica, mova esse código para um manipulador de eventos.
- Se partes diferentes do seu efeito devem ser executadas novamente por motivos diferentes, divida-o em vários efeitos.
- Se você deseja atualizar algum estado com base no estado anterior, passe uma função atualizadora.
- Se você deseja ler o valor mais recente sem "reagir" a ele, extraia um Evento de Efeito do seu efeito.
- Em JavaScript, objetos e funções são considerados diferentes se foram criados em momentos diferentes.
- Tente evitar dependências de objetos e funções. Mova-as para fora do componente ou para dentro do efeito.

</Recap>

<Challenges>

#### Corrigir um intervalo de redefinição {/*fix-a-resetting-interval*/}

Este Efeito configura um intervalo que marca a cada segundo. Você notou algo estranho acontecendo: parece que o intervalo é destruído e recriado toda vez que ele marca. Corrija o código para que o intervalo não seja constantemente recriado.

<Hint>

Parece que o código deste efeito depende de `count`. Existe alguma maneira de não precisar dessa dependência? Deve haver uma maneira de atualizar o estado `count` com base em seu valor anterior sem adicionar uma dependência nesse valor.

</Hint>

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function Timer() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log('✅ Criando um intervalo');
    const id = setInterval(() => {
      console.log('⏰ Marca do intervalo');
      setCount(count + 1);
    }, 1000);
    return () => {
      console.log('❌ Limpando um intervalo');
      clearInterval(id);
    };
  }, [count]);

  return <h1>Contador: {count}</h1>
}
```

</Sandpack>

<Solution>

Você deseja atualizar o estado `count` para ser `count + 1` de dentro do efeito. No entanto, isso faz com que seu efeito dependa de `count`, que muda a cada marca, e é por isso que seu intervalo é recriado a cada marca.

Para resolver isso, use a [função atualizadora](/reference/react/useState#updating-state-based-on-the-previous-state) e escreva `setCount(c => c + 1)` em vez de `setCount(count + 1)`:

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function Timer() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log('✅ Criando um intervalo');
    const id = setInterval(() => {
      console.log('⏰ Marca do intervalo');
      setCount(c => c + 1);
    }, 1000);
    return () => {
      console.log('❌ Limpando um intervalo');
      clearInterval(id);
    };
  }, []);

  return <h1>Contador: {count}</h1>
}
```

</Sandpack>

Em vez de ler `count` dentro do efeito, você passa uma instrução `c => c + 1` ("incremente este número!") para o React. React irá aplicá-lo na próxima renderização. E como você não precisa mais ler o valor de `count` dentro do seu efeito, você pode manter as dependências do seu efeito vazias (`[]`). Isso impede que seu efeito recrie o intervalo a cada marca.

</Solution>

#### Corrigir uma animação de retrigger {/*fix-a-retriggering-animation*/}

Neste exemplo, quando você pressiona "Mostrar", uma mensagem de boas-vindas desaparece. A animação leva um segundo. Quando você pressiona "Remover", a mensagem de boas-vindas desaparece imediatamente. A lógica da animação de entrada é implementada no arquivo `animation.js` como um simples [loop de animação](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame) em JavaScript simples. Você não precisa alterar essa lógica. Você pode tratá-la como uma biblioteca de terceiros. Seu efeito cria uma instância de `FadeInAnimation` para o nó DOM e, em seguida, chama `start(duration)` ou `stop()` para controlar a animação. A `duration` é controlada por um controle deslizante. Ajuste o controle deslizante e veja como a animação muda.

Este código já funciona, mas há algo que você deseja alterar. Atualmente, quando você move o controle deslizante que controla a variável de estado `duration`, ele reinicia a animação. Altere o comportamento para que o efeito não "reaja" à variável `duration`. Quando você pressionar "Mostrar", o efeito deverá usar a `duration` atual no controle deslizante. No entanto, mover o próprio controle deslizante não deve, por si só, reiniciar a animação.

<Hint>

Existe uma linha de código dentro do efeito que não deve ser reativa? Como você pode mover o código não reativo do efeito?

</Hint>

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```js
import { useState, useEffect, useRef } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';
import { FadeInAnimation } from './animation.js';

function Welcome({ duration }) {
  const ref = useRef(null);

  useEffect(() => {
    const animation = new FadeInAnimation(ref.current);
    animation.start(duration);
    return () => {
      animation.stop();
    };
  }, [duration]);

  return (
    <h1
      ref={ref}
      style={{
        opacity: 0,
        color: 'white',
        padding: 50,
        textAlign: 'center',
        fontSize: 50,
        backgroundImage: 'radial-gradient(circle, rgba(63,94,251,1) 0%, rgba(252,70,107,1) 100%)'
      }}
    >
      Welcome
    </h1>
  );
}

export default function App() {
  const [duration, setDuration] = useState(1000);
  const [show, setShow] = useState(false);

  return (
    <>
      <label>
        <input
          type="range"
          min="100"
          max="3000"
          value={duration}
          onChange={e => setDuration(Number(e.target.value))}
        />
        <br />
        Fade in duration: {duration} ms
      </label>
      <button onClick={() => setShow(!show)}>
        {show ? 'Remove' : 'Show'}
      </button>
      <hr />
      {show && <Welcome duration={duration} />}
    </>
  );
}
```

```js src/animation.js
export class FadeInAnimation {
  constructor(node) {
    this.node = node;
  }
  start(duration) {
    this.duration = duration;
    if (this.duration === 0) {
      // Jump to end immediately
      this.onProgress(1);
    } else {
      this.onProgress(0);
      // Start animating
      this.startTime = performance.now();
      this.frameId = requestAnimationFrame(() => this.onFrame());
    }
  }
  onFrame() {
    const timePassed = performance.now() - this.startTime;
    const progress = Math.min(timePassed / this.duration, 1);
    this.onProgress(progress);
    if (progress < 1) {
      // We still have more frames to paint
      this.frameId = requestAnimationFrame(() => this.onFrame());
    }
  }
  onProgress(progress) {
    this.node.style.opacity = progress;
  }
  stop() {
    cancelAnimationFrame(this.frameId);
    this.startTime = null;
    this.frameId = null;
    this.duration = 0;
  }
}
```

```css
label, button { display: block; margin-bottom: 20px; }
html, body { min-height: 300px; }
```

</Sandpack>

<Solution>

Seu efeito precisa ler o valor mais recente de `duration`, mas você não quer que ele "reaja" às alterações em `duration`. Você usa `duration` para iniciar a animação, mas iniciar a animação não é reativo. Extraia a linha de código não reativa para um Evento de Efeito e chame essa função do seu Efeito.

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```js
import { useState, useEffect, useRef } from 'react';
import { FadeInAnimation } from './animation.js';
import { experimental_useEffectEvent as useEffectEvent } from 'react';

function Welcome({ duration }) {
  const ref = useRef(null);

  const onAppear = useEffectEvent(animation => {
    animation.start(duration);
  });

  useEffect(() => {
    const animation = new FadeInAnimation(ref.current);
    onAppear(animation);
    return () => {
      animation.stop();
    };
  }, []);

  return (
    <h1
      ref={ref}
      style={{
        opacity: 0,
        color: 'white',
        padding: 50,
        textAlign: 'center',
        fontSize: 50,
        backgroundImage: 'radial-gradient(circle, rgba(63,94,251,1) 0%, rgba(252,70,107,1) 100%)'
      }}
    >
      Welcome
    </h1>
  );
}

export default function App() {
  const [duration, setDuration] = useState(1000);
  const [show, setShow] = useState(false);

  return (
    <>
      <label>
        <input
          type="range"
          min="100"
          max="3000"
          value={duration}
          onChange={e => setDuration(Number(e.target.value))}
        />
        <br />
        Fade in duration: {duration} ms
      </label>
      <button onClick={() => setShow(!show)}>
        {show ? 'Remove' : 'Show'}
      </button>
      <hr />
      {show && <Welcome duration={duration} />}
    </>
  );
}
```

```js src/animation.js
export class FadeInAnimation {
  constructor(node) {
    this.node = node;
  }
  start(duration) {
    this.duration = duration;
    this.onProgress(0);
    this.startTime = performance.now();
    this.frameId = requestAnimationFrame(() => this.onFrame());
  }
  onFrame() {
    const timePassed = performance.now() - this.startTime;
    const progress = Math.min(timePassed / this.duration, 1);
    this.onProgress(progress);
    if (progress < 1) {
      // We still have more frames to paint
      this.frameId = requestAnimationFrame(() => this.onFrame());
    }
  }
  onProgress(progress) {
    this.node.style.opacity = progress;
  }
  stop() {
    cancelAnimationFrame(this.frameId);
    this.startTime = null;
    this.frameId = null;
    this.duration = 0;
  }
}
```

```css
label, button { display: block; margin-bottom: 20px; }
html, body { min-height: 300px; }
```

</Sandpack>

Eventos de Efeito como `onAppear` não são reativos, portanto, você pode ler `duration` dentro sem reativar a animação.

</Solution>

#### Corrigir um bate-papo de reconexão {/*fix-a-reconnecting-chat*/}

Neste exemplo, toda vez que você pressiona "Alternar tema", o bate-papo se reconecta. Por que isso acontece? Corrija o erro para que o bate-papo se reconecte somente quando você edita a URL do servidor ou escolhe uma sala de bate-papo diferente.

Trate `chat.js` como uma biblioteca externa de terceiros: você pode consultá-la para verificar sua API, mas não a edite.

<Hint>

Há mais de uma maneira de corrigir isso, mas, em última análise, você deseja evitar ter um objeto como sua dependência.

</Hint>

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

export default function App() {
  const [isDark, setIsDark] = useState(false);
  const [roomId, setRoomId] = useState('general');
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  const options = {
    serverUrl: serverUrl,
    roomId: roomId
  };

  return (
    <div className={isDark ? 'dark' : 'light'}>
      <button onClick={() => setIsDark(!isDark)}>
        Alternar tema
      </button>
      <label>
        URL do servidor:{' '}
        <input
          value={serverUrl}
          onChange={e => setServerUrl(e.target.value)}
        />
      </label>
      <label>
        Escolha a sala de bate-papo:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">geral</option>
          <option value="travel">viagem</option>
          <option value="music">música</option>
        </select>
      </label>
      <hr />
      <ChatRoom options={options} />
    </div>
  );
}
```

```js src/ChatRoom.js active
import { useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom({ options }) {
  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]);

  return <h1>Bem-vindo à sala {options.roomId}!</h1>;
}
```

```js src/chat.js
export function createConnection({ serverUrl, roomId }) {
  // Uma implementação real realmente se conectaria ao servidor
  if (typeof serverUrl !== 'string') {
    throw Error('Esperava-se que serverUrl fosse uma string. Recebido: ' + serverUrl);
  }
  if (typeof roomId !== 'string') {
    throw Error('Esperava-se que roomId fosse uma string. Recebido: ' + roomId);
  }
  return {
    connect() {
      console.log('✅ Conectando-se à sala "' + roomId + '" em ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Desconectado da sala "' + roomId + '" em ' + serverUrl);
    }
  };
}
```

```css
label, button { display: block; margin-bottom: 5px; }
.dark { background: #222; color: #eee; }
```

</Sandpack>

<Solution>

Seu efeito está sendo executado novamente porque depende do objeto `options`. Objetos podem ser recriados intencionalmente, você deve tentar evitá-los como dependências de seus efeitos sempre que possível.

A correção menos invasiva é ler `roomId` e `serverUrl` logo fora do efeito e, em seguida, fazer com que o efeito dependa desses valores primitivos (que não podem mudar intencionalmente). Dentro do Efeito, crie um objeto e passe-o para `createConnection`:

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

export default function App() {
  const [isDark, setIsDark] = useState(false);
  const [roomId, setRoomId] = useState('general');
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  const options = {
    serverUrl: serverUrl,
    roomId: roomId
  };

  return (
    <div className={isDark ? 'dark' : 'light'}>
      <button onClick={() => setIsDark(!isDark)}>
        Alternar tema
      </button>
      <label>
        URL do servidor:{' '}
        <input
          value={serverUrl}
          onChange={e => setServerUrl(e.target.value)}
        />
      </label>
      <label>
        Escolha a sala de bate-papo:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">geral</option>
          <option value="travel">viagem</option>
          <option value="music">música</option>
        </select>
      </label>
      <hr />
      <ChatRoom options={options} />
    </div>
  );
}
```

```js src/ChatRoom.js active
import { useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom({ options }) {
  const { roomId, serverUrl } = options;
  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]);

  return <h1>Bem-vindo à sala {options.roomId}!</h1>;
}
``````js src/chat.js
export function createConnection({ serverUrl, roomId }) {
  // Uma implementação real realmente iria conectar ao servidor
  if (typeof serverUrl !== 'string') {
    throw Error('Esperava serverUrl para ser uma string. Recebido: ' + serverUrl);
  }
  if (typeof roomId !== 'string') {
    throw Error('Esperava roomId para ser uma string. Recebido: ' + roomId);
  }
  return {
    connect() {
      console.log('✅ Conectando à sala "' + roomId + '" em ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Desconectado da sala "' + roomId + '" em ' + serverUrl);
    }
  };
}
```

```css
label, button { display: block; margin-bottom: 5px; }
.dark { background: #222; color: #eee; }
```

</Sandpack>

Seria ainda melhor substituir a prop `options` do objeto pelas props mais específicas, como `roomId` e `serverUrl`:

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

export default function App() {
  const [isDark, setIsDark] = useState(false);
  const [roomId, setRoomId] = useState('general');
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  return (
    <div className={isDark ? 'dark' : 'light'}>
      <button onClick={() => setIsDark(!isDark)}>
        Alternar tema
      </button>
      <label>
        URL do Servidor:{' '}
        <input
          value={serverUrl}
          onChange={e => setServerUrl(e.target.value)}
        />
      </label>
      <label>
        Escolha a sala de bate-papo:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">geral</option>
          <option value="travel">viagem</option>
          <option value="music">música</option>
        </select>
      </label>
      <hr />
      <ChatRoom
        roomId={roomId}
        serverUrl={serverUrl}
      />
    </div>
  );
}
```

```js src/ChatRoom.js active
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom({ roomId, serverUrl }) {
  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]);

  return <h1>Bem-vindo à sala {roomId}!</h1>;
}
```

```js src/chat.js
export function createConnection({ serverUrl, roomId }) {
  // Uma implementação real realmente iria conectar ao servidor
  if (typeof serverUrl !== 'string') {
    throw Error('Esperava serverUrl para ser uma string. Recebido: ' + serverUrl);
  }
  if (typeof roomId !== 'string') {
    throw Error('Esperava roomId para ser uma string. Recebido: ' + roomId);
  }
  return {
    connect() {
      console.log('✅ Conectando à sala "' + roomId + '" em ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Desconectado da sala "' + roomId + '" em ' + serverUrl);
    }
  };
}
```

```css
label, button { display: block; margin-bottom: 5px; }
.dark { background: #222; color: #eee; }
```

</Sandpack>

Manter as props primitivas sempre que possível facilita a otimização de seus componentes mais tarde.

</Solution>

#### Corrigir um chat reconectando, novamente {/*fix-a-reconnecting-chat-again*/}

Este exemplo se conecta ao chat com ou sem criptografia. Marque a caixa de seleção e observe as mensagens diferentes no console quando a criptografia está ativada e desativada. Tente mudar a sala. Em seguida, tente alternar o tema. Quando você estiver conectado a uma sala de chat, receberá novas mensagens a cada poucos segundos. Verifique se a cor corresponde ao tema que você escolheu.

Neste exemplo, o chat se reconecta toda vez que você tenta alterar o tema. Corrija isso. Após a correção, a alteração do tema não deve reconectar o chat, mas a alternância das configurações de criptografia ou a alteração da sala devem reconectar.

Não altere nenhum código em `chat.js`. Fora isso, você pode alterar qualquer código, desde que resulte no mesmo comportamento. Por exemplo, pode ser útil alterar quais props estão sendo passadas.

<Hint>

Você está passando duas funções: `onMessage` e `createConnection`. Ambas são criadas do zero toda vez que `App` renderiza novamente. Elas são consideradas novos valores toda vez, e é por isso que elas reacionam o seu Effect.

Uma dessas funções é um manipulador de eventos. Você conhece alguma forma de chamar um manipulador de eventos de um Effect sem "reacionar" aos novos valores da função manipuladora de eventos? Isso seria útil!

Outra dessas funções só existe para passar algum estado para um método de API importado. Essa função é realmente necessária? Qual é a informação essencial que está sendo passada? Pode ser necessário mover algumas imports de `App.js` para `ChatRoom.js`.

</Hint>

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest",
    "toastify-js": "1.12.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```js src/App.js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';
import {
  createEncryptedConnection,
  createUnencryptedConnection,
} from './chat.js';
import { showNotification } from './notifications.js';

export default function App() {
  const [isDark, setIsDark] = useState(false);
  const [roomId, setRoomId] = useState('general');
  const [isEncrypted, setIsEncrypted] = useState(false);

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Usar tema escuro
      </label>
      <label>
        <input
          type="checkbox"
          checked={isEncrypted}
          onChange={e => setIsEncrypted(e.target.checked)}
        />
        Habilitar criptografia
      </label>
      <label>
        Escolha a sala de bate-papo:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">geral</option>
          <option value="travel">viagem</option>
          <option value="music">música</option>
        </select>
      </label>
      <hr />
      <ChatRoom
        roomId={roomId}
        onMessage={msg => {
          showNotification('Nova mensagem: ' + msg, isDark ? 'dark' : 'light');
        }}
        createConnection={() => {
          const options = {
            serverUrl: 'https://localhost:1234',
            roomId: roomId
          };
          if (isEncrypted) {
            return createEncryptedConnection(options);
          } else {
            return createUnencryptedConnection(options);
          }
        }}
      />
    </>
  );
}
```

```js src/ChatRoom.js active
import { useState, useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';

export default function ChatRoom({ roomId, createConnection, onMessage }) {
  useEffect(() => {
    const connection = createConnection();
    connection.on('message', (msg) => onMessage(msg));
    connection.connect();
    return () => connection.disconnect();
  }, [createConnection, onMessage]);

  return <h1>Bem-vindo à sala {roomId}!</h1>;
}
```

```js src/chat.js
export function createEncryptedConnection({ serverUrl, roomId }) {
  // Uma implementação real realmente iria conectar ao servidor
  if (typeof serverUrl !== 'string') {
    throw Error('Esperava serverUrl para ser uma string. Recebido: ' + serverUrl);
  }
  if (typeof roomId !== 'string') {
    throw Error('Esperava roomId para ser uma string. Recebido: ' + roomId);
  }
  let intervalId;
  let messageCallback;
  return {
    connect() {
      console.log('✅ 🔐 Conectando à sala "' + roomId + '"... (criptografado)');
      clearInterval(intervalId);
      intervalId = setInterval(() => {
        if (messageCallback) {
          if (Math.random() > 0.5) {
            messageCallback('hey')
          } else {
            messageCallback('lol');
          }
        }
      }, 3000);
    },
    disconnect() {
      clearInterval(intervalId);
      messageCallback = null;
      console.log('❌ 🔐 Desconectado da sala "' + roomId + '" (criptografado)');
    },
    on(event, callback) {
      if (messageCallback) {
        throw Error('Não é possível adicionar o manipulador duas vezes.');
      }
      if (event !== 'message') {
        throw Error('Apenas o evento "message" é suportado.');
      }
      messageCallback = callback;
    },
  };
}

export function createUnencryptedConnection({ serverUrl, roomId }) {
  // Uma implementação real realmente iria conectar ao servidor
  if (typeof serverUrl !== 'string') {
    throw Error('Esperava serverUrl para ser uma string. Recebido: ' + serverUrl);
  }
  if (typeof roomId !== 'string') {
    throw Error('Esperava roomId para ser uma string. Recebido: ' + roomId);
  }
  let intervalId;
  let messageCallback;
  return {
    connect() {
      console.log('✅ Conectando à sala "' + roomId + '" (não criptografado)...');
      clearInterval(intervalId);
      intervalId = setInterval(() => {
        if (messageCallback) {
          if (Math.random() > 0.5) {
            messageCallback('hey')
          } else {
            messageCallback('lol');
          }
        }
      }, 3000);
    },
    disconnect() {
      clearInterval(intervalId);
      messageCallback = null;
      console.log('❌ Desconectado da sala "' + roomId + '" (não criptografado)');
    },
    on(event, callback) {
      if (messageCallback) {
        throw Error('Não é possível adicionar o manipulador duas vezes.');
      }
      if (event !== 'message') {
        throw Error('Apenas o evento "message" é suportado.');
      }
      messageCallback = callback;
    },
  };
}
```

```js src/notifications.js
import Toastify from 'toastify-js';
import 'toastify-js/src/toastify.css';

export function showNotification(message, theme) {
  Toastify({
    text: message,
    duration: 2000,
    gravity: 'top',
    position: 'right',
    style: {
      background: theme === 'dark' ? 'black' : 'white',
      color: theme === 'dark' ? 'white' : 'black',
    },
  }).showToast();
}
```

```css
label, button { display: block; margin-bottom: 5px; }
```

</Sandpack>

<Solution>

Há mais de uma forma correta de resolver isso, mas aqui está uma possível solução.

No exemplo original, alternar o tema causou a criação e o envio de funções `onMessage` e `createConnection` diferentes. Como o Effect dependia dessas funções, o chat se reconectava toda vez que você alternava o tema.

Para corrigir o problema com `onMessage`, você precisava encapsulá-lo em um Evento de Effect:

```js {1,2,6}
export default function ChatRoom({ roomId, createConnection, onMessage }) {
  const onReceiveMessage = useEffectEvent(onMessage);

  useEffect(() => {
    const connection = createConnection();
    connection.on('message', (msg) => onReceiveMessage(msg));
    // ...
```

Ao contrário da prop `onMessage`, o Evento de Effect `onReceiveMessage` não é reativo. É por isso que ele não precisa ser uma dependência do seu Effect. Como resultado, as alterações em `onMessage` não farão com que o chat se reconecte.

Você não pode fazer o mesmo com `createConnection` porque ele *deve* ser reativo. Você *quer* que o Effect re-dispare se o usuário alternar entre uma conexão criptografada e uma não criptografada, ou se o usuário mudar a sala atual. No entanto, como `createConnection` é uma função, você não pode verificar se as informações que ela lê foram *realmente* alteradas ou não. Para resolver isso, em vez de passar `createConnection` do componente `App`, passe os valores brutos de `roomId` e `isEncrypted`:

```js {2-3}
      <ChatRoom
        roomId={roomId}
        isEncrypted={isEncrypted}
        onMessage={msg => {
          showNotification('Nova mensagem: ' + msg, isDark ? 'dark' : 'light');
        }}
      />
```

Agora você pode mover a função `createConnection` *dentro* do Effect em vez de passá-la do `App`:

```js {1-4,6,10-20}
import {
  createEncryptedConnection,
  createUnencryptedConnection,
} from './chat.js';

export default function ChatRoom({ roomId, isEncrypted, onMessage }) {
  const onReceiveMessage = useEffectEvent(onMessage);

  useEffect(() => {
    function createConnection() {
      const options = {
        serverUrl: 'https://localhost:1234',
        roomId: roomId
      };
      if (isEncrypted) {
        return createEncryptedConnection(options);
      } else {
        return createUnencryptedConnection(options);
      }
    }
    // ...
```

Após essas duas alterações, seu Effect não depende mais de nenhum valor de função:

```js {1,8,10,21}
export default function ChatRoom({ roomId, isEncrypted, onMessage }) { // Valores reativos
  const onReceiveMessage = useEffectEvent(onMessage); // Não reativo

  useEffect(() => {
    function createConnection() {
      const options = {
        serverUrl: 'https://localhost:1234',
        roomId: roomId // Ler um valor reativo
      };
      if (isEncrypted) { // Ler um valor reativo
        return createEncryptedConnection(options);
      } else {
        return createUnencryptedConnection(options);
      }
    }

    const connection = createConnection();
    connection.on('message', (msg) => onReceiveMessage(msg));
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, isEncrypted]); // ✅ Todas as dependências declaradas
```

Como resultado, o chat se reconecta somente quando algo significativo (`roomId` ou `isEncrypted`) muda:

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest",
    "toastify-js": "1.12.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```js src/App.js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

import { showNotification } from './notifications.js';

export default function App() {
  const [isDark, setIsDark] = useState(false);
  const [roomId, setRoomId] = useState('general');
  const [isEncrypted, setIsEncrypted] = useState(false);

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Usar tema escuro
      </label>
      <label>
        <input
          type="checkbox"
          checked={isEncrypted}
          onChange={e => setIsEncrypted(e.target.checked)}
        />
        Habilitar criptografia
      </label>
      <label>
        Escolha a sala de bate-papo:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">geral</option>
          <option value="travel">viagem</option>
          <option value="music">música</option>
        </select>
      </label>
      <hr />
      <ChatRoom
        roomId={roomId}
        isEncrypted={isEncrypted}
        onMessage={msg => {
          showNotification('Nova mensagem: ' + msg, isDark ? 'dark' : 'light');
        }}
      />
    </>
  );
}
```

```js src/ChatRoom.js active
import { useState, useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';
import {
  createEncryptedConnection,
  createUnencryptedConnection,
} from './chat.js';

export default function ChatRoom({ roomId, isEncrypted, onMessage }) {
  const onReceiveMessage = useEffectEvent(onMessage);

  useEffect(() => {
    function createConnection() {
      const options = {
        serverUrl: 'https://localhost:1234',
        roomId: roomId
      };
      if (isEncrypted) {
        return createEncryptedConnection(options);
      } else {
        return createUnencryptedConnection(options);
      }
    }

    const connection = createConnection();
    connection.on('message', (msg) => onReceiveMessage(msg));
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, isEncrypted]);

  return <h1>Bem-vindo à sala {roomId}!</h1>;
}
```

```js src/chat.js
export function createEncryptedConnection({ serverUrl, roomId }) {
  // Uma implementação real realmente iria conectar ao servidor
  if (typeof serverUrl !== 'string') {
    throw Error('Esperava serverUrl para ser uma string. Recebido: ' + serverUrl);
  }
  if (typeof roomId !== 'string') {
    throw Error('Esperava roomId para ser uma string. Recebido: ' + roomId);
  }
  let intervalId;
  let messageCallback;
  return {
    connect() {
      console.log('✅ 🔐 Conectando à sala "' + roomId + '"... (criptografado)');
      clearInterval(intervalId);
      intervalId = setInterval(() => {
        if (messageCallback) {
          if (Math.random() > 0.5) {
            messageCallback('hey')
          } else {
            messageCallback('lol');
          }
        }
      }, 3000);
    },
    disconnect() {
      clearInterval(intervalId);
      messageCallback = null;
      console.log('❌ 🔐 Desconectado da sala "' + roomId + '" (criptografado)');
    },
    on(event, callback) {
      if (messageCallback) {
        throw Error('Não é possível adicionar o manipulador duas vezes.');
      }
      if (event !== 'message') {
        throw Error('Apenas o evento "message" é suportado.');
      }
      messageCallback = callback;
    },
  };
}

export function createUnencryptedConnection({ serverUrl, roomId }) {
  // Uma implementação real realmente iria conectar ao servidor
  if (typeof serverUrl !== 'string') {
    throw Error('Esperava serverUrl para ser uma string. Recebido: ' + serverUrl);
  }
  if (typeof roomId !== 'string') {
    throw Error('Esperava roomId para ser uma string. Recebido: ' + roomId);
  }
  let intervalId;
  let messageCallback;
  return {
    connect() {
      console.log('✅ Conectando à sala "' + roomId + '" (não criptografado)...');
      clearInterval(intervalId);
      intervalId = setInterval(() => {
        if (messageCallback) {
          if (Math.random() > 0.5) {
            messageCallback('hey')
          } else {
            messageCallback('lol');
          }
        }
      }, 3000);
    },
    disconnect() {
      clearInterval(intervalId);
      messageCallback = null;
      console.log('❌ Desconectado da sala "' + roomId + '" (não criptografado)');
    },
    on(event, callback) {
      if (messageCallback) {
        throw Error('Não é possível adicionar o manipulador duas vezes.');
      }
      if (event !== 'message') {
        throw Error('Apenas o evento "message" é suportado.');
      }
      messageCallback = callback;
    },
  };
}
```

```js src/notifications.js
import Toastify from 'toastify-js';
import 'toastify-js/src/toastify.css';

export function showNotification(message, theme) {
  Toastify({
    text: message,
    duration: 2000,
    gravity: 'top',
    position: 'right',
    style: {
      background: theme === 'dark' ? 'black' : 'white',
      color: theme === 'dark' ? 'white' : 'black',
    },
  }).showToast();
}
```

```css
label, button { display: block; margin-bottom: 5px; }
```

</Sandpack>

</Solution>

</Challenges>