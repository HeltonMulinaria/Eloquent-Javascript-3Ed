# Capítulo 3 Funções

> As pessoas pensam que a ciência da computação é a arte dos gênios, mas a realidade real é o oposto, apenas muitas pessoas fazendo coisas que se constroem umas sobre as outras, como uma parede de mini pedras.
>
> *—Donald Knuth*

![Imagem de folhas de samambaia com forma de fractal](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/chapter_picture_3.jpg)

As funções são o pão com manteiga da programação JavaScript. O conceito de agrupar um pedaço de programa em um valor tem muitos usos. Ele nos fornece uma maneira de estruturar programas maiores, reduzir a repetição, associar nomes a subprogramas e isolar esses subprogramas um do outro.

A aplicação mais óbvia de funções é definir novo vocabulário. Criar novas palavras em prosa geralmente é de mau estilo. Mas na programação, é indispensável.

Os falantes adultos típicos de inglês têm cerca de 20.000 palavras em seu vocabulário. Poucas linguagens de programação vêm com 20.000 comandos internos. E o vocabulário que *é* disponível tende a ser mais precisamente definido e, portanto, menos flexível, que na linguagem humana. Portanto, geralmente *temos* que introduzir novos conceitos para evitar repetir-nos demais.

## Definindo uma função

Uma definição de função é uma ligação regular em que o valor da ligação é uma função. Por exemplo, este código define `square`para se referir a uma função que produz o quadrado de um determinado número:

```js
const square = function(x) {
  return x * x;
};

console.log(square(12));
// → 144
```

Uma função é criada com uma expressão que começa com a palavra-chave `function`. As funções têm um conjunto de *parâmetros* (somente neste caso `x`) e um *corpo* , que contém as instruções que devem ser executadas quando a função é chamada. O corpo da função criada dessa maneira sempre deve ser colocado entre chaves, mesmo quando consiste em apenas uma única instrução.

Uma função pode ter vários parâmetros ou nenhum parâmetro. No exemplo a seguir, `makeNoise`não lista nenhum nome de parâmetro, enquanto `power`lista dois:

```js
const makeNoise = function() {
  console.log("Pling!");
};

makeNoise();
// → Pling!

const power = function(base, exponent) {
  let result = 1;
  for (let count = 0; count < exponent; count++) {
    result *= base;
  }
  return result;
};

console.log(power(2, 10));
// → 1024
```

Algumas funções produzem um valor, como `power`e `square`, e outras não, como `makeNoise`, cujo único resultado é um efeito colateral. Uma `return`instrução determina o valor que a função retorna. Quando o controle se depara com essa instrução, ele sai imediatamente da função atual e dá o valor retornado ao código que chamou a função. Uma `return`palavra-chave sem uma expressão depois fará com que a função retorne `undefined`. Funções que não possuem uma `return`declaração, como, por exemplo `makeNoise`, retornam `undefined`.

Os parâmetros de uma função se comportam como ligações regulares, mas seus valores iniciais são dados pelo *responsável* pela *chamada* da função, não pelo código da própria função.

## Ligações e escopos

Cada ligação tem um *escopo* , que é a parte do programa em que a ligação é visível. Para ligações definidas fora de qualquer função ou bloco, o escopo é o programa inteiro - você pode consultar essas ligações onde quiser. Estes são chamados *globais* .

Mas ligações criadas para parâmetros de função ou declaradas dentro de uma função podem ser referenciadas apenas nessa função, portanto, são conhecidas como ligações *locais* . Sempre que a função é chamada, novas instâncias dessas ligações são criadas. Isso fornece algum isolamento entre funções - cada chamada de função atua em seu próprio mundinho (seu ambiente local) e pode ser entendida muitas vezes sem se saber muito sobre o que está acontecendo no ambiente global.

As ligações declaradas com `let`e `const`são de fato locais para o *bloco em* que estão declaradas; portanto, se você criar uma dessas dentro de um loop, o código antes e depois do loop não poderá "vê-lo". No JavaScript anterior a 2015, apenas as funções criavam novos escopos; portanto, as associações de estilo antigo, criadas com a `var`palavra - chave, são visíveis em toda a função em que aparecem - ou em todo o escopo global, se não estiverem em uma função.

```js
let x = 10;
if (true) {
  let y = 20;
  var z = 30;
  console.log(x + y + z);
  // → 60
}
// y is not visible here
console.log(x + z);
// → 40
```

Cada escopo pode "olhar para fora" no escopo ao seu redor, portanto `x`é visível dentro do bloco no exemplo. A exceção é quando várias ligações têm o mesmo nome - nesse caso, o código pode ver apenas a mais interna. Por exemplo, quando o código dentro da `halve`função se refere `n`, ele está vendo o seu *próprio* `n` , não o global `n`.

```js
const halve = function(n) {
  return n / 2;
};

let n = 10;
console.log(halve(100));
// → 50
console.log(n);
// → 10
```

### Escopo aninhado

JavaScript distingue não apenas ligações *globais* e *locais* . Blocos e funções podem ser criados dentro de outros blocos e funções, produzindo vários graus de localidade.

Por exemplo, essa função - que gera os ingredientes necessários para fazer um lote de hummus - tem outra função:

```js
const hummus = function(factor) {
  const ingredient = function(amount, unit, name) {
    let ingredientAmount = amount * factor;
    if (ingredientAmount > 1) {
      unit += "s";
    }
    console.log(`${ingredientAmount} ${unit} ${name}`);
  };
  ingredient(1, "can", "chickpeas");
  ingredient(0.25, "cup", "tahini");
  ingredient(0.25, "cup", "lemon juice");
  ingredient(1, "clove", "garlic");
  ingredient(2, "tablespoon", "olive oil");
  ingredient(0.5, "teaspoon", "cumin");
};
```

O código dentro da `ingredient`função pode ver a `factor`ligação da função externa. Mas suas ligações locais, como `unit`ou `ingredientAmount`, não são visíveis na função externa.

O conjunto de ligações visíveis dentro de um bloco é determinado pelo local desse bloco no texto do programa. Cada escopo local também pode ver todos os escopos locais que o contêm e todos os escopos podem ver o escopo global. Essa abordagem da visibilidade de ligação é chamada de *escopo lexical* .

## Funções como valores

Uma ligação de função geralmente simplesmente atua como um nome para uma parte específica do programa. Essa ligação é definida uma vez e nunca é alterada. Isso facilita a confusão da função e seu nome.

Mas os dois são diferentes. Um valor de função pode fazer tudo o que outros valores podem fazer - você pode usá-lo em expressões arbitrárias, e não apenas chamá-lo. É possível armazenar um valor de função em uma nova ligação, passá-lo como argumento para uma função e assim por diante. Da mesma forma, uma ligação que mantém uma função ainda é apenas uma ligação regular e pode, se não for constante, receber um novo valor, da seguinte forma:

```js
let launchMissiles = function() {
  missileSystem.launch("now");
};
if (safeMode) {
  launchMissiles = function() {/* do nothing */};
}
```

No [Capítulo 5](https://eloquentjavascript.net/05_higher_order.html) , discutiremos as coisas interessantes que podem ser feitas passando valores de função para outras funções.

## Notação de declaração

Existe uma maneira um pouco mais curta de criar uma ligação de função. Quando a `function`palavra-chave é usada no início de uma instrução, ela funciona de maneira diferente.

```js
 function square(x) {
  return x * x;
}
```

Esta é uma *declaração de* função . A instrução define a ligação `square`e aponta para a função especificada. É um pouco mais fácil escrever e não requer um ponto e vírgula após a função.

Há uma sutileza nessa forma de definição de função.

```js
console.log("The future says:", future());

function future() {
  return "You'll never have flying cars";
}
```

O código anterior funciona, mesmo que a função esteja definida *abaixo* do código que a utiliza. As declarações de função não fazem parte do fluxo regular de cima para baixo do controle. Eles são movidos conceitualmente para o topo de seu escopo e podem ser usados por todo o código nesse escopo. Às vezes, isso é útil porque oferece a liberdade de solicitar código de uma maneira que pareça significativa, sem se preocupar em definir todas as funções antes de serem usadas.

## Arrow Functions

Há uma terceira notação para funções, que parece muito diferente das outras. Em vez da `function`palavra-chave, ela usa uma seta ( `=>`) composta por um sinal de igual e um caractere maior que (não deve ser confundido com o operador maior que ou igual, que está escrito `>=`).

```js
const power = (base, exponent) => {
  let result = 1;
  for (let count = 0; count < exponent; count++) {
    result *= base;
  }
  return result;
};
```

A seta vem *após* a lista de parâmetros e é seguida pelo corpo da função. Expressa algo como "essa entrada (os parâmetros) produz esse resultado (o corpo)".

Quando existe apenas um nome de parâmetro, você pode omitir os parênteses ao redor da lista de parâmetros. Se o corpo for uma expressão única, em vez de um bloco entre chaves, essa expressão será retornada da função. Então, essas duas definições de `square`fazem a mesma coisa:

```js
const square1 = (x) => { return x * x; };
const square2 = x => x * x;
```

Quando uma função de seta não possui parâmetros, sua lista de parâmetros é apenas um conjunto vazio de parênteses.

```js
const horn = () => {
  console.log("Toot");
};
```

Não há razão profunda para ter funções de seta e `function`expressões no idioma. Além de um pequeno detalhe, que discutiremos no [Capítulo 6](https://eloquentjavascript.net/06_object.html) , eles fazem a mesma coisa. As funções de seta foram adicionadas em 2015, principalmente para permitir escrever pequenas expressões de função de uma maneira menos detalhada. Vamos usá-los bastante no [capítulo 5](https://eloquentjavascript.net/05_higher_order.html) .

## A pilha de chamadas

O modo como o controle flui através das funções está um pouco envolvido. Vamos dar uma olhada mais de perto. Aqui está um programa simples que faz algumas chamadas de função:

```js
function greet(who) {
  console.log("Hello " + who);
}
greet("Harry");
console.log("Bye");
```

Uma execução nesse programa é mais ou menos assim: a chamada para `greet`faz com que o controle pule para o início dessa função (linha 2). A função chama `console.log`, que assume o controle, faz seu trabalho e, em seguida, retorna o controle para a linha 2. Aí atinge o final da `greet`função, retornando ao local que a chamou, que é a linha 4. A linha depois chama `console.log`novamente . Após o retorno, o programa chega ao fim.

Poderíamos mostrar o fluxo de controle esquematicamente assim:

```js
not in function
   in greet
        in console.log
   in greet
not in function
   in console.log
not in function
```

Como uma função precisa retornar ao local que a chamou quando retorna, o computador deve se lembrar do contexto em que a chamada ocorreu. Em um caso, `console.log`deve retornar à `greet`função quando estiver concluída. No outro caso, ele retorna ao final do programa.

O local em que o computador armazena esse contexto é a *pilha de chamadas* . Toda vez que uma função é chamada, o contexto atual é armazenado no topo dessa pilha. Quando uma função retorna, ela remove o contexto superior da pilha e usa esse contexto para continuar a execução.

Armazenar essa pilha requer espaço na memória do computador. Quando a pilha cresce muito, o computador falha com uma mensagem como "espaço sem pilha" ou "muita recursão". O código a seguir ilustra isso, fazendo ao computador uma pergunta realmente difícil que causa um infinito retorno entre duas funções. Em vez disso, *seria* infinito se o computador tivesse uma pilha infinita. Como está, ficaremos sem espaço ou "explodiremos a pilha".

```js
function chicken() {
  return egg();
}
function egg() {
  return chicken();
}
console.log(chicken() + " came first.");
// → ??
```

## Argumentos opcionais

O código a seguir é permitido e é executado sem nenhum problema:

```js
function square(x) { return x * x; }
console.log(square(4, true, "hedgehog"));
// → 16
```

Definimos `square`com apenas um parâmetro. No entanto, quando chamamos isso de três, o idioma não se queixa. Ignora os argumentos extras e calcula o quadrado do primeiro.

O JavaScript é extremamente amplo sobre o número de argumentos que você passa para uma função. Se você passar muitos, os extras serão ignorados. Se você passar muito pouco, os parâmetros ausentes serão atribuídos ao valor `undefined`.

A desvantagem disso é que é possível - provavelmente até - que você acidentalmente passe o número errado de argumentos para funções. E ninguém vai falar sobre isso.

A vantagem é que esse comportamento pode ser usado para permitir que uma função seja chamada com diferentes números de argumentos. Por exemplo, essa `minus`função tenta imitar o `-`operador agindo em um ou dois argumentos:

```js
function minus(a, b) {
  if (b === undefined) return -a;
  else return a - b;
}

console.log(minus(10));
// → -10
console.log(minus(10, 5));
// → 5
```

Se você escrever um `=`operador após um parâmetro, seguido por uma expressão, o valor dessa expressão substituirá o argumento quando ele não for fornecido.

Por exemplo, esta versão do `power`torna seu segundo argumento opcional. Se você não fornecer ou passar o valor `undefined`, o valor padrão será dois e a função se comportará como `square`.

```js
function power(base, exponent = 2) {
  let result = 1;
  for (let count = 0; count < exponent; count++) {
    result *= base;
  }
  return result;
}

console.log(power(4));
// → 16
console.log(power(2, 6));
// → 64
```

No [próximo capítulo](https://eloquentjavascript.net/04_data.html#rest_parameters) , veremos uma maneira pela qual um corpo de função pode obter toda a lista de argumentos pelos quais foi passado. Isso é útil porque permite que uma função aceite qualquer número de argumentos. Por exemplo, `console.log`faz isso - gera todos os valores que são fornecidos.

```js
console.log("C", "O", 2);
// → C O 2
```

## Closure

A capacidade de tratar funções como valores, combinada com o fato de que ligações locais são recriadas toda vez que uma função é chamada, levanta uma questão interessante. O que acontece com as ligações locais quando a chamada de função que as criou não está mais ativa?

O código a seguir mostra um exemplo disso. Ele define uma função `wrapValue`, que cria uma ligação local. Em seguida, ele retorna uma função que acessa e retorna essa ligação local.

```js
function wrapValue(n) {
  let local = n;
  return () => local;
}

let wrap1 = wrapValue(1);
let wrap2 = wrapValue(2);
console.log(wrap1());
// → 1
console.log(wrap2());
// → 2
```

Isso é permitido e funciona como você espera - as duas instâncias da ligação ainda podem ser acessadas. Essa situação é uma boa demonstração do fato de que ligações locais são criadas novamente para cada chamada e chamadas diferentes não podem pisar nas ligações locais uma da outra.

Esse recurso - poder referenciar uma instância específica de uma ligação local em um escopo anexo - é chamado de *fechamento* . Uma função que referências ligações de âmbitos locais em torno dele é chamado *um* fechamento. Esse comportamento não apenas libera você de se preocupar com a vida útil das ligações, mas também possibilita o uso de valores de função de algumas maneiras criativas.

Com uma pequena alteração, podemos transformar o exemplo anterior em uma maneira de criar funções que se multiplicam por uma quantia arbitrária.

```js
function multiplier(factor) {
  return number => number * factor;
}

let twice = multiplier(2);
console.log(twice(5));
// → 10
```

A `local`ligação explícita do `wrapValue`exemplo não é realmente necessária, pois um parâmetro é uma ligação local.

Pensar em programas como esse requer alguma prática. Um bom modelo mental é pensar nos valores das funções como contendo o código em seu corpo e o ambiente em que são criados. Quando chamado, o corpo da função vê o ambiente em que foi criado, não o ambiente em que é chamado.

No exemplo, `multiplier`é chamado e cria um ambiente no qual seu `factor`parâmetro está vinculado a 2. O valor da função em que ele retorna, armazenado `twice`, lembra esse ambiente. Então, quando isso é chamado, ele multiplica seu argumento por 2.

## Recursão

É perfeitamente aceitável uma função chamar a si mesma, desde que não faça isso com tanta frequência que transborda a pilha. Uma função que se chama é chamada *recursiva* . A recursão permite que algumas funções sejam escritas em um estilo diferente. Tome, por exemplo, esta implementação alternativa de `power`:

```js
function power(base, exponent) {
  if (exponent == 0) {
    return 1;
  } else {
    return base * power(base, exponent - 1);
  }
}

console.log(power(2, 3));
// → 8
```

Isso é bem parecido com o modo como os matemáticos definem exponenciação e, sem dúvida, descreve o conceito mais claramente do que a variante em loop. A função chama a si mesma várias vezes com expoentes cada vez menores para alcançar a multiplicação repetida.

Mas essa implementação tem um problema: nas implementações típicas de JavaScript, é cerca de três vezes mais lenta que a versão em loop. Executar um loop simples geralmente é mais barato do que chamar uma função várias vezes.

O dilema de velocidade versus elegância é interessante. Você pode vê-lo como uma espécie de continuum entre amizade humana e amizade com máquinas. Quase qualquer programa pode ser feito mais rápido, tornando-o maior e mais complicado. O programador precisa decidir sobre um saldo apropriado.

No caso da `power`função, a versão inelegante (loop) ainda é bastante simples e fácil de ler. Não faz muito sentido substituí-lo pela versão recursiva. Muitas vezes, porém, um programa lida com conceitos tão complexos que é útil desistir de alguma eficiência para tornar o programa mais direto.

Preocupar-se com a eficiência pode ser uma distração. Esse é outro fator que complica o design do programa e, quando você está fazendo algo que já é difícil, essa coisa extra com que se preocupar pode ser paralisante.

Portanto, comece sempre escrevendo algo que seja correto e fácil de entender. Se você estiver preocupado com o fato de ser muito lento - o que geralmente não ocorre, já que a maioria dos códigos simplesmente não é executada com frequência suficiente para levar um tempo significativo - você pode medir posteriormente e melhorá-lo, se necessário.

A recursão nem sempre é apenas uma alternativa ineficiente ao loop. Alguns problemas são realmente mais fáceis de resolver com recursão do que com loops. Na maioria das vezes, são problemas que exigem a exploração ou o processamento de várias "ramificações", cada uma das quais pode se ramificar novamente em mais ramificações.

Considere este quebra-cabeça: começando pelo número 1 e adicionando repetidamente 5 ou multiplicando por 3, um conjunto infinito de números pode ser produzido. Como você escreveria uma função que, dada um número, tenta encontrar uma sequência dessas adições e multiplicações que produz esse número?

Por exemplo, o número 13 pode ser alcançado multiplicando primeiro por 3 e adicionando 5 duas vezes, enquanto o número 15 não pode ser alcançado.

Aqui está uma solução recursiva:

```js
function findSolution(target) {
  function find(current, history) {
    if (current == target) {
      return history;
    } else if (current > target) {
      return null;
    } else {
      return find(current + 5, `(${history} + 5)`) ||
             find(current * 3, `(${history} * 3)`);
    }
  }
  return find(1, "1");
}

console.log(findSolution(24));
// → (((1 * 3) + 5) * 3)
```

Observe que este programa não encontra necessariamente a *menor* seqüência de operações. É satisfeito quando encontra qualquer sequência.

Tudo bem se você não vê como funciona imediatamente. Vamos trabalhar com isso, uma vez que contribui para um ótimo exercício de pensamento recursivo.

A função interna `find`faz a recorrência real. São necessários dois argumentos: o número atual e uma sequência que registra como atingimos esse número. Se encontrar uma solução, ele retornará uma string que mostra como chegar ao destino. Se nenhuma solução puder ser encontrada a partir desse número, ela retornará `null`.

Para fazer isso, a função executa uma das três ações. Se o número atual for o número de destino, o histórico atual é uma maneira de atingir esse destino e, portanto, é retornado. Se o número atual for maior que o destino, não há sentido em explorar mais esse ramo, pois a adição e a multiplicação apenas aumentarão o número e, portanto, ele retornará `null`. Por fim, se ainda estamos abaixo do número de destino, a função tenta os dois caminhos possíveis que começam no número atual, chamando-se duas vezes, uma para adição e outra para multiplicação. Se a primeira chamada retornar algo que não é `null`, ela será retornada. Caso contrário, a segunda chamada será retornada, independentemente de produzir uma sequência ou `null`.

Para entender melhor como essa função produz o efeito que estamos procurando, vejamos todas as chamadas `find`feitas ao procurar uma solução para o número 13.

```js
find(1, "1")
  find(6, "(1 + 5)")
    find(11, "((1 + 5) + 5)")
      find(16, "(((1 + 5) + 5) + 5)")
        too big
      find(33, "(((1 + 5) + 5) * 3)")
        too big
    find(18, "((1 + 5) * 3)")
      too big
  find(3, "(1 * 3)")
    find(8, "((1 * 3) + 5)")
      find(13, "(((1 * 3) + 5) + 5)")
        found!
```

O recuo indica a profundidade da pilha de chamadas. A primeira vez que `find`é chamada, ela começa chamando a si mesma para explorar a solução que começa com ela `(1 + 5)`. Essa chamada será mais recomendada para explorar *todas as* soluções contínuas que produzam um número menor ou igual ao número de destino. Como não encontra um que atinja o alvo, ele `null`volta à primeira chamada. Lá, o `||`operador faz com que a chamada que explora `(1 * 3)`aconteça. Essa pesquisa tem mais sorte - sua primeira chamada recursiva, através de *outra* chamada recursiva, atinge o número de destino. Essa chamada mais interna retorna uma sequência, e cada um dos `||`operadores nas chamadas intermediárias passa essa sequência, retornando a solução.

## Funções crescentes

Existem duas maneiras mais ou menos naturais de introduzir funções nos programas.

A primeira é que você se escreve escrevendo código semelhante várias vezes. Você prefere não fazer isso. Ter mais código significa mais espaço para ocultar erros e mais material para ler para as pessoas que tentam entender o programa. Então você pega a funcionalidade repetida, encontra um bom nome para ela e a coloca em uma função.

A segunda maneira é que você precisa de algumas funcionalidades que ainda não escreveu e que parecem merecer sua própria função. Você começará nomeando a função e depois escreverá seu corpo. Você pode até começar a escrever o código que usa a função antes de definir a própria função.

O quão difícil é encontrar um bom nome para uma função é uma boa indicação de quão claro é um conceito que você está tentando entender. Vamos passar por um exemplo.

Queremos escrever um programa que imprima dois números: o número de vacas e galinhas em uma fazenda, com as palavras `Cows`e `Chickens`depois delas e zeros preenchidos antes dos dois números, para que tenham sempre três dígitos.

```js
007 Cows
011 Chicken
```

Isso exige uma função de dois argumentos - o número de vacas e o número de galinhas. Vamos começar a codificar.

```js
function printFarmInventory(cows, chickens) {
  let cowString = String(cows);
  while (cowString.length < 3) {
    cowString = "0" + cowString;
  }
  console.log(`${cowString} Cows`);
  let chickenString = String(chickens);
  while (chickenString.length < 3) {
    chickenString = "0" + chickenString;
  }
  console.log(`${chickenString} Chickens`);
}
printFarmInventory(7, 11);
```

Escrever `.length`após uma expressão de string nos dará o tamanho dessa string. Assim, os `while`loops continuam adicionando zeros na frente das seqüências numéricas até que tenham pelo menos três caracteres.

Missão cumprida! Mas quando estamos prestes a enviar o código ao criador (junto com uma fatura pesada), ela liga e diz que também começou a criar porcos, e não poderíamos, por favor, estender o software para também imprimir porcos?

Temos certeza que podemos. Mas, assim como estamos copiando e colando essas quatro linhas mais uma vez, paramos e reconsideramos. Tem que haver uma maneira melhor. Aqui está uma primeira tentativa:

```js
function printZeroPaddedWithLabel(number, label) {
  let numberString = String(number);
  while (numberString.length < 3) {
    numberString = "0" + numberString;
  }
  console.log(`${numberString} ${label}`);
}

function printFarmInventory(cows, chickens, pigs) {
  printZeroPaddedWithLabel(cows, "Cows");
  printZeroPaddedWithLabel(chickens, "Chickens");
  printZeroPaddedWithLabel(pigs, "Pigs");
}

printFarmInventory(7, 11, 3);
```

Funciona! Mas esse nome `printZeroPaddedWithLabel`,, é um pouco estranho. Combina três coisas - impressão, preenchimento zero e adição de etiqueta - em uma única função.

Em vez de retirar a parte repetida do nosso programa por atacado, vamos tentar escolher um único *conceito* .

```js
function printZeroPaddedWithLabel(number, label) {
  let numberString = String(number);
  while (numberString.length < 3) {
    numberString = "0" + numberString;
  }
  console.log(`${numberString} ${label}`);
}

function printFarmInventory(cows, chickens, pigs) {
  printZeroPaddedWithLabel(cows, "Cows");
  printZeroPaddedWithLabel(chickens, "Chickens");
  printZeroPaddedWithLabel(pigs, "Pigs");
}

printFarmInventory(7, 11, 3);
```

Uma função com um nome agradável e óbvio como `zeroPad`facilita a quem lê o código descobrir o que ele faz. E essa função é útil em mais situações do que apenas neste programa específico. Por exemplo, você pode usá-lo para ajudar a imprimir tabelas de números bem alinhadas.

Quão inteligente e versátil *deve* ser a nossa função? Poderíamos escrever qualquer coisa, desde uma função terrivelmente simples que só pode preencher um número com três caracteres de largura a um sistema de formatação de números generalizado complicado que lida com números fracionários, números negativos, alinhamento de pontos decimais, preenchimento com caracteres diferentes e assim por diante .

Um princípio útil é não adicionar inteligência, a menos que você tenha certeza absoluta de que precisará disso. Pode ser tentador escrever "frameworks" gerais para todas as funcionalidades que você encontrar. Resista a esse desejo. Você não realizará nenhum trabalho real - apenas escreverá um código que nunca usará.

## Funções e efeitos colaterais

As funções podem ser divididas aproximadamente naquelas chamadas por seus efeitos colaterais e naquelas chamadas por seu valor de retorno. (Embora também seja definitivamente possível que ambos tenham efeitos colaterais e retornem um valor).

A primeira função auxiliar no exemplo do farm `printZeroPaddedWithLabel`, é chamada pelo seu efeito colateral: imprime uma linha. A segunda versão,, `zeroPad`é chamada por seu valor de retorno. Não é por acaso que o segundo é útil em mais situações que o primeiro. As funções que criam valores são mais fáceis de combinar de novas maneiras do que as funções que executam diretamente efeitos colaterais.

Uma função *pura* é um tipo específico de função produtora de valor que não apenas não tem efeitos colaterais, mas também não se baseia nos efeitos colaterais de outro código - por exemplo, não lê ligações globais cujo valor pode mudar. Uma função pura tem a propriedade agradável de que, quando chamada com os mesmos argumentos, sempre produz o mesmo valor (e não faz mais nada). Uma chamada para essa função pode ser substituída por seu valor de retorno sem alterar o significado do código. Quando você não tem certeza de que uma função pura está funcionando corretamente, você pode testá-la simplesmente chamando-a e saber que, se funcionar nesse contexto, funcionará em qualquer contexto. Funções não puras tendem a exigir mais andaimes para serem testadas.

Ainda assim, não é necessário se sentir mal ao escrever funções que não são puras ou fazer uma guerra santa para eliminá-las do seu código. Os efeitos colaterais costumam ser úteis. Não há como escrever uma versão pura `console.log`, por exemplo, e `console.log`é bom ter. Algumas operações também são mais fáceis de expressar de maneira eficiente quando usamos efeitos colaterais; portanto, a velocidade da computação pode ser um motivo para evitar a pureza.

## Sumário

Este capítulo ensinou como escrever suas próprias funções. A `function`palavra-chave, quando usada como expressão, pode criar um valor de função. Quando usado como uma instrução, pode ser usado para declarar uma ligação e atribuir a ela uma função como seu valor. As funções de seta são mais uma maneira de criar funções.

```js
// Define f to hold a function value
const f = function(a) {
  console.log(a + 2);
};

// Declare g to be a function
function g(a, b) {
  return a * b * 3.5;
}

// A less verbose function value
let h = a => a % 3;
```

Um aspecto fundamental na compreensão de funções é a compreensão de escopos. Cada bloco cria um novo escopo. Parâmetros e ligações declarados em um determinado escopo são locais e não são visíveis de fora. As ligações declaradas com `var`comportamento diferente - elas acabam no escopo de função mais próximo ou no escopo global.

Separar as tarefas que seu programa executa em diferentes funções é útil. Você não precisará se repetir tanto, e as funções podem ajudar a organizar um programa, agrupando o código em partes que fazem coisas específicas.

## Exercícios

### Mínimo

O [capítulo anterior](https://eloquentjavascript.net/02_program_structure.html#return_values) apresentou a função padrão `Math.min`que retorna seu menor argumento. Podemos construir algo assim agora. Escreva uma função `min`que use dois argumentos e retorne o mínimo.

```js
// Your code here.

console.log(min(0, 10));
// → 0
console.log(min(0, -10));
// → -10
```

Se você tiver problemas para colocar chaves e parênteses no lugar certo para obter uma definição de função válida, comece copiando um dos exemplos deste capítulo e modificando-o.

Uma função pode conter várias `return`instruções.

### Recursão

Vimos que `%`(o operador restante) pode ser usado para testar se um número é par ou ímpar usando `% 2`para ver se é divisível por dois. Aqui está outra maneira de definir se um número inteiro positivo é par ou ímpar:

- Zero é par.
- Um é estranho.
- Para qualquer outro número *N* , sua uniformidade é igual a *N* - 2.

Defina uma função recursiva `isEven`correspondente a esta descrição. A função deve aceitar um único parâmetro (um número inteiro positivo) e retornar um booleano.

Teste-o em 50 e 75. Veja como ele se comporta em -1. Por quê? Você consegue pensar em uma maneira de consertar isso?

```
// Your code here.

console.log(isEven(50));
// → true
console.log(isEven(75));
// → false
console.log(isEven(-1));
// → ??
```

Sua função provavelmente será parecida com a `find`função interna no `findSolution` [exemplo](https://eloquentjavascript.net/03_functions.html#recursive_puzzle) recursivo deste capítulo, com uma cadeia `if`/ `else if`/ `else`que testa qual dos três casos se aplica. A final `else`, correspondente ao terceiro caso, faz a chamada recursiva. Cada uma das ramificações deve conter uma `return`declaração ou, de alguma outra maneira, providenciar para que um valor específico seja retornado.

Quando recebe um número negativo, a função é repetida várias vezes, passando para si um número cada vez mais negativo, ficando cada vez mais longe do retorno de um resultado. Acabará ficando sem espaço na pilha e abortará.

### Contagem de feijão

Você pode obter o enésimo caractere, ou letra, de uma string escrevendo `"string"[N]`. O valor retornado será uma sequência que contém apenas um caractere (por exemplo, `"b"`). O primeiro caractere tem a posição 0, o que faz com que o último seja encontrado na posição . Em outras palavras, uma cadeia de dois caracteres tem comprimento 2 e seus caracteres têm as posições 0 e 1.`string.length - 1`

Escreva uma função `countBs`que use uma string como seu único argumento e retorne um número que indica quantos caracteres “B” maiúsculos existem na string.

Em seguida, escreva uma função chamada `countChar`que se comporte como `countBs`, exceto que é necessário um segundo argumento que indica o caractere que deve ser contado (em vez de contar apenas os caracteres "B" maiúsculos). Reescreva `countBs`para fazer uso desta nova função.

```
// Your code here.

console.log(countBs("BBC"));
// → 2
console.log(countChar("kakkerlak", "k"));
// → 4
```

Sua função precisará de um loop que analise todos os caracteres da string. Ele pode executar um índice de zero a um abaixo do seu comprimento ( ). Se o caractere na posição atual for o mesmo que a função está procurando, ele adicionará 1 a uma variável de contador. Depois que o loop terminar, o contador poderá ser retornado.`< string.length`

Tome cuidado para tornar todas as ligações usadas na função *local* para a função, declarando-as adequadamente com a palavra-chave `let`ou `const`.