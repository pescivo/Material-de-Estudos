# Roteamento Estático vs Dinâmico

Esse é um dos poucos temas de rede onde eu acho que a ordem de aprendizado importa mais do que o conteúdo em si. Se você aprende roteamento dinâmico antes de sentir na pele a limitação do roteamento estático, o roteamento dinâmico vira só "mais um protocolo pra decorar comando", sem você entender de verdade por que ele existe. Foi por isso que esse material te fez configurar rota estática no Lab 01 antes de qualquer OSPF.

## O que é uma rota, tecnicamente

Toda vez que um roteador recebe um pacote, ele consulta sua tabela de rotas pra decidir por qual interface (e, em geral, por qual próximo salto — next-hop) encaminhar aquele pacote, com base no endereço de destino. Uma "rota" nada mais é do que uma entrada dessa tabela: "pra chegar na rede X, mande pro endereço Y".

Essa tabela pode ser preenchida de duas formas fundamentalmente diferentes: alguém escreve a rota manualmente (estática), ou um protocolo de roteamento descobre e mantém essas rotas automaticamente, trocando informação com outros roteadores (dinâmica).

## Rota estática: simples, previsível, e não escala

Rota estática é exatamente o que você configurou no Lab 01: uma linha de comando dizendo explicitamente pra qual rede, através de qual next-hop, o tráfego deve ir. Ela não muda sozinha — se a topologia mudar (um link cair, um caminho alternativo aparecer), a rota estática continua exatamente como foi escrita, mesmo que ela já não faça mais sentido.

Vantagens reais: previsibilidade total (você sabe exatamente o caminho que o tráfego vai seguir, sempre), sem overhead de processamento ou de troca de mensagens entre roteadores, e sem exposição a alguns tipos de ataque que exploram justamente a natureza dinâmica de protocolos de roteamento. Por isso, rota estática nunca "acaba" completamente numa rede — ela continua sendo a escolha certa pra cenários pequenos, previsíveis, ou pra rotas específicas dentro de uma rede maior (como uma rota padrão apontando pro provedor de internet, que você configurou no Lab 05, mesmo dentro de uma topologia já rodando OSPF).

Limitação real, que o próprio Marcelo (fictício) descreveu no e-mail do Lab 03: não escala. Cada novo site, cada nova rede, significa reconfiguração manual em potencialmente todo roteador da topologia. E pior que o trabalho manual em si: é fácil esquecer de atualizar uma rota em algum ponto da rede quando a topologia muda, e esse tipo de inconsistência é notoriamente difícil de detectar até gerar um problema real de conectividade.

## Roteamento dinâmico: escala, mas tem custo

Um protocolo de roteamento dinâmico (OSPF, EIGRP, BGP, entre outros) faz os roteadores trocarem informação de topologia entre si automaticamente, calculando e atualizando rotas sem intervenção manual. Se um link cai, os roteadores vizinhos percebem essa mudança (através de mecanismos específicos de cada protocolo) e recalculam automaticamente um caminho alternativo, se existir um.

O ganho óbvio é escala e resiliência — foi exatamente isso que resolveu o problema descrito no Lab 03. O custo, que muita gente subestima: complexidade adicional de configuração e operação, algum overhead de processamento e de tráfego de controle circulando na rede (as próprias mensagens de troca de informação do protocolo), e uma superfície de risco maior — um roteador comprometido ou mal configurado dentro de um domínio de roteamento dinâmico pode, em cenários específicos, injetar rotas falsas e desviar tráfego de forma maliciosa, algo que roteamento estático simplesmente não permite, porque não existe negociação nenhuma acontecendo.

## Protocolos de estado de enlace vs vetor de distância

Dentro de roteamento dinâmico, existe uma divisão conceitual importante: protocolos de **vetor de distância** (como o histórico RIP) tomam decisão de rota com base em informação recebida de vizinhos diretos, sem visão completa da topologia — cada roteador sabe "por onde" chegar num destino, mas não necessariamente "como é" o caminho inteiro. Protocolos de **estado de enlace** (como OSPF, que você já configurou) constroem um mapa completo da topologia dentro de uma área, com cada roteador sabendo exatamente como todos os links se conectam, e calculando o melhor caminho a partir desse mapa completo, usando o algoritmo de Dijkstra (Shortest Path First).

Estado de enlace converge mais rápido e toma decisões mais precisas, ao custo de exigir mais processamento e mais memória — cada roteador precisa manter esse mapa completo da topologia, não só uma tabela simples de distâncias.

## Quando usar cada um, na prática

Rota estática faz sentido em: redes pequenas com topologia simples e estável, rota padrão apontando pra um único ponto de saída (como internet), ou situações onde previsibilidade absoluta importa mais que flexibilidade automática.

Roteamento dinâmico faz sentido em: redes com múltiplos caminhos redundantes, topologia que muda com alguma frequência, ou ambientes que precisam de recuperação automática de falha sem intervenção manual imediata.

Na prática — e você já viu isso funcionando junto, desde o Lab 05 — as duas abordagens coexistem na maioria das redes reais. Rota estática pra um propósito específico e previsível (a saída pra internet), roteamento dinâmico cuidando de toda a complexidade interna que se beneficia de automação.

## Perguntas para reflexão

Se você tivesse uma rede de exatamente dois roteadores, ligados por um único link, sem nenhuma perspectiva de crescimento nos próximos anos, você recomendaria rota estática ou roteamento dinâmico? O que mudaria na sua resposta se essa mesma rede tivesse um segundo link redundante entre os mesmos dois roteadores?

Segunda pergunta, olhando pra trás no roadmap: por que faz sentido pedagógico (e também prático, em ambiente real) configurar rota estática pra saída de internet mesmo numa rede que já roda OSPF internamente, em vez de simplesmente aprender a rota padrão do provedor via protocolo dinâmico também?
