---
title: "O Plano para React 18"
author: Andrew Clark, Brian Vaughn, Christine Abernathy, Dan Abramov, Rachel Nabors, Rick Hanlon, Sebastian Markbage, and Seth Webster
date: 2021/06/08
description: The React team is excited to share a few updates. We’ve started work on the React 18 release, which will be our next major version. We’ve created a Working Group to prepare the community for gradual adoption of new features in React 18. We’ve published a React 18 Alpha so that library authors can try it and provide feedback...
---

8 de Junho de 2021 por [Andrew Clark](https://twitter.com/acdlite), [Brian Vaughn](https://github.com/bvaughn), [Christine Abernathy](https://twitter.com/abernathyca), [Dan Abramov](https://bsky.app/profile/danabra.mov), [Rachel Nabors](https://twitter.com/rachelnabors), [Rick Hanlon](https://twitter.com/rickhanlonii), [Sebastian Markbåge](https://twitter.com/sebmarkbage), e [Seth Webster](https://twitter.com/sethwebster)

---

<Intro>

A equipe React está animada para compartilhar algumas atualizações:

1. Iniciamos o trabalho na versão React 18, que será nossa próxima versão principal.
2. Criamos um Grupo de Trabalho para preparar a comunidade para a adoção gradual de novos recursos no React 18.
3. Publicamos um React 18 Alpha para que os autores de bibliotecas possam experimentá-lo e fornecer feedback.

Essas atualizações são direcionadas principalmente aos mantenedores de bibliotecas de terceiros. Se você está aprendendo, ensinando ou usando o React para criar aplicativos voltados para o usuário, pode ignorar esta publicação com segurança. Mas você é bem-vindo para acompanhar as discussões no React 18 Working Group, se estiver curioso!

---

</Intro>

## O que vem no React 18 {/*whats-coming-in-react-18*/}

Quando for lançado, o React 18 incluirá melhorias prontas para uso (como [agrupamento automático](https://github.com/reactwg/react-18/discussions/21)), novas APIs (como [`startTransition`](https://github.com/reactwg/react-18/discussions/41)) e um [novo renderizador de servidor de streaming](https://github.com/reactwg/react-18/discussions/37) com suporte integrado para `React.lazy`.

Esses recursos são possíveis graças a um novo mecanismo opcional que estamos adicionando no React 18. Ele é chamado de "renderização concorrente" e permite que o React prepare várias versões da UI ao mesmo tempo. Essa mudança é principalmente nos bastidores, mas ela desbloqueia novas possibilidades para melhorar o desempenho real e percebido do seu aplicativo.

Se você está acompanhando nossa pesquisa sobre o futuro do React (não esperamos que você acompanhe!), pode ter ouvido falar de algo chamado "modo concorrente" ou que ele pode quebrar seu aplicativo. Em resposta a esse feedback da comunidade, redesenhamos a estratégia de atualização para adoção gradual. Em vez de um "modo" de tudo ou nada, a renderização concorrente será ativada apenas para atualizações acionadas por um dos novos recursos. Na prática, isso significa que **você poderá adotar o React 18 sem reescritas e experimentar os novos recursos no seu próprio ritmo.**

## Uma estratégia de adoção gradual {/*a-gradual-adoption-strategy*/}

Como a concorrência no React 18 é opcional, não há alterações significativas de quebra do comportamento do componente prontas para uso. **Você pode atualizar para o React 18 com alterações mínimas ou nenhuma alteração no código do seu aplicativo, com um nível de esforço comparável a uma versão principal típica do React**. Com base em nossa experiência na conversão de vários aplicativos para React 18, esperamos que muitos usuários consigam atualizar em uma única tarde.

Entregamos com sucesso recursos concorrentes para dezenas de milhares de componentes no Facebook e, em nossa experiência, descobrimos que a maioria dos componentes React "simplesmente funciona" sem alterações adicionais. Estamos comprometidos em garantir que esta seja uma atualização tranquila para toda a comunidade, então hoje estamos anunciando o React 18 Working Group.

## Trabalhando com a comunidade {/*working-with-the-community*/}

Estamos tentando algo novo para esta versão: Convidamos um painel de especialistas, desenvolvedores, autores de bibliotecas e educadores de toda a comunidade React para participar do nosso [React 18 Working Group](https://github.com/reactwg/react-18) para fornecer feedback, fazer perguntas e colaborar no lançamento. Não pudemos convidar todos que queríamos para este grupo inicial e pequeno, mas se este experimento funcionar, esperamos que haja mais no futuro!

**O objetivo do React 18 Working Group é preparar o ecossistema para uma adoção tranquila e gradual do React 18 por aplicativos e bibliotecas existentes.** O Grupo de Trabalho é hospedado no [GitHub Discussions](https://github.com/reactwg/react-18/discussions) e está disponível para o público ler. Os membros do grupo de trabalho podem deixar feedback, fazer perguntas e compartilhar ideias. A equipe central também usará o repositório de discussões para compartilhar nossas conclusões da pesquisa. Conforme a versão estável se aproximar, qualquer informação importante também será postada neste blog.

Para obter mais informações sobre como atualizar para o React 18, ou recursos adicionais sobre o lançamento, consulte a [postagem de anúncio do React 18](https://github.com/reactwg/react-18/discussions/4).

## Acessando o React 18 Working Group {/*accessing-the-react-18-working-group*/}

Todos podem ler as discussões no [repositório do React 18 Working Group](https://github.com/reactwg/react-18).

Como esperamos uma onda inicial de interesse no Grupo de Trabalho, apenas os membros convidados poderão criar ou comentar em tópicos. No entanto, os tópicos são totalmente visíveis ao público, então todos têm acesso à mesma informação. Acreditamos que este é um bom compromisso entre a criação de um ambiente produtivo para os membros do grupo de trabalho, mantendo a transparência com a comunidade em geral.

Como sempre, você pode enviar relatórios de erros, perguntas e feedback geral para nosso [rastreador de problemas](https://github.com/facebook/react/issues).

## Como experimentar o React 18 Alpha hoje mesmo {/*how-to-try-react-18-alpha-today*/}

Novos alphas são [publicados regularmente no npm usando a tag `@alpha`](https://github.com/reactwg/react-18/discussions/9). Essas versões são criadas usando o commit mais recente em nosso repositório principal. Quando um recurso ou correção de bug é mesclado, ele aparecerá em um alpha no dia útil seguinte.

Pode haver alterações significativas de comportamento ou API entre as versões alfa. Por favor, lembre-se que **versões alfa não são recomendadas para aplicativos de produção voltados para o usuário**.

## Cronograma de lançamento projetado do React 18 {/*projected-react-18-release-timeline*/}

Não temos uma data de lançamento específica agendada, mas esperamos que leve vários meses de feedback e iteração antes que o React 18 esteja pronto para a maioria dos aplicativos de produção.

* Alpha da Biblioteca: Disponível hoje
* Beta Pública: Pelo menos vários meses
* Candidato a lançamento (RC): Pelo menos várias semanas após a versão Beta
* Disponibilidade geral: Pelo menos várias semanas após o RC

Mais detalhes sobre nosso cronograma de lançamento projetado estão [disponíveis no Grupo de Trabalho](https://github.com/reactwg/react-18/discussions/9). Publicaremos atualizações neste blog quando estivermos mais próximos de um lançamento público.