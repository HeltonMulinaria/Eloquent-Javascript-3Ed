# Capítulo 15 Tratamento de eventos

> Você tem poder sobre sua mente - não eventos externos. Perceba isso e você encontrará força.
>
> *—Marco Aurélio, Meditações*

![Imagine uma máquina Rube Goldberg](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/chapter_picture_15.jpg)

Alguns programas funcionam com entrada direta do usuário, como ações de mouse e teclado. Esse tipo de entrada não está disponível como uma estrutura de dados bem organizada - é fornecida peça por peça, em tempo real, e espera-se que o programa responda a ela assim que acontece.

## Manipuladores de eventos

Imagine uma interface em que a única maneira de descobrir se uma tecla do teclado está sendo pressionada é ler o estado atual dessa tecla. Para poder reagir a pressionamentos de tecla, é necessário ler constantemente o estado da chave para capturá-la antes que ela seja liberada novamente. Seria perigoso realizar outros cálculos que demandam muito tempo, pois você pode perder o pressionamento de tecla.

Algumas máquinas primitivas lidam com entradas assim. Um passo adiante seria o hardware ou o sistema operacional perceber o pressionamento de tecla e colocá-lo em uma fila. Um programa pode então verificar periodicamente a fila para novos eventos e reagir ao que encontra lá.

Obviamente, é preciso lembrar de olhar para a fila e fazê-lo com frequência, porque qualquer momento entre a tecla ser pressionada e o programa que está percebendo o evento fará com que o software pareça não responder. Essa abordagem é chamada de *polling* . A maioria dos programadores prefere evitá-lo.

Um mecanismo melhor é o sistema notificar ativamente nosso código quando ocorrer um evento. Os navegadores fazem isso, permitindo que registremos funções como *manipuladores* para eventos específicos.

```html
<p>Click this document to activate the handler.</p>
<script>
  window.addEventListener("click", () => {
    console.log("You knocked?");
  });
</script>
```

A `window`ligação refere-se a um objeto interno fornecido pelo navegador. Representa a janela do navegador que contém o documento. A chamada de seu `addEventListener`método registra o segundo argumento a ser chamado sempre que o evento descrito por seu primeiro argumento ocorre.

## Eventos e nós DOM

Cada manipulador de eventos do navegador é registrado em um contexto. No exemplo anterior, chamamos `addEventListener`o `window`objeto para registrar um manipulador para toda a janela. Esse método também pode ser encontrado em elementos DOM e em alguns outros tipos de objetos. Os ouvintes de eventos são chamados apenas quando o evento acontece no contexto do objeto em que estão registrados.

```html
<button>Click me</button>
<p>No handler here.</p>
<script>
  let button = document.querySelector("button");
  button.addEventListener("click", () => {
    console.log("Button clicked.");
  });
</script>
```

Esse exemplo anexa um manipulador ao nó do botão. Cliques no botão fazem com que o manipulador seja executado, mas cliques no restante do documento não.

Dar um `onclick`atributo a um nó tem um efeito semelhante. Isso funciona para a maioria dos tipos de eventos - você pode anexar um manipulador através do atributo cujo nome é o nome do evento à sua `on`frente.

Mas um nó pode ter apenas um `onclick`atributo, portanto, você pode registrar apenas um manipulador por nó dessa maneira. O `addEventListener`método permite adicionar qualquer número de manipuladores para que seja seguro adicionar manipuladores, mesmo que já exista outro manipulador no elemento.

O `removeEventListener`método, chamado com argumentos semelhantes a `addEventListener`, remove um manipulador.

```html
<button>Act-once button</button>
<script>
  let button = document.querySelector("button");
  function once() {
    console.log("Done.");
    button.removeEventListener("click", once);
  }
  button.addEventListener("click", once);
</script>
```

A função atribuída `removeEventListener`deve ser o mesmo valor da função atribuída `addEventListener`. Portanto, para cancelar o registro de um manipulador, dê um nome à função ( `once`no exemplo) para poder passar o mesmo valor da função para os dois métodos.

## Objetos de evento

Embora o tenhamos ignorado até agora, as funções do manipulador de eventos recebem um argumento: o *objeto de evento* . Este objeto contém informações adicionais sobre o evento. Por exemplo, se queremos saber *qual* botão do mouse foi pressionado, podemos olhar para a `button`propriedade do objeto de evento .

```html
<button>Click me any way you want</button>
<script>
  let button = document.querySelector("button");
  button.addEventListener("mousedown", event => {
    if (event.button == 0) {
      console.log("Left button");
    } else if (event.button == 1) {
      console.log("Middle button");
    } else if (event.button == 2) {
      console.log("Right button");
    }
  });
</script>
```

As informações armazenadas em um objeto de evento diferem por tipo de evento. Discutiremos diferentes tipos mais adiante neste capítulo. A `type`propriedade do objeto sempre mantém uma cadeia de caracteres que identifica o evento (como `"click"`ou `"mousedown"`).

## Propagação

Para a maioria dos tipos de eventos, os manipuladores registrados em nós com filhos também receberão eventos que acontecem nos filhos. Se um botão dentro de um parágrafo for clicado, os manipuladores de eventos no parágrafo também verão o evento click.

Mas se o parágrafo e o botão tiverem um manipulador, o manipulador mais específico - aquele no botão - será o primeiro. Diz-se que o evento se *propaga* para fora, a partir do nó em que ocorreu com o nó pai desse nó e até a raiz do documento. Finalmente, depois que todos os manipuladores registrados em um nó específico tiverem sua vez, os manipuladores registrados em toda a janela terão a chance de responder ao evento.

A qualquer momento, um manipulador de eventos pode chamar o `stopPropagation`método no objeto de evento para impedir que os manipuladores recebam o evento. Isso pode ser útil quando, por exemplo, você possui um botão dentro de outro elemento clicável e não deseja cliques no botão para ativar o comportamento de cliques do elemento externo.

O exemplo a seguir registra os `"mousedown"`manipuladores em um botão e o parágrafo ao seu redor. Quando clicado com o botão direito do mouse, o manipulador do botão chama `stopPropagation`, o que impedirá a execução do manipulador no parágrafo. Quando o botão é clicado com outro botão do mouse, os dois manipuladores serão executados.

```html
<p>A paragraph with a <button>button</button>.</p>
<script>
  let para = document.querySelector("p");
  let button = document.querySelector("button");
  para.addEventListener("mousedown", () => {
    console.log("Handler for paragraph.");
  });
  button.addEventListener("mousedown", event => {
    console.log("Handler for button.");
    if (event.button == 2) event.stopPropagation();
  });
</script>
```

A maioria dos objetos de eventos possui uma `target`propriedade que se refere ao nó em que foram originados. Você pode usar essa propriedade para garantir que não esteja manipulando acidentalmente algo que se propagou a partir de um nó que não deseja manipular.

Também é possível usar a `target`propriedade para lançar uma ampla rede para um tipo específico de evento. Por exemplo, se você tiver um nó que contém uma lista longa de botões, pode ser mais conveniente registrar um manipulador de clique único no nó externo e usar a `target`propriedade para descobrir se um botão foi clicado, em vez de registrar manipuladores individuais em todos os botões.

```html
<button>A</button>
<button>B</button>
<button>C</button>
<script>
  document.body.addEventListener("click", event => {
    if (event.target.nodeName == "BUTTON") {
      console.log("Clicked", event.target.textContent);
    }
  });
</script>
```

## Ações padrão

Muitos eventos têm uma ação padrão associada a eles. Se você clicar em um link, será direcionado para o destino do link. Se você pressionar a seta para baixo, o navegador rolará a página para baixo. Se você clicar com o botão direito, obterá um menu de contexto. E assim por diante.

Para a maioria dos tipos de eventos, os manipuladores de eventos JavaScript são chamados *antes que* o comportamento padrão ocorra. Se o manipulador não desejar que esse comportamento normal aconteça, geralmente porque ele já cuidou do tratamento do evento, poderá chamar o `preventDefault`método no objeto de evento.

Isso pode ser usado para implementar seus próprios atalhos de teclado ou menu de contexto. Também pode ser usado para interferir desagradável com o comportamento que os usuários esperam. Por exemplo, aqui está um link que não pode ser seguido:

```html
<a href="https://developer.mozilla.org/">MDN</a>
<script>
  let link = document.querySelector("a");
  link.addEventListener("click", event => {
    console.log("Nope.");
    event.preventDefault();
  });
</script>
```

Tente não fazer essas coisas, a menos que você tenha um bom motivo para fazê-lo. Será desagradável para as pessoas que usam sua página quando o comportamento esperado é interrompido.

Dependendo do navegador, alguns eventos não podem ser interceptados. No Chrome, por exemplo, o atalho de teclado para fechar a guia atual ( controle -W ou comando -W) não pode ser manipulado pelo JavaScript.

## Eventos principais

Quando uma tecla do teclado é pressionada, o navegador dispara um `"keydown"`evento. Quando é lançado, você recebe um `"keyup"`evento.

```html
<p>This page turns violet when you hold the V key.</p>
<script>
  window.addEventListener("keydown", event => {
    if (event.key == "v") {
      document.body.style.background = "violet";
    }
  });
  window.addEventListener("keyup", event => {
    if (event.key == "v") {
      document.body.style.background = "";
    }
  });
</script>
```

Apesar do nome, é `"keydown"`acionado não apenas quando a tecla é pressionada fisicamente. Quando uma tecla é pressionada e mantida, o evento é acionado novamente toda vez que a tecla é *repetida* . Às vezes você tem que ter cuidado com isso. Por exemplo, se você adicionar um botão ao DOM quando uma tecla for pressionada e removê-lo novamente quando a tecla for liberada, acidentalmente poderá adicionar centenas de botões quando a tecla for pressionada por mais tempo.

O exemplo analisou a `key`propriedade do objeto de evento para ver de que tecla trata o evento. Essa propriedade contém uma sequência que, para a maioria das teclas, corresponde à coisa que pressionar essa tecla seria digitada. Para chaves especiais, como enter , ele contém uma sequência que nomeia a chave ( `"Enter"`neste caso). Se você pressionar Shift enquanto pressiona uma tecla, isso também pode influenciar o nome da tecla - `"v"`se torna `"V"`, e `"1"`pode se tornar `"!"`, se é isso que pressionar Shift -1 produz no teclado.

As teclas modificadoras, como shift , control , alt e meta ( comando no Mac), geram eventos-chave como as teclas normais. Mas quando se olha para combinações de teclas, você também pode descobrir se estas teclas são pressionadas por olhar para o `shiftKey`, `ctrlKey`, `altKey`, e `metaKey`propriedades de eventos de teclado e mouse.

```html
<p>Press Control-Space to continue.</p>
<script>
  window.addEventListener("keydown", event => {
    if (event.key == " " && event.ctrlKey) {
      console.log("Continuing!");
    }
  });
</script>
```

O nó DOM em que um evento de chave se origina depende do elemento em foco quando a tecla é pressionada. A maioria dos nós não pode ter foco, a menos que você atribua a eles um `tabindex`atributo, mas coisas como links, botões e campos de formulário podem. Voltaremos aos campos de formulário no [capítulo 18](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2018%20-%20%20HTTP%20e%20formul%C3%A1rios.md#forms) . Quando nada em particular tem foco, `document.body`atua como o nó de destino dos principais eventos.

Quando o usuário está digitando texto, é problemático o uso de eventos importantes para descobrir o que está sendo digitado. Algumas plataformas, principalmente o teclado virtual em telefones Android, não acionam eventos importantes. Mas mesmo quando você possui um teclado antiquado, alguns tipos de entrada de texto não correspondem às teclas pressionadas de maneira direta, como o software IME ( *Input Method Editor* ) usado por pessoas cujos scripts não cabem no teclado, onde várias teclas são combinadas para criar caracteres.

Para observar quando algo foi digitado, os elementos nos quais você pode digitar, como as tags ``e ``, acionam `"input"`eventos sempre que o usuário altera seu conteúdo. Para obter o conteúdo real digitado, é melhor lê-lo diretamente no campo em foco. [O capítulo 18](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2018%20-%20%20HTTP%20e%20formul%C3%A1rios.md#forms) mostrará como.

## Eventos de ponteiro

Atualmente, existem duas maneiras amplamente usadas de apontar para as coisas na tela: mouses (incluindo dispositivos que agem como mouses, como touchpads e trackballs) e telas sensíveis ao toque. Eles produzem diferentes tipos de eventos.

### Cliques do mouse

Pressionar o botão do mouse faz com que vários eventos sejam acionados. O `"mousedown"`e `"mouseup"`eventos são semelhantes `"keydown"`e `"keyup"`e fogo quando o botão é pressionado e liberado. Isso acontece nos nós DOM que estão imediatamente abaixo do ponteiro do mouse quando o evento ocorre.

Após o `"mouseup"`evento, um `"click"`evento é acionado no nó mais específico que continha a imprensa e o lançamento do botão. Por exemplo, se eu pressionar o botão do mouse em um parágrafo e depois mover o ponteiro para outro parágrafo e soltar o botão, o `"click"`evento ocorrerá no elemento que contém os dois parágrafos.

Se dois cliques acontecerem juntos, um `"dblclick"`evento (clique duplo) também será acionado após o segundo evento de clique.

Para obter informações precisas sobre o local em que um evento do mouse ocorreu, você pode observar as propriedades `clientX`e `clientY`, que contêm as coordenadas do evento (em pixels) em relação ao canto superior esquerdo da janela ou `pageX`e e em `pageY`relação ao topo canto esquerdo do documento inteiro (que pode ser diferente quando a janela foi rolada).

A seguir, implementa um programa de desenho primitivo. Sempre que você clica no documento, ele adiciona um ponto ao ponteiro do mouse. Veja o [Capítulo 19](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2019%20%20-%20Projeto%20Um%20editor%20de%20pixel%20art.md) para um programa de desenho menos primitivo.

```html
<style>
  body {
    height: 200px;
    background: beige;
  }
  .dot {
    height: 8px; width: 8px;
    border-radius: 4px; /* rounds corners */
    background: blue;
    position: absolute;
  }
</style>
<script>
  window.addEventListener("click", event => {
    let dot = document.createElement("div");
    dot.className = "dot";
    dot.style.left = (event.pageX - 4) + "px";
    dot.style.top = (event.pageY - 4) + "px";
    document.body.appendChild(dot);
  });
</script>
```

### Movimento do mouse

Sempre que o ponteiro do mouse se move, um `"mousemove"`evento é disparado. Este evento pode ser usado para rastrear a posição do mouse. Uma situação comum em que isso é útil é ao implementar alguma forma de funcionalidade de arrastar o mouse.

Como exemplo, o programa a seguir exibe uma barra e configura manipuladores de eventos para que arrastar para a esquerda ou direita nessa barra a torne mais estreita ou mais larga:

```html
<p>Drag the bar to change its width:</p>
<div style="background: orange; width: 60px; height: 20px">
</div>
<script>
  let lastX; // Tracks the last observed mouse X position
  let bar = document.querySelector("div");
  bar.addEventListener("mousedown", event => {
    if (event.button == 0) {
      lastX = event.clientX;
      window.addEventListener("mousemove", moved);
      event.preventDefault(); // Prevent selection
    }
  });

  function moved(event) {
    if (event.buttons == 0) {
      window.removeEventListener("mousemove", moved);
    } else {
      let dist = event.clientX - lastX;
      let newWidth = Math.max(10, bar.offsetWidth + dist);
      bar.style.width = newWidth + "px";
      lastX = event.clientX;
    }
  }
</script>
```

Observe que o `"mousemove"`manipulador está registrado em toda a janela. Mesmo que o mouse saia da barra durante o redimensionamento, enquanto o botão estiver pressionado, ainda queremos atualizar seu tamanho.

Devemos parar de redimensionar a barra quando o botão do mouse for liberado. Para isso, podemos usar a `buttons`propriedade (observe o plural), que nos informa sobre os botões atualmente pressionados. Quando este é zero, nenhum botão está pressionado. Quando os botões são pressionados, seu valor é a soma dos códigos para esses botões - o botão esquerdo tem o código 1, o botão direito 2 e o meio 4. Dessa forma, você pode verificar se um determinado botão é pressionado pressionando o botão restante do valor de `buttons`e seu código.

Observe que a ordem desses códigos é diferente da usada por `button`onde o botão do meio veio antes do direito. Como mencionado, a consistência não é realmente um ponto forte da interface de programação do navegador.

### Eventos de toque

O estilo do navegador gráfico que usamos foi projetado com interfaces de mouse em mente, em um momento em que as telas sensíveis ao toque eram raras. Para fazer a Web "funcionar" nos primeiros telefones com tela sensível ao toque, os navegadores desses dispositivos fingiram, até certo ponto, que os eventos de toque eram eventos de mouse. Se você tocar em sua tela, você vai ter `"mousedown"`, `"mouseup"`e `"click"`eventos.

Mas essa ilusão não é muito robusta. Uma tela sensível ao toque funciona de maneira diferente de um mouse: não possui vários botões, não é possível rastrear o dedo quando não está na tela (para simular `"mousemove"`) e permite que vários dedos estejam na tela ao mesmo tempo .

Os eventos do mouse abrangem a interação por toque apenas em casos simples - se você adicionar um `"click"`manipulador a um botão, os usuários ainda poderão usá-lo. Mas algo como a barra redimensionável no exemplo anterior não funciona em uma tela sensível ao toque.

Existem tipos de eventos específicos acionados por interação por toque. Quando um dedo começa a tocar na tela, você recebe um `"touchstart"`evento. Quando é movido enquanto se toca, os `"touchmove"`eventos disparam. Por fim, quando ele parar de tocar na tela, você verá um `"touchend"`evento.

Como muitas telas sensíveis ao toque podem detectar vários dedos ao mesmo tempo, esses eventos não têm um único conjunto de coordenadas associado a eles. Em vez disso, seus objetos de evento tem uma `touches`propriedade, que possui um objeto de matriz semelhante de pontos, cada um dos quais tem seus próprios `clientX`, `clientY`, `pageX`, e `pageY`propriedades.

Você pode fazer algo assim para mostrar círculos vermelhos ao redor de cada dedo tocante:

```html
<style>
  dot { position: absolute; display: block;
        border: 2px solid red; border-radius: 50px;
        height: 100px; width: 100px; }
</style>
<p>Touch this page</p>
<script>
  function update(event) {
    for (let dot; dot = document.querySelector("dot");) {
      dot.remove();
    }
    for (let i = 0; i < event.touches.length; i++) {
      let {pageX, pageY} = event.touches[i];
      let dot = document.createElement("dot");
      dot.style.left = (pageX - 50) + "px";
      dot.style.top = (pageY - 50) + "px";
      document.body.appendChild(dot);
    }
  }
  window.addEventListener("touchstart", update);
  window.addEventListener("touchmove", update);
  window.addEventListener("touchend", update);
</script>
```

Em geral, convém chamar os `preventDefault`manipuladores de eventos de toque para substituir o comportamento padrão do navegador (que pode incluir a rolagem da página ao passar o dedo) e impedir que os eventos do mouse sejam acionados, para os quais você *também* pode ter um manipulador.

## Eventos de rolagem

Sempre que um elemento é rolado, um `"scroll"`evento é acionado nele. Isso tem vários usos, como saber o que o usuário está vendo no momento (para desativar animações fora da tela ou enviar relatórios de espionagem para o quartel general do mal) ou mostrar alguma indicação de progresso (destacando parte de um sumário ou mostrando uma página) número).

O exemplo a seguir desenha uma barra de progresso acima do documento e a atualiza para preencher à medida que você rola para baixo:

```html
<style>
  #progress {
    border-bottom: 2px solid blue;
    width: 0;
    position: fixed;
    top: 0; left: 0;
  }
</style>
<div id="progress"></div>
<script>
  // Create some content
  document.body.appendChild(document.createTextNode(
    "supercalifragilisticexpialidocious ".repeat(1000)));

  let bar = document.querySelector("#progress");
  window.addEventListener("scroll", () => {
    let max = document.body.scrollHeight - innerHeight;
    bar.style.width = `${(pageYOffset / max) * 100}%`;
  });
</script>
```

Atribuir um elemento a `position`de `fixed`forma muito semelhante a uma `absolute`posição, mas também impede que ele role com o restante do documento. O efeito é fazer com que nossa barra de progresso fique no topo. Sua largura é alterada para indicar o progresso atual. Usamos `%`, em vez de `px`, como uma unidade ao definir a largura para que o elemento seja dimensionado em relação à largura da página.

A `innerHeight`encadernação global nos fornece a altura da janela, que é preciso subtrair da altura total de rolagem - você não pode continuar rolando quando atinge a parte inferior do documento. Há também um `innerWidth`para a largura da janela. Ao dividir `pageYOffset`a posição atual de rolagem, pela posição máxima de rolagem e multiplicar por 100, obtemos a porcentagem da barra de progresso.

A chamada `preventDefault`de um evento de rolagem não impede a rolagem. De fato, o manipulador de eventos é chamado somente *após* a rolagem.

## Eventos de foco

Quando um elemento ganha foco, o navegador dispara um `"focus"`evento nele. Quando perde o foco, o elemento recebe um `"blur"`evento.

Ao contrário dos eventos discutidos anteriormente, esses dois eventos não se propagam. Um manipulador em um elemento pai não é notificado quando um elemento filho ganha ou perde o foco.

O exemplo a seguir exibe o texto de ajuda para o campo de texto que atualmente tem foco:

```html
<p>Name: <input type="text" data-help="Your full name"></p>
<p>Age: <input type="text" data-help="Your age in years"></p>
<p id="help"></p>

<script>
  let help = document.querySelector("#help");
  let fields = document.querySelectorAll("input");
  for (let field of Array.from(fields)) {
    field.addEventListener("focus", event => {
      let text = event.target.getAttribute("data-help");
      help.textContent = text;
    });
    field.addEventListener("blur", event => {
      help.textContent = "";
    });
  }
</script>
```

O objeto da janela receberá `"focus"`e `"blur"`eventos quando o usuário for de ou para a guia ou janela do navegador na qual o documento é mostrado.

## Carregar evento

Quando uma página termina de carregar, o `"load"`evento é acionado na janela e nos objetos do corpo do documento. Isso geralmente é usado para agendar ações de inicialização que exigem que todo o documento tenha sido construído. Lembre-se de que o conteúdo das ``tags é executado imediatamente quando a tag é encontrada. Isso pode ser muito cedo, por exemplo, quando o script precisa fazer algo com partes do documento que aparecem após a ``tag.

Elementos como imagens e tags de script que carregam um arquivo externo também têm um `"load"`evento que indica que os arquivos a que se referem foram carregados. Como os eventos relacionados ao foco, os eventos de carregamento não se propagam.

Quando uma página é fechada ou afastada (por exemplo, seguindo um link), um `"beforeunload"`evento é acionado. O principal uso desse evento é impedir que o usuário perca acidentalmente o trabalho fechando um documento. Se você impedir o comportamento padrão desse evento *e* definir a `returnValue`propriedade no objeto de evento como uma sequência, o navegador mostrará ao usuário uma caixa de diálogo perguntando se ele realmente deseja sair da página. Essa caixa de diálogo pode incluir sua string, mas como alguns sites maliciosos tentam usá-las para confundir as pessoas a permanecerem em suas páginas para olhar para anúncios de perda de peso desonestos, a maioria dos navegadores não os exibe mais.

## Eventos e o loop de eventos

No contexto do loop de eventos, conforme discutido no [Capítulo 11](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2011%20-%20Programa%C3%A7%C3%A3o%20ass%C3%ADncrona.md) , os manipuladores de eventos do navegador se comportam como outras notificações assíncronas. Eles são agendados quando o evento ocorre, mas devem aguardar a conclusão de outros scripts em execução antes que eles possam executar.

O fato de que os eventos podem ser processados somente quando nada mais estiver em execução significa que, se o loop de eventos estiver vinculado a outro trabalho, qualquer interação com a página (que ocorre por meio de eventos) será adiada até que haja tempo para processá-lo. Portanto, se você agendar muito trabalho, seja com manipuladores de eventos de longa duração ou com muitos de curta duração, a página ficará lenta e complicada de usar.

Nos casos em que você *realmente* deseja fazer algo demorado em segundo plano sem congelar a página, os navegadores fornecem algo chamado *trabalhadores da Web* . Um trabalhador é um processo JavaScript executado ao lado do script principal, em sua própria linha do tempo.

Imagine que o quadrado de um número é uma computação pesada e de longa execução que queremos executar em um encadeamento separado. Poderíamos escrever um arquivo chamado que responde às mensagens calculando um quadrado e enviando uma mensagem de volta.`code/squareworker.js`

```js
addEventListener("message", event => {
  postMessage(event.data * event.data);
});
```

Para evitar os problemas de ter vários threads tocando nos mesmos dados, os trabalhadores não compartilham seu escopo global ou qualquer outro dado com o ambiente do script principal. Em vez disso, você precisa se comunicar com eles enviando mensagens para frente e para trás.

Esse código gera um trabalhador executando esse script, envia algumas mensagens e gera as respostas.

```js
let squareWorker = new Worker("code/squareworker.js");
squareWorker.addEventListener("message", event => {
  console.log("The worker responded:", event.data);
});
squareWorker.postMessage(10);
squareWorker.postMessage(24);
```

A `postMessage`função envia uma mensagem, o que fará com que um `"message"`evento seja disparado no receptor. O script que criou o trabalhador envia e recebe mensagens através do `Worker`objeto, enquanto o trabalhador fala com o script que o criou enviando e ouvindo diretamente em seu escopo global. Somente valores que podem ser representados como JSON podem ser enviados como mensagens - o outro lado receberá uma *cópia* deles, em vez do próprio valor.

## Temporizadores

Vimos a `setTimeout`função no [capítulo 11](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2011%20-%20Programa%C3%A7%C3%A3o%20ass%C3%ADncrona.md) . Ele agenda outra função a ser chamada posteriormente, após um determinado número de milissegundos.

Às vezes, você precisa cancelar uma função que agendou. Isso é feito armazenando o valor retornado `setTimeout`e chamando `clearTimeout`-o.

```js
let bombTimer = setTimeout(() => {
  console.log("BOOM!");
}, 500);

if (Math.random() < 0.5) { // 50% chance
  console.log("Defused.");
  clearTimeout(bombTimer);
}
```

A `cancelAnimationFrame`função funciona da mesma maneira que: `clearTimeout`chamar um valor retornado por `requestAnimationFrame`cancelará esse quadro (supondo que ele ainda não tenha sido chamado).

Um conjunto semelhante de funções, `setInterval`e `clearInterval`, é usado para definir temporizadores que devem *repetir a* cada *X* milissegundos.

```js
let ticks = 0;
let clock = setInterval(() => {
  console.log("tick", ticks++);
  if (ticks == 10) {
    clearInterval(clock);
    console.log("stop.");
  }
}, 200);
```

## Debouncing

Alguns tipos de eventos têm o potencial de disparar rapidamente, muitas vezes seguidas (os eventos `"mousemove"`e `"scroll"`, por exemplo). Ao manipular esses eventos, você deve tomar cuidado para não fazer algo que consome muito tempo ou seu manipulador levará tanto tempo que a interação com o documento começará a parecer lenta.

Se você precisar fazer algo não trivial nesse manipulador, use-o `setTimeout`para garantir que não o faça com muita frequência. Isso geralmente é chamado de *renúncia* ao evento. Existem várias abordagens ligeiramente diferentes para isso.

No primeiro exemplo, queremos reagir quando o usuário digitar algo, mas não queremos fazê-lo imediatamente para cada evento de entrada. Quando eles estão digitando rapidamente, queremos apenas aguardar uma pausa. Em vez de executar imediatamente uma ação no manipulador de eventos, definimos um tempo limite. Também limpamos o tempo limite anterior (se houver) para que, quando os eventos ocorram juntos (mais perto do que o atraso de tempo limite), o tempo limite do evento anterior seja cancelado.

```html
<textarea>Type something here...</textarea>
<script>
  let textarea = document.querySelector("textarea");
  let timeout;
  textarea.addEventListener("input", () => {
    clearTimeout(timeout);
    timeout = setTimeout(() => console.log("Typed!"), 500);
  });
</script>
```

Dar um valor indefinido `clearTimeout`ou chamá-lo com um tempo limite que já foi acionado não tem efeito. Portanto, não precisamos ter cuidado sobre quando chamá-lo, e simplesmente o fazemos para todos os eventos.

Podemos usar um padrão ligeiramente diferente se quisermos espaçar as respostas para que elas sejam separadas por pelo menos um certo período de tempo, mas deseje acioná-las *durante* uma série de eventos, e não apenas posteriormente. Por exemplo, podemos querer responder aos `"mousemove"`eventos mostrando as coordenadas atuais do mouse, mas apenas a cada 250 milissegundos.

```html
<script>
  let scheduled = null;
  window.addEventListener("mousemove", event => {
    if (!scheduled) {
      setTimeout(() => {
        document.body.textContent =
          `Mouse at ${scheduled.pageX}, ${scheduled.pageY}`;
        scheduled = null;
      }, 250);
    }
    scheduled = event;
  });
</script>
```

## Sumário

Os manipuladores de eventos possibilitam detectar e reagir a eventos que acontecem em nossa página da web. O `addEventListener`método é usado para registrar esse manipulador.

Cada evento tem um tipo ( `"keydown"`, `"focus"`e assim por diante) que o identifica. A maioria dos eventos é chamada em um elemento DOM específico e, em seguida, é *propagada* para os ancestrais desse elemento, permitindo que manipuladores associados a esses elementos os manipulem.

Quando um manipulador de eventos é chamado, é passado um objeto de evento com informações adicionais sobre o evento. Este objeto também possui métodos que nos permitem interromper a propagação ( `stopPropagation`) e impedir o tratamento padrão do navegador pelo evento ( `preventDefault`).

Pressionar uma tecla é acionado `"keydown"`e `"keyup"`eventos. Pressionando um botão do mouse incêndios `"mousedown"`, `"mouseup"`, e `"click"`eventos. Mover o mouse dispara `"mousemove"`eventos. Interação Touchscreen vai resultar em `"touchstart"`, `"touchmove"`e `"touchend"`eventos.

A rolagem pode ser detectada com o `"scroll"`evento e as alterações de foco podem ser detectadas com os eventos `"focus"`e `"blur"`. Quando o documento termina o carregamento, um `"load"`evento é disparado na janela.

## Exercícios

### Balão

Escreva uma página que exiba um balão (usando o emoji do balão, 🎈). Quando você pressiona a seta para cima, ela deve inflar (crescer) 10% e, quando você pressiona a seta para baixo, ela deve esvaziar (diminuir) 10%.

Você pode controlar o tamanho do texto (emoji é texto) configurando a `font-size`propriedade CSS ( `style.fontSize`) em seu elemento pai. Lembre-se de incluir uma unidade no valor - por exemplo, pixels ( `10px`).

Os nomes das teclas de seta são `"ArrowUp"`e `"ArrowDown"`. Verifique se as teclas alteram apenas o balão, sem rolar a página.

Quando isso funcionar, adicione um recurso em que, se você explodir o balão após um determinado tamanho, ele explodirá. Nesse caso, explodir significa que ele é substituído por um oji emoji e o manipulador de eventos é removido (para que você não possa inflar ou desinflar a explosão).

```html
<p>🎈</p>

<script>
  // Your code here
</script>
```

Você deseja registrar um manipulador para o `"keydown"`evento e verificar `event.key`se a tecla de seta para cima ou para baixo foi pressionada.

O tamanho atual pode ser mantido em uma ligação, para que você possa basear o novo tamanho nele. Será útil definir uma função que atualize o tamanho - a ligação e o estilo do balão no DOM - para que você possa chamá-lo do seu manipulador de eventos e, possivelmente, também uma vez ao iniciar, para definir o tamanho inicial .

Você pode alterar o balão para uma explosão substituindo o nó de texto por outro (usando `replaceChild`) ou configurando a `textContent`propriedade do nó pai para uma nova sequência.

### Mouse trail

Nos primórdios do JavaScript, que era o momento das home pages berrantes com muitas imagens animadas, as pessoas criaram maneiras verdadeiramente inspiradoras de usar a linguagem.

Uma delas era a *trilha* do *mouse* - uma série de elementos que seguiriam o ponteiro do mouse à medida que você o movia pela página.

Neste exercício, quero que você implemente uma trilha de mouse. Use ``elementos absolutamente posicionados com tamanho fixo e cor de fundo (consulte o [código]https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2015%20-%20Tratamento%20de%20eventos.md#mouse_drawing) na seção "Cliques do mouse" para obter um exemplo). Crie vários desses elementos e, quando o mouse se mover, exiba-os na sequência do ponteiro do mouse.

Existem várias abordagens possíveis aqui. Você pode tornar sua solução tão simples ou complexa quanto desejar. Uma solução simples para começar é manter um número fixo de elementos da trilha e percorrê-los, movendo o próximo para a posição atual do mouse toda vez `"mousemove"`que ocorrer um evento.

```html
<style>
  .trail { /* className for the trail elements */
    position: absolute;
    height: 6px; width: 6px;
    border-radius: 3px;
    background: teal;
  }
  body {
    height: 300px;
  }
</style>

<script>
  // Your code here.
</script>
```

A criação dos elementos é melhor feita com um loop. Anexe-os ao documento para que eles apareçam. Para poder acessá-los posteriormente para alterar sua posição, você deseja armazenar os elementos em uma matriz.

É possível dar um ciclo através deles mantendo uma variável de contador e adicionando 1 a ela toda vez que o `"mousemove"`evento for disparado. O operador restante ( ) pode ser usado para obter um índice de matriz válido para escolher o elemento que você deseja posicionar durante um determinado evento.`% elements.length`

Outro efeito interessante pode ser alcançado através da modelagem de um sistema físico simples. Use o `"mousemove"`evento apenas para atualizar um par de ligações que rastreiam a posição do mouse. Em seguida, use `requestAnimationFrame`para simular os elementos finais que estão sendo atraídos para a posição do ponteiro do mouse. Em cada etapa da animação, atualize sua posição com base na posição relativa ao ponteiro (e, opcionalmente, na velocidade que é armazenada para cada elemento). Descobrir uma boa maneira de fazer isso é com você.

### Guias

Os painéis com guias são amplamente utilizados nas interfaces do usuário. Eles permitem que você selecione um painel de interface, escolhendo entre várias guias "destacadas" acima de um elemento.

Neste exercício, você deve implementar uma interface com guias simples. Escreva uma função `asTabs`,, que pegue um nó DOM e crie uma interface com guias mostrando os elementos filhos desse nó. Ele deve inserir uma lista de ``elementos na parte superior do nó, um para cada elemento filho, contendo o texto recuperado do `data-tabname`atributo do filho. Todos, exceto um dos filhos originais, devem estar ocultos (dado um `display`estilo de `none`). O nó atualmente visível pode ser selecionado clicando nos botões.

Quando isso funcionar, estenda-o para estilizar o botão da guia atualmente selecionada de maneira diferente, para que fique óbvio qual guia está selecionada.

```html
<tab-panel>
  <div data-tabname="one">Tab one</div>
  <div data-tabname="two">Tab two</div>
  <div data-tabname="three">Tab three</div>
</tab-panel>
<script>
  function asTabs(node) {
    // Your code here.
  }
  asTabs(document.querySelector("tab-panel"));
</script>
```

Uma armadilha que você pode encontrar é que não pode usar diretamente a `childNodes`propriedade do nó como uma coleção de nós de guias. Por um lado, quando você adiciona os botões, eles também se tornam nós filhos e acabam nesse objeto porque é uma estrutura de dados ativa. Por outro lado, os nós de texto criados para o espaço em branco entre os nós também estão, `childNodes`mas não devem ter suas próprias guias. Você pode usar em `children`vez de `childNodes`ignorar nós de texto.

Você pode começar criando uma variedade de guias para ter acesso fácil a elas. Para implementar o estilo dos botões, você pode armazenar objetos que contenham o painel da guia e seu botão.

Eu recomendo escrever uma função separada para alterar as guias. Você pode armazenar a guia selecionada anteriormente e alterar apenas os estilos necessários para ocultá-la e mostrar a nova, ou pode simplesmente atualizar o estilo de todas as guias sempre que uma nova guia for selecionada.

Convém chamar essa função imediatamente para tornar a interface inicial com a primeira guia visível.