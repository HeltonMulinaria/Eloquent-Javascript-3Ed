# Capítulo 19 Projeto: Um editor de pixel art

> Eu olho para as muitas cores diante de mim. Eu olho para a minha tela em branco. Então, tento aplicar cores como palavras que modelam poemas, como notas que modelam música.
>
> *—Joan Miro*

![Imagem de um mosaico de azulejos](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/chapter_picture_19.jpg)

O material dos capítulos anteriores fornece todos os elementos necessários para criar um aplicativo Web básico. Neste capítulo, faremos exatamente isso.

Nosso aplicativo será um programa de desenho de pixels, no qual você poderá modificar uma imagem pixel por pixel, manipulando uma visualização ampliada, mostrada como uma grade de quadrados coloridos. Você pode usar o programa para abrir arquivos de imagem, rabiscar neles com o mouse ou outro dispositivo indicador e salvá-los. É assim que será:

![A interface do editor de pixels, com pixels coloridos na parte superior e vários controles abaixo desse](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/pixel_editor.png)

Pintar em um computador é ótimo. Você não precisa se preocupar com materiais, habilidades ou talentos. Você apenas começa a sujar.

## Componentes

A interface do aplicativo mostra um grande ``elemento na parte superior, com vários campos de formulário abaixo. O usuário desenha a imagem selecionando uma ferramenta em um ``campo e, em seguida, clicando, tocando ou arrastando pela tela. Existem ferramentas para desenhar pixels únicos ou retângulos, preencher uma área e escolher uma cor da imagem.

Estruturaremos a interface do editor como um número de *componentes* , objetos que são responsáveis por uma parte do DOM e que podem conter outros componentes dentro deles.

O estado do aplicativo consiste na imagem atual, na ferramenta selecionada e na cor selecionada. Vamos configurar as coisas para que o estado resida em um único valor, e os componentes da interface sempre baseiem a aparência do estado atual.

Para ver por que isso é importante, vamos considerar a alternativa - distribuir partes do estado por toda a interface. Até certo ponto, isso é mais fácil de programar. Podemos apenas inserir um campo de cores e ler seu valor quando precisamos conhecer a cor atual.

Porém, adicionamos o seletor de cores - uma ferramenta que permite clicar na imagem para selecionar a cor de um determinado pixel. Para manter o campo de cores mostrando a cor correta, essa ferramenta precisa saber que ela existe e atualizá-la sempre que escolher uma nova cor. Se você adicionar outro local que torne a cor visível (talvez o cursor do mouse possa mostrá-la), será necessário atualizar o código de alteração de cor para mantê-la sincronizada.

Com efeito, isso cria um problema em que cada parte da interface precisa conhecer todas as outras partes, o que não é muito modular. Para pequenas aplicações como a deste capítulo, isso pode não ser um problema. Para projetos maiores, pode se transformar em um verdadeiro pesadelo.

Para evitar esse pesadelo por princípio, seremos rigorosos com o *fluxo de dados* . Há um estado, e a interface é desenhada com base nesse estado. Um componente da interface pode responder às ações do usuário atualizando o estado; nesse momento, os componentes têm a chance de se sincronizar com esse novo estado.

Na prática, cada componente é configurado para que, ao receber um novo estado, ele também notifique seus componentes filhos, na medida em que eles precisem ser atualizados. Configurar isso é um pouco complicado. Tornar isso mais conveniente é o principal ponto de venda de muitas bibliotecas de programação de navegadores. Mas para um aplicativo pequeno como esse, podemos fazer isso sem essa infraestrutura.

As atualizações no estado são representadas como objetos, que chamaremos de *ações* . Os componentes podem criar essas ações e *despachá* -las - fornecê-las para uma função central de gerenciamento de estado. Essa função calcula o próximo estado, após o qual os componentes da interface se atualizam para esse novo estado.

Estamos assumindo a tarefa complicada de executar uma interface do usuário e aplicar alguma estrutura a ela. Embora as peças relacionadas ao DOM ainda estejam cheias de efeitos colaterais, elas são sustentadas por uma espinha dorsal conceitualmente simples: o ciclo de atualização do estado. O estado determina a aparência do DOM e a única maneira de os eventos do DOM mudarem de estado é enviando ações para o estado.

Existem *muitas* variantes dessa abordagem, cada uma com seus próprios benefícios e problemas, mas sua ideia central é a mesma: as mudanças de estado devem passar por um único canal bem definido, não acontecer em todo o lugar.

Nossos componentes serão classes em conformidade com uma interface. O construtor recebe um estado - que pode ser todo o estado do aplicativo ou algum valor menor, se não precisar de acesso a tudo - e usa isso para construir uma `dom`propriedade. Este é o elemento DOM que representa o componente. A maioria dos construtores também aceita outros valores que não mudam ao longo do tempo, como a função que eles podem usar para despachar uma ação.

Cada componente possui um `syncState`método usado para sincronizá-lo com um novo valor de estado. O método usa um argumento, o estado, que é do mesmo tipo que o primeiro argumento para seu construtor.

## O Estado

O estado da aplicação será um objecto com `picture`, `tool`e `color`propriedades. A própria imagem é um objeto que armazena a largura, a altura e o conteúdo de pixels da imagem. Os pixels são armazenados em uma matriz, da mesma maneira que a classe da matriz do [Capítulo 6 -](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2006%20-%20A%20Vida%20Secreta%20dos%20Objetos.md) linha por linha, de cima para baixo.

```css
class Picture {
  constructor(width, height, pixels) {
    this.width = width;
    this.height = height;
    this.pixels = pixels;
  }
  static empty(width, height, color) {
    let pixels = new Array(width * height).fill(color);
    return new Picture(width, height, pixels);
  }
  pixel(x, y) {
    return this.pixels[x + y * this.width];
  }
  draw(pixels) {
    let copy = this.pixels.slice();
    for (let {x, y, color} of pixels) {
      copy[x + y * this.width] = color;
    }
    return new Picture(this.width, this.height, copy);
  }
}
```

Queremos poder tratar uma imagem como um valor imutável, por razões às quais voltaremos mais adiante neste capítulo. Mas às vezes também precisamos atualizar um monte de pixels por vez. Para ser capaz de fazer isso, a classe tem um `draw`método que espera uma matriz de pixels-objetos atualizados com `x`, `y`e `color`propriedades e cria uma nova imagem com esses pixels substituídas. Esse método usa `slice`sem argumentos para copiar toda a matriz de pixels - o início da fatia é padronizado como 0 e o final é padronizado para o comprimento da matriz.

O `empty`método usa duas partes da funcionalidade da matriz que não vimos antes. O `Array`construtor pode ser chamado com um número para criar uma matriz vazia do comprimento especificado. O `fill`método pode ser usado para preencher essa matriz com um determinado valor. Eles são usados para criar uma matriz na qual todos os pixels têm a mesma cor.

As cores são armazenadas como seqüências de caracteres contendo códigos de cores CSS tradicionais compostos por um sinal de hash ( `#`) seguido por seis dígitos hexadecimais (base 16) - dois para o componente vermelho, dois para o componente verde e dois para o componente azul. Essa é uma maneira um tanto enigmática e inconveniente de escrever cores, mas é o formato que o campo de entrada de cores HTML usa e pode ser usado na `fillColor`propriedade de um contexto de desenho de tela, portanto, pelas formas como usaremos cores neste programa , é prático o suficiente.

O preto, onde todos os componentes são zero, é gravado `"#000000"`e o rosa brilhante se parece `"#ff00ff"`, onde os componentes vermelho e azul têm o valor máximo de 255, gravados `ff`em dígitos hexadecimais (que usam *a* a *f* para representar os dígitos de 10 a 15).

Permitiremos que a interface despache ações como objetos cujas propriedades substituem as propriedades do estado anterior. O campo de cores, quando o usuário o altera, pode despachar um objeto como , do qual essa função de atualização pode calcular um novo estado.`{color: field.value}`

```js
function updateState(state, action) {
  return Object.assign({}, state, action);
}
```

Esse padrão bastante complicado, no qual `Object.assign`é usado primeiro para adicionar as propriedades de `state`um objeto vazio e, em seguida, substituir algumas delas com as propriedades de `action`, é comum no código JavaScript que usa objetos imutáveis. Uma notação mais conveniente para isso, na qual o operador de ponto triplo é usado para incluir todas as propriedades de outro objeto em uma expressão de objeto, está nos estágios finais de ser padronizado. Com essa adição, você pode escrever . No momento da escrita, isso ainda não funciona em todos os navegadores.`{...state, ...action}`

## Edifício DOM

Uma das principais coisas que os componentes de interface fazem é criar a estrutura DOM. Novamente, não queremos usar diretamente os métodos DOM detalhados para isso, então aqui está uma versão ligeiramente expandida da `elt`função:

```js
function elt(type, props, ...children) {
  let dom = document.createElement(type);
  if (props) Object.assign(dom, props);
  for (let child of children) {
    if (typeof child != "string") dom.appendChild(child);
    else dom.appendChild(document.createTextNode(child));
  }
  return dom;
}
```

A principal diferença entre esta versão e a que usamos no [Capítulo 16](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2016%20-%20%20Projeto%20Um%20Jogo%20de%20Plataforma.md) é que ela atribui *propriedades* a nós DOM, não a *atributos* . Isso significa que não pode usá-lo para definir atributos arbitrários, mas *pode* usá-lo para definir propriedades cujo valor não é uma string, tais como `onclick`, o que pode ser definida como uma função para registrar um manipulador de eventos clique.

Isso permite o seguinte estilo de registrar manipuladores de eventos:

```html
<body>
  <script>
    document.body.appendChild(elt("button", {
      onclick: () => console.log("click")
    }, "The button"));
  </script>
</body>
```

## O Canvas

O primeiro componente que definiremos é a parte da interface que exibe a imagem como uma grade de caixas coloridas. Esse componente é responsável por duas coisas: mostrar uma imagem e comunicar eventos de ponteiro nessa imagem para o restante do aplicativo.

Como tal, podemos defini-lo como um componente que conhece apenas a imagem atual, não o estado completo do aplicativo. Por não saber como o aplicativo como um todo funciona, ele não pode despachar diretamente as ações. Em vez disso, ao responder a eventos de ponteiro, ele chama uma função de retorno de chamada fornecida pelo código que o criou, que manipulará as partes específicas do aplicativo.

```js
const scale = 10;

class PictureCanvas {
  constructor(picture, pointerDown) {
    this.dom = elt("canvas", {
      onmousedown: event => this.mouse(event, pointerDown),
      ontouchstart: event => this.touch(event, pointerDown)
    });
    this.syncState(picture);
  }
  syncState(picture) {
    if (this.picture == picture) return;
    this.picture = picture;
    drawPicture(this.picture, this.dom, scale);
  }
}
```

Desenhamos cada pixel como um quadrado de 10 por 10, conforme determinado pela `scale`constante. Para evitar trabalho desnecessário, o componente monitora sua imagem atual e faz um redesenho somente quando `syncState`recebe uma nova imagem.

A função de desenho real define o tamanho da tela com base na escala e no tamanho da imagem e a preenche com uma série de quadrados, um para cada pixel.

```js
function drawPicture(picture, canvas, scale) {
  canvas.width = picture.width * scale;
  canvas.height = picture.height * scale;
  let cx = canvas.getContext("2d");

  for (let y = 0; y < picture.height; y++) {
    for (let x = 0; x < picture.width; x++) {
      cx.fillStyle = picture.pixel(x, y);
      cx.fillRect(x * scale, y * scale, scale, scale);
    }
  }
}
```

Quando o botão esquerdo do mouse é pressionado enquanto o mouse está sobre a tela da imagem, o componente chama o `pointerDown`retorno de chamada, fornecendo a posição do pixel que foi clicado - nas coordenadas da imagem. Isso será usado para implementar a interação do mouse com a imagem. O retorno de chamada pode retornar outra função de retorno de chamada a ser notificada quando o ponteiro for movido para um pixel diferente enquanto o botão estiver pressionado.

```js
PictureCanvas.prototype.mouse = function(downEvent, onDown) {
  if (downEvent.button != 0) return;
  let pos = pointerPosition(downEvent, this.dom);
  let onMove = onDown(pos);
  if (!onMove) return;
  let move = moveEvent => {
    if (moveEvent.buttons == 0) {
      this.dom.removeEventListener("mousemove", move);
    } else {
      let newPos = pointerPosition(moveEvent, this.dom);
      if (newPos.x == pos.x && newPos.y == pos.y) return;
      pos = newPos;
      onMove(newPos);
    }
  };
  this.dom.addEventListener("mousemove", move);
};

function pointerPosition(pos, domNode) {
  let rect = domNode.getBoundingClientRect();
  return {x: Math.floor((pos.clientX - rect.left) / scale),
          y: Math.floor((pos.clientY - rect.top) / scale)};
}
```

Como sabemos o tamanho dos pixels e podemos `getBoundingClientRect`encontrar a posição da tela na tela, é possível ir das coordenadas do evento do mouse ( `clientX`e `clientY`) às coordenadas da imagem. Eles são sempre arredondados para baixo para que se refiram a um pixel específico.

Com os eventos de toque, temos que fazer algo semelhante, mas usando eventos diferentes e certificando-se de que chamamos `preventDefault`o `"touchstart"`evento para evitar panning.

```js
PictureCanvas.prototype.touch = function(startEvent,
                                         onDown) {
  let pos = pointerPosition(startEvent.touches[0], this.dom);
  let onMove = onDown(pos);
  startEvent.preventDefault();
  if (!onMove) return;
  let move = moveEvent => {
    let newPos = pointerPosition(moveEvent.touches[0],
                                 this.dom);
    if (newPos.x == pos.x && newPos.y == pos.y) return;
    pos = newPos;
    onMove(newPos);
  };
  let end = () => {
    this.dom.removeEventListener("touchmove", move);
    this.dom.removeEventListener("touchend", end);
  };
  this.dom.addEventListener("touchmove", move);
  this.dom.addEventListener("touchend", end);
};
```

Para eventos de toque, `clientX`e `clientY`não estão disponíveis diretamente no objeto de evento, mas podemos usar as coordenadas do primeiro objeto de toque na `touches`propriedade

## A aplicação

Para possibilitar a construção do aplicativo, peça por peça, implementaremos o componente principal como um shell em torno de uma tela de figuras e um conjunto dinâmico de ferramentas e controles que passamos ao seu construtor.

Os *controles* são os elementos da interface que aparecem abaixo da imagem. Eles serão fornecidos como uma matriz de construtores de componentes.

As *ferramentas* fazem coisas como desenhar pixels ou preencher uma área. O aplicativo mostra o conjunto de ferramentas disponíveis como um ``campo. A ferramenta atualmente selecionada determina o que acontece quando o usuário interage com a imagem com um dispositivo apontador. O conjunto de ferramentas disponíveis é fornecido como um objeto que mapeia os nomes que aparecem no campo suspenso para funções que implementam as ferramentas. Tais funções obtêm uma posição de imagem, um estado atual do aplicativo e uma `dispatch`função como argumentos. Eles podem retornar uma função de manipulador de movimento que é chamada com uma nova posição e um estado atual quando o ponteiro se move para um pixel diferente.

```js
class PixelEditor {
  constructor(state, config) {
    let {tools, controls, dispatch} = config;
    this.state = state;

    this.canvas = new PictureCanvas(state.picture, pos => {
      let tool = tools[this.state.tool];
      let onMove = tool(pos, this.state, dispatch);
      if (onMove) return pos => onMove(pos, this.state);
    });
    this.controls = controls.map(
      Control => new Control(state, config));
    this.dom = elt("div", {}, this.canvas.dom, elt("br"),
                   ...this.controls.reduce(
                     (a, c) => a.concat(" ", c.dom), []));
  }
  syncState(state) {
    this.state = state;
    this.canvas.syncState(state.picture);
    for (let ctrl of this.controls) ctrl.syncState(state);
  }
}
```

O manipulador de ponteiro fornecido para `PictureCanvas`chama a ferramenta atualmente selecionada com os argumentos apropriados e, se isso retorna um manipulador de movimentação, adapta-o para também receber o estado.

Todos os controles são construídos e armazenados `this.controls`para que possam ser atualizados quando o estado do aplicativo for alterado. A chamada para `reduce`introduz espaços entre os elementos DOM dos controles. Dessa forma, eles não parecem tão pressionados juntos.

O primeiro controle é o menu de seleção de ferramentas. Ele cria um ``elemento com uma opção para cada ferramenta e configura um `"change"`manipulador de eventos que atualiza o estado do aplicativo quando o usuário seleciona uma ferramenta diferente.

```js
class ToolSelect {
  constructor(state, {tools, dispatch}) {
    this.select = elt("select", {
      onchange: () => dispatch({tool: this.select.value})
    }, ...Object.keys(tools).map(name => elt("option", {
      selected: name == state.tool
    }, name)));
    this.dom = elt("label", null, "🖌 Tool: ", this.select);
  }
  syncState(state) { this.select.value = state.tool; }
}
```

Ao agrupar o texto do rótulo e o campo em um ``elemento, informamos ao navegador que o rótulo pertence a esse campo para que você possa, por exemplo, clicar no rótulo para focar no campo.

Também precisamos alterar a cor, então vamos adicionar um controle para isso. Um ``elemento HTML com um `type`atributo de `color`fornece um campo de formulário especializado para selecionar cores. O valor desse campo é sempre um código de cores CSS no `"#RRGGBB"`formato (componentes vermelho, verde e azul, dois dígitos por cor). O navegador mostrará uma interface do seletor de cores quando o usuário interagir com ele.

Esse controle cria esse campo e o conecta para permanecer sincronizado com a `color`propriedade do estado do aplicativo .

```js
class ColorSelect {
  constructor(state, {dispatch}) {
    this.input = elt("input", {
      type: "color",
      value: state.color,
      onchange: () => dispatch({color: this.input.value})
    });
    this.dom = elt("label", null, "🎨 Color: ", this.input);
  }
  syncState(state) { this.input.value = state.color; }
}
```

## Ferramentas de desenho

Antes que possamos desenhar qualquer coisa, precisamos implementar as ferramentas que controlarão a funcionalidade dos eventos do mouse ou toque na tela.

A ferramenta mais básica é a ferramenta de desenho, que altera qualquer pixel em que você clica ou toca na cor atualmente selecionada. Ele despacha uma ação que atualiza a imagem para uma versão na qual o pixel apontado recebe a cor selecionada no momento.

```js
function draw(pos, state, dispatch) {
  function drawPixel({x, y}, state) {
    let drawn = {x, y, color: state.color};
    dispatch({picture: state.picture.draw([drawn])});
  }
  drawPixel(pos, state);
  return drawPixel;
}
```

A função chama a função imediatamente, `drawPixel`mas também a retorna para que seja chamada novamente para pixels recém-tocados quando o usuário arrastar ou passar o mouse sobre a imagem.

Para desenhar formas maiores, pode ser útil criar retângulos rapidamente. A `rectangle`ferramenta desenha um retângulo entre o ponto em que você começa a arrastar e o ponto em que você arrasta.

```js
function rectangle(start, state, dispatch) {
  function drawRectangle(pos) {
    let xStart = Math.min(start.x, pos.x);
    let yStart = Math.min(start.y, pos.y);
    let xEnd = Math.max(start.x, pos.x);
    let yEnd = Math.max(start.y, pos.y);
    let drawn = [];
    for (let y = yStart; y <= yEnd; y++) {
      for (let x = xStart; x <= xEnd; x++) {
        drawn.push({x, y, color: state.color});
      }
    }
    dispatch({picture: state.picture.draw(drawn)});
  }
  drawRectangle(start);
  return drawRectangle;
}
```

Um detalhe importante nesta implementação é que, ao arrastar, o retângulo é redesenhado na imagem a partir do estado *original* . Dessa forma, você pode aumentar e diminuir o retângulo novamente enquanto o cria, sem que os retângulos intermediários permaneçam na imagem final. Essa é uma das razões pelas quais objetos imutáveis de imagem são úteis - veremos outra razão mais tarde.

A implementação do preenchimento de cheias está um pouco mais envolvida. Essa é uma ferramenta que preenche o pixel sob o ponteiro e todos os pixels adjacentes que têm a mesma cor. "Adjacente" significa diretamente na horizontal ou na vertical adjacente, não na diagonal. Esta imagem ilustra o conjunto de pixels coloridos quando a ferramenta de preenchimento é usada no pixel marcado:

![Uma grade de pixels mostrando a área preenchida por uma operação de preenchimento de inundação](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/flood-grid.svg)

Curiosamente, a maneira como faremos isso se parece um pouco com o código de localização de caminhos do [Capítulo 7](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2007%20-%20Projeto%20Um%20Rob%C3%B4.md) . Enquanto esse código pesquisava um gráfico para encontrar uma rota, esse código pesquisava uma grade para encontrar todos os pixels "conectados". O problema de acompanhar um conjunto ramificado de rotas possíveis é semelhante.

```js
const around = [{dx: -1, dy: 0}, {dx: 1, dy: 0},
                {dx: 0, dy: -1}, {dx: 0, dy: 1}];

function fill({x, y}, state, dispatch) {
  let targetColor = state.picture.pixel(x, y);
  let drawn = [{x, y, color: state.color}];
  for (let done = 0; done < drawn.length; done++) {
    for (let {dx, dy} of around) {
      let x = drawn[done].x + dx, y = drawn[done].y + dy;
      if (x >= 0 && x < state.picture.width &&
          y >= 0 && y < state.picture.height &&
          state.picture.pixel(x, y) == targetColor &&
          !drawn.some(p => p.x == x && p.y == y)) {
        drawn.push({x, y, color: state.color});
      }
    }
  }
  dispatch({picture: state.picture.draw(drawn)});
}
```

A matriz de pixels desenhados também funciona como a lista de trabalho da função. Para cada pixel atingido, precisamos verificar se algum pixel adjacente tem a mesma cor e ainda não foi pintado. O contador de loop fica atrás do comprimento da `drawn`matriz à medida que novos pixels são adicionados. Quaisquer pixels à frente ainda precisam ser explorados. Quando alcança o comprimento, nenhum pixel inexplorado permanece e a função é concluída.

A ferramenta final é um seletor de cores, que permite apontar para uma cor na imagem para usá-la como a cor do desenho atual.

```js
function pick(pos, state, dispatch) {
  dispatch({color: state.picture.pixel(pos.x, pos.y)});
}
```

Agora podemos testar nossa aplicação!

```html
<div></div>
<script>
  let state = {
    tool: "draw",
    color: "#000000",
    picture: Picture.empty(60, 30, "#f0f0f0")
  };
  let app = new PixelEditor(state, {
    tools: {draw, fill, rectangle, pick},
    controls: [ToolSelect, ColorSelect],
    dispatch(action) {
      state = updateState(state, action);
      app.syncState(state);
    }
  });
  document.querySelector("div").appendChild(app.dom);
</script>
```

## Salvando e carregando

Quando desenharmos nossa obra-prima, guardaremos para mais tarde. Devemos adicionar um botão para baixar a imagem atual como um arquivo de imagem. Este controle fornece esse botão:

```js
class SaveButton {
  constructor(state) {
    this.picture = state.picture;
    this.dom = elt("button", {
      onclick: () => this.save()
    }, "💾 Save");
  }
  save() {
    let canvas = elt("canvas");
    drawPicture(this.picture, canvas, 1);
    let link = elt("a", {
      href: canvas.toDataURL(),
      download: "pixelart.png"
    });
    document.body.appendChild(link);
    link.click();
    link.remove();
  }
  syncState(state) { this.picture = state.picture; }
}
```

O componente controla a imagem atual para que possa acessá-la ao salvar. Para criar o arquivo de imagem, ele usa um ``elemento no qual desenha a imagem (na escala de um pixel por pixel).

O `toDataURL`método em um elemento de tela cria uma URL que começa com `data:`. Ao contrário `http:`e`https:` URLs, URLs de dados contêm todo o recurso no URL. Eles geralmente são muito longos, mas permitem criar links de trabalho para fotos arbitrárias, bem aqui no navegador.

Para realmente fazer o navegador baixar a imagem, criamos um elemento de link que aponta para este URL e tem um `download`atributo. Esses links, quando clicados, fazem com que o navegador mostre uma caixa de diálogo para salvar arquivos. Adicionamos esse link ao documento, simulamos um clique nele e o removemos novamente.

Você pode fazer muito com a tecnologia do navegador, mas às vezes a maneira de fazê-lo é bastante estranha.

E piora. Também queremos carregar arquivos de imagem existentes em nosso aplicativo. Para fazer isso, definimos novamente um componente de botão.

```js
class LoadButton {
  constructor(_, {dispatch}) {
    this.dom = elt("button", {
      onclick: () => startLoad(dispatch)
    }, "📁 Load");
  }
  syncState() {}
}

function startLoad(dispatch) {
  let input = elt("input", {
    type: "file",
    onchange: () => finishLoad(input.files[0], dispatch)
  });
  document.body.appendChild(input);
  input.click();
  input.remove();
}
```

Para obter acesso a um arquivo no computador do usuário, precisamos que o usuário selecione o arquivo através de um campo de entrada de arquivo. Como não quero que o botão carregar pareça um campo de entrada de arquivo, criamos a entrada de arquivo quando o botão é clicado e fingimos que a entrada do arquivo foi clicada.

Quando o usuário seleciona um arquivo, podemos usá `FileReader`-lo para obter acesso ao seu conteúdo, novamente como um URL de dados. Esse URL pode ser usado para criar um ``elemento, mas como não podemos obter acesso direto aos pixels em uma imagem, não podemos criar um `Picture`objeto a partir disso.

```js
function finishLoad(file, dispatch) {
  if (file == null) return;
  let reader = new FileReader();
  reader.addEventListener("load", () => {
    let image = elt("img", {
      onload: () => dispatch({
        picture: pictureFromImage(image)
      }),
      src: reader.result
    });
  });
  reader.readAsDataURL(file);
}
```

Para obter acesso aos pixels, devemos primeiro desenhar a imagem em um ``elemento. O contexto da tela possui um `getImageData`método que permite que um script leia seus pixels. Assim, quando a imagem estiver na tela, podemos acessá-la e construir um `Picture`objeto.

```js
function pictureFromImage(image) {
  let width = Math.min(100, image.width);
  let height = Math.min(100, image.height);
  let canvas = elt("canvas", {width, height});
  let cx = canvas.getContext("2d");
  cx.drawImage(image, 0, 0);
  let pixels = [];
  let {data} = cx.getImageData(0, 0, width, height);

  function hex(n) {
    return n.toString(16).padStart(2, "0");
  }
  for (let i = 0; i < data.length; i += 4) {
    let [r, g, b] = data.slice(i, i + 3);
    pixels.push("#" + hex(r) + hex(g) + hex(b));
  }
  return new Picture(width, height, pixels);
}
```

Limitaremos o tamanho das imagens a 100 por 100 pixels, pois qualquer coisa maior parecerá *enorme* em nossa tela e poderá desacelerar a interface.

A `data`propriedade do objeto retornado por `getImageData`é uma matriz de componentes de cores. Para cada pixel no retângulo especificado pelos argumentos, ele contém quatro valores, que representam os componentes vermelho, verde, azul e *alfa* da cor do pixel, como números entre 0 e 255. A parte alfa representa opacidade - quando é zero , o pixel é totalmente transparente e, quando é 255, é totalmente opaco. Para nosso propósito, podemos ignorá-lo.

Os dois dígitos hexadecimais por componente, conforme usados em nossa notação de cores, correspondem precisamente ao intervalo de 0 a 255 - dois dígitos da base 16 podem expressar 16 2 = 256 números diferentes. O `toString`método dos números pode receber uma base como argumento, portanto `n.toString(16)`, produzirá uma representação de string na base 16. Temos que garantir que cada número ocupe dois dígitos, para que a `hex`função auxiliar chame `padStart`para adicionar um zero à esquerda quando necessário.

Podemos carregar e salvar agora! Isso deixa mais um recurso antes de terminarmos.

## Desfazer histórico

Metade do processo de edição está cometendo pequenos erros e corrigindo-os. Portanto, um recurso importante em um programa de desenho é um histórico de desfazer.

Para poder desfazer alterações, precisamos armazenar as versões anteriores da imagem. Como é um valor imutável, isso é fácil. Mas requer um campo adicional no estado do aplicativo.

Adicionaremos uma `done`matriz para manter as versões anteriores da imagem. A manutenção dessa propriedade requer uma função de atualização de estado mais complicada que adiciona imagens à matriz.

Mas não queremos armazenar *todas as* alterações, apenas altera uma certa quantidade de tempo. Para fazer isso, precisamos de uma segunda propriedade `doneAt`, que rastreie o horário em que armazenamos uma imagem pela última vez no histórico.

```js
function historyUpdateState(state, action) {
  if (action.undo == true) {
    if (state.done.length == 0) return state;
    return Object.assign({}, state, {
      picture: state.done[0],
      done: state.done.slice(1),
      doneAt: 0
    });
  } else if (action.picture &&
             state.doneAt < Date.now() - 1000) {
    return Object.assign({}, state, action, {
      done: [state.picture, ...state.done],
      doneAt: Date.now()
    });
  } else {
    return Object.assign({}, state, action);
  }
}
```

Quando a ação é uma ação desfazer, a função tira a foto mais recente do histórico e a torna atual. Ele é definido `doneAt`como zero para que a próxima alteração seja garantida para armazenar a imagem de volta no histórico, permitindo que você volte a ela outra vez, se desejar.

Caso contrário, se a ação contiver uma nova imagem e a última vez que armazenamos algo há mais de um segundo (1000 milissegundos) atrás, as propriedades `done`e `doneAt`são atualizadas para armazenar a imagem anterior.

O componente do botão desfazer não faz muito. Ele despacha as ações de desfazer quando clicadas e se desabilita quando não há nada a desfazer.

```js
class UndoButton {
  constructor(state, {dispatch}) {
    this.dom = elt("button", {
      onclick: () => dispatch({undo: true}),
      disabled: state.done.length == 0
    }, "⮪ Undo");
  }
  syncState(state) {
    this.dom.disabled = state.done.length == 0;
  }
}
```

## Vamos desenhar

Para configurar o aplicativo, precisamos criar um estado, um conjunto de ferramentas, um conjunto de controles e uma função de despacho. Podemos passá-los ao `PixelEditor`construtor para criar o componente principal. Como precisamos criar vários editores nos exercícios, primeiro definimos algumas ligações.

```js
const startState = {
  tool: "draw",
  color: "#000000",
  picture: Picture.empty(60, 30, "#f0f0f0"),
  done: [],
  doneAt: 0
};

const baseTools = {draw, fill, rectangle, pick};

const baseControls = [
  ToolSelect, ColorSelect, SaveButton, LoadButton, UndoButton
];

function startPixelEditor({state = startState,
                           tools = baseTools,
                           controls = baseControls}) {
  let app = new PixelEditor(state, {
    tools,
    controls,
    dispatch(action) {
      state = historyUpdateState(state, action);
      app.syncState(state);
    }
  });
  return app.dom;
}
```

Ao destruir um objeto ou matriz, você pode usar `=`após um nome de ligação para atribuir à associação um valor padrão, que é usado quando a propriedade está ausente ou em espera `undefined`. A `startPixelEditor`função utiliza isso para aceitar um objeto com várias propriedades opcionais como argumento. Se você não fornecer uma `tools`propriedade, por exemplo, `tools`será vinculado a `baseTools`.

É assim que temos um editor real na tela:

```html
<div></div>
<script>
  document.querySelector("div")
    .appendChild(startPixelEditor({}));
</script>
```

Vá em frente e desenhe algo. Eu vou esperar.

## Por que isso é tão difícil?

A tecnologia do navegador é incrível. Ele fornece um conjunto poderoso de componentes básicos de interface, maneiras de estilizá-los e manipulá-los e ferramentas para inspecionar e depurar seus aplicativos. O software que você escreve para o navegador pode ser executado em quase todos os computadores e telefones do planeta.

Ao mesmo tempo, a tecnologia do navegador é ridícula. Você precisa aprender um grande número de truques tolos e fatos obscuros para dominá-lo, e o modelo de programação padrão que ele fornece é tão problemático que a maioria dos programadores prefere cobri-lo em várias camadas de abstração, em vez de lidar diretamente com ele.

E, embora a situação esteja definitivamente melhorando, ela o faz principalmente na forma de mais elementos sendo adicionados para solucionar as deficiências - criando ainda mais complexidade. Um recurso usado por um milhão de sites não pode realmente ser substituído. Mesmo que pudesse, seria difícil decidir com o que deveria ser substituído.

A tecnologia nunca existe no vácuo - somos constrangidos por nossas ferramentas e pelos fatores sociais, econômicos e históricos que as produziram. Isso pode ser irritante, mas geralmente é mais produtivo tentar entender bem como a realidade técnica *existente* funciona - e por que é assim - do que se enfurecer contra ela ou defender outra realidade.

Novas abstrações *podem* ser úteis. O modelo de componente e a convenção de fluxo de dados que usei neste capítulo são uma forma bruta disso. Como mencionado, existem bibliotecas que tentam tornar a programação da interface do usuário mais agradável. No momento da redação deste artigo, [React](https://reactjs.org/) e [Angular](https://angular.io/) são escolhas populares, mas há toda uma indústria caseira dessas estruturas. Se você estiver interessado em programar aplicativos da Web, recomendo investigar alguns deles para entender como eles funcionam e quais benefícios eles oferecem.

## Exercícios

Ainda há espaço para melhorias em nosso programa. Vamos adicionar mais alguns recursos como exercícios.

### Ligações do teclado

Adicione atalhos de teclado ao aplicativo. A primeira letra do nome de uma ferramenta seleciona a ferramenta e o controle -Z ou o comando -Z é ativado para desfazer.

Faça isso modificando o `PixelEditor`componente. Adicione uma `tabIndex`propriedade 0 ao ``elemento de agrupamento para que ele possa receber o foco do teclado. Observe que a *propriedade* correspondente ao `tabindex` *atributo* é chamada `tabIndex`, com I maiúsculo, e nossa `elt`função espera nomes de propriedades. Registre os principais manipuladores de eventos diretamente nesse elemento. Isso significa que você precisa clicar, tocar ou tabular para o aplicativo antes de poder interagir com ele com o teclado.

Lembre-se de que os eventos do teclado têm `ctrlKey`e `metaKey`(para a tecla de comando no Mac) propriedades que você pode usar para verificar se essas teclas estão pressionadas.

```html
<div></div>
<script>
  // The original PixelEditor class. Extend the constructor.
  class PixelEditor {
    constructor(state, config) {
      let {tools, controls, dispatch} = config;
      this.state = state;

      this.canvas = new PictureCanvas(state.picture, pos => {
        let tool = tools[this.state.tool];
        let onMove = tool(pos, this.state, dispatch);
        if (onMove) {
          return pos => onMove(pos, this.state, dispatch);
        }
      });
      this.controls = controls.map(
        Control => new Control(state, config));
      this.dom = elt("div", {}, this.canvas.dom, elt("br"),
                     ...this.controls.reduce(
                       (a, c) => a.concat(" ", c.dom), []));
    }
    syncState(state) {
      this.state = state;
      this.canvas.syncState(state.picture);
      for (let ctrl of this.controls) ctrl.syncState(state);
    }
  }

  document.querySelector("div")
    .appendChild(startPixelEditor({}));
</script>
```

A `key`propriedade de eventos para chaves de letras será a própria letra minúscula, se o turno não estiver sendo mantido. Não estamos interessados em eventos importantes com mudança aqui.

Um `"keydown"`manipulador pode inspecionar seu objeto de evento para ver se ele corresponde a algum dos atalhos. Você pode obter automaticamente a lista de primeiras letras do `tools`objeto para não precisar escrevê-las.

Quando o evento-chave corresponder a um atalho, chame `preventDefault`-o e despache a ação apropriada.

### Desenho eficiente

Durante o desenho, a maior parte do trabalho que nosso aplicativo realiza `drawPicture`. Criar um novo estado e atualizar o restante do DOM não é muito caro, mas repintar todos os pixels na tela é um pouco de trabalho.

Encontre uma maneira de tornar o `syncState`método `PictureCanvas`mais rápido redesenhando apenas os pixels que realmente mudaram.

Lembre-se de que `drawPicture`também é usado pelo botão Salvar, portanto, se você o alterar, verifique se as alterações não quebram o uso antigo ou crie uma nova versão com um nome diferente.

Observe também que alterar o tamanho de um ``elemento, definindo suas propriedades `width`ou `height`, o limpa, tornando-o totalmente transparente novamente.

```html
<div></div>
<script>
  // Change this method
  PictureCanvas.prototype.syncState = function(picture) {
    if (this.picture == picture) return;
    this.picture = picture;
    drawPicture(this.picture, this.dom, scale);
  };

  // You may want to use or change this as well
  function drawPicture(picture, canvas, scale) {
    canvas.width = picture.width * scale;
    canvas.height = picture.height * scale;
    let cx = canvas.getContext("2d");

    for (let y = 0; y < picture.height; y++) {
      for (let x = 0; x < picture.width; x++) {
        cx.fillStyle = picture.pixel(x, y);
        cx.fillRect(x * scale, y * scale, scale, scale);
      }
    }
  }

  document.querySelector("div")
    .appendChild(startPixelEditor({}));
</script>
```

Este exercício é um bom exemplo de como estruturas de dados imutáveis podem tornar o código *mais rápido* . Como temos a foto antiga e a nova, podemos compará-las e redesenhar apenas os pixels que mudaram de cor, economizando mais de 99% do desenho na maioria dos casos.

Você pode escrever uma nova função `updatePicture`ou ter `drawPicture`um argumento extra, que pode ser indefinido ou na imagem anterior. Para cada pixel, a função verifica se uma imagem anterior foi passada com a mesma cor nessa posição e ignora o pixel quando for o caso.

Como a tela é limpa quando alteramos seu tamanho, você também deve evitar tocar nas propriedades `width`e `height`quando a imagem antiga e a nova imagem tiverem o mesmo tamanho. Se eles forem diferentes, o que acontecerá quando uma nova imagem for carregada, você poderá definir a encadernação que mantém a imagem antiga como nula após alterar o tamanho da tela, pois não deverá pular nenhum pixel depois de alterar o tamanho da tela.

### Círculos

Defina uma ferramenta chamada `circle`que desenha um círculo preenchido quando você arrasta. O centro do círculo fica no ponto em que o gesto de arrastar ou tocar inicia e seu raio é determinado pela distância arrastada.

```html
<div></div>
<script>
  function circle(pos, state, dispatch) {
    // Your code here
  }

  let dom = startPixelEditor({
    tools: Object.assign({}, baseTools, {circle})
  });
  document.querySelector("div").appendChild(dom);
</script>
```

Você pode se inspirar na `rectangle`ferramenta. Assim, você continuará desenhando a imagem *inicial* , e não a atual, quando o ponteiro se mover.

Para descobrir quais pixels colorir, você pode usar o teorema de Pitágoras. Primeiro, calcule a distância entre a posição atual do ponteiro e a posição inicial, tomando a raiz quadrada ( `Math.sqrt`) da soma do quadrado ( `Math.pow(x, 2)`) da diferença nas coordenadas x e o quadrado da diferença nas coordenadas y. Em seguida, faça um loop sobre um quadrado de pixels em torno da posição inicial, cujos lados são pelo menos duas vezes o raio, e pinte os que estão dentro do raio do círculo, novamente usando a fórmula pitagórica para descobrir a distância do centro.

Certifique-se de não tentar colorir pixels que estão fora dos limites da imagem.

### Linhas apropriadas

Este é um exercício mais avançado que os dois anteriores e exigirá que você projete uma solução para um problema não trivial. Certifique-se de ter bastante tempo e paciência antes de começar a trabalhar neste exercício e não desanime com as falhas iniciais.

Na maioria dos navegadores, quando você seleciona a `draw`ferramenta e arrasta rapidamente a imagem, não recebe uma linha fechada. Em vez disso, você obtém pontos com espaços entre eles porque os eventos `"mousemove"`ou `"touchmove"`não dispararam com rapidez suficiente para atingir cada pixel.

Melhore a `draw`ferramenta para fazer uma linha completa. Isso significa que você deve fazer com que a função do manipulador de movimento se lembre da posição anterior e conecte-a à atual.

Para fazer isso, como os pixels podem estar a uma distância arbitrária, você precisará escrever uma função geral de desenho de linha.

Uma linha entre dois pixels é uma cadeia de pixels conectada, o mais reta possível, indo do início ao fim. Pixels adjacentes na diagonal contam como conectados. Portanto, uma linha inclinada deve se parecer com a imagem à esquerda, não com a imagem à direita.

![Duas linhas pixeladas, uma luz, pulando os pixels na diagonal e outra pesada, com todos os pixels conectados horizontal ou verticalmente](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/line-grid.svg)

Finalmente, se temos um código que desenha uma linha entre dois pontos arbitrários, também podemos usá-lo para definir também uma `line`ferramenta, que desenha uma linha reta entre o início e o final de um arrasto.

```html
<div></div>
<script>
  // The old draw tool. Rewrite this.
  function draw(pos, state, dispatch) {
    function drawPixel({x, y}, state) {
      let drawn = {x, y, color: state.color};
      dispatch({picture: state.picture.draw([drawn])});
    }
    drawPixel(pos, state);
    return drawPixel;
  }

  function line(pos, state, dispatch) {
    // Your code here
  }

  let dom = startPixelEditor({
    tools: {draw, line, fill, rectangle, pick}
  });
  document.querySelector("div").appendChild(dom);
</script>
```

O problema do desenho de uma linha pixelizada é que na verdade são quatro problemas semelhantes, mas um pouco diferentes. É fácil desenhar uma linha horizontal da esquerda para a direita - você passa pelas coordenadas x e pinta um pixel a cada passo. Se a linha tiver uma inclinação leve (menos de 45 graus ou ¼π radianos), você poderá interpolar a coordenada y ao longo da inclinação. Você ainda precisa de um pixel por posição *x* , com *y* posição desses pixels determinada pela inclinação.

Mas assim que sua inclinação ultrapassar 45 graus, é necessário mudar a maneira como trata as coordenadas. Agora você precisa de um pixel por *ano* posição , pois a linha sobe mais do que a esquerda. E então, quando você cruza 135 graus, é necessário voltar ao loop pelas coordenadas x, mas da direita para a esquerda.

Na verdade, você não precisa escrever quatro loops. Como desenhar uma linha de *A* a *B* é o mesmo que desenhar uma linha de *B* a *A* , você pode trocar as posições inicial e final por linhas da direita para a esquerda e tratá-las como da esquerda para a direita.

Então você precisa de dois loops diferentes. A primeira coisa que sua função de desenho de linha deve fazer é verificar se a diferença entre as coordenadas x é maior que a diferença entre as coordenadas y. Se for, essa é uma linha horizontal horizontal e, se não for, vertical.

Certifique-se de comparar os *absolutos* valores dos *x* e *y* diferença, que você pode começar com `Math.abs`.

Depois de saber ao longo de qual eixo você fará o loop, é possível verificar se o ponto inicial possui uma coordenada mais alta ao longo do eixo que o ponto final e trocá-los, se necessário. Uma maneira sucinta de trocar os valores de duas ligações no JavaScript usa a atribuição de desestruturação como esta:

```js
[start, end] = [end, start];
```

Em seguida, você pode calcular a inclinação da linha, que determina a quantidade que a coordenada no outro eixo muda para cada etapa que você dá ao longo do eixo principal. Com isso, você pode executar um loop ao longo do eixo principal enquanto rastreia a posição correspondente no outro eixo e pode desenhar pixels em todas as iterações. Certifique-se de arredondar as coordenadas do eixo não principal, pois elas provavelmente são fracionárias e o `draw`método não responde bem às coordenadas fracionárias.