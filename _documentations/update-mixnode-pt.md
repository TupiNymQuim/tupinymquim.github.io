---
permalink: /update-mixnode-pt/
layout: home
author_profile: false
---

# Atualize seu mixnode do jeito certo

> Por supermeia da TupiNymQuim

Considere nos dar suporte por meio de stake em algum dos seguintes mixnodes:
* [TupiNymQuim-nodes](https://tupinymquim.github.io/nodes/)
* [supermeia-node](https://explorer.nymtech.net/network-components/mixnode/927)

## Sumário

1. [Contexto](#contexto)
    1. [Objetivo](#objetivo) 
    2. [Porque o routing score cai](#porque-o-routing-score-cai)
    3. [Solução por alto](#solução-por-alto)
2. [Tutorial](#tutorial)
    1. [Setup pela primeira vez](#setup-pela-primeira-vez)
        1. [Clonar mixnode](#clonar-mixnode)
        2. [Abrir portas alternativas](#abrir-portas-alternativas)
        3. [Armazenando binários](#armazenando-binários)
        4. [Arquivos de serviço](#arquivos-de-serviços)
    2. [Atualizando](#atualizando)
        1. [Armazenando o binario atualizado](#armazenando-o-binário-atualizado)
        2. [Inicializando o mixnode](#inicializando-o-mixnode)
        3. [Iniciar e habilitar o serviço](#iniciar-e-habilitar-o-serviço)
        4. [Atualizar informação na wallet](#atualizar-informação-na-wallet)
        5. [Desligar serviço desatualizado](#desligar-serviço-desatualizado)
3. [Conclusão](#conclusão)

## Contexto

Se estiver aqui apenas para saber o passo a passo de como atualizar sem perder
o routing score, pode passar diretamente para o
[tutorial](#tutorial).


### Objetivo

A atualização do mixnode é um processo muito simples, mas tem um efeito
secundário indesejável de baixar o routing score do seu node durante um
período de tempo. O objetivo deste guia é te dar o **passo a passo** de como começar
a atualizar o seu mixnode com um método **que lhe permite não perder o seu
precioso routing score**

### Porque o routing score cai

O motivo por trás da queda do routing score não está confirmado (pelo menos que
eu saiba), mas depois de alguma experiência na execução de mixnodes e na
observação de seu comportamento. Ficou claro para mim que a queda do routing 
score está diretamente relacionada à reinicialização do processo que executa o
mixnode. Isso significa que, se o processo do seu mixnode em execução for
reiniciado por qualquer motivo (desligamento do servidor, reinicialização do
servidor, interrupção manual etc.), o routing do seu mixnode começará a cair
automaticamente depois disso.

### Solução por alto

P: Então, como é possível atualizar o mixnode sem perder o routing score? 

R: Simplesmente não interrompendo o processo do mixnode!

Em resumo, o que faremos para evitar a interrupção do processo do mixnode é:

* Criar um clone do nosso mixnode no mesmo VPS
    * Isso também poderia ser feito com dois VPS diferentes de uma maneira mais
    simples, porém mais cara
* Executaremos os dois simultaneamente em portas diferentes durante a
atualização
    * O clone terá a nova versão do binário, enquanto o original terá a versão
    desatualizada
* Atualizaremos as portas na wallet para corresponder às portas do processo
atualizado, atualizando nosso mixnode
* Após o intervalo de tempo de 1 hora, encerraremos o processo original
desatualizado do mixnode
    * Esse intervalo de tempo não foi testado exaustivamente, mas nunca
    resultou na queda da pontuação de roteamento durante o desenvolvimento
    desse método

## Tutorial

> Este guia pressupõe que você tenha um mixnode já instalado com a configuração
de automação do systemd. Se esse não for o seu caso, consulte a [documentação
oficial da Nym](https://nymtech.net/operators/introduction.html) para obter
um guia sobre o processo de instalação e automação.

> Se esta for a primeira vez que usa esse método, você deverá seguir as etapas
da seção [Setup pela primeira vez](#Setup-pela-primeira-vez).

> Esse tutorial foi montado com VPSes rodando Ubuntu 20.04.

* **Ao longo deste guia, o mixnode original será chamado de `<DEFAULT>` e
presume-se que esteja sendo executado nas portas padrão. O mixnode clonado em
execução com portas alternativas será chamado de `<ALTERNATE>`. Sempre que essas
strings aparecerem nos comandos, você precisará substituí-las de acordo com sua
configuração.**

* **Neste guia, as portas alternativas usadas são 1337, 1338 e 1339. Essa
escolha não tem nenhum motivo específico e você pode escolher outras portas sem
nenhuma implicação na eficácia do método.**

### Setup pela primeira vez

Antes de começar a atualizar sem que seu routing score caia, você precisará
fazer algumas configurações iniciais.

#### Clonar mixnode

Para podermos executar dois processos mixnodes ao mesmo tempo, precisaremos
clonar nosso mixnode.

Podemos fazer isso executando o seguinte comando:

```bash
cp -rf $HOME/.nym/mixnodes/<DEFAULT> $HOME/.nym/mixnodes/<ALTERNATE>
```

#### Abrir portas alternativas

Para que o mixnode `<ALTERNATE>` possa ser executado, precisaremos executá-lo
com portas diferentes das do mixnode `<DEFAULT>`. E, por isso, precisamos
permitir essas portas alternativas em nosso firewall.

Podemos permitir as portas com o seguinte comando:

```bash
sudo ufw allow 1337,1338,1339/tcp
```

Você pode verificar as portas permitidas com o seguinte comando:

```bash
sudo ufw status
```

#### Armazenando binários

Como nossos mixnodes serão executados com binários diferentes, é uma boa ideia
criar uma estrutura de diretórios para armazenar os binários de forma
organizada.

Podemos criar os diretórios para nossos binários com os seguintes comandos:

```bash
mkdir -p $HOME/binaries/<DEFAULT>
mkdir -p $HOME/binaries/<ALTERNATE>
```

Com isso, teremos uma estrutura de diretórios de tal forma:

```
└──── $HOME
    └── binaries
        ├── <DEFAULT>
        └── <ALTERNATE>
```

#### Arquivos de serviço

Agora que temos uma estrutura para armazenar os diferentes binários, podemos
criar o arquivo de serviço para o nosso mixnode `<ALTERNATE>`. Esse arquivo de
serviço será muito semelhante ao que temos para o nosso mixnode `<DEFAULT>`,
apenas mudaremos o caminho no Execstart.

Crie o seguinte arquivo de serviço `/etc/systemd/system/<ALTERNATE>.service` e
coloque isso dentro dele:

> Substitua \<USER\> pelo seu username e \<VERSION\> pela versão do binário.

```
[Unit]
Description=Nym Mixnode <VERSION>
StartLimitInterval=350
StartLimitBurst=10

[Service]
User=<USER>
LimitNOFILE=65536
ExecStart=/home/<USER>/binaries/<ALTERNATE>/nym-mixnode run --id <ALTERNATE>
KillSignal=SIGINT
Restart=on-failure
RestartSec=30

[Install]
WantedBy=multi-user.target
```

Você também deve alterar o arquivo de serviço do mixnode `<DEFAULT>` para
apontar também para o novo diretório, mas não recarregue o daemon ainda.

Modifique o arquivo de serviço `/etc/systemd/system/<DEFAULT>.service` para
ficar dessa forma:

> Substitua \<USER\> pelo seu username e \<VERSION\> pela versão do binário.

```
[Unit]
Description=Nym Mixnode <VERSION>
StartLimitInterval=350
StartLimitBurst=10

[Service]
User=<USER>
LimitNOFILE=65536
ExecStart=/home/<USER>/binaries/<DEFAULT>/nym-mixnode run --id <DEFAULT>
KillSignal=SIGINT
Restart=on-failure
RestartSec=30

[Install]
WantedBy=multi-user.target
```

Com isso, todas as etapas de configuração foram concluídas e podemos prosseguir
com a atualização do node sem perder o routing score.

### Atualizando

> Nesta seção, presume-se que você tenha feito a configuração apresentada na
[seção anterior](#setup-pela-primeira-vez).

**Nesta seção, serão mostradas 2 opções de comandos:**
* `<DEFAULT>` desatualizado: se o seu mixnode desatualizado é aquele com as
portas padrões
* `<ALTERNATE>` desatualizado: se o seu mixnode desatualizado é aquele com as
portas alternativas 

> Se essa for sua primeira vez seguindo o tutorial você irá seguir os passos
marcados como `<DEFAULT>` desatualizado

#### Armazenando o binário atualizado

Se você não souber como obter os binários, poderá encontrar instruções na
[documentação oficial da Nym](https://nymtech.net/operators/introduction.html).

Depois de baixar ou compilar o binário nym-mixnode atualizado, será necessário
movê-lo para o diretório correspondente:

> Substitua \<PATH_TO_UPDATED_BINARY\> pelo path aonde você baixou ou compilou
o binário atualizado

Para `<DEFAULT>` desatualizado.

```bash
mv <PATH_TO_UPDATED_BINARY>/nym-mixnode $HOME/binaries/<ALTERNATE>/
```

Para `<ALTERNATE>` desatualizado.

```bash
mv <PATH_TO_UPDATED_BINARY>/nym-mixnode $HOME/binaries/<DEFAULT>/
```

#### Inicializando o mixnode

Agora você precisa iniciar o node com o novo binário:

Para `<DEFAULT>` desatualizado.

```bash
$HOME/binaries/<ALTERNATE>/nym-mixnode init --id <ALTERNATE> --host $(curl -4 https://ifconfig.me) --mix-port 1337 --verloc-port 1338 --http-api-port 1339
```

Para `<ALTERNATE>` desatualizado.

```bash
$HOME/binaries/<DEFAULT>/nym-mixnode init --id <DEFAULT> --host $(curl -4 https://ifconfig.me)
```

#### Iniciar e habilitar o serviço

Agora que seu mixnode está atualizado e precisamos colocá-lo em funcionamento,
podemos fazer isso habilitando e iniciando seu serviço:

Para `<DEFAULT>` desatualizado.

```bash
sudo systemctl enable <ALTERNATE>.service
sudo systemctl start <ALTERNATE>.service
```

Para `<ALTERNATE>` desatualizado.

```bash
sudo systemctl enable <DEFAULT>.service
sudo systemctl start <DEFAULT>.service
```

Você pode verificar se o mixnode está sendo executado corretamente com o
seguinte comando:

Para `<DEFAULT>` desatualizado.

```bash
systemctl status <ALTERNATE>.service
```

Para `<ALTERNATE>` desatualizado.

```bash
systemctl status <DEFAULT>.service
```

#### Atualizar informação na wallet

Agora você precisa atualizar as portas e a versão do mixnode na wallet, para
que o mixnode atualizado comece a receber pacotes.

Para fazer isso, você precisa abrir a wallet, clicar em Bonding e ir para
Node Settings: `Bonding -> Node Settings`

Na página de configurações do node, você precisará atualizar os campos de
acordo:

Para `<DEFAULT>` desatualizado.

```bash
Mix port = 1337
Verloc port = 1338
HTTP port = 1339
Version = nova versão do mixnode
```

Para `<ALTERNATE>` desatualizado.

```bash
Mix port = 1789
Verloc port = 1790
HTTP port = 8000
Version = nova versão do mixnode
```
Depois disso o mixnode atualizado estará recebendo os pacotes

#### Desligar serviço desatualizado

**Depois de esperar 1 hora** a partir do momento em que você atualizou as
portas na wallet, você pode desligar o serviço que está executando o mixnode
desatualizado:
> O intervalo de tempo ainda não foi totalmente testado, mas 1 hora deve ser um
intervalo seguro para que o seu routing score não caia

Para `<DEFAULT>` desatualizado.

```bash
sudo systemctl disable <DEFAULT>.service
sudo systemctl stop <DEFAULT>.service
```

Para `<ALTERNATE>` desatualizado.

```bash
sudo systemctl disable <ALTERNATE>.service
sudo systemctl stop <ALTERNATE>.service
```

Se esta foi a primeira vez que você atualizou com esse método, deverá
recarregar o daemon para registrar as alterações que fizemos no arquivo de
serviço `<DEFAULT>`. Você pode fazer isso executando o seguinte comando:

```bash
sudo systemctl daemon-reload
```

Com isso, seu mixnode agora está sendo executado com o binário mais recente e
seu routing score permaneceu intacto.

## Conclusão

Se você tiver alguma contribuição que gostaria de fazer, sinta-se à vontade
para abrir um pull request ou uma issue no
[repositório do github](https://github.com/TupiNymQuim/tupinymquim.github.io).

E se isso foi útil para você, considere nos apoiar dando stake em um dos
seguintes mixnodes:
* [TupiNymQuim-nodes](https://tupinymquim.github.io/nodes/)
* [supermeia-node](https://explorer.nymtech.net/network-components/mixnode/927)
