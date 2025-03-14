---
title: React Compiler
---

<Intro>
Esta página fornecerá uma introdução ao React Compiler e como experimentá-lo com sucesso.
</Intro>

<Wip>
Estes documentos ainda estão em andamento. Mais documentação está disponível no [repositório do Grupo de Trabalho do React Compiler](https://github.com/reactwg/react-compiler/discussions) e será transferida para estes documentos quando estiverem mais estáveis.
</Wip>

<YouWillLearn>

* Começando com o compilador
* Instalando o compilador e o plugin ESLint
* Solução de problemas

</YouWillLearn>

<Note>
O React Compiler é um novo compilador atualmente em versão Beta que disponibilizamos com código aberto para obter feedback inicial da comunidade. Embora tenha sido usado em produção em empresas como a Meta, a implantação do compilador em produção para seu aplicativo dependerá da saúde do seu código base e de como você seguiu as [Regras do React](/reference/rules).

A versão Beta mais recente pode ser encontrada com a tag `@beta` e as versões experimentais diárias com `@experimental`.
</Note>

O React Compiler é um novo compilador que disponibilizamos com código aberto para obter feedback inicial da comunidade. É uma ferramenta somente em tempo de compilação que otimiza automaticamente seu aplicativo React. Ele funciona com JavaScript simples e entende as [Regras do React](/reference/rules), para que você não precise reescrever nenhum código para usá-lo.

O compilador também inclui um [plugin ESLint](#installing-eslint-plugin-react-compiler) que mostra a análise do compilador diretamente no seu editor. **Recomendamos fortemente que todos usem o linter hoje.** O linter não exige que você tenha o compilador instalado, portanto, você pode usá-lo mesmo que não esteja pronto para experimentar o compilador.

O compilador está atualmente lançado como `beta` e está disponível para experimentar em aplicativos e bibliotecas React 17+. Para instalar a versão Beta:

<TerminalBlock>
npm install -D babel-plugin-react-compiler@beta eslint-plugin-react-compiler@beta
</TerminalBlock>

Ou, se você estiver usando Yarn:

<TerminalBlock>
yarn add -D babel-plugin-react-compiler@beta eslint-plugin-react-compiler@beta
</TerminalBlock>

Se você ainda não estiver usando o React 19, consulte [a seção abaixo](#using-react-compiler-with-react-17-or-18) para obter mais instruções.

### O que o compilador faz? {/*what-does-the-compiler-do*/}

Para otimizar os aplicativos, o React Compiler memoriza automaticamente seu código. Você pode estar familiarizado hoje com a memorização por meio de APIs como `useMemo`, `useCallback` e `React.memo`. Com essas APIs, você pode dizer ao React que certas partes do seu aplicativo não precisam ser recalculadas se suas entradas não forem alteradas, reduzindo o trabalho nas atualizações. Embora poderoso, é fácil esquecer de aplicar a memorização ou aplicá-la incorretamente. Isso pode levar a atualizações ineficientes, pois o React precisa verificar partes da sua interface do usuário que não têm alterações _significativas_.

O compilador usa seu conhecimento do JavaScript e das regras do React para memorizar automaticamente valores ou grupos de valores dentro de seus componentes e hooks. Se detectar quebras nas regras, ele ignorará automaticamente apenas esses componentes ou hooks e continuará compilando o restante do código com segurança.

<Note>
O React Compiler pode detectar estaticamente quando as Regras do React são violadas e optar com segurança por não otimizar apenas os componentes ou hooks afetados. Não é necessário que o compilador otimize 100% do seu código base.
</Note>

Se seu código base já estiver muito bem memorizado, você pode não esperar grandes melhorias no desempenho com o compilador. No entanto, na prática, memorizar as dependências corretas que causam problemas de performance é complicado de fazer corretamente manualmente.

<DeepDive>
#### Que tipo de memorização o React Compiler adiciona? {/*what-kind-of-memoization-does-react-compiler-add*/}

A versão inicial do React Compiler se concentra principalmente em **melhorar o desempenho das atualizações** (reativar a renderização de componentes existentes), por isso se concentra nesses dois casos de uso:

1. **Ignorando a re-renderização em cascata de componentes**
    * A re-renderização de `<Parent />` faz com que muitos componentes em sua árvore de componentes sejam re-renderizados, embora apenas `<Parent />` tenha sido alterado
1. **Ignorando cálculos caros de fora do React**
    * Por exemplo, chamar `expensivelyProcessAReallyLargeArrayOfObjects()` dentro do seu componente ou hook que precisa desses dados

#### Otimizando as re-renderizações {/*optimizing-re-renders*/}

React permite que você expresse sua interface do usuário como uma função de seu estado atual (mais concretamente: suas props, state e context). Em sua implementação atual, quando o estado de um componente muda, React re-renderizará esse componente _e todos os seus filhos_ — a menos que você tenha aplicado alguma forma de memorização manual com `useMemo()`, `useCallback()` ou `React.memo()`. Por exemplo, no exemplo a seguir, `<MessageButton>` será re-renderizado sempre que o estado de `<FriendList>` for alterado:

```javascript
function FriendList({ friends }) {
  const onlineCount = useFriendOnlineCount();
  if (friends.length === 0) {
    return <NoFriends />;
  }
  return (
    <div>
      <span>{onlineCount} online</span>
      {friends.map((friend) => (
        <FriendListCard key={friend.id} friend={friend} />
      ))}
      <MessageButton />
    </div>
  );
}
```
[_Veja este exemplo no React Compiler Playground_](https://playground.react.dev/#N4Igzg9grgTgxgUxALhAMygOzgFwJYSYAEAYjHgpgCYAyeYOAFMEWuZVWEQL4CURwADrEicQgyKEANnkwIAwtEw4iAXiJQwCMhWoB5TDLmKsTXgG5hRInjRFGbXZwB0UygHMcACzWr1ABn4hEWsYBBxYYgAeADkIHQ4uAHoAPksRbisiMIiYYkYs6yiqPAA3FMLrIiiwAAcAQ0wU4GlZBSUcbklDNqikusaKkKrgR0TnAFt62sYHdmp+VRT7SqrqhOo6Bnl6mCoiAGsEAE9VUfmqZzwqLrHqM7ubolTVol5eTOGigFkEMDB6u4EAAhKA4HCEZ5DNZ9ErlLIWYTcEDcIA)

O React Compiler aplica automaticamente o equivalente à memorização manual, garantindo que apenas as partes relevantes de um aplicativo sejam re-renderizadas à medida que o estado muda, o que às vezes é chamado de "reatividade granular". No exemplo acima, o React Compiler determina que o valor de retorno de `<FriendListCard />` pode ser reutilizado mesmo quando `friends` muda e pode evitar a recriação desse JSX _e_ evitar a re-renderização de `<MessageButton>` à medida que a contagem muda.

#### Cálculos caros também são memorizados {/*expensive-calculations-also-get-memoized*/}

O compilador também pode memorizar automaticamente cálculos caros usados durante a renderização:

```js
// **Não** memorizado pelo React Compiler, pois esta não é um componente ou hook
function expensivelyProcessAReallyLargeArrayOfObjects() { /* ... */ }

// Memorizado pelo React Compiler, pois este é um componente
function TableContainer({ items }) {
  // Esta chamada de função seria memorizada:
  const data = expensivelyProcessAReallyLargeArrayOfObjects(items);
  // ...
}
```
[_Veja este exemplo no React Compiler Playground_](https://playground.react.dev/#N4Igzg9grgTgxgUxALhAejQAgFTYHIQAuumAtgqRAJYBeCAJpgEYCemASggIZyGYDCEUgAcqAGwQwANJjBUAdokyEAFlTCZ1meUUxdMcIcIjyE8vhBiYVECAGsAOvIBmURYSonMCAB7CzcgBuCGIsAAowEIhgYACCnFxioQAyXDAA5gixMDBcLADyzvlMAFYIvGAAFACUmMCYaNiYAHStOFgAvk5OGJgAshTUdIysHNy8AkbikrIKSqpaWvqGIiZmhE6u7p7ymAAqXEwSguZcCpKV9VSEFBodtcBOmAYmYHz0XIT6ALzefgFUYKhCJRBAxeLcJIsVIZLI5PKFYplCqVa63aoAbm6u0wMAQhFguwAPPRAQA+YAfL4dIloUmBMlODogDpAA)

No entanto, se `expensivelyProcessAReallyLargeArrayOfObjects` for realmente uma função cara, você pode considerar a implementação de sua própria memorização fora do React, porque:

- O React Compiler só memoriza componentes e hooks do React, não todas as funções
- A memorização do React Compiler não é compartilhada entre vários componentes ou hooks

Portanto, se `expensivelyProcessAReallyLargeArrayOfObjects` fosse usado em muitos componentes diferentes, mesmo que os mesmos itens exatos fossem passados, esse cálculo caro seria executado repetidamente. Recomendamos [fazer o perfil](https://react.dev/reference/react/useMemo#how-to-tell-if-a-calculation-is-expensive) primeiro para ver se é realmente tão caro antes de tornar o código mais complicado.
</DeepDive>

### Devo experimentar o compilador? {/*should-i-try-out-the-compiler*/}

Observe que o compilador ainda está em Beta e tem muitas arestas. Embora tenha sido usado em produção em empresas como a Meta, a implantação do compilador em produção para seu aplicativo dependerá da saúde do seu código base e de como você seguiu as [Regras do React](/reference/rules).

**Você não precisa correr para usar o compilador agora. Tudo bem esperar até que ele atinja uma versão estável antes de adotá-lo.** No entanto, agradecemos que você o experimente em pequenos experimentos em seus aplicativos para que você possa [fornecer feedback](#reporting-issues) para nos ajudar a tornar o compilador melhor.

## Começando {/*getting-started*/}

Além destes documentos, recomendamos verificar o [Grupo de Trabalho do React Compiler](https://github.com/reactwg/react-compiler) para obter informações e discussões adicionais sobre o compilador.

### Instalando eslint-plugin-react-compiler {/*installing-eslint-plugin-react-compiler*/}

O React Compiler também alimenta um plugin ESLint. O plugin ESLint pode ser usado **independentemente** do compilador, o que significa que você pode usar o plugin ESLint mesmo que não use o compilador.

<TerminalBlock>
npm install -D eslint-plugin-react-compiler@beta
</TerminalBlock>

Em seguida, adicione-o à sua configuração ESLint:

```js
import reactCompiler from 'eslint-plugin-react-compiler'

export default [
  {
    plugins: {
      'react-compiler': reactCompiler,
    },
    rules: {
      'react-compiler/react-compiler': 'error',
    },
  },
]
```

Ou, no formato de configuração eslintrc obsoleto:

```js
module.exports = {
  plugins: [
    'eslint-plugin-react-compiler',
  ],
  rules: {
    'react-compiler/react-compiler': 'error',
  },
}
```

O plugin ESLint exibirá quaisquer violações das regras do React no seu editor. Quando ele fizer isso, significa que o compilador ignorou a otimização desse componente ou hook. Isso é perfeitamente aceitável, e o compilador pode se recuperar e continuar otimizando outros componentes em seu código base.

<Note>
**Você não precisa corrigir todas as violações do ESLint imediatamente.** Você pode abordá-las no seu próprio ritmo para aumentar a quantidade de componentes e hooks sendo otimizados, mas não é necessário corrigir tudo antes de poder usar o compilador.
</Note>

### Implantando o compilador em seu código base {/*using-the-compiler-effectively*/}

#### Projetos existentes {/*existing-projects*/}
O compilador foi projetado para compilar componentes funcionais e hooks que seguem as [Regras do React](/reference/rules). Ele também pode lidar com o código que quebra essas regras saindo (ignorando) esses componentes ou hooks. No entanto, devido à natureza flexível do JavaScript, o compilador não pode detectar todas as violações possíveis e pode compilar com falsos negativos: ou seja, o compilador pode compilar acidentalmente um componente/hook que quebra as Regras do React, o que pode levar a um comportamento indefinido.

Por esta razão, para adotar o compilador com sucesso em projetos existentes, recomendamos executá-lo primeiro em um pequeno diretório no seu código do produto. Você pode fazer isso configurando o compilador para ser executado apenas em um conjunto específico de diretórios:

```js {3}
const ReactCompilerConfig = {
  sources: (filename) => {
    return filename.indexOf('src/path/to/dir') !== -1;
  },
};
```

Quando você tiver mais confiança em implantar o compilador, você pode expandir a cobertura para outros diretórios também e implantá-lo lentamente em todo o seu aplicativo.

#### Novos projetos {/*new-projects*/}

Se você estiver iniciando um novo projeto, você pode habilitar o compilador em todo o seu código base, que é o comportamento padrão.

### Usando o React Compiler com React 17 ou 18 {/*using-react-compiler-with-react-17-or-18*/}

O React Compiler funciona melhor com o React 19 RC. Se você não conseguir atualizar, pode instalar o pacote extra `react-compiler-runtime`, que permitirá que o código compilado seja executado em versões anteriores à 19. No entanto, observe que a versão mínima suportada é 17.

<TerminalBlock>
npm install react-compiler-runtime@beta
</TerminalBlock>

Você também deve adicionar o `target` correto à sua configuração do compilador, onde o `target` é a versão principal do React que você está direcionando:

```js {3}
// babel.config.js
const ReactCompilerConfig = {
  target: '18' // '17' | '18' | '19'
};

module.exports = function () {
  return {
    plugins: [
      ['babel-plugin-react-compiler', ReactCompilerConfig],
    ],
  };
};
```

### Usando o compilador em bibliotecas {/*using-the-compiler-on-libraries*/}

O React Compiler também pode ser usado para compilar bibliotecas. Como o React Compiler precisa ser executado no código fonte original antes de quaisquer transformações de código, não é possível para o pipeline de construção de um aplicativo compilar as bibliotecas que ele usa. Portanto, nossa recomendação é que os mantenedores da biblioteca compilarem e testarem suas bibliotecas independentemente com o compilador e enviarem o código compilado para o npm.

Como seu código é pré-compilado, os usuários da sua biblioteca não precisarão ter o compilador habilitado para se beneficiarem da memorização automática aplicada à sua biblioteca. Se sua biblioteca direciona aplicativos que ainda não estão no React 19, especifique um [`target` mínimo e adicione `react-compiler-runtime` como uma dependência direta](#using-react-compiler-with-react-17-or-18). O pacote de tempo de execução usará a implementação correta das APIs dependendo da versão do aplicativo e preencherá as APIs ausentes, se necessário.

O código da biblioteca pode frequentemente exigir padrões mais complexos e o uso de saídas de emergência. Por esta razão, recomendamos garantir que você tenha testes suficientes para identificar quaisquer problemas que possam surgir ao usar o compilador em sua biblioteca. Se você identificar algum problema, você sempre pode desativar os componentes ou hooks específicos com a diretiva [`'use no memo'`](#something-is-not-working-after-compilation).

Semelhante aos aplicativos, não é necessário compilar totalmente 100% de seus componentes ou hooks para ver benefícios em sua biblioteca. Um bom ponto de partida pode ser identificar as partes mais sensíveis ao desempenho de sua biblioteca e garantir que elas não quebrem as [Regras do React](/reference/rules), que você pode usar `eslint-plugin-react-compiler` para identificar.

## Uso {/*installation*/}

### Babel {/*usage-with-babel*/}

<TerminalBlock>
npm install babel-plugin-react-compiler@beta
</TerminalBlock>

O compilador inclui um plugin Babel que você pode usar em seu pipeline de construção para executar o compilador.

Após a instalação, adicione-o à sua configuração do Babel. Observe que é fundamental que o compilador seja executado **primeiro** no pipeline:

```js {7}
// babel.config.js
const ReactCompilerConfig = { /* ... */ };

module.exports = function () {
  return {
    plugins: [
      ['babel-plugin-react-compiler', ReactCompilerConfig], // deve ser executado primeiro!
      // ...
    ],
  };
};
```

`babel-plugin-react-compiler` deve ser executado primeiro antes de outros plugins Babel, pois o compilador requer as informações da fonte de entrada para uma análise sólida.

### Vite {/*usage-with-vite*/}

Se você usar o Vite, pode adicionar o plugin ao vite-plugin-react:

```js {10}
// vite.config.js
const ReactCompilerConfig = { /* ... */ };

export default defineConfig(() => {
  return {
    plugins: [
      react({
        babel: {
          plugins: [
            ["babel-plugin-react-compiler", ReactCompilerConfig],
          ],
        },
      }),
    ],
    // ...
  };
});
```

### Next.js {/*usage-with-nextjs*/}

Consulte os [documentos do Next.js](https://nextjs.org/docs/app/api-reference/next-config-js/reactCompiler) para obter mais informações.

### Remix {/*usage-with-remix*/}
Instale `vite-plugin-babel` e adicione o plugin Babel do compilador a ele:

<TerminalBlock>
npm install vite-plugin-babel
</TerminalBlock>

```js {2,14}
// vite.config.js
import babel from "vite-plugin-babel";

const ReactCompilerConfig = { /* ... */ };

export default defineConfig({
  plugins: [
    remix({ /* ... */}),
    babel({
      filter: /\.[jt]sx?$/,
      babelConfig: {
        presets: ["@babel/preset-typescript"], // se você usar TypeScript
        plugins: [
          ["babel-plugin-react-compiler", ReactCompilerConfig],
        ],
      },
    }),
  ],
});
```

### Webpack {/*usage-with-webpack*/}

Um carregador Webpack da comunidade [está agora disponível aqui](https://github.com/SukkaW/react-compiler-webpack).

### Expo {/*usage-with-expo*/}

Consulte os [documentos do Expo](https://docs.expo.dev/guides/react-compiler/) para habilitar e usar o React Compiler em aplicativos Expo.

### Metro (React Native) {/*usage-with-react-native-metro*/}

React Native usa Babel via Metro, portanto, consulte a seção [Uso com Babel](#usage-with-babel) para obter instruções de instalação.

### Rspack {/*usage-with-rspack*/}

Consulte os [documentos da Rspack](https://rspack.dev/guide/tech/react#react-compiler) para habilitar e usar o React Compiler em aplicativos Rspack.

### Rsbuild {/*usage-with-rsbuild*/}

Consulte os [documentos da Rsbuild](https://rsbuild.dev/guide/framework/react#react-compiler) para habilitar e usar o React Compiler em aplicativos Rsbuild.

## Solução de problemas {/*troubleshooting*/}

Para relatar problemas, primeiro crie uma reprodução mínima no [React Compiler Playground](https://playground.react.dev/) e inclua-a em seu relatório de bug. Você pode abrir problemas no repositório [facebook/react](https://github.com/facebook/react/issues).

Você também pode fornecer feedback no Grupo de Trabalho do React Compiler solicitando ser um membro. Consulte [o README para obter mais detalhes sobre como participar](https://github.com/reactwg/react-compiler).

### O que o compilador assume? {/*what-does-the-compiler-assume*/}

O React Compiler assume que seu código:

1. É JavaScript válido e semântico.
2. Testes que valores nulos/opcionais e propriedades são definidos antes de acessá-los (por exemplo, habilitando [`strictNullChecks`](https://www.typescriptlang.org/tsconfig/#strictNullChecks) se estiver usando TypeScript), ou seja, `if (object.nullableProperty) { object.nullableProperty.foo }` ou com encadeamento opcional `object.nullableProperty?.foo`.
3. Segue as [Regras do React](https://react.dev/reference/rules).

O React Compiler pode verificar muitas das Regras do React estaticamente e ignorará com segurança a compilação quando detectar um erro. Para ver os erros, recomendamos também instalar [eslint-plugin-react-compiler](https://www.npmjs.com/package/eslint-plugin-react-compiler).

### Como sei que meus componentes foram otimizados? {/*how-do-i-know-my-components-have-been-optimized*/}

[React DevTools](/learn/react-developer-tools) (v5.0+) e [React Native DevTools](https://reactnative.dev/docs/react-native-devtools) têm suporte integrado para React Compiler e exibirão um selo "Memo ✨" ao lado dos componentes que foram otimizados pelo compilador.

### Algo não está funcionando após a compilação {/*something-is-not-working-after-compilation*/}
Se você tiver eslint-plugin-react-compiler instalado, o compilador exibirá quaisquer violações das regras do React no seu editor. Quando ele fizer isso, significa que o compilador ignorou a otimização desse componente ou hook. Isso é perfeitamente aceitável, e o compilador pode se recuperar e continuar otimizando outros componentes em seu código base. **Você não precisa corrigir todas as violações do ESLint imediatamente.** Você pode abordá-las no seu próprio ritmo para aumentar a quantidade de componentes e hooks sendo otimizados.

Devido à natureza flexível e dinâmica do JavaScript, no entanto, não é possível detectar de forma abrangente todos os casos. Bugs e comportamento indefinido, como loops infinitos, podem ocorrer nesses casos.

Se seu aplicativo não funcionar corretamente após a compilação e você não estiver vendo nenhum erro do ESLint, o compilador pode estar compilando incorretamente seu código. Para confirmar isso, tente fazer o problema desaparecer desativando agressivamente qualquer componente ou hook que você acha que pode estar relacionado por meio da diretiva [`"use no memo"`](#opt-out-of-the-compiler-for-a-component).

```js {2}
function SuspiciousComponent() {
  "use no memo"; // desativa este componente de ser compilado pelo React Compiler
  // ...
}
```

<Note>
#### `"use no memo"` {/*use-no-memo*/}

`"use no memo"` é uma saída de emergência _temporária_ que permite que você desative componentes e hooks de serem compilados pelo React Compiler. Esta diretiva não deve ser de longa duração da mesma forma que, por exemplo, [`"use client"`](/reference/rsc/use-client) é.

Não é recomendado recorrer a esta diretiva, a menos que seja estritamente necessário. Depois de desativar um componente ou hook, ele é desativado para sempre até que a diretiva seja removida. Isso significa que mesmo que você corrija o código, o compilador ainda ignorará a compilação, a menos que você remova a diretiva.
</Note>

Quando você fizer o erro desaparecer, confirme que a remoção da diretiva de desativação faz com que o problema volte. Em seguida, compartilhe um relatório de bug conosco (você pode tentar reduzi-lo para uma pequena reprodução ou, se for código de código aberto, você também pode colar toda a fonte) usando o [React Compiler Playground](https://playground.react.dev) para que possamos identificar e ajudar a corrigir o problema.

### Outros problemas {/*other-issues*/}

Consulte https://github.com/reactwg/react-compiler/discussions/7.
```