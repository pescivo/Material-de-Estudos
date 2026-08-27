# Roadmap de Cybersecurity — de fundamentos até Red vs Blue

Se você chegou aqui direto, sem passar pela trilha de Networking, para e volta um pouco. Sério. Eu vou repetir isso quantas vezes precisar porque é o erro mais comum que eu vejo gente cometendo: tentar aprender a invadir ou defender uma rede sem entender como ela funciona por baixo. Você não vai conseguir interpretar um log de firewall, identificar um scan de porta numa captura de tráfego, ou entender por que um ataque de lateral movement funciona, se não souber como pacote se move numa rede.

Presumindo que essa base já tá com você, vamos pro roadmap. Ele tem quatro fases e termina nos laboratórios de Red vs Blue, que são o ponto mais avançado desse material inteiro.

## Fase 1 — Fundamentos que sustentam tudo o resto

Antes de ferramenta, antes de ataque, antes de defesa: você precisa internalizar a tríade CIA — Confidencialidade, Integridade e Disponibilidade. Parece básico até demais pra merecer atenção, mas na prática é o critério que você vai usar pra avaliar qualquer decisão de segurança daqui pra frente. "Essa configuração compromete confidencialidade?" "Esse controle afeta disponibilidade do serviço?" Esse tipo de pergunta é o que separa quem aplica checklist de quem realmente entende risco.

Junto disso, entender os tipos de ameaça mais comuns — malware, phishing, engenharia social, exploração de vulnerabilidade — e a diferença entre vulnerabilidade, ameaça e risco. Essas três palavras são usadas de forma intercambiável por gente que não devia, e isso gera confusão de análise séria mais na frente.

## Fase 2 — Blue Team: aprenda a defender antes de aprender a atacar

Esse roadmap coloca defesa antes de ataque, e essa ordem é intencional. Você vai entender ataque muito melhor depois de ter passado horas configurando defesa e vendo, na prática, onde ela falha.

Nessa fase entra hardening de sistemas — Windows Server é o ponto de partida óbvio, principalmente se você, como eu, já tem alguma vivência com Active Directory e GPO. Reduzir superfície de ataque, desabilitar serviço desnecessário, aplicar política de senha decente, configurar auditoria. Depois entra análise de log: aprender a olhar Event Viewer, entender o que é um log relevante versus ruído, e dar o primeiro passo em direção a conceito de SIEM (Security Information and Event Management) — mesmo que você ainda não vá operar uma ferramenta comercial completa nessa fase, entender o conceito de centralização e correlação de log já muda como você enxerga monitoramento.

Um ponto que eu faço questão de reforçar: Blue Team não é "instalar antivírus e configurar firewall". É constante, é analítico, é sobre entender padrão de comportamento numa rede pra identificar quando algo foge do normal. É trabalho de observação tanto quanto de configuração.

## Fase 3 — Governança: a parte que ninguém acha empolgante mas que abre porta

NIST CSF e ISO 27001 entram aqui. Eu sei que "framework de governança" soa como a coisa mais distante de "hacker de filme" que existe, mas essa é literalmente a diferença entre alguém que sabe rodar ferramenta e alguém que consegue trabalhar dentro de uma empresa de verdade, com processo, auditoria e compliance envolvidos.

NIST CSF te dá uma estrutura de pensamento — identificar, proteger, detectar, responder, recuperar — que funciona como um mapa mental pra qualquer situação de segurança, do incidente mais simples ao mais grave. ISO 27001 entra na parte de sistema de gestão de segurança da informação, que é o que empresas maiores exigem de fornecedor e de time interno.

Não precisa se aprofundar a ponto de virar auditor. Mas entender a lógica desses frameworks, mesmo que de forma introdutória, é o tipo de conhecimento que aparece em entrevista e que sinaliza maturidade profissional — é isso que diferencia currículo de quem só fez CTF de currículo de quem entende o contexto de negócio por trás da segurança.

## Fase 4 — Red Team: agora sim, mas com responsabilidade

Chegou a parte que provavelmente te trouxe até aqui. Antes de qualquer coisa técnica: relê a seção de ética do arquivo de boas-vindas se você ainda não fez isso com atenção. Tudo que vem a seguir só existe em ambiente controlado, autorizado, isolado. Sem exceção.

Metodologia primeiro — PTES (Penetration Testing Execution Standard) te dá a estrutura de como um teste de intrusão profissional realmente acontece: reconhecimento, modelagem de ameaça, análise de vulnerabilidade, exploração, pós-exploração, relatório. Reparo que citei "relatório" como etapa formal, porque é isso que separa pentest profissional de "brincar de hacker" — o valor real que você entrega pro cliente é o relatório, não o exploit em si.

Dentro dessa fase entram reconhecimento e enumeração — a parte que consome mais tempo num pentest real e que a maioria dos iniciantes subestima —, e uma introdução a ataques contra Active Directory, que faz todo sentido dado que você (assim como eu) já tem vivência prévia com AD do lado defensivo. Entender AD do ponto de vista ofensivo, depois de já ter mexido nele como administrador, é uma vantagem real que pouca gente entrando em red team tem.

## Fase 5 — Red vs Blue: onde as duas trilhas se encontram

Os labs finais da pasta `Labs_Red_vs_Blue` colocam as duas perspectivas em cenário conjunto: um lado ataca, o outro detecta e responde, dentro do mesmo ambiente controlado. É o exercício mais próximo de uma situação real que esse material oferece, porque força você a pensar dos dois lados ao mesmo tempo — que é exatamente o que faz um profissional de segurança ser bom nos dois sentidos, mesmo que a especialização final seja só um deles.

## Certificações, com o mesmo aviso de sempre

Na pasta `04_Certificacoes` eu deixo referência a opções gratuitas — Microsoft SC-900 e SC-300 fazem sentido pra quem quer uma introdução mais formal a conceitos de segurança em ambiente Microsoft, e existem badges gratuitos da AWS pra quem quer começar a olhar segurança em nuvem. De novo: código de exame e formato de prova mudam com frequência, então trata essa lista como ponto de partida e confirma sempre na fonte oficial antes de investir tempo se preparando.

## Onde isso te leva

Terminando essa trilha — Blue Team sólido, governança entendida, Red Team com fundamento ético e técnico junto — você tem o que precisa pra começar a construir portfólio real de segurança. É esse portfólio, documentado na pasta `03_Projetos_Portfolio`, que vai fazer diferença quando chegar a hora de conversar com recrutador ou de migrar de carreira, que é exatamente o que eu tô fazendo agora, junto com quem tá lendo isso.

Vai com calma na fase 1. Governança e fundamento parecem chatos até você ver o quanto eles evitam dor de cabeça mais na frente.
