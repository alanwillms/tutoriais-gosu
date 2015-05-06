# Loop Principal da Janela

(Observação: essa é uma tradução do texto [Window Main Loop](https://github.com/jlnr/gosu/wiki/Window-Main-Loop)).

Geralmente você criará uma sub-classe de `Gosu::Window` e sobrescreverá as partes que você precisar. Então você cria uma instância dessa classe e chama o método `show` dela. O que acontece é o seguinte:

![Loop Principal](https://github.com/jlnr/gosu/wiki/main_loop.png)

Os exemplos do que você pode fazer nesses eventos são mostrados no [Tutorial em Ruby](Tutorial-Ruby.md) e no [Tutorial em C++](https://github.com/jlnr/gosu/wiki/C---Tutorial).

Existem alguns detalhes que podem lhe interessar:

  * `draw` é _muito_ flexível. O fluxo geral é mostrado acima, mas `draw` geralmente é chamada toda vez que o sistema operacional quiser redesenhar a janela. Isso pode acontecer logo depois de chamar `show`, ou duas vezes entre chamadas ao método `update`, ou pode nem ser chamado por um frame se a janela estiver coberta.
  * Desta forma _você não consegue saber_ de modo algum qual _callback_ será chamado primeiro. O método `initialize` (ou o construtor se for C++) _deve_ definir um estado válido. Se o método `update` conter `if player == nil then respawn()`, então `initialize` também deve conter `respawn()`. Você pode simplesmente chamar `update()` na última linha do construtor, se isso ajudar.
  * Uma vez que `draw` é tão flexível, ele só deveria renderizar o estado atual das coisas e não mudar nada, nem mesmo avançar animações. Se você escrever `draw` de modo funcional, somente-leitura, então você estará seguro.
  * No método `update`, você pode querer checar por `button_down?` (ou `input().down()` em C++) para pegar o estado mais recente de um botão. Já que os eventos de entrada logicamente devem ser disparados _antes_ de `button_down?` ser atualizado, segue-se que todos eles precisam ocorrer antes de `update`. A notícia triste é que isso significa que você não pode reagir diretamente aos botões recém-pressionados em `draw`; uma chamada `update` sempre acontecerá entre elas.
  * Uma vez que `TextInput` pode executar um _callback_ do jogo (_callback_ `filter`), ele não é bufferizado e pode acontecer a qualquer momento durante o loop. Você também não pode confiar em `text()` ser o mesmo entre os métodos `update` e `draw`.
  * No entanto, não se assuste, pois nada em Gosu é _multi-thread_ e nunca nenhum _callback_ é interrompido por causa de outro!

# Degradação de Performance

O que acontece se o sistema estiver lento demais para rodar o jogo na taxa de frames esperada? Existem duas soluções principais.

## Lógica Discreta

A princípio você pode escrever a lógica do seu jogo no estilo dos jogos de exemplo. A cada frame lógico (`update`), o jogador se move por uma quantidade fixa de pixels. As animações também avançam a cada frame lógico, etc. Isso resultará em um código baseado em inteiros, discreto. A CPU também gostará disso, especialmente com Ruby ou com processadores ARM.

No entanto, se o sistema estiver lento demais para o seu jogo, você tem poucas opções:

  * Você pode reduzir a quantidade de partículas ou reduzir as "decorações" de outro jeito. Isso só funciona se tiver muitos efeitos visuais pra começar.
  * Você pode pular frames retornando `false` no callback `needs_redraw? `/`needsRedraw()`. No entanto você tem que tomar cuidado para não pular muitos frames de uma vez só.

## Física com Tempo Delta (Δt)

Você também pode escrever seus jogos checando o retorno de `Gosu::milliseconds()` entre uma chamada a `update` e outra, e então atualizar o estado do seu jogo baseado na diferença de tempo (tempo delta, Δt). Isso permite que a física seja escrita de um jeito parecido com a física da escola (`posicao += celeridade * dt`) e isso é muito comum ao trabalhar com _engines_ de física.

Esse estilo naturalmente funciona melhor com uma taxa de frames baixa, mas pode necessitar de mais reflexão, e os cálculos podem ser mais lentos em geral porque números `float` são mais naturais aqui.

Essa definitivamente é uma escolha de acordo com o caso, e o Gosu tenta suportar ambos os estilos de programação.
