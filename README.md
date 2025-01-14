# pong_DE1-SoC_ARM

## Proposta do projeto
Neste projeto, programou-se, em Assembly, o jogo pong, rodando no processador ARM do kit de desenvolvimento DE1-Soc.

## Detalhes
### Sobre o kit
- Dos componentes presentes no kit, foram efetivamente utilizados neste projeto:
    - Processador Dual-core ARM Cortex-A9;
    - Saída VGA;
    - todos os quatro botões.
- A saída VGA suporta uma resolução de 640 x 480; no buffer de pixel, porém, trabalha-se com uma resolução de 320 x 240. A coordenada (0,0) se encontra porção superior esquerda da tela. O buffer de pixels abrange a porção da memória entre as posições 0x08000000 e 0x0803BE7E; e o mapeamento se dá da seguinte forma, considerando um representação em bits: [00001000000000][7 bits para y][9 bits para x][0]. Na posição referente a cada pixel, cuja tamanho é de meia palavra (16 bits), registra-se um valor que caracteriza sua cor em uma representação RGB, que se dá da seguinte forma: [5 bits para vermelho][6 bits para verde][5 bits para azul].
- A posição na memória de endereço 0xFF200050 refere-se aos botões. Cada um dos 4 bits nesse endereço reflete o estado de um dos botões -- bit 1 se acionado, bit 0 se não acionado.

### Sobre o funcionamento
- Na memória, são reservados os endereços 0x00001000, 0x00001004, 0x00001008, 0x0000100C, 0x00001010, 0x00001014, 0x00001018 para registrar as seguintes informações, respectivamente: posição superior (coordenada y) do player à direita, posição inferior (coordenada y) do player à direita, posição superior (coordenada y) do player à esquerda, posição inferior (coordenada y) do player à esquerda, coordenada y da bola, coordenada x da bola, direção da bola. O ponto referencial da posição da bola encontra-se em seu ponto superior esquerdo. No caso da direção, guarda-se um valor variando de 1 a 4 (1 para diagonal cima-direita, 2 para diagonal baixo-direita, 3 para diagonal baixo-esquerda, 4 para posição cima-esquerda)
- A princípio, pinta-se a tela inteira de preto. Em seguida, desenham-se, em branco, os players; primeiro, à esquerda (diagonal determinada pelos pontos (0, 80) e (12, 160)), depois, à direita (diagonal determinada pelos pontos (308, 80) e (320, 160)). Enfim, desenha-se a bola -- quadrado de 10x10 pixels cujo centro coincide com o centro da tela. Dispostos todos os elementos, salvam-se as informações de posicionamento nas respectivas posições de memória. E o jogo está pronto para começar.
- O jogo espera, então, que todos os 4 botões sejam apertados simultaneamente para iniciar. Enquanto espera, o valor referente à direção da bola muda constantemente, de forma que, iniciado o jogo, a bola pode seguir qualquer das 4 direções possíveis.
- Pressionados os botões, inicia-se o jogo. Ele consiste, em suma, em um loop nos seguintes moldes:
    - Primeiro, trabalha-se com o player à direita.
        - Verifica-se se o botão mais à direita está acionado; e, caso esteja, pinta-se de branco a porção um pixel acima do player e, em sequência, de preto os pixels brancos mais abaixo, de forma a gerar uma ilusão, pelo looping, de movimento.
        - Em seguida, verifica-se o botão imediatamente à esquerda do primeiro; e, em caso de acionamento, realiza-se o mesmo, mas invertido -- isto é, pinta-se de branco um pixel abaixo, e de preto os pixels brancos mais acima.
        - No caso de os dois botões serem simultaneamente acionados, move-se o player antes um pixel acima, depois, um abaixo; e é como se não houvesse movimento.
    - Após isso, trabalha-se, de forma semelhante, com o player à esquerda.
        - Verifica-se se o segundo botão da esquerda para a direita está acionado; e, caso esteja, pinta-se de branco a porção um pixel acima do player e, em sequência, de preto os pixels brancos mais abaixo, da mesma forma que com o player à direita.
        - Em seguida, verifica-se o botão mais à esquerda; e, em caso de acionamento, realiza-se o mesmo, mas invertido -- isto é, pinta-se de branco um pixel abaixo, e de preto os pixels brancos mais acima.
        - No caso de os dois botões serem simultaneamente acionados, move-se o player, igualmente ao da direita, antes um pixel acima, depois, um abaixo; e é como se não houvesse movimento.
    - Por fim, trabalha-se com a bola.
        - Primeiramente, verifica-se a direção atual da bola -- 1 = cima-direita, 2 = baixo-direita, 3 = baixo-esquerda, 4 = cima-esquerda. 
        - Verifica-se, então, um pixel acima (direções 1 e 4) ou abaixo (direções 2 e 3) para identificar se alcançou-se o limite da tela (y < 0 ou y > 240); e, caso o tenha, altera-se a direção segundo este esquema: de 1 para 2, de 2 para 1, de 3 para 4, de 4 para 3. 
        - Verifica-se, em seguida, se alcançaram-se os limites laterais (x > 340, para direções 1 e 2, ou x < 0, para direções 3 e 4); e, se for este o caso, o jogo é reinicializado, voltando ao estado de espera inicial -- aguardando o pressionar simultâneo dos 4 botões.
        - Após isso, analisa-se a colisão com algum dos players. Verifica-se, então, se um dos pixels à direita (direções 1 e 2) ou à esquerda (direções 3 e 4) da bola é branco; e, caso o seja, indicando, pois, a presença de um player para colidir, altera-se a direção segundo este esquema: de 1 para 4, de 2 para 3, de 3 para 2, de 4 para 1.
        - enfim, realizados todos os testes, pintam-se os pixels em preto e em branco para locomover a bola em conformidade com sua direção correta.
    - Terminado o ciclo de execução, para tornar o andamento temporal adequado à percepção humana, realiza-se um delay simples.

## Como jogar
- Pode-se jogar o jogo por um simulador (disponível em: https://cpulator.01xz.net/) ou no kit físico.
    - No caso do simulador, basta copiar o código, colá-lo no espaço adequado do simulador, compilá-lo e executá-lo; a tela e os botões, acionáveis com o click do mouse, estarão dispostos à direita. É muito provável, entretanto, que o jogo rode, neste caso, de forma demasiadamente lenta.

    
