<Intro>

Normalmente, você passa informações de um componente pai para um componente filho através de props. Mas passar props pode se tornar verboso e inconveniente se você precisar passá-las por muitos componentes no meio, ou se muitos componentes em seu aplicativo precisarem das mesmas informações. O *contexto* permite que o componente pai disponibilize algumas informações para qualquer componente na árvore abaixo dele — não importa quão profunda — sem passá-las explicitamente por props.

</Intro>

<YouWillLearn>

- O que é "prop drilling"
- Como substituir a passagem repetitiva de props por contexto
- Casos de uso comuns para contexto
- Alternativas comuns ao contexto

</YouWillLearn>

## O problema de passar props {/*the-problem-with-passing-props*/}

[Passar props](/learn/passing-props-to-a-component) é uma ótima maneira de direcionar explicitamente dados através da árvore de UI para os componentes que os utilizam.

Mas passar props pode se tornar verboso e inconveniente quando você precisa passar uma prop profundamente através da árvore, ou se muitos componentes precisam da mesma prop. O ancestral comum mais próximo pode estar muito distante dos componentes que precisam dos dados, e [elevar o estado](/learn/sharing-state-between-components) tão alto pode levar a uma situação chamada "prop drilling".

<DiagramGroup>

<Diagram name="passing_data_lifting_state" height={160} width={608} captionPosition="top" alt="Diagrama com uma árvore de três componentes. O pai contém uma bolha representando um valor destacado em roxo. O valor flui para cada um dos dois filhos, ambos destacados em roxo." >

Elevar o estado

</Diagram>
<Diagram name="passing_data_prop_drilling" height={430} width={608} captionPosition="top" alt="Diagrama com uma árvore de dez nós, cada nó com dois filhos ou menos. O nó raiz contém uma bolha representando um valor destacado em roxo. O valor flui para os dois filhos, cada um dos quais passa o valor, mas não o contém. O filho esquerdo passa o valor para dois filhos que estão ambos destacados em roxo. O filho direito da raiz passa o valor para um de seus dois filhos - o da direita, que está destacado em roxo. Esse filho passa o valor para seu único filho, que o passa para ambos os seus dois filhos, que estão destacados em roxo.">

Prop drilling

</Diagram>

</DiagramGroup>

Não seria ótimo se houvesse uma maneira de "teletransportar" dados para os componentes na árvore que os precisam sem passar props? Com o recurso de contexto do React, existe!

## Contexto: uma alternativa para passar props {/*context-an-alternative-to-passing-props*/}

O contexto permite que um componente pai forneça dados para toda a árvore abaixo dele. Existem muitos usos para o contexto. Aqui está um exemplo. Considere este componente `Heading` que aceita um `level` para seu tamanho:

<Sandpack>

```js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading level={1}>Title</Heading>
      <Heading level={2}>Heading</Heading>
      <Heading level={3}>Sub-heading</Heading>
      <Heading level={4}>Sub-sub-heading</Heading>
      <Heading level={5}>Sub-sub-sub-heading</Heading>
      <Heading level={6}>Sub-sub-sub-sub-heading</Heading>
    </Section>
  );
}
```

```js src/Section.js
export default function Section({ children }) {
  return (
    <section className="section">
      {children}
    </section>
  );
}
```

```js src/Heading.js
export default function Heading({ level, children }) {
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Unknown level: ' + level);
  }
}
```

```css
.section {
  padding: 10px;
  margin: 5px;
  border-radius: 5px;
  border: 1px solid #aaa;
}
```

</Sandpack>

Vamos dizer que você quer que vários títulos dentro da mesma `Section` tenham sempre o mesmo tamanho:

<Sandpack>

```js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading level={1}>Title</Heading>
      <Section>
        <Heading level={2}>Heading</Heading>
        <Heading level={2}>Heading</Heading>
        <Heading level={2}>Heading</Heading>
        <Section>
          <Heading level={3}>Sub-heading</Heading>
          <Heading level={3}>Sub-heading</Heading>
          <Heading level={3}>Sub-heading</Heading>
          <Section>
            <Heading level={4}>Sub-sub-heading</Heading>
            <Heading level={4}>Sub-sub-heading</Heading>
            <Heading level={4}>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

```js src/Section.js
export default function Section({ children }) {
  return (
    <section className="section">
      {children}
    </section>
  );
}
```

```js src/Heading.js
export default function Heading({ level, children }) {
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Unknown level: ' + level);
  }
}
```

```css
.section {
  padding: 10px;
  margin: 5px;
  border-radius: 5px;
  border: 1px solid #aaa;
}
```

</Sandpack>

Atualmente, você passa a prop `level` para cada `<Heading>` separadamente:

```js
<Section>
  <Heading level={3}>About</Heading>
  <Heading level={3}>Photos</Heading>
  <Heading level={3}>Videos</Heading>
</Section>
```

Seria bom se você pudesse passar a prop `level` para o componente `<Section>` em vez disso e removê-la do `<Heading>`. Desta forma, você poderia garantir que todos os títulos na mesma seção tenham o mesmo tamanho:

```js
<Section level={3}>
  <Heading>About</Heading>
  <Heading>Photos</Heading>
  <Heading>Videos</Heading>
</Section>
```

Mas como o componente `<Heading>` saberia o nível de sua `<Section>` mais próxima? **Isso exigiria alguma maneira para um filho "perguntar" por dados de algum lugar acima na árvore.**

Você não pode fazer isso apenas com props. É aqui que o contexto entra em jogo. Você fará isso em três etapas:

1. **Crie** um contexto. (Você pode chamá-lo de `LevelContext`, já que é para o nível do título.)
2. **Use** esse contexto do componente que precisa dos dados. (`Heading` usará `LevelContext`.)
3. **Forneça** esse contexto do componente que especifica os dados. (`Section` fornecerá `LevelContext`.)

O contexto permite que um pai — mesmo um distante! — forneça alguns dados para toda a árvore dentro dele.

<DiagramGroup>

<Diagram name="passing_data_context_close" height={160} width={608} captionPosition="top" alt="Diagrama com uma árvore de três componentes. O pai contém uma bolha representando um valor destacado em laranja que se projeta para os dois filhos, cada um destacado em laranja." >

Usando contexto em filhos próximos

</Diagram>

<Diagram name="passing_data_context_far" height={430} width={608} captionPosition="top" alt="Diagrama com uma árvore de dez nós, cada nó com dois filhos ou menos. O nó pai raiz contém uma bolha representando um valor destacado em laranja. O valor se projeta diretamente para quatro folhas e um componente intermediário na árvore, que estão todos destacados em laranja. Nenhum dos outros componentes intermediários está destacado.">

Usando contexto em filhos distantes

</Diagram>

</DiagramGroup>

### Etapa 1: Crie o contexto {/*step-1-create-the-context*/}

Primeiro, você precisa criar o contexto. Você precisará **exportá-lo de um arquivo** para que seus componentes possam usá-lo:

<Sandpack>

```js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading level={1}>Title</Heading>
      <Section>
        <Heading level={2}>Heading</Heading>
        <Heading level={2}>Heading</Heading>
        <Heading level={2}>Heading</Heading>
        <Section>
          <Heading level={3}>Sub-heading</Heading>
          <Heading level={3}>Sub-heading</Heading>
          <Heading level={3}>Sub-heading</Heading>
          <Section>
            <Heading level={4}>Sub-sub-heading</Heading>
            <Heading level={4}>Sub-sub-heading</Heading>
            <Heading level={4}>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

```js src/Section.js
export default function Section({ children }) {
  return (
    <section className="section">
      {children}
    </section>
  );
}
```

```js src/Heading.js
export default function Heading({ level, children }) {
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Unknown level: ' + level);
  }
}
```

```js src/LevelContext.js active
import { createContext } from 'react';

export const LevelContext = createContext(1);
```

```css
.section {
  padding: 10px;
  margin: 5px;
  border-radius: 5px;
  border: 1px solid #aaa;
}
```

</Sandpack>

O único argumento para `createContext` é o valor _padrão_. Aqui, `1` refere-se ao maior nível de título, mas você pode passar qualquer tipo de valor (até mesmo um objeto). Você verá a importância do valor padrão na próxima etapa.

### Etapa 2: Use o contexto {/*step-2-use-the-context*/}

Importe o Hook `useContext` do React e seu contexto:

```js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';
```

Atualmente, o componente `Heading` lê `level` das props:

```js
export default function Heading({ level, children }) {
  // ...
}
```

Em vez disso, remova a prop `level` e leia o valor do contexto que você acabou de importar, `LevelContext`:

```js {2}
export default function Heading({ children }) {
  const level = useContext(LevelContext);
  // ...
}
```

`useContext` é um Hook. Assim como `useState` e `useReducer`, você só pode chamar um Hook imediatamente dentro de um componente React (não dentro de loops ou condições). **`useContext` diz ao React que o componente `Heading` deseja ler o `LevelContext`.**

Agora que o componente `Heading` não tem mais a prop `level`, você não precisa mais passar a prop `level` para `Heading` em seu JSX assim:

```js
<Section>
  <Heading level={4}>Sub-sub-heading</Heading>
  <Heading level={4}>Sub-sub-heading</Heading>
  <Heading level={4}>Sub-sub-heading</Heading>
</Section>
```

Atualize o JSX para que seja a `Section` que o recebe em vez disso:

```jsx
<Section level={4}>
  <Heading>Sub-sub-heading</Heading>
  <Heading>Sub-sub-heading</Heading>
  <Heading>Sub-sub-heading</Heading>
</Section>
```

Como um lembrete, esta é a marcação que você estava tentando fazer funcionar:

<Sandpack>

```js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section level={1}>
      <Heading>Title</Heading>
      <Section level={2}>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Section level={3}>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Section level={4}>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

```js src/Section.js
export default function Section({ children }) {
  return (
    <section className="section">
      {children}
    </section>
  );
}
```

```js src/Heading.js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Heading({ children }) {
  const level = useContext(LevelContext);
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Unknown level: ' + level);
  }
}
```

```js src/LevelContext.js
import { createContext } from 'react';

export const LevelContext = createContext(1);
```

```css
.section {
  padding: 10px;
  margin: 5px;
  border-radius: 5px;
  border: 1px solid #aaa;
}
```

</Sandpack>

Note que este exemplo ainda não funciona completamente! Todos os títulos têm o mesmo tamanho porque **mesmo que você esteja *usando* o contexto, você ainda não o *forneceu*.** O React não sabe de onde obtê-lo!

Se você não fornecer o contexto, o React usará o valor padrão que você especificou na etapa anterior. Neste exemplo, você especificou `1` como argumento para `createContext`, então `useContext(LevelContext)` retorna `1`, definindo todos esses títulos como `<h1>`. Vamos corrigir esse problema fazendo com que cada `Section` forneça seu próprio contexto.

### Etapa 3: Forneça o contexto {/*step-3-provide-the-context*/}

O componente `Section` atualmente renderiza seus filhos:

```js
export default function Section({ children }) {
  return (
    <section className="section">
      {children}
    </section>
  );
}
```

**Envolva-os com um provedor de contexto** para fornecer o `LevelContext` a eles:

```js {1,6,8}
import { LevelContext } from './LevelContext.js';

export default function Section({ level, children }) {
  return (
    <section className="section">
      <LevelContext value={level}>
        {children}
      </LevelContext>
    </section>
  );
}
```

Isso diz ao React: "se algum componente dentro desta `<Section>` pedir por `LevelContext`, dê a eles este `level`." O componente usará o valor do `<LevelContext>` mais próximo na árvore de UI acima dele.

<Sandpack>

```js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section level={1}>
      <Heading>Title</Heading>
      <Section level={2}>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Section level={3}>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Section level