# Lab 01 — Solução Recomendada

⚠️ Isso aqui é gabarito de instrutor, não um atalho. Se você ainda não tentou o `Lab01_Topologia_Basica_Desafio.md` por conta própria, volta lá primeiro. O valor desse lab está no tempo que você passa travado, não na configuração em si — que, sinceramente, é bem curta.

---

## Recapitulando o que precisava ser feito

Interfaces configuradas dos dois lados, hosts com IP e gateway corretos, rota estática nos dois roteadores apontando pra rede remota, e conectividade validada fim a fim.

## Passo a passo — R-MATRIZ (sintaxe Arista EOS)

```
enable
configure terminal
hostname R-MATRIZ

interface Ethernet0/0
   description LAN-MATRIZ
   no switchport
   ip address 192.168.10.1/24
   no shutdown

interface Ethernet0/1
   description LINK-PARA-FILIAL
   no switchport
   ip address 10.0.0.1/30
   no shutdown

ip route 192.168.20.0/24 10.0.0.2

end
write memory
```

**Se você estiver usando Cisco IOS (Packet Tracer ou CML)**, a lógica é idêntica, só muda a sintaxe de interface e de rota:

```
interface GigabitEthernet0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown

interface GigabitEthernet0/1
 ip address 10.0.0.1 255.255.255.252
 no shutdown

ip route 192.168.20.0 255.255.255.0 10.0.0.2
```

## Passo a passo — R-FILIAL (Arista EOS)

```
enable
configure terminal
hostname R-FILIAL

interface Ethernet0/1
   description LINK-PARA-MATRIZ
   no switchport
   ip address 10.0.0.2/30
   no shutdown

interface Ethernet0/0
   description LAN-FILIAL
   no switchport
   ip address 192.168.20.1/24
   no shutdown

ip route 192.168.10.0/24 10.0.0.1

end
write memory
```

Repara na simetria: cada roteador tem uma rota estática só, apontando pra rede que ele *não* enxerga diretamente, usando como next-hop o IP da outra ponta do link. Esse é o padrão mental que você vai usar em praticamente toda configuração de rota estática simples daqui pra frente.

## Configuração dos hosts (VPCS, dentro do GNS3)

```
PC-MATRIZ> ip 192.168.10.10/24 192.168.10.1
PC-FILIAL> ip 192.168.20.10/24 192.168.20.1
```

## Validação

No PC-MATRIZ:

```
PC-MATRIZ> ping 192.168.20.10
```

Você deve ver resposta com sucesso (algo como `84 bytes from 192.168.20.10... time=Xms`). Repita o teste no sentido contrário a partir do PC-FILIAL.

Nos roteadores, dois comandos que valem a pena rodar pra confirmar que a rota está correta, não só que o ping funcionou:

```
show ip route
show ip interface brief
```

`show ip route` te mostra a tabela de roteamento completa — você deve ver a rede remota marcada como rota estática (geralmente identificada com `S` no início da linha), com o next-hop correto. Esse comando vai ser seu melhor amigo em praticamente todo troubleshooting de camada 3 daqui pra frente.

## Resposta às perguntas de reflexão

**O que acontece com um pacote sem rota configurada?** O roteador descarta o pacote (drop) e, dependendo da configuração, pode gerar um ICMP "Destination Unreachable" de volta pra origem. Sem rota estática nem dinâmica cobrindo a rede de destino, e sem rota padrão configurada, não existe caminho conhecido — o roteador simplesmente não sabe pra onde encaminhar.

**Por que o next-hop é o IP do outro lado do link?** Porque rota estática funciona respondendo à pergunta "pra chegar nessa rede de destino, eu preciso mandar o pacote pra qual endereço diretamente alcançável?". Nesse cenário, o único caminho físico entre Matriz e Filial é o link ponto a ponto — então o próximo salto só pode ser o IP da interface do outro roteador, do outro lado desse link.

## Diagnóstico do cenário bônus — a rota alterada

Se você seguiu a proposta e trocou o next-hop de `10.0.0.2` para `10.0.0.3` em R-MATRIZ, o ping do PC-MATRIZ pro PC-FILIAL volta a falhar. Veja como um profissional de verdade investigaria isso, na ordem certa:

**Primeiro comando: `show ip route`.** Isso é sempre o primeiro passo quando conectividade entre redes para de funcionar — antes até de mexer em cabo ou reiniciar qualquer coisa. Você veria a rota pra 192.168.20.0/24 apontando pra 10.0.0.3, um endereço que não existe dentro da /30 que você configurou (a /30 só comporta 10.0.0.1 e 10.0.0.2 como hosts válidos).

**Segundo passo, se não estivesse óbvio de cara:** rodar `ping 10.0.0.3` a partir de R-MATRIZ. Esse ping falharia, porque não existe nada respondendo naquele IP — e isso é o sinal de que o problema não está na LAN nem no link físico, está especificamente na rota configurada apontando pro lugar errado.

A correção é reaplicar a rota com o next-hop certo. Em Arista EOS, como não existe rota duplicada por padrão nesse contexto simples, basta reconfigurar:

```
configure terminal
no ip route 192.168.20.0/24 10.0.0.3
ip route 192.168.20.0/24 10.0.0.2
```

## O que voce deveria levar desse lab

Esse foi o lab mais simples da trilha, de propósito — é o primeiro. Mas o raciocínio de diagnóstico que você praticou aqui (`show ip route` como primeiro passo, isolar se o problema é de rota ou de alcançabilidade do próprio next-hop) é o mesmo raciocínio que você vai aplicar em cenário complexo de OSPF multi-área lá na frente, só que com mais camadas de complexidade em cima. Guarda esse hábito.

Documenta esse lab na sua planilha de progresso antes de ir pro Lab 02.
