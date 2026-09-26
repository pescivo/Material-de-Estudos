# BGP — Fundamentos

Diferente dos outros arquivos de teoria dessa pasta, esse aqui não tem lab prático correspondente na trilha até agora — BGP é um protocolo de escala e contexto diferentes dos que você configurou até aqui, e simular ele de forma realista exige uma topologia que foge do escopo desse material introdutório. Isso não diminui a importância de entender o conceito: BGP é, sem exagero, o protocolo que sustenta a internet como rede única e funcional. Vale entender pelo menos os fundamentos, mesmo sem lab de mão na massa por enquanto.

## O que BGP resolve, e por que OSPF não serve pra isso

OSPF (e a maioria dos outros protocolos de roteamento interno) foi desenhado pra funcionar dentro de uma única organização — um Sistema Autônomo (Autonomous System, ou AS), que é, na prática, um conjunto de redes sob administração única, com uma política de roteamento coerente e centralizada. A NexaCorp, na sua topologia inteira construída ao longo dos dez labs, é um Sistema Autônomo único, mesmo com múltiplos sites e múltiplas áreas OSPF internas.

A internet, por outro lado, é a interconexão de dezenas de milhares de Sistemas Autônomos diferentes — cada provedor de internet, cada grande empresa com conectividade própria, cada instituição acadêmica de porte, todos operando sua própria rede, com suas próprias políticas, e precisando trocar informação de roteamento entre si de forma que nenhum deles precise confiar cegamente nos outros. Esse é um problema de natureza completamente diferente do que OSPF resolve — não é sobre calcular o caminho tecnicamente mais eficiente dentro de uma rede controlada, é sobre negociar, entre organizações independentes, política de conectividade, sem que uma organização tenha visibilidade ou controle total sobre a rede da outra.

BGP (Border Gateway Protocol) é o protocolo criado especificamente pra esse cenário: roteamento entre Sistemas Autônomos diferentes.

## eBGP vs iBGP

BGP opera em dois contextos distintos, com o mesmo protocolo mas propósitos diferentes:

**eBGP (External BGP)** — troca de rotas entre Sistemas Autônomos diferentes. É o que dois provedores de internet diferentes, ou uma empresa e seu provedor, usam pra trocar informação de quais redes cada lado consegue alcançar. Foi conceitualmente esse tipo de relação que você simulou, de forma simplificada com rota estática, entre R-MATRIZ e R-ISP no Lab 05 — numa rede real de maior porte, esse relacionamento provavelmente seria estabelecido via eBGP, não rota estática manual, especialmente se a empresa tivesse mais de um provedor de internet (o que é chamado de multihoming, uma prática comum em empresas que não podem tolerar indisponibilidade de conectividade).

**iBGP (Internal BGP)** — troca de rotas BGP entre roteadores dentro do mesmo Sistema Autônomo. Usado tipicamente quando uma organização grande, com presença significativa na internet (como um provedor de internet, ou uma empresa muito grande com múltiplos pontos de conexão externa), precisa propagar rotas aprendidas via BGP internamente entre seus próprios roteadores.

## Como BGP decide o melhor caminho

Diferente de OSPF, que otimiza primariamente por custo técnico (largura de banda), BGP é fundamentalmente um protocolo orientado a **política**, não só a eficiência técnica. A decisão de qual caminho usar entre múltiplos possíveis considera uma sequência de atributos — alguns dos mais relevantes incluem preferência local configurada manualmente (refletindo acordo comercial ou preferência operacional da organização), o comprimento do caminho de ASes atravessados (AS-Path, uma medida de "quantas organizações diferentes" o tráfego passaria por), e origem da rota, entre outros critérios.

Isso significa que o caminho "mais curto" tecnicamente nem sempre é o caminho escolhido — uma organização pode, por exemplo, preferir deliberadamente rotear tráfego por um caminho com mais saltos entre Sistemas Autônomos, porque esse caminho passa por um provedor com quem existe um acordo comercial mais vantajoso. Esse é um dos pontos que mais surpreende quem vem de uma formação só em roteamento interno (OSPF, EIGRP): BGP não é "OSPF, só que maior" — é uma ferramenta que resolve um problema de natureza diferente, onde política de negócio é literalmente parte do algoritmo de decisão.

## Escala: por que BGP precisa ser assim

A tabela de rotas BGP da internet, na visão de um roteador de borda de um provedor de internet de porte real, contém centenas de milhares de rotas — a internet inteira, resumida. Isso é uma escala completamente diferente de qualquer coisa que OSPF foi desenhado pra suportar (OSPF degrada significativamente muito antes de chegar perto dessa escala, justamente pelo motivo que você já viu no arquivo de fundamentos de OSPF: cada roteador mantendo mapa de topologia completo). BGP foi desenhado desde o início considerando esse volume, com mecanismos de agregação de rota (resumir múltiplas redes menores numa única entrada maior) sendo parte essencial de como ele se mantém operável nessa escala.

## Por que isso importa pra você, mesmo sem configurar ainda

Você provavelmente não vai configurar BGP no seu primeiro (nem segundo) emprego em rede — é uma habilidade tipicamente associada a profissionais mais seniores, operando em provedores de internet ou empresas de porte considerável com conectividade redundante a múltiplos provedores. Mas entender o conceito, mesmo sem prática hands-on ainda, muda como você enxerga a internet como um todo: ela não é uma rede única e centralizada — é uma federação de milhares de redes independentes, cada uma tomando decisões próprias sobre como e com quem trocar tráfego, coordenadas por um protocolo que assume desconfiança mútua como ponto de partida, não confiança automática.

## Perguntas para reflexão

Por que você acha que seria problemático, ou até perigoso, se BGP simplesmente confiasse automaticamente em qualquer rota anunciada por qualquer Sistema Autônomo vizinho, sem nenhum mecanismo de verificação ou política aplicada? (Dica: pensa no que aconteceria se um Sistema Autônomo, por erro de configuração ou má intenção, anunciasse ser dono de um bloco de endereço que na verdade pertence a outra organização.)

Segunda pergunta: dado que BGP prioriza política de negócio sobre eficiência técnica pura, em que situação prática uma empresa escolheria deliberadamente um caminho de rede mais longo (em número de saltos entre ASes) em vez do caminho tecnicamente mais direto disponível?
