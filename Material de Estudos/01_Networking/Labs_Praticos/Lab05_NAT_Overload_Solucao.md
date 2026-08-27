# Lab 05 — Solução Recomendada

⚠️ Gabarito de instrutor. Esse lab junta NAT com o OSPF do Lab 03 — se você ainda não tentou sozinho, principalmente a parte de propagar a rota padrão pro resto da topologia, volta lá primeiro. É fácil configurar o NAT perfeitamente em R-MATRIZ e mesmo assim o resto da empresa continuar sem internet, só porque esse passo específico ficou de fora.

---

## Passo a passo — R-ISP (simulando o provedor/internet)

```
enable
configure terminal
hostname R-ISP

interface Ethernet1
   description LINK-PARA-NEXACORP
   no switchport
   ip address 203.0.113.2/30
   no shutdown

interface Loopback0
   description SERVIDOR-DE-TESTE-SIMULADO
   ip address 200.200.200.10/32

end
write memory
```

R-ISP não precisa de nenhuma rota adicional além do que já está diretamente conectado — porque, do ponto de vista dele, 203.0.113.0/30 já é uma rede conhecida (interface local), e é justamente pra esse endereço que o NAT vai traduzir todo o tráfego saindo da NexaCorp. R-ISP nunca vai saber que existem redes 192.168.x.x por trás — e essa é exatamente a ideia central do NAT: esconder o endereçamento interno do lado de fora.

## Passo a passo — R-MATRIZ

Nova interface, voltada ao provedor:

```
enable
configure terminal

interface Ethernet2
   description LINK-PARA-PROVEDOR
   no switchport
   ip address 203.0.113.1/30
   no shutdown
```

Rota padrão apontando pro provedor:

```
ip route 0.0.0.0/0 203.0.113.2
```

Propagação da rota padrão para dentro do OSPF — esse é o passo que resolve a questão de Filial e Data Center também conseguirem sair pra internet:

```
router ospf 1
   default-information originate
```

Sem esse comando específico, a rota `0.0.0.0/0` existe só localmente em R-MATRIZ e nunca é anunciada pros outros roteadores do domínio OSPF — R-FILIAL e R-DATACENTER simplesmente não saberiam que existe um caminho pra qualquer coisa fora da topologia interna deles.

Definindo quais redes internas são elegíveis pra tradução NAT:

```
ip access-list NAT-INSIDE
   10 permit ip 192.168.10.0/24 any
   20 permit ip 192.168.20.0/24 any
   30 permit ip 192.168.30.0/24 any
   40 permit ip 192.168.11.0/24 any
   50 permit ip 192.168.12.0/24 any
   60 permit ip 192.168.13.0/24 any
```

E aplicando o NAT com overload na interface de saída:

```
interface Ethernet2
   ip nat source dynamic access-list NAT-INSIDE overload
```

```
end
write memory
```

**Se você estiver em Cisco IOS**, a lógica equivalente é um pouco mais verbosa, com o conceito de interface `inside`/`outside` explícito:

```
ip access-list extended NAT-INSIDE
 permit ip 192.168.10.0 0.0.0.255 any
 permit ip 192.168.20.0 0.0.0.255 any
 permit ip 192.168.30.0 0.0.0.255 any
 permit ip 192.168.11.0 0.0.0.255 any
 permit ip 192.168.12.0 0.0.0.255 any
 permit ip 192.168.13.0 0.0.0.255 any

interface GigabitEthernet0/2
 ip nat outside

interface GigabitEthernet0/0
 ip nat inside
! (repetir ip nat inside em toda interface voltada pra dentro da rede)

ip nat inside source list NAT-INSIDE interface GigabitEthernet0/2 overload
```

## Validação

```
PC-MATRIZ> ping 200.200.200.10
PC-FILIAL> ping 200.200.200.10
PC-DATACENTER> ping 200.200.200.10
```

Os três devem responder — mesmo PC-DATACENTER, que está a dois roteadores de distância de R-MATRIZ e três hops de distância do próprio provedor.

Em R-MATRIZ, o comando que mostra a tradução acontecendo em tempo real:

```
show ip nat translations
```

Depois de rodar os três pings acima, você deve ver uma entrada de tradução pra cada host de origem diferente, todas mostrando o IP público 203.0.113.1 do lado de fora, mas com portas de origem diferentes entre si — é isso que permite múltiplas conexões simultâneas compartilhando um único IP público.

## Resposta às perguntas de reflexão

**O que o overload/PAT muda em relação a NAT dinâmico comum?** NAT dinâmico tradicional (sem overload) mantém uma relação de um-para-um entre IP interno e IP público — se você tem só um IP público disponível, só um host interno consegue usar internet por vez, o que é inviável numa empresa. O overload resolve isso adicionando a porta de origem (porta TCP ou UDP) como parte da informação usada pra diferenciar conexões, além do próprio IP. Isso permite que centenas de hosts internos compartilhem o mesmo IP público simultaneamente, porque cada conexão é identificada de forma única pela combinação IP+porta, não só pelo IP sozinho.

**Como o pacote de volta sabe pra qual host interno pertence?** R-MATRIZ mantém uma tabela de tradução (exatamente o que `show ip nat translations` exibe) relacionando cada combinação IP interno + porta interna com a porta pública específica que foi usada na tradução de saída. Quando o pacote de resposta chega do provedor endereçado a 203.0.113.1 numa porta específica, R-MATRIZ consulta essa tabela, encontra a entrada correspondente àquela porta exata, e reescreve o pacote de volta pro IP e porta internos originais antes de encaminhar pra dentro da rede. É esse mapeamento de porta que faz o "compartilhamento" de um único IP público funcionar sem confundir tráfego de hosts diferentes.

## Diagnóstico do cenário bônus — rede do Data Center fora da ACL de NAT

Se você removeu a linha `30 permit ip 192.168.30.0/24 any` da ACL `NAT-INSIDE`, o roteamento continua perfeito — um `traceroute` a partir de PC-DATACENTER mostraria o pacote saindo corretamente até R-MATRIZ e potencialmente até tentando sair pela interface do provedor. O problema não está no caminho, está na tradução.

Sem essa linha na ACL, tráfego originado em 192.168.30.0/24 não é mais considerado "tráfego interno elegível" pra NAT — o pacote chega em R-MATRIZ, tenta sair pela interface Ethernet2 com o endereço de origem privado original (192.168.30.10, por exemplo), sem nenhuma tradução acontecer. Um endereço privado (faixa RFC 1918) nunca deveria trafegar pela internet pública de verdade — na prática, mesmo que esse laboratório simulado não bloqueie isso explicitamente, é assim que o comportamento se manifestaria num cenário real: o provedor descartaria esse pacote, porque endereço privado não é roteável na internet.

O comando certo pra confirmar isso rapidamente:

```
show ip nat translations
```

Depois de tentar pingar a partir de PC-DATACENTER, você não veria **nenhuma** entrada de tradução envolvendo a rede 192.168.30.0/24 — mesmo com Matriz e Filial aparecendo normalmente. Ausência total de entrada na tabela de tradução, com roteamento comprovadamente funcional, aponta diretamente pra uma exclusão na definição de tráfego elegível — não pra um problema de rota, nem de OSPF, nem de interface caída.

Correção:

```
configure terminal
ip access-list NAT-INSIDE
   30 permit ip 192.168.30.0/24 any
```

## O que você deveria levar desse lab

A lição técnica principal é sobre overload/PAT e sobre como uma rota padrão precisa ser explicitamente propagada pra dentro de um domínio de roteamento dinâmico — ela não se espalha sozinha só porque existe. Mas a lição de diagnóstico é a mais valiosa: quando roteamento está confirmadamente correto e mesmo assim algo não sai, o próximo lugar a olhar não é "mais uma vez a rota", é a camada seguinte — nesse caso, a política de tradução. Separar "isso é problema de caminho" de "isso é problema de política aplicada sobre um caminho que já funciona" é uma habilidade que vai se repetir bastante daqui pra frente, inclusive quando você chegar em firewall e segurança de perímetro na trilha de Cybersecurity.

Documenta esse lab e segue pro Lab 06.
