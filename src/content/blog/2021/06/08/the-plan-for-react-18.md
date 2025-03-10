---
title: "O Plano para React 18"
author: Andrew Clark, Brian Vaughn, Christine Abernathy, Dan Abramov, Rachel Nabors, Rick Hanlon, Sebastian Markbage, and Seth Webster
date: 2021/06/08
description: A equipe do React está entusiasmada em compartilhar algumas atualizações. Começamos a trabalhar no lançamento do React 18, que será nossa próxima versão principal. Criamos um Grupo de Trabalho para preparar a comunidade para a adoção gradual de novos recursos no React 18. Publicamos um React 18 Alpha para que os autores de bibliotecas possam experimentá-lo e dar feedback...
---

8 de Junho de 2021 por [Andrew Clark](https://twitter.com/acdlite), [Brian Vaughn](https://github.com/bvaughn), [Christine Abernathy](https://twitter.com/abernathyca), [Dan Abramov](https://bsky.app/profile/danabra.mov), [Rachel Nabors](https://twitter.com/rachelnabors), [Rick Hanlon](https://twitter.com/rickhanlonii), [Sebastian Markbåge](https://twitter.com/sebmarkbage), and [Seth Webster](https://twitter.com/sethwebster)

---

<Intro>

A equipe do React está entusiasmada em compartilhar algumas atualizações:

1. Começamos a trabalhar no lançamento do React 18, que será nossa próxima versão principal.
2. Criamos um Grupo de Trabalho para preparar a comunidade para a adoção gradual de novos recursos no React 18.
3. Publicamos um React 18 Alpha para que os autores de bibliotecas possam experimentá-lo e dar feedback.

Estas atualizações são direcionadas principalmente para os mantenedores de bibliotecas de terceiros. Se você está aprendendo, ensinando ou usando o React para construir aplicações voltadas para o usuário, você pode ignorar este post com segurança. Mas você é bem-vindo para seguir as discussões no React 18 Working Group se estiver curioso!

---

</Intro>

## O que vem no React 18 {/*whats-coming-in-react-18*/}

Quando for lançado, o React 18 incluirá melhorias prontas para uso (como [automatic batching](https://github.com/reactwg/react-18/discussions/21)), novas APIs (como [`startTransition`](https://github.com/reactwg/react-18/discussions/41)), e um [novo streaming server renderer](https://github.com/reactwg/react-18/discussions/37) com suporte embutido para `React.lazy`.

Esses recursos são possíveis graças a um novo mecanismo de aceitação que estamos adicionando no React 18. É chamado de "renderização concorrente" e permite que o React prepare múltiplas versões da UI ao mesmo tempo. Essa mudança é principalmente nos bastidores, mas ela libera novas possibilidades para melhorar o desempenho real e percebido do seu app.

Se você tem acompanhado nossa pesquisa sobre o futuro do React (não esperamos que você acompanhe!), pode ter ouvido falar de algo chamado “concurrent mode” ou que isso pode quebrar seu app. Em resposta a este feedback da comunidade, redesenhamos a estratégia de atualização para adoção gradual. Em vez de um "modo" de tudo ou nada, a renderização concorrente só será habilitada para atualizações acionadas por um dos novos recursos. Na prática, isso significa que **você poderá adotar o React 18 sem reescritas e experimentar os novos recursos no seu próprio ritmo.**

## Uma estratégia de adoção gradual {/*a-gradual-adoption-strategy*/}

Como a concorrência no React 18 é opcional, não há mudanças significativas e *prontas para uso* que quebrem o comportamento do componente. **Você pode atualizar para o React 18 com um mínimo ou nenhuma mudança no código da sua aplicação, com um nível de esforço comparável a um lançamento principal típico do React**. Com base em nossa experiência convertendo vários apps para o React 18, esperamos que muitos usuários possam atualizar em uma única tarde.

Nós enviamos com sucesso recursos concorrentes para dezenas de milhares de componentes no Facebook e, em nossa experiência, descobrimos que a maioria dos componentes React "simplesmente funciona" sem alterações adicionais. Estamos comprometidos em garantir que essa seja uma atualização tranquila para toda a comunidade, então hoje estamos anunciando o React 18 Working Group.

## Trabalhando com a comunidade {/*working-with-the-community*/}

Estamos tentando algo novo para este lançamento: Convidamos um painel de especialistas, desenvolvedores, autores de bibliotecas e educadores de toda a comunidade React para participar do nosso [React 18 Working Group](https://github.com/reactwg/react-18) para fornecer feedback, fazer perguntas e colaborar no lançamento. Não pudemos convidar todos que queríamos para este pequeno grupo inicial, mas se esse experimento funcionar, esperamos que haja mais no futuro!

**O objetivo do React 18 Working Group é preparar o ecossistema para uma adoção suave e gradual do React 18 por aplicações e bibliotecas existentes.** O Working Group é hospedado em [GitHub Discussions](https://github.com/reactwg/react-18/discussions) e está disponível para leitura pública. Os membros do grupo de trabalho podem deixar feedback, fazer perguntas e compartilhar ideias. A equipe principal também usará o repositório de discussões para compartilhar nossas descobertas de pesquisa. À medida que o lançamento estável se aproxima, qualquer informação importante também será postada neste blog.

Para obter mais informações sobre a atualização para o React 18 ou recursos adicionais sobre o lançamento, consulte a [postagem de anúncio do React 18](https://github.com/reactwg/react-18/discussions/4).

## Acessando o React 18 Working Group {/*accessing-the-react-18-working-group*/}

Todos podem ler as discussões no [repositório do React 18 Working Group](https://github.com/reactwg/react-18).

Como esperamos um interesse inicial no Working Group, somente os membros convidados poderão criar ou comentar em tópicos. No entanto, os tópicos são totalmente visíveis ao público, para que todos tenham acesso às mesmas informações. Acreditamos que este é um bom compromisso entre a criação de um ambiente produtivo para os membros do grupo de trabalho e a manutenção da transparência com a comunidade em geral.

Como sempre, você pode enviar relatórios de erros, perguntas e feedback geral para nosso [rastreador de problemas](https://github.com/facebook/react/issues).

## Como experimentar o React 18 Alpha hoje {/*how-to-try-react-18-alpha-today*/}

Novos alphas são [regularmente publicados no npm usando a tag `@alpha`](https://github.com/reactwg/react-18/discussions/9). Esses lançamentos são construídos usando o commit mais recente em nosso repositório principal. Quando um recurso ou correção de bug é mesclado, ele aparecerá em um alpha no dia útil seguinte.

Pode haver mudanças significativas de comportamento ou de API entre as versões alfa. Por favor, lembre-se que **as versões alfa não são recomendadas para aplicações de produção voltadas para o usuário**.

## Cronograma previsto de lançamento do React 18 {/*projected-react-18-release-timeline*/}

Não temos uma data de lançamento específica agendada, mas esperamos que leve vários meses de feedback e iteração antes que o React 18 esteja pronto para a maioria das aplicações de produção.

* Alpha da biblioteca: Disponível hoje
* Beta pública: Pelo menos vários meses
* Release Candidate (RC): Pelo menos várias semanas após a versão Beta
* Disponibilidade geral: Pelo menos várias semanas após a versão RC

Mais detalhes sobre nosso cronograma de lançamento projetado estão [disponíveis no Working Group](https://github.com/reactwg/react-18/discussions/9). Postaremos atualizações neste blog quando estivermos mais próximos de um lançamento público.