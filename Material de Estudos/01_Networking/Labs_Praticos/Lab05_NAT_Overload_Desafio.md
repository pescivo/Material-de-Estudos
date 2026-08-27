# Lab 05 — Abrindo a porta pra internet (com controle)

🟡 Intermediário · ⏱️ 75–90 min · 🧰 GNS3, Arista vEOS, VPCS

---

## O chamado

> **De:** Marcelo Andrade (Coordenador de TI)
> **Para:** Você
> **Assunto:** Link de internet contratado — precisa configurar até amanhã
>
> Fechamos com o provedor. Eles vão entregar um link dedicado direto na Matriz, com um único IP público — não ganhamos bloco, é só um IP mesmo, isso é o que cabe no contrato que fechamos por enquanto. O provedor confirmou o endereço deles do outro lado do link e o nosso, te mando os dados abaixo.
>
> O que eu preciso: todo mundo da empresa — Matriz, Filial, Data Center, os três departamentos que você segmentou lá atrás — precisa conseguir sair pra internet através desse link único, usando esse único IP público que o provedor nos deu. Eu sei que isso significa que centenas de máquinas vão "compartilhar" um IP só do lado de fora, mas foi o que o provedor ofereceu no plano que contratamos, e sinceramente nem faria sentido pagar por IP público pra cada máquina interna.
>
> Ah, e não esquece: isso precisa funcionar pra rede inteira, não só pra quem tá fisicamente na Matriz. O pessoal do Data Center e da Filial também precisa sair pela internet através desse mesmo link — eles não vão ganhar link próprio.
>
> Testa com o servidor de atualização do antivírus que o próprio provedor disponibilizou pra gente simular esse teste, o IP tá nos dados técnicos que anexei.

O provedor te passou os seguintes dados técnicos:

- IP da NexaCorp na borda (nosso lado do link): `203.0.113.1/30`
- IP do roteador do provedor (outro lado do link): `203.0.113.2/30`
- Servidor de teste do provedor (simulando um host público qualquer na internet): `200.200.200.10/32`

## Objetivo do Lab

Configurar NAT com sobrecarga (overload/PAT) em R-MATRIZ, traduzindo o tráfego de saída de todas as redes internas da NexaCorp para o único IP público fornecido pelo provedor, e garantir que essa saída para a internet funcione a partir de qualquer ponto da topologia — incluindo Filial e Data Center, que estão a múltiplos saltos de distância de R-MATRIZ.

## Topologia

```
                                                                    [R-ISP]
                                                                  Ethernet1: 203.0.113.2/30
                                                                  Loopback0: 200.200.200.10/32
                                                                       |
                                                                  Ethernet2 (NOVA)
                                                                  203.0.113.1/30
                                                                       |
[PC-DATACENTER] -- R-DATACENTER -- R-FILIAL -- R-MATRIZ -- (Ethernet2 = saída pra internet)
                                       |            |
                                  [PC-FILIAL]  [PC-MATRIZ + VLANs do Lab 02]
```

R-MATRIZ ganha uma terceira interface, ligada diretamente ao roteador do provedor (R-ISP, que representa a internet nesse laboratório). Toda a topologia interna que você já construiu — OSPF incluso — continua exatamente como estava.

## O que você precisa entregar

1. Configurar R-ISP com a interface voltada pra Matriz e o servidor de teste (uma interface loopback funciona bem pra simular um host público fixo).
2. Configurar a nova interface de R-MATRIZ voltada ao provedor, com o IP público informado.
3. Configurar uma rota padrão (default route) em R-MATRIZ apontando para o roteador do provedor, pra qualquer destino desconhecido sair por ali.
4. Propagar essa rota padrão para dentro do domínio OSPF, de forma que R-FILIAL e R-DATACENTER também aprendam automaticamente que "qualquer coisa que eu não conheço, mando pra Matriz". Sem isso, só quem está fisicamente na Matriz vai conseguir sair pra internet — e o Marcelo foi claro que isso precisa valer pra empresa inteira.
5. Configurar NAT com overload em R-MATRIZ, definindo especificamente quais redes internas têm permissão de ter seu tráfego traduzido — não existe "internal" implícito nesse tipo de configuração, você precisa dizer explicitamente quais faixas de origem contam como tráfego interno elegível pra tradução.
6. Validar que PC-MATRIZ, PC-FILIAL e PC-DATACENTER conseguem alcançar o servidor de teste do provedor (200.200.200.10), mesmo estando a distâncias diferentes dentro da topologia.

## Perguntas pra você responder antes de configurar

O provedor te deu um único IP público, mas a NexaCorp inteira — potencialmente centenas de hosts — precisa sair por ele ao mesmo tempo. Um NAT dinâmico "normal" (sem overload) mapeia um IP privado pra um IP público de forma exclusiva, um pra um — isso obviamente não escala pra essa situação. O que exatamente o "overload" (também chamado de PAT, Port Address Translation) muda nesse comportamento, e o que ele usa como informação extra pra conseguir diferenciar múltiplas conexões internas simultâneas compartilhando o mesmo IP público de saída?

Segunda pergunta: quando o pacote de resposta volta do servidor do provedor, endereçado ao IP público 203.0.113.1, como exatamente R-MATRIZ sabe pra qual host interno específico (entre potencialmente centenas) aquele pacote de volta pertence?

## Cenário bônus — depois que tudo estiver funcionando

Um mês depois, o Marcelo escreve de novo:

> "Pedro, o pessoal do Data Center reportou que não conseguem mais acessar nenhum site nem baixar atualização de segurança. Filial e Matriz continuam normais. Isso é grave, eles processam dados sensíveis e precisam de atualização em dia."

Reproduza esse cenário: na configuração de quais redes internas são elegíveis pra tradução NAT, remova especificamente a entrada referente à rede do Data Center (192.168.30.0/24) — sem mexer em mais nada, nem na rota padrão, nem no OSPF, nem nas outras redes.

Diagnostique: o roteamento (rota padrão propagada via OSPF) continua perfeitamente funcional pro Data Center — um traceroute a partir de lá provavelmente mostra o pacote saindo em direção certa. Então onde exatamente esse pacote está sendo descartado, e por quê? Qual comando mostra quais traduções NAT estão ativas em tempo real, e o que a ausência de qualquer entrada relacionada ao Data Center nesse comando te diz sobre a causa raiz — mesmo com o roteamento inteiro funcionando perfeitamente?

---

⚠️ Esse lab depende do OSPF do Lab 03 estar funcional. Se você pulou ele ou não tem certeza que está estável, vale revisar antes de continuar.

Quando terminar, siga para `Lab05_NAT_Overload_Solucao.md`.
