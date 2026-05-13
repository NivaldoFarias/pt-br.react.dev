---
title: Compilando Bibliotecas
---

<Intro>
Este guia ajuda os autores de bibliotecas a entender como usar o React Compiler para fornecer código de biblioteca otimizado para seus usuários.
</Intro>

<InlineToc />

## Por que fornecer código compilado? {/*why-ship-compiled-code*/}

Como autor de uma biblioteca, você pode compilar o código da sua biblioteca antes de publicar no npm. Isso oferece vários benefícios:

- **Melhorias de desempenho para todos os usuários** - Os usuários da sua biblioteca obtêm código otimizado mesmo que ainda não estejam usando o React Compiler
- **Nenhuma configuração exigida pelos usuários** - As otimizações funcionam imediatamente
- **Comportamento consistente** - Todos os usuários obtêm a mesma versão otimizada, independentemente da configuração de compilação

## Configurando a compilação {/*setting-up-compilation*/}

Adicione o React Compiler ao processo de compilação da sua biblioteca:

<TerminalBlock>
npm install -D babel-plugin-react-compiler@rc
</TerminalBlock>

Configure sua ferramenta de compilação para compilar sua biblioteca. Por exemplo, com Babel:

```js
// babel.config.js
module.exports = {
  plugins: [
    'babel-plugin-react-compiler',
  ],
  // ... other config
};
```

## Compatibilidade com versões anteriores {/*backwards-compatibility*/}

Se sua biblioteca suportar versões do React anteriores a 19, você precisará de configuração adicional:

### 1. Instale o pacote de tempo de execução {/*install-runtime-package*/}

Recomendamos instalar o react-compiler-runtime como uma dependência direta:

<TerminalBlock>
npm install react-compiler-runtime@rc
</TerminalBlock>

```json
{
  "dependencies": {
    "react-compiler-runtime": "^19.1.0-rc.2"
  },
  "peerDependencies": {
    "react": "^17.0.0 || ^18.0.0 || ^19.0.0"
  }
}
```

### 2. Configure a versão de destino {/*configure-target-version*/}

Defina a versão mínima do React que sua biblioteca suporta:

```js
{
  target: '17', // Versão mínima do React suportada
}
```

## Estratégia de teste {/*testing-strategy*/}

Teste sua biblioteca com e sem compilação para garantir a compatibilidade. Execute sua suíte de testes existente no código compilado e também crie uma configuração de teste separada que ignore o compilador. Isso ajuda a detectar quaisquer problemas que possam surgir do processo de compilação e garante que sua biblioteca funcione corretamente em todos os cenários.

## Solução de problemas {/*troubleshooting*/}

### Biblioteca não funciona com versões mais antigas do React {/*library-doesnt-work-with-older-react-versions*/}

Se sua biblioteca compilada gerar erros no React 17 ou 18:

1. Verifique se você instalou `react-compiler-runtime` como uma dependência
2. Verifique se sua configuração `target` corresponde à sua versão mínima do React suportada
3. Certifique-se de que o pacote de tempo de execução está incluído no seu pacote publicado

### Conflitos de compilação com outros plugins do Babel {/*compilation-conflicts-with-other-babel-plugins*/}

Alguns plugins do Babel podem entrar em conflito com o React Compiler:

1. Coloque `babel-plugin-react-compiler` no início da sua lista de plugins
2. Desative as otimizações conflitantes em outros plugins
3. Teste exaustivamente a saída da sua compilação

### Módulo de tempo de execução não encontrado {/*runtime-module-not-found*/}

Se os usuários virem "Cannot find module 'react-compiler-runtime'":

1. Certifique-se de que o tempo de execução está listado em `dependencies`, não em `devDependencies`
2. Verifique se seu empacotador inclui o tempo de execução na saída
3. Verifique se o pacote está publicado no npm com sua biblioteca

## Próximos passos {/*next-steps*/}

- Aprenda sobre as [técnicas de depuração](/learn/react-compiler/debugging) para código compilado
- Verifique as [opções de configuração](/reference/react-compiler/configuration) para todas as opções do compilador
- Explore os [modos de compilação](/reference/react-compiler/compilationMode) para otimização seletiva