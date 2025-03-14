---
title: "APIs React Legadas"
---

<Intro>

Estas APIs são exportadas do pacote `react`, mas não são recomendadas para uso em código recém-escrito. Veja as páginas individuais da API vinculadas para as alternativas sugeridas.

</Intro>

---

## APIs Legadas {/*legacy-apis*/}

* [`Children`](/reference/react/Children) permite que você manipule e transforme o JSX recebido como a `children` prop. [Veja alternativas.](/reference/react/Children#alternatives)
* [`cloneElement`](/reference/react/cloneElement) permite criar um elemento React usando outro elemento como ponto de partida. [Veja alternativas.](/reference/react/cloneElement#alternatives)
* [`Component`](/reference/react/Component) permite que você defina um componente React como uma classe JavaScript. [Veja alternativas.](/reference/react/Component#alternatives)
* [`createElement`](/reference/react/createElement) permite criar um elemento React. Normalmente, você usará JSX em vez disso.
* [`createRef`](/reference/react/createRef) cria um objeto ref que pode conter um valor arbitrário. [Veja alternativas.](/reference/react/createRef#alternatives)
* [`forwardRef`](/reference/react/forwardRef) permite que seu componente exponha um nó do DOM ao componente pai com uma [ref.](/learn/manipulating-the-dom-with-refs)
* [`isValidElement`](/reference/react/isValidElement) verifica se um valor é um elemento React. Normalmente usado com [`cloneElement`.](/reference/react/cloneElement)
* [`PureComponent`](/reference/react/PureComponent) é similar a [`Component`,](/reference/react/Component) mas ele pula re-renders com as mesmas props. [Veja alternativas.](/reference/react/PureComponent#alternatives)

---

## APIs Removidas {/*removed-apis*/}

Estas APIs foram removidas no React 19:

* [`createFactory`](https://18.react.dev/reference/react/createFactory): use JSX em vez disso.
* Class Components: [`static contextTypes`](https://18.react.dev//reference/react/Component#static-contexttypes): use [`static contextType`](#static-contexttype) em vez disso.
* Class Components: [`static childContextTypes`](https://18.react.dev//reference/react/Component#static-childcontexttypes): use [`static contextType`](#static-contexttype) em vez disso.
* Class Components: [`static getChildContext`](https://18.react.dev//reference/react/Component#getchildcontext): use [`Context.Provider`](/reference/react/createContext#provider) em vez disso.
* Class Components: [`static propTypes`](https://18.react.dev//reference/react/Component#static-proptypes): use a type system como [TypeScript](https://www.typescriptlang.org/) em vez disso.
* Class Components: [`this.refs`](https://18.react.dev//reference/react/Component#refs): use [`createRef`](/reference/react/createRef) em vez disso.