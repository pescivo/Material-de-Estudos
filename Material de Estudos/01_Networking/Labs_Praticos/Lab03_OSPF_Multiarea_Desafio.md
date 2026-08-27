# Lab 03 — OSPF Multiárea: a NexaCorp abre um Data Center

🟡 Intermediário · ⏱️ 75–90 min · 🧰 GNS3, Arista vEOS, VPCS

---

## O chamado

Seis meses depois do Lab 02. A NexaCorp tá crescendo rápido — rápido demais pro jeito que você vem administrando a rede. E o Marcelo chega com isso:

> **De:** Marcelo Andrade (Coordenador de TI)
> **Para:** Você
> **Assunto:** Novo Data Center + problema que já vi vindo há meses
>
> Boa notícia primeiro: fechamos contrato de um Data Center próprio, vai ficar conectado direto na Filial. É onde vamos hospedar os sistemas internos que hoje ainda rodam na Matriz.
>
> Má notícia: com esse novo site entrando, ficou insustentável continuar com rota estática. Hoje já são duas redes (Matriz e Filial) com uma rota fixa cada uma apontando pra outra. Com o Data Center entrando, são três sites, e cada novo site que a gente abrir no futuro significa eu ter que lembrar de mexer em rota estática em todo roteador da rede, manualmente, um por um. Isso não escala e é receita pra erro humano — vai chegar o dia que alguém esquece de atualizar uma rota em algum lugar e ninguém percebe até dar problema em produção.
>
> Quero que você migre pra um protocolo de roteamento dinâmico. Conversei com o pessoal mais sênior e a orientação foi OSPF, dividido em áreas — o Data Center fica isolado numa área própria, separado da área que já existe entre Matriz e Filial. Eles explicaram que isso ajuda a conter o impacto se alguma coisa instável acontecer numa parte da rede, sem a instabilidade se espalhar pra rede inteira. Faz sentido pra você, então bora.
>
> Ah — e não apaga a rota estática dos labs anteriores até ter certeza que o OSPF tá funcionando 100%. Prefiro rede redundante por um tempo a ficar sem conectividade nenhuma no meio da migração.

Esse é o primeiro lab da trilha que exige você pensar em escala, não só em "fazer dois pontos se enxergarem". É um salto real de complexidade em relação aos labs anteriores.

## Objetivo do Lab

Migrar a topologia Matriz–Filial (herdada do Lab 01) de rota estática para OSPF, e integrar um novo site — o Data Center, conectado à Filial — como uma segunda área OSPF, com o roteador da Filial atuando como fronteira entre as duas áreas (ABR).

## Topologia

```
[PC-MATRIZ]                                                              [PC-DATACENTER]
     |                                                                          |
192.168.10.0/24                                                        192.168.30.0/24
     |                                                                          |
Gi0/0 [R-MATRIZ] Gi0/1 ---- Gi0/1 [R-FILIAL] Gi0/2 ---- Gi0/1 [R-DATACENTER] Gi0/0
                    \       /         \                          /
                 10.0.0.1  10.0.0.2  10.0.1.1                10.0.1.2
                  (link /30 — ÁREA 0)   (link /30 — ÁREA 1)

                              PC-FILIAL em 192.168.20.0/24, atrás de R-FILIAL Gi0/0
                              (não desenhado acima por espaço — está na mesma LAN da Filial de sempre)
```

R-FILIAL é o roteador que fica na fronteira entre as duas áreas — ele tem uma perna na Área 0 (o backbone, que já existia desde o Lab 01, entre Matriz e Filial) e outra perna na Área 1 (o novo link até o Data Center).

## Tabela de Endereçamento e Áreas

| Dispositivo | Interface | IP | Área OSPF |
|---|---|---|---|
| R-MATRIZ | Gi0/0 (LAN) | 192.168.10.1/24 | 0 |
| R-MATRIZ | Gi0/1 (link p/ Filial) | 10.0.0.1/30 | 0 |
| R-FILIAL | Gi0/1 (link p/ Matriz) | 10.0.0.2/30 | 0 |
| R-FILIAL | Gi0/0 (LAN) | 192.168.20.1/24 | 0 |
| R-FILIAL | Gi0/2 (link p/ Data Center) | 10.0.1.1/30 | 1 |
| R-DATACENTER | Gi0/1 (link p/ Filial) | 10.0.1.2/30 | 1 |
| R-DATACENTER | Gi0/0 (LAN) | 192.168.30.1/24 | 1 |
| PC-MATRIZ | — | 192.168.10.10/24 | gateway 192.168.10.1 |
| PC-FILIAL | — | 192.168.20.10/24 | gateway 192.168.20.1 |
| PC-DATACENTER | — | 192.168.30.10/24 | gateway 192.168.30.1 |

## O que você precisa entregar

1. Configurar R-DATACENTER (novo roteador) com as interfaces conforme a tabela.
2. Habilitar OSPF nos três roteadores, anunciando as redes corretas em cada área — R-MATRIZ e a perna do R-FILIAL voltada pra Matriz ficam na Área 0; a perna do R-FILIAL voltada pro Data Center e o R-DATACENTER inteiro ficam na Área 1.
3. Confirmar que os três roteadores formam adjacência OSPF entre si (nos pontos que fazem sentido — R-MATRIZ e R-DATACENTER nunca vão formar adjacência direta, eles não têm link físico entre si; a comunicação entre eles acontece através de R-FILIAL).
4. Validar que PC-DATACENTER consegue alcançar PC-MATRIZ e PC-FILIAL, e vice-versa, com as rotas aprendidas dinamicamente via OSPF — não mais via rota estática.
5. Confirmar, olhando a tabela de rotas, que as rotas aprendidas por OSPF aparecem marcadas de forma diferente das que seriam estáticas ou diretamente conectadas.

Ponto de atenção que o Marcelo já deixou claro no e-mail: não apague as rotas estáticas do Lab 01 ainda. A tarefa aqui é fazer o OSPF funcionar corretamente ao lado delas — mais pra frente, em lab futuro, vamos tratar de remover a rota estática com segurança.

## Perguntas pra você responder antes de configurar

O Marcelo mencionou que dividir em áreas "ajuda a conter o impacto se alguma coisa instável acontecer numa parte da rede". Sem áreas, um protocolo de estado de enlace como OSPF inunda informação de topologia pra todo roteador do domínio inteiro toda vez que algo muda. Com base nisso, o que exatamente fica contido dentro dos limites de uma área, e por que isso importa numa rede que só tende a crescer?

Segunda pergunta, mais prática: dado que R-MATRIZ e R-DATACENTER não têm link físico entre si, como exatamente o PC-DATACENTER vai aparecer na tabela de rotas de R-MATRIZ? Que tipo de rota OSPF você espera ver ali, considerando que ela atravessa uma fronteira de área pra chegar lá?

## Cenário bônus — depois que tudo estiver funcionando

Uma semana depois, com tudo funcionando, o time de rede faz uma manutenção programada em R-FILIAL de madrugada e, sem querer, uma configuração fica errada. Na manhã seguinte, o Data Center inteiro fica inacessível a partir da Matriz e da Filial — mas curiosamente, dentro do próprio Data Center tudo continua normal.

Reproduza isso: depois de validar que o lab inteiro funciona, vá em R-FILIAL e altere a área OSPF configurada na interface Gi0/2 (o link pro Data Center) de Área 1 para Área 0 — sem mudar mais nada.

Diagnostique como se fosse esse chamado de madrugada: qual comando mostra o estado da adjacência OSPF entre dois roteadores, e o que exatamente você esperaria ver diferente nele quando existe incompatibilidade de área entre as duas pontas de um link? Esse é um dos poucos erros de OSPF que nem chega a formar vizinhança — diferente de outros problemas de OSPF que formam adjacência parcial e falham depois.

---

⚠️ OSPF tem mais peças em movimento que os labs anteriores. Se travar, revisita a Fase 3 do roadmap antes de ir direto pra solução.

Quando terminar, siga para `Lab03_OSPF_Multiarea_Solucao.md`.
