# Material de Apoio — Redes e Cibersegurança

Repositório de estudo gratuito com laboratórios práticos, roadmaps e projetos de portfólio em Networking e Cybersecurity. Feito pra acompanhar o conteúdo que eu (Pedro) posto no Instagram e no TikTok, mas construído pra funcionar sozinho, mesmo se você chegou aqui sem nunca ter visto um post meu.

Se você não sabe por onde começar, a resposta é simples: [**clica aqui e lê o arquivo de boas-vindas primeiro**](00_Comece_Aqui/Boas-vindas_e_Como_Usar.md). Ele explica como esse material foi pensado e por que a ordem das pastas importa — pular direto pros labs sem ler isso é o jeito mais garantido de se perder no meio do caminho.

---

## Como isso está organizado

```
00_Comece_Aqui/
    Boas-vindas_e_Como_Usar.md      → comece por aqui, sem exceção
    Roadmap_Networking.md            → trilha completa de redes, do zero ao avançado
    Roadmap_Cybersecurity.md         → trilha completa de segurança, de fundamentos a Red Team

01_Networking/
    Teoria/                          → OSI, TCP/IP, VLSM, OSPF, BGP e mais
    Labs_Praticos/                   → os 10 laboratórios em GNS3, do básico ao avançado
    Cheatsheets/                     → referência rápida de comandos, incluindo Arista x Cisco

02_Cybersecurity/
    Blue_Team/                       → hardening, análise de log, governança
    Red_Team/                        → metodologia de pentest, reconhecimento, AD attacks
    Labs_Red_vs_Blue/                → laboratórios integrados de ataque e defesa

03_Projetos_Portfolio/
    Como documentar seu trabalho e transformar lab em material de entrevista

04_Certificacoes_e_Recursos/
    Referências de certificações gratuitas — sempre confirme na fonte oficial
```

Cada laboratório vem em dois arquivos separados: um de **desafio** (o cenário, a topologia, o que precisa ser entregue — sem comando de resposta) e outro de **solução** (o passo a passo comentado, com o raciocínio de diagnóstico explicado). Isso é proposital. Se você abrir a solução antes de tentar, tá roubando de você mesmo — o aprendizado real acontece na hora que você trava.

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

Pessoas entre 16 e 35 anos começando em TI, migrando de infraestrutura para segurança, ou já com alguma base querendo consolidar fundamentos antes de se especializar. Mais detalhe sobre isso — e sobre a responsabilidade ética que vem junto com a parte ofensiva do material — está no [arquivo de boas-vindas](00_Comece_Aqui/Boas-vindas_e_Como_Usar.md).

## Uso e responsabilidade

Este material é gratuito e aberto para estudo pessoal. Todo conteúdo relacionado a técnicas ofensivas (Red Team) é destinado exclusivamente a ambientes de laboratório controlados e isolados. Aplicar essas técnicas contra sistemas ou redes sem autorização formal é crime — os detalhes legais completos estão no arquivo de boas-vindas, e vale a pena ler antes de tocar em qualquer lab da pasta `02_Cybersecurity/Red_Team`.

## Contato e atualizações

Novos labs, correções e conteúdo adicional são publicados aqui com frequência. Atualização relevante eu aviso nas redes — me segue lá pra não perder:

- Instagram: [@pescivo](https://instagram.com/pescivo)
- TikTok: [@pescivo](https://tiktok.com/@pescivo)

Se encontrar erro técnico em algum arquivo, abre uma *issue* aqui no repositório — é a forma mais rápida de eu ver e corrigir.

---

Bons estudos. Começa pelo [Boas-vindas_e_Como_Usar.md](00_Comece_Aqui/Boas-vindas_e_Como_Usar.md).
