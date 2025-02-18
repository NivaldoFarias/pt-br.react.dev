```
---
title: "O Plano para o React 18"
author: Andrew Clark, Brian Vaughn, Christine Abernathy, Dan Abramov, Rachel Nabors, Rick Hanlon, Sebastian Markbage, and Seth Webster
date: 2021/06/08
description: A equipe do React está animada para compartilhar algumas atualizações. Começamos a trabalhar no lançamento do React 18, que será nossa próxima versão principal. Criamos um Grupo de Trabalho para preparar a comunidade para a adoção gradual de novos recursos no React 18. Publicamos um React 18 Alpha para que os autores de bibliotecas possam testá-lo e fornecer feedback...
---

8 de junho de 2021 por [Andrew Clark](https://twitter.com/acdlite), [Brian Vaughn](https://github.com/bvaughn), [Christine Abernathy](https://twitter.com/abernathyca), [Dan Abramov](https://bsky.app/profile/danabra.mov), [Rachel Nabors](https://twitter.com/rachelnabors), [Rick Hanlon](https://twitter.com/rickhanlonii), [Sebastian Markbåge](https://twitter.com/sebmarkbage), and [Seth Webster](https://twitter.com/sethwebster)

---

<Intro>

A equipe do React está animada para compartilhar algumas atualizações:

1. Começamos a trabalhar no lançamento do React 18, que será nossa próxima versão principal.
2. Criamos um Grupo de Trabalho para preparar a comunidade para a adoção gradual de novos recursos no React 18.
3. Publicamos um React 18 Alpha para que os autores de bibliotecas possam testá-lo e fornecer feedback.

Estas atualizações são direcionadas principalmente aos mantenedores de bibliotecas de terceiros. Se você estiver aprendendo, ensinando ou usando o React para criar aplicações voltadas para o usuário, você pode ignorar esta postagem com segurança. Mas você pode acompanhar as discussões no Grupo de Trabalho do React 18 se estiver curioso!

---

</Intro>

## O que vem por aí no React 18 {/*whats-coming-in-react-18*/}

Quando for lançado, o React 18 incluirá melhorias prontas para uso (como [agrupamento automático](https://github.com/reactwg/react-18/discussions/21)), novas APIs (como [`startTransition`](https://github.com/reactwg/react-18/discussions/41)) e um [novo renderizador de servidor de streaming](https://github.com/reactwg/react-18/discussions/37) com suporte integrado para `React.lazy`.

Esses recursos são possíveis graças a um novo mecanismo de ativação que estamos adicionando no React 18. Ele é chamado de "renderização concorrente" e permite que o React prepare várias versões da UI ao mesmo tempo. Essa mudança está principalmente nos bastidores, mas ela desbloqueia novas possibilidades para melhorar tanto o desempenho real quanto o percebido do seu app.

Se você esteve acompanhando nossa pesquisa sobre o futuro do React (não esperamos que você faça isso!), pode ter ouvido falar de algo chamado "modo concorrente" ou que isso pode quebrar seu app. Em resposta a este feedback da comunidade, redesenhamos a estratégia de atualização para adoção gradual. Em vez de um "modo" de tudo ou nada, a renderização concorrente só será ativada para atualizações acionadas por um dos novos recursos. Na prática, isso significa que **você poderá adotar o React 18 sem reescritas e experimentar os novos recursos no seu próprio ritmo.**

## Uma estratégia de adoção gradual {/*a-gradual-adoption-strategy*/}

Como a concorrência no React 18 é opcional, não há mudanças significativas de comportamento dos componentes prontas para uso. **Você pode atualizar para o React 18 com o mínimo ou nenhuma alteração no código da sua aplicação, com um nível de esforço comparável a um lançamento principal típico do React**. Com base em nossa experiência na conversão de vários apps para React 18, esperamos que muitos usuários possam atualizar em uma única tarde.

Lançamos com sucesso recursos concorrentes para dezenas de milhares de componentes no Facebook e, em nossa experiência, descobrimos que a maioria dos componentes React "simplesmente funciona" sem alterações adicionais. Estamos comprometidos em garantir que esta seja uma atualização tranquila para toda a comunidade, então hoje estamos anunciando o Grupo de Trabalho do React 18.

## Trabalhando com a comunidade {/*working-with-the-community*/}

Estamos tentando algo novo para este lançamento: Convidamos um painel de especialistas, desenvolvedores, autores de bibliotecas e educadores de toda a comunidade React a participar do nosso [Grupo de Trabalho do React 18](https://github.com/reactwg/react-18) para fornecer feedback, fazer perguntas e colaborar no lançamento. Não pudemos convidar todos que queríamos para este pequeno grupo inicial, mas se este experimento funcionar, esperamos que haja mais no futuro!

**O objetivo do Grupo de Trabalho do React 18 é preparar o ecossistema para uma adoção suave e gradual do React 18 por aplicações e bibliotecas existentes.** O Grupo de Trabalho é hospedado no [GitHub Discussions](https://github.com/reactwg/react-18/discussions) e está disponível para o público ler. Os membros do grupo de trabalho podem deixar feedback, fazer perguntas e compartilhar ideias. A equipe principal também usará o repo de discussões para compartilhar nossas descobertas de pesquisa. À medida que a versão estável se aproxima, qualquer informação importante também será postada neste blog.

Para obter mais informações sobre como atualizar para o React 18 ou recursos adicionais sobre o lançamento, consulte a [postagem de anúncio do React 18](https://github.com/reactwg/react-18/discussions/4).

## Acessando o Grupo de Trabalho do React 18 {/*accessing-the-react-18-working-group*/}

Todos podem ler as discussões no [repo do Grupo de Trabalho do React 18](https://github.com/reactwg/react-18).

Como esperamos um aumento inicial de interesse no Grupo de Trabalho, apenas membros convidados poderão criar ou comentar tópicos. No entanto, os tópicos são totalmente visíveis ao público, então todos têm acesso às mesmas informações. Acreditamos que este é um bom compromisso entre a criação de um ambiente produtivo para os membros do grupo de trabalho, ao mesmo tempo em que mantém a transparência com a comunidade mais ampla.

Como sempre, você pode enviar relatórios de erros (bug reports), perguntas e feedback geral para nosso [rastreador de problemas](https://github.com/facebook/react/issues).

## Como experimentar o React 18 Alpha hoje {/*how-to-try-react-18-alpha-today*/}

Novos alphas são [publicados regularmente no npm usando a tag `@alpha`](https://github.com/reactwg/react-18/discussions/9). Essas versões são construídas usando o commit mais recente em nosso repositório principal. Quando um recurso ou correção de erro é mesclado, ele aparecerá em um alpha no dia de semana seguinte.

Pode haver mudanças significativas de comportamento ou API entre as versões alpha. Lembre-se de que **as versões alpha não são recomendadas para aplicações de produção voltadas para o usuário**.

## Cronograma de lançamento projetado do React 18 {/*projected-react-18-release-timeline*/}

Não temos uma data de lançamento específica agendada, mas esperamos que leve vários meses de feedback e iteração antes que o React 18 esteja pronto para a maioria das aplicações de produção.

* Alpha da biblioteca: Disponível hoje
* Beta pública: Pelo menos vários meses
* Release Candidate (RC): Pelo menos várias semanas após a Beta
* Disponibilidade geral: Pelo menos várias semanas após RC

Mais detalhes sobre nossa linha do tempo de lançamento projetada estão [disponíveis no Grupo de Trabalho](https://github.com/reactwg/react-18/discussions/9). Postaremos atualizações neste blog quando estivermos mais próximos de um lançamento público.
```