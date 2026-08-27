# Lab 02 — Segmentando a Matriz que cresceu demais

🟡 Intermediário · ⏱️ 60–75 min · 🧰 GNS3, Arista vEOS (switch L3), switch L2 (Arista vEOS ou similar), VPCS

---

## O chamado

Já se passaram três meses desde que você resolveu a conectividade entre Matriz e Filial no Lab 01. A NexaCorp cresceu — contrataram mais gente pro Financeiro e montaram um time de TI interno maior. E aí chegou isso:

> **De:** Marcelo Andrade (Coordenador de TI)
> **Para:** Você
> **Assunto:** Segmentação da rede da Matriz — auditoria pediu isso ontem
>
> Cara, a auditoria de segurança que contratamos passou aqui semana passada e apontou um problema sério: hoje toda a Matriz tá numa rede flat só, 192.168.10.0/24, todo mundo no mesmo domínio de broadcast. Financeiro, TI, Vendas — todo mundo enxergando o tráfego de todo mundo. Isso não pode continuar assim, principalmente com o Financeiro processando dado sensível.
>
> Preciso que você segmente isso em VLANs: uma pro Financeiro, uma pra TI, uma pra Vendas. Cada área precisa continuar conseguindo acessar a internet e os sistemas compartilhados — então elas precisam conseguir rotear entre si quando fizer sentido, só não podem mais compartilhar o mesmo domínio de broadcast.
>
> Ah, e um detalhe: o Financeiro fica no mesmo andar que TI e Vendas, mas as máquinas estão espalhadas em pontos diferentes do prédio, ligadas num switch de acesso separado do switch principal. Então você vai precisar garantir que múltiplas VLANs consigam trafegar entre os dois switches por um único link físico — não temos cabo sobrando pra passar um por VLAN.
>
> Falo com você amanhã. Valeu.

Esse chamado é praticamente um resumo da Fase 2 do roadmap inteira: VLAN, trunk e inter-VLAN routing, tudo junto, num cenário real de auditoria de segurança batendo na porta.

## Objetivo do Lab

Segmentar a rede local da Matriz em três VLANs (Financeiro, TI e Vendas), garantir que o tráfego entre elas atravesse um único link trunk entre dois switches, e permitir comunicação roteada entre as VLANs através de interfaces virtuais (SVI) no switch núcleo — sem usar um roteador físico separado pra isso.

## Topologia

```
                    [SW-CORE]  (switch de núcleo, faz roteamento entre VLANs)
                        |
                     Ethernet1 (TRUNK — precisa carregar as 3 VLANs)
                        |
                   [SW-ACESSO]  (switch de acesso, só L2)
                    /    |    \
                Eth2    Eth3   Eth4
                 |        |      |
            [PC-FIN]  [PC-TI]  [PC-VENDAS]
```

O SW-CORE é o único ponto da rede com inteligência de camada 3 nesse lab — é ele que vai ter as interfaces virtuais (SVI) fazendo o papel de gateway de cada VLAN. O SW-ACESSO só enxerga e encaminha quadro de camada 2, nada de roteamento nele.

## Tabela de Endereçamento

| VLAN | Nome | Rede | Gateway (SVI no SW-CORE) |
|---|---|---|---|
| 10 | FINANCEIRO | 192.168.11.0/24 | 192.168.11.1 |
| 20 | TI | 192.168.12.0/24 | 192.168.12.1 |
| 30 | VENDAS | 192.168.13.0/24 | 192.168.13.1 |

| Host | VLAN | IP | Gateway |
|---|---|---|---|
| PC-FIN | 10 | 192.168.11.10/24 | 192.168.11.1 |
| PC-TI | 20 | 192.168.12.10/24 | 192.168.12.1 |
| PC-VENDAS | 30 | 192.168.13.10/24 | 192.168.13.1 |

## O que você precisa entregar

1. Criar as três VLANs (10, 20, 30) em ambos os switches, com os nomes indicados na tabela.
2. Configurar as portas de acesso no SW-ACESSO, associando cada uma à VLAN correta (Eth2 → VLAN 10, Eth3 → VLAN 20, Eth4 → VLAN 30).
3. Configurar o link entre SW-CORE e SW-ACESSO (Ethernet1 em ambos) como trunk, permitindo explicitamente as três VLANs.
4. Criar as SVIs no SW-CORE com os IPs de gateway da tabela, e habilitar roteamento entre elas — em Arista EOS isso exige um comando específico de nível global, além de criar a interface virtual. Não vou te dizer qual.
5. Configurar os três hosts.
6. Validar que PC-FIN consegue pingar PC-TI e PC-VENDAS (comunicação inter-VLAN via SVI) e que todos alcançam seus respectivos gateways.

Antes de sair digitando: se o SW-ACESSO é só camada 2, como exatamente o quadro de um host na VLAN 10 chega até a interface virtual VLAN 10 que fica fisicamente no SW-CORE, passando por um único cabo (o trunk) que também carrega tráfego da VLAN 20 e 30 ao mesmo tempo? Se você não consegue responder isso com confiança, revisa 802.1Q antes de configurar qualquer coisa — vai economizar retrabalho.

## Cenário bônus — depois que tudo estiver funcionando

No dia seguinte, o Marcelo te manda uma nova mensagem:

> "O pessoal de Vendas tá reclamando que não consegue nem pingar o próprio gateway. Financeiro e TI estão ok. Da uma olhada?"

Reproduza isso você mesmo: depois de validar que as três VLANs funcionam, vá até a configuração de trunk no SW-ACESSO (ou no SW-CORE, escolha um dos dois) e remova a VLAN 30 da lista de VLANs permitidas no trunk — sem apagar a VLAN em si, só tirar ela da allowed list do link trunk.

Diagnostique: qual comando mostra quais VLANs estão realmente passando por uma porta trunk, e não só quais VLANs existem no switch? Esses são dois comandos diferentes — se você usar o errado, vai levar mais tempo pra encontrar o problema do que devia.

---

⚠️ Antes de ir pra solução, tenta resolver isso sem olhar. Esse lab mistura três conceitos de uma vez — é normal errar a ordem de configuração na primeira tentativa.

Quando terminar, siga para `Lab02_VLANs_e_InterVLAN_Routing_Solucao.md`.
