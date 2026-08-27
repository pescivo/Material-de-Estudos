# Roadmap de Networking — do zero até você conseguir sustentar uma rede sozinho

Antes de te jogar uma lista de tópicos, preciso te dizer uma coisa que eu demorei pra entender: rede não é uma coleção de comandos decorados. É um jeito de raciocinar sobre como informação se move de um ponto A até um ponto B, passando por N problemas no meio do caminho. Todo esse roadmap foi desenhado em cima dessa ideia. Cada fase constrói o raciocínio da anterior — não pula fase achando que vai economizar tempo, porque não vai.

Vou dividir isso em seis fases. As cinco primeiras são teoria + prática amarradas. A sexta é onde tudo se junta nos dez laboratórios que ficam na pasta `Labs_Praticos`.

## Fase 1 — A base que ninguém quer estudar mas todo mundo devia

Aqui entra o Modelo OSI e a pilha TCP/IP, endereçamento IPv4, máscara de sub-rede e VLSM. Eu sei que parece a parte mais chata do processo inteiro — todo curso começa por aqui e a maioria das pessoas passa correndo pra chegar na parte "legal". Não faz isso.

A razão é simples: quando você for fazer troubleshooting de verdade, daqui a alguns meses, o que vai te salvar não é saber decorar comando de switch. É conseguir olhar pra uma captura de tráfego e pensar "ok, isso tá na camada 3, então o problema não é de switching, é de roteamento". Isso só vem de entender profundamente onde cada coisa acontece na pilha.

O que estudar nessa fase:
- Modelo OSI e TCP/IP — não decoreba de camada, mas entender o que cada camada resolve e por que ela existe separada das outras
- Endereçamento IPv4: classes, máscara, notação CIDR
- VLSM na prática — pegar um bloco e subdividir de forma eficiente, sem desperdiçar endereço
- Cabeamento e meios físicos, o suficiente pra você não ficar boiando quando alguém falar de fibra vs par trançado

Cheatsheet de apoio: `Cheatsheets/Cisco_IOS_Comandos.md`.

## Fase 2 — Switching: como uma rede local realmente se comporta

Depois que você entende endereçamento, entra a comutação. VLANs, trunking (802.1Q), Spanning Tree Protocol, EtherChannel quando fizer sentido.

Esse é o ponto onde a maioria das pessoas que estuda sozinha começa a decorar comando sem entender o motivo. "Ah, faz `switchport mode trunk` e pronto." Só que se você não entende por que um trunk carrega múltiplas VLANs numa única porta física, você não vai saber diagnosticar quando uma VLAN some do outro lado do link e ninguém sabe por quê. Isso vai acontecer. Prepara pra isso agora.

Tópicos centrais: VLANs e segmentação lógica, trunking e encapsulamento 802.1Q, Spanning Tree e por que ele existe (loop de camada 2 é um dos problemas mais desagradáveis de debugar em produção), inter-VLAN routing.

Aqui vale um parêntese técnico importante: se você tá seguindo esse material comigo usando Arista vEOS (que é a escolha que eu fiz pro meu laboratório, porque o Cisco CML com IOS completo é pago), a forma de fazer inter-VLAN routing muda. Em vez do clássico Router-on-a-Stick que você vê em todo curso de Cisco, no Arista EOS você trabalha com SVI (Switch Virtual Interface) diretamente no switch. É a mesma ideia, sintaxe diferente. Tem um cheatsheet de tradução Arista/Cisco na pasta pra te ajudar nessa transição.

## Fase 3 — Roteamento: o coração da coisa toda

Estático primeiro, sempre. Só depois de você configurar rota estática o suficiente pra sentir na pele a limitação dela (não escala, não se adapta a mudança de topologia) é que o roteamento dinâmico vai fazer sentido de verdade.

OSPF é o ponto de entrada pra roteamento dinâmico nesse roadmap — é o mais usado em ambiente corporativo de médio porte e o que a maioria das certificações cobra pesado. Depois de OSPF sólido, entra BGP, que é outra categoria de complexidade: é o protocolo que sustenta a internet como um todo, e entender ele — mesmo que de forma introdutória — muda como você enxerga rede corporativa se conectando com provedor.

Nessa fase o objetivo não é decorar sintaxe de configuração. É entender: como um roteador decide qual caminho é o melhor, o que acontece quando duas rotas competem, e o que fazer quando a convergência não acontece do jeito esperado.

## Fase 4 — Serviços que fazem a rede ser usável

Rede sem serviço é só cabo conectando nada. Aqui entra DHCP, DNS, NAT e ACLs.

ACL merece atenção redobrada, porque é literalmente o primeiro contato que você vai ter com "segurança de rede" antes mesmo de chegar na trilha de Cybersecurity. Uma ACL mal configurada é, ao mesmo tempo, o motivo de metade dos chamados de "não consigo acessar tal serviço" e a primeira linha de defesa contra tráfego indevido. Vale a pena parar e entender de verdade a ordem de avaliação das regras — é um erro clássico até de gente com experiência.

## Fase 5 — Onde a coisa fica séria: redes avançadas e automação

Depois que a base tá consolidada, entra o que separa quem "sabe configurar rede" de quem realmente sustenta infraestrutura em escala: conceitos de SDN (Software Defined Networking), e automação de rede com Python (bibliotecas como Netmiko) ou Ansible.

Automação não é luxo nem "coisa de empresa grande". É reconhecer que configurar 40 switches manualmente, um por um, é onde erro humano vira incidente. Mesmo que seu primeiro emprego não peça isso, ter esse conhecimento te coloca à frente de quem só sabe interface de linha de comando manual.

## Fase 6 — Os dez laboratórios: onde a teoria vira mão na massa

Toda a teoria das fases 1 a 5 se materializa nos dez projetos práticos da pasta `Labs_Praticos`, organizados em ordem crescente de dificuldade:

Os primeiros labs cobrem topologia básica com dois ou três roteadores, endereçamento e conectividade simples — é ali que você comete os primeiros erros bobos de digitação de IP e aprende a debugar isso rápido. Na sequência entram VLANs e inter-VLAN routing, depois OSPF em cenário multi-área, ACLs aplicadas em cenário realista, NAT em borda simulando saída pra internet, e os últimos labs já misturam múltiplos protocolos rodando juntos, do jeito que uma rede corporativa real funciona — nada isolado, tudo interagindo e, às vezes, conflitando.

Cada lab individual, dentro da pasta, já vem com o objetivo específico, a topologia completa com endereçamento, o passo a passo e o desafio de troubleshooting. Esse roadmap é só o mapa; o trabalho pesado acontece lab por lab.

## Sobre certificações, rapidinho

Na pasta `04_Certificacoes` eu listo algumas opções gratuitas que valem a pena olhar como complemento — Cisco Networking Academy é a mais direta pra quem tá nessa trilha. Só um aviso importante: exame muda de formato, código muda, conteúdo é atualizado pelas próprias empresas com frequência. O que eu escrevo lá é um ponto de partida, não a fonte definitiva — sempre confirma direto no site oficial antes de se inscrever ou investir tempo estudando pra um formato de prova que já mudou.

## Depois disso

Quando você fechar os dez labs e sentir que consegue montar uma topologia do zero, com VLAN, roteamento dinâmico e serviços básicos funcionando, sem copiar comando de lugar nenhum — você tá pronto pra trilha de Cybersecurity com uma base de verdade embaixo do pé. É literalmente o que separa quem entende o que tá defendendo de quem só aprendeu ferramenta.

Bora pra fase 1.
