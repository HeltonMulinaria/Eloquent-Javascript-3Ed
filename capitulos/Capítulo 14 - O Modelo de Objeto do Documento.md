# Capítulo 14 O Modelo de Objeto do Documento

> Que pena! Mesma história de sempre! Depois de terminar de construir sua casa, você percebe que acidentalmente aprendeu algo que realmente deveria saber - antes de começar.
>
> *—Friedrich Nietzsche, além do bem e do mal*

![Imagem de uma árvore com letras e scripts pendurados em seus galhos](https://eloquentjavascript.net/img/chapter_picture_14.jpg)

Quando você abre uma página da web em seu navegador, o navegador recupera o texto HTML da página e o analisa, da mesma forma que nosso analisador do [Capítulo 12](https://eloquentjavascript.net/12_language.html#parsing) analisou os programas. O navegador cria um modelo da estrutura do documento e usa esse modelo para desenhar a página na tela.

Essa representação do documento é um dos brinquedos que um programa JavaScript tem disponível em sua sandbox. É uma estrutura de dados que você pode ler ou modificar. Ele atua como uma estrutura de dados *ativa* : quando é modificada, a página na tela é atualizada para refletir as alterações.

## Estrutura do documento

Você pode imaginar um documento HTML como um conjunto aninhado de caixas. Tags como ``e ``incluem outras tags, que por sua vez contêm outras tags ou texto. Aqui está o documento de exemplo do [capítulo anterior](https://eloquentjavascript.net/13_browser.html) :

```html
<!doctype html>
<html>
  <head>
    <title>My home page</title>
  </head>
  <body>
    <h1>My home page</h1>
    <p>Hello, I am Marijn and this is my home page.</p>
    <p>I also wrote a book! Read it
      <a href="http://eloquentjavascript.net">here</a>.</p>
  </body>
</html>
```

Esta página possui a seguinte estrutura:

![Documento HTML como caixas aninhadas](https://eloquentjavascript.net/img/html-boxes.svg)

A estrutura de dados que o navegador usa para representar o documento segue esta forma. Para cada caixa, há um objeto com o qual podemos interagir para descobrir coisas como qual tag HTML representa e quais caixas e texto ele contém. Essa representação é chamada de *Document Object Model* , ou DOM, para abreviar.

A ligação global `document`nos dá acesso a esses objetos. Sua `documentElement`propriedade refere-se ao objeto que representa a ``tag. Como todo documento HTML tem uma cabeça e um corpo, também possui `head`e `body`propriedades, apontando para esses elementos.

## Árvores

Pense nas árvores de sintaxe do [Capítulo 12](https://eloquentjavascript.net/12_language.html#parsing) por um momento. Suas estruturas são surpreendentemente semelhantes à estrutura do documento de um navegador. Cada *nó* pode se referir a outros nós, *filhos* , que por sua vez podem ter seus próprios filhos. Essa forma é típica de estruturas aninhadas, em que os elementos podem conter subelementos semelhantes a si mesmos.

Chamamos uma estrutura de dados de *árvore* quando ela possui uma estrutura de ramificação, não possui ciclos (um nó pode não se conter, direta ou indiretamente) e possui uma única *raiz* bem definida . No caso do DOM, serve como raiz.`document.documentElement`

As árvores surgem muito na ciência da computação. Além de representar estruturas recursivas, como documentos ou programas HTML, eles são frequentemente usados para manter conjuntos de dados classificados, porque os elementos geralmente podem ser encontrados ou inseridos com mais eficiência em uma árvore do que em uma matriz plana.

Uma árvore típica possui diferentes tipos de nós. A árvore de sintaxe [da linguagem Egg](https://eloquentjavascript.net/12_language.html) tinha identificadores, valores e nós de aplicativos. Nós de aplicativo podem ter filhos, enquanto identificadores e valores são *folhas* ou nós sem filhos.

O mesmo vale para o DOM. Nós para *elementos* , que representam tags HTML, determinam a estrutura do documento. Estes podem ter nós filhos. Um exemplo desse nó é `document.body`. Alguns desses filhos podem ser nós de folha, como partes de texto ou de comentários.

Cada objeto do nó DOM possui uma `nodeType`propriedade, que contém um código (número) que identifica o tipo de nó. Os elementos têm o código 1, que também é definido como a propriedade constante . Nós de texto, representando uma seção de texto no documento, obtêm o código 3 ( ). Os comentários têm o código 8 ( ).`Node.ELEMENT_NODE``Node.TEXT_NODE``Node.COMMENT_NODE`

Outra maneira de visualizar nossa árvore de documentos é a seguinte:

![Documento HTML como uma árvore](https://eloquentjavascript.net/img/html-tree.svg)

As folhas são nós de texto e as setas indicam relacionamentos pai-filho entre nós.

## O padrão

Usar códigos numéricos enigmáticos para representar tipos de nós não é algo muito semelhante ao JavaScript. Mais adiante neste capítulo, veremos que outras partes da interface DOM também parecem pesadas e estranhas. A razão para isso é que o DOM não foi projetado apenas para JavaScript. Em vez disso, ele tenta ser uma interface neutra em linguagem que também pode ser usada em outros sistemas - não apenas para HTML, mas também para XML, que é um formato de dados genérico com uma sintaxe semelhante a HTML.

Isto é um infortúnio. Os padrões são frequentemente úteis. Mas, neste caso, a vantagem (consistência entre idiomas) não é tão atraente. Ter uma interface que esteja adequadamente integrada ao idioma que você está usando poupará mais tempo do que uma interface familiar entre idiomas.

Como um exemplo dessa integração ruim, considere a `childNodes`propriedade que os nós do elemento no DOM possuem. Essa propriedade contém um objeto semelhante a uma matriz, com uma `length`propriedade e propriedades rotuladas por números para acessar os nós filhos. Mas é uma instância do `NodeList`tipo, não uma matriz real, portanto, não possui métodos como `slice`e `map`.

Depois, há questões que são simplesmente um design ruim. Por exemplo, não há como criar um novo nó e adicionar imediatamente filhos ou atributos a ele. Em vez disso, você deve primeiro criá-lo e, em seguida, adicionar os filhos e os atributos um por um, usando efeitos colaterais. O código que interage fortemente com o DOM tende a ficar longo, repetitivo e feio.

Mas essas falhas não são fatais. Como o JavaScript nos permite criar nossas próprias abstrações, é possível criar maneiras aprimoradas de expressar as operações que você está executando. Muitas bibliotecas destinadas à programação de navegadores vêm com essas ferramentas.

## Movendo-se através da árvore

Nós DOM contêm diversos links para outros nós próximos. O diagrama a seguir ilustra estes:

![Links entre nós DOM](https://eloquentjavascript.net/img/html-links.svg)

Embora o diagrama mostre apenas um link de cada tipo, cada nó tem uma `parentNode`propriedade que aponta para o nó do qual faz parte, se houver. Da mesma forma, todo nó do elemento (tipo 1 do nó) possui uma `childNodes`propriedade que aponta para um objeto semelhante a uma matriz que contém seus filhos.

Em teoria, você pode mover-se para qualquer lugar da árvore usando apenas esses links pai e filho. Mas o JavaScript também fornece acesso a vários links de conveniência adicionais. As propriedades `firstChild`e `lastChild`apontam para o primeiro e o último elemento filho ou têm o valor `null`para nós sem filhos. Da mesma forma, `previousSibling`e `nextSibling`aponte para nós adjacentes, que são os nós com o mesmo pai que aparece imediatamente antes ou após o próprio nó. Para um primeiro filho, `previousSibling`será nulo e, para um último filho, `nextSibling`será nulo.

Há também a `children`propriedade, que é como, `childNodes`mas contém apenas filhos do elemento (tipo 1), não outros tipos de nós filhos. Isso pode ser útil quando você não estiver interessado em nós de texto.

Ao lidar com uma estrutura de dados aninhada como esta, funções recursivas geralmente são úteis. A função a seguir verifica um documento em busca de nós de texto que contêm uma determinada string e retorna `true`quando encontra uma:

```js
function talksAbout(node, string) {
  if (node.nodeType == Node.ELEMENT_NODE) {
    for (let i = 0; i < node.childNodes.length; i++) {
      if (talksAbout(node.childNodes[i], string)) {
        return true;
      }
    }
    return false;
  } else if (node.nodeType == Node.TEXT_NODE) {
    return node.nodeValue.indexOf(string) > -1;
  }
}

console.log(talksAbout(document.body, "book"));
// → true
```

Como `childNodes`não é um array real, não podemos fazer um loop sobre ele com `for`/ `of`e precisamos executar o intervalo de índice usando um `for`loop ou uso regular `Array.from`.

A `nodeValue`propriedade de um nó de texto mantém a sequência de texto que ele representa.

## Localizando elementos

Navegar nesses links entre pais, filhos e irmãos geralmente é útil. Porém, se queremos encontrar um nó específico no documento, alcançá-lo iniciando `document.body`e seguindo um caminho fixo de propriedades é uma má idéia. Fazer isso faz suposições em nosso programa sobre a estrutura precisa do documento - uma estrutura que você pode querer alterar mais tarde. Outro fator complicador é que os nós de texto são criados mesmo para o espaço em branco entre os nós. A ``tag do documento de exemplo não possui apenas três filhos ( ``e dois ``elementos), mas na verdade possui sete: esses três, mais os espaços antes, depois e entre eles.

Portanto, se queremos obter o `href`atributo do link nesse documento, não queremos dizer algo como "Obter o segundo filho do sexto filho do corpo do documento". Seria melhor se pudéssemos dizer "Obtenha o primeiro link no documento". E nós podemos.

```js
let link = document.body.getElementsByTagName("a")[0];
console.log(link.href);
```

Todos os nós do elemento têm um `getElementsByTagName`método, que coleta todos os elementos com o nome de marca especificado que são descendentes (filhos diretos ou indiretos) desse nó e os retorna como um objeto semelhante a uma matriz.

Para encontrar um nó *único* específico , você pode atribuir um `id`atributo a ele e usá-lo .`document.getElementById`

```html
<p>My ostrich Gertrude:</p>
<p><img id="gertrude" src="img/ostrich.png"></p>

<script>
  let ostrich = document.getElementById("gertrude");
  console.log(ostrich.src);
</script>
```

Um terceiro método semelhante é o `getElementsByClassName`qual, por exemplo `getElementsByTagName`, pesquisa o conteúdo de um nó de elemento e recupera todos os elementos que possuem a sequência especificada em seu `class`atributo.

## Alterando o documento

Quase tudo sobre a estrutura de dados do DOM pode ser alterado. A forma da árvore de documentos pode ser modificada alterando os relacionamentos pai-filho. Os nós têm um `remove`método para removê-los do nó pai atual. Para adicionar um nó filho a um nó de elemento, podemos usar o `appendChild`que o coloca no final da lista de filhos ou o `insertBefore`que insere o nó fornecido como o primeiro argumento antes do nó fornecido como o segundo argumento.

```html
<p>One</p>
<p>Two</p>
<p>Three</p>

<script>
  let paragraphs = document.body.getElementsByTagName("p");
  document.body.insertBefore(paragraphs[2], paragraphs[0]);
</script>
```

Um nó pode existir no documento em apenas um local. Assim, a inserção do parágrafo *Três* na frente do parágrafo *Um* primeiro o removerá do final do documento e depois o inserirá na frente, resultando em *Três* / *Um* / *Dois* . Todas as operações que inserem um nó em algum lugar, como efeito colateral, fazem com que ele seja removido de sua posição atual (se houver).

O `replaceChild`método é usado para substituir um nó filho por outro. Toma como argumento dois nós: um novo nó e o nó a ser substituído. O nó substituído deve ser filho do elemento em que o método é chamado. Observe que ambos `replaceChild`e `insertBefore`esperam o *novo* nó como seu primeiro argumento.

## Criando nós

Digamos que queremos escrever um script que substitua todas as imagens ( ``tags) no documento pelo texto contido em seus `alt`atributos, o que especifica uma representação textual alternativa da imagem.

Isso envolve não apenas remover as imagens, mas adicionar um novo nó de texto para substituí-las. Nós de texto são criados com o método`document.createTextNode`

```html
<p>The <img src="img/cat.png" alt="Cat"> in the
  <img src="img/hat.png" alt="Hat">.</p>

<p><button onclick="replaceImages()">Replace</button></p>

<script>
  function replaceImages() {
    let images = document.body.getElementsByTagName("img");
    for (let i = images.length - 1; i >= 0; i--) {
      let image = images[i];
      if (image.alt) {
        let text = document.createTextNode(image.alt);
        image.parentNode.replaceChild(text, image);
      }
    }
  }
</script>
```

Dada uma string, `createTextNode`nos fornece um nó de texto que podemos inserir no documento para que ele apareça na tela.

O loop que percorre as imagens começa no final da lista. Isso é necessário porque a lista de nós retornada por um método como `getElementsByTagName`(ou uma propriedade como `childNodes`) está *ativa* . Ou seja, é atualizado à medida que o documento é alterado. Se começássemos de frente, remover a primeira imagem faria com que a lista perdesse seu primeiro elemento, de modo que na segunda vez que o loop se repete, onde `i`é 1, ele pararia porque o comprimento da coleção agora também é 1.

Se você deseja uma coleção *sólida* de nós, em oposição a um nó ativo, poderá converter a coleção em uma matriz real chamando `Array.from`.

```js
let arrayish = {0: "one", 1: "two", length: 2};
let array = Array.from(arrayish);
console.log(array.map(s => s.toUpperCase()));
// → ["ONE", "TWO"]
```

Para criar nós de elemento, você pode usar o método Este método usa um nome de marca e retorna um novo nó vazio do tipo especificado.`document.createElement`

O exemplo a seguir define um utilitário `elt`, que cria um nó de elemento e trata o restante de seus argumentos como filhos desse nó. Essa função é usada para adicionar uma atribuição a uma cotação.

```html
<blockquote id="quote">
  No book can ever be finished. While working on it we learn
  just enough to find it immature the moment we turn away
  from it.
</blockquote>

<script>
  function elt(type, ...children) {
    let node = document.createElement(type);
    for (let child of children) {
      if (typeof child != "string") node.appendChild(child);
      else node.appendChild(document.createTextNode(child));
    }
    return node;
  }

  document.getElementById("quote").appendChild(
    elt("footer", "—",
        elt("strong", "Karl Popper"),
        ", preface to the second edition of ",
        elt("em", "The Open Society and Its Enemies"),
        ", 1950"));
</script>
```

## Atributos

Alguns atributos do elemento, como `href`links, podem ser acessados por meio de uma propriedade com o mesmo nome no objeto DOM do elemento. Este é o caso dos atributos padrão mais usados.

Mas o HTML permite definir qualquer atributo desejado nos nós. Isso pode ser útil porque permite armazenar informações extras em um documento. Se você criar seus próprios nomes de atributo, esses atributos não estarão presentes como propriedades no nó do elemento. Em vez disso, você deve usar os métodos `getAttribute`e `setAttribute`para trabalhar com eles.

```html
<p data-classified="secret">The launch code is 00000000.</p>
<p data-classified="unclassified">I have two feet.</p>

<script>
  let paras = document.body.getElementsByTagName("p");
  for (let para of Array.from(paras)) {
    if (para.getAttribute("data-classified") == "secret") {
      para.remove();
    }
  }
</script>
```

Recomenda-se prefixar os nomes desses atributos criados `data-`para garantir que eles não entrem em conflito com outros atributos.

Existe um atributo comumente usado `class`, que é uma palavra-chave na linguagem JavaScript. Por razões históricas - algumas implementações antigas de JavaScript não conseguiam lidar com nomes de propriedades que correspondiam a palavras-chave - a propriedade usada para acessar esse atributo é chamada `className`. Você também pode acessá-lo com seu nome real `"class"`, usando os métodos `getAttribute`e `setAttribute`.

## Layout

Você deve ter notado que diferentes tipos de elementos são dispostos de maneira diferente. Alguns, como parágrafos ( ``) ou títulos ( ``), ocupam toda a largura do documento e são renderizados em linhas separadas. Estes são chamados de elementos de *bloco* . Outros, como links ( ``) ou o ``elemento, são renderizados na mesma linha com o texto ao redor. Esses elementos são chamados de elementos *embutidos* .

Para qualquer documento, os navegadores podem calcular um layout, que fornece a cada elemento um tamanho e uma posição com base em seu tipo e conteúdo. Esse layout é usado para desenhar o documento.

O tamanho e a posição de um elemento podem ser acessados no JavaScript. As propriedades `offsetWidth`e `offsetHeight`fornecem o espaço que o elemento ocupa em *pixels* . Um pixel é a unidade básica de medida no navegador. Tradicionalmente, corresponde ao menor ponto que a tela pode desenhar, mas em telas modernas, que podem desenhar pontos *muito* pequenos, que podem não ser mais o caso, e um pixel do navegador pode abranger vários pontos de exibição.

Da mesma forma, `clientWidth`e `clientHeight`forneça o tamanho do espaço *dentro* do elemento, ignorando a largura da borda.

```html
<p style="border: 3px solid red">
  I'm boxed in
</p>

<script>
  let para = document.body.getElementsByTagName("p")[0];
  console.log("clientHeight:", para.clientHeight);
  console.log("offsetHeight:", para.offsetHeight);
</script>
```

A maneira mais eficaz de encontrar a posição precisa de um elemento na tela é o `getBoundingClientRect`método. Ele retorna um objecto com `top`, `bottom`, `left`, e `right`propriedades, indicando as posições de pixel dos lados do elemento em relação à parte superior esquerda da tela. Se você os quiser em relação ao documento inteiro, adicione a posição de rolagem atual, que pode ser encontrada nas encadernações `pageXOffset`e `pageYOffset`.

A disposição de um documento pode ser bastante trabalhosa. No interesse da velocidade, os mecanismos do navegador não reposicionam imediatamente um documento toda vez que você o altera, mas esperam o máximo que puderem. Quando um programa JavaScript que alterou o documento terminar a execução, o navegador precisará calcular um novo layout para desenhar o documento alterado na tela. Quando um programa *solicita* a posição ou o tamanho de algo lendo propriedades como `offsetHeight`ou chamando `getBoundingClientRect`, fornecer informações corretas também requer o cálculo de um layout.

Um programa que alterna repetidamente entre ler as informações de layout do DOM e alterar o DOM força muitos cálculos de layout a acontecer e, consequentemente, é executado muito lentamente. O código a seguir é um exemplo disso. Ele contém dois programas diferentes que constroem uma linha de caracteres *X com* 2.000 pixels de largura e medem o tempo que cada um leva.

```html
<p><span id="one"></span></p>
<p><span id="two"></span></p>

<script>
  function time(name, action) {
    let start = Date.now(); // Current time in milliseconds
    action();
    console.log(name, "took", Date.now() - start, "ms");
  }

  time("naive", () => {
    let target = document.getElementById("one");
    while (target.offsetWidth < 2000) {
      target.appendChild(document.createTextNode("X"));
    }
  });
  // → naive took 32 ms

  time("clever", function() {
    let target = document.getElementById("two");
    target.appendChild(document.createTextNode("XXXXX"));
    let total = Math.ceil(2000 / (target.offsetWidth / 5));
    target.firstChild.nodeValue = "X".repeat(total);
  });
  // → clever took 1 ms
</script>
```

## Estilizando

Vimos que diferentes elementos HTML são desenhados de maneira diferente. Alguns são exibidos como blocos, outros em linha. Alguns adicionam estilos - ``tornam seu conteúdo em negrito, ``o azul e o sublinha.

A maneira como uma ``tag mostra uma imagem ou ``tag faz com que um link seja seguido quando clicado é fortemente vinculado ao tipo de elemento. Mas podemos alterar o estilo associado a um elemento, como a cor ou o sublinhado do texto. Aqui está um exemplo que usa a `style`propriedade:

```html
<p><a href=".">Normal link</a></p>
<p><a href="." style="color: green">Green link</a></p>
```

Um atributo de estilo pode conter uma ou mais *declarações* , que são uma propriedade (como `color`) seguida de dois pontos e um valor (como `green`). Quando existe mais do que uma declaração, eles devem ser separados por ponto e vírgula, como em `"color: red; border: none"`.

Muitos aspectos do documento podem ser influenciados pelo estilo. Por exemplo, a `display`propriedade controla se um elemento é exibido como um bloco ou um elemento embutido.

```html
This text is displayed <strong>inline</strong>,
<strong style="display: block">as a block</strong>, and
<strong style="display: none">not at all</strong>.
```

A `block`tag terminará em sua própria linha, pois os elementos do bloco não serão exibidos alinhados com o texto ao seu redor. A última tag não é exibida - `display: none`impede que um elemento seja exibido na tela. Esta é uma maneira de ocultar elementos. Geralmente, é preferível removê-los totalmente do documento, pois facilita a revelação novamente mais tarde.

O código JavaScript pode manipular diretamente o estilo de um elemento através da `style`propriedade do elemento . Esta propriedade mantém um objeto que possui propriedades para todas as propriedades de estilo possíveis. Os valores dessas propriedades são strings, nas quais podemos escrever para alterar um aspecto específico do estilo do elemento.

```html
<p id="para" style="color: purple">
  Nice text
</p>

<script>
  let para = document.getElementById("para");
  console.log(para.style.color);
  para.style.color = "magenta";
</script>
```

Alguns nomes de propriedades de estilo contêm hífens, como `font-family`. Como esses nomes de propriedades são difíceis de trabalhar em JavaScript (você diria `style["font-family"]`), os nomes de propriedades no `style`objeto para essas propriedades têm seus hífens removidos e as letras depois delas são maiúsculas ( `style.fontFamily`).

## Estilos em cascata

O sistema de estilo para HTML é chamado CSS, para *Cascading Style Sheets* . Uma *folha de estilos* é um conjunto de regras sobre como estilizar elementos em um documento. Pode ser fornecido dentro de uma ``tag.

```html
<style>
  strong {
    font-style: italic;
    color: gray;
  }
</style>
<p>Now <strong>strong text</strong> is italic and gray.</p>
```

A *cascata* no nome refere-se ao fato de que várias dessas regras são combinadas para produzir o estilo final de um elemento. No exemplo, o estilo padrão das ``tags, que as fornece `font-weight: bold`, é sobreposto pela regra na ``tag, que adiciona `font-style`e `color`.

Quando várias regras definem um valor para a mesma propriedade, a regra de leitura mais recente obtém uma precedência mais alta e vence. Portanto, se a regra na ``tag fosse incluída `font-weight: normal`, contradizendo a `font-weight`regra padrão , o texto seria normal, *não em* negrito. Os estilos em um `style`atributo aplicado diretamente ao nó têm a maior precedência e sempre vencem.

É possível segmentar outras coisas além dos nomes das tags nas regras CSS. Uma regra para `.abc`aplica-se a todos os elementos com `"abc"`em seus `class`atributos. Uma regra para `#xyz`aplica-se ao elemento com um `id`atributo de `"xyz"`(que deve ser exclusivo no documento).

```css
.subtle {
  color: gray;
  font-size: 80%;
}
#header {
  background: blue;
  color: white;
}
/* p elements with id main and with classes a and b */
p#main.a.b {
  margin-bottom: 20px;
}
```

A regra de precedência que favorece a regra definida mais recentemente se aplica somente quando as regras têm a mesma *especificidade* . A especificidade de uma regra é uma medida de como ela descreve com precisão os elementos correspondentes, determinados pelo número e tipo (tag, classe ou ID) dos aspectos dos elementos necessários. Por exemplo, uma regra que `p.a`é segmentada é mais específica do que regras que são direcionadas `p`ou justas `.a`e, portanto, têm precedência sobre elas.

A notação `p > a {…}`aplica os estilos fornecidos a todas as ``tags que são filhos diretos das ``tags. Da mesma forma, `p a {…}`aplica-se a todas as ``tags dentro das ``tags, sejam filhos diretos ou indiretos.

## Query selectors

Não usaremos muito as folhas de estilo neste livro. Compreendê-los é útil ao programar no navegador, mas eles são complicados o suficiente para justificar um livro separado.

A principal razão pela qual introduzi a sintaxe do *seletor* - a notação usada nas folhas de estilo para determinar a quais elementos um conjunto de estilos se aplica - é que podemos usar esse mesmo mini-idioma como uma maneira eficaz de encontrar elementos DOM.

O `querySelectorAll`método, definido no `document`objeto e nos nós do elemento, pega uma sequência de seleção e retorna a `NodeList`contendo todos os elementos correspondentes.

```html
<p>And if you go chasing
  <span class="animal">rabbits</span></p>
<p>And you know you're going to fall</p>
<p>Tell 'em a <span class="character">hookah smoking
  <span class="animal">caterpillar</span></span></p>
<p>Has given you the call</p>

<script>
  function count(selector) {
    return document.querySelectorAll(selector).length;
  }
  console.log(count("p"));           // All <p> elements
  // → 4
  console.log(count(".animal"));     // Class animal
  // → 2
  console.log(count("p .animal"));   // Animal inside of <p>
  // → 2
  console.log(count("p > .animal")); // Direct child of <p>
  // → 1
</script>
```

Ao contrário de métodos como `getElementsByTagName`, o objeto retornado por *não*`querySelectorAll` é ativo. Não muda quando você altera o documento. No entanto, ainda não é uma matriz real, então você ainda precisa ligar se quiser tratá-la como uma.`Array.from`

O `querySelector`método (sem a `All`parte) funciona de maneira semelhante. Este é útil se você deseja um elemento único e específico. Ele retornará apenas o primeiro elemento correspondente ou nulo quando nenhum elemento corresponder.

## Posicionamento e animação

A `position`propriedade style influencia o layout de maneira poderosa. Por padrão, ele tem um valor de `static`, o que significa que o elemento fica em seu lugar normal no documento. Quando definido como `relative`, o elemento ainda ocupa espaço no documento, mas agora as propriedades `top`e `left`style podem ser usadas para movê-lo em relação àquele local normal. Quando `position`definido como `absolute`, o elemento é removido do fluxo normal de documentos, ou seja, não ocupa mais espaço e pode se sobrepor a outros elementos. Além disso, suas propriedades `top`e `left`podem ser usadas para posicioná-lo absolutamente em relação ao canto superior esquerdo do elemento envolvente mais próximo, cuja `position`propriedade não é `static`, ou em relação ao documento, se esse elemento envolvente não existir.

Podemos usar isso para criar uma animação. O documento a seguir exibe uma imagem de um gato que se move em uma elipse:

```html
<p style="text-align: center">
  <img src="img/cat.png" style="position: relative">
</p>
<script>
  let cat = document.querySelector("img");
  let angle = Math.PI / 2;
  function animate(time, lastTime) {
    if (lastTime != null) {
      angle += (time - lastTime) * 0.001;
    }
    cat.style.top = (Math.sin(angle) * 20) + "px";
    cat.style.left = (Math.cos(angle) * 200) + "px";
    requestAnimationFrame(newTime => animate(newTime, time));
  }
  requestAnimationFrame(animate);
</script>
```

Nossa imagem é centralizada na página e recebe um `position`de `relative`. Atualizaremos repetidamente a imagem `top`e os `left`estilos para movê-la.

O script usa `requestAnimationFrame`para agendar a `animate`execução da função sempre que o navegador estiver pronto para redesenhar a tela. A `animate`própria função novamente chama `requestAnimationFrame`para agendar a próxima atualização. Quando a janela (ou guia) do navegador estiver ativa, isso fará com que as atualizações aconteçam a uma taxa de cerca de 60 por segundo, o que tende a produzir uma animação bonita.

Se apenas atualizássemos o DOM em um loop, a página congelaria e nada seria exibido na tela. Os navegadores não atualizam sua exibição enquanto um programa JavaScript está em execução, nem permitem qualquer interação com a página. É por isso que precisamos `requestAnimationFrame`- ele permite que o navegador saiba que já terminamos, e ele pode seguir em frente e fazer as coisas que os navegadores fazem, como atualizar a tela e responder às ações do usuário.

A função de animação é passada no horário atual como argumento. Para garantir que o movimento do gato por milissegundo seja estável, ele baseia a velocidade na qual o ângulo muda na diferença entre o tempo atual e a última vez que a função foi executada. Se apenas movesse o ângulo em um valor fixo por etapa, o movimento seria interrompido se, por exemplo, outra tarefa pesada executada no mesmo computador fosse impedir que a função funcionasse por uma fração de segundo.

A movimentação em círculos é feita usando as funções trigonométricas `Math.cos`e `Math.sin`. Para aqueles que não estão familiarizados com isso, vou apresentá-los brevemente, pois ocasionalmente os usaremos neste livro.

`Math.cos`e `Math.sin`são úteis para encontrar pontos que se encontram em um círculo ao redor do ponto (0,0) com um raio de um. Ambas as funções interpretam seu argumento como a posição nesse círculo, com zero denotando o ponto à direita do círculo, indo no sentido horário até 2π (cerca de 6,28) nos levar ao redor de todo o círculo. `Math.cos`informa a coordenada x do ponto que corresponde à posição especificada e `Math.sin`produz a coordenada y. Posições (ou ângulos) maiores do que 2π ou menos do que 0 são válidas as repeties-rotação, de forma que *um* + 2π refere-se ao mesmo ângulo que *um* .

Essa unidade para medir ângulos é chamada radianos - um círculo completo é 2π radianos, semelhante a 360 graus ao medir em graus. A constante π está disponível como `Math.PI`no JavaScript.

![Usando cosseno e seno para calcular coordenadas](https://eloquentjavascript.net/img/cos_sin.svg)

O código de animação cat mantém um contador, `angle`para o ângulo atual da animação e o incrementa toda vez que a `animate`função é chamada. Em seguida, ele pode usar esse ângulo para calcular a posição atual do elemento de imagem. O `top`estilo é calculado `Math.sin`e multiplicado por 20, que é o raio vertical da nossa elipse. O `left`estilo é baseado `Math.cos`e multiplicado por 200 para que a elipse seja muito mais larga do que alta.

Observe que os estilos geralmente precisam de *unidades* . Nesse caso, precisamos acrescentar `"px"`ao número para informar ao navegador que estamos contando em pixels (ao contrário de centímetros, "ems" ou outras unidades). Isso é fácil de esquecer. O uso de números sem unidades fará com que seu estilo seja ignorado - a menos que o número seja 0, o que sempre significa a mesma coisa, independentemente da unidade.

## Sumário

Os programas JavaScript podem inspecionar e interferir no documento que o navegador está exibindo por meio de uma estrutura de dados chamada DOM. Essa estrutura de dados representa o modelo do navegador do documento e um programa JavaScript pode modificá-lo para alterar o documento visível.

O DOM é organizado como uma árvore, na qual os elementos são organizados hierarquicamente de acordo com a estrutura do documento. Os objetos que representam elementos têm propriedades como `parentNode`e `childNodes`, que podem ser usadas para navegar por essa árvore.

A maneira como um documento é exibido pode ser influenciada pelo *estilo* , anexando estilos diretamente aos nós e definindo regras que correspondem a determinados nós. Existem muitas propriedades de estilo diferentes, como `color`ou `display`. O código JavaScript pode manipular o estilo de um elemento diretamente através de sua `style`propriedade

## Exercícios

### Construa uma tabela

Uma tabela HTML é criada com a seguinte estrutura de tags:

```html
<table>
  <tr>
    <th>name</th>
    <th>height</th>
    <th>place</th>
  </tr>
  <tr>
    <td>Kilimanjaro</td>
    <td>5895</td>
    <td>Tanzania</td>
  </tr>
</table>
```

Para cada *linha* , a ``tag contém uma tag. Dentro dessas tags, podemos colocar elementos de célula: células de cabeçalho ( `) ou células regulares ( `).

Dado um conjunto de dados de montanhas, um conjunto de objectos com `name`, `height`e `place`propriedades, gerar a estrutura de DOM para uma tabela que enumera os objetos. Ele deve ter uma coluna por chave e uma linha por objeto, além de uma linha de cabeçalho com `elementos na parte superior, listando os nomes das colunas.

Escreva isso para que as colunas sejam derivadas automaticamente dos objetos, levando os nomes de propriedades do primeiro objeto nos dados.

Adicione a tabela resultante ao elemento com um `id`atributo de, `"mountains"`para que fique visível no documento.

Depois de fazer isso funcionar, alinhe à direita as células que contêm valores numéricos, definindo sua `style.textAlign`propriedade como `"right"`.

```html
<h1>Mountains</h1>

<div id="mountains"></div>

<script>
  const MOUNTAINS = [
    {name: "Kilimanjaro", height: 5895, place: "Tanzania"},
    {name: "Everest", height: 8848, place: "Nepal"},
    {name: "Mount Fuji", height: 3776, place: "Japan"},
    {name: "Vaalserberg", height: 323, place: "Netherlands"},
    {name: "Denali", height: 6168, place: "United States"},
    {name: "Popocatepetl", height: 5465, place: "Mexico"},
    {name: "Mont Blanc", height: 4808, place: "Italy/France"}
  ];

  // Your code here
</script>
```

Você pode usar para criar novos nós de elemento, criar nós de texto e o método para colocar nós em outros nós.`document.createElement``document.createTextNode``appendChild`

Você desejará repetir os nomes das chaves uma vez para preencher a linha superior e depois novamente para cada objeto na matriz para construir as linhas de dados. Para obter uma matriz de nomes de chave do primeiro objeto, `Object.keys`será útil.

Para adicionar a tabela ao nó pai correto, você pode usar ou para encontrar o nó com o atributo apropriado .`document.getElementById``document.querySelector``id`

### Elementos por nome da tag

O método retorna todos os elementos filho com um nome de tag específico. Implemente sua própria versão disso como uma função que utiliza um nó e uma string (o nome da tag) como argumentos e retorna uma matriz contendo todos os nós do elemento descendente com o nome da tag fornecida.`document.getElementsByTagName`

Para encontrar o nome da marca de um elemento, use sua `nodeName`propriedade Mas observe que isso retornará o nome da tag em maiúsculas. Use os métodos `toLowerCase`ou `toUpperCase`string para compensar isso.

```html
<h1>Heading with a <span>span</span> element.</h1>
<p>A paragraph with <span>one</span>, <span>two</span>
  spans.</p>

<script>
  function byTagName(node, tagName) {
    // Your code here.
  }

  console.log(byTagName(document.body, "h1").length);
  // → 1
  console.log(byTagName(document.body, "span").length);
  // → 3
  let para = document.querySelector("p");
  console.log(byTagName(para, "span").length);
  // → 2
</script>
```

A solução é mais facilmente expressa com uma função recursiva, semelhante à [`talksAbout`função](https://eloquentjavascript.net/14_dom.html#talksAbout) definida anteriormente neste capítulo.

Você poderia se chamar `byTagname`recursivamente, concatenando as matrizes resultantes para produzir a saída. Ou você pode criar uma função interna que se chama recursivamente e que tenha acesso a uma ligação de matriz definida na função externa, à qual pode adicionar os elementos correspondentes que encontrar. Não se esqueça de chamar a função interna uma vez a partir da função externa para iniciar o processo.

A função recursiva deve verificar o tipo de nó. Aqui, estamos interessados apenas no nó tipo 1 ( ). Para esses nós, precisamos fazer um loop sobre seus filhos e, para cada filho, ver se o filho corresponde à consulta enquanto também faz uma chamada recursiva para inspecionar seus próprios filhos.`Node.ELEMENT_NODE`

### O chapéu do gato

Estenda a animação do gato definida [anteriormente](https://eloquentjavascript.net/14_dom.html#animation) para que o gato e seu chapéu ( ) orbitam em lados opostos da elipse.``

Ou faça o chapéu circular em torno do gato. Ou altere a animação de alguma outra maneira interessante.

Para facilitar o posicionamento de vários objetos, provavelmente é uma boa ideia mudar para o posicionamento absoluto. Isso significa que `top`e `left`são contados em relação à parte superior esquerda do documento. Para evitar o uso de coordenadas negativas, o que faria com que a imagem se movesse para fora da página visível, você pode adicionar um número fixo de pixels aos valores da posição.

```html
<style>body { min-height: 200px }</style>
<img src="img/cat.png" id="cat" style="position: absolute">
<img src="img/hat.png" id="hat" style="position: absolute">

<script>
  let cat = document.querySelector("#cat");
  let hat = document.querySelector("#hat");

  let angle = 0;
  let lastTime = null;
  function animate(time) {
    if (lastTime != null) angle += (time - lastTime) * 0.001;
    lastTime = time;
    cat.style.top = (Math.sin(angle) * 40 + 40) + "px";
    cat.style.left = (Math.cos(angle) * 200 + 230) + "px";

    // Your extensions here.

    requestAnimationFrame(animate);
  }
  requestAnimationFrame(animate);
</script>
```

`Math.cos`e `Math.sin`meça os ângulos em radianos, onde um círculo completo é 2π. Para um determinado ângulo, você pode obter o ângulo oposto adicionando metade disso, que é `Math.PI`. Isso pode ser útil para colocar o chapéu no lado oposto da órbita.