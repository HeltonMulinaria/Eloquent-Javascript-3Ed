# Projeto do

# Capítulo 12: 

# Uma linguagem de programação

> O avaliador, que determina o significado das expressões em uma linguagem de programação, é apenas outro programa.
>
> *—Hal Abelson e Gerald Sussman, Estrutura e Interpretação de Programas de Computador*

![Imagem de um ovo com ovos menores dentro](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/chapter_picture_12.jpg)

Construir sua própria linguagem de programação é surpreendentemente fácil (desde que você não mire muito alto) e muito esclarecedor.

A principal coisa que quero mostrar neste capítulo é que não há mágica envolvida na construção de seu próprio idioma. Eu sempre senti que algumas invenções humanas eram tão imensamente inteligentes e complicadas que eu nunca seria capaz de entendê-las. Mas, com um pouco de leitura e experimentação, elas geralmente se mostram bastante mundanas.

Vamos construir uma linguagem de programação chamada Egg. Será uma linguagem minúscula e simples - mas poderosa o suficiente para expressar qualquer cálculo que você possa imaginar. Permitirá abstração simples com base em funções.

## Análise

A parte mais imediatamente visível de uma linguagem de programação é sua *sintaxe* ou notação. Um *analisador* é um programa que lê um pedaço de texto e produz uma estrutura de dados que reflete a estrutura do programa contida nesse texto. Se o texto não formar um programa válido, o analisador deve apontar o erro.

Nossa linguagem terá uma sintaxe simples e uniforme. Tudo no Egg é uma expressão. Uma expressão pode ser o nome de uma ligação, um número, uma sequência ou um *aplicativo* . Os aplicativos são usados para chamadas de função, mas também para construções como `if`ou `while`.

Para manter o analisador simples, as seqüências de caracteres no Egg não suportam nada como escapes de barra invertida. Uma string é simplesmente uma sequência de caracteres que não são aspas duplas, agrupadas entre aspas duplas. Um número é uma sequência de dígitos. Os nomes de ligação podem consistir em qualquer caractere que não esteja em branco e que não tenha um significado especial na sintaxe.

Os aplicativos são gravados da maneira que estão no JavaScript, colocando parênteses após uma expressão e tendo qualquer número de argumentos entre esses parênteses, separados por vírgulas.

```js
do(define(x, 10),
   if(>(x, 5),
      print("large"),
      print("small")))
```

A uniformidade da linguagem Egg significa que coisas que são operadores em JavaScript (como `>`) são ligações normais nessa linguagem, aplicadas como outras funções. E como a sintaxe não tem conceito de bloco, precisamos de uma `do`construção para representar a realização de várias coisas em sequência.

A estrutura de dados que o analisador usará para descrever um programa consiste em objetos de expressão, cada um com uma `type`propriedade indicando o tipo de expressão que é e outras propriedades para descrever seu conteúdo.

Expressões do tipo `"value"`representam cadeias literais ou números. Sua `value`propriedade contém o valor da sequência ou número que eles representam. Expressões do tipo `"word"`são usadas para identificadores (nomes). Esses objetos têm uma `name`propriedade que contém o nome do identificador como uma sequência. Finalmente, `"apply"`expressões representam aplicativos. Eles têm uma `operator`propriedade que se refere à expressão que está sendo aplicada, bem como uma `args`propriedade que contém uma matriz de expressões de argumento.

A `>(x, 5)`parte do programa anterior seria representada assim:

```js
{
  type: "apply",
  operator: {type: "word", name: ">"},
  args: [
    {type: "word", name: "x"},
    {type: "value", value: 5}
  ]
}
```

Essa estrutura de dados é chamada de *árvore de sintaxe* . Se você imaginar os objetos como pontos e os links entre eles como linhas entre esses pontos, ele terá uma forma de árvore. O fato de expressões conterem outras expressões, que por sua vez podem conter mais expressões, é semelhante à maneira como os galhos das árvores se dividem e se dividem novamente.

![A estrutura de uma árvore de sintaxe](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/syntax_tree.svg)

Compare isso com o analisador que escrevemos para o formato do arquivo de configuração no [Capítulo 9](https://eloquentjavascript.net/09_regexp.html#ini) , que tinha uma estrutura simples: dividia a entrada em linhas e manipulava essas linhas uma por vez. Havia apenas algumas formas simples que uma linha podia ter.

Aqui devemos encontrar uma abordagem diferente. As expressões não são separadas em linhas e possuem uma estrutura recursiva. Expressões de aplicativo *contêm* outras expressões.

Felizmente, esse problema pode ser resolvido muito bem, escrevendo uma função de analisador que é recursiva de uma maneira que reflete a natureza recursiva da linguagem.

Definimos uma função `parseExpression`, que recebe uma string como entrada e retorna um objeto que contém a estrutura de dados da expressão no início da string, junto com a parte da string deixada após analisar essa expressão. Ao analisar subexpressões (o argumento de um aplicativo, por exemplo), essa função pode ser chamada novamente, produzindo a expressão do argumento e o texto restante. Este texto, por sua vez, pode conter mais argumentos ou pode ser o parêntese de fechamento que termina a lista de argumentos.

Esta é a primeira parte do analisador:

```js
function parseExpression(program) {
  program = skipSpace(program);
  let match, expr;
  if (match = /^"([^"]*)"/.exec(program)) {
    expr = {type: "value", value: match[1]};
  } else if (match = /^\d+\b/.exec(program)) {
    expr = {type: "value", value: Number(match[0])};
  } else if (match = /^[^\s(),#"]+/.exec(program)) {
    expr = {type: "word", name: match[0]};
  } else {
    throw new SyntaxError("Unexpected syntax: " + program);
  }

  return parseApply(expr, program.slice(match[0].length));
}

function skipSpace(string) {
  let first = string.search(/\S/);
  if (first == -1) return "";
  return string.slice(first);
}
```

Como o Egg, como o JavaScript, permite qualquer quantidade de espaço em branco entre seus elementos, temos que cortar repetidamente o espaço em branco no início da string do programa. É com isso que a `skipSpace`função ajuda.

Depois de pular qualquer espaço inicial, `parseExpression`usa três expressões regulares para identificar os três elementos atômicos suportados por Egg: cadeias, números e palavras. O analisador constrói um tipo diferente de estrutura de dados, dependendo de qual deles corresponde. Se a entrada não corresponder a uma dessas três formas, não será uma expressão válida e o analisador emitirá um erro. Em `SyntaxError`vez disso, usamos `Error`como construtor de exceção, que é outro tipo de erro padrão, porque é um pouco mais específico - também é o tipo de erro lançado quando é feita uma tentativa de executar um programa JavaScript inválido.

Em seguida, cortamos a parte correspondente à seqüência do programa e passamos isso, junto com o objeto para a expressão, para `parseApply`, que verifica se a expressão é um aplicativo. Nesse caso, analisa uma lista de argumentos entre parênteses.

```js
function parseApply(expr, program) {
  program = skipSpace(program);
  if (program[0] != "(") {
    return {expr: expr, rest: program};
  }

  program = skipSpace(program.slice(1));
  expr = {type: "apply", operator: expr, args: []};
  while (program[0] != ")") {
    let arg = parseExpression(program);
    expr.args.push(arg.expr);
    program = skipSpace(arg.rest);
    if (program[0] == ",") {
      program = skipSpace(program.slice(1));
    } else if (program[0] != ")") {
      throw new SyntaxError("Expected ',' or ')'");
    }
  }
  return parseApply(expr, program.slice(1));
}
```

Se o próximo caractere no programa não for um parêntese de abertura, não será um aplicativo e `parseApply`retornará a expressão que foi dada.

Caso contrário, ele ignora o parêntese de abertura e cria o objeto da árvore de sintaxe para esta expressão de aplicativo. Em seguida, recursivamente chama `parseExpression`para analisar cada argumento até que um parêntese de fechamento seja encontrado. A recursão é indireta, através `parseApply`e `parseExpression`chamando um ao outro.

Como uma expressão de aplicativo pode ser aplicada (como em `multiplier(2)(1)`), `parseApply`deve, depois de analisar um aplicativo, se chamar novamente para verificar se outro par de parênteses segue.

É tudo o que precisamos para analisar o Egg. Nós o envolvemos em uma `parse`função conveniente que verifica se atingiu o final da sequência de entrada após analisar a expressão (um programa Egg é uma expressão única) e que nos fornece a estrutura de dados do programa.

```js
function parse(program) {
  let {expr, rest} = parseExpression(program);
  if (skipSpace(rest).length > 0) {
    throw new SyntaxError("Unexpected text after program");
  }
  return expr;
}

console.log(parse("+(a, 10)"));
// → {type: "apply",
//    operator: {type: "word", name: "+"},
//    args: [{type: "word", name: "a"},
//           {type: "value", value: 10}]}
```

Funciona! Ele não fornece informações muito úteis quando falha e não armazena a linha e a coluna em que cada expressão é iniciada, o que pode ser útil ao relatar erros posteriormente, mas é bom o suficiente para nossos propósitos.

## O avaliador

O que podemos fazer com a árvore de sintaxe de um programa? Corra, é claro! E é isso que o avaliador faz. Você fornece a ele uma árvore de sintaxe e um objeto de escopo que associam nomes a valores, e ele avaliará a expressão que a árvore representa e retornará o valor que isso produz.

```js
const specialForms = Object.create(null);

function evaluate(expr, scope) {
  if (expr.type == "value") {
    return expr.value;
  } else if (expr.type == "word") {
    if (expr.name in scope) {
      return scope[expr.name];
    } else {
      throw new ReferenceError(
        `Undefined binding: ${expr.name}`);
    }
  } else if (expr.type == "apply") {
    let {operator, args} = expr;
    if (operator.type == "word" &&
        operator.name in specialForms) {
      return specialForms[operator.name](expr.args, scope);
    } else {
      let op = evaluate(operator, scope);
      if (typeof op == "function") {
        return op(...args.map(arg => evaluate(arg, scope)));
      } else {
        throw new TypeError("Applying a non-function.");
      }
    }
  }
}
```

O avaliador possui código para cada um dos tipos de expressão. Uma expressão de valor literal produz seu valor. (Por exemplo, a expressão `100`apenas é avaliada como o número 100.) Para uma ligação, devemos verificar se ela está realmente definida no escopo e, se for, buscar o valor da ligação.

Os aplicativos estão mais envolvidos. Se eles são uma forma especial, como `if`, não avaliamos nada e passamos as expressões de argumento, juntamente com o escopo, para a função que lida com essa forma. Se for uma chamada normal, avaliamos o operador, verificamos que é uma função e a chamamos com os argumentos avaliados.

Usamos valores simples da função JavaScript para representar os valores da função do Egg. Voltaremos a isso [mais tarde](https://eloquentjavascript.net/12_language.html#egg_fun) , quando o formulário especial chamado `fun`for definido.

A estrutura recursiva de se `evaluate`assemelha à estrutura semelhante do analisador, e ambos espelham a estrutura da própria linguagem. Também seria possível integrar o analisador ao avaliador e avaliar durante a análise, mas dividi-los dessa maneira torna o programa mais claro.

Isso é realmente tudo o que é necessário para interpretar Egg. É simples assim. Mas sem definir algumas formas especiais e adicionar alguns valores úteis ao ambiente, você ainda não pode fazer muito com esse idioma.

## Formulários especiais

O `specialForms`objeto é usado para definir sintaxe especial no Egg. Associa palavras a funções que avaliam tais formas. Está vazio no momento. Vamos adicionar `if`.

```js
specialForms.if = (args, scope) => {
  if (args.length != 3) {
    throw new SyntaxError("Wrong number of args to if");
  } else if (evaluate(args[0], scope) !== false) {
    return evaluate(args[1], scope);
  } else {
    return evaluate(args[2], scope);
  }
};
```

A `if`construção de Egg espera exatamente três argumentos. Ele avaliará o primeiro e, se o resultado não for o valor `false`, avaliará o segundo. Caso contrário, o terceiro será avaliado. Este `if`formulário é mais semelhante ao `?:`operador ternário do JavaScript do que ao do JavaScript `if`. É uma expressão, não uma afirmação, e produz um valor, a saber, o resultado do segundo ou terceiro argumento.

Egg também difere do JavaScript em como ele lida com o valor da condição `if`. Não tratará coisas como zero ou a string vazia como falsa, apenas o valor exato `false`.

A razão é preciso representar `if`como uma forma especial, em vez de uma função regular, é que todos os argumentos para funções são avaliadas antes que a função é chamada, enquanto que `if`deve avaliar apenas *quer* seu segundo ou terceiro argumento, dependendo do valor do primeiro .

O `while`formulário é semelhante.

```js
specialForms.while = (args, scope) => {
  if (args.length != 2) {
    throw new SyntaxError("Wrong number of args to while");
  }
  while (evaluate(args[0], scope) !== false) {
    evaluate(args[1], scope);
  }

  // Since undefined does not exist in Egg, we return false,
  // for lack of a meaningful result.
  return false;
};
```

Outro componente básico é o `do`qual executa todos os seus argumentos de cima para baixo. Seu valor é o valor produzido pelo último argumento.

```js
specialForms.do = (args, scope) => {
  let value = false;
  for (let arg of args) {
    value = evaluate(arg, scope);
  }
  return value;
};
```

Para poder criar ligações e fornecer novos valores, também criamos um formulário chamado `define`. Ele espera uma palavra como seu primeiro argumento e uma expressão produzindo o valor a ser atribuído a essa palavra como seu segundo argumento. Como `define`, como tudo, é uma expressão, ele deve retornar um valor. Vamos fazer com que ele retorne o valor que foi atribuído (assim como o `=`operador do JavaScript ).

```js
specialForms.define = (args, scope) => {
  if (args.length != 2 || args[0].type != "word") {
    throw new SyntaxError("Incorrect use of define");
  }
  let value = evaluate(args[1], scope);
  scope[args[0].name] = value;
  return value;
};
```

## O ambiente

O escopo aceito por `evaluate`é um objeto com propriedades cujos nomes correspondem a nomes de ligação e cujos valores correspondem aos valores aos quais essas ligações estão associadas. Vamos definir um objeto para representar o escopo global.

Para poder usar a `if`construção que acabamos de definir, precisamos ter acesso aos valores booleanos. Como existem apenas dois valores booleanos, não precisamos de sintaxe especial para eles. Nós simplesmente ligar dois nomes para os valores `true`e `false`e usá-los.

```js
const topScope = Object.create(null);

topScope.true = true;
topScope.false = false;
```

Agora podemos avaliar uma expressão simples que nega um valor booleano.

```js
let prog = parse(`if(true, false, true)`);
console.log(evaluate(prog, topScope));
// → false
```

Para fornecer operadores aritméticos e de comparação básicos, também adicionaremos alguns valores de função ao escopo. No interesse de manter o código curto, usaremos `Function`para sintetizar várias funções do operador em um loop, em vez de defini-las individualmente.

```js
for (let op of ["+", "-", "*", "/", "==", "<", ">"]) {
  topScope[op] = Function("a, b", `return a ${op} b;`);
}
```

Uma maneira de gerar valores também é útil, portanto, agruparemos `console.log`uma função e a chamaremos `print`.

```js
topScope.print = value => {
  console.log(value);
  return value;
};
```

Isso nos dá ferramentas elementares suficientes para escrever programas simples. A função a seguir fornece uma maneira conveniente de analisar um programa e executá-lo em um novo escopo:

```js
function run(program) {
  return evaluate(parse(program), Object.create(topScope));
}
```

Usaremos cadeias de protótipos de objetos para representar escopos aninhados, para que o programa possa adicionar ligações ao seu escopo local sem alterar o escopo de nível superior.

```js
run(`
do(define(total, 0),
   define(count, 1),
   while(<(count, 11),
         do(define(total, +(total, count)),
            define(count, +(count, 1)))),
   print(total))
`);
// → 55
```

Este é o programa que vimos várias vezes antes, que calcula a soma dos números de 1 a 10, expressos em Egg. É claramente mais feio que o programa JavaScript equivalente - mas não é ruim para uma linguagem implementada em menos de 150 linhas de código.

## Funções

Uma linguagem de programação sem funções é realmente uma linguagem de programação ruim.

Felizmente, não é difícil adicionar uma `fun`construção, que trata seu último argumento como o corpo da função e usa todos os argumentos anteriores como nomes dos parâmetros da função.

```js
specialForms.fun = (args, scope) => {
  if (!args.length) {
    throw new SyntaxError("Functions need a body");
  }
  let body = args[args.length - 1];
  let params = args.slice(0, args.length - 1).map(expr => {
    if (expr.type != "word") {
      throw new SyntaxError("Parameter names must be words");
    }
    return expr.name;
  });

  return function() {
    if (arguments.length != params.length) {
      throw new TypeError("Wrong number of arguments");
    }
    let localScope = Object.create(scope);
    for (let i = 0; i < arguments.length; i++) {
      localScope[params[i]] = arguments[i];
    }
    return evaluate(body, localScope);
  };
};
```

As funções no Egg têm seu próprio escopo local. A função produzida pelo `fun`formulário cria esse escopo local e adiciona as ligações de argumento a ele. Ele avalia o corpo da função nesse escopo e retorna o resultado.

```js
run(`
do(define(plusOne, fun(a, +(a, 1))),
   print(plusOne(10)))
`);
// → 11

run(`
do(define(pow, fun(base, exp,
     if(==(exp, 0),
        1,
        *(base, pow(base, -(exp, 1)))))),
   print(pow(2, 10)))
`);
// → 1024
```

## Compilação

O que construímos é um intérprete. Durante a avaliação, atua diretamente na representação do programa produzido pelo analisador.

*Compilação* é o processo de adicionar outra etapa entre a análise e a execução de um programa, que transforma o programa em algo que pode ser avaliado com mais eficiência, fazendo o máximo de trabalho possível com antecedência. Por exemplo, em linguagens bem projetadas, é óbvio, para cada uso de uma ligação, à qual a ligação está sendo referida, sem realmente executar o programa. Isso pode ser usado para evitar procurar a ligação pelo nome sempre que for acessada, em vez de buscá-la diretamente de algum local de memória predeterminado.

Tradicionalmente, a compilação envolve a conversão do programa em código de máquina, o formato bruto que o processador de um computador pode executar. Mas qualquer processo que converta um programa em uma representação diferente pode ser considerado uma compilação.

Seria possível escrever uma estratégia de avaliação alternativa para o Egg, que primeiro converte o programa em um programa JavaScript, usa `Function`para chamar o compilador JavaScript e depois executa o resultado. Quando bem feito, isso faria o Egg rodar muito rápido e ainda assim ser bastante simples de implementar.

Se você estiver interessado neste tópico e disposto a dedicar algum tempo a ele, encorajo você a tentar implementar um compilador como um exercício.

## Traindo

Quando definimos `if`e `while`, você provavelmente percebeu que eles eram invólucros mais ou menos triviais em torno do próprio JavaScript `if`e `while`. Da mesma forma, os valores no Egg são apenas valores regulares do JavaScript antigo.

Se você comparar a implementação do Egg, construída sobre JavaScript, com a quantidade de trabalho e complexidade necessária para criar uma linguagem de programação diretamente na funcionalidade bruta fornecida por uma máquina, a diferença é enorme. Independentemente disso, este exemplo deu uma impressão ideal de como as linguagens de programação funcionam.

E quando se trata de fazer algo, trapacear é mais eficaz do que fazer tudo sozinho. Embora a linguagem de brinquedo neste capítulo não faz nada que não poderia ser feito melhor em JavaScript, não *são* situações em escrever pequenas línguas ajuda a trabalhar de verdade.

Essa linguagem não precisa se parecer com uma linguagem de programação típica. Se o JavaScript não vier equipado com expressões regulares, por exemplo, você poderá escrever seu próprio analisador e avaliador para expressões regulares.

Ou imagine que você está construindo um dinossauro robótico gigante e precisa programar seu comportamento. JavaScript pode não ser a maneira mais eficaz de fazer isso. Você pode optar por um idioma parecido com este:

```
behavior walk
  perform when
    destination ahead
  actions
    move left-foot
    move right-foot

behavior attack
  perform when
    Godzilla in-view
  actions
    fire laser-eyes
    launch arm-rockets
```

Isso é o que geralmente é chamado *de linguagem específica de domínio* , uma linguagem adaptada para expressar um domínio restrito do conhecimento. Essa linguagem pode ser mais expressiva do que uma linguagem de uso geral, porque foi projetada para descrever exatamente as coisas que precisam ser descritas em seu domínio e nada mais.

## Exercícios

### Matrizes

Adicione suporte a matrizes no Egg adicionando as três funções a seguir no escopo superior: `array(...values)`construir uma matriz contendo os valores do argumento, `length(array)`obter o comprimento de uma matriz e `element(array, n)`buscar o enésimo elemento de uma matriz.

```js
// Modify these definitions...

topScope.array = "...";

topScope.length = "...";

topScope.element = "...";

run(`
do(define(sum, fun(array,
     do(define(i, 0),
        define(sum, 0),
        while(<(i, length(array)),
          do(define(sum, +(sum, element(array, i))),
             define(i, +(i, 1)))),
        sum))),
   print(sum(array(1, 2, 3))))
`);
// → 6
```

A maneira mais fácil de fazer isso é representar matrizes Egg com matrizes JavaScript.

Os valores adicionados ao escopo superior devem ser funções. Usando um argumento de descanso (com notação de ponto triplo), a definição de `array`pode ser *muito* simples.

### Closure

A maneira como definimos `fun`permite que as funções no Egg façam referência ao escopo circundante, permitindo que o corpo da função use valores locais visíveis no momento em que a função foi definida, assim como as funções JavaScript.

O programa a seguir ilustra isso: function `f`retorna uma função que adiciona seu argumento ao `f`argumento de, o que significa que ele precisa acessar o escopo local interno `f`para poder usar a ligação `a`.

```
run(`
do(define(f, fun(a, fun(b, +(a, b)))),
   print(f(4)(5)))
`);
// → 9
```

Volte para a definição do `fun`formulário e explique qual mecanismo faz com que isso funcione.

Novamente, estamos usando um mecanismo JavaScript para obter o recurso equivalente no Egg. Os formulários especiais recebem o escopo local em que são avaliados, para que possam avaliar seus subformulários nesse escopo. A função retornada por `fun`tem acesso ao `scope`argumento fornecido à sua função anexa e a utiliza para criar o escopo local da função quando é chamada.

Isso significa que o protótipo do escopo local será o escopo em que a função foi criada, o que torna possível acessar as ligações nesse escopo a partir da função. Isso é tudo o que há para implementar o fechamento (embora, para compilá-lo de uma maneira que seja realmente eficiente, você precisará trabalhar mais).

### Comentários

Seria bom se pudéssemos escrever comentários em Egg. Por exemplo, sempre que encontramos um sinal de hash ( `#`), poderíamos tratar o restante da linha como um comentário e ignorá-lo, semelhante ao `//`JavaScript.

Não precisamos fazer grandes alterações no analisador para suportar isso. Podemos simplesmente mudar `skipSpace`para pular comentários como se fossem espaços em branco, para que todos os pontos `skipSpace`chamados agora também pule comentários. Faça essa alteração.

```js
// This is the old skipSpace. Modify it...
function skipSpace(string) {
  let first = string.search(/\S/);
  if (first == -1) return "";
  return string.slice(first);
}

console.log(parse("# hello\nx"));
// → {type: "word", name: "x"}

console.log(parse("a # one\n   # two\n()"));
// → {type: "apply",
//    operator: {type: "word", name: "a"},
//    args: []}
```

Verifique se a sua solução lida com vários comentários seguidos, com espaço em branco potencial entre ou depois deles.

Uma expressão regular é provavelmente a maneira mais fácil de resolver isso. Escreva algo que corresponda a "espaço em branco ou comentário, zero ou mais vezes". Use o método `exec`ou `match`e observe o comprimento do primeiro elemento na matriz retornada (a correspondência inteira) para descobrir quantos caracteres devem ser cortados.

### Escopo de fixação

Atualmente, a única maneira de atribuir um valor a uma ligação é `define`. Essa construção atua como uma maneira de definir novas ligações e atribuir às existentes um novo valor.

Essa ambiguidade causa um problema. Ao tentar atribuir um valor novo a uma ligação não local, você acabará definindo uma local com o mesmo nome. Alguns idiomas funcionam assim por design, mas sempre achei uma maneira estranha de lidar com o escopo.

Adicione um formulário especial `set`, semelhante a `define`, que dê a uma ligação um novo valor, atualizando a ligação em um escopo externo, se ainda não existir no escopo interno. Se a ligação não estiver definida, lance a `ReferenceError`(outro tipo de erro padrão).

A técnica de representar os escopos como objetos simples, o que tornou as coisas convenientes até agora, interferirá um pouco nesse ponto. Você pode querer usar a função, que retorna o protótipo de um objeto. Lembre-se também de que os escopos não derivam ; portanto, se você quiser chamá -los, use esta expressão desajeitada:`Object.getPrototypeOf``Object.prototype``hasOwnProperty`

```js
specialForms.set = (args, scope) => {
  // Your code here.
};

run(`
do(define(x, 4),
   define(setx, fun(val, set(x, val))),
   setx(50),
   print(x))
`);
// → 50
run(`set(quux, true)`);
// → Some kind of ReferenceError
```

Você terá que percorrer um escopo por vez, usando para ir para o próximo escopo externo. Para cada escopo, use para descobrir se a ligação, indicada pela propriedade do primeiro argumento para , existe nesse escopo. Se houver, defina-o como o resultado da avaliação do segundo argumento e, em seguida, retorne esse valor.`Object.getPrototypeOf``hasOwnProperty``name``set``set`

Se o escopo mais externo for atingido ( retorna nulo) e ainda não encontramos a ligação, ela não existe e um erro deve ser gerado.`Object.getPrototypeOf`