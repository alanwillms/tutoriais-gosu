# Tutorial da LibGosu com Ruby

(Observação: essa é uma tradução do [tutorial em inglês](https://github.com/jlnr/gosu/wiki/Ruby-Tutorial). Para este tutorial, baixe os seguintes _assets_: [Space.png](https://raw.githubusercontent.com/jlnr/gosu/master/examples/media/Space.png), [Starfighter.bmp](https://raw.githubusercontent.com/jlnr/gosu/master/examples/media/Star.png), [Star.png](https://raw.githubusercontent.com/jlnr/gosu/master/examples/media/Star.png) e [Beep.wav](https://raw.githubusercontent.com/jlnr/gosu/master/examples/media/Beep.wav).)

## Código fonte

Este e outros jogos de exemplo estão inclusos com o restante da biblioteca. Por exemplo, no Mac OS X 10.5, os exemplos podem ser encontrados em `/Library/Ruby/Gems/1.8/gems/gosu-<version>/examples`.

Se você não tem um editor que suporta a execução direta do arquivo (TextMate, SciTE…), então basta dar um `cd` nesse diretório e executar com `ruby Tutorial.rb`.

## Vamos aos Negócios
### 1. Sobrescrevendo os callbacks da janela

A forma mais fácil de criar uma aplicação Gosu completa é escrever uma nova classe que deriva de `Gosu::Window` (consulte a documentação para uma descrição completa de sua interface). Uma classe `GameWindow` mínima se parece com a seguinte:

```ruby
require 'gosu'

class GameWindow < Gosu::Window
  def initialize
    super 640, 480, false
    self.caption = "Gosu Tutorial Game"
  end
  
  def update
  end
  
  def draw
  end
end

window = GameWindow.new
window.show
```
O construtor inicializa a classe base `Gosu::Window`. Os parâmetros passados criam uma janela de 640 por 480 pixels, sem estar em tela cheia. Ele também muda o título da janela, que até então estava vazio.

`update()` e `draw()` sobrescrevem os métodos de mesmo nome da `Gosu::Window`'. O método `update()` é chamado 60 vezes por segundo (por padrão) e deve conter a lógica principal do jogo: mover objetos, tratar colisões, etc.

O método `draw()` é chamado depois dele, e também em qualquer momento que a janela precisar redesenhar por outros motivos, e também eventualmente pode ser ignorado se a taxa de FPS (frames por segundo) baixar muito. Ele deve conter o código para redesenhar a tela inteira, e nenhuma lógica.

Então segue o programa principal. Uma janela é criada e seu método `show()` é chamado, que não retorna nada até que a janela tenha sido fechada pelo usuário ou pelo próprio código. Ta-dã – agora você tem uma pequena janela preta com o título que você escolheu!

Um diagrama do loop principal é explicado na página [Loop Principal da Janela](Loop-Principal-Janela.md).

### 2. Usando Imagens

```ruby
require 'gosu'

class GameWindow < Gosu::Window
  def initialize
    super(640, 480, false)
    self.caption = "Gosu Tutorial Game"
    
    @background_image = Gosu::Image.new(self, "media/Space.png", true)
  end
  
  def update
  end
  
  def draw
    @background_image.draw(0, 0, 0)
  end
end

window = GameWindow.new
window.show
```

`Gosu::Image#initialize` recebe três argumentos. Primeiramente, como todos os recursos de mídia, ela está atrelada a uma janela (`self`). Todos os recursos Gosu precisam de uma `Window` para a inicialização e conterão uma referência interna à mesma. Em segundo lugar, o nome do arquivo da imagem é passado. O terceiro argumento especifica se a imagem deve ser criada com bordas opacas. Consulte a página [Basic Concepts](https://github.com/jlnr/gosu/wiki/Basic-Concepts) para uma explicação melhor.

Conforme mencionado na seção anterior, o método `draw()` da janela é o lugar certo para desenhar tudo, então é aqui que vamos desenhar nossa imagem de fundo.

A maioria dos argumentos é óbvia. A imagem é desenhada na posição (0;0) - o terceiro argumento é a posição Z; novamente, consulte a página [Basic Concepts](https://github.com/jlnr/gosu/wiki/Basic-Concepts).

#### 2.1 Jogador & movimento

Segue aqui uma classe `Player` (_jogador_) simples:

```ruby
class Player
  def initialize(window)
    @image = Gosu::Image.new(window, "media/Starfighter.bmp", false)
    @x = @y = @vel_x = @vel_y = @angle = 0.0
    @score = 0
  end

  def warp(x, y)
    @x, @y = x, y
  end
  
  def turn_left
    @angle -= 4.5
  end
  
  def turn_right
    @angle += 4.5
  end
  
  def accelerate
    @vel_x += Gosu::offset_x(@angle, 0.5)
    @vel_y += Gosu::offset_y(@angle, 0.5)
  end
  
  def move
    @x += @vel_x
    @y += @vel_y
    @x %= 640
    @y %= 480
    
    @vel_x *= 0.95
    @vel_y *= 0.95
  end

  def draw
    @image.draw_rot(@x, @y, 1, @angle)
  end
end
```

Há algumas observações a fazer sobre isso:

![Ângulos com Gosu](https://github.com/jlnr/gosu/wiki/corner_indices.png)

  * `Player#accelerate` utiliza os métodos `offset_x` e `offset_y`. Eles são semelhantes a aquilo para o qual muitas pessoas usam senos e cossenos: por exemplo, se alguma coisa se mover 100 pixels em um ângulo de 30°, se deslocaria `offset_x(30, 100)` pixels horizontalmente e `offset_y(30, 100)` pixels verticalmente.
  * Ao carregar arquivos BMP, o Gosu substitui a cor `0xff00ff` (fúcsia/magenta; aquele cor-de-rosa feio) com pixels transparentes.
  * Perceba que o método `draw_rot` coloca o *centro* da imagem em (x; y) - *e não* o canto superior esquerdo como o método `draw` faz! Isso pode ser controlado pelos argumentos `center_x` e `center_y` se você quiser.
  * O jogador é desenhado em z=1, ou seja, por cima do fundo (é óbvio). Vamos substituir estes números fixos com algo melhor depois.
  * Além disso, consulte o [rdoc](https://www.libgosu.org/rdoc/Gosu/Image.html) para conhecer todos os métodos de desenho e seus argumentos.

#### 2.2 Integrando o Jogador com a Janela

```ruby
class GameWindow < Gosu::Window
  def initialize
    super(640, 480, false)
    self.caption = "Gosu Tutorial Game"

    @background_image = Gosu::Image.new(self, "media/Space.png", true)

    @player = Player.new(self)
    @player.warp(320, 240)
  end

  def update
    if button_down? Gosu::KbLeft or button_down? Gosu::GpLeft then
      @player.turn_left
    end
    if button_down? Gosu::KbRight or button_down? Gosu::GpRight then
      @player.turn_right
    end
    if button_down? Gosu::KbUp or button_down? Gosu::GpButton0 then
      @player.accelerate
    end
    @player.move
  end

  def draw
    @player.draw
    @background_image.draw(0, 0, 0);
  end

  def button_down(id)
    if id == Gosu::KbEscape
      close
    end
  end
end

window = GameWindow.new
window.show
```

Como você pode ver, nós introduzimos o uso de um teclado ou joystick!

De forma semelhante aos métodos `update()` e `draw()`, a `Gosu::Window` fornece dois métodos: `button_down(id)` e `button_up(id)`, que podem ser sobrescritos e não fazem nada por padrão. Aqui nós sobrescrevemos `button_down(id)` para fechar a janela com a tecla `ESC`.  (Para ver uma lista com as constantes de teclas e botões, consulte o [rdoc](https://www.libgosu.org/rdoc/Gosu.html)).

Embora obter feedback de botões pressionados é algo útil para eventos que acontecem uma vez só como interações com a interface, pulos e digitação, isso é meio inútil para ações que duram vários frames - por exemplo, mover-se segurando teclas pressionadas. É aqui que o método `update()` entra em campo, onde só chama os métodos de movimento do objeto `player`. Se você executar o código desse exemplo, você vai conseguir voar com sua nave por aí!

### 3. Animações Simples

Primeiramente, vamos nos livrar dos números que fixamos para as posições Z. De agora em diante vamos utilizar as seguintes constantes:

```ruby
module ZOrder
  Background, Stars, Player, UI = *0..3
end
```

O que é uma animação? É uma sequência de imagens – então vamos usar a classe `Array` do Ruby para armazená-las. (Para um jogo de verdade, não tem como escapar de escrever classes específicas para as necessidades individuais do jogo, mas nós vamos mostrar a solução simples por enquanto.)

Vamos introduzir as estrelas, que são o objeto principal deste tutorial. As estrelas aparecem aleatoriamente na tela e vivem sua vida animada até que o jogador as colete. A definição da classe `Star` é muito simples:

```ruby
class Star
  attr_reader :x, :y

  def initialize(animation)
    @animation = animation
    @color = Gosu::Color.new(0xff000000)
    @color.red = rand(256 - 40) + 40
    @color.green = rand(256 - 40) + 40
    @color.blue = rand(256 - 40) + 40
    @x = rand * 640
    @y = rand * 480
  end

  def draw  
    img = @animation[Gosu::milliseconds / 100 % @animation.size];
    img.draw(@x - img.width / 2.0, @y - img.height / 2.0,
        ZOrder::Stars, 1, 1, @color, :add)
  end
end
```

Uma vez que não queremos que toda estrela carregue a animação novamente, nós não podemos fazer isso no construtor, então vamos passá-la de algum outro lugar. (A classe `Window` carregará a animação em cerca de três parágrafos).

Para mostrar um frame diferente da animação a cada 100 milisegundos, a hora retornada por `Gosu::milliseconds` é dividida por 100 e então encontramos seu módulo pelo número de frames da animação. Essa é própria imagem que será desenhada, centralizada na posição da estrela e alterada por uma cor aleatório que geramos no construtor.

Agora vamos adicionar um código simples para o jogador coletar as estrelas de um _array_:

```ruby
class Player
  ...
  def score
    @score
  end

  def collect_stars(stars)
    if stars.reject! {|star| Gosu::distance(@x, @y, star.x, star.y) < 35 } then
      @score += 1
    end
  end
end
```

Agora vamos estender a classe `Window` para carregar a animação, criar as novas estrelas, fazer o jogador coletá-las e desenhar as que sobram:

```ruby
...
class Window < Gosu::Window
  def initialize
    super(640, 480, false)
    self.caption = "Gosu Tutorial Game"

    @background_image = Gosu::Image.new(self, "media/Space.png", true)

    @player = Player.new(self)
    @player.warp(320, 240)

    @star_anim = Gosu::Image::load_tiles(self, "media/Star.png", 25, 25, false)
    @stars = Array.new
  end

  def update
    ...
    @player.move
    @player.collect_stars(@stars)

    if rand(100) < 4 and @stars.size < 25 then
      @stars.push(Star.new(@star_anim))
    end
  end

  def draw
    @background_image.draw(0, 0, ZOrder::Background)
    @player.draw
    @stars.each { |star| star.draw }
  end
  ...
```

Feito! Agora você pode coletar estrelas.

### 4. Som e Texto

Por fim, queremos escrever a pontuação atual usando uma fonte bitmap e tocar um som de "bipe" toda vez que o jogador coletar uma estrela. A classe `Window` se encarregará da parte do texto, carregando uma fonte com 20 pixels de altura:

```ruby
class Window < Gosu::Window
  def initialize
    ...
    @font = Gosu::Font.new(self, Gosu::default_font_name, 20)
  end

  ...

  def draw
    @background_image.draw(0, 0, ZOrder::Background)
    @player.draw
    @stars.each { |star| star.draw }
    @font.draw("Score: #{@player.score}", 10, 10, ZOrder::UI, 1.0, 1.0, 0xffffff00)
  end
end
```

O que sobrou pro jogador? Ah é: um contador para a pontuação, carregar o som e reproduzí-lo.

```ruby
class Player
  attr_reader :score

  def initialize(window)
    @image = Gosu::Image.new(window, "media/Starfighter.bmp", false)
    @beep = Gosu::Sample.new(window, "media/Beep.wav")
    @x = @y = @vel_x = @vel_y = @angle = 0.0
    @score = 0
  end

  ...

  def collect_stars(stars)
    stars.reject! do |star|
      if Gosu::distance(@x, @y, star.x, star.y) < 35 then
        @score += 10
        @beep.play
        true
      else
        false
      end
    end
  end
end
```

Como você pode ver, carregar e tocar efeitos sonoros não poderia ser mais fácil! Consulte o [rdoc](https://www.libgosu.org/rdoc/Gosu/Sample.html) para formas mais poderosas de reproduzir sons – brincar com o volume, posição e tom.

É isso! O resto só depende da sua imaginação. Se você não consegue imaginar como isso é o suficiente para criar jogos, então dê uma olhada nos [exemplos do fórum][https://www.libgosu.org/cgi-bin/mwf/board_show.pl?bid=2].
