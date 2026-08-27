# Lab 02 — Solução Recomendada

⚠️ Gabarito de instrutor. Se você ainda não tentou o `Lab02_VLANs_e_InterVLAN_Routing_Desafio.md` sozinho, volta lá. Esse lab em especial rende muito mais aprendizado quando você erra a ordem de configuração umas duas vezes antes de chegar aqui.

---

## Passo a passo — SW-CORE (Arista EOS)

Primeiro, criar as VLANs:

```
enable
configure terminal
hostname SW-CORE

vlan 10
   name FINANCEIRO
vlan 20
   name TI
vlan 30
   name VENDAS
```

Agora o trunk, na porta que liga ao SW-ACESSO:

```
interface Ethernet1
   description TRUNK-PARA-SW-ACESSO
   switchport mode trunk
   switchport trunk allowed vlan 10,20,30
```

Esse comando `switchport trunk allowed vlan` é o ponto que mais gente esquece — sem ele, dependendo da configuração padrão da plataforma, o trunk pode não estar carregando as VLANs que você acabou de criar, mesmo com o modo trunk habilitado corretamente.

Agora as interfaces virtuais (SVI) e o roteamento entre elas:

```
interface Vlan10
   ip address 192.168.11.1/24
   no shutdown

interface Vlan20
   ip address 192.168.12.1/24
   no shutdown

interface Vlan30
   ip address 192.168.13.1/24
   no shutdown

ip routing
```

O comando `ip routing`, em modo de configuração global, é a parte que responde à pergunta que eu deixei em aberto no desafio: sem ele, o Arista EOS trata o switch como puramente camada 2, mesmo com as SVIs criadas e com IP configurado nelas. É esse comando que liga o roteamento entre VLANs de fato. É um erro comum — e fácil de não perceber, porque as SVIs sobem normalmente (`no shutdown`), só o roteamento entre elas que não acontece.

```
end
write memory
```

## Passo a passo — SW-ACESSO (Arista EOS, atuando só como L2)

```
enable
configure terminal
hostname SW-ACESSO

vlan 10
   name FINANCEIRO
vlan 20
   name TI
vlan 30
   name VENDAS

interface Ethernet1
   description TRUNK-PARA-SW-CORE
   switchport mode trunk
   switchport trunk allowed vlan 10,20,30

interface Ethernet2
   description ACESSO-FINANCEIRO
   switchport mode access
   switchport access vlan 10

interface Ethernet3
   description ACESSO-TI
   switchport mode access
   switchport access vlan 20

interface Ethernet4
   description ACESSO-VENDAS
   switchport mode access
   switchport access vlan 30

end
write memory
```

Repara que o SW-ACESSO nunca recebe `ip routing` nem SVI com endereço — ele não precisa, e não deveria, porque o roteamento é responsabilidade do SW-CORE nesse desenho. Colocar SVI e IP no switch de acesso, mesmo que funcione, foge do que foi pedido no chamado e cria complexidade desnecessária de manter depois.

## Configuração dos hosts (VPCS)

```
PC-FIN> ip 192.168.11.10/24 192.168.11.1
PC-TI> ip 192.168.12.10/24 192.168.12.1
PC-VENDAS> ip 192.168.13.10/24 192.168.13.1
```

## Validação

```
PC-FIN> ping 192.168.12.10
PC-FIN> ping 192.168.13.10
```

Ambos devem responder — isso confirma que o tráfego saiu da VLAN 10, atravessou o trunk até o SW-CORE, foi roteado pela SVI, e voltou pelo mesmo caminho até a VLAN de destino.

No SW-CORE, dois comandos pra confirmar que a configuração está correta de fato, não só que o ping funcionou:

```
show vlan brief
show interfaces trunk
show ip route
```

`show vlan brief` mostra as VLANs existentes e quais portas estão associadas a cada uma. `show interfaces trunk` é o comando que responde à pergunta que ficou em aberto no desafio — ele mostra especificamente quais VLANs estão realmente permitidas e ativas num link trunk, que é uma informação diferente de "quais VLANs existem no switch". `show ip route` deve listar as três redes (192.168.11.0/24, .12.0/24, .13.0/24) como rotas diretamente conectadas, uma pra cada SVI.

## Resposta à pergunta de reflexão

Um quadro saindo do PC-FIN (VLAN 10) chega na porta de acesso Eth2 do SW-ACESSO sem nenhuma marcação de VLAN — a porta de acesso é quem "sabe" que aquele tráfego pertence à VLAN 10, porque foi configurada assim. Ao encaminhar esse quadro pro trunk (Eth1) rumo ao SW-CORE, o switch adiciona uma tag 802.1Q identificando a VLAN de origem. Do outro lado, o SW-CORE lê essa tag, entrega o quadro internamente pra interface virtual Vlan10 correspondente, e a partir daí o processo é roteamento normal de camada 3. A tag 802.1Q é removida antes do quadro sair por uma porta de acesso do lado de destino — o host final nunca vê marcação nenhuma, só o link trunk no meio do caminho carrega essa informação.

## Diagnóstico do cenário bônus — VLAN removida do trunk

Se você seguiu a proposta e tirou a VLAN 30 da allowed list do trunk (digamos, no SW-ACESSO), o PC-VENDAS para de conseguir pingar até o próprio gateway (192.168.13.1), mesmo com toda a configuração de IP correta nele.

O comando certo pra investigar isso rápido é `show interfaces trunk`, não `show vlan brief`. A diferença importa: `show vlan brief` continua mostrando a VLAN 30 existindo normalmente no switch, com a porta Eth4 associada a ela — porque a VLAN em si nunca foi apagada, só removida da lista de VLANs permitidas no trunk. Se você checasse só isso, ia concluir erroneamente que "a VLAN está ok" e perder tempo procurando o problema em outro lugar, tipo na configuração de IP do host.

`show interfaces trunk` mostra exatamente quais VLANs estão na "allowed list" de cada porta trunk — e é ali que você veria a VLAN 30 ausente, mesmo com Eth1 em modo trunk e up.

A correção:

```
interface Ethernet1
   switchport trunk allowed vlan 10,20,30
```

## O que você deveria levar desse lab

A lição técnica é sobre trunk, SVI e o comando `ip routing`. Mas a lição maior é sobre diagnóstico: existem comandos que parecem responder a mesma pergunta mas na verdade respondem perguntas ligeiramente diferentes (`show vlan brief` vs `show interfaces trunk`), e escolher o comando errado te faz perder tempo checando a coisa errada. Isso vale pra praticamente todo troubleshooting de rede que vem depois — inclusive quando você chegar em análise de tráfego na trilha de Cybersecurity.

Documenta esse lab e segue pro Lab 03.
