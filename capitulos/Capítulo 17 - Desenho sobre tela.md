# Capítulo 17 Desenho sobre tela

> Desenhar é engano.
>
> *—MC Escher, citado por Bruno Ernst em The Magic Mirror of MC Escher*

![Imagem de um braço de robô, desenho em papel](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/chapter_picture_17.jpg)

Os navegadores nos dão várias maneiras de exibir gráficos. A maneira mais simples é usar estilos para posicionar e colorir elementos DOM regulares. Isso pode levar você muito longe, como mostrou o jogo no [capítulo anterior](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2016%20-%20%20Projeto%20Um%20Jogo%20de%20Plataforma.md) . Ao adicionar imagens de plano de fundo parcialmente transparentes aos nós, podemos fazer com que pareçam exatamente da maneira que queremos. É ainda possível girar ou inclinar nós com o `transform`estilo.

Mas estaríamos usando o DOM para algo para o qual não foi originalmente projetado. Algumas tarefas, como desenhar uma linha entre pontos arbitrários, são extremamente difíceis de fazer com elementos HTML regulares.

Existem duas alternativas. O primeiro é baseado em DOM, mas utiliza *Scalable Vector Graphics* (SVG), em vez de HTML. Pense no SVG como um dialeto de marcação de documento que se concentra nas formas e não no texto. Você pode incorporar um documento SVG diretamente em um documento HTML ou incluí-lo com uma ``tag.

A segunda alternativa é chamada de *tela* . Uma tela é um único elemento DOM que encapsula uma imagem. Ele fornece uma interface de programação para desenhar formas no espaço ocupado pelo nó. A principal diferença entre uma tela e uma imagem SVG é que, no SVG, a descrição original das formas é preservada para que possam ser movidas ou redimensionadas a qualquer momento. Uma tela, por outro lado, converte as formas em pixels (pontos coloridos em uma varredura) assim que são desenhadas e não lembra o que esses pixels representam. A única maneira de mover uma forma em uma tela é limpar a tela (ou a parte da tela em torno da forma) e redesenhá-la com a forma em uma nova posição.

## SVG

Este livro não entrará em detalhes no SVG, mas explicarei brevemente como ele funciona. No [https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2017%20-%20Desenho%20sobre%20tela.md) , voltarei às compensações que você deve considerar ao decidir qual mecanismo de desenho é apropriado para uma determinada aplicação.

Este é um documento HTML com uma imagem SVG simples:

```html
<p>Normal HTML here.</p>
<svg xmlns="http://www.w3.org/2000/svg">
  <circle r="50" cx="50" cy="50" fill="red"/>
  <rect x="120" y="5" width="90" height="90"
        stroke="blue" fill="none"/>
</svg>
```

O `xmlns`atributo altera um elemento (e seus filhos) para um *espaço para nome XML* diferente . Este espaço para nome, identificado por uma URL, especifica o dialeto que estamos falando no momento. As tags ``e ``, que não existem no HTML, têm um significado no SVG - elas desenham formas usando o estilo e a posição especificados por seus atributos.

Essas tags criam elementos DOM, assim como as tags HTML, com as quais os scripts podem interagir. Por exemplo, isso altera o ``elemento para ser colorido em ciano:

```js
let circle = document.querySelector("circle");
circle.setAttribute("fill", "cyan");
```

## O elemento canvas

Os gráficos de tela podem ser desenhados em um ``elemento. Você pode fornecer esse elemento `width`e `height`atributos para determinar seu tamanho em pixels.

Uma nova tela está vazia, o que significa que é totalmente transparente e, portanto, aparece como espaço vazio no documento.

A <canvas>tag destina-se a permitir diferentes estilos de desenho. Para obter acesso a uma interface de desenho real, primeiro precisamos criar um contexto , um objeto cujos métodos fornecem a interface de desenho. Atualmente, existem dois estilos de desenho amplamente suportados: "2d"para gráficos bidimensionais e "webgl"gráficos tridimensionais através da interface OpenGL.

Este livro não abordará o WebGL - seguiremos duas dimensões. Mas se você estiver interessado em gráficos tridimensionais, encorajo-o a procurar no WebGL. Ele fornece uma interface direta com o hardware gráfico e permite renderizar cenas complicadas com eficiência, usando JavaScript.

Você cria um contexto com o `getContext`método no ``elemento DOM.

```html
<p>Before canvas.</p>
<canvas width="120" height="60"></canvas>
<p>After canvas.</p>
<script>
  let canvas = document.querySelector("canvas");
  let context = canvas.getContext("2d");
  context.fillStyle = "red";
  context.fillRect(10, 10, 100, 50);
</script>
```

Após criar o objeto de contexto, o exemplo desenha um retângulo vermelho com 100 pixels de largura e 50 pixels de altura, com o canto superior esquerdo nas coordenadas (10,10).

Assim como no HTML (e SVG), o sistema de coordenadas usado pela tela coloca (0,0) no canto superior esquerdo e o eixo y positivo desce a partir daí. Então (10,10) fica 10 pixels abaixo e à direita do canto superior esquerdo.

## Linhas e superfícies

Na interface de telas, uma forma pode ser *preenchido* , o que significa que a sua área é dada uma determinada cor ou padrão, ou pode ser *afagado* , o que significa uma linha é desenhada ao longo de sua borda. A mesma terminologia é usada pelo SVG.

O `fillRect`método preenche um retângulo. Ele pega primeiro as coordenadas x e y do canto superior esquerdo do retângulo, depois a largura e a altura. Um método semelhante `strokeRect`, desenha o contorno de um retângulo.

Nenhum dos métodos aceita outros parâmetros. A cor do preenchimento, a espessura do traçado e assim por diante não são determinadas por um argumento para o método (como seria de esperar), mas pelas propriedades do objeto de contexto.

A `fillStyle`propriedade controla a maneira como as formas são preenchidas. Pode ser definido como uma sequência que especifica uma cor, usando a notação de cores usada pelo CSS.

A `strokeStyle`propriedade funciona da mesma forma, mas determina a cor usada para uma linha tracejada. A largura dessa linha é determinada pela `lineWidth`propriedade, que pode conter qualquer número positivo.

```html
<canvas></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");
  cx.strokeStyle = "blue";
  cx.strokeRect(5, 5, 50, 50);
  cx.lineWidth = 5;
  cx.strokeRect(135, 5, 50, 50);
</script>
```

Quando nenhum atributo `width`ou `height`é especificado, como no exemplo, um elemento da tela obtém uma largura padrão de 300 pixels e uma altura de 150 pixels.

## Caminhos

Um caminho é uma sequência de linhas. A interface de tela 2D adota uma abordagem peculiar para descrever esse caminho. É feito inteiramente através de efeitos colaterais. Caminhos não são valores que podem ser armazenados e transmitidos. Em vez disso, se você quiser fazer algo com um caminho, faça uma sequência de chamadas de método para descrever sua forma.

```html
<canvas></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");
  cx.beginPath();
  for (let y = 10; y < 100; y += 10) {
    cx.moveTo(10, y);
    cx.lineTo(90, y);
  }
  cx.stroke();
</script>
```

Este exemplo cria um caminho com vários segmentos de linhas horizontais e depois o traça usando o `stroke`método Cada segmento criado com `lineTo`inicia na posição *atual* do caminho . Essa posição geralmente é o fim do último segmento, a menos que tenha `moveTo`sido chamado. Nesse caso, o próximo segmento começaria na posição passada para `moveTo`.

Ao preencher um caminho (usando o `fill`método), cada forma é preenchida separadamente. Um caminho pode conter várias formas - cada `moveTo`movimento inicia um novo. Mas o caminho precisa ser *fechado* (o que significa que seu início e fim estão na mesma posição) antes de poder ser preenchido. Se o caminho ainda não estiver fechado, uma linha será adicionada do final ao início e a forma delimitada pelo caminho completo será preenchida.

```html
<canvas></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");
  cx.beginPath();
  cx.moveTo(50, 10);
  cx.lineTo(10, 70);
  cx.lineTo(90, 70);
  cx.fill();
</script>
```

Este exemplo desenha um triângulo preenchido. Observe que apenas dois lados do triângulo são explicitamente desenhados. O terceiro, do canto inferior direito até o topo, está implícito e não estaria lá quando você traçar o caminho.

Você também pode usar o `closePath`método para fechar explicitamente um caminho adicionando um segmento de linha real de volta ao início do caminho. Este segmento *é* desenhado ao traçar o caminho.

## Curvas

Um caminho também pode conter linhas curvas. Infelizmente, estes são um pouco mais complicados de desenhar.

O `quadraticCurveTo`método desenha uma curva para um determinado ponto. Para determinar a curvatura da linha, o método recebe um ponto de controle e um ponto de destino. Imagine esse ponto de controle como *atraindo* a linha, dando-lhe sua curva. A linha não passará pelo ponto de controle, mas sua direção nos pontos inicial e final será tal que uma linha reta nessa direção apontaria para o ponto de controle. O exemplo a seguir ilustra isso:

```html
<canvas></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");
  cx.beginPath();
  cx.moveTo(10, 90);
  // control=(60,10) goal=(90,90)
  cx.quadraticCurveTo(60, 10, 90, 90);
  cx.lineTo(60, 10);
  cx.closePath();
  cx.stroke();
</script>
```

Desenhamos uma curva quadrática da esquerda para a direita, com (60,10) como ponto de controle e, em seguida, desenhamos dois segmentos de linha passando por esse ponto de controle e retornando ao início da linha. O resultado lembra um pouco as insígnias de *Star Trek* . Você pode ver o efeito do ponto de controle: as linhas que saem dos cantos inferiores começam na direção do ponto de controle e depois curvam-se em direção ao alvo.

O `bezierCurveTo`método desenha um tipo semelhante de curva. Em vez de um único ponto de controle, este possui dois - um para cada um dos pontos de extremidade da linha. Aqui está um esboço semelhante para ilustrar o comportamento dessa curva:

```html
<canvas></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");
  cx.beginPath();
  cx.moveTo(10, 90);
  // control1=(10,10) control2=(90,10) goal=(50,90)
  cx.bezierCurveTo(10, 10, 90, 10, 50, 90);
  cx.lineTo(90, 10);
  cx.lineTo(10, 10);
  cx.closePath();
  cx.stroke();
</script>
```

Os dois pontos de controle especificam a direção nas duas extremidades da curva. Quanto mais distanciados estiverem do ponto correspondente, mais a curva se elevará nessa direção.

É difícil trabalhar com essas curvas - nem sempre é claro como encontrar os pontos de controle que fornecem a forma que você está procurando. Às vezes, você pode calculá-las e, às vezes, apenas precisa encontrar um valor adequado por tentativa e erro.

O `arc`método é uma maneira de desenhar uma linha que se curva ao longo da borda de um círculo. É necessário um par de coordenadas para o centro do arco, um raio e, em seguida, um ângulo inicial e final.

Esses dois últimos parâmetros tornam possível desenhar apenas parte do círculo. Os ângulos são medidos em radianos, não em graus. Isso significa que um círculo completo tem um ângulo de 2π, ou `2 * Math.PI`, que é cerca de 6,28. O ângulo começa a contar no ponto à direita do centro do círculo e segue no sentido horário a partir daí. Você pode usar um início de 0 e um final maior que 2π (por exemplo, 7) para desenhar um círculo completo.

```html
<canvas></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");
  cx.beginPath();
  // center=(50,50) radius=40 angle=0 to 7
  cx.arc(50, 50, 40, 0, 7);
  // center=(150,50) radius=40 angle=0 to ½π
  cx.arc(150, 50, 40, 0, 0.5 * Math.PI);
  cx.stroke();
</script>
```

A imagem resultante contém uma linha da direita do círculo completo (primeira chamada para `arc`) à direita do quarto de círculo (segunda chamada). Como outros métodos de desenho de caminho, uma linha desenhada `arc`é conectada ao segmento de caminho anterior. Você pode chamar `moveTo`ou iniciar um novo caminho para evitar isso.

## Desenhando um gráfico de pizza

Imagine que você acabou de conseguir um emprego na EconomiCorp, Inc., e sua primeira tarefa é desenhar um gráfico dos resultados da pesquisa de satisfação do cliente.

A `results`ligação contém uma matriz de objetos que representam as respostas da pesquisa.

```js
const results = [
  {name: "Satisfied", count: 1043, color: "lightblue"},
  {name: "Neutral", count: 563, color: "lightgreen"},
  {name: "Unsatisfied", count: 510, color: "pink"},
  {name: "No comment", count: 175, color: "silver"}
];
```

Para desenhar um gráfico de pizza, desenhamos várias fatias de pizza, cada uma composta de um arco e um par de linhas no centro desse arco. Podemos calcular o ângulo assumido por cada arco dividindo um círculo completo (2π) pelo número total de respostas e multiplicando esse número (o ângulo por resposta) pelo número de pessoas que escolheram uma determinada opção.

```html
<canvas width="200" height="200"></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");
  let total = results
    .reduce((sum, {count}) => sum + count, 0);
  // Start at the top
  let currentAngle = -0.5 * Math.PI;
  for (let result of results) {
    let sliceAngle = (result.count / total) * 2 * Math.PI;
    cx.beginPath();
    // center=100,100, radius=100
    // from current angle, clockwise by slice's angle
    cx.arc(100, 100, 100,
           currentAngle, currentAngle + sliceAngle);
    currentAngle += sliceAngle;
    cx.lineTo(100, 100);
    cx.fillStyle = result.color;
    cx.fill();
  }
</script>
```

Mas um gráfico que não nos diz o que as fatias significam não é muito útil. Precisamos de uma maneira de desenhar texto na tela.

## Texto

Um contexto de desenho de tela 2D fornece os métodos `fillText`e `strokeText`. O último pode ser útil para delinear letras, mas geralmente `fillText`é o que você precisa. Ele preencherá o contorno do texto fornecido com a corrente `fillStyle`.

```html
<canvas></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");
  cx.font = "28px Georgia";
  cx.fillStyle = "fuchsia";
  cx.fillText("I can draw text, too!", 10, 50);
</script>
```

Você pode especificar o tamanho, estilo e fonte do texto com a `font`propriedade Este exemplo fornece apenas o tamanho da fonte e o nome da família. Também é possível adicionar `italic`ou `bold`ao início da string para selecionar um estilo.

Os dois últimos argumentos `fillText`e `strokeText`fornecem a posição em que a fonte é desenhada. Por padrão, eles indicam a posição do início da linha de base alfabética do texto, que é a linha na qual as letras “permanecem”, sem contar as partes suspensas em letras como *j* ou *p* . Você pode alterar a posição horizontal, definindo a `textAlign`propriedade para `"end"`ou `"center"`e a posição vertical, ajustando `textBaseline`para `"top"`, `"middle"`ou `"bottom"`.

Voltaremos ao gráfico de pizza e ao problema de rotular as fatias nos [exercícios](https://eloquentjavascript.net/17_canvas.html#exercise_pie_chart) no final do capítulo.

## Imagens

Em computação gráfica, é frequentemente feita uma distinção entre gráficos *vetoriais* e gráficos de *bitmap* . A primeira é o que temos feito até agora neste capítulo - especificando uma imagem, fornecendo uma descrição lógica das formas. Os gráficos de bitmap, por outro lado, não especificam formas reais, mas funcionam com dados de pixel (rasters de pontos coloridos).

O `drawImage`método permite desenhar dados de pixel em uma tela. Esses dados de pixel podem se originar de um ``elemento ou de outra tela. O exemplo a seguir cria um ``elemento desanexado e carrega um arquivo de imagem nele. Mas ele não pode começar imediatamente a desenhar a partir desta imagem porque o navegador ainda não o carregou. Para lidar com isso, registramos um `"load"`manipulador de eventos e fazemos o desenho após o carregamento da imagem.

```html
<canvas></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");
  let img = document.createElement("img");
  img.src = "img/hat.png";
  img.addEventListener("load", () => {
    for (let x = 10; x < 200; x += 30) {
      cx.drawImage(img, x, 10);
    }
  });
</script>
```

Por padrão, `drawImage`desenha a imagem no tamanho original. Você também pode fornecer dois argumentos adicionais para definir uma largura e altura diferentes.

Quando `drawImage`são dados *nove* argumentos, ele pode ser usado para desenhar apenas um fragmento de uma imagem. O segundo ao quinto argumento indica o retângulo (x, y, largura e altura) na imagem de origem que deve ser copiada, e o sexto ao nono argumentos fornecem o retângulo (na tela) no qual ele deve ser copiado.

Isso pode ser usado para compactar vários *sprites* (elementos de imagem) em um único arquivo de imagem e desenhar apenas a parte necessária. Por exemplo, temos esta imagem contendo um personagem do jogo em várias poses:

![Várias poses de um personagem do jogo](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/player_big.png)

Alternando qual pose desenhamos, podemos mostrar uma animação que se parece com um personagem ambulante.

Para animar uma imagem em uma tela, o `clearRect`método é útil. Assemelha-se `fillRect`, mas, em vez de colorir o retângulo, torna-o transparente, removendo os pixels desenhados anteriormente.

Sabemos que cada *sprite* , cada sub-imagem, tem 24 pixels de largura e 30 pixels de altura. O código a seguir carrega a imagem e configura um intervalo (timer repetido) para desenhar o próximo quadro:

```html
<canvas></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");
  let img = document.createElement("img");
  img.src = "img/player.png";
  let spriteW = 24, spriteH = 30;
  img.addEventListener("load", () => {
    let cycle = 0;
    setInterval(() => {
      cx.clearRect(0, 0, spriteW, spriteH);
      cx.drawImage(img,
                   // source rectangle
                   cycle * spriteW, 0, spriteW, spriteH,
                   // destination rectangle
                   0,               0, spriteW, spriteH);
      cycle = (cycle + 1) % 8;
    }, 120);
  });
</script>
```

A `cycle`encadernação rastreia nossa posição na animação. Para cada quadro, ele é incrementado e depois recortado no intervalo de 0 a 7 usando o operador restante. Essa ligação é usada para calcular a coordenada x que o sprite para a pose atual possui na imagem.

## Transformação

Mas e se quisermos que nosso personagem caminhe para a esquerda em vez de para a direita? Poderíamos desenhar outro conjunto de sprites, é claro. Mas também podemos instruir a tela a desenhar o quadro ao contrário.

Chamar o `scale`método fará com que qualquer coisa desenhada depois seja dimensionada. Este método utiliza dois parâmetros, um para definir uma escala horizontal e outro para definir uma escala vertical.

```html
<canvas></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");
  cx.scale(3, .5);
  cx.beginPath();
  cx.arc(50, 50, 40, 0, 7);
  cx.lineWidth = 3;
  cx.stroke();
</script>
```

A escala fará com que tudo sobre a imagem desenhada, incluindo a largura da linha, seja esticada ou comprimida, conforme especificado. Escalar uma quantidade negativa irá inverter a imagem. A inversão ocorre em torno do ponto (0,0), o que significa que também irá inverter a direção do sistema de coordenadas. Quando uma escala horizontal de -1 é aplicada, uma forma desenhada na posição x 100 terminará no que costumava ser a posição -100.

Portanto, para inverter uma imagem, não podemos simplesmente adicionar `cx.scale(-1, 1)`antes da chamada, `drawImage`porque isso moveria nossa imagem para fora da tela, onde não será visível. Você pode ajustar as coordenadas dadas para `drawImage`compensar isso desenhando a imagem na posição x -50 em vez de 0. Outra solução, que não exige que o código que faz o desenho saiba sobre a mudança de escala, é ajustar o eixo em torno do qual a escala acontece.

Além disso, existem vários outros métodos `scale`que influenciam o sistema de coordenadas de uma tela. Você pode girar formas desenhadas posteriormente com o `rotate`método e movê-las com o `translate`método. O interessante - e confuso - é que essas transformações se *acumulam* , o que significa que cada uma acontece em relação às transformações anteriores.

Portanto, se traduzirmos por 10 pixels horizontais duas vezes, tudo será desenhado 20 pixels para a direita. Se primeiro movermos o centro do sistema de coordenadas para (50,50) e depois girarmos 20 graus (cerca de 0,1π radianos), essa rotação ocorrerá em *torno do* ponto (50,50).

![Transformações de empilhamento](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/transform.svg)

Mas se girarmos *primeiro* 20 graus e *depois* traduzirmos (50,50), a tradução ocorrerá no sistema de coordenadas giradas e, assim, produzirá uma orientação diferente. A ordem na qual as transformações são aplicadas é importante.

Para inverter uma imagem ao redor da linha vertical em uma determinada posição x, podemos fazer o seguinte:

```js
function flipHorizontally(context, around) {
  context.translate(around, 0);
  context.scale(-1, 1);
  context.translate(-around, 0);
}
```

Movemos o eixo y para onde queremos que nosso espelho esteja, aplicamos o espelhamento e, finalmente, movemos o eixo y de volta ao seu devido lugar no universo espelhado. A figura a seguir explica por que isso funciona:

![Espelhar em torno de uma linha vertical](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/mirror.svg)

Isso mostra os sistemas de coordenadas antes e depois do espelhamento na linha central. Os triângulos são numerados para ilustrar cada etapa. Se desenharmos um triângulo na posição x positiva, ele estará, por padrão, no local onde está o triângulo 1. Uma chamada para `flipHorizontally`primeiro faz uma tradução para a direita, o que nos leva ao triângulo 2. Em seguida, ele é escalado, passando o triângulo para a posição 3. Este não é o local onde deveria estar, se estivesse espelhado na linha especificada. A segunda `translate`chamada corrige isso - "cancela" a tradução inicial e faz o triângulo 4 aparecer exatamente onde deveria.

Agora podemos desenhar um personagem espelhado na posição (100,0) ao virar o mundo ao redor do centro vertical do personagem.

```html
<canvas></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");
  let img = document.createElement("img");
  img.src = "img/player.png";
  let spriteW = 24, spriteH = 30;
  img.addEventListener("load", () => {
    flipHorizontally(cx, 100 + spriteW / 2);
    cx.drawImage(img, 0, 0, spriteW, spriteH,
                 100, 0, spriteW, spriteH);
  });
</script>
```

## Armazenando e Limpando Transformações

As transformações permanecem por aí. Todo o resto que desenhamos após desenhar esse personagem espelhado também seria espelhado. Isso pode ser inconveniente.

É possível salvar a transformação atual, fazer alguns desenhos e transformações e, em seguida, restaurar a transformação antiga. Geralmente, é a coisa certa a fazer para uma função que precisa transformar temporariamente o sistema de coordenadas. Primeiro, salvamos qualquer transformação que o código que chamou a função estava usando. Em seguida, a função funciona, adicionando mais transformações além da transformação atual. Finalmente, voltamos à transformação com a qual começamos.

Os métodos `save`e `restore`no contexto de tela 2D fazem esse gerenciamento de transformação. Eles conceitualmente mantêm uma pilha de estados de transformação. Quando você chama `save`, o estado atual é empurrado para a pilha e, quando você chama `restore`, o estado no topo da pilha é retirado e usado como transformação atual do contexto. Você também pode ligar `resetTransform`para redefinir completamente a transformação.

A `branch`função no exemplo a seguir ilustra o que você pode fazer com uma função que altera a transformação e, em seguida, chama uma função (neste caso), que continua desenhando com a transformação especificada.

Essa função desenha uma forma de árvore, desenhando uma linha, movendo o centro do sistema de coordenadas para o final da linha e chamando-se duas vezes - primeiro girado para a esquerda e depois girado para a direita. Cada chamada reduz o comprimento do ramo desenhado e a recursão para quando o comprimento cai abaixo de 8.

```html
<canvas width="600" height="300"></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");
  function branch(length, angle, scale) {
    cx.fillRect(0, 0, 1, length);
    if (length < 8) return;
    cx.save();
    cx.translate(0, length);
    cx.rotate(-angle);
    branch(length * scale, angle, scale);
    cx.rotate(2 * angle);
    branch(length * scale, angle, scale);
    cx.restore();
  }
  cx.translate(300, 0);
  branch(60, 0.5, 0.8);
</script>
```

Se as chamadas para `save`e `restore`não existissem, a segunda chamada recursiva `branch`terminaria com a posição e a rotação criadas pela primeira chamada. Não seria conectado ao ramo atual, mas ao ramo mais à direita, mais à direita, desenhado pela primeira chamada. A forma resultante também pode ser interessante, mas definitivamente não é uma árvore.

## De volta ao jogo

Agora sabemos o suficiente sobre desenho de tela para começar a trabalhar em um sistema de exibição baseado em tela para o jogo do [capítulo anterior](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2016%20-%20%20Projeto%20Um%20Jogo%20de%20Plataforma.md) . A nova tela não exibirá mais apenas caixas coloridas. Em vez disso, usaremos `drawImage`para desenhar figuras que representam os elementos do jogo.

Definimos outro tipo de objeto de exibição chamado `CanvasDisplay`, suportando a mesma interface `DOMDisplay`do [Capítulo 16](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2016%20-%20%20Projeto%20Um%20Jogo%20de%20Plataforma.md#domdisplay) , a saber, os métodos `syncState`e `clear`.

Este objeto mantém um pouco mais de informação do que `DOMDisplay`. Em vez de usar a posição de rolagem de seu elemento DOM, ele rastreia sua própria janela de visualização, que nos diz qual parte do nível estamos vendo atualmente. Por fim, ele mantém uma `flipPlayer`propriedade para que, mesmo quando o jogador esteja parado, fique voltado para a direção em que se mudou pela última vez.

```js
class CanvasDisplay {
  constructor(parent, level) {
    this.canvas = document.createElement("canvas");
    this.canvas.width = Math.min(600, level.width * scale);
    this.canvas.height = Math.min(450, level.height * scale);
    parent.appendChild(this.canvas);
    this.cx = this.canvas.getContext("2d");

    this.flipPlayer = false;

    this.viewport = {
      left: 0,
      top: 0,
      width: this.canvas.width / scale,
      height: this.canvas.height / scale
    };
  }

  clear() {
    this.canvas.remove();
  }
}
```

O `syncState`método primeiro calcula uma nova janela de visualização e depois desenha a cena do jogo na posição apropriada.

```js
CanvasDisplay.prototype.syncState = function(state) {
  this.updateViewport(state);
  this.clearDisplay(state.status);
  this.drawBackground(state.level);
  this.drawActors(state.actors);
};
```

Ao contrário `DOMDisplay`, este estilo de apresentação *não* tem que redesenhar o fundo em cada atualização. Como as formas em uma tela são apenas pixels, depois de desenhá-las, não há uma boa maneira de movê-las (ou removê-las). A única maneira de atualizar a exibição da tela é limpá-la e redesenhar a cena. Também podemos ter rolado, o que exige que o plano de fundo esteja em uma posição diferente.

O `updateViewport`método é semelhante ao `DOMDisplay`do `scrollPlayerIntoView`método. Ele verifica se o player está muito perto da borda da tela e move a janela de visualização quando esse for o caso.

```js
CanvasDisplay.prototype.updateViewport = function(state) {
  let view = this.viewport, margin = view.width / 3;
  let player = state.player;
  let center = player.pos.plus(player.size.times(0.5));

  if (center.x < view.left + margin) {
    view.left = Math.max(center.x - margin, 0);
  } else if (center.x > view.left + view.width - margin) {
    view.left = Math.min(center.x + margin - view.width,
                         state.level.width - view.width);
  }
  if (center.y < view.top + margin) {
    view.top = Math.max(center.y - margin, 0);
  } else if (center.y > view.top + view.height - margin) {
    view.top = Math.min(center.y + margin - view.height,
                        state.level.height - view.height);
  }
};
```

As chamadas `Math.max`e `Math.min`garantem que a janela de exibição não mostre espaço fora do nível. `Math.max(x, 0)`garante que o número resultante não seja menor que zero. `Math.min`Da mesma forma, garante que um valor permaneça abaixo de um determinado limite.

Ao limpar a tela, usaremos uma cor ligeiramente diferente, dependendo de o jogo ser ganho (mais brilhante) ou perdido (mais escuro).

```js
CanvasDisplay.prototype.clearDisplay = function(status) {
  if (status == "won") {
    this.cx.fillStyle = "rgb(68, 191, 255)";
  } else if (status == "lost") {
    this.cx.fillStyle = "rgb(44, 136, 214)";
  } else {
    this.cx.fillStyle = "rgb(52, 166, 251)";
  }
  this.cx.fillRect(0, 0,
                   this.canvas.width, this.canvas.height);
};
```

Para desenhar o plano de fundo, percorremos os blocos visíveis na viewport atual, usando o mesmo truque usado no `touches`método do [capítulo anterior](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2016%20-%20%20Projeto%20Um%20Jogo%20de%20Plataforma.md) .

```js
let otherSprites = document.createElement("img");
otherSprites.src = "img/sprites.png";

CanvasDisplay.prototype.drawBackground = function(level) {
  let {left, top, width, height} = this.viewport;
  let xStart = Math.floor(left);
  let xEnd = Math.ceil(left + width);
  let yStart = Math.floor(top);
  let yEnd = Math.ceil(top + height);

  for (let y = yStart; y < yEnd; y++) {
    for (let x = xStart; x < xEnd; x++) {
      let tile = level.rows[y][x];
      if (tile == "empty") continue;
      let screenX = (x - left) * scale;
      let screenY = (y - top) * scale;
      let tileX = tile == "lava" ? scale : 0;
      this.cx.drawImage(otherSprites,
                        tileX,         0, scale, scale,
                        screenX, screenY, scale, scale);
    }
  }
};
```

Azulejos que não estão vazios são desenhados com `drawImage`. A `otherSprites`imagem contém as imagens usadas para outros elementos que não o reprodutor. Ele contém, da esquerda para a direita, o revestimento de parede, o revestimento de lava e o sprite para uma moeda.

![Sprites para o nosso jogo](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/sprites_big.png)

Os blocos de plano de fundo têm 20 por 20 pixels, pois usaremos a mesma escala em que usamos `DOMDisplay`. Assim, o deslocamento para ladrilhos de lava é 20 (o valor da `scale`ligação) e o deslocamento para paredes é 0.

Não nos incomodamos em esperar a imagem do sprite carregar. Ligar `drawImage`com uma imagem que ainda não foi carregada simplesmente não fará nada. Portanto, podemos não conseguir desenhar o jogo adequadamente nos primeiros quadros, enquanto a imagem ainda está carregando, mas isso não é um problema sério. Como continuamos atualizando a tela, a cena correta aparecerá assim que o carregamento terminar.

O personagem ambulante mostrado anteriormente será usado para representar o jogador. O código que desenha precisa escolher o sprite e a direção certos, com base no movimento atual do jogador. Os oito primeiros sprites contêm uma animação ambulante. Quando o jogador está se movendo ao longo de um piso, alternamos entre eles com base no horário atual. Queremos trocar de quadro a cada 60 milissegundos, para que o tempo seja dividido por 60 primeiro. Quando o jogador está parado, desenhamos o nono sprite. Durante os saltos, reconhecidos pelo fato de que a velocidade vertical não é zero, usamos o décimo sprite mais à direita.

Como os sprites são um pouco mais largos que o objeto do jogador - 24 em vez de 16 pixels para permitir espaço para pés e braços - o método precisa ajustar a coordenada x e a largura em uma determinada quantidade ( `playerXOverlap`).

```js
let playerSprites = document.createElement("img");
playerSprites.src = "img/player.png";
const playerXOverlap = 4;

CanvasDisplay.prototype.drawPlayer = function(player, x, y,
                                              width, height){
  width += playerXOverlap * 2;
  x -= playerXOverlap;
  if (player.speed.x != 0) {
    this.flipPlayer = player.speed.x < 0;
  }

  let tile = 8;
  if (player.speed.y != 0) {
    tile = 9;
  } else if (player.speed.x != 0) {
    tile = Math.floor(Date.now() / 60) % 8;
  }

  this.cx.save();
  if (this.flipPlayer) {
    flipHorizontally(this.cx, x + width / 2);
  }
  let tileX = tile * width;
  this.cx.drawImage(playerSprites, tileX, 0, width, height,
                                   x,     y, width, height);
  this.cx.restore();
};
```

O `drawPlayer`método é chamado por `drawActors`, responsável por desenhar todos os atores do jogo.

```js
CanvasDisplay.prototype.drawActors = function(actors) {
  for (let actor of actors) {
    let width = actor.size.x * scale;
    let height = actor.size.y * scale;
    let x = (actor.pos.x - this.viewport.left) * scale;
    let y = (actor.pos.y - this.viewport.top) * scale;
    if (actor.type == "player") {
      this.drawPlayer(actor, x, y, width, height);
    } else {
      let tileX = (actor.type == "coin" ? 2 : 1) * scale;
      this.cx.drawImage(otherSprites,
                        tileX, 0, width, height,
                        x,     y, width, height);
    }
  }
};
```

Ao desenhar algo que não é o jogador, examinamos seu tipo para encontrar o deslocamento do sprite correto. O ladrilho de lava é encontrado no deslocamento 20 e o sprite de moeda é encontrado em 40 (duas vezes `scale`).

Temos que subtrair a posição da viewport ao calcular a posição do ator, já que (0,0) em nossa tela corresponde à parte superior esquerda da viewport, não à parte superior esquerda do nível. Nós também poderíamos ter usado `translate`para isso. De qualquer maneira funciona.

Este documento conecta a nova exibição em `runGame`:

```html
<body>
  <script>
    runGame(GAME_LEVELS, CanvasDisplay);
  </script>
</body>
```

## Escolhendo uma interface gráfica

Portanto, quando você precisar gerar gráficos no navegador, poderá escolher entre HTML simples, SVG e tela. Não existe uma *melhor* abordagem única que funcione em todas as situações. Cada opção tem pontos fortes e fracos.

HTML simples tem a vantagem de ser simples. Também se integra bem ao texto. Tanto o SVG quanto a tela de desenho permitem desenhar texto, mas não ajudam a posicionar o texto ou quebrá-lo quando ele ocupa mais de uma linha. Em uma imagem baseada em HTML, é muito mais fácil incluir blocos de texto.

O SVG pode ser usado para produzir gráficos nítidos com boa aparência em qualquer nível de zoom. Ao contrário do HTML, ele é projetado para desenhar e, portanto, é mais adequado para esse fim.

SVG e HTML constroem uma estrutura de dados (o DOM) que representa sua imagem. Isso torna possível modificar elementos depois que eles são desenhados. Se você precisar alterar repetidamente uma pequena parte de uma imagem grande em resposta ao que o usuário está fazendo ou como parte de uma animação, fazê-lo em uma tela pode ser desnecessariamente caro. O DOM também nos permite registrar manipuladores de eventos do mouse em todos os elementos da imagem (mesmo nas formas desenhadas com SVG). Você não pode fazer isso com tela.

Mas a abordagem orientada a pixels do canvas pode ser uma vantagem ao desenhar um grande número de elementos minúsculos. O fato de não criar uma estrutura de dados, mas apenas desenhar repetidamente na mesma superfície de pixel, proporciona à tela um custo menor por forma.

Também existem efeitos, como renderizar uma cena um pixel de cada vez (por exemplo, usando um traçador de raios) ou pós-processamento de uma imagem com JavaScript (desfocando ou distorcendo), que podem ser manipulados realisticamente apenas por uma abordagem baseada em pixels.

Em alguns casos, convém combinar várias dessas técnicas. Por exemplo, você pode desenhar um gráfico com SVG ou tela, mas mostrar informações textuais posicionando um elemento HTML na parte superior da imagem.

Para aplicativos não exigentes, realmente não importa muito qual interface você escolher. A exibição que construímos para o nosso jogo neste capítulo poderia ter sido implementada usando qualquer uma dessas três tecnologias gráficas, uma vez que não é necessário desenhar texto, manipular a interação do mouse ou trabalhar com um número extraordinariamente grande de elementos.

## Sumário

Neste capítulo, discutimos técnicas para desenhar gráficos no navegador, focando no ``elemento.

Um nó de tela representa uma área em um documento que nosso programa pode usar. Este desenho é feito através de um objeto de contexto de desenho, criado com o `getContext`método

A interface de desenho 2D nos permite preencher e traçar várias formas. A `fillStyle`propriedade do contexto determina como as formas são preenchidas. As propriedades `strokeStyle`e `lineWidth`controlam a maneira como as linhas são desenhadas.

Retângulos e pedaços de texto podem ser desenhados com uma única chamada de método. Os métodos `fillRect`e `strokeRect`desenham retângulos e os métodos `fillText`e `strokeText`desenham texto. Para criar formas personalizadas, primeiro precisamos criar um caminho.

A chamada `beginPath`inicia um novo caminho. Vários outros métodos adicionam linhas e curvas ao caminho atual. Por exemplo, `lineTo`pode adicionar uma linha reta. Quando um caminho é concluído, ele pode ser preenchido com o `fill`método ou traçado com o `stroke`método.

A movimentação de pixels de uma imagem ou outra tela para a nossa tela é feita com o `drawImage`método Por padrão, esse método desenha toda a imagem de origem, mas, ao fornecer mais parâmetros, você pode copiar uma área específica da imagem. Usamos isso no nosso jogo, copiando poses individuais do personagem do jogo de uma imagem que continha muitas dessas poses.

As transformações permitem desenhar uma forma em várias orientações. Um contexto 2D desenho tem uma transformação de corrente que pode ser mudado com os `translate`, `scale`e `rotate`métodos. Isso afetará todas as operações de desenho subsequentes. Um estado de transformação pode ser salvo com o `save`método e restaurado com o `restore`método.

Ao mostrar uma animação em uma tela, o `clearRect`método pode ser usado para limpar parte da tela antes de redesenhar.

## Exercícios

### Formas

Escreva um programa que desenhe as seguintes formas em uma tela:

1. Um trapézio (um retângulo mais largo de um lado)
2. Um diamante vermelho (um retângulo girado 45 graus ou ¼π radianos)
3. Uma linha em zigue-zague
4. Uma espiral composta por 100 segmentos de linha reta
5. Uma estrela amarela

![As formas a desenhar](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/exercise_shapes.png)

Ao desenhar os dois últimos, convém consultar a explicação `Math.cos`e `Math.sin`no [Capítulo 14](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2014%20-%20O%20Modelo%20de%20Objeto%20do%20Documento.md#sin_cos) , que descreve como obter coordenadas em um círculo usando essas funções.

Eu recomendo criar uma função para cada forma. Passe a posição e, opcionalmente, outras propriedades, como o tamanho ou o número de pontos, como parâmetros. A alternativa, que é codificar números em todo o código, tende a tornar desnecessariamente difícil ler e modificar o código.

```html
<canvas width="600" height="200"></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");

  // Your code here.
</script>
```

O trapézio (1) é mais fácil de desenhar usando um caminho. Escolha as coordenadas do centro adequadas e adicione cada um dos quatro cantos ao redor do centro.

O diamante (2) pode ser desenhado de maneira direta, com um caminho, ou o interessante, com uma `rotate`transformação. Para usar a rotação, você terá que aplicar um truque semelhante ao que fizemos na `flipHorizontally`função. Como você deseja girar em torno do centro do seu retângulo e não em torno do ponto (0,0), primeiro você deve `translate`ir para lá, depois girar e traduzir novamente.

Certifique-se de redefinir a transformação depois de desenhar qualquer forma que a crie.

Para o ziguezague (3), torna-se impraticável escrever uma nova chamada `lineTo`para cada segmento de linha. Em vez disso, você deve usar um loop. Você pode fazer com que cada iteração desenhe dois segmentos de linha (à direita e depois à esquerda novamente) ou um; nesse caso, você deve usar a uniformidade ( `% 2`) do índice de loop para determinar se vai para a esquerda ou para a direita.

Você também precisará de um loop para a espiral (4). Se você desenhar uma série de pontos, com cada ponto avançando ao longo de um círculo ao redor do centro da espiral, você obtém um círculo. Se, durante o loop, você variar o raio do círculo no qual está colocando o ponto atual e girar mais de uma vez, o resultado será uma espiral.

A estrela (5) representada é construída a partir de `quadraticCurveTo`linhas. Você também pode desenhar um com linhas retas. Divida um círculo em oito peças para uma estrela com oito pontos, ou quantas peças desejar. Desenhe linhas entre esses pontos, fazendo-os se curvar em direção ao centro da estrela. Com `quadraticCurveTo`, você pode usar o centro como ponto de controle.

### O gráfico de pizza

[No início](https://eloquentjavascript.net/17_canvas.html#pie_chart) do capítulo, vimos um programa de exemplo que desenhou um gráfico de pizza. Modifique este programa para que o nome de cada categoria seja mostrado ao lado da fatia que o representa. Tente encontrar uma maneira agradável de posicionar automaticamente esse texto que funcionaria também para outros conjuntos de dados. Você pode assumir que as categorias são grandes o suficiente para deixar amplo espaço para seus rótulos.

Você pode precisar `Math.sin`e `Math.cos`mais uma vez, que são descritos no [Capítulo 14]https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2014%20-%20O%20Modelo%20de%20Objeto%20do%20Documento.md#sin_cos) .

```html
<canvas width="600" height="300"></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");
  let total = results
    .reduce((sum, {count}) => sum + count, 0);
  let currentAngle = -0.5 * Math.PI;
  let centerX = 300, centerY = 150;

  // Add code to draw the slice labels in this loop.
  for (let result of results) {
    let sliceAngle = (result.count / total) * 2 * Math.PI;
    cx.beginPath();
    cx.arc(centerX, centerY, 100,
           currentAngle, currentAngle + sliceAngle);
    currentAngle += sliceAngle;
    cx.lineTo(centerX, centerY);
    cx.fillStyle = result.color;
    cx.fill();
  }
</script>
```

Você precisará chamar `fillText`e definir o contexto `textAlign`e as `textBaseline`propriedades de forma que o texto termine onde você desejar.

Uma maneira sensata de posicionar os rótulos seria colocar o texto na linha que vai do centro da torta até o meio da fatia. Você não deseja colocar o texto diretamente na lateral da torta, mas mover o texto para a lateral da torta por um determinado número de pixels.

O ângulo desta linha é . O código a seguir localiza uma posição nesta linha a 120 pixels do centro:`currentAngle + 0.5 * sliceAngle`

```js
let middleAngle = currentAngle + 0.5 * sliceAngle;
let textX = Math.cos(middleAngle) * 120 + centerX;
let textY = Math.sin(middleAngle) * 120 + centerY;
```

Pois `textBaseline`, o valor `"middle"`provavelmente é apropriado ao usar essa abordagem. O que usar `textAlign`depende de qual lado do círculo estamos. À esquerda, deve estar `"right"`, e à direita, deve estar `"left"`, para que o texto fique posicionado longe da torta.

Se você não tiver certeza de como descobrir de que lado do círculo está um determinado ângulo, veja a explicação `Math.cos`no [Capítulo 14](https://eloquentjavascript.net/14_dom.html#sin_cos) . O cosseno de um ângulo nos diz a que coordenada x corresponde, o que, por sua vez, nos diz exatamente em que lado do círculo estamos.

### Uma bola quicando

Use a `requestAnimationFrame`técnica que vimos no [capítulo 14](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2014%20-%20O%20Modelo%20de%20Objeto%20do%20Documento.md#animationFrame) e no [capítulo 16](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2016%20-%20%20Projeto%20Um%20Jogo%20de%20Plataforma.md#runAnimation) para desenhar uma caixa com uma bola quicando. A bola se move a uma velocidade constante e bate nos lados da caixa quando a atinge.

```html
<canvas width="400" height="400"></canvas>
<script>
  let cx = document.querySelector("canvas").getContext("2d");

  let lastTime = null;
  function frame(time) {
    if (lastTime != null) {
      updateAnimation(Math.min(100, time - lastTime) / 1000);
    }
    lastTime = time;
    requestAnimationFrame(frame);
  }
  requestAnimationFrame(frame);

  function updateAnimation(step) {
    // Your code here.
  }
</script>
```

É fácil desenhar uma caixa `strokeRect`. Defina uma encadernação que mantenha seu tamanho ou defina duas encadernações se a largura e a altura da caixa forem diferentes. Para criar uma bola redonda, inicie um caminho e uma chamada `arc(x, y, radius, 0, 7)`, que cria um arco que vai de zero a mais do que um círculo inteiro. Em seguida, preencha o caminho.

Para modelar a posição e a velocidade da bola, você pode usar a `Vec`classe do [Capítulo 16]https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2016%20-%20%20Projeto%20Um%20Jogo%20de%20Plataforma.md#vector) (que está disponível nesta página). Dê a ela uma velocidade inicial, de preferência uma que não seja puramente vertical ou horizontal, e para cada quadro multiplique essa velocidade pela quantidade de tempo decorrido. Quando a bola se aproximar demais de uma parede vertical, inverta o componente x em sua velocidade. Da mesma forma, inverta o componente y quando atingir uma parede horizontal.

Depois de encontrar a nova posição e velocidade da bola, use `clearRect`para excluir a cena e redesenhá-la usando a nova posição.

### Espelhamento pré-computado

Uma coisa infeliz sobre transformações é que elas diminuem a velocidade do desenho dos bitmaps. A posição e o tamanho de cada pixel precisam ser transformados e, embora seja possível que os navegadores fiquem mais espertos sobre a transformação no futuro, eles atualmente causam um aumento mensurável no tempo necessário para desenhar um bitmap.

Em um jogo como o nosso, onde estamos desenhando apenas um único sprite transformado, isso não é uma questão. Mas imagine que precisamos desenhar centenas de caracteres ou milhares de partículas rotativas de uma explosão.

Pense em uma maneira de nos permitir desenhar um caractere invertido sem carregar arquivos de imagem adicionais e sem precisar fazer `drawImage`chamadas transformadas a cada quadro.

A chave da solução é o fato de podermos usar um elemento de tela como imagem de origem ao usá-lo `drawImage`. É possível criar um ``elemento extra , sem adicioná-lo ao documento, e desenhar nossos sprites invertidos uma vez. Ao desenhar um quadro real, apenas copiamos os sprites já invertidos na tela principal.

Alguns cuidados seriam necessários porque as imagens não são carregadas instantaneamente. Fazemos o desenho invertido apenas uma vez e, se o fizermos antes da imagem carregar, ela não desenhará nada. Um `"load"`manipulador na imagem pode ser usado para desenhar as imagens invertidas na tela extra. Essa tela pode ser usada como uma fonte de desenho imediatamente (ela ficará em branco até que desenhemos o personagem).