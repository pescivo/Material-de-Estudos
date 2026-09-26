# VLANs e Trunking

Se você já fez o Lab 02, boa parte disso vai soar familiar — de propósito. Esse arquivo existe pra consolidar o conceito de forma mais sistemática do que dá pra fazer no meio de um laboratório com prazo e chamado de cliente fictício puxando sua atenção. Se você ainda não fez o Lab 02, vale ler isso antes.

## O problema que VLAN resolve

Por padrão, todo dispositivo conectado num mesmo switch físico compartilha o mesmo domínio de broadcast — quando um host manda um broadcast (por exemplo, uma requisição ARP perguntando "quem tem o IP X?"), esse quadro é encaminhado pra literalmente todas as outras portas do switch. Numa rede pequena isso não é problema. Numa rede de centenas de dispositivos, com departamentos diferentes que não têm motivo nenhum pra "ouvir" o tráfego uns dos outros, isso vira ineficiência e, mais sério, risco de segurança — foi exatamente esse ponto que a auditoria fictícia da NexaCorp levantou no Lab 02.

VLAN (Virtual Local Area Network) resolve isso segmentando logicamente um switch físico em múltiplos domínios de broadcast independentes, sem precisar de switch físico separado pra cada segmento. Do ponto de vista lógico, cada VLAN se comporta como se fosse uma rede local completamente separada — mesmo compartilhando o mesmo hardware físico por baixo.

## Porta de acesso (access) vs porta trunk

Uma porta configurada como **access** pertence a uma única VLAN, e o host conectado nela nunca vê nem precisa saber que VLAN existe — pra ele, é só uma rede local normal. Isso é o que você configurou em cada porta ligada diretamente a PC-FIN, PC-TI e PC-VENDAS no Lab 02.

Uma porta **trunk**, por outro lado, é projetada pra carregar tráfego de múltiplas VLANs simultaneamente através de um único link físico — geralmente usada entre dois switches, ou entre um switch e um roteador que precisa rotear entre várias VLANs. Pra isso funcionar, cada quadro que atravessa um link trunk precisa carregar uma identificação de qual VLAN ele pertence — e é aí que entra o 802.1Q.

## 802.1Q: a etiqueta que torna o trunk possível

802.1Q é o padrão que define como um switch insere uma tag (etiqueta) de 4 bytes dentro do cabeçalho de um quadro Ethernet, identificando a qual VLAN aquele quadro pertence, antes de enviá-lo por uma porta trunk. Isso acontece de forma completamente transparente pro host final — ele nunca vê nem gera essa tag; é o switch que adiciona a tag na saída de uma porta de acesso em direção a um trunk, e remove a tag antes de entregar o quadro numa porta de acesso do outro lado.

Isso responde uma pergunta que muita gente tem, mas raramente formula com clareza: como múltiplas VLANs conseguem trafegar por um único cabo físico sem se misturar? A resposta é essa tag — ela funciona como um "envelope com remetente identificado" que permite ao switch do outro lado saber pra qual domínio de broadcast entregar aquele quadro específico, mesmo com vários chegando misturados pelo mesmo fio.

## VLAN nativa: o detalhe que costuma confundir

Existe um conceito específico chamado VLAN nativa — a VLAN que, por convenção do padrão 802.1Q, trafega **sem** tag num link trunk. Historicamente isso existia por compatibilidade com equipamentos antigos que não entendiam 802.1Q. Na prática moderna, isso é mais frequentemente citado como um risco de segurança do que como uma funcionalidade útil: existe uma classe de ataque (VLAN hopping) que explora justamente o tráfego não-tagueado da VLAN nativa pra tentar pular entre segmentos que deveriam estar isolados. Prática recomendada, que você vai ver reforçada quando chegar na trilha de Cybersecurity: nunca deixar a VLAN nativa como a VLAN 1 padrão de fábrica, e idealmente configurar uma VLAN nativa dedicada, sem hosts nela, exclusivamente pra esse propósito.

## Inter-VLAN routing: colocando VLANs pra conversar entre si

VLANs, por definição, são domínios de broadcast isolados — hosts em VLANs diferentes não se enxergam diretamente, mesmo estando fisicamente no mesmo switch. Isso é a funcionalidade, não um bug. Mas raramente uma empresa quer isolamento absoluto entre departamentos; geralmente existe necessidade legítima de tráfego controlado entre eles (acesso a um servidor compartilhado, por exemplo). Isso exige roteamento entre VLANs — e existem duas formas clássicas de fazer isso, que já apareceram nesse material:

**Router-on-a-Stick** — um roteador físico separado, ligado ao switch por uma única porta configurada como trunk, com subinterfaces lógicas (uma pra cada VLAN) fazendo o papel de gateway. É o método clássico em ambiente Cisco com switch puramente L2.

**SVI (Switch Virtual Interface)** — quando o próprio switch tem capacidade de camada 3 (um switch multilayer, ou, no caso desse material, o Arista vEOS), você cria uma interface virtual dentro do switch pra cada VLAN, com IP próprio, e o roteamento acontece internamente, sem precisar de roteador físico separado. Foi esse o método usado no Lab 02 — e o detalhe que mais gente esquece é que, além de criar a SVI, é necessário habilitar `ip routing` em modo global; sem isso, a SVI existe e responde normalmente, mas o switch nunca roteia tráfego entre VLANs diferentes.

## Perguntas para reflexão

Se um host na VLAN 10 envia um quadro de broadcast, e esse host está conectado a um switch de acesso ligado por trunk a um switch núcleo — até onde exatamente esse broadcast se propaga? Ele atravessa o trunk? Ele chega em hosts de outras VLANs do outro lado?

Segunda pergunta, ligada direto ao Lab 02: por que colocar SVI e IP de roteamento no switch de acesso (em vez de centralizar isso só no switch núcleo) seria uma decisão de design questionável, mesmo que tecnicamente funcionasse?
