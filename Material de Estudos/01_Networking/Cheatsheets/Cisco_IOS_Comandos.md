# Cheatsheet — Comandos Cisco IOS

Isso aqui não substitui entender o que cada comando faz — se você não sabe por que tá digitando algo, volta pro lab ou pro roadmap antes. Isso é só pra consulta rápida quando você já entende o conceito e só precisa lembrar a sintaxe exata.

Uso principalmente Arista EOS nos labs (motivo: CML da Cisco com IOS completo é pago, vEOS é gratuito), mas boa parte do público estuda em Packet Tracer ou CML-Free, que rodam Cisco IOS de verdade. Esse cheatsheet cobre a sintaxe Cisco. Se você quer a comparação lado a lado entre as duas plataformas, isso está no arquivo `Arista_EOS_vs_Cisco_IOS.md`, aqui do lado.

---

## Modos de operação

```
Router>                          modo usuário (visualização limitada)
Router> enable                   entra no modo privilegiado
Router#                          modo privilegiado (visualização completa)
Router# configure terminal
Router(config)#                  modo de configuração global
Router(config)# interface Gi0/0
Router(config-if)#               modo de configuração de interface
```

## Configuração básica

```
hostname R-MATRIZ                 define o nome do dispositivo
enable secret SENHA                senha criptografada pro modo privilegiado
line console 0
 password SENHA
 login
line vty 0 4
 password SENHA
 login
 transport input ssh
service password-encryption       criptografa senhas em texto plano na config
```

## Interfaces

```
interface GigabitEthernet0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown
 description LAN-MATRIZ
```

```
show ip interface brief           status resumido de todas as interfaces
show interfaces GigabitEthernet0/0   detalhe completo de uma interface específica
```

## VLAN e Trunking (ver Lab 02)

```
vlan 10
 name FINANCEIRO

interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10

interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
```

```
show vlan brief                   VLANs existentes e portas associadas
show interfaces trunk             VLANs realmente ativas num link trunk
```

**Inter-VLAN routing no Cisco IOS** — diferente do Arista, que usa SVI direto no switch. No Cisco clássico (com switch L2 puro + roteador separado), é Router-on-a-Stick, com subinterfaces:

```
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.11.1 255.255.255.0

interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.12.1 255.255.255.0
```

Se o seu switch for um Multilayer Switch (L3), a lógica de SVI é igual à do Arista:

```
interface Vlan10
 ip address 192.168.11.1 255.255.255.0
 no shutdown
ip routing
```

## Rota estática (ver Lab 01)

```
ip route 192.168.20.0 255.255.255.0 10.0.0.2
ip route 0.0.0.0 0.0.0.0 203.0.113.2      rota padrão
```

```
show ip route                     tabela de rotas completa
show ip route static              só as rotas estáticas
```

## OSPF (ver Lab 03)

```
router ospf 1
 network 192.168.10.0 0.0.0.255 area 0
 network 10.0.0.0 0.0.0.3 area 0
 default-information originate
```

Repara que o Cisco IOS usa **wildcard mask** (máscara invertida) no `network`, não a máscara normal. É uma das pegadinhas mais clássicas pra quem tá começando: `0.0.0.255` numa rede /24, não `255.255.255.0`.

```
show ip ospf neighbor             estado das adjacências
show ip ospf interface Gi0/1      área e configuração OSPF de uma interface específica
show ip protocols                 resumo de todos os protocolos de roteamento ativos
```

## ACL (ver Lab 04)

```
ip access-list extended DATACENTER-PROTECAO
 permit ip 192.168.11.0 0.0.0.255 192.168.30.0 0.0.0.255
 permit ip 192.168.12.0 0.0.0.255 192.168.30.0 0.0.0.255
 deny ip 192.168.13.0 0.0.0.255 192.168.30.0 0.0.0.255
 permit ip any any

interface GigabitEthernet0/1
 ip access-group DATACENTER-PROTECAO in
```

```
show access-lists                 conteúdo e contadores de hit de todas as ACLs
show ip interface Gi0/1           mostra, entre outras coisas, qual ACL está aplicada e em qual direção
```

Lembre sempre: ACL estendida tem deny-all implícito no final. E a ordem das regras importa — a primeira que casar com o pacote decide, o resto nem é avaliado.

## NAT (ver Lab 05)

```
interface GigabitEthernet0/2
 ip nat outside

interface GigabitEthernet0/0
 ip nat inside

ip access-list extended NAT-INSIDE
 permit ip 192.168.10.0 0.0.0.255 any
 permit ip 192.168.20.0 0.0.0.255 any

ip nat inside source list NAT-INSIDE interface GigabitEthernet0/2 overload
```

```
show ip nat translations          traduções ativas em tempo real
show ip nat statistics             estatística geral de uso do NAT
```

## Salvando configuração

```
copy running-config startup-config
```
ou, forma mais curta:
```
write memory
```
ou ainda mais curta:
```
wr
```

Se você reiniciar o dispositivo sem salvar, tudo que você configurou desde o último `write memory` some. Isso já aconteceu com todo mundo pelo menos uma vez — inclusive comigo.

## Comandos de diagnóstico gerais

```
ping <ip>
traceroute <ip>
show running-config               configuração ativa completa
show running-config interface Gi0/0   configuração ativa só de uma interface
show version                      versão do IOS, tempo de atividade, hardware
show cdp neighbors                 dispositivos Cisco vizinhos detectados diretamente
```

---

Se algum comando aqui não bater exatamente com a versão do IOS que você está usando (Packet Tracer simplifica alguns comandos em relação a um IOS real), não é erro seu — Packet Tracer não implementa o conjunto de comandos IOS por completo. Isso é esperado.
