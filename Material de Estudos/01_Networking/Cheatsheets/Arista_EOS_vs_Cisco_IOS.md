# Cheatsheet — Arista EOS x Cisco IOS

Eu escolhi Arista vEOS como plataforma principal desse material por um motivo prático, não técnico: é gratuita (só precisa de conta), enquanto o Cisco CML com IOS completo é pago. Mas boa parte de quem estuda por aí usa Packet Tracer ou tem contato profissional com Cisco puro — e as sintaxes divergem em pontos específicos que confundem bastante gente na primeira migração entre as duas. Esse arquivo existe pra resolver exatamente isso.

Não é uma lista exaustiva de todo comando das duas plataformas — é focado nos pontos onde a diferença realmente importa e onde eu vi gente travar.

---

## Estrutura de modos — praticamente idêntica

```
> ... enable ... # ... configure terminal ... (config)#
```

Isso é igual nas duas plataformas. Se você já sabe navegar em um, navega no outro sem esforço nenhum.

## A diferença mais importante: interfaces e switchport

Arista EOS trata toda interface física como potencialmente roteada (camada 3) por padrão, o que exige um comando explícito pra "desligar" o comportamento de switch nela quando você quer usar como interface roteada pura:

| Situação | Arista EOS | Cisco IOS |
|---|---|---|
| Interface roteada (L3) | `no switchport` + `ip address` | `ip address` (já é roteada por padrão em roteador; em switch L3, precisa de `no switchport`) |
| Interface de acesso (L2) | `switchport mode access` | `switchport mode access` |
| Interface trunk (L2) | `switchport mode trunk` | `switchport mode trunk` |

Isso confunde muita gente vindo de Cisco puro (roteador Cisco ISR não tem `switchport` nem `no switchport` — a interface já é roteada por natureza). Em Arista, mesmo rodando num roteador virtual, é comum precisar do `no switchport` pra deixar claro que aquela interface não vai fazer parte de nenhuma VLAN.

## Inter-VLAN routing: a diferença mais conceitual das duas

Essa é a diferença que mais aparece no roadmap e nos labs.

**Cisco (clássico, com switch L2 + roteador separado):** Router-on-a-Stick, usando subinterfaces com encapsulamento 802.1Q:

```
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.11.1 255.255.255.0
```

**Arista EOS (e Cisco também, se o switch for Multilayer/L3):** SVI (Switch Virtual Interface) direto no switch, sem precisar de roteador físico separado:

```
interface Vlan10
   ip address 192.168.11.1/24
   no shutdown

ip routing
```

O ponto que mais gente esquece migrando pra Arista: o comando `ip routing`, em modo de configuração global, é obrigatório pra habilitar roteamento entre VLANs de fato. Sem ele, as SVIs sobem normalmente (você não vê erro nenhum), mas o tráfego simplesmente não é roteado entre elas — foi exatamente o que aconteceu no Lab 02 se você esqueceu desse comando.

## Máscara: CIDR vs Wildcard

Essa é outra fonte clássica de erro pra quem transita entre as duas.

| Contexto | Arista EOS | Cisco IOS |
|---|---|---|
| Endereço de interface | `ip address 192.168.10.1/24` (CIDR direto) | `ip address 192.168.10.1 255.255.255.0` (máscara normal) |
| Rede em OSPF (`network`) | `network 192.168.10.0/24 area 0` (CIDR direto) | `network 192.168.10.0 0.0.0.255 area 0` (wildcard mask — máscara invertida) |
| Rede em ACL | `permit ip 192.168.10.0/24 any` (CIDR direto) | `permit ip 192.168.10.0 0.0.0.255 any` (wildcard mask) |

Arista é consistente: CIDR em praticamente tudo. Cisco IOS clássico mistura máscara normal (em interface) com wildcard mask (em OSPF e ACL) — e wildcard mask é a máscara **invertida**: onde a máscara normal tem `255`, a wildcard tem `0`, e vice-versa. Uma /24 (255.255.255.0) vira wildcard `0.0.0.255`. Esse é, sem exagero, um dos erros mais comuns de quem está aprendendo Cisco — inclusive comigo, no começo.

## Rota estática e rota padrão

Idêntico em espírito, sintaxe levemente diferente:

```
Arista:  ip route 192.168.20.0/24 10.0.0.2
Cisco:   ip route 192.168.20.0 255.255.255.0 10.0.0.2
```

## OSPF

Estrutura de configuração é praticamente igual — a diferença real já foi coberta acima (CIDR vs wildcard mask no `network`). O restante (`default-information originate`, verificação com `show ip ospf neighbor`) é idêntico nas duas plataformas.

## NAT: aqui a diferença é estrutural, não só de sintaxe

Cisco IOS exige que você marque explicitamente cada interface como `ip nat inside` ou `ip nat outside` antes de aplicar a regra de tradução:

```
interface GigabitEthernet0/0
 ip nat inside

interface GigabitEthernet0/2
 ip nat outside

ip nat inside source list NAT-INSIDE interface GigabitEthernet0/2 overload
```

Arista EOS aplica a tradução diretamente na interface de saída, sem precisar declarar `inside`/`outside` em cada interface interna separadamente:

```
interface Ethernet2
   ip nat source dynamic access-list NAT-INSIDE overload
```

Isso significa que, em Arista, você não precisa "marcar" cada interface interna individualmente — a ACL já define quais origens são elegíveis, e a tradução é vinculada só à interface de saída. Menos passos, mas também menos explícito sobre a intenção — vale entender os dois modelos, porque cada plataforma que você encontrar no mercado vai seguir uma lógica ou outra.

## Comandos de verificação — quase todos idênticos

```
show ip route
show ip interface brief
show vlan brief
show interfaces trunk
show ip ospf neighbor
show ip nat translations
show running-config
```

Isso é praticamente copiar e colar entre as duas plataformas — a Arista foi desenhada de propósito pra ter familiaridade com quem já conhece Cisco, exatamente nos comandos de visualização/diagnóstico. As diferenças reais estão quase todas do lado de configuração, não de verificação.

## Salvando configuração

```
Arista:  write memory   (ou copy running-config startup-config, igual ao Cisco)
Cisco:   copy running-config startup-config   (ou write memory / wr, forma abreviada)
```
Praticamente equivalente nas duas.

---

Se você é dos que estuda alternando entre GNS3 com Arista e Packet Tracer com Cisco, o padrão de erro mais comum é justamente CIDR vs wildcard mask e o `ip routing`/`no switchport` do Arista. Vale grifar essas duas seções mentalmente antes de qualquer lab novo.
