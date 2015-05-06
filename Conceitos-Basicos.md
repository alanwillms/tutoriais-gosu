# Conceitos Básicos

## Posição Z

Todas as operações de desenho do Gosu aceitam um valor de ponto flutuante chamado "z" (tecnicamente, um `double`). As coisas que são desenhadas com uma posição "z" maior aparecerão por cima das que tem uma posição menor. Se duas coisas têm a mesma posição "z", elas serão desenhadas seguindo a ordem em que foram chamadas.

Se você não quiser usar a posição "z", apenas passe a mesma constante o tempo todo.

## Tiles

As funções relativas à criação de imagens aceitam um argumento booleano chamado `tileable`. Essa é uma consequência de usar aceleração de hardware. Tente reparar na sutil diferença entre essas duas imagens que foram ampliadas:

![Efeitos do parâmetro `tileable`](https://github.com/jlnr/gosu/wiki/hard_borders.png)

Quando você desenha uma imagem com os fatores de alongamento (_stretch_) diferentes de 1.0 (10.0 neste exemplo) ou em coordenadas estranhas, ela será interpolada, o que em geral é bem melhor do que ter uma imagem toda pixelizada.

Mas observe as bordas das imagens. A imagem da menina à esquerda foi criada com `tileable = false` (o padrão) e as bordas desvanecem. A imagem da menina à direita, que foi criada com `tileable = true`, não desvanece nas bordas, apenas termina. borders.

Embora a maioria das imagens não preciser ser "tileáveis", você sempre deve passar `true` para tiles de mapas.

## Ordem dos Cantos

Em todos os método que recebem todos os quatro cantos de um retângulo ou quadrilátero, exceto `ImageData::draw` (em C++), você pode ou passar coordenadas no sentido horário, ou coordenadas na ordem a seguir (no formato de uma letra Z):

![Ordem dos cantos no Gosu](https://github.com/jlnr/gosu/wiki/corner_indices.png)

## Desenhando com Cores

Quase todos os métodos de desenho de imagens aceitam cores para modulação. As cores de todos os pixels na imagem fonte serão multiplicadas por estas cores, onde um valor de canal de 255 corresponde ao valor máximo de 1.0. Isso significa que a modulação de cores pode ser usada somente para reduzir canais em particular de uma imagem.

A utilidade mais óbvia disso é fornecer uma cor com um valor alpha menor do que 255 de modo que a imagem seja desenhada transparente, mas você também pode usar isso para escurecer imagens ou desenhá-las com um matiz diferente (o que funciona melhor se a imagem original for em maior parte cinza).
