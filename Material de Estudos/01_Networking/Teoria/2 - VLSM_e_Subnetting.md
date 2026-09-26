# VLSM e Subnetting

Confissão: essa foi a parte da minha própria formação que eu mais evitei estudar de verdade no começo. Parecia matemática decorada, sem aplicação real na frente da tela. Levei um tempo pra perceber que é exatamente o oposto — é uma das habilidades mais práticas que existem, porque endereçamento mal planejado te persegue meses depois, quando a empresa cresce e você percebe que não sobrou espaço pra crescer dentro do bloco que você reservou.

## O problema que subnetting resolve

Uma rede IPv4 classe C "cheia" (uma /24, tipo 192.168.1.0/24) te dá 254 endereços utilizáveis. Isso é ótimo se você tem uma rede só, com 200 dispositivos nela. É um desperdício brutal se você precisa de várias redes menores — por exemplo, um link ponto a ponto entre dois roteadores, que só precisa de dois endereços utilizáveis (você viu isso, na prática, lá no Lab 01, com a /30 entre R-MATRIZ e R-FILIAL). Reservar uma /24 inteira (254 endereços) pra um link que só usa 2 é desperdiçar 252 endereços que poderiam estar servindo outra rede.

Subnetting é o processo de pegar um bloco de endereço e dividir ele em blocos menores, mais adequados ao tamanho real de cada rede que você precisa criar.

## Máscara de sub-rede: o que ela realmente faz

A máscara de sub-rede define onde termina a parte de "rede" do endereço IP e começa a parte de "host". Numa /24 (255.255.255.0), os primeiros 24 bits identificam a rede, os últimos 8 identificam o host dentro dela — dando 2⁸ = 256 combinações possíveis, das quais 254 são utilizáveis (a primeira é o endereço da própria rede, a última é o endereço de broadcast).

A notação CIDR (a barra seguida de um número, tipo /24, /30, /27) é só uma forma mais compacta de expressar quantos bits são de rede. Você vê essa notação em praticamente todo comando que configurou nos labs até aqui — é a forma que o Arista EOS usa por padrão (diferente do Cisco IOS clássico, que usa a máscara escrita por extenso na maioria dos comandos de interface, mas volta pra formato de wildcard mask em OSPF e ACL — isso já foi coberto no cheatsheet comparativo, se você não lembra).

## VLSM: máscara de tamanho variável

VLSM (Variable Length Subnet Mask) é o nome técnico pra prática de usar máscaras diferentes dentro do mesmo bloco de endereço original, de acordo com a necessidade real de cada sub-rede — em vez de dividir tudo em blocos do mesmo tamanho fixo, que é o que "subnetting clássico" faria.

Exemplo prático, direto de uma situação real de projeto: imagina que você recebeu o bloco 192.168.0.0/22 (que cobre 192.168.0.0 até 192.168.3.255, mais de mil endereços) pra distribuir entre: uma rede de 200 hosts (Financeiro), uma rede de 50 hosts (TI), uma rede de 20 hosts (Vendas), e três links ponto a ponto entre roteadores, cada um precisando de só 2 endereços utilizáveis.

Se você dividisse tudo em blocos iguais (por exemplo, todos /26, dando 62 hosts utilizáveis cada), a rede de Financeiro (200 hosts) simplesmente não caberia num único bloco, e os links ponto a ponto desperdiçariam 60 endereços cada um. VLSM resolve isso permitindo que cada sub-rede tenha exatamente o tamanho que ela precisa: Financeiro pode ficar com uma /24 (254 hosts, cobrindo os 200 necessários com folga pra crescimento), TI com uma /26 (62 hosts), Vendas com uma /27 (30 hosts), e cada link ponto a ponto com uma /30 (2 hosts utilizáveis, exatamente o suficiente) — exatamente o padrão que você usou em todo link entre roteador nos Labs 01, 03, 05 e 08.

## Calculando na prática — o método que eu realmente uso

Esqueça fórmula decorada sem entender. O jeito que funciona, na prática, é pensar em potências de 2:

Pra descobrir quantos hosts utilizáveis uma máscara oferece: pegue o número de bits de host (32 menos o prefixo), eleve 2 a essa potência, subtraia 2 (um endereço pra identificar a rede, um pra broadcast).

Uma /27 tem 5 bits de host (32-27=5). 2⁵ = 32. Menos 2 (rede e broadcast) = 30 hosts utilizáveis.

Uma /30 tem 2 bits de host. 2² = 4. Menos 2 = 2 hosts utilizáveis — exatamente o suficiente pra um link ponto a ponto, nem um endereço a mais desperdiçado.

Pra alocar VLSM de forma organizada: sempre comece pela maior sub-rede necessária, aloque o bloco pra ela primeiro (a partir do início do espaço disponível), depois continue com a próxima maior, e assim sucessivamente, terminando pelos blocos menores (como os links ponto a ponto). Fazer isso na ordem inversa — começando pelos blocos pequenos — costuma resultar em fragmentação de espaço que trava você mais na frente, quando sobra um "buraco" de tamanho errado.

## O erro mais comum que eu vejo

Confundir "quantos hosts eu tenho hoje" com "quantos hosts eu vou ter". Dimensionar uma sub-rede exatamente no limite do que é necessário agora, sem margem de crescimento, é a razão mais comum de alguém precisar re-endereçar uma rede inteira um ano depois — um dos trabalhos mais chatos e arriscados que existem em rede em produção, porque significa reconfigurar host por host, atualizando DNS, DHCP, ACL, tudo junto. Vale sempre deixar folga.

## Perguntas para reflexão

Você recebeu o bloco 10.0.0.0/24 pra distribuir entre quatro sub-redes: uma de 100 hosts, uma de 50 hosts, uma de 20 hosts, e um link ponto a ponto entre dois roteadores. Qual máscara você usaria pra cada uma, em que ordem você alocaria os blocos dentro do /24, e sobra algum espaço não utilizado no final?

Segunda pergunta, ligada direto ao Lab 01: por que uma rede de link ponto a ponto entre dois roteadores quase sempre usa /30 (ou, em designs mais modernos que otimizam ainda mais, /31) em vez de qualquer máscara maior — o que exatamente uma /29 ali desperdiçaria, comparado à /30?
