---
title: cache
canary: true
---

<RSC>

`cache` é usado apenas com [Componentes React Server](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#react-server-components).

</RSC>

<Intro>

`cache` permite que você faça o cache do resultado de uma busca ou computação de dados.

```js
const cachedFn = cache(fn);
```

</Intro>

<InlineToc />

---

## Referência {/*reference*/}

### `cache(fn)` {/*cache*/}

Chame `cache` fora de qualquer componente para criar uma versão da função com cache.

```js {4,7}
import {cache} from 'react';
import calculateMetrics from 'lib/metrics';

const getMetrics = cache(calculateMetrics);

function Chart({data}) {
  const report = getMetrics(data);
  // ...
}
```

Quando `getMetrics` é chamado pela primeira vez com `data`, `getMetrics` chamará `calculateMetrics(data)` e armazenará o resultado no cache. Se `getMetrics` for chamado novamente com o mesmo `data`, ele retornará o resultado em cache em vez de chamar `calculateMetrics(data)` novamente.

[Veja mais exemplos abaixo.](#usage)

#### Parâmetros {/*parameters*/}

- `fn`: A função para a qual você deseja armazenar resultados em cache. `fn` pode receber quaisquer argumentos e retornar qualquer valor.

#### Retorna {/*returns*/}

`cache` retorna uma versão em cache de `fn` com a mesma assinatura de tipo. Ele não chama `fn` no processo.

Ao chamar `cachedFn` com determinados argumentos, ele primeiro verifica se um resultado em cache existe no cache. Se um resultado em cache existir, ele retorna o resultado. Caso contrário, ele chama `fn` com os argumentos, armazena o resultado no cache e retorna o resultado. A única vez que `fn` é chamado é quando há uma falha no cache.

<Note>

A otimização do armazenamento em cache de valores de retorno com base nas entradas é conhecida como [_memoização_](https://pt.wikipedia.org/wiki/Memoization). Nos referimos à função retornada de `cache` como uma função memorizada.

</Note>

#### Ressalvas {/*caveats*/}

[//]: # 'TODO: add links to Server/Client Component reference once https://github.com/reactjs/react.dev/pull/6177 is merged'

- O React invalidará o cache para todas as funções memorizadas para cada solicitação do servidor.
- Cada chamada para `cache` cria uma nova função. Isso significa que chamar `cache` com a mesma função várias vezes retornará funções memorizadas diferentes que não compartilham o mesmo cache.
- `cachedFn` também armazenará erros em cache. Se `fn` lançar um erro para determinados argumentos, ele será armazenado em cache e o mesmo erro será lançado novamente quando `cachedFn` for chamado com os mesmos argumentos.
- `cache` é para uso em [Componentes Server](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#react-server-components) somente.

---

## Uso {/*usage*/}

### Armazenar em cache uma computação cara {/*cache-expensive-computation*/}

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

Se o mesmo objeto `user` for renderizado em `Profile` e `TeamReport`, os dois componentes podem compartilhar trabalho e chamar `calculateUserMetrics` apenas uma vez para esse `user`.

Suponha que `Profile` seja renderizado primeiro. Ele chamará <CodeStep step={1}>`getUserMetrics`</CodeStep> e verificará se há um resultado em cache. Como é a primeira vez que `getUserMetrics` é chamado com esse `user`, haverá uma falha no cache. `getUserMetrics` chamará então `calculateUserMetrics` com aquele `user` e escreverá o resultado no cache.

Quando `TeamReport` renderizar sua lista de `users` e atingir o mesmo objeto `user`, ele chamará <CodeStep step={2}>`getUserMetrics`</CodeStep> e lerá o resultado do cache.

<Pitfall>

##### Chamar funções memorizadas diferentes lerá de caches diferentes. {/*pitfall-different-memoized-functions*/}

Para acessar o mesmo cache, os componentes devem chamar a mesma função memorizada.

```js [[1, 7, "getWeekReport"], [1, 7, "cache(calculateWeekReport)"], [1, 8, "getWeekReport"]]
// Temperature.js
import {cache} from 'react';
import {calculateWeekReport} from './report';

export function Temperature({cityData}) {
  // 🚩 Errado: Chamar `cache` no componente cria um novo `getWeekReport` para cada renderização
  const getWeekReport = cache(calculateWeekReport);
  const report = getWeekReport(cityData);
  // ...
}
```

```js [[2, 6, "getWeekReport"], [2, 6, "cache(calculateWeekReport)"], [2, 9, "getWeekReport"]]
// Precipitation.js
import {cache} from 'react';
import {calculateWeekReport} from './report';

// 🚩 Errado: `getWeekReport` é acessível apenas para o componente `Precipitation`.
const getWeekReport = cache(calculateWeekReport);

export function Precipitation({cityData}) {
  const report = getWeekReport(cityData);
  // ...
}
```

No exemplo acima, <CodeStep step={2}>`Precipitation`</CodeStep> e <CodeStep step={1}>`Temperature`</CodeStep> chamam `cache` para criar uma nova função memorizada com sua própria pesquisa de cache. Se ambos os componentes renderizarem para o mesmo `cityData`, eles farão trabalho duplicado para chamar `calculateWeekReport`.

Além disso, `Temperature` cria uma <CodeStep step={1}>nova função memorizada</CodeStep> toda vez que o componente é renderizado, o que não permite nenhum compartilhamento de cache.

Para maximizar os acertos do cache e reduzir o trabalho, os dois componentes devem chamar a mesma função memorizada para acessar o mesmo cache. Em vez disso, defina a função memorizada em um módulo dedicado que pode ser [`import`-ado](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import) em todos os componentes.

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
Aqui, ambos os componentes chamam a <CodeStep step={3}>mesma função memorizada</CodeStep> exportada de `./getWeekReport.js` para ler e gravar no mesmo cache.
</Pitfall>

### Compartilhar um snapshot de dados {/*take-and-share-snapshot-of-data*/}

Para compartilhar um snapshot de dados entre componentes, chame `cache` com uma função de busca de dados como `fetch`. Quando vários componentes fazem a mesma busca de dados, apenas uma solicitação é feita e os dados retornados são armazenados em cache e compartilhados entre os componentes. Todos os componentes se referem ao mesmo snapshot de dados durante a renderização do servidor.

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

Se `AnimatedWeatherCard` e `MinimalWeatherCard` renderizarem para a mesma <CodeStep step={1}>cidade</CodeStep>, eles receberão o mesmo snapshot de dados da <CodeStep step={2}>função memorizada</CodeStep>.

Se `AnimatedWeatherCard` e `MinimalWeatherCard` fornecerem argumentos de <CodeStep step={1}>cidade</CodeStep> diferentes para <CodeStep step={2}>`getTemperature`</CodeStep>, então `fetchTemperature` será chamado duas vezes e cada local de chamada receberá dados diferentes.

A <CodeStep step={1}>cidade</CodeStep> atua como uma chave de cache.

<Note>

[//]: # 'TODO: add links to Server Components when merged.'

<CodeStep step={3}>Renderização assíncrona</CodeStep> é suportada apenas para Componentes Server.

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

Ao armazenar em cache uma busca de dados de longa execução, você pode iniciar um trabalho assíncrono antes de renderizar o componente.

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

Ao renderizar `Page`, o componente chama <CodeStep step={1}>`getUser`</CodeStep>, mas observe que ele não usa os dados retornados. Essa chamada inicial <CodeStep step={1}>`getUser`</CodeStep> inicia a consulta assíncrona do banco de dados que ocorre enquanto `Page` está fazendo outro trabalho computacional e renderizando filhos.

Ao renderizar `Profile`, chamamos <CodeStep step={2}>`getUser`</CodeStep> novamente. Se a chamada <CodeStep step={1}>`getUser`</CodeStep> inicial já foi retornada e armazenou em cache os dados do usuário, quando `Profile` <CodeStep step={2}>solicita e espera por esses dados</CodeStep>, ele pode simplesmente ler do cache sem exigir outra chamada de procedimento remoto. Se a <CodeStep step={1}>solicitação de dados inicial</CodeStep> não foi concluída, o pré-carregamentode dados nesse padrão reduz o atraso na busca de dados.

<DeepDive>

#### Armazenamento em cache de trabalho assíncrono {/*caching-asynchronous-work*/}

Ao avaliar uma [função assíncrona](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function), você receberá uma [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) para esse trabalho. A promise mantém o estado desse trabalho (_pendente_, _cumprido_, _falhou_) e seu eventual resultado estabelecido.

Neste exemplo, a função assíncrona <CodeStep step={1}>`fetchData`</CodeStep> retorna uma promise que está aguardando o `fetch`.

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

Ao chamar <CodeStep step={2}>`getData`</CodeStep> pela primeira vez, a promise retornada de <CodeStep step={1}>`fetchData`</CodeStep> é armazenada em cache. Pesquisas subsequentes retornarão então a mesma promise.

Observe que a primeira chamada <CodeStep step={2}>`getData`</CodeStep> não `await` enquanto a <CodeStep step={3}>segunda</CodeStep> sim. [`await`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await) é um operador JavaScript que aguardará e retornará o resultado estabelecido da promise. A primeira chamada <CodeStep step={2}>`getData`</CodeStep> simplesmente inicia o `fetch` para armazenar a promise em cache para a <CodeStep step={3}>segunda</CodeStep> pesquisa de `getData`.

Se na <CodeStep step={3}>segunda chamada</CodeStep> a promise ainda estiver _pendente_, então `await` fará uma pausa pelo resultado. A otimização é que enquanto esperamos pelo `fetch`, o React pode continuar com o trabalho computacional, reduzindo assim o tempo de espera para a <CodeStep step={3}>segunda chamada</CodeStep>.

Se a promise já estiver estabelecida, seja para um erro ou para o resultado _cumprido_, `await` retornará esse valor imediatamente. Em ambos os resultados, há um benefício de desempenho.
</DeepDive>

<Pitfall>

##### Chamar uma função memorizada fora de um componente não usará o cache. {/*pitfall-memoized-call-outside-component*/}

```jsx [[1, 3, "getUser"]]
import {cache} from 'react';

const getUser = cache(async (userId) => {
  return await db.user.query(userId);
});

// 🚩 Errado: Chamar função memorizada fora do componente não memorizará.
getUser('demo-id');

async function DemoProfile() {
  // ✅ Bom: `getUser` memorizará.
  const user = await getUser('demo-id');
  return <Profile user={user} />;
}
```

O React só fornece acesso ao cache para a função memorizada em um componente. Ao chamar <CodeStep step={1}>`getUser`</CodeStep> fora de um componente, ele ainda avaliará a função, mas não lerá nem atualizará o cache.

Isso ocorre porque o acesso ao cache é fornecido por meio de um [contexto](/learn/passing-data-deeply-with-context) que só é acessível de um componente.

</Pitfall>

<DeepDive>

#### Quando devo usar `cache`, [`memo`](/reference/react/memo) ou [`useMemo`](/reference/react/useMemo)? {/*cache-memo-usememo*/}

Todas as APIs mencionadas oferecem memorização, mas a diferença é o que elas se destinam a memorizar, quem pode acessar o cache e quando seu cache é invalidado.

#### `useMemo` {/*deep-dive-use-memo*/}

Em geral, você deve usar [`useMemo`](/reference/react/useMemo) para armazenar em cache uma computação cara em um Componente Cliente entre as renderizações. Como exemplo, para memorizar uma transformação de dados dentro de um componente.

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
Neste exemplo, `App` renderiza dois `WeatherReport`s com o mesmo registro. Mesmo que ambos os componentes façam o mesmo trabalho, eles não podem compartilhar o trabalho. O cache do `useMemo` é apenas local para o componente.

No entanto, `useMemo` garante que, se `App` for renderizado novamente e o objeto `record` não for alterado, cada instância do componente pulará o trabalho e usará o valor memorizado de `avgTemp`. `useMemo` só armazenará em cache a última computação de `avgTemp` com as dependências fornecidas.

#### `cache` {/*deep-dive-cache*/}

Em geral, você deve usar `cache` em Componentes Server para memorizar o trabalho que pode ser compartilhado entre componentes.

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
Reescrevendo o exemplo anterior para usar `cache`, neste caso a <CodeStep step={3}>segunda instância de `WeatherReport`</CodeStep> poderá pular o trabalho duplicado e ler do mesmo cache que a <CodeStep step={1}>primeira `WeatherReport`</CodeStep>. Outra diferença em relação ao exemplo anterior é que `cache` também é recomendado para <CodeStep step={2}>memorizar buscas de dados</CodeStep>, ao contrário de `useMemo`, que só deve ser usado para computações.

No momento, `cache` só deve ser usado em Componentes Server, e o cache será invalidado entre as solicitações do servidor.

#### `memo` {/*deep-dive-memo*/}

Você deve usar [`memo`](reference/react/memo) para evitar que um componente seja renderizado novamente se suas props não forem alteradas.

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

Neste exemplo, ambos os componentes `MemoWeatherReport` chamarão `calculateAvg` quando renderizados pela primeira vez. No entanto, se `App` for renderizado novamente, sem alterações no `record`, nenhuma das props foi alterada e `MemoWeatherReport` não será renderizado novamente.

Em comparação com `useMemo`, `memo` memoriza a renderização do componente com base nas props vs. computações específicas. Semelhante a `useMemo`, o componente memorizado só armazena em cache a última renderização com os últimos valores das props. Assim que as props mudam, o cache é invalidado e o componente é renderizado novamente.

</DeepDive>

---

## Solução de problemas {/*troubleshooting*/}

### Minha função memorizada ainda é executada, embora eu a tenha chamado com os mesmos argumentos {/*memoized-function-still-runs*/}

Veja as armadilhas mencionadas anteriormente
* [Chamar funções memorizadas diferentes lerá de caches diferentes.](#pitfall-different-memoized-functions)
* [Chamar uma função memorizada fora de um componente não usará o cache.](#pitfall-memoized-call-outside-component)

Se nada do que foi dito acima se aplicar, pode ser um problema com a forma como o React verifica se algo existe no cache.

Se seus argumentos não forem [primitivos](https://developer.mozilla.org/en-US/docs/Glossary/Primitive) (por exemplo, objetos, funções, arrays), certifique-se de estar passando a mesma referência de objeto.

Ao chamar uma função memorizada, o React procurará os argumentos de entrada para ver se um resultado já está em cache. O React usará a igualdade superficial dos argumentos para determinar se há um acerto no cache.

```js
import {cache} from 'react';

const calculateNorm = cache((vector) => {
  // ...
});

function MapMarker(props) {
  // 🚩 Errado: props é um objeto que muda a cada renderização.
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

Nesse caso, os dois `MapMarker`s parecem estar fazendo o mesmo trabalho e chamando `calculateNorm` com o mesmo valor de `{x: 10, y: 10, z:10}`. Mesmo que os objetos contenham os mesmos valores, eles não são a mesma referência de objeto, pois cada componente cria seu próprio objeto `props`.

React chamará [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) na entrada para verificar se há um acerto no cache.

```js {3,9}
import {cache} from 'react';

const calculateNorm = cache((x, y, z) => {
  // ...
});

function MapMarker(props) {
  // ✅ Bom: Passar primitivos para a função memorizada
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

Uma forma de resolver isso seria passar as dimensões do vetor para `calculateNorm`. Isso funciona porque as próprias dimensões são primitivas.

Outra solução pode ser passar o próprio objeto vetor como uma prop para o componente. Precisaremos passar o mesmo objeto para ambas as instâncias do componente.

```js {3,9,14}
import {cache} from 'react';

const calculateNorm = cache((vector) => {
  // ...
});

function MapMarker(props) {
  // ✅ Bom: Passar o mesmo objeto `vector`
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