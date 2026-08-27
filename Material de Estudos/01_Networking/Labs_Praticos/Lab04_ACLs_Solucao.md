# Lab 04 — Solução Recomendada

⚠️ Gabarito de instrutor. A parte de "onde aplicar a ACL" vale mais que a sintaxe em si — se você já decidiu isso sozinho antes de chegar aqui, mesmo que tenha escolhido diferente do que vem abaixo, já valeu o esforço. Compara o raciocínio, não só o resultado.

---

## Onde aplicar a ACL — e por quê

A resposta recomendada é aplicar a ACL na interface de R-DATACENTER que recebe tráfego vindo de R-FILIAL (o ponto de entrada do Data Center), em direção de entrada (`in`).

O raciocínio: hoje existem três redes de origem (Financeiro, TI, Vendas), mas isso é um detalhe momentâneo da empresa. Se a NexaCorp abrir um quarto departamento amanhã, esse novo departamento vai, mais cedo ou mais tarde, também precisar trafegar por esse mesmo caminho físico pra alcançar o Data Center — porque é o único caminho que existe. Se a ACL estivesse aplicada em algum ponto mais próximo da origem (por exemplo, na saída do SW-CORE), você precisaria replicar ou revisar essa configuração toda vez que uma nova origem aparecesse. Aplicando no ponto de entrada do recurso protegido — o Data Center — você define a política uma vez, baseada em "quem pode entrar aqui", e ela continua correta independente de quantas redes novas surgirem do outro lado.

Esse é um princípio real de design de segurança de rede: proteger o recurso sensível na borda dele, não tentar controlar todo ponto de origem possível espalhado pela rede. É o mesmo raciocínio por trás de segmentar um Data Center ou um ambiente de pagamento numa área isolada com controle de acesso centralizado na entrada, em vez de depender de disciplina de configuração em dezenas de pontos diferentes.

## Passo a passo — R-DATACENTER (Arista EOS)

Criar a ACL nomeada:

```
enable
configure terminal

ip access-list DATACENTER-PROTECAO
   10 permit ip 192.168.11.0/24 192.168.30.0/24
   20 permit ip 192.168.12.0/24 192.168.30.0/24
   30 deny ip 192.168.13.0/24 192.168.30.0/24
   40 permit ip any any
```

Cada linha numerada (10, 20, 30, 40) importa — é essa numeração que define a ordem de avaliação, e é ela que você vai manipular se precisar inserir uma regra no meio da lista mais tarde, sem precisar reescrever a ACL inteira.

A linha 40 (`permit ip any any`) é o que garante que nenhum outro tráfego — de Financeiro, TI, Vendas indo pra qualquer destino que não seja o Data Center, ou de qualquer outra origem futura — seja pego pelo "deny all" implícito que toda ACL estendida carrega no final. Sem essa linha, você resolveria o problema pedido, mas quebraria silenciosamente outras conexões legítimas passando por essa mesma interface — exatamente o tipo de efeito colateral que o e-mail do Marcelo pediu pra evitar.

Aplicar na interface correta, em direção de entrada:

```
interface Ethernet1
   description LINK-PARA-FILIAL
   ip access-group DATACENTER-PROTECAO in
```

```
end
write memory
```

## Validação

```
PC-FINANCEIRO> ping 192.168.30.10
PC-TI> ping 192.168.30.10
```

Ambos devem responder normalmente.

```
PC-VENDAS> ping 192.168.30.10
```

Deve falhar — sem resposta, ou explicitamente indicando destino inalcançável, dependendo de como o simulador reporta.

```
PC-VENDAS> ping 192.168.10.10
PC-VENDAS> ping 192.168.20.10
```

Ambos devem continuar respondendo normalmente — isso confirma que a restrição afetou exclusivamente o destino Data Center, não o restante do acesso de Vendas.

Em R-DATACENTER, o comando que mostra o conteúdo da ACL junto com estatística de uso:

```
show ip access-lists DATACENTER-PROTECAO
```

Você deve ver, junto de cada linha, um contador de quantos pacotes já bateram com aquela regra especificamente — as linhas 10 e 20 acumulando contagem conforme Financeiro e TI acessam normalmente, e a linha 30 acumulando contagem toda vez que Vendas tenta e é barrado.

## Diagnóstico do cenário bônus — ordem das regras trocada

Se você reproduziu o cenário movendo `permit ip any any` pra antes da regra de negação de Vendas, a ACL passa a ficar assim, na prática:

```
ip access-list DATACENTER-PROTECAO
   10 permit ip 192.168.11.0/24 192.168.30.0/24
   20 permit ip 192.168.12.0/24 192.168.30.0/24
   25 permit ip any any
   30 deny ip 192.168.13.0/24 192.168.30.0/24
```

ACL em roteador é avaliada de cima pra baixo, e a avaliação **para na primeira linha que casar com o pacote** — ela não continua lendo o resto da lista procurando uma regra "melhor" ou "mais específica". Um pacote de Vendas indo pro Data Center chega, é comparado com a linha 10 (não bate, origem diferente), depois linha 20 (não bate, origem diferente), depois linha 25 — e `any any` bate com absolutamente qualquer coisa. Nesse ponto o roteador já decidiu permitir o pacote e nem chega a avaliar a linha 30, mesmo que ela exista logo depois e fosse tecnicamente mais específica pro caso.

O comando `show ip access-lists DATACENTER-PROTECAO` expõe isso de forma direta: você veria o contador da linha 25 subindo a cada tentativa de Vendas, e o contador da linha 30 permanecendo zerado — apesar da regra de negação existir, ela nunca é alcançada. Esse é o sintoma mais confiável de erro de ordenação em ACL: regra que existe, está sintaticamente correta, mas o contador de uso mostra zero, porque uma regra anterior mais genérica está capturando o tráfego antes.

Correção: reordenar a lista, movendo a regra de negação pra antes da permissão genérica. Em Arista EOS, você pode reinserir a linha com um número que a posicione corretamente, sem precisar apagar a ACL inteira:

```
configure terminal
ip access-list DATACENTER-PROTECAO
   no 25
   30 deny ip 192.168.13.0/24 192.168.30.0/24
```

(Ajuste os números de linha conforme o que já existir na sua ACL — o importante é garantir que qualquer `deny` ou `permit` específico sempre venha antes de uma regra genérica que possa capturá-lo primeiro.)

## O que você deveria levar desse lab

Duas lições, e as duas importam igual. A técnica: ACL é avaliada em ordem, sempre, e regra genérica antes de regra específica é o erro de ordenação mais comum e mais silencioso que existe — ele não gera erro de sintaxe nem aviso, só comportamento errado que passa despercebido até alguém perceber por acaso, como no chamado do Marcelo. E a de design: onde você aplica um controle de segurança importa tanto quanto o conteúdo do controle em si — aplicar próximo ao recurso que você está protegendo, em vez de tentar cobrir toda origem possível, é uma decisão que se paga sozinha conforme a rede cresce.

Documenta esse lab — incluindo a justificativa de posicionamento, porque a auditoria fictícia pediu isso, mas principalmente porque isso é exatamente o tipo de raciocínio que aparece em entrevista técnica de verdade.
