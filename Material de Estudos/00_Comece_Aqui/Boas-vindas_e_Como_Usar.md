# Bem-vindo(a). Isso aqui não é mais um "material gratuito de IA sobre redes"

Eu sou o Pedro, e antes de você navegar pelas pastas eu preciso te contar por que esse material existe, porque isso muda como você vai usá-lo.

Trabalho com redes há um bom tempo. Nos últimos meses tenho me dedicado pesado pra migrar pra pentest — o plano é estar atuando em red team até o início de 2027 — e isso significou voltar pra estaca zero em várias coisas que eu achava que já sabia. Montar laboratório do zero, apanhar de VirtualBox não pegando IP, quebrar a cabeça com diferença de sintaxe entre Cisco e Arista porque o CML da Cisco é pago e eu não ia gastar dinheiro à toa. Esse material nasceu justamente dessas horas de tela travada às 23h tentando entender por que um roteador não trocava rota com o outro.

Então não é teoria empacotada de curso pronto. É o caminho que eu tô fazendo, documentado pra quem tá alguns passos atrás de mim.

## Pra quem é isso

Se você tem entre 16 e 35 anos e se encaixa em pelo menos uma dessas situações, o material foi pensado pra você:

Você nunca configurou um roteador na vida e quer uma base de verdade, não só "decorar comando pra passar em prova". Você já manja de rede mas quer entender segurança ofensiva e defensiva do jeito certo, sem pular etapa. Ou você tá exatamente onde eu tô: migrando de infra pra segurança e precisa consolidar os dois lados antes de se especializar.

Uma coisa que eu preciso deixar clara logo de cara: eu não vou te entregar respostas prontas. Cada laboratório aqui foi desenhado pra você travar em algum ponto. Isso é proposital. Ninguém aprende troubleshooting vendo alguém resolver — aprende resolvendo. Eu só vou te dar o cenário e as ferramentas pra chegar lá.

## Como as pastas estão organizadas e por que separei assim

```
00_Comece_Aqui          -> onde você está agora
01_Networking           -> teoria, os 10 labs práticos em GNS3, cheatsheets
02_Cybersecurity         -> Blue Team, Red Team, labs de ataque e defesa
03_Projetos_Portfolio    -> como transformar o que você fez em algo que abre porta em entrevista
04_Certificacoes         -> certificações gratuitas que valem a pena olhar
```

Separei Networking de Cybersecurity de propósito, e não foi só organização por estética. Já vi gente pulando direto pra "hackear coisa" sem entender o que é uma sub-rede, e o resultado é sempre o mesmo: a pessoa decora comando de ferramenta mas não entende o que tá acontecendo por trás. Se você não sabe como um pacote se move de um host até outro, você não vai entender por que um ataque de ARP spoofing funciona, nem vai saber identificar ele numa captura de tráfego. Rede vem antes. Sem exceção, mesmo que pareça mais chato no começo.

Se você já tem essa base sólida, pode ir direto pra pasta 02. Mas dá uma passada nos cheatsheets da pasta 01 antes — leva dez minutos e evita buraco.

## O padrão que todo laboratório segue

Pra não ficar cada arquivo com uma cara diferente, todo lab segue a mesma estrutura:

**Objetivo do Lab** — o que você vai construir e, mais importante, por que isso importa na prática, não só "porque tá no roteiro".

**Topologia e Requisitos** — quais dispositivos, endereçamento IP, softwares. Aqui não tem meio-termo: os IPs são exatos, os comandos são exatos.

**Guia Passo a Passo** — a configuração completa, comando por comando.

**Cenário de Teste e Validação** — como provar, com ping, traceroute, nmap ou o que for necessário, que sua rede realmente faz o que deveria fazer. Configuração que "parece" funcionar mas não foi validada é a origem de 90% dos incidentes reais.

**Desafio de Troubleshooting** — uma falha que eu insiro de propósito. Você vai ter que descobrir sozinho o que quebrou.

Sobre esse último ponto: resista à tentação de rolar até o final pra ver a solução. Passa pelo menos 20, 30 minutos travado antes disso. É desconfortável, eu sei — eu ainda travo em coisa que devia ser básica pra mim. Mas é literalmente esse desconforto que separa quem entende rede de quem só decorou uma topologia.

Cada arquivo usa alguns símbolos rápidos pra você identificar o que precisa antes de começar:

| Símbolo | O que indica |
|---|---|
| 🟢 🟡 🔴 | Nível de dificuldade (fácil, intermediário, avançado) |
| ⏱️ | Tempo estimado pra concluir o lab |
| 🧰 | Ferramentas e softwares que você precisa ter instalado |
| ⚠️ | Aviso técnico, ético ou legal — leia sempre |

## Sobre a parte de segurança ofensiva — isso aqui não é opcional de ler

Vou ser direto porque esse ponto não dá pra suavizar. Uma parte desse material ensina técnicas de reconhecimento, exploração e teste de intrusão. Isso é conhecimento real, o mesmo usado por profissionais de pentest no mercado. E junto com esse conhecimento vem uma responsabilidade que eu levo a sério e espero que você também leve.

Tudo que envolve ataque aqui — scan de porta, exploração de vulnerabilidade, ataque a Active Directory, qualquer coisa nesse estilo — é pra rodar exclusivamente em laboratório isolado: GNS3, EVE-NG, máquinas virtuais que são suas, ou plataformas de treinamento reconhecidas como TryHackHome ou HackTheBox, onde o ambiente já é preparado pra isso. Nunca, em hipótese nenhuma, aplique essas técnicas em rede, sistema ou dispositivo que não seja seu ou pro qual você não tenha autorização formal e por escrito. No Brasil isso é crime — a Lei 12.737/2012 trata especificamente de invasão de dispositivo informático, e existe ainda o próprio Código Penal cobrindo outras frentes. "Eu só queria testar" não é defesa legal, e não vai ser defesa pra sua carreira também: a reputação nessa área é praticamente tudo, e um deslize desses fecha portas de forma permanente.

Eu não coloco esse aviso aqui por formalidade. Coloco porque quero genuinamente que você chegue no red team em 2027, 2028, sei lá quando for o seu caso, sem ter estragado o próprio caminho por impaciência.

## O que você precisa ter instalado antes de começar

Pra acompanhar Networking e a maior parte de Cybersecurity você vai precisar de uma máquina com pelo menos 8 GB de RAM — 16 GB torna sua vida bem mais tranquila quando os labs começam a rodar várias VMs ao mesmo tempo. GNS3 é obrigatório, Packet Tracer ajuda em alguns momentos específicos, e você vai precisar instalar e configurar algum hypervisor (eu uso VirtualBox, mas isso pode mudar dependendo do lab). Se você nunca mexeu com linha de comando, não se preocupa — isso vai ser construído junto com o conteúdo, ninguém nasce sabendo.

Cada lab individual detalha exatamente o que ele precisa na seção de requisitos. Não tem surpresa.

## Como transformar isso em progresso real, não só arquivo lido

Ler um lab e nunca montar a topologia é a forma mais fácil de esquecer tudo em duas semanas. Sugestão prática: crie uma planilha simples, uma linha por lab, e marque quatro coisas: se você montou a topologia sozinho, se validou que funciona, se resolveu o troubleshooting sem espiar a solução, e se documentou o resultado com print e explicação escrita.

Esse último ponto parece bobo, mas é o que vira o seu portfólio. Ninguém em entrevista de emprego vai perguntar "você fez o lab 7?". Vão perguntar "me mostra algo que você construiu". Documentação é o que transforma horas de estudo em prova concreta de competência.

## Isso vai crescer com o tempo

Esse material não é estático. Vou adicionar lab, corrigir o que tiver errado, aprofundar o que for raso demais — inclusive porque eu mesmo tô aprendendo boa parte disso junto com quem acompanha. Atualização relevante eu aviso nas redes.

## Próximo passo

Segue pra `01_Networking/Roadmap_Networking.md` se você tá começando do zero, ou pra `02_Cybersecurity/Roadmap_Cybersecurity.md` se a base de rede já tá consolidada. E se ficar travado em algo, é assim mesmo que deve ser no começo. Segue o processo.
