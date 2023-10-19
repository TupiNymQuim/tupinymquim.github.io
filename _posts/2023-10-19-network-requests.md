---
permalink: /network-requests/
---

[link da postagem original](https://blog.nymtech.net/tech-deepdive-network-requesters-e5359a6cc31c)

Apresentando o primeiro Provedor de Serviços Nym.

![Imagem da postagem](https://miro.medium.com/v2/resize:fit:720/0*aWIyMCX_7C2WEufP/)

A Nym é uma infraestrutura de privacidade aberta e descentralizada. Essa infraestrutura é operada por pessoas de todo o mundo e é composta por diferentes tipos de nós, cada um desempenhando funções ligeiramente diferentes. Em resumo, ao utilizar a infraestrutura para a privacidade, o tráfego de suas comunicações primeiro é criptografado por um cliente Nym em seu dispositivo e depois enviado para:

- [Gateway](https://blog.nymtech.net/gateways-to-privacy-51196005bf5), que são o primeiro salto na mixnet;
- [Mixnodes](https://blog.nymtech.net/choose-your-character-an-overview-of-nym-network-actors-19e6a9808540) que mesclam o tráfego das pessoas através de três conjuntos de nós, tornando-o não rastreável;

E, finalmente, os Network Requesters são o salto de saída que leva o usuário do gateway de saída para um aplicativo - que eles agora estão usando em um modo aprimorado de privacidade!

[Postagens anteriores no blog](https://blog.nymtech.net/choose-your-character-an-overview-of-nym-network-actors-19e6a9808540) abordaram os Gateways, os Mixnodes e os Validadores. Esta postagem é uma análise profunda dos [Network Requesters](https://nymtech.net/operators/nodes/network-requester-setup.html), que são o primeiro tipo de Provedor de Serviços Nym, conectando a infraestrutura central de privacidade a aplicativos voltados para os usuários.

Antes de aprofundarmos nos Network Requesters, vamos dar uma olhada mais ampla no papel dos Provedores de Serviços em geral - dos quais os Network Requesters são apenas um tipo...

OBS.: A Nym em breve estará emitindo concessões para iniciar e operar os Network Requesters. Se você deseja estar entre os primeiros a fornecer acesso em massa à privacidade em nível de rede, inscreva-se [aqui](http://eepurl.com/h6uPSD) e fique atento...


## Provedores de Serviços (Service Providers)

O espaço de design para Provedores de Serviços (SPs) é muito amplo: alguns permitirão o acesso a serviços externos na internet, protegendo o usuário dos olhos curiosos de coleta de dados e observadores habilitados para aprendizado de máquina; outros SPs permitirão o acesso a serviços protegidos que só podem ser acessados através da mixnet.

O desenvolvimento de SPs não envolve repensar como você projeta seu aplicativo desde o início (em termos de código, de qualquer forma). Você pode pensar em SPs apenas como "aplicativos que podem se comunicar com um cliente Nym local". Isso significa que você pode gastar mais tempo explorando o crescente espaço de design de aplicativos "privados por design" e menos tempo em integração!

É provável que as pessoas que executam SPs também executem seu próprio gateway para seu aplicativo, para garantir confiabilidade e tempo de atividade. Dito isso, um pequeno SP poderia facilmente depender dos gateways públicos disponíveis, se desejassem.

#### Na internet, ninguém precisa saber que você está usando a mixnet...

Você pode estar se perguntando como os Provedores de Serviços (SPs) protegem a privacidade. Os SPs permitem que as pessoas usem até mesmo aplicativos que não são tão privados de maneira preservadora de privacidade. Eles podem facilitar requisições anonimizadas para serviços web públicos fora da rede Nym para aplicativos como Keybase ou Telegram, servidores de e-mail ou até mesmo blockchains.

Os SPs fazem isso da mesma forma que um serviço de proxy padrão: eles recebem sua solicitação da mixnet e a encaminham para o endpoint específico em outro lugar. O destinatário na clearnet - como um servidor web - não precisa (e não deveria) sequer estar ciente da Nym. Para eles, o SP é apenas mais um aplicativo que envia uma solicitação web. Os metadados expostos pela solicitação serão apenas os do SP. Como resultado, a origem da solicitação - o usuário da Nym - será protegida, e qualquer observador monitorando o tráfego na clearnet não saberá mais do que se estivesse interagindo com o servidor a partir de seus metadados.

Os SPs que não se comunicam fora da rede Nym podem oferecer serviços semelhantes à gama padrão de soluções de aplicativos ou servidores disponíveis na internet, mas com aprimoramento de privacidade e existindo exclusivamente dentro da rede Nym. Algo como backups de dados seguros e inrastreáveis é um exemplo imediato: pense em um Dropbox anônimo ou serviços de hospedagem de arquivos.


### Apresentando o Nym Network Requester

Já temos um Provedor de Serviços em nosso [repositório no GitHub](https://github.com/nymtech/nym/tree/develop/service-providers/network-requester) e a [documentação para você tentar executar](https://nymtech.net/operators/nodes/network-requester-setup.html) o Network Requester (NR). O NR é um executável que pode ser executado ao lado de um cliente Nym em um VPS, permitindo a realização de requisições de rede privadas fora da mixnet a partir do seu cliente Nym local.

Por que você usaria isso? Quando você está usando seu aplicativo de desktop, ele se comunica com os servidores de backend do aplicativo ou faz solicitações web externas. O Network Requester fica do "outro lado" da mixnet fazendo essas solicitações em seu nome, ocultando assim seus metadados.

Por exemplo: você pode usar o cliente Nym SOCKS5 para conectar seu aplicativo de desktop do Telegram aos servidores do Telegram para coletar e enviar mensagens. Isso significa que os servidores do Telegram veem uma solicitação do Network Requester em vez do seu computador local. O Network Requester age de forma semelhante a um proxy, roteando solicitações para você e enviando de volta as respostas que obtém dos servidores de mensagens do Telegram.

Como isso funciona? É um trecho de código que verifica uma solicitação de entrada do cliente Nym, verifica o domínio que está sendo solicitado e se esse domínio está em uma lista branca (whitelist) e, se estiver, encaminha a solicitação e depois passa a resposta de volta ao cliente solicitante (ou seja, você).

Do ponto de vista do usuário, tudo isso acontece nos bastidores. Tudo o que você precisa fazer é apontar seu cliente SOCKS5 local para um NR que suporta o domínio que deseja acessar e, em seguida, direcionar seu aplicativo local para o cliente SOCKS5!

[Aprenda como executar um Network Requester](https://nymtech.net/docs/stable/run-nym-nodes/nodes/requester](https://nymtech.net/operators/nodes/network-requester-setup.html/)

[Explore a plataforma de privacidade Nym.](https://nymtech.net/docs))

[Junte-se ao Discord da Nym.](https://discord.com/invite/nym)

## A privacidade adora companhia.

[Discord](https://discord.com/invite/nym) **_//_** [Telegram](https://t.me/nymtech) **_//_** [Element](https://matrix.to/#/%23dev:nymtech.chat) **_//_** [Twitter](https://twitter.com/nymproject)

## A internet é global, e a Nym também: junte-se à Comunidade Nym onde quer que esteja e ajude a construir a internet privada hoje.

[English](https://t.me/nymchan) // [中文](https://t.me/nymchina) // [Русский](https://t.me/NYM_Russian) // [Türkçe](https://t.me/NYM_turkey) // [Tiếng Việt](https://t.me/nymtechvn) // [日本](https://t.me/nymjapanese) // [Française](https://t.me/nymfrench) // [Español](https://t.me/NYMSPANISH) // [Português](https://t.me/nymportuguese) // [한국인](https://t.me/nymkorean)
