# Lab 01 — Conectando a Matriz à Filial Nova

🟢 Fácil · ⏱️ 45–60 min · 🧰 GNS3, Arista vEOS (ou Cisco IOS/Packet Tracer como alternativa), VPCS

---

## O chamado

Você acabou de ser contratado como estagiário de infraestrutura na **NexaCorp Soluções Ltda.**, uma empresa de médio porte que abriu uma filial nova há duas semanas. O time de TI é pequeno — praticamente só você e o coordenador, o Marcelo — e o e-mail abaixo chegou na sua caixa de entrada na sua segunda semana de trabalho:

> **De:** Marcelo Andrade (Coordenador de TI)
> **Para:** Você
> **Assunto:** Conectividade Matriz x Filial — precisa estar pronto até sexta
>
> Fala! Bom te ter no time. Preciso que você configure a conectividade básica entre a Matriz e a Filial que abrimos mês passado. Já passamos o link entre os dois prédios (é ponto a ponto, não se preocupa com o meio físico agora), só falta a configuração lógica.
>
> O pessoal da Filial precisa conseguir acessar os servidores que ficam na rede da Matriz — isso é prioridade, porque o financeiro deles depende do sistema que roda lá. Te mandei o endereçamento que já reservamos junto com a equipe de rede. Qualquer dúvida me chama, mas queria que você tentasse resolver sozinho primeiro — é assim que a gente aprende aqui.
>
> Valeu, Marcelo

Esse é o seu primeiro trabalho de verdade nesse material. Sem rede de segurança além dos comandos que você já viu na Fase 1 e 2 do roadmap.

## Objetivo do Lab

Estabelecer conectividade completa (fim a fim) entre a rede local da Matriz e a rede local da Filial, através de um link direto entre dois roteadores, usando endereçamento estático e rota estática.

Ao final, um host na rede da Filial precisa conseguir pingar um host na rede da Matriz — e vice-versa — sem intervenção manual em cada teste.

## Topologia

```
   [PC-MATRIZ]                                      [PC-FILIAL]
        |                                                 |
   192.168.10.0/24                                  192.168.20.0/24
        |                                                 |
   Gi0/0 [R-MATRIZ] Gi0/1 ---------- Gi0/1 [R-FILIAL] Gi0/0
                        \            /
                     10.0.0.1    10.0.0.2
                          (link /30)
```

Dois roteadores, um link ponto a ponto entre eles, uma rede local atrás de cada um, um host de teste em cada rede local.

## Tabela de Endereçamento (fornecida pela equipe de rede da NexaCorp)

| Dispositivo | Interface | Endereço IP | Máscara | Observação |
|---|---|---|---|---|
| R-MATRIZ | Gi0/0 | 192.168.10.1 | /24 | Gateway da LAN da Matriz |
| R-MATRIZ | Gi0/1 | 10.0.0.1 | /30 | Link para R-FILIAL |
| R-FILIAL | Gi0/1 | 10.0.0.2 | /30 | Link para R-MATRIZ |
| R-FILIAL | Gi0/0 | 192.168.20.1 | /24 | Gateway da LAN da Filial |
| PC-MATRIZ | — | 192.168.10.10 | /24 | Gateway: 192.168.10.1 |
| PC-FILIAL | — | 192.168.20.10 | /24 | Gateway: 192.168.20.1 |

Repare que o link entre os roteadores usa uma /30 — isso não foi acidente da equipe de rede, é o jeito padrão de endereçar um enlace ponto a ponto sem desperdiçar bloco de IP. Se você não lembra por que /30 dá exatamente dois hosts utilizáveis, vale revisar VLSM antes de continuar.

## O que você precisa entregar

1. Configurar as interfaces de ambos os roteadores conforme a tabela acima.
2. Configurar os hosts de teste (PC-MATRIZ e PC-FILIAL) com IP, máscara e gateway corretos.
3. Fazer com que R-MATRIZ saiba como chegar até a rede 192.168.20.0/24 (Filial) — e que R-FILIAL saiba como chegar até 192.168.10.0/24 (Matriz). Nenhum protocolo de roteamento dinâmico está liberado pra esse lab; a Fase 3 do roadmap cobre isso depois. Aqui é só rota estática mesmo.
4. Validar, com o comando adequado, que PC-MATRIZ consegue alcançar PC-FILIAL e vice-versa.

Note que eu não te disse o comando exato de rota estática, nem o comando exato de configuração de interface. Se você seguiu a Fase 1 e 2 do roadmap, você tem o que precisa pra chegar lá. Tenta.

## Perguntas pra você responder antes de configurar qualquer coisa

Antes de digitar um único comando, responde isso no papel ou mentalmente — isso evita que você configure no escuro:

Quando R-MATRIZ recebe um pacote destinado a 192.168.20.10 e não tem rota nenhuma configurada além da rede diretamente conectada, o que acontece com esse pacote? E qual comando de rota estática resolveria exatamente esse problema, considerando o próximo salto (next-hop) que você tem disponível na tabela de endereçamento?

## Cenário bônus — depois que tudo estiver funcionando

Duas horas depois de você validar a conectividade e avisar o Marcelo que terminou, ele te manda outra mensagem:

> "Ei, o pessoal da Filial tá reclamando que não conseguem mais acessar o sistema da Matriz. Você mexeu em algo?"

Você não mexeu em nada — mas alguém da equipe de rede fez uma alteração de manutenção no roteador da Matriz sem te avisar (isso acontece mais do que devia no mundo real). Reproduza esse cenário você mesmo: depois de validar que tudo funciona, altere deliberadamente a rota estática em R-MATRIZ, trocando o next-hop correto (10.0.0.2) por um endereço errado, mas plausível, dentro da mesma sub-rede do link — por exemplo 10.0.0.3.

Agora diagnostique como se fosse um chamado real: qual comando você usaria pra primeiro confirmar que a rota está mesmo errada, sem já saber a resposta de antemão? E qual seria seu segundo passo se o primeiro comando não deixasse o problema óbvio?

Não corrige ainda. Documenta o que você tentou e o raciocínio, mesmo que não chegue na resposta. Isso importa mais do que acertar de primeira.

---

⚠️ Quando terminar de tentar — com ou sem sucesso — vá para `Lab01_Topologia_Basica_Solucao.md`. Ele só faz sentido depois que você já suou a camisa aqui.
