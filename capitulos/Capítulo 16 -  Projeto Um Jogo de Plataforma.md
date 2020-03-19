# Capítulo 16 Projeto: Um Jogo de Plataforma

> Toda realidade é um jogo.
>
> *—Iain Banks, o jogador dos jogos*

![Imagem de um personagem do jogo pulando sobre lava](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/chapter_picture_16.jpg)

Grande parte do meu fascínio inicial por computadores, como o de muitas crianças nerds, tinha a ver com jogos de computador. Fui atraído pelos minúsculos mundos simulados que eu poderia manipular e em que histórias (meio que) se desenrolaram - suponho mais, por causa da maneira como projetei minha imaginação neles do que por causa das possibilidades que elas realmente ofereciam.

Eu não desejo uma carreira em programação de jogos para ninguém. Assim como a indústria da música, a discrepância entre o número de jovens ansiosos que desejam trabalhar nela e a demanda real por tais pessoas cria um ambiente bastante prejudicial. Mas escrever jogos por diversão é divertido.

Este capítulo mostrará a implementação de um pequeno jogo de plataforma. Jogos de plataforma (ou jogos de "pular e correr") são jogos que esperam que o jogador mova uma figura através de um mundo, que geralmente é bidimensional e visto de lado, pulando sobre as coisas.

## O jogo

Nosso jogo será basicamente baseado em [Dark Blue,](http://www.lessmilk.com/games/10) de Thomas Palef. Eu escolhi esse jogo porque é divertido e minimalista e porque pode ser construído sem muito código. Se parece com isso:

![O jogo Dark Blue](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/darkblue.png)

A caixa escura representa o jogador, cuja tarefa é coletar as caixas amarelas (moedas), evitando o material vermelho (lava). Um nível é concluído quando todas as moedas foram coletadas.

O jogador pode andar com as teclas de seta esquerda e direita e pode pular com a seta para cima. O salto é uma especialidade desse personagem do jogo. Pode atingir várias vezes a sua própria altura e pode mudar de direção no ar. Isso pode não ser totalmente realista, mas ajuda a dar ao jogador a sensação de estar no controle direto do avatar na tela.

O jogo consiste em um plano de fundo estático, disposto como uma grade, com os elementos móveis sobrepostos nesse plano de fundo. Cada campo na grade é vazio, sólido ou lava. Os elementos móveis são o jogador, moedas e certas peças de lava. As posições desses elementos não são restritas à grade - suas coordenadas podem ser fracionárias, permitindo movimentos suaves.

## A tecnologia

Usaremos o DOM do navegador para exibir o jogo e leremos as informações do usuário manipulando os principais eventos.

O código relacionado à tela e ao teclado é apenas uma pequena parte do trabalho que precisamos fazer para criar este jogo. Como tudo se parece com caixas coloridas, o desenho é simples: criamos elementos DOM e usamos estilos para fornecer a cor, tamanho e posição do plano de fundo.

Podemos representar o plano de fundo como uma tabela, pois é uma grade imutável de quadrados. Os elementos móveis podem ser sobrepostos usando elementos absolutamente posicionados.

Em jogos e outros programas que devem animar gráficos e responder à entrada do usuário sem demora perceptível, a eficiência é importante. Embora o DOM não tenha sido projetado originalmente para gráficos de alto desempenho, ele é realmente melhor do que o esperado. Você viu algumas animações no [capítulo 14](https://eloquentjavascript.net/14_dom.html#animation) . Em uma máquina moderna, um jogo simples como esse funciona bem, mesmo que não nos preocupemos muito com a otimização.

No [próximo capítulo](https://eloquentjavascript.net/17_canvas.html) , exploraremos outra tecnologia de navegador, a ``tag, que fornece uma maneira mais tradicional de desenhar gráficos, trabalhando em termos de formas e pixels em vez de elementos DOM.

## Níveis

Queremos uma maneira legível por humanos e editável por humanos para especificar níveis. Como não há problema em tudo começar em uma grade, poderíamos usar grandes strings nas quais cada caractere representa um elemento - uma parte da grade de segundo plano ou um elemento em movimento.

O plano para um nível pequeno pode ser assim:

```js
let simpleLevelPlan = `
......................
..#................#..
..#..............=.#..
..#.........o.o....#..
..#.@......#####...#..
..#####............#..
......#++++++++++++#..
......##############..
......................`;
```

Os pontos são espaços vazios, os `#`caracteres hash ( ) são paredes e os sinais de adição são lava. A posição inicial do jogador é o sinal de arroba ( `@`). Todo caractere O é uma moeda, e o sinal de igual ( `=`) na parte superior é um bloco de lava que se move para frente e para trás horizontalmente.

Apoiaremos dois tipos adicionais de lava em movimento: o caractere de tubo ( `|`) cria bolhas em movimento vertical e `v`indica lava *pingando -* lava em movimento vertical que não salta para frente e para trás, mas apenas se move para baixo, pulando de volta para sua posição inicial quando bate no chão.

Um jogo inteiro consiste em vários níveis que o jogador deve completar. Um nível é concluído quando todas as moedas foram coletadas. Se o jogador tocar em lava, o nível atual será restaurado à sua posição inicial e o jogador poderá tentar novamente.

## Lendo um nível

A classe a seguir armazena um objeto de nível. Seu argumento deve ser a string que define o nível.

```js
class Level {
  constructor(plan) {
    let rows = plan.trim().split("\n").map(l => [...l]);
    this.height = rows.length;
    this.width = rows[0].length;
    this.startActors = [];

    this.rows = rows.map((row, y) => {
      return row.map((ch, x) => {
        let type = levelChars[ch];
        if (typeof type == "string") return type;
        this.startActors.push(
          type.create(new Vec(x, y), ch));
        return "empty";
      });
    });
  }
}
```

O `trim`método é usado para remover os espaços em branco no início e no final da sequência do plano. Isso permite que nosso plano de exemplo comece com uma nova linha para que todas as linhas fiquem diretamente abaixo uma da outra. A seqüência restante é dividida em caracteres de nova linha e cada linha é espalhada em uma matriz, produzindo matrizes de caracteres.

Portanto, `rows`contém uma matriz de matrizes de caracteres, as linhas do plano. Podemos derivar a largura e altura do nível a partir delas. Mas ainda devemos separar os elementos móveis da grade de fundo. Chamaremos *atores de* elementos em movimento . Eles serão armazenados em uma variedade de objetos. O fundo será uma matriz de matrizes de cadeias, que prende os tipos de campos, tais como `"empty"`, `"wall"`, ou `"lava"`.

Para criar essas matrizes, mapeamos as linhas e, em seguida, o conteúdo delas. Lembre-se de que `map`passa o índice da matriz como um segundo argumento para a função de mapeamento, que indica as coordenadas xe y de um determinado caractere. As posições no jogo serão armazenadas como pares de coordenadas, com o canto superior esquerdo a 0,0 e cada quadrado de fundo sendo 1 unidade de altura e largura.

Para interpretar os caracteres no plano, o `Level`construtor usa o `levelChars`objeto, que mapeia elementos de segundo plano para seqüências de caracteres e caracteres de ator para classes. Quando `type`é uma classe de ator, seu `create`método estático é usado para criar um objeto ao qual é adicionado `startActors`e a função de mapeamento retorna `"empty"`para esse quadrado de plano de fundo.

A posição do ator é armazenada como um `Vec`objeto. Este é um vetor bidimensional, um objeto com `x`e `y`propriedades, como visto nos exercícios do [Capítulo 6](https://eloquentjavascript.net/06_object.html#exercise_vector) .

À medida que o jogo corre, os atores acabam em lugares diferentes ou até desaparecem completamente (como as moedas quando coletadas). Usaremos uma `State`classe para rastrear o estado de um jogo em execução.

```js
class State {
  constructor(level, actors, status) {
    this.level = level;
    this.actors = actors;
    this.status = status;
  }

  static start(level) {
    return new State(level, level.startActors, "playing");
  }

  get player() {
    return this.actors.find(a => a.type == "player");
  }
}
```

A `status`propriedade mudará para `"lost"`ou `"won"`quando o jogo terminar.

Essa é novamente uma estrutura de dados persistente - a atualização do estado do jogo cria um novo estado e deixa o antigo intacto.

## Atores

Os objetos de ator representam a posição e o estado atuais de um determinado elemento em movimento em nosso jogo. Todos os objetos de ator estão em conformidade com a mesma interface. Sua `pos`propriedade mantém as coordenadas do canto superior esquerdo do elemento e sua `size`propriedade mantém seu tamanho.

Depois, eles têm um `update`método, que é usado para calcular seu novo estado e posição após um determinado intervalo de tempo. Ele simula o que o ator faz - movendo-se em resposta às teclas de seta do jogador e saltando para a frente e para trás na lava - e retorna um novo objeto de ator atualizado.

A `type`propriedade contém uma string que identifica o tipo de ator- `"player"`, `"coin"`ou `"lava"`. Isso é útil ao desenhar o jogo - a aparência do retângulo desenhado para um ator é baseada em seu tipo.

As classes de ator têm um `create`método estático usado pelo `Level`construtor para criar um ator a partir de um personagem no plano de nível. São dadas as coordenadas do personagem e do próprio personagem, o que é necessário porque a `Lava`classe lida com vários caracteres diferentes.

Essa é a `Vec`classe que usaremos para nossos valores bidimensionais, como a posição e o tamanho dos atores.

```js
class Vec {
  constructor(x, y) {
    this.x = x; this.y = y;
  }
  plus(other) {
    return new Vec(this.x + other.x, this.y + other.y);
  }
  times(factor) {
    return new Vec(this.x * factor, this.y * factor);
  }
}
```

O `times`método dimensiona um vetor por um determinado número. Será útil quando precisarmos multiplicar um vetor de velocidade por um intervalo de tempo para obter a distância percorrida durante esse tempo.

Os diferentes tipos de atores recebem suas próprias classes, pois seu comportamento é muito diferente. Vamos definir essas classes. Nós vamos chegar aos `update`métodos deles mais tarde.

A classe de jogador possui uma propriedade `speed`que armazena sua velocidade atual para simular o momento e a gravidade.

```js
class Player {
  constructor(pos, speed) {
    this.pos = pos;
    this.speed = speed;
  }

  get type() { return "player"; }

  static create(pos) {
    return new Player(pos.plus(new Vec(0, -0.5)),
                      new Vec(0, 0));
  }
}

Player.prototype.size = new Vec(0.8, 1.5);
```

Como um jogador tem um quadrado e meio de altura, sua posição inicial é definida como meio quadrado acima da posição em que o `@`personagem apareceu. Dessa forma, seu fundo se alinha com o fundo do quadrado em que apareceu.

A `size`propriedade é a mesma para todas as instâncias de `Player`, portanto, a armazenamos no protótipo e não nas próprias instâncias. Poderíamos ter usado um getter como `type`, mas isso criaria e retornaria um novo `Vec`objeto toda vez que a propriedade for lida, o que seria um desperdício. (As strings, sendo imutáveis, não precisam ser recriadas toda vez que são avaliadas.)

Ao construir um `Lava`ator, precisamos inicializar o objeto de maneira diferente, dependendo do personagem em que ele se baseia. A lava dinâmica se move na velocidade atual até atingir um obstáculo. Nesse ponto, se tiver uma `reset`propriedade, retornará à sua posição inicial (pingando). Caso contrário, inverterá a velocidade e continuará na outra direção (saltando).

O `create`método examina o personagem que o `Level`construtor passa e cria o ator de lava apropriado.

```js
class Lava {
  constructor(pos, speed, reset) {
    this.pos = pos;
    this.speed = speed;
    this.reset = reset;
  }

  get type() { return "lava"; }

  static create(pos, ch) {
    if (ch == "=") {
      return new Lava(pos, new Vec(2, 0));
    } else if (ch == "|") {
      return new Lava(pos, new Vec(0, 2));
    } else if (ch == "v") {
      return new Lava(pos, new Vec(0, 3), pos);
    }
  }
}

Lava.prototype.size = new Vec(1, 1);
```

`Coin`atores são relativamente simples. Eles geralmente apenas se sentam no lugar deles. Mas, para animar um pouco o jogo, eles recebem uma "oscilação", um leve movimento vertical de vaivém. Para rastrear isso, um objeto de moeda armazena uma posição base e uma `wobble`propriedade que rastreia a fase do movimento de salto. Juntos, eles determinam a posição real da moeda (armazenada na `pos`propriedade).

```js
class Coin {
  constructor(pos, basePos, wobble) {
    this.pos = pos;
    this.basePos = basePos;
    this.wobble = wobble;
  }

  get type() { return "coin"; }

  static create(pos) {
    let basePos = pos.plus(new Vec(0.2, 0.1));
    return new Coin(basePos, basePos,
                    Math.random() * Math.PI * 2);
  }
}

Coin.prototype.size = new Vec(0.6, 0.6);
```

No [capítulo 14](https://eloquentjavascript.net/14_dom.html#sin_cos) , vimos que `Math.sin`nos dá a coordenada y de um ponto em um círculo. Essa coordenada vai e volta em uma forma de onda suave à medida que avançamos ao longo do círculo, o que torna a função seno útil para modelar um movimento ondulado.

Para evitar uma situação em que todas as moedas se movam para cima e para baixo de forma síncrona, a fase inicial de cada moeda é aleatória. A *fase* da `Math.sin`onda de, a largura de uma onda que produz, é 2π. Multiplicamos o valor retornado por `Math.random`esse número para dar à moeda uma posição inicial aleatória na onda.

Agora podemos definir o `levelChars`objeto que mapeia os caracteres do plano para os tipos de grade de segundo plano ou classes de ator.

```js
const levelChars = {
  ".": "empty", "#": "wall", "+": "lava",
  "@": Player, "o": Coin,
  "=": Lava, "|": Lava, "v": Lava
};
```

Isso nos dá todas as partes necessárias para criar uma `Level`instância.

```js
let simpleLevel = new Level(simpleLevelPlan);
console.log(`${simpleLevel.width} by ${simpleLevel.height}`);
// → 22 by 9
```

A tarefa a seguir é exibir esses níveis na tela e modelar o tempo e o movimento dentro deles.

## Encapsulamento como um fardo

A maior parte do código deste capítulo não se preocupa muito com o encapsulamento por dois motivos. Primeiro, o encapsulamento requer um esforço extra. Aumenta os programas e requer a introdução de conceitos e interfaces adicionais. Como há tanto código que você pode lançar em um leitor antes que seus olhos se vejam, fiz um esforço para manter o programa pequeno.

Segundo, os vários elementos deste jogo estão tão intimamente ligados que, se o comportamento de um deles mudar, é improvável que qualquer um dos outros seja capaz de permanecer o mesmo. As interfaces entre os elementos acabariam codificando muitas suposições sobre o funcionamento do jogo. Isso os torna muito menos eficazes - sempre que você altera uma parte do sistema, ainda precisa se preocupar com a maneira como isso afeta as outras partes, porque suas interfaces não cobririam a nova situação.

Alguns *pontos de corte* em um sistema se prestam bem à separação por meio de interfaces rigorosas, mas outros não. Tentar encapsular algo que não é um limite adequado é uma maneira de desperdiçar muita energia. Quando você está cometendo esse erro, geralmente percebe que suas interfaces estão ficando desajeitadamente grandes e detalhadas e que precisam ser alteradas com frequência, à medida que o programa evolui.

Há uma coisa que *vai* encapsular, e que é o subsistema de desenho. A razão para isso é que exibiremos o mesmo jogo de uma maneira diferente no [próximo capítulo](https://eloquentjavascript.net/17_canvas.html#canvasdisplay) . Ao colocar o desenho atrás de uma interface, podemos carregar o mesmo programa de jogo e conectar um novo módulo de exibição.

## Desenhando

O encapsulamento do código de desenho é feito através da definição de um objeto de *exibição* , que exibe um determinado nível e estado. O tipo de exibição que definimos neste capítulo é chamado `DOMDisplay`porque usa elementos DOM para mostrar o nível.

Usaremos uma folha de estilos para definir as cores reais e outras propriedades fixas dos elementos que compõem o jogo. Também seria possível atribuir diretamente à `style`propriedade dos elementos quando os criamos, mas isso produziria programas mais detalhados.

A seguinte função auxiliar fornece uma maneira sucinta de criar um elemento e fornecer alguns atributos e nós filhos:

```js
function elt(name, attrs, ...children) {
  let dom = document.createElement(name);
  for (let attr of Object.keys(attrs)) {
    dom.setAttribute(attr, attrs[attr]);
  }
  for (let child of children) {
    dom.appendChild(child);
  }
  return dom;
}
```

Uma exibição é criada fornecendo a ele um elemento pai ao qual deve se anexar e um objeto de nível.

```js
class DOMDisplay {
  constructor(parent, level) {
    this.dom = elt("div", {class: "game"}, drawGrid(level));
    this.actorLayer = null;
    parent.appendChild(this.dom);
  }

  clear() { this.dom.remove(); }
}
```

A grade de fundo do nível, que nunca muda, é desenhada uma vez. Os atores são redesenhados toda vez que a exibição é atualizada com um determinado estado. A `actorLayer`propriedade será usada para rastrear o elemento que contém os atores, para que possam ser facilmente removidos e substituídos.

Nossas coordenadas e tamanhos são rastreados em unidades de grade, onde um tamanho ou distância de 1 significa um bloco de grade. Ao definir o tamanho dos pixels, teremos que escalar essas coordenadas - tudo no jogo seria ridiculamente pequeno em um único pixel por quadrado. A `scale`constante fornece o número de pixels que uma única unidade ocupa na tela.

```js
const scale = 20;

function drawGrid(level) {
  return elt("table", {
    class: "background",
    style: `width: ${level.width * scale}px`
  }, ...level.rows.map(row =>
    elt("tr", {style: `height: ${scale}px`},
        ...row.map(type => elt("td", {class: type})))
  ));
}
```

Como mencionado, o plano de fundo é desenhado como um ``elemento. Isso corresponde perfeitamente à estrutura da `rows`propriedade do nível - cada linha da grade é transformada em uma linha da tabela ( elemento). As cadeias na grade são usadas como nomes de classe para os `elementos da célula da tabela ( ). O operador spread (ponto triplo) é usado para passar matrizes de nós filhos para `elt`argumentos separados.

O CSS a seguir faz a tabela parecer com o plano de fundo que queremos:

```css
.background    { background: rgb(52, 166, 251);
                 table-layout: fixed;
                 border-spacing: 0;              }
.background td { padding: 0;                     }
.lava          { background: rgb(255, 100, 100); }
.wall          { background: white;              }
```

Alguns deles ( `table-layout`, `border-spacing`e `padding`) são usados para suprimir o comportamento padrão indesejado. Não queremos que o layout da tabela dependa do conteúdo de suas células e não queremos espaço entre as células da tabela ou preenchimento dentro delas.

A `background`regra define a cor do plano de fundo. O CSS permite que as cores sejam especificadas como palavras ( `white`) ou com um formato como `rgb(R, G, B)`, onde os componentes vermelho, verde e azul da cor são separados em três números de 0 a 255. Assim, em `rgb(52, 166, 251)`, o componente vermelho é 52, o verde é 166 e o azul é 251. Como o componente azul é o maior, a cor resultante será azulada. Você pode ver que, na `.lava`regra, o primeiro número (vermelho) é o maior.

Nós desenhamos cada ator criando um elemento DOM para ele e definindo a posição e o tamanho desse elemento com base nas propriedades do ator. Os valores devem ser multiplicados `scale`para passar de unidades de jogo para pixels.

```js
function drawActors(actors) {
  return elt("div", {}, ...actors.map(actor => {
    let rect = elt("div", {class: `actor ${actor.type}`});
    rect.style.width = `${actor.size.x * scale}px`;
    rect.style.height = `${actor.size.y * scale}px`;
    rect.style.left = `${actor.pos.x * scale}px`;
    rect.style.top = `${actor.pos.y * scale}px`;
    return rect;
  }));
}
```

Para atribuir a um elemento mais de uma classe, separamos os nomes das classes por espaços. No código CSS mostrado a seguir, a `actor`classe dá aos atores sua posição absoluta. O nome do tipo é usado como uma classe extra para dar uma cor. Não precisamos definir a `lava`classe novamente porque estamos reutilizando a classe para os quadrados da grade de lava que definimos anteriormente.

```css
.actor  { position: absolute;            }
.coin   { background: rgb(241, 229, 89); }
.player { background: rgb(64, 64, 64);   }
```

O `syncState`método é usado para fazer a exibição mostrar um determinado estado. Primeiro, ele remove os gráficos antigos do ator, se houver, e depois redesenha os atores em suas novas posições. Pode ser tentador tentar reutilizar os elementos DOM para atores, mas, para fazer esse trabalho, precisaríamos de muita contabilidade adicional para associar atores a elementos DOM e garantir que removemos elementos quando seus atores desaparecerem. Como normalmente haverá apenas alguns atores no jogo, redesenhar todos eles não é caro.

```js
DOMDisplay.prototype.syncState = function(state) {
  if (this.actorLayer) this.actorLayer.remove();
  this.actorLayer = drawActors(state.actors);
  this.dom.appendChild(this.actorLayer);
  this.dom.className = `game ${state.status}`;
  this.scrollPlayerIntoView(state);
};
```

Ao adicionar o status atual do nível como um nome de classe ao wrapper, podemos estilizar o ator de maneira ligeiramente diferente quando o jogo é vencido ou perdido, adicionando uma regra CSS que entra em vigor somente quando o jogador tem um elemento ancestral com uma determinada classe.

```css
.lost .player {
  background: rgb(160, 64, 64);
}
.won .player {
  box-shadow: -4px -7px 8px white, 4px -7px 8px white;
}
```

Depois de tocar na lava, a cor do jogador fica vermelha escura, sugerindo abrasador. Quando a última moeda é coletada, adicionamos duas sombras brancas borradas - uma no canto superior esquerdo e outra no canto superior direito - para criar um efeito de halo branco.

Não podemos assumir que o nível sempre se encaixa na *janela de visualização* - o elemento no qual desenhamos o jogo. É por isso que a `scrollPlayerIntoView`ligação é necessária. Ele garante que, se o nível estiver saliente fora da janela de visualização, rolaremos essa janela de visualização para garantir que o jogador esteja próximo ao centro. O CSS a seguir fornece um tamanho máximo para o elemento DOM do quebra-cabeças do jogo e garante que qualquer coisa que saia da caixa do elemento não seja visível. Também atribuímos uma posição relativa para que os atores dentro dela sejam posicionados em relação ao canto superior esquerdo do nível.

```css
.game {
  overflow: hidden;
  max-width: 600px;
  max-height: 450px;
  position: relative;
}
```

No `scrollPlayerIntoView`método, encontramos a posição do jogador e atualizamos a posição de rolagem do elemento de empacotamento. Mudamos a posição de rolagem manipulando esse elemento `scrollLeft`e `scrollTop`propriedades quando o jogador está muito perto da borda.

```js
DOMDisplay.prototype.scrollPlayerIntoView = function(state) {
  let width = this.dom.clientWidth;
  let height = this.dom.clientHeight;
  let margin = width / 3;

  // The viewport
  let left = this.dom.scrollLeft, right = left + width;
  let top = this.dom.scrollTop, bottom = top + height;

  let player = state.player;
  let center = player.pos.plus(player.size.times(0.5))
                         .times(scale);

  if (center.x < left + margin) {
    this.dom.scrollLeft = center.x - margin;
  } else if (center.x > right - margin) {
    this.dom.scrollLeft = center.x + margin - width;
  }
  if (center.y < top + margin) {
    this.dom.scrollTop = center.y - margin;
  } else if (center.y > bottom - margin) {
    this.dom.scrollTop = center.y + margin - height;
  }
};
```

A maneira como o centro do jogador é encontrado mostra como os métodos em nosso `Vec`tipo permitem que os cálculos com objetos sejam escritos de maneira relativamente legível. Para encontrar o centro do ator, adicionamos sua posição (seu canto superior esquerdo) e metade do seu tamanho. Esse é o centro das coordenadas de nível, mas precisamos dele em coordenadas de pixel, para multiplicarmos o vetor resultante pela nossa escala de exibição.

Em seguida, uma série de testes verifica se a posição do jogador não está fora do intervalo permitido. Observe que, às vezes, isso define coordenadas de rolagem sem sentido que estão abaixo de zero ou além da área de rolagem do elemento. Tudo bem - o DOM os restringirá a valores aceitáveis. Definir `scrollLeft`-10 fará com que ele se torne 0.

Teria sido um pouco mais simples tentar sempre rolar o player para o centro da janela de exibição. Mas isso cria um efeito bastante dissonante. Enquanto você pula, a visão muda constantemente para cima e para baixo. É mais agradável ter uma área "neutra" no meio da tela, onde você pode se mover sem causar rolagem.

Agora somos capazes de exibir nosso pequeno nível.

```html
<link rel="stylesheet" href="css/game.css">

<script>
  let simpleLevel = new Level(simpleLevelPlan);
  let display = new DOMDisplay(document.body, simpleLevel);
  display.syncState(State.start(simpleLevel));
</script>
```

A ``tag, quando usada com `rel="stylesheet"`, é uma maneira de carregar um arquivo CSS em uma página. O arquivo `game.css`contém os estilos necessários para o nosso jogo.

## Movimento e colisão

Agora estamos no ponto em que podemos começar a adicionar movimento - o aspecto mais interessante do jogo. A abordagem básica, adotada pela maioria dos jogos como esse, é dividir o tempo em pequenos passos e, para cada passo, mover os atores por uma distância correspondente à sua velocidade multiplicada pelo tamanho do intervalo de tempo. Mediremos o tempo em segundos, para que as velocidades sejam expressas em unidades por segundo.

Mover coisas é fácil. A parte difícil é lidar com as interações entre os elementos. Quando o jogador atinge uma parede ou piso, ele não deve simplesmente se mover através dela. O jogo deve perceber quando um determinado movimento faz com que um objeto atinja outro objeto e responda de acordo. Para paredes, o movimento deve ser interrompido. Ao bater uma moeda, ela deve ser coletada. Ao tocar na lava, o jogo deve ser perdido.

Resolver isso para o caso geral é uma grande tarefa. Você pode encontrar bibliotecas, geralmente chamadas de *mecanismos de física* , que simulam a interação entre objetos físicos em duas ou três dimensões. Adotaremos uma abordagem mais modesta neste capítulo, lidando apenas com colisões entre objetos retangulares e de uma maneira bastante simplista.

Antes de mover o jogador ou um bloco de lava, testamos se o movimento o levaria para dentro de uma parede. Se isso acontecer, simplesmente cancelamos a moção por completo. A resposta a tal colisão depende do tipo de ator - o jogador irá parar, enquanto um bloco de lava se recuperará.

Essa abordagem exige que nossos passos de tempo sejam bastante pequenos, pois fará com que o movimento pare antes que os objetos realmente se toquem. Se os passos do tempo (e, portanto, os passos do movimento) forem muito grandes, o jogador acabará pairando uma distância perceptível acima do solo. Outra abordagem, sem dúvida melhor, mas mais complicada, seria encontrar o ponto exato da colisão e mudar para lá. Adotaremos a abordagem simples e esconderemos seus problemas, garantindo que a animação prossiga em pequenas etapas.

Este método nos diz se um retângulo (especificado por uma posição e um tamanho) toca um elemento de grade do tipo especificado.

```js
Level.prototype.touches = function(pos, size, type) {
  var xStart = Math.floor(pos.x);
  var xEnd = Math.ceil(pos.x + size.x);
  var yStart = Math.floor(pos.y);
  var yEnd = Math.ceil(pos.y + size.y);

  for (var y = yStart; y < yEnd; y++) {
    for (var x = xStart; x < xEnd; x++) {
      let isOutside = x < 0 || x >= this.width ||
                      y < 0 || y >= this.height;
      let here = isOutside ? "wall" : this.rows[y][x];
      if (here == type) return true;
    }
  }
  return false;
};
```

O método calcula o conjunto de quadrados da grade com os quais o corpo se sobrepõe usando `Math.floor`e `Math.ceil`em suas coordenadas. Lembre-se de que os quadrados da grade são de 1 por 1 unidades. Ao arredondar os lados de uma caixa para cima e para baixo, obtemos o intervalo de quadrados de fundo em que a caixa toca.

![Localizando colisões em uma grade](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/game-grid.svg)

Passamos pelo bloco de quadrados da grade encontrado arredondando as coordenadas e retornamos `true`quando um quadrado correspondente é encontrado. Quadrados fora do nível são sempre tratados `"wall"`para garantir que o jogador não possa deixar o mundo e que não tentaremos acidentalmente ler fora dos limites da nossa `rows`matriz.

O `update`método state usa `touches`para descobrir se o jogador está tocando na lava.

```js
State.prototype.update = function(time, keys) {
  let actors = this.actors
    .map(actor => actor.update(time, this, keys));
  let newState = new State(this.level, actors, this.status);

  if (newState.status != "playing") return newState;

  let player = newState.player;
  if (this.level.touches(player.pos, player.size, "lava")) {
    return new State(this.level, actors, "lost");
  }

  for (let actor of actors) {
    if (actor != player && overlap(actor, player)) {
      newState = actor.collide(newState);
    }
  }
  return newState;
};
```

O método passa por uma etapa do tempo e uma estrutura de dados que informa quais chaves estão sendo mantidas pressionadas. A primeira coisa que faz é chamar o `update`método em todos os atores, produzindo uma variedade de atores atualizados. Os atores também recebem o tempo, as chaves e o estado, para que possam basear sua atualização naqueles. Somente o jogador realmente lê as teclas, já que esse é o único ator controlado pelo teclado.

Se o jogo já tiver terminado, nenhum processamento adicional deverá ser feito (o jogo não poderá ser ganho depois de perdido, ou vice-versa). Caso contrário, o método testa se o jogador está tocando a lava de fundo. Nesse caso, o jogo está perdido, e terminamos. Finalmente, se o jogo ainda estiver em andamento, ele verá se outros atores se sobrepõem ao jogador.

A sobreposição entre atores é detectada com a `overlap`função Ele pega dois objetos de ator e retorna verdadeiro quando eles tocam - o que acontece quando eles se sobrepõem ao longo do eixo x e do eixo y.

```js
function overlap(actor1, actor2) {
  return actor1.pos.x + actor1.size.x > actor2.pos.x &&
         actor1.pos.x < actor2.pos.x + actor2.size.x &&
         actor1.pos.y + actor1.size.y > actor2.pos.y &&
         actor1.pos.y < actor2.pos.y + actor2.size.y;
}
```

Se algum ator se sobrepuser, seu `collide`método terá a chance de atualizar o estado. Tocar em um ator de lava define o status do jogo para `"lost"`. As moedas desaparecem quando você as toca e define o status para `"won"`quando elas são a última moeda do nível.

```js
Lava.prototype.collide = function(state) {
  return new State(state.level, state.actors, "lost");
};

Coin.prototype.collide = function(state) {
  let filtered = state.actors.filter(a => a != this);
  let status = state.status;
  if (!filtered.some(a => a.type == "coin")) status = "won";
  return new State(state.level, filtered, status);
};
```

## Atualizações de atores

Os `update`métodos dos objetos atores tomam como argumentos a etapa do tempo, o objeto de estado e um `keys`objeto. A do `Lava`tipo ator ignora o `keys`objeto.

```js
Lava.prototype.update = function(time, state) {
  let newPos = this.pos.plus(this.speed.times(time));
  if (!state.level.touches(newPos, this.size, "wall")) {
    return new Lava(newPos, this.speed, this.reset);
  } else if (this.reset) {
    return new Lava(this.reset, this.speed, this.reset);
  } else {
    return new Lava(this.pos, this.speed.times(-1));
  }
};
```

Este `update`método calcula uma nova posição adicionando o produto do intervalo de tempo e a velocidade atual à sua posição antiga. Se nenhum obstáculo bloquear essa nova posição, ele se moverá para lá. Se houver um obstáculo, o comportamento depende do tipo do bloco de lava - a lava pingando tem uma `reset`posição, para a qual salta para trás quando bate em alguma coisa. A lava saltando inverte sua velocidade multiplicando-a por -1, para que comece a se mover na direção oposta.

As moedas usam seu `update`método para oscilar. Eles ignoram colisões com a grade, uma vez que estão simplesmente oscilando dentro de sua própria praça.

```js
const wobbleSpeed = 8, wobbleDist = 0.07;

Coin.prototype.update = function(time) {
  let wobble = this.wobble + time * wobbleSpeed;
  let wobblePos = Math.sin(wobble) * wobbleDist;
  return new Coin(this.basePos.plus(new Vec(0, wobblePos)),
                  this.basePos, wobble);
};
```

A `wobble`propriedade é incrementada para rastrear o tempo e, em seguida, usada como argumento `Math.sin`para encontrar a nova posição na onda. A posição atual da moeda é calculada a partir de sua posição base e um deslocamento com base nessa onda.

Isso deixa o próprio jogador. O movimento do jogador é tratado separadamente por eixo, porque bater no chão não deve impedir o movimento horizontal e bater na parede não deve parar de cair ou pular.

```js
const playerXSpeed = 7;
const gravity = 30;
const jumpSpeed = 17;

Player.prototype.update = function(time, state, keys) {
  let xSpeed = 0;
  if (keys.ArrowLeft) xSpeed -= playerXSpeed;
  if (keys.ArrowRight) xSpeed += playerXSpeed;
  let pos = this.pos;
  let movedX = pos.plus(new Vec(xSpeed * time, 0));
  if (!state.level.touches(movedX, this.size, "wall")) {
    pos = movedX;
  }

  let ySpeed = this.speed.y + time * gravity;
  let movedY = pos.plus(new Vec(0, ySpeed * time));
  if (!state.level.touches(movedY, this.size, "wall")) {
    pos = movedY;
  } else if (keys.ArrowUp && ySpeed > 0) {
    ySpeed = -jumpSpeed;
  } else {
    ySpeed = 0;
  }
  return new Player(pos, new Vec(xSpeed, ySpeed));
};
```

O movimento horizontal é calculado com base no estado das teclas de seta esquerda e direita. Quando não há parede bloqueando a nova posição criada por esse movimento, ela é usada. Caso contrário, a posição antiga é mantida.

O movimento vertical funciona de maneira semelhante, mas precisa simular salto e gravidade. A velocidade vertical do jogador ( `ySpeed`) é primeiro acelerada para explicar a gravidade.

Verificamos paredes novamente. Se não acertarmos, a nova posição é usada. Se não *é* uma parede, há dois resultados possíveis. Quando a seta para cima é pressionada *e* estamos descendo (o que atingimos está abaixo de nós), a velocidade é definida para um valor negativo relativamente grande. Isso faz com que o jogador pule. Se não for esse o caso, o jogador simplesmente esbarra em algo, e a velocidade é definida como zero.

A força da gravidade, a velocidade de salto e praticamente todas as outras constantes deste jogo foram definidas por tentativa e erro. Testei valores até encontrar uma combinação que gostei.

## Teclas de rastreamento

Para um jogo como esse, não queremos que as teclas entrem em vigor uma vez por pressionamento de tecla. Em vez disso, queremos que o efeito deles (mover a figura do jogador) permaneça ativo enquanto for mantido.

Precisamos configurar um manipulador de chaves que armazene o estado atual das teclas de seta esquerda, direita e seta para cima. Também queremos chamar `preventDefault`essas teclas para que elas não rolem a página.

A função a seguir, quando recebe uma matriz de nomes de chaves, retornará um objeto que rastreia a posição atual dessas chaves. Ele registra os manipuladores de eventos `"keydown"`e os `"keyup"`eventos e, quando o código da chave está presente no conjunto de códigos que está rastreando, atualiza o objeto.

```js
function trackKeys(keys) {
  let down = Object.create(null);
  function track(event) {
    if (keys.includes(event.key)) {
      down[event.key] = event.type == "keydown";
      event.preventDefault();
    }
  }
  window.addEventListener("keydown", track);
  window.addEventListener("keyup", track);
  return down;
}

const arrowKeys =
  trackKeys(["ArrowLeft", "ArrowRight", "ArrowUp"]);
```

A mesma função de manipulador é usada para ambos os tipos de eventos. Ele examina a `type`propriedade do objeto de evento para determinar se o estado da chave deve ser atualizado para true ( `"keydown"`) ou false ( `"keyup"`).

## Executando o jogo

A `requestAnimationFrame`função, que vimos no [Capítulo 14](https://eloquentjavascript.net/14_dom.html#animationFrame) , fornece uma boa maneira de animar um jogo. Mas sua interface é bastante primitiva - para usá-lo, precisamos rastrear o horário em que nossa função foi chamada da última vez e chamar `requestAnimationFrame`novamente após cada quadro.

Vamos definir uma função auxiliar que agrupe essas partes chatas em uma interface conveniente e nos permita simplesmente chamar `runAnimation`, fornecendo uma função que espera uma diferença de tempo como argumento e desenha um único quadro. Quando a função de quadro retorna o valor `false`, a animação para.

```js
function runAnimation(frameFunc) {
  let lastTime = null;
  function frame(time) {
    if (lastTime != null) {
      let timeStep = Math.min(time - lastTime, 100) / 1000;
      if (frameFunc(timeStep) === false) return;
    }
    lastTime = time;
    requestAnimationFrame(frame);
  }
  requestAnimationFrame(frame);
}
```

Eu configurei uma etapa de quadro máximo de 100 milissegundos (um décimo de segundo). Quando a guia ou janela do navegador com nossa página estiver oculta, as `requestAnimationFrame`chamadas serão suspensas até que a guia ou janela seja exibida novamente. Nesse caso, a diferença entre `lastTime`e `time`será o tempo inteiro em que a página ficou oculta. O avanço desse jogo em um único passo pareceria bobo e poderia causar efeitos colaterais estranhos, como o jogador cair no chão.

A função também converte as etapas de tempo em segundos, que são uma quantidade mais fácil de se pensar do que milissegundos.

A `runLevel`função pega um `Level`objeto e um construtor de exibição e retorna uma promessa. Ele exibe o nível (in `document.body`) e permite que o usuário o reproduza. Quando o nível terminar (perdido ou vencido), `runLevel`aguarda mais um segundo (para permitir que o usuário veja o que acontece) e depois limpa a tela, interrompe a animação e resolve a promessa do status final do jogo.

```js
function runLevel(level, Display) {
  let display = new Display(document.body, level);
  let state = State.start(level);
  let ending = 1;
  return new Promise(resolve => {
    runAnimation(time => {
      state = state.update(time, arrowKeys);
      display.syncState(state);
      if (state.status == "playing") {
        return true;
      } else if (ending > 0) {
        ending -= time;
        return true;
      } else {
        display.clear();
        resolve(state.status);
        return false;
      }
    });
  });
}
```

Um jogo é uma sequência de níveis. Sempre que o jogador morre, o nível atual é reiniciado. Quando um nível é concluído, passamos para o próximo nível. Isso pode ser expresso pela seguinte função, que utiliza uma matriz de planos de nível (strings) e um construtor de exibição:

```js
async function runGame(plans, Display) {
  for (let level = 0; level < plans.length;) {
    let status = await runLevel(new Level(plans[level]),
                                Display);
    if (status == "won") level++;
  }
  console.log("You've won!");
}
```

Como fizemos `runLevel`uma promessa de devolução, ela `runGame`pode ser escrita usando uma `async`função, como mostra o [Capítulo 11](https://eloquentjavascript.net/11_async.html) . Ele retorna outra promessa, que resolve quando o jogador termina o jogo.

Há um conjunto de planos de nível disponíveis na `GAME_LEVELS`encadernação na [caixa de proteção deste capítulo](https://eloquentjavascript.net/code#16) . Esta página os alimenta para `runGame`iniciar um jogo real.

```html
<link rel="stylesheet" href="css/game.css">

<body>
  <script>
    runGame(GAME_LEVELS, DOMDisplay);
  </script>
</body>
```

Veja se você pode vencê-los. Eu me diverti bastante construindo-os.

## Exercícios

### Fim de jogo

É tradicional para jogos de plataforma que o jogador comece com um número limitado de *vidas* e subtraia uma vida cada vez que morre. Quando o jogador está sem vida, o jogo reinicia desde o início.

Ajuste `runGame`para implementar vidas. Faça o jogador começar com três. Emita o número atual de vidas (usando `console.log`) toda vez que um nível é iniciado.

```html
<link rel="stylesheet" href="css/game.css">

<body>
<script>
  // The old runGame function. Modify it...
  async function runGame(plans, Display) {
    for (let level = 0; level < plans.length;) {
      let status = await runLevel(new Level(plans[level]),
                                  Display);
      if (status == "won") level++;
    }
    console.log("You've won!");
  }
  runGame(GAME_LEVELS, DOMDisplay);
</script>
</body>
```

### Pausando o jogo

Torne possível pausar (suspender) e cancelar a pausa do jogo pressionando a tecla Esc.

Isso pode ser feito alterando a `runLevel`função para usar outro manipulador de eventos do teclado e interrompendo ou retomando a animação sempre que a tecla Esc for pressionada.

A `runAnimation`interface pode não parecer adequada à primeira vista, mas é se você reorganizar a maneira como a `runLevel`chama.

Quando você tem esse trabalho, há outra coisa que você pode tentar. A maneira como registramos os manipuladores de eventos do teclado é um tanto problemática. O `arrowKeys`objeto é atualmente um acordo global vinculativo, e seus manipuladores de eventos são mantidas ao redor, mesmo quando nenhum jogo está sendo executado. Você poderia dizer que eles *vazam* do nosso sistema. Estenda `trackKeys`para fornecer uma maneira de cancelar o registro de seus manipuladores e, em seguida, altere `runLevel`para registrar seus manipuladores quando iniciar e cancele o registro novamente quando terminar.

```html
<link rel="stylesheet" href="css/game.css">

<body>
<script>
  // The old runLevel function. Modify this...
  function runLevel(level, Display) {
    let display = new Display(document.body, level);
    let state = State.start(level);
    let ending = 1;
    return new Promise(resolve => {
      runAnimation(time => {
        state = state.update(time, arrowKeys);
        display.syncState(state);
        if (state.status == "playing") {
          return true;
        } else if (ending > 0) {
          ending -= time;
          return true;
        } else {
          display.clear();
          resolve(state.status);
          return false;
        }
      });
    });
  }
  runGame(GAME_LEVELS, DOMDisplay);
</script>
</body>
```

Uma animação pode ser interrompida retornando `false`da função atribuída a `runAnimation`. Para continuar, ligue `runAnimation`novamente.

Portanto, precisamos comunicar o fato de que estamos pausando o jogo para a função atribuída `runAnimation`. Para isso, você pode usar uma ligação à qual o manipulador de eventos e essa função têm acesso.

Ao encontrar uma maneira de cancelar o registro dos manipuladores registrados por `trackKeys`, lembre-se de que *exatamente o* mesmo valor da função que foi passado `addEventListener`deve ser passado para `removeEventListener`remover com êxito um manipulador. Portanto, o `handler`valor da função criada em `trackKeys`deve estar disponível para o código que cancela o registro dos manipuladores.

Você pode adicionar uma propriedade ao objeto retornado por `trackKeys`, contendo esse valor da função ou um método que lida diretamente com o cancelamento de registro.

### Um monstro

É tradicional que os jogos de plataforma tenham inimigos que você pode pular em cima de para derrotar. Este exercício pede que você adicione esse tipo de ator ao jogo.

Vamos chamar de monstro. Monstros se movem apenas horizontalmente. Você pode fazê-los se mover na direção do jogador, saltar para frente e para trás como lava horizontal ou ter qualquer padrão de movimento desejado. A classe não precisa lidar com a queda, mas deve garantir que o monstro não atravesse as paredes.

Quando um monstro toca o jogador, o efeito depende se o jogador está pulando em cima dele ou não. Você pode aproximar isso verificando se o fundo do jogador está próximo ao topo do monstro. Se for esse o caso, o monstro desaparece. Caso contrário, o jogo está perdido.

```html
<link rel="stylesheet" href="css/game.css">
<style>.monster { background: purple }</style>

<body>
  <script>
    // Complete the constructor, update, and collide methods
    class Monster {
      constructor(pos, /* ... */) {}

      get type() { return "monster"; }

      static create(pos) {
        return new Monster(pos.plus(new Vec(0, -1)));
      }

      update(time, state) {}

      collide(state) {}
    }

    Monster.prototype.size = new Vec(1.2, 2);

    levelChars["M"] = Monster;

    runLevel(new Level(`
..................................
.################################.
.#..............................#.
.#..............................#.
.#..............................#.
.#...........................o..#.
.#..@...........................#.
.##########..............########.
..........#..o..o..o..o..#........
..........#...........M..#........
..........################........
..................................
`), DOMDisplay);
  </script>
</body>
```

Se você deseja implementar um tipo de movimento com estado, como salto, certifique-se de armazenar o estado necessário no objeto ator - inclua-o como argumento construtor e adicione-o como uma propriedade.

Lembre-se de que `update`retorna um *novo* objeto, em vez de alterar o antigo.

Ao lidar com uma colisão, encontre o jogador `state.actors`e compare sua posição com a posição do monstro. Para obter a *parte inferior* do player, você deve adicionar o tamanho vertical à posição vertical. A criação de um estado atualizado será semelhante quer `Coin`do `collide`método (removendo o ator) ou `Lava`'s (alteração do status para `"lost"`), dependendo da posição do jogador.