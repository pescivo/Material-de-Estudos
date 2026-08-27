# Lab 03 — Solução Recomendada

⚠️ Gabarito de instrutor. OSPF tem mais peça em movimento que os labs anteriores — se você ainda não tentou o `Lab03_OSPF_Multiarea_Desafio.md` sozinho, principalmente a parte de rodar `show ip ospf neighbor` e interpretar o resultado, volta lá primeiro. É essa parte que mais vale a pena treinar.

---

## Passo a passo — R-MATRIZ (Arista EOS)

As interfaces já existiam desde o Lab 01. O que muda aqui é a adição do processo OSPF:

```
enable
configure terminal

router ospf 1
   network 192.168.10.0/24 area 0
   network 10.0.0.0/30 area 0
```

## Passo a passo — R-FILIAL (o roteador de fronteira entre áreas)

```
enable
configure terminal

interface Ethernet2
   description LINK-PARA-DATACENTER
   no switchport
   ip address 10.0.1.1/30
   no shutdown

router ospf 1
   network 10.0.0.0/30 area 0
   network 192.168.20.0/24 area 0
   network 10.0.1.0/30 area 1
```

Repare que R-FILIAL tem três declarações de rede dentro do mesmo processo OSPF, mas com áreas diferentes atribuídas — é exatamente isso que faz dele um ABR (Area Border Router). Não existem dois processos OSPF separados, nem duas instâncias distintas rodando; é um único processo, com interfaces distribuídas entre áreas diferentes.

## Passo a passo — R-DATACENTER (roteador novo)

```
enable
configure terminal
hostname R-DATACENTER

interface Ethernet1
   description LINK-PARA-FILIAL
   no switchport
   ip address 10.0.1.2/30
   no shutdown

interface Ethernet0
   description LAN-DATACENTER
   no switchport
   ip address 192.168.30.1/24
   no shutdown

router ospf 1
   network 10.0.1.0/30 area 1
   network 192.168.30.0/24 area 1

end
write memory
```

## Configuração do host novo (VPCS)

```
PC-DATACENTER> ip 192.168.30.10/24 192.168.30.1
```

## Validação

Primeiro, confirmar que as adjacências OSPF se formaram corretamente:

```
show ip ospf neighbor
```

Em R-FILIAL, você deve ver **dois** vizinhos: R-MATRIZ (aprendido pela interface da Área 0) e R-DATACENTER (aprendido pela interface da Área 1), ambos em estado `FULL`. Em R-MATRIZ, você deve ver só R-FILIAL como vizinho — e o mesmo vale pro R-DATACENTER, só enxergando R-FILIAL. Isso confirma o que a topologia física já diz: não existe link direto entre Matriz e Data Center, então nunca vai existir adjacência OSPF direta entre eles.

Depois, verificar a tabela de rotas:

```
show ip route
```

Em R-MATRIZ, a rede 192.168.30.0/24 (Data Center) deve aparecer marcada como rota aprendida via OSPF — geralmente identificada com `O` no início da linha, e especificamente como rota inter-área (frequentemente indicada como `O IA` ou equivalente, dependendo da versão), justamente porque ela cruza a fronteira entre a Área 1 e a Área 0 pra chegar até ali. Isso responde diretamente a segunda pergunta de reflexão do desafio.

Teste de conectividade fim a fim:

```
PC-DATACENTER> ping 192.168.10.10
PC-DATACENTER> ping 192.168.20.10
```

Ambos devem responder.

## Resposta às perguntas de reflexão

**O que fica contido dentro dos limites de uma área?** A inundação (flooding) de LSAs de tipo 1 e 2 — que carregam o detalhe completo da topologia interna, roteador por roteador, link por link — fica restrita à própria área. Fora dela, o que atravessa a fronteira via ABR é só a informação resumida de quais redes existem e a que distância (LSAs de tipo 3, sumarização entre áreas), sem o detalhe topológico interno completo. Isso significa que uma instabilidade dentro da Área 1 — um link caindo e subindo repetidamente, por exemplo — gera recálculo de SPF (Shortest Path First) só entre os roteadores daquela área. R-MATRIZ, na Área 0, não recalcula sua topologia interna inteira por causa disso; ele só atualiza a rota resumida que chega via R-FILIAL. Numa rede que só cresce, isso é a diferença entre uma instabilidade local e uma instabilidade que derruba a performance da rede inteira.

**Que tipo de rota aparece em R-MATRIZ para o Data Center?** Como já confirmado na validação acima: uma rota OSPF inter-área, porque ela se origina numa área diferente da que R-MATRIZ pertence, e precisa atravessar o ABR (R-FILIAL) pra ser conhecida do outro lado.

## Diagnóstico do cenário bônus — incompatibilidade de área

Se você seguiu a proposta e mudou a interface Gi0/2 de R-FILIAL de Área 1 para Área 0 (sem mexer em R-DATACENTER, que continua declarando essa mesma rede como Área 1), o comando certo pra investigar é o mesmo de sempre:

```
show ip ospf neighbor
```

E aqui está o ponto central desse cenário: rodando esse comando em R-FILIAL ou em R-DATACENTER, **o vizinho simplesmente não aparece na lista** — nem em estado inicial, nem parcial, nada. Isso é diferente de outros problemas clássicos de OSPF (como incompatibilidade de temporizador hello/dead, ou de MTU), que geralmente permitem a formação de alguma adjacência antes de falhar em um estágio mais avançado do processo. Incompatibilidade de área é filtrada logo no início da troca de pacotes Hello — os dois roteadores nem chegam a se reconhecer como vizinhos potenciais, porque o pacote Hello carrega o ID da área, e cada lado descarta o pacote do outro por não bater com a área configurada localmente.

Pra confirmar a causa exata, sem depender só de deduzir pelo sintoma:

```
show ip ospf interface Ethernet2
```

Esse comando mostra explicitamente qual área está associada àquela interface especificamente — é ali que você veria Área 0 em R-FILIAL, contra Área 1 esperada, uma discrepância direta com o que R-DATACENTER está anunciando do outro lado do link.

Correção:

```
router ospf 1
   no network 10.0.1.0/30 area 0
   network 10.0.1.0/30 area 1
```

## O que você deveria levar desse lab

A diferença de comportamento entre incompatibilidade de área (adjacência nunca se forma) e outros erros de OSPF (adjacência se forma parcialmente e trava num estágio específico) é uma das distinções mais úteis que existem pra diagnóstico rápido de OSPF no mundo real. Guarda isso — no dia a dia, saber que "nem apareceu na lista de vizinhos" já elimina um conjunto inteiro de causas possíveis, e te leva direto pra checar área e tipo de rede antes de qualquer outra coisa.

Documenta esse lab e segue pro Lab 04.
