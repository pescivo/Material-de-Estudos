# Lab 04 — Quem pode falar com o Data Center

🟡 Intermediário · ⏱️ 60–75 min · 🧰 GNS3, Arista vEOS, VPCS

---

## O chamado

A rede da NexaCorp evoluiu desde o Lab 03. O SW-CORE que você configurou no Lab 02 ganhou um uplink até R-MATRIZ e hoje já fala OSPF, anunciando as três redes de departamento — Financeiro (192.168.11.0/24), TI (192.168.12.0/24) e Vendas (192.168.13.0/24) — dentro da Área 0. Isso já está funcionando e não é o foco deste lab: assuma que PC-FIN, PC-TI e PC-VENDAS já conseguem alcançar qualquer ponto da topologia, incluindo o Data Center, normalmente. É exatamente isso que vai virar problema agora.

> **De:** Marcelo Andrade (Coordenador de TI)
> **Para:** Você
> **Assunto:** RES: Auditoria — achado crítico no Data Center
>
> A auditoria voltou. Dessa vez o achado foi mais sério: qualquer estação da empresa, incluindo o time de Vendas, consegue alcançar os servidores do Data Center hoje. O relatório classificou isso como risco alto, porque o Data Center processa dado financeiro sensível e Vendas não tem nenhuma relação de negócio com esse sistema.
>
> O que precisamos: Financeiro e TI continuam com acesso normal ao Data Center — isso é operação, não mexe. Vendas precisa ser bloqueado especificamente do Data Center, sem afetar o resto do acesso deles (internet, sistema de vendas, etc. continua tudo normal).
>
> Um adendo que o próprio time de auditoria fez questão de destacar: eles pediram pra você documentar exatamente onde na topologia você aplicou o bloqueio e por quê ali e não em outro ponto. Eles querem entender se a decisão foi pensada ou só "colocada em qualquer lugar que funcionou".

Esse último parágrafo não é enfeite — é o cerne do lab. ACL mal posicionada na topologia é um erro tão comum quanto ACL mal escrita.

## Objetivo do Lab

Restringir o acesso da rede de Vendas (192.168.13.0/24) ao Data Center (192.168.30.0/24), preservando acesso total de Financeiro (192.168.11.0/24) e TI (192.168.12.0/24) ao mesmo destino, e sem impactar nenhum outro tráfego dessas redes.

## Topologia (relevante para este lab)

```
[PC-FINANCEIRO 192.168.11.10]  \
[PC-TI 192.168.12.10]           >---- SW-CORE ---- R-MATRIZ ---- R-FILIAL ---- R-DATACENTER ---- [PC-DATACENTER 192.168.30.10]
[PC-VENDAS 192.168.13.10]      /                                                    |
                                                                              192.168.30.0/24
```

As três redes de departamento chegam até o Data Center passando pelos mesmos roteadores — R-MATRIZ, R-FILIAL e R-DATACENTER — porque é o único caminho físico que existe na topologia inteira. Isso é relevante pra pensar em onde aplicar o filtro.

## O que você precisa entregar

1. Criar uma ACL estendida que permita tráfego de Financeiro e TI em direção ao Data Center, negue tráfego de Vendas em direção ao mesmo destino, e permita todo o resto do tráfego que passar por ali (lembre que ACL estendida tem um "deny all" implícito no final — se você esquecer de tratar isso, corre o risco de derrubar tráfego que nunca devia ter sido afetado).
2. Decidir em qual roteador e em qual interface aplicar essa ACL, e em qual direção (entrada ou saída), justificando essa escolha — é literalmente o que o time de auditoria pediu pra você documentar.
3. Validar que PC-FINANCEIRO e PC-TI continuam alcançando PC-DATACENTER normalmente.
4. Validar que PC-VENDAS não alcança mais PC-DATACENTER.
5. Validar que PC-VENDAS continua alcançando normalmente qualquer outro destino da topologia (por exemplo, PC-MATRIZ ou PC-FILIAL) — isso prova que sua ACL está cirurgicamente restrita ao que foi pedido, e não bloqueando tráfego que não deveria.

## Pergunta pra você responder antes de configurar qualquer coisa

Existem, no mínimo, três lugares fisicamente possíveis pra aplicar esse filtro: na saída de R-FILIAL em direção ao Data Center, na entrada de R-DATACENTER vindo da Filial, ou até mais perto da origem, nalgum ponto entre SW-CORE e R-MATRIZ. Pensa nos seguintes fatores antes de escolher: quantas redes de origem diferentes esse filtro precisa considerar hoje, o que acontece se a NexaCorp abrir um quarto departamento daqui a três meses, e qual ponto da topologia continua sendo o lugar certo de aplicar a regra mesmo depois dessa mudança futura, sem você precisar voltar e reconfigurar tudo de novo. A resposta certa não é a única tecnicamente possível — é a que exige menos manutenção repetida conforme a empresa cresce.

## Cenário bônus — depois que tudo estiver funcionando

Duas semanas depois, o Marcelo te chama de novo:

> "Pedro, será que a ACL parou de funcionar? Vendas conseguiu acessar o sistema do Data Center hoje de manhã, um analista comentou isso comigo sem querer."

Você não mudou o conteúdo da ACL — mas alguém, numa manutenção, reordenou as linhas dela, movendo a regra de permissão geral (`permit ip any any`) pra antes da regra específica que nega Vendas, em vez de deixá-la por último. Reproduza isso: recria a mesma ACL, mas com a ordem das linhas trocada dessa forma.

Diagnostique: ACL em roteador processa as regras em qual ordem — de cima pra baixo, avaliando todas antes de decidir, ou parando na primeira que casar com o pacote? Baseado nisso, qual comando mostra não só o conteúdo da ACL, mas quantas vezes cada linha específica bateu com tráfego real — informação que ajuda a confirmar exatamente qual regra está "roubando" o tráfego que deveria cair na regra de negação?

---

⚠️ Antes de ir pra solução, decide sozinho onde aplicar a ACL e por quê — essa parte importa tanto quanto escrever a regra certa.

Quando terminar, siga para `Lab04_ACLs_Solucao.md`.
