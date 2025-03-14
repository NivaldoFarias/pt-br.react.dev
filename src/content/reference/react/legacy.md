---
title: "APIs React Legadas"
---

<Intro>

Estas APIs são exportadas do pacote `react`, mas não são recomendadas para uso em código recém-escrito. Consulte as páginas de API individuais vinculadas para as alternativas sugeridas.

</Intro>

---

## APIs Legadas {/*legacy-apis*/}

* [`Children`](/reference/react/Children) permite manipular e transformar o JSX recebido como a prop `children`. [Veja alternativas.](/reference/react/Children#alternatives)
* [`cloneElement`](/reference/react/cloneElement) permite criar um Elemento React usando outro Elemento como um ponto de partida. [Veja alternativas.](/reference/react/cloneElement#alternatives)
* [`Component`](/reference/react/Component) permite definir um Componente React como uma classe JavaScript. [Veja alternativas.](/reference/react/Component#alternatives)
* [`createElement`](/reference/react/createElement) permite criar um Elemento React. Normalmente, você usará JSX em vez disso.
* [`createRef`](/reference/react/createRef) cria um objeto ref que pode conter um valor arbitrário. [Veja alternativas.](/reference/react/createRef#alternatives)
* [`forwardRef`](/reference/react/forwardRef) permite que seu Componente exponha um nó do DOM ao Componente pai com uma [ref.](/learn/manipulating-the-dom-with-refs)
* [`isValidElement`](/reference/react/isValidElement) verifica se um valor é um Elemento React. Normalmente usado com [`cloneElement`.](/reference/react/cloneElement)
* [`PureComponent`](/reference/react/PureComponent) é semelhante ao [`Component`,](/reference/react/Component) mas pula re-renders com as mesmas props. [Veja alternativas.](/reference/react/PureComponent#alternatives)

---

## APIs Removidas {/*removed-apis*/}

Estas APIs foram removidas no React 19:

* [`createFactory`](https://18.react.dev/reference/react/createFactory): use JSX em vez disso.
* Componentes de Classe: [`static contextTypes`](https://18.react.dev//reference/react/Component#static-contexttypes): use [`static contextType`](#static-contexttype) em vez disso.
* Componentes de Classe: [`static childContextTypes`](https://18.react.dev//reference/react/Component#static-childcontexttypes): use [`static contextType`](#static-contexttype) em vez disso.
* Componentes de Classe: [`static getChildContext`](https://18.react.dev//reference/react/Component#getchildcontext): use [`Context.Provider`](/reference/react/createContext#provider) em vez disso.
* Componentes de Classe: [`static propTypes`](https://18.react.dev//reference/react/Component#static-proptypes): use um sistema de tipo como [TypeScript](https://www.typescriptlang.org/) em vez disso.
* Componentes de Classe: [`this.refs`](https://18.react.dev//reference/react/Component#refs): use [`createRef`](/reference/react/createRef) em vez disso.
```