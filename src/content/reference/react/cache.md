---
title: cache
canary: true
---

<RSC>

`cache` é apenas para uso com [React Server Components](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#react-server-components).

</RSC>

<Intro>

`cache` permite que você armazene em cache o resultado de uma busca ou cálculo de dados.

```js
const cachedFn = cache(fn);
```

</Intro>

<InlineToc />

---

## Referência {/*reference*/}

### `cache(fn)` {/*cache*/}

Chame `cache` fora de quaisquer componentes para criar uma versão da função com cache.

```js {4,7}
import {cache} from 'react';
import calculateMetrics from 'lib/metrics';

const getMetrics = cache(calculateMetrics);

function Chart({data}) {
  const report = getMetrics(data);
  // ...
}
```

Quando `getMetrics` é chamado pela primeira vez com `data`, `getMetrics` chamará `calculateMetrics(data)` e armazenará o resultado em cache. Se `getMetrics` for chamado novamente com os mesmos `data`, ele retornará o resultado em cache em vez de chamar `calculateMetrics(data)` novamente.

[Veja mais exemplos abaixo.](#usage)

#### Parâmetros {/*parameters*/}

- `fn`: A função para a qual você deseja armazenar resultados em cache. `fn` pode receber quaisquer argumentos e retornar qualquer valor.

#### Retorna {/*returns*/}

`cache` retorna uma versão em cache de `fn` com a mesma assinatura de tipo. Ele não chama `fn` no processo.

Ao chamar `cachedFn` com determinados argumentos, ele primeiro verifica se um resultado em cache existe no cache. Se um resultado em cache existir, ele retorna o resultado. Caso contrário, ele chama `fn` com os argumentos, armazena o resultado no cache e retorna o resultado. A única vez que `fn` é chamada é quando há uma falha de cache.

<Note>

A otimização para armazenar em cache os valores de retorno com base nas entradas é conhecida como [_memoization_](https://en.wikipedia.org/wiki/Memoization). Nós nos referimos à função retornada de `cache` como uma função memoizada.

</Note>

#### Ressalvas {/*caveats*/}

[//]: # 'TODO: add links to Server/Client Component reference once https://github.com/reactjs/react.dev/pull/6177 is merged'

- React irá invalidar o cache para todas as funções memoizadas para cada solicitação do servidor.
- Cada chamada para `cache` cria uma nova função. Isso significa que chamar `cache` com a mesma função várias vezes retornará diferentes funções memoizadas que não compartilham o mesmo cache.
- `cachedFn` também irá armazenar erros em cache. Se `fn` lançar um erro para determinados argumentos, ele será armazenado em cache, e o mesmo erro será lançado novamente quando `cachedFn` for chamado com esses mesmos argumentos.
- `cache` é para uso apenas em [Componentes de Servidor](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#react-server-components).

---

## Uso {/*usage*/}

### Armazenar um cálculo dispendioso em cache {/*cache-expensive-computation*/}

Use `cache` para pular trabalho duplicado.

```js [[1, 7, "getUserMetrics(user)"],[2, 13, "getUserMetrics(user)"]]
import {cache} from 'react';
import calculateUserMetrics from 'lib/user';

const getUserMetrics = cache(calculateUserMetrics);

function Profile({user}) {
  const metrics = getUserMetrics(user);
  // ...
}

function TeamReport({users}) {
  for (let user in users) {
    const metrics = getUserMetrics(user);
    // ...
  }
  // ...
}
```

Se o mesmo objeto `user` for renderizado em `Profile` e `TeamReport`, os dois componentes podem compartilhar trabalho e chamar apenas `calculateUserMetrics` uma vez para esse `user`.

Suponha que `Profile` seja renderizado primeiro. Ele chamará <CodeStep step={1}>`getUserMetrics`</CodeStep> e verificará se existe um resultado em cache. Como é a primeira vez que `getUserMetrics` é chamado com esse `user`, haverá uma falha de cache. `getUserMetrics` então chamará `calculateUserMetrics` com esse `user` e escreverá o resultado no cache.

Quando `TeamReport` renderizar sua lista de `users` e atingir o mesmo objeto `user`, chamará <CodeStep step={2}>`getUserMetrics`</CodeStep> e lerá o resultado do cache.

<Pitfall>

##### Chamar diferentes funções memoizadas lerá de diferentes caches. {/*pitfall-different-memoized-functions*/}

Para acessar o mesmo cache, os componentes devem chamar a mesma função memoizada.

```js [[1, 7, "getWeekReport"], [1, 7, "cache(calculateWeekReport)"], [1, 8, "getWeekReport"]]
// Temperature.js
import {cache} from 'react';
import {calculateWeekReport} from './report';

export function Temperature({cityData}) {
  // 🚩 Erro: Chamada de `cache` no componente cria um novo `getWeekReport` para cada renderização
  const getWeekReport = cache(calculateWeekReport);
  const report = getWeekReport(cityData);
  // ...
}
```

```js [[2, 6, "getWeekReport"], [2, 6, "cache(calculateWeekReport)"], [2, 9, "getWeekReport"]]
// Precipitation.js
import {cache} from 'react';
import {calculateWeekReport} from './report';

// 🚩 Erro: `getWeekReport` é acessível apenas para o componente `Precipitation`.
const getWeekReport = cache(calculateWeekReport);

export function Precipitation({cityData}) {
  const report = getWeekReport(cityData);
  // ...
}
```

No exemplo acima, <CodeStep step={2}>`Precipitation`</CodeStep> e <CodeStep step={1}>`Temperature`</CodeStep> cada um chama `cache` para criar uma nova função memoizada com sua própria pesquisa em cache. Se ambos os componentes renderizarem para o mesmo `cityData`, eles farão um trabalho duplicado para chamar `calculateWeekReport`.

Além disso, `Temperature` cria uma <CodeStep step={1}>nova função memoizada</CodeStep> toda vez que o componente é renderizado, o que não permite nenhum compartilhamento de cache.

Para maximizar as ocorrências de cache e reduzir o trabalho, os dois componentes devem chamar a mesma função memoizada para acessar o mesmo cache. Em vez disso, defina a função memoizada em um módulo dedicado que possa ser [`import`-ado](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import) entre os componentes.

```js [[3, 5, "export default cache(calculateWeekReport)"]]
// getWeekReport.js
import {cache} from 'react';
import {calculateWeekReport} from './report';

export default cache(calculateWeekReport);
```

```js [[3, 2, "getWeekReport", 0], [3, 5, "getWeekReport"]]
// Temperature.js
import getWeekReport from './getWeekReport';

export default function Temperature({cityData}) {
	const report = getWeekReport(cityData);
  // ...
}
```

```js [[3, 2, "getWeekReport", 0], [3, 5, "getWeekReport"]]
// Precipitation.js
import getWeekReport from './getWeekReport';

export default function Precipitation({cityData}) {
  const report = getWeekReport(cityData);
  // ...
}
```
Aqui, ambos os componentes chamam a <CodeStep step={3}>mesma função memoizada</CodeStep> exportada de `./getWeekReport.js` para ler e escrever no mesmo cache.
</Pitfall>

### Compartilhar um snapshot de dados {/*take-and-share-snapshot-of-data*/}

Para compartilhar um snapshot de dados entre os componentes, chame `cache` com uma função de busca de dados como `fetch`. Quando vários componentes fazem a mesma busca de dados, apenas uma solicitação é feita e os dados retornados são armazenados em cache e compartilhados entre os componentes. Todos os componentes se referem ao mesmo snapshot de dados na renderização do servidor.

```js [[1, 4, "city"], [1, 5, "fetchTemperature(city)"], [2, 4, "getTemperature"], [2, 9, "getTemperature"], [1, 9, "city"], [2, 14, "getTemperature"], [1, 14, "city"]]
import {cache} from 'react';
import {fetchTemperature} from './api.js';

const getTemperature = cache(async (city) => {
	return await fetchTemperature(city);
});

async function AnimatedWeatherCard({city}) {
	const temperature = await getTemperature(city);
	// ...
}

async function MinimalWeatherCard({city}) {
	const temperature = await getTemperature(city);
	// ...
}
```

Se `AnimatedWeatherCard` e `MinimalWeatherCard` renderizarem para o mesmo <CodeStep step={1}>city</CodeStep>, eles receberão o mesmo snapshot de dados da <CodeStep step={2}>função memoizada</CodeStep>.

Se `AnimatedWeatherCard` e `MinimalWeatherCard` fornecerem diferentes argumentos <CodeStep step={1}>city</CodeStep> para <CodeStep step={2}>`getTemperature`</CodeStep>, então `fetchTemperature` será chamado duas vezes e cada local de chamada receberá dados diferentes.

O <CodeStep step={1}>city</CodeStep> age como uma chave de cache.

<Note>

[//]: # 'TODO: add links to Server Components when merged.'

<CodeStep step={3}>Renderização assíncrona</CodeStep> é suportada apenas para Componentes de Servidor.

```js [[3, 1, "async"], [3, 2, "await"]]
async function AnimatedWeatherCard({city}) {
	const temperature = await getTemperature(city);
	// ...
}
```
[//]: # 'TODO: add link and mention to use documentation when merged'
[//]: # 'To render components that use asynchronous data in Client Components, see `use` documentation.'

</Note>

### Pré-carregar dados {/*preload-data*/}

Ao armazenar em cache uma busca de dados de longa duração, você pode iniciar o trabalho assíncrono antes de renderizar o componente.

```jsx [[2, 6, "await getUser(id)"], [1, 17, "getUser(id)"]]
const getUser = cache(async (id) => {
  return await db.user.query(id);
});

async function Profile({id}) {
  const user = await getUser(id);
  return (
    <section>
      <img src={user.profilePic} />
      <h2>{user.name}</h2>
    </section>
  );
}

function Page({id}) {
  // ✅ Bom: começar a buscar os dados do usuário
  getUser(id);
  // ... some computational work
  return (
    <>
      <Profile id={id} />
    </>
  );
}
```

Ao renderizar `Page`, o componente chama <CodeStep step={1}>`getUser`</CodeStep>, mas observe que ele não usa os dados retornados. Essa primeira chamada <CodeStep step={1}>`getUser`</CodeStep> inicia a consulta assíncrona do banco de dados que ocorre enquanto `Page` está fazendo outro trabalho computacional e renderizando os filhos.

Ao renderizar `Profile`, chamamos <CodeStep step={2}>`getUser`</CodeStep> novamente. Se a chamada inicial <CodeStep step={1}>`getUser`</CodeStep> já retornou e armazenou em cache os dados do usuário, quando `Profile` <CodeStep step={2}>pergunta e espera por esses dados</CodeStep>, ele pode simplesmente ler do cache sem exigir outra chamada de procedimento remoto. Se a <CodeStep step={1}>solicitação inicial de dados</CodeStep> não foi concluída, o pré-carregamento de dados nesse padrão reduz o atraso na busca de dados.

<DeepDive>

#### Armazenando o trabalho assíncrono em cache {/*caching-asynchronous-work*/}

Ao avaliar uma [função assíncrona](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function), você receberá uma [Promessa](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) para esse trabalho. A promessa contém o estado desse trabalho (_pendente_, _cumprido_, _falha_) e seu eventual resultado resolvido.

Neste exemplo, a função assíncrona <CodeStep step={1}>`fetchData`</CodeStep> retorna uma promessa que está aguardando o `fetch`.

```js [[1, 1, "fetchData()"], [2, 8, "getData()"], [3, 10, "getData()"]]
async function fetchData() {
  return await fetch(`https://...`);
}

const getData = cache(fetchData);

async function MyComponent() {
  getData();
  // ... some computational work  
  await getData();
  // ...
}
```

Ao chamar <CodeStep step={2}>`getData`</CodeStep> pela primeira vez, a promessa retornada de <CodeStep step={1}>`fetchData`</CodeStep> é armazenada em cache. As pesquisas subsequentes retornarão a mesma promessa.

Observe que a primeira chamada <CodeStep step={2}>`getData`</CodeStep> não `await` enquanto a <CodeStep step={3}>segunda</CodeStep> faz. [`await`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await) é um operador JavaScript que esperará e retornará o resultado resolvido da promessa. A primeira chamada <CodeStep step={2}>`getData`</CodeStep> simplesmente inicia o `fetch` para armazenar em cache a promessa para a segunda <CodeStep step={3}>`getData`</CodeStep> que irá pesquisar.

Se, ao chamar <CodeStep step={3}>pela segunda vez</CodeStep>, a promessa ainda estiver _pendente_, então `await` irá pausar aguardando o resultado. A otimização é que, enquanto esperamos pelo `fetch`, o React pode continuar com o trabalho computacional, reduzindo assim o tempo de espera para a <CodeStep step={3}>segunda chamada</CodeStep>.

Se a promessa já estiver resolvida, seja para um erro ou para o resultado _cumprido_, `await` retornará esse valor imediatamente. Em ambos os resultados, há um benefício de desempenho.
</DeepDive>

<Pitfall>

##### Chamar uma função memoizada fora de um componente não usará o cache. {/*pitfall-memoized-call-outside-component*/}

```jsx [[1, 3, "getUser"]]
import {cache} from 'react';

const getUser = cache(async (userId) => {
  return await db.user.query(userId);
});

// 🚩 Erro: Chamada de função memoizada fora do componente não memoizará.
getUser('demo-id');

async function DemoProfile() {
  // ✅ Bom: `getUser` irá memoizar.
  const user = await getUser('demo-id');
  return <Profile user={user} />;
}
```

O React só fornece acesso ao cache para a função memoizada em um componente. Ao chamar <CodeStep step={1}>`getUser`</CodeStep> fora de um componente, ele ainda avaliará a função, mas não lerá ou atualizará o cache.

Isso ocorre porque o acesso ao cache é fornecido por meio de um [contexto](/learn/passing-data-deeply-with-context) que só pode ser acessado a partir de um componente.

</Pitfall>

<DeepDive>

#### Quando devo usar `cache`, [`memo`](/reference/react/memo) ou [`useMemo`](/reference/react/useMemo)? {/*cache-memo-usememo*/}

Todas as APIs mencionadas oferecem memoização, mas a diferença é o que elas se destinam a memoizar, quem pode acessar o cache e quando seu cache é invalidado.

#### `useMemo` {/*deep-dive-use-memo*/}

Em geral, você deve usar [`useMemo`](/reference/react/useMemo) para armazenar em cache um cálculo dispendioso em um Componente de Cliente entre as renderizações. Como exemplo, para memoizar uma transformação de dados dentro de um componente.

```jsx {4}
'use client';

function WeatherReport({record}) {
  const avgTemp = useMemo(() => calculateAvg(record), record);
  // ...
}

function App() {
  const record = getRecord();
  return (
    <>
      <WeatherReport record={record} />
      <WeatherReport record={record} />
    </>
  );
}
```
Neste exemplo, `App` renderiza dois `WeatherReport`s com o mesmo registro. Mesmo que ambos os componentes façam o mesmo trabalho, eles não podem compartilhar trabalho. O cache do `useMemo` é apenas local para o componente.

No entanto, `useMemo` garante que, se `App` rerenderizar e o objeto `record` não mudar, cada instância de componente pularia o trabalho e usaria o valor memoizado de `avgTemp`. `useMemo` apenas armazenará em cache o último cálculo de `avgTemp` com as dependências fornecidas.

#### `cache` {/*deep-dive-cache*/}

Em geral, você deve usar `cache` em Componentes de Servidor para memoizar o trabalho que pode ser compartilhado entre os componentes.

```js [[1, 12, "<WeatherReport city={city} />"], [3, 13, "<WeatherReport city={city} />"], [2, 1, "cache(fetchReport)"]]
const cachedFetchReport = cache(fetchReport);

function WeatherReport({city}) {
  const report = cachedFetchReport(city);
  // ...
}

function App() {
  const city = "Los Angeles";
  return (
    <>
      <WeatherReport city={city} />
      <WeatherReport city={city} />
    </>
  );
}
```
Reescrevendo o exemplo anterior para usar `cache`, neste caso a <CodeStep step={3}>segunda instância de `WeatherReport`</CodeStep> poderá pular o trabalho duplicado e ler do mesmo cache que a <CodeStep step={1}>primeira `WeatherReport`</CodeStep>. Outra diferença em relação ao exemplo anterior é que `cache` também é recomendado para <CodeStep step={2}>memoizar buscas de dados</CodeStep>, ao contrário de `useMemo`, que deve ser usado apenas para cálculos.

No momento, `cache` só deve ser usado em Componentes de Servidor e o cache será invalidado entre as solicitações do servidor.

#### `memo` {/*deep-dive-memo*/}

Você deve usar [`memo`](reference/react/memo) para evitar que um componente seja renderizado novamente se seus props não forem alterados.

```js
'use client';

function WeatherReport({record}) {
  const avgTemp = calculateAvg(record); 
  // ...
}

const MemoWeatherReport = memo(WeatherReport);

function App() {
  const record = getRecord();
  return (
    <>
      <MemoWeatherReport record={record} />
      <MemoWeatherReport record={record} />
    </>
  );
}
```

Neste exemplo, ambos os componentes `MemoWeatherReport` chamarão `calculateAvg` quando renderizados pela primeira vez. No entanto, se `App` for renderizado novamente, sem alterações em `record`, nenhuma das props foi alterada e `MemoWeatherReport` não será renderizado novamente.

Em comparação com `useMemo`, `memo` memoiza a renderização do componente com base em props em comparação com cálculos específicos. Semelhante a `useMemo`, o componente memoizado apenas armazena em cache a última renderização com os últimos valores da prop. Depois que as props mudam, o cache é invalidado e o componente é renderizado novamente.

</DeepDive>

---

## Solução de problemas {/*troubleshooting*/}

### Minha função memoizada ainda é executada, mesmo que eu a tenha chamado com os mesmos argumentos {/*memoized-function-still-runs*/}

Veja as armadilhas mencionadas anteriormente
* [Chamar diferentes funções memoizadas lerá de diferentes caches.](#pitfall-different-memoized-functions)
* [Chamar uma função memoizada fora de um componente não usará o cache.](#pitfall-memoized-call-outside-component)

Se nenhuma das opções acima se aplicar, pode ser um problema com a forma como o React verifica se algo existe em cache.

Se seus argumentos não forem [primitivos](https://developer.mozilla.org/en-US/docs/Glossary/Primitive) (ex. objetos, funções, arrays), certifique-se de passar a mesma referência de objeto.

Ao chamar uma função memoizada, o React procurará nos argumentos de entrada para ver se um resultado já está armazenado em cache. O React usará a igualdade rasa dos argumentos para determinar se há uma ocorrência de cache.

```js
import {cache} from 'react';

const calculateNorm = cache((vector) => {
  // ...
});

function MapMarker(props) {
  // 🚩 Erro: props é um objeto que muda a cada renderização.
  const length = calculateNorm(props);
  // ...
}

function App() {
  return (
    <>
      <MapMarker x={10} y={10} z={10} />
      <MapMarker x={10} y={10} z={10} />
    </>
  );
}
```

Neste caso, os dois `MapMarker`s parecem estar fazendo o mesmo trabalho e chamando `calculateNorm` com o mesmo valor de `{x: 10, y: 10, z:10}`. Mesmo que os objetos contenham os mesmos valores, eles não são a mesma referência de objeto, pois cada componente cria seu próprio objeto `props`.

O React chamará [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) na entrada para verificar se há uma ocorrência de cache.

```js {3,9}
import {cache} from 'react';

const calculateNorm = cache((x, y, z) => {
  // ...
});

function MapMarker(props) {
  // ✅ Bom: Passe primitivos para a função memoizada
  const length = calculateNorm(props.x, props.y, props.z);
  // ...
}

function App() {
  return (
    <>
      <MapMarker x={10} y={10} z={10} />
      <MapMarker x={10} y={10} z={10} />
    </>
  );
}
```

Uma maneira de resolver isso seria passar as dimensões do vetor para `calculateNorm`. Isso funciona porque as próprias dimensões são primitivas.

Outra solução pode ser passar o próprio objeto vetorial como uma prop para o componente. Precisaremos passar o mesmo objeto para ambas as instâncias do componente.

```js {3,9,14}
import {cache} from 'react';

const calculateNorm = cache((vector) => {
  // ...
});

function MapMarker(props) {
  // ✅ Bom: passe o mesmo objeto `vector`
  const length = calculateNorm(props.vector);
  // ...
}

function App() {
  const vector = [10, 10, 10];
  return (
    <>
      <MapMarker vector={vector} />
      <MapMarker vector={vector} />
    </>
  );
}
```
```