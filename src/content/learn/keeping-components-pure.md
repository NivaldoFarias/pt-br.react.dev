---
title: Mantendo os Componentes Puros
---

<Intro>

Algumas funções JavaScript são *puras*. Funções puras executam apenas um cálculo e nada mais. Ao escrever estritamente seus componentes apenas como funções puras, você pode evitar uma classe inteira de erros intrigantes e comportamentos imprevisíveis à medida que seu código base cresce. Para obter esses benefícios, no entanto, existem algumas regras que você deve seguir.

</Intro>

<YouWillLearn>

* O que é pureza e como ela ajuda você a evitar erros
* Como manter os componentes puros, mantendo as alterações fora da fase de renderização
* Como usar o Modo Estrito para encontrar erros em seus componentes

</YouWillLearn>

## Pureza: Componentes como fórmulas {/*purity-components-as-formulas*/}

Em ciência da computação (e especialmente no mundo da programação funcional), [uma função pura](https://wikipedia.org/wiki/Pure_function) é uma função com as seguintes características:

* **Ela cuida de seus próprios assuntos.** Ela não altera nenhum objeto ou variável que existia antes de ser chamada.
* **Mesmas entradas, mesma saída.** Dadas as mesmas entradas, uma função pura deve sempre retornar o mesmo resultado.

Você já pode estar familiarizado com um exemplo de funções puras: fórmulas em matemática.

Considere esta fórmula matemática: <Math><MathI>y</MathI> = 2<MathI>x</MathI></Math>.

Se <Math><MathI>x</MathI> = 2</Math> então <Math><MathI>y</MathI> = 4</Math>. Sempre.

Se <Math><MathI>x</MathI> = 3</Math> então <Math><MathI>y</MathI> = 6</Math>. Sempre.

Se <Math><MathI>x</MathI> = 3</Math>, <MathI>y</MathI> não será às vezes  <Math>9</Math> ou <Math>–1</Math> ou <Math>2.5</Math> dependendo da hora do dia ou do estado do mercado de ações.

Se <Math><MathI>y</MathI> = 2<MathI>x</MathI></Math> e <Math><MathI>x</MathI> = 3</Math>, <MathI>y</MathI> será _sempre_ <Math>6</Math>.

Se transformássemos isso em uma função JavaScript, ela se pareceria com isso:

```js
function double(number) {
  return 2 * number;
}
```

No exemplo acima, `double` é uma **função pura.** Se você passar `3` para ela, ela retornará `6`. Sempre.

o React é projetado em torno desse conceito. **o React assume que cada componente que você escreve é uma função pura.** Isso significa que os Componentes React que você escreve devem sempre retornar o mesmo JSX dadas as mesmas entradas:

<Sandpack>

```js src/App.js
function Recipe({ drinkers }) {
  return (
    <ol>    
      <li>Ferva {drinkers} xícaras de água.</li>
      <li>Adicione {drinkers} colheres de chá de chá e {0.5 * drinkers} colheres de chá de especiarias.</li>
      <li>Adicione {0.5 * drinkers} xícaras de leite para ferver e açúcar a gosto.</li>
    </ol>
  );
}

export default function App() {
  return (
    <section>
      <h1>Receita de Chai com Especiarias</h1>
      <h2>Para dois</h2>
      <Recipe drinkers={2} />
      <h2>Para uma reunião</h2>
      <Recipe drinkers={4} />
    </section>
  );
}
```

</Sandpack>

Quando você passa `drinkers={2}` para `Recipe`, ela retornará JSX contendo `2 xícaras de água`. Sempre.

Se você passar `drinkers={4}`, ela retornará JSX contendo `4 xícaras de água`. Sempre.

Assim como uma fórmula matemática.

Você pode pensar em seus componentes como receitas: se você as seguir e não introduzir novos ingredientes durante o processo de cozimento, obterá o mesmo prato toda vez. Esse "prato" é o JSX que o componente serve ao React para [renderizar.](/learn/render-and-commit)

<Illustration src="/images/docs/illustrations/i_puritea-recipe.png" alt="Uma receita de chá para x pessoas: pegue x xícaras de água, adicione x colheres de chá de chá e 0,5x colheres de chá de especiarias e 0,5x xícaras de leite" />

## Efeitos colaterais: consequências (não)intencionais {/*side-effects-unintended-consequences*/}

O processo de renderização do React deve ser sempre puro. Os componentes devem apenas *retornar* seu JSX e não *alterar* nenhum objeto ou variável que existia antes da renderização—isso os tornaria impuros!

Aqui está um componente que quebra essa regra:

<Sandpack>

```js
let guest = 0;

function Cup() {
  // Ruim: alterando uma variável pré-existente!
  guest = guest + 1;
  return <h2>Caneca de chá para o convidado #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup />
      <Cup />
      <Cup />
    </>
  );
}
```

</Sandpack>

Este componente está lendo e gravando uma variável `guest` declarada fora dele. Isso significa que **chamar este componente várias vezes produzirá diferentes JSX!** E o que é mais, se _outros_ componentes lerem `guest`, eles também produzirão diferentes JSX, dependendo de quando foram renderizados! Isso não é previsível.

Voltando à nossa fórmula <Math><MathI>y</MathI> = 2<MathI>x</MathI></Math>, agora mesmo que <Math><MathI>x</MathI> = 2</Math>, não podemos confiar que <Math><MathI>y</MathI> = 4</Math>. Nossos testes podem falhar, nossos usuários ficariam perplexos, os aviões cairiam do céu - você pode ver como isso levaria a erros confusos!

Você pode corrigir este componente [passando `guest` como uma prop em vez disso](/learn/passing-props-to-a-component):

<Sandpack>

```js
function Cup({ guest }) {
  return <h2>Caneca de chá para o convidado #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup guest={1} />
      <Cup guest={2} />
      <Cup guest={3} />
    </>
  );
}
```

</Sandpack>

Agora seu componente é puro, pois o JSX que ele retorna depende apenas da prop `guest`.

Em geral, você não deve esperar que seus componentes sejam renderizados em nenhuma ordem específica. Não importa se você chama <Math><MathI>y</MathI> = 2<MathI>x</MathI></Math> antes ou depois de <Math><MathI>y</MathI> = 5<MathI>x</MathI></Math>: ambas as fórmulas serão resolvidas independentemente uma da outra. Da mesma forma, cada componente deve apenas "pensar por si mesmo" e não tentar coordenar ou depender de outros durante a renderização. Renderizar é como um exame escolar: cada componente deve calcular JSX por conta própria!

<DeepDive>

#### Detectando cálculos impuros com o Modo Estrito {/*detecting-impure-calculations-with-strict-mode*/}

Embora você possa não tê-los usado todos ainda, no React existem três tipos de entradas que você pode ler durante a renderização: [props](/learn/passing-props-to-a-component), [state](/learn/state-a-components-memory) e [context.](/learn/passing-data-deeply-with-context) Você sempre deve tratar essas entradas como somente leitura.

Quando você deseja *alterar* algo em resposta à entrada do usuário, você deve [definir o estado](/learn/state-a-components-memory) em vez de escrever em uma variável. Você nunca deve alterar variáveis ou objetos pré-existentes enquanto seu componente está renderizando.

o React oferece um "Modo Estrito" no qual ele chama a função de cada componente duas vezes durante o desenvolvimento. **Ao chamar as funções do componente duas vezes, o Modo Estrito ajuda a encontrar componentes que quebram essas regras.**

Observe como o exemplo original exibiu "Convidado #2", "Convidado #4" e "Convidado #6" em vez de "Convidado #1", "Convidado #2" e "Convidado #3". A função original era impura, então chamá-la duas vezes a quebrou. Mas a versão pura corrigida funciona mesmo que a função seja chamada duas vezes sempre. **Funções puras apenas calculam, então chamá-las duas vezes não mudará nada** - assim como chamar `double(2)` duas vezes não altera o que é retornado e resolver <Math><MathI>y</MathI> = 2<MathI>x</MathI></Math> duas vezes não altera o que <MathI>y</MathI> é. Mesmas entradas, mesmas saídas. Sempre.

O Modo Estrito não tem efeito na produção, então não diminuirá a velocidade do aplicativo para seus usuários. Para participar do Modo Estrito, você pode envolver seu componente raiz em `<React.StrictMode>`. Alguns frameworks fazem isso por padrão.

</DeepDive>

### Mutação local: O pequeno segredo do seu componente {/*local-mutation-your-components-little-secret*/}

No exemplo acima, o problema era que o componente alterava uma variável *pré-existente* durante a renderização. Isso geralmente é chamado de **"mutação"** para parecer um pouco mais assustador. Funções puras não mudam variáveis fora do escopo da função ou objetos que foram criados antes da chamada — isso as torna impuras!

No entanto, **é perfeitamente aceitável alterar variáveis e objetos que você *acabou* de criar durante a renderização.** Neste exemplo, você cria um array `[]`, atribui-o a uma variável `cups` e, em seguida, `push` uma dúzia de xícaras nele:

<Sandpack>

```js
function Cup({ guest }) {
  return <h2>Caneca de chá para o convidado #{guest}</h2>;
}

export default function TeaGathering() {
  let cups = [];
  for (let i = 1; i <= 12; i++) {
    cups.push(<Cup key={i} guest={i} />);
  }
  return cups;
}
```

</Sandpack>

Se a variável `cups` ou o array `[]` fossem criados fora da função `TeaGathering`, isso seria um grande problema! Você estaria alterando um objeto *pré-existente* inserindo itens nesse array.

No entanto, tudo bem porque você os criou *durante a mesma renderização*, dentro de `TeaGathering`. Nenhum código fora de `TeaGathering` saberá que isso aconteceu. Isso é chamado de **"mutação local"** — é como o pequeno segredo do seu componente.

## Onde você _pode_ causar efeitos colaterais {/*where-you-_can_-cause-side-effects*/}

Embora a programação funcional se baseie fortemente na pureza, em algum momento, em algum lugar, _algo_ precisa mudar. Esse é o ponto da programação! Essas mudanças — atualizar a tela, iniciar uma animação, alterar os dados — são chamadas de **efeitos colaterais.** São coisas que acontecem _"à parte"_, não durante a renderização.

No React, **efeitos colaterais geralmente pertencem dentro de [manipuladores de eventos.](/learn/responding-to-events)** Manipuladores de eventos são funções que o React executa quando você executa alguma ação — por exemplo, quando você clica em um botão. Mesmo que os manipuladores de eventos sejam definidos *dentro* do seu componente, eles não são executados *durante* a renderização! **Portanto, os manipuladores de eventos não precisam ser puros.**

Se você esgotou todas as outras opções e não consegue encontrar o manipulador de evento certo para seu efeito colateral, ainda pode anexá-lo ao seu JSX retornado com uma chamada [`useEffect`](/apis/useeffect) em seu componente. Isso diz ao React para executá-lo mais tarde, após a renderização, quando os efeitos colaterais são permitidos. **No entanto, essa abordagem deve ser seu último recurso.**

Quando possível, tente expressar sua lógica apenas com renderização. Você ficará surpreso com o quão longe isso pode te levar!

<DeepDive>

#### Por que o React se importa com a pureza? {/*why-does-react-care-about-purity*/}

Escrever funções puras exige um pouco de hábito e disciplina. Mas também desbloqueia oportunidades maravilhosas:

* Seus componentes podem ser executados em um ambiente diferente — por exemplo, no servidor! Como eles retornam o mesmo resultado para as mesmas entradas, um componente pode atender a muitas solicitações de usuários.
* Você pode melhorar o desempenho [ignorando a renderização](/reference/react/memo) de componentes cujas entradas não foram alteradas. Isso é seguro porque as funções puras sempre retornam os mesmos resultados, portanto, é seguro armazená-las em cache.
* Se alguns dados forem alterados no meio da renderização de uma árvore de componentes profundos, o React pode reiniciar a renderização sem perder tempo completando a renderização desatualizada. A pureza torna seguro parar de calcular a qualquer momento.

Cada novo recurso do React que estamos construindo aproveita a pureza. Da busca de dados às animações e ao desempenho, manter os componentes puros desbloqueia o poder do paradigma React.

</DeepDive>

<Recap>

* Um componente deve ser puro, o que significa:
  * **Ele cuida de seus próprios negócios.** Ele não deve alterar nenhum objeto ou variável que existia antes da renderização.
  * **Mesmas entradas, mesma saída.** Dadas as mesmas entradas, um componente deve sempre retornar o mesmo JSX.
* A renderização pode acontecer a qualquer momento, portanto, os componentes não devem depender da sequência de renderização uns dos outros.
* Você não deve alterar nenhuma das entradas que seus componentes usam para renderizar. Isso inclui props, state e context. Para atualizar a tela, ["defina" o state](/learn/state-a-components-memory) em vez de alterar objetos preexistentes.
* Esforce-se para expressar a lógica do seu componente no JSX que você retorna. Quando você precisar "mudar as coisas", geralmente desejará fazê-lo em um manipulador de eventos. Como último recurso, você pode usar `useEffect`.
* Escrever funções puras exige um pouco de prática, mas desbloqueia o poder do paradigma do React.

</Recap>

  
<Challenges>

#### Corrigir um relógio quebrado {/*fix-a-broken-clock*/}

Este componente tenta definir a classe CSS do `<h1>` como `"night"` durante o tempo da meia-noite às seis horas da manhã e `"day"` em todos os outros horários. No entanto, não funciona. Você pode corrigir este componente?

Você pode verificar se sua solução funciona alterando temporariamente o fuso horário do computador. Quando a hora atual estiver entre meia-noite e seis da manhã, o relógio deverá ter cores invertidas!

<Hint>

Renderizar é um *cálculo*, ele não deve tentar "fazer" coisas. Você pode expressar a mesma ideia de forma diferente?

</Hint>

<Sandpack>

```js src/Clock.js active
export default function Clock({ time }) {
  let hours = time.getHours();
  if (hours >= 0 && hours <= 6) {
    document.getElementById('time').className = 'night';
  } else {
    document.getElementById('time').className = 'day';
  }
  return (
    <h1 id="time">
      {time.toLocaleTimeString()}
    </h1>
  );
}
```

```js src/App.js hidden
import { useState, useEffect } from 'react';
import Clock from './Clock.js';

function useTime() {
  const [time, setTime] = useState(() => new Date());
  useEffect(() => {
    const id = setInterval(() => {
      setTime(new Date());
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return time;
}

export default function App() {
  const time = useTime();
  return (
    <Clock time={time} />
  );
}
```

```css
body > * {
  width: 100%;
  height: 100%;
}
.day {
  background: #fff;
  color: #222;
}
.night {
  background: #222;
  color: #fff;
}
```

</Sandpack>

<Solution>

Você pode corrigir este componente calculando a `className` e incluindo-a na saída da renderização:

<Sandpack>

```js src/Clock.js active
export default function Clock({ time }) {
  let hours = time.getHours();
  let className;
  if (hours >= 0 && hours <= 6) {
    className = 'night';
  } else {
    className = 'day';
  }
  return (
    <h1 className={className}>
      {time.toLocaleTimeString()}
    </h1>
  );
}
```

```js src/App.js hidden
import { useState, useEffect } from 'react';
import Clock from './Clock.js';

function useTime() {
  const [time, setTime] = useState(() => new Date());
  useEffect(() => {
    const id = setInterval(() => {
      setTime(new Date());
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return time;
}

export default function App() {
  const time = useTime();
  return (
    <Clock time={time} />
  );
}
```

```css
body > * {
  width: 100%;
  height: 100%;
}
.day {
  background: #fff;
  color: #222;
}
.night {
  background: #222;
  color: #fff;
}
```

</Sandpack>

Neste exemplo, o efeito colateral (modificar o DOM) não era necessário. Você só precisava retornar JSX.

</Solution>

#### Corrigir um perfil quebrado {/*fix-a-broken-profile*/}

Dois componentes `Profile` são renderizados lado a lado com dados diferentes. Pressione "Comprimir" no primeiro perfil e, em seguida, "Expandir". Você notará que ambos os perfis agora mostram a mesma pessoa. Isso é um erro.

Encontre a causa do erro e corrija-o.

<Hint>

O código com erros está em `Profile.js`. Certifique-se de lê-lo de cima a baixo!

</Hint>

<Sandpack>

```js src/Profile.js
import Panel from './Panel.js';
import { getImageUrl } from './utils.js';

let currentPerson;

export default function Profile({ person }) {
  currentPerson = person;
  return (
    <Panel>
      <Header />
      <Avatar />
    </Panel>
  )
}

function Header() {
  return <h1>{currentPerson.name}</h1>;
}

function Avatar() {
  return (
    <img
      className="avatar"
      src={getImageUrl(currentPerson)}
      alt={currentPerson.name}
      width={50}
      height={50}
    />
  );
}
```

```js src/Panel.js hidden
import { useState } from 'react';

export default function Panel({ children }) {
  const [open, setOpen] = useState(true);
  return (
    <section className="panel">
      <button onClick={() => setOpen(!open)}>
        {open ? 'Comprimir' : 'Expandir'}
      </button>
      {open && children}
    </section>
  );
}
```

```js src/App.js
import Profile from './Profile.js';

export default function App() {
  return (
    <>
      <Profile person={{
        imageId: 'lrWQx8l',
        name: 'Subrahmanyan Chandrasekhar',
      }} />
      <Profile person={{
        imageId: 'MK3eW3A',
        name: 'Creola Katherine Johnson',
      }} />
    </>
  )
}
```

```js src/utils.js hidden
export function getImageUrl(person, size = 's') {
  return (
    'https://i.imgur.com/' +
    person.imageId +
    size +
    '.jpg'
  );
}
```

```css
.avatar { margin: 5px; border-radius: 50%; }
.panel {
  border: 1px solid #aaa;
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
  width: 200px;
}
h1 { margin: 5px; font-size: 18px; }
```

</Sandpack>

<Solution>

O problema é que o componente `Profile` grava em uma variável pré-existente chamada `currentPerson`, e os componentes `Header` e `Avatar` leem dela. Isso torna *todos os três* impuros e difíceis de prever.

Para corrigir o erro, remova a variável `currentPerson`. Em vez disso, passe todas as informações de `Profile` para `Header` e `Avatar` via props. Você precisará adicionar uma prop `person` aos dois componentes e passá-la até o final.

<Sandpack>

```js src/Profile.js active
import Panel from './Panel.js';
import { getImageUrl } from './utils.js';

export default function Profile({ person }) {
  return (
    <Panel>
      <Header person={person} />
      <Avatar person={person} />
    </Panel>
  )
}

function Header({ person }) {
  return <h1>{person.name}</h1>;
}

function Avatar({ person }) {
  return (
    <img
      className="avatar"
      src={getImageUrl(person)}
      alt={person.name}
      width={50}
      height={50}
    />
  );
}
```

```js src/Panel.js hidden
import { useState } from 'react';

export default function Panel({ children }) {
  const [open, setOpen] = useState(true);
  return (
    <section className="panel">
      <button onClick={() => setOpen(!open)}>
        {open ? 'Comprimir' : 'Expandir'}
      </button>
      {open && children}
    </section>
  );
}
```

```js src/App.js
import Profile from './Profile.js';

export default function App() {
  return (
    <>
      <Profile person={{
        imageId: 'lrWQx8l',
        name: 'Subrahmanyan Chandrasekhar',
      }} />
      <Profile person={{
        imageId: 'MK3eW3A',
        name: 'Creola Katherine Johnson',
      }} />
    </>
  );
}
```

```js src/utils.js hidden
export function getImageUrl(person, size = 's') {
  return (
    'https://i.imgur.com/' +
    person.imageId +
    size +
    '.jpg'
  );
}
```

```css
.avatar { margin: 5px; border-radius: 50%; }
.panel {
  border: 1px solid #aaa;
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
  width: 200px;
}
h1 { margin: 5px; font-size: 18px; }
```

</Sandpack>

Lembre-se de que o React não garante que as funções do componente serão executadas em nenhuma ordem específica, portanto, você não pode se comunicar entre elas definindo variáveis. Toda a comunicação deve acontecer por meio de props.

</Solution>

#### Corrigir uma bandeja de histórias quebrada {/*fix-a-broken-story-tray*/}

O CEO da sua empresa está pedindo que você adicione "histórias" ao seu aplicativo de relógio online, e você não pode dizer não. Você escreveu um componente `StoryTray` que aceita uma lista de `stories`, seguido por um espaço reservado "Criar História".

Você implementou o espaço reservado "Criar História" inserindo mais uma história falsa ao final do array `stories` que você recebe como prop. Mas, por algum motivo, "Criar História" aparece mais de uma vez. Corrija o problema.

<Sandpack>

```js src/StoryTray.js active
export default function StoryTray({ stories }) {
  stories.push({
    id: 'create',
    label: 'Criar História'
  });

  return (
    <ul>
      {stories.map(story => (
        <li key={story.id}>
          {story.label}
        </li>
      ))}
    </ul>
  );
}
```

```js src/App.js hidden
import { useState, useEffect } from 'react';
import StoryTray from './StoryTray.js';

let initialStories = [
  {id: 0, label: "História do Ankit" },
  {id: 1, label: "História da Taylor" },
];

export default function App() {
  let [stories, setStories] = useState([...initialStories])
  let time = useTime();

  // HACK: Evite que a memória cresça para sempre enquanto você lê a documentação.
  // Estamos quebrando nossas próprias regras aqui.
  if (stories.length > 100) {
    stories.length = 100;
  }

  return (
    <div
      style={{
        width: '100%',
        height: '100%',
        textAlign: 'center',
      }}
    >
      <h2>São {time.toLocaleTimeString()} agora.</h2>
      <StoryTray stories={stories} />
    </div>
  );
}

function useTime() {
  const [time, setTime] = useState(() => new Date());
  useEffect(() => {
    const id = setInterval(() => {
      setTime(new Date());
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return time;
}
```

```css
ul {
  margin: 0;
  list-style-type: none;
}

li {
  border: 1px solid #aaa;
  border-radius: 6px;
  float: left;
  margin: 5px;
  margin-bottom: 20px;
  padding: 5px;
  width: 70px;
  height: 100px;
}
```

```js sandbox.config.json hidden
{
  "hardReloadOnChange": true
}
```

</Sandpack>

<Solution>

Observe como sempre que o relógio é atualizado, "Criar História" é adicionado duas vezes. Isso serve como uma dica de que temos uma mutação durante a renderização - o Modo Estrito chama os componentes duas vezes para tornar esses problemas mais perceptíveis.

A função `StoryTray` não é pura. Ao chamar `push` na matriz recebida `stories` (uma prop!), ela está alterando um objeto que foi criado *antes* do `StoryTray` começar a renderizar. Isso o torna instável e muito difícil de prever.

A solução mais simples é não tocar no array e renderizar "Criar História" separadamente:

<Sandpack>

```js src/StoryTray.js active
export default function StoryTray({ stories }) {
  return (
    <ul>
      {stories.map(story => (
        <li key={story.id}>
          {story.label}
        </li>
      ))}
      <li>Criar História</li>
    </ul>
  );
}
```

```js src/App.js hidden
import { useState, useEffect } from 'react';
import StoryTray from './StoryTray.js';

let initialStories = [
  {id: 0, label: "História do Ankit" },
  {id: 1, label: "História da Taylor" },
];

export default function App() {
  let [stories, setStories] = useState([...initialStories])
  let time = useTime();

  // HACK: Evite que a memória cresça para sempre enquanto você lê a documentação.
  // Estamos quebrando nossas próprias regras aqui.
  if (stories.length > 100) {
    stories.length = 100;
  }

  return (
    <div
      style={{
        width: '100%',
        height: '100%',
        textAlign: 'center',
      }}
    >
      <h2>São {time.toLocaleTimeString()} agora.</h2>
      <StoryTray stories={stories} />
    </div>
  );
}

function useTime() {
  const [time, setTime] = useState(() => new Date());
  useEffect(() => {
    const id = setInterval(() => {
      setTime(new Date());
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return time;
}
```

```css
ul {
  margin: 0;
  list-style-type: none;
}

li {
  border: 1px solid #aaa;
  border-radius: 6px;
  float: left;
  margin: 5px;
  margin-bottom: 20px;
  padding: 5px;
  width: 70px;
  height: 100px;
}
```

</Sandpack>

Alternativamente, você pode criar um array _novo_ (copiando o existente) antes de inserir um item nele:

<Sandpack>

```js src/StoryTray.js active
export default function StoryTray({ stories }) {
  // Copie o array!
  let storiesToDisplay = stories.slice();

  // Não afeta o array original:
  storiesToDisplay.push({
    id: 'create',
    label: 'Criar História'
  });

  return (
    <ul>
      {storiesToDisplay.map(story => (
        <li key={story.id}>
          {story.label}
        </li>
      ))}
    </ul>
  );
}
```

```js src/App.js hidden
import { useState, useEffect } from 'react';
import StoryTray from './StoryTray.js';

let initialStories = [
  {id: 0, label: "História do Ankit" },
  {id: 1, label: "História da Taylor" },
];

export default function App() {
  let [stories, setStories] = useState([...initialStories])
  let time = useTime();

  // HACK: Evite que a memória cresça para sempre enquanto você lê a documentação.
  // Estamos quebrando nossas próprias regras aqui.
  if (stories.length > 100) {
    stories.length = 100;
  }

  return (
    <div
      style={{
        width: '100%',
        height: '100%',
        textAlign: 'center',
      }}
    >
      <h2>São {time.toLocaleTimeString()} agora.</h2>
      <StoryTray stories={stories} />
    </div>
  );
}

function useTime() {
  const [time, setTime] = useState(() => new Date());
  useEffect(() => {
    const id = setInterval(() => {
      setTime(new Date());
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return time;
}
```

```css
ul {
  margin: 0;
  list-style-type: none;
}

li {
  border: 1px solid #aaa;
  border-radius: 6px;
  float: left;
  margin: 5px;
  margin-bottom: 20px;
  padding: 5px;
  width: 70px;
  height: 100px;
}
```

</Sandpack>

Isso mantém sua mutação local e sua função de renderização pura. No entanto, você ainda precisa ter cuidado: por exemplo, se você tentasse alterar algum dos itens existentes do array, teria que clonar esses itens também.

É útil lembrar quais operações em arrays os mutam e quais não os mutam. Por exemplo, `push`, `pop`, `reverse` e `sort` mudarão o array original, mas `slice`, `filter` e `map` criarão um novo.

</Solution>

</Challenges>