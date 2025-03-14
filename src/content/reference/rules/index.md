---
title: Regras do React
---

<Intro>
Assim como diferentes linguagens de programação têm suas próprias maneiras de expressar conceitos, o React tem seus próprios idiomas — ou regras — para como expressar padrões de uma maneira que seja fácil de entender e produza aplicações de alta qualidade.
</Intro>

<InlineToc />

---

<Note>
Para saber mais sobre como expressar UIs com o React, recomendamos ler [Pensando em React](/learn/thinking-in-react).
</Note>

Esta seção descreve as regras que você precisa seguir para escrever um código React idiomático. Escrever um código React idiomático pode ajudá-lo a escrever aplicações bem organizadas, seguras e compostas. Essas propriedades tornam seu aplicativo mais resistente a mudanças e facilita o trabalho com outros desenvolvedores, bibliotecas e ferramentas.

Estas regras são conhecidas como as **Regras do React**. Elas são regras – e não apenas diretrizes – no sentido de que, se forem quebradas, seu aplicativo provavelmente terá erros. Seu código também se torna não idiomático e mais difícil de entender e raciocinar.

Recomendamos fortemente o uso do [Strict Mode](/reference/react/StrictMode) junto com o [plugin ESLint](https://www.npmjs.com/package/eslint-plugin-react-hooks) do React para ajudar seu codebase a seguir as Regras do React. Ao seguir as Regras do React, você poderá encontrar e solucionar esses erros e manter seu aplicativo.

---

## Componentes e Hooks devem ser puros {/*components-and-hooks-must-be-pure*/}

[Pureza em Componentes e Hooks](/reference/rules/components-and-hooks-must-be-pure) é uma regra chave do React que torna seu aplicativo previsível, fácil de depurar e permite que o React otimize automaticamente seu código.

*   [Componentes devem ser idempotentes](/reference/rules/components-and-hooks-must-be-pure#components-and-hooks-must-be-idempotent) – Componentes React são assumidos para sempre retornar a mesma saída com relação a suas entradas – props, state e context.
*   [Side effects devem ser executados fora do render](/reference/rules/components-and-hooks-must-be-pure#side-effects-must-run-outside-of-render) – Side effects não devem ser executados no render, pois o React pode renderizar componentes várias vezes para criar a melhor experiência possível para o usuário.
*   [Props e state são imutáveis](/reference/rules/components-and-hooks-must-be-pure#props-and-state-are-immutable) – As props e o state de um componente são snapshots imutáveis com relação a um único render. Nunca os mute diretamente.
*   [Valores de retorno e argumentos para Hooks são imutáveis](/reference/rules/components-and-hooks-must-be-pure#return-values-and-arguments-to-hooks-are-immutable) – Depois que os valores são passados para um Hook, você não deve modificá-los. Como props em JSX, os valores se tornam imutáveis quando passados para um Hook.
*   [Valores são imutáveis depois de serem passados para JSX](/reference/rules/components-and-hooks-must-be-pure#values-are-immutable-after-being-passed-to-jsx) – Não mute valores depois de serem usados em JSX. Mova a mutação antes da criação do JSX.

---

## React chama Componentes e Hooks {/*react-calls-components-and-hooks*/}

[O React é responsável por renderizar componentes e hooks quando necessário para otimizar a experiência do usuário.](/reference/rules/react-calls-components-and-hooks) É declarativo: você diz ao React o que renderizar na lógica do seu componente, e o React descobrirá a melhor forma de exibi-lo para seu usuário.

*   [Nunca chame funções de componentes diretamente](/reference/rules/react-calls-components-and-hooks#never-call-component-functions-directly) – Componentes devem ser usados apenas em JSX. Não os chame como funções regulares.
*   [Nunca passe hooks como valores regulares](/reference/rules/react-calls-components-and-hooks#never-pass-around-hooks-as-regular-values) – Os Hooks devem ser chamados apenas dentro de componentes. Nunca os passe como um valor regular.

---

## Regras dos Hooks {/*rules-of-hooks*/}

Hooks são definidos usando funções JavaScript, mas eles representam um tipo especial de lógica de UI reutilizável com restrições sobre onde eles podem ser chamados. Você precisa seguir as [Regras dos Hooks](/reference/rules/rules-of-hooks) ao usá-los.

*   [Chame Hooks somente no nível superior](/reference/rules/rules-of-hooks#only-call-hooks-at-the-top-level) – Não chame Hooks dentro de loops, condições ou funções aninhadas. Em vez disso, sempre use Hooks no nível superior da sua função React, antes de quaisquer retornos antecipados.
*   [Chame Hooks somente de funções React](/reference/rules/rules-of-hooks#only-call-hooks-from-react-functions) – Não chame Hooks de funções JavaScript regulares.
```