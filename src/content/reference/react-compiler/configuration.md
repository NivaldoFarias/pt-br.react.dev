---
title: Configuração
---

<Intro>

Esta página lista todas as opções de configuração disponíveis no React Compiler.

</Intro>

<Note>

Para a maioria dos aplicativos, as opções padrão devem funcionar imediatamente. Se você tiver uma necessidade especial, pode usar estas opções avançadas.

</Note>

```js
// babel.config.js
module.exports = {
  plugins: [
    [
      'babel-plugin-react-compiler', {
        // compiler options
      }
    ]
  ]
};
```

---

## Controle de Compilação {/*compilation-control*/}

Estas opções controlam *o que* o compilador otimiza e *como* ele seleciona componentes e hooks para compilar.

* [`compilationMode`](/reference/react-compiler/compilationMode) controla a estratégia para selecionar funções a serem compiladas (por exemplo, todas as funções, apenas as anotadas ou a detecção inteligente).

```js
{
  compilationMode: 'annotation' // Only compile "use memo" functions
}
```

---

## Compatibilidade de Versão {/*version-compatibility*/}

A configuração da versão do React garante que o compilador gere código compatível com sua versão do React.

[`target`](/reference/react-compiler/target) especifica qual versão do React você está usando (17, 18 ou 19).

```js
// For React 18 projects
{
  target: '18' // Also requires react-compiler-runtime package
}
```

---

## Tratamento de Erros {/*error-handling*/}

Estas opções controlam como o compilador responde ao código que não segue as [Regras do React](/reference/rules).

[`panicThreshold`](/reference/react-compiler/panicThreshold) determina se a compilação falhará ou se os componentes problemáticos serão ignorados.

```js
// Recomendado para produção
{
  panicThreshold: 'none' // Skip components with errors instead of failing the build
}
```

---

## Depuração {/*debugging*/}

As opções de registro e análise ajudam você a entender o que o compilador está fazendo.

[`logger`](/reference/react-compiler/logger) fornece registro personalizado para eventos de compilação.

```js
{
  logger: {
    logEvent(filename, event) {
      if (event.kind === 'CompileSuccess') {
        console.log('Compiled:', filename);
      }
    }
  }
}
```

---

## Feature Flags {/*feature-flags*/}

A compilação condicional permite que você controle quando o código otimizado é usado.

[`gating`](/reference/react-compiler/gating) habilita feature flags em tempo de execução para testes A/B ou lançamentos graduais.

```js
{
  gating: {
    source: 'my-feature-flags',
    importSpecifierName: 'isCompilerEnabled'
  }
}
```

---

## Padrões de Configuração Comuns {/*common-patterns*/}

### Configuração padrão {/*default-configuration*/}

Para a maioria dos aplicativos React 19, o compilador funciona sem configuração:

```js
// babel.config.js
module.exports = {
  plugins: [
    'babel-plugin-react-compiler'
  ]
};
```

### Projetos React 17/18 {/*react-17-18*/}

Versões mais antigas do React precisam do pacote de tempo de execução e da configuração de destino:

```bash
npm install react-compiler-runtime@rc
```

```js
{
  target: '18' // or '17'
}
```

### Adoção incremental {/*incremental-adoption*/}

Comece com diretórios específicos e expanda gradualmente:

```js
{
  compilationMode: 'annotation' // Only compile "use memo" functions
}
```