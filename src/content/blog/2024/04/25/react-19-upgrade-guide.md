---
title: "Guia de Atualização do React 19"
author: Ricky Hanlon
date: 2024/04/25
description: As melhorias adicionadas ao React 19 exigem algumas alterações que quebram a compatibilidade, mas trabalhamos para tornar a atualização o mais tranquila possível e não esperamos que as mudanças impactem a maioria dos aplicativos. Neste post, guiaremos você pelas etapas para atualizar aplicativos e bibliotecas para o React 19.
---

25 de abril de 2024 por [Ricky Hanlon](https://twitter.com/rickhanlonii)

---


<Intro>

As melhorias adicionadas ao React 19 exigem algumas alterações que quebram a compatibilidade, mas trabalhamos para tornar a atualização o mais tranquila possível e não esperamos que as mudanças impactem a maioria dos aplicativos.

</Intro>

<Note>

#### O React 18.3 também foi publicado {/*react-18-3*/}

Para ajudar a facilitar a atualização para o React 19, publicamos uma versão `react@18.3` que é idêntica à 18.2, mas adiciona avisos para APIs preteridas e outras mudanças necessárias para o React 19.

Recomendamos atualizar para o React 18.3 primeiro para ajudar a identificar quaisquer problemas antes de atualizar para o React 19.

Para uma lista de mudanças na 18.3, veja as [Notas de Lançamento](https://github.com/facebook/react/blob/main/CHANGELOG.md#1830-april-25-2024).

</Note>

Neste post, guiaremos você pelas etapas para atualizar para o React 19:

- [Instalação](#installing)
- [Codemods](#codemods)
- [Alterações que quebram a compatibilidade](#breaking-changes)
- [Novas depreciações](#new-deprecations)
- [Mudanças notáveis](#notable-changes)
- [Mudanças no TypeScript](#typescript-changes)
- [Changelog](#changelog)

Se você gostaria de nos ajudar a testar o React 19, siga as etapas deste guia de atualização e [relate quaisquer problemas](https://github.com/facebook/react/issues/new?assignees=&labels=React+19&projects=&template=19.md&title=%5BReact+19%5D) que encontrar. Para uma lista de novos recursos adicionados ao React 19, veja o [post de lançamento do React 19](/blog/2024/12/05/react-19).

---
## Instalação {/*installing*/}

<Note>

#### O Novo Transform JSX é agora obrigatório {/*new-jsx-transform-is-now-required*/}

Introduzimos um [novo transform JSX](https://legacy.reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html) em 2020 para melhorar o tamanho do bundle e usar JSX sem importar o React. No React 19, adicionamos melhorias adicionais, como usar `ref` como uma prop e melhorias de velocidade do JSX que exigem o novo transform.

Se o novo transform não estiver habilitado, você verá este aviso:

<ConsoleBlockMulti>

<ConsoleLogLine level="error">

Seu aplicativo (ou uma de suas dependências) está usando um transform JSX desatualizado. Atualize para o transform JSX moderno para um desempenho mais rápido: https://react.dev/link/new-jsx-transform

</ConsoleLogLine>

</ConsoleBlockMulti>


Esperamos que a maioria dos aplicativos não seja afetada, pois o transform já está habilitado na maioria dos ambientes. Para instruções manuais sobre como atualizar, consulte o [post de anúncio](https://legacy.reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html).

</Note>


Para instalar a versão mais recente do React e React DOM:

```bash
npm install --save-exact react@^19.0.0 react-dom@^19.0.0
```

Ou, se você estiver usando Yarn:

```bash
yarn add --exact react@^19.0.0 react-dom@^19.0.0
```

Se você estiver usando TypeScript, também precisará atualizar os tipos.
```bash
npm install --save-exact @types/react@^19.0.0 @types/react-dom@^19.0.0
```

Ou, se você estiver usando Yarn:
```bash
yarn add --exact @types/react@^19.0.0 @types/react-dom@^19.0.0
```

Também estamos incluindo um codemod para as substituições mais comuns. Veja [Mudanças no TypeScript](#typescript-changes) abaixo.

## Codemods {/*codemods*/}

Para ajudar na atualização, trabalhamos com a equipe da [codemod.com](https://codemod.com) para publicar codemods que atualizarão automaticamente seu código para muitas das novas APIs e padrões no React 19.

Todos os codemods estão disponíveis no repositório [`react-codemod`](https://github.com/reactjs/react-codemod) e a equipe Codemod se juntou para ajudar a manter os codemods. Para executar esses codemods, recomendamos usar o comando `codemod` em vez de `react-codemod`, pois ele é mais rápido, lida com migrações de código mais complexas e oferece melhor suporte para TypeScript.


<Note>

#### Execute todos os codemods do React 19 {/*run-all-react-19-codemods*/}

Execute todos os codemods listados neste guia com a receita `codemod` do React 19:

```bash
npx codemod@latest react/19/migration-recipe
```

Isso executará os seguintes codemods de `react-codemod`:
- [`replace-reactdom-render`](https://github.com/reactjs/react-codemod?tab=readme-ov-file#replace-reactdom-render)
- [`replace-string-ref`](https://github.com/reactjs/react-codemod?tab=readme-ov-file#replace-string-ref)
- [`replace-act-import`](https://github.com/reactjs/react-codemod?tab=readme-ov-file#replace-act-import)
- [`replace-use-form-state`](https://github.com/reactjs/react-codemod?tab=readme-ov-file#replace-use-form-state)
- [`prop-types-typescript`](https://github.com/reactjs/react-codemod#react-proptypes-to-prop-types)

Isso não inclui as mudanças no TypeScript. Veja [Mudanças no TypeScript](#typescript-changes) abaixo.

</Note>

As mudanças que incluem um codemod são indicadas com o comando abaixo.

Para uma lista de todos os codemods disponíveis, veja o repositório [`react-codemod`](https://github.com/reactjs/react-codemod).

## Alterações que quebram a compatibilidade {/*breaking-changes*/}

### Erros em renderização não são mais relançados {/*errors-in-render-are-not-re-thrown*/}

Em versões anteriores do React, erros lançados durante a renderização eram capturados e relançados. Em modo de desenvolvimento (DEV), também registraríamos no `console.error`, resultando em logs de erro duplicados.

No React 19, [melhoramos como os erros são tratados](/blog/2024/04/25/react-19#error-handling) para reduzir a duplicação, não relançando-os:

- **Erros Não Capturados**: Erros que não são capturados por um Error Boundary são reportados para `window.reportError`.
- **Erros Capturados**: Erros que são capturados por um Error Boundary são reportados para `console.error`.

Essa mudança não deve impactar a maioria dos aplicativos, mas se o seu sistema de relatórios de erros em produção depender do relançamento de erros, você pode precisar atualizar seu tratamento de erros. Para dar suporte a isso, adicionamos novos métodos a `createRoot` e `hydrateRoot` para tratamento de erros personalizado:

```js [[1, 2, "onUncaughtError"], [2, 5, "onCaughtError"]]
const root = createRoot(container, {
  onUncaughtError: (error, errorInfo) => {
    // ... registrar relatório de erro
  },
  onCaughtError: (error, errorInfo) => {
    // ... registrar relatório de erro
  }
});
```

Para mais informações, veja a documentação de [`createRoot`](https://react.dev/reference/react-dom/client/createRoot) e [`hydrateRoot`](https://react.dev/reference/react-dom/client/hydrateRoot).


### APIs preteridas do React removidas {/*removed-deprecated-react-apis*/}

#### Removido: `propTypes` e `defaultProps` para funções {/*removed-proptypes-and-defaultprops*/}
`PropTypes` foram preteridos em [abril de 2017 (v15.5.0)](https://legacy.reactjs.org/blog/2017/04/07/react-v15.5.0.html#new-deprecation-warnings).

No React 19, removemos as verificações de `propTypes` do pacote React, e usá-los será silenciosamente ignorado. Se você estiver usando `propTypes`, recomendamos migrar para TypeScript ou outra solução de verificação de tipos.

Também estamos removendo `defaultProps` de componentes de função em favor de parâmetros padrão do ES6. Componentes de classe continuarão a suportar `defaultProps`, pois não há alternativa ES6.

```js
// Antes
import PropTypes from 'prop-types';

function Heading({text}) {
  return <h1>{text}</h1>;
}
Heading.propTypes = {
  text: PropTypes.string,
};
Heading.defaultProps = {
  text: 'Hello, world!',
};
```
```ts
// Depois
interface Props {
  text?: string;
}
function Heading({text = 'Hello, world!'}: Props) {
  return <h1>{text}</h1>;
}
```

<Note>

Codemod de `propTypes` para TypeScript com:

```bash
npx codemod@latest react/prop-types-typescript
```

</Note>

#### Removido: Contexto Legado usando `contextTypes` e `getChildContext` {/*removed-removing-legacy-context*/}

O Contexto Legado foi preterido em [outubro de 2018 (v16.6.0)](https://legacy.reactjs.org/blog/2018/10/23/react-v-16-6.html).

O Contexto Legado estava disponível apenas em componentes de classe usando as APIs `contextTypes` e `getChildContext`, e foi substituído por `contextType` devido a bugs sutis que eram fáceis de perder. No React 19, removemos o Contexto Legado para tornar o React um pouco menor e mais rápido.

Se você ainda estiver usando Contexto Legado em componentes de classe, precisará migrar para a nova API `contextType`:

```js {5-11,19-21}
// Antes
import PropTypes from 'prop-types';

class Parent extends React.Component {
  static childContextTypes = {
    foo: PropTypes.string.isRequired,
  };

  getChildContext() {
    return { foo: 'bar' };
  }

  render() {
    return <Child />;
  }
}

class Child extends React.Component {
  static contextTypes = {
    foo: PropTypes.string.isRequired,
  };

  render() {
    return <div>{this.context.foo}</div>;
  }
}
```

```js {2,7,9,15}
// Depois
const FooContext = React.createContext();

class Parent extends React.Component {
  render() {
    return (
      <FooContext value='bar'>
        <Child />
      </FooContext>
    );
  }
}

class Child extends React.Component {
  static contextType = FooContext;

  render() {
    return <div>{this.context}</div>;
  }
}
```

#### Removido: refs de string {/*removed-string-refs*/}
Refs de string foram preteridas em [março de 2018 (v16.3.0)](https://legacy.reactjs.org/blog/2018/03/27/update-on-async-rendering.html).

Componentes de classe suportavam refs de string antes de serem substituídos por callbacks de ref devido a [vários desvantagens](https://github.com/facebook/react/issues/1373). No React 19, removemos refs de string para tornar o React mais simples e fácil de entender.

Se você ainda estiver usando refs de string em componentes de classe, precisará migrar para callbacks de ref:

```js {4,8}
// Antes
class MyComponent extends React.Component {
  componentDidMount() {
    this.refs.input.focus();
  }

  render() {
    return <input ref='input' />;
  }
}
```

```js {4,8}
// Depois
class MyComponent extends React.Component {
  componentDidMount() {
    this.input.focus();
  }

  render() {
    return <input ref={input => this.input = input} />;
  }
}
```

<Note>

Codemod de refs de string para callbacks de `ref`:

```bash
npx codemod@latest react/19/replace-string-ref
```

</Note>

#### Removido: fábricas de padrão de módulo {/*removed-module-pattern-factories*/}
Fábricas de padrão de módulo foram preteridas em [agosto de 2019 (v16.9.0)](https://legacy.reactjs.org/blog/2019/08/08/react-v16.9.0.html#deprecating-module-pattern-factories).

Este padrão era raramente usado e suportá-lo faz com que o React seja um pouco maior e mais lento do que o necessário. No React 19, removemos o suporte para fábricas de padrão de módulo, e você precisará migrar para funções regulares:

```js
// Antes
function FactoryComponent() {
  return { render() { return <div />; } }
}
```

```js
// Depois
function FactoryComponent() {
  return <div />;
}
```

#### Removido: `React.createFactory` {/*removed-createfactory*/}
`createFactory` foi preterido em [fevereiro de 2020 (v16.13.0)](https://legacy.reactjs.org/blog/2020/02/26/react-v16.13.0.html#deprecating-createfactory).

Usar `createFactory` era comum antes do amplo suporte a JSX, mas é raramente usado hoje e pode ser substituído por JSX. No React 19, removemos `createFactory` e você precisará migrar para JSX:

```js
// Antes
import { createFactory } from 'react';

const button = createFactory('button');
```

```js
// Depois
const button = <button />;
```

#### Removido: `react-test-renderer/shallow` {/*removed-react-test-renderer-shallow*/}

No React 18, atualizamos `react-test-renderer/shallow` para reexportar [react-shallow-renderer](https://github.com/enzymejs/react-shallow-renderer). No React 19, removemos `react-test-render/shallow` para preferir instalar o pacote diretamente:

```bash
npm install react-shallow-renderer --save-dev
```
```diff
- import ShallowRenderer from 'react-test-renderer/shallow';
+ import ShallowRenderer from 'react-shallow-renderer';
```

<Note>

##### Por favor, reconsidere a renderização superficial {/*please-reconsider-shallow-rendering*/}

A renderização superficial depende de detalhes internos do React e pode impedi-lo de futuras atualizações. Recomendamos migrar seus testes para [@testing-library/react](https://testing-library.com/docs/react-testing-library/intro/) ou [@testing-library/react-native](https://testing-library.com/docs/react-native-testing-library/intro).

</Note>

### APIs preteridas do React DOM removidas {/*removed-deprecated-react-dom-apis*/}

#### Removido: `react-dom/test-utils` {/*removed-react-dom-test-utils*/}

Movemos `act` de `react-dom/test-utils` para o pacote `react`:

<ConsoleBlockMulti>

<ConsoleLogLine level="error">

`ReactDOMTestUtils.act` é preterido em favor de `React.act`. Importe `act` de `react` em vez de `react-dom/test-utils`. Veja https://react.dev/warnings/react-dom-test-utils para mais informações.

</ConsoleLogLine>

</ConsoleBlockMulti>

Para corrigir este aviso, você pode importar `act` de `react`:

```diff
- import {act} from 'react-dom/test-utils'
+ import {act} from 'react';
```

Todas as outras funções de `test-utils` foram removidas. Essas utilidades eram incomuns e tornavam muito fácil depender de detalhes de implementação de baixo nível de