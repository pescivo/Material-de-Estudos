# OSPF — Fundamentos

Esse arquivo assume que você já leu `Roteamento_Estatico_vs_Dinamico.md` e já tem, no mínimo, feito o Lab 03. Aqui eu vou aprofundar o mecanismo por trás do que você já configurou na prática — o "por dentro" do protocolo, não só o comando.

## O que é OSPF, de forma direta

OSPF (Open Shortest Path First) é um protocolo de roteamento dinâmico de estado de enlace (link-state), amplamente usado em redes corporativas de médio a grande porte. "Open" no nome se refere a ele ser um padrão aberto, não proprietário de nenhum fabricante específico — diferente, por exemplo, do EIGRP, que nasceu como protocolo proprietário da Cisco.

## Como OSPF constrói sua visão da rede

Cada roteador rodando OSPF constrói e mantém uma base de dados de estado de enlace (Link-State Database, ou LSDB) — essencialmente um mapa completo de todos os roteadores e links dentro da mesma área, obtido através da troca de mensagens chamadas LSAs (Link-State Advertisements). Diferente de um protocolo de vetor de distância, onde um roteador só sabe "por onde" chegar num destino, com OSPF cada roteador de uma área acaba tendo, na prática, o mesmo mapa completo que todos os outros roteadores daquela área têm.

A partir desse mapa, cada roteador roda independentemente o algoritmo de Dijkstra (Shortest Path First, o "SPF" no nome do protocolo) pra calcular o melhor caminho até cada destino conhecido, considerando o custo configurado em cada link. Isso significa que a decisão de melhor caminho não é tomada de forma centralizada nem repassada de roteador em roteador — cada um calcula, de forma independente, a partir da mesma informação de topologia.

## Formação de adjacência: como dois roteadores "se conhecem"

Antes de trocar qualquer informação de topologia, dois roteadores OSPF precisam formar uma adjacência — um relacionamento de vizinhança confirmado. Isso começa com a troca de pacotes Hello, enviados periodicamente em cada interface habilitada pra OSPF. Esses pacotes carregam informações que precisam bater entre as duas pontas pra adjacência se formar: a área configurada na interface (você viu, no Lab 03, o que acontece quando isso não bate — a adjacência simplesmente não se forma), o intervalo de tempo entre Hellos e o tempo de espera antes de considerar o vizinho morto (hello/dead timer), e, dependendo do tipo de rede, outros parâmetros como máscara de sub-rede.

Uma vez que os Hellos confirmam compatibilidade, os roteadores trocam suas bases de dados de estado de enlace, sincronizando o conhecimento de topologia, até chegarem no estado `FULL` — adjacência completamente formada, o mesmo estado que você conferiu com `show ip ospf neighbor` em cada lab que envolveu OSPF.

## Por que dividir em áreas

Numa rede OSPF de área única, todo roteador precisa manter o mapa completo de toda a rede, e qualquer mudança de topologia — um link caindo e subindo, por exemplo — dispara recálculo de SPF em absolutamente todo roteador do domínio, não importa quão distante fisicamente ele esteja da mudança. Isso funciona bem em redes pequenas, mas degrada rapidamente conforme a rede cresce: mais roteadores significam bases de dados maiores, mais processamento, e instabilidade em qualquer ponto se propagando pra rede inteira.

Dividir em múltiplas áreas resolve isso limitando o escopo de propagação: a inundação completa de LSAs de tipo 1 (roteador) e tipo 2 (rede), que carregam o detalhe topológico completo, fica restrita aos limites de cada área. O que atravessa a fronteira entre áreas, através de um roteador de fronteira (Area Border Router, o papel que R-FILIAL assumiu no Lab 03), é uma versão resumida — LSAs de tipo 3, que informam apenas quais redes existem do outro lado e a que distância, sem o detalhe interno completo daquela área. Isso significa que uma instabilidade dentro de uma área específica não força recálculo de SPF completo em roteadores de outras áreas — eles só atualizam a informação resumida.

Toda topologia OSPF multiárea precisa ter uma Área 0 (a área de backbone), e toda outra área precisa se conectar, direta ou indiretamente, a essa área de backbone através de um ABR. Essa é uma regra estrutural do protocolo, não uma convenção opcional.

## Custo e escolha de melhor caminho

OSPF calcula o custo de cada link tipicamente como uma função inversa da largura de banda da interface (link mais rápido, custo menor) — embora esse valor possa ser ajustado manualmente quando faz sentido pra engenharia de tráfego específica. O caminho escolhido pelo algoritmo SPF é o de menor custo total acumulado até o destino, não necessariamente o de menor número de saltos — um caminho com mais roteadores no meio, mas todos com links de alta capacidade, pode ter custo total menor do que um caminho mais curto em número de saltos mas com um link lento no meio.

## Tipos de rota que você já viu na prática

Rota intra-área: destino dentro da mesma área do roteador consultando a tabela — a mais "direta" em termos de informação, porque o roteador tem o mapa completo daquele trecho.

Rota inter-área (a que você identificou marcada como `O IA` nos Labs 03 e 08): destino em outra área, aprendida através de um ABR, com informação resumida em vez de detalhe completo.

## Perguntas para reflexão

Se você tivesse uma rede com quinze roteadores, todos numa única área (Área 0), e um link entre dois roteadores no "canto" mais distante da rede começasse a oscilar repetidamente (subindo e descendo), o que aconteceria com o processamento de todos os outros treze roteadores, mesmo os que não têm relação nenhuma direta com esse link específico? E como dividir essa mesma rede em múltiplas áreas mudaria esse cenário?

Segunda pergunta, olhando pro Lab 08: por que faz sentido que uma rede aprendida via OSPF inter-área continue atravessando corretamente uma ACL aplicada na borda de um recurso protegido, sem a ACL precisar "entender" nada sobre áreas OSPF?
