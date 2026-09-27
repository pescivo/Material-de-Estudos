# Material de Apoio — Trilha de Cibersegurança

Repositório de estudo gratuito com uma trilha completa: nove módulos de fundamentos e sete frentes de especialização em cibersegurança, com laboratório prático em praticamente todo módulo. Feito pra acompanhar o conteúdo que eu (Pedro) posto no Instagram e no TikTok, mas construído pra funcionar sozinho, mesmo se você chegou aqui sem nunca ter visto um post meu.

Se você não sabe por onde começar, a resposta é simples: [**clica aqui e lê o arquivo de boas-vindas primeiro**](00_Comece_Aqui/Boas-vindas_e_Como_Usar.md). Ele explica como esse material foi pensado e por que a ordem das pastas importa — pular direto pros labs sem ler isso é o jeito mais garantido de se perder no meio do caminho.

---

## Como isso está organizado

```
00_Comece_Aqui/
    Boas-vindas_e_Como_Usar.md      → comece por aqui, sem exceção
    Roadmap_Geral.md                 → visão macro de toda a trilha, Fundamentos → Especializações
    Roadmap_Networking.md            → detalhe do módulo de Networking
    Roadmap_Cybersecurity.md         → detalhe do caminho de especialização em segurança

01_Fundamentos/
    01_Fundamentos_de_Computacao/    → hardware, binário/hex, sistema operacional, boot
    02_Networking/                   → 12 laboratórios completos, do básico ao troubleshooting avançado
        Teoria/
        Labs_Praticos/
        Cheatsheets/
    03_Fundamentos_de_Linux/         → shell, sistema de arquivos, permissões, processos
    04_Fundamentos_de_Windows/       → Active Directory, GPO, PowerShell, Kerberos
    05_Fundamentos_de_Ciberseguranca/→ tríade CIA, vulnerabilidade x ameaça x risco
    06_Fundamentos_Web/              → HTTP/HTTPS, cookies, sessão, como um site funciona
    07_Scripting_Basico/             → Python com foco em automação e segurança
    08_Git/                          → versionamento, o básico de branch e commit
    09_Troubleshooting/              → metodologia de diagnóstico, além de rede

02_Especializacoes/
    01_Red_Team_Pentest/             → PTES, reconhecimento, ataques a Active Directory
    02_Blue_Team_SOC/                → hardening, análise de log, SIEM, resposta a incidente
    03_Cloud_Security/               → segurança em AWS/Azure/GCP
    04_GRC/                          → NIST CSF, ISO 27001
    05_DFIR_Analise_Malware/         → forense digital, resposta a incidente, análise de malware
    06_Criptografia/                 → cifra simétrica/assimétrica, hash, assinatura digital
    07_Seguranca_de_Aplicacoes/      → OWASP Top 10, segurança de aplicação web
    Labs_Red_vs_Blue/                → laboratórios integrados de ataque e defesa

03_Projetos_Portfolio/
    Como documentar seu trabalho e transformar lab em material de entrevista

04_Certificacoes_e_Recursos/
    Referências de certificações gratuitas — sempre confirme na fonte oficial
```

Cada laboratório vem em dois arquivos separados: um de **desafio** (o cenário, a topologia, o que precisa ser entregue — sem comando de resposta) e outro de **solução** (o passo a passo comentado, com o raciocínio de diagnóstico explicado). Isso é proposital. Se você abrir a solução antes de tentar, tá roubando de você mesmo — o aprendizado real acontece na hora que você trava. Todo módulo tem alguma forma de prática associada, mesmo quando o formato não é um laboratório completo de rede — pode ser exercício guiado, script pra completar, ou cenário de análise.

---

## Como usar este repositório

Você não precisa saber Git pra aproveitar isso. As três formas mais comuns:

- **Ler direto no navegador** — clica em qualquer arquivo `.md` acima e o GitHub renderiza formatado, sem precisar baixar nada.
- **Baixar tudo de uma vez** — no botão verde **Code → Download ZIP**, no topo desta página, sem precisar instalar Git.
- **Clonar o repositório**, se você já usa Git ou quer acompanhar atualizações com `git pull`:

```
git clone https://github.com/SEU-USUARIO/SEU-REPOSITORIO.git
```

Pra quem prefere estudar offline ou tem menos familiaridade com GitHub, deixo também uma cópia sincronizada no Google Drive — o link fica fixado nos meus destaques do Instagram.

---

## Pra quem é isso

Pessoas entre 16 e 35 anos que querem entrar na área de cibersegurança do zero, ou que já têm alguma base técnica e querem consolidar fundamentos antes de se especializar. Mais detalhe sobre isso — e sobre a responsabilidade ética que vem junto com a parte ofensiva do material — está no [arquivo de boas-vindas](00_Comece_Aqui/Boas-vindas_e_Como_Usar.md).

## Uso e responsabilidade

Este material é gratuito e aberto para estudo pessoal. Todo conteúdo relacionado a técnicas ofensivas (Red Team, e partes de DFIR/Análise de Malware) é destinado exclusivamente a ambientes de laboratório controlados e isolados. Aplicar essas técnicas contra sistemas ou redes sem autorização formal é crime — os detalhes legais completos estão no arquivo de boas-vindas, e vale a pena ler antes de tocar em qualquer conteúdo da pasta `02_Especializacoes/01_Red_Team_Pentest`.

## Contato e atualizações

Novos módulos, correções e conteúdo adicional são publicados aqui com frequência — esse material cresceu bastante desde a primeira versão e continua em construção ativa. Atualização relevante eu aviso nas redes — me segue lá pra não perder:

- Instagram: [@pescivo](https://instagram.com/pescivo)

Se encontrar erro técnico em algum arquivo, abre uma *issue* aqui no repositório — é a forma mais rápida de eu ver e corrigir.

---

Bons estudos. Começa pelo [Boas-vindas_e_Como_Usar.md](00_Comece_Aqui/Boas-vindas_e_Como_Usar.md).
