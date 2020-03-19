# Capítulo 2 Estrutura do Programa

> E meu coração brilha em vermelho sob minha pele translúcida e transparente e eles precisam administrar 10cc de JavaScript para que eu volte. (Eu respondo bem a toxinas no sangue.) Cara, essas coisas vão chutar os pêssegos de suas brânquias!
>
> *—_why, Why's (Poignant) Guia de Ruby*

![Imagem de tentáculos segurando objetos](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/chapter_picture_2.jpg)

Neste capítulo, começaremos a fazer coisas que realmente podem ser chamadas de *programação* . Expandiremos nosso comando da linguagem JavaScript além dos substantivos e fragmentos de frases que vimos até agora, a ponto de expressar uma prosa significativa.

## Expressões e declarações

No [Capítulo 1](https://eloquentjavascript.net/01_values.html) , criamos valores e aplicamos operadores a eles para obter novos valores. Criar valores como esse é a principal substância de qualquer programa JavaScript. Mas essa substância precisa ser enquadrada em uma estrutura maior para ser útil. Então é isso que abordaremos a seguir.

Um fragmento de código que produz um valor é chamado de *expressão* . Todo valor que é escrito literalmente (como `22`ou `"psychoanalysis"`) é uma expressão. Uma expressão entre parênteses também é uma expressão, assim como um operador binário aplicado a duas expressões ou um operador unário aplicado a uma.

Isso mostra parte da beleza de uma interface baseada em idioma. As expressões podem conter outras expressões de maneira semelhante à aninhada de subsentências em idiomas humanos - uma subsentência pode conter suas próprias subsentências e assim por diante. Isso nos permite criar expressões que descrevem cálculos arbitrariamente complexos.

Se uma expressão corresponder a um fragmento de sentença, uma *instrução* JavaScript corresponderá a uma sentença completa. Um programa é uma lista de instruções.

O tipo mais simples de afirmação é uma expressão com ponto e vírgula depois. Este é um programa:

```js
1;
!false;
```

É um programa inútil, no entanto. Uma expressão pode ser conteúdo apenas para produzir um valor, que pode ser usado pelo código anexo. Uma afirmação é independente e, portanto, equivale a algo apenas se afeta o mundo. Pode exibir algo na tela - que conta como mudar o mundo - ou pode mudar o estado interno da máquina de uma maneira que afetará as declarações que vêm depois dela. Essas mudanças são chamadas de *efeitos colaterais* . As demonstrações no exemplo anterior apenas produzir os valores `1`e `true`e depois jogá-los imediatamente afastado. Isso não deixa nenhuma impressão ao mundo. Quando você executa este programa, nada observável acontece.

Em alguns casos, o JavaScript permite omitir o ponto e vírgula no final de uma instrução. Em outros casos, ele deve estar lá ou a próxima linha será tratada como parte da mesma instrução. As regras para quando pode ser omitido com segurança são algo complexas e propensas a erros. Portanto, neste livro, toda declaração que precise de um ponto e vírgula sempre terá uma. Eu recomendo que você faça o mesmo, pelo menos até aprender mais sobre as sutilezas dos pontos e vírgulas ausentes.

## Ligações

Como um programa mantém um estado interno? Como se lembra das coisas? Vimos como produzir novos valores a partir de valores antigos, mas isso não altera os valores antigos, e o novo valor deve ser usado imediatamente ou será dissipado novamente. Para capturar e manter valores, o JavaScript fornece uma coisa chamada *associação* ou *variável* :

```js
let caught = 5 * 5;			
```

Esse é um segundo tipo de afirmação. A palavra especial (palavra- *chave* ) `let`indica que esta frase vai definir uma ligação. É seguido pelo nome da ligação e, se queremos atribuir um valor imediatamente, por um `=`operador e uma expressão.

A instrução anterior cria uma ligação chamada `caught`e a usa para manter o número produzido multiplicando 5 por 5.

Após a definição de uma ligação, seu nome pode ser usado como uma expressão. O valor dessa expressão é o valor que a ligação mantém atualmente. Aqui está um exemplo:

```js
let ten = 10;
console.log(ten * ten);
// → 100
```

Quando uma ligação aponta para um valor, isso não significa que está vinculada a esse valor para sempre. O `=`operador pode ser usado a qualquer momento nas ligações existentes para desconectá-las de seu valor atual e fazer com que aponte para um novo.

```js
let mood = "light";
console.log(mood);
// → light
mood = "dark";
console.log(mood);
// → dark
```

Você deve imaginar encadernações como tentáculos, em vez de caixas. Eles não *contêm* valores; eles as *compreendem* - duas ligações podem se referir ao mesmo valor. Um programa pode acessar apenas os valores aos quais ainda tem uma referência. Quando você precisa se lembrar de algo, você cria um tentáculo para segurá-lo ou reconecta um de seus tentáculos existentes.

Vejamos outro exemplo. Para lembrar o número de dólares que Luigi ainda lhe deve, crie uma ligação. E então, quando ele paga $ 35, você atribui a esse vínculo um novo valor.

```js
let luigisDebt = 140;
luigisDebt = luigisDebt - 35;
console.log(luigisDebt);
// → 105
```

Quando você define uma encadernação sem atribuir um valor a ela, o tentáculo não tem nada a entender e termina no ar. Se você solicitar o valor de uma ligação vazia, receberá o valor `undefined`.

Uma única `let`instrução pode definir várias ligações. As definições devem ser separadas por vírgulas.

```js
let one = 1, two = 2;
console.log(one + two);
// → 3
```

As palavras `var`e `const`também podem ser usadas para criar ligações, de maneira semelhante a `let`.

```js
var name = "Ayda";
const greeting = "Hello ";
console.log(greeting + name);
// → Hello Ayda
```

A primeira `var`(abreviação de "variable") é a maneira como as ligações foram declaradas no JavaScript anterior a 2015. Voltarei à maneira exata em que difere `let`no [próximo capítulo](https://eloquentjavascript.net/03_functions.html) . Por enquanto, lembre-se de que ele faz a mesma coisa, mas raramente o usaremos neste livro porque possui algumas propriedades confusas.

A palavra `const`significa *constante* . Ele define uma ligação constante, que aponta para o mesmo valor enquanto durar. Isso é útil para ligações que atribuem um nome a um valor, para que você possa consultá-lo facilmente mais tarde.

## Nomes de ligação

Nomes de ligação podem ser qualquer palavra. Os dígitos podem fazer parte dos nomes de ligação - `catch22`é um nome válido, por exemplo - mas o nome não deve começar com um dígito. Um nome de ligação pode incluir cifrões ( `$`) ou sublinhados ( `_`), mas nenhuma outra pontuação ou caracteres especiais.

Palavras com um significado especial, como `let`, são *palavras-chave* e não podem ser usadas como nomes de ligação. Também há várias palavras que são "reservadas para uso" em versões futuras do JavaScript, que também não podem ser usadas como nomes de ligação. A lista completa de palavras-chave e palavras reservadas é bastante longa.

```
caso de interrupção captura classe const continuar depurador padrão
delete do else enum export estende false finalmente para
função se implementa a interface de importação no caso de deixar
novo pacote público protegido privado retorno super estático
alterne esse lance true try typeof var void enquanto estiver com yield
```

Não se preocupe em memorizar esta lista. Ao criar uma ligação produz um erro inesperado de sintaxe, verifique se você está tentando definir uma palavra reservada.

## O ambiente

A coleção de ligações e seus valores que existem em um determinado momento é chamada de *ambiente* . Quando um programa é iniciado, esse ambiente não está vazio. Ele sempre contém ligações que fazem parte do padrão de idioma e, na maioria das vezes, também contém ligações que fornecem maneiras de interagir com o sistema circundante. Por exemplo, em um navegador, existem funções para interagir com o site carregado no momento e ler as entradas do mouse e do teclado.

## Funções

Muitos dos valores fornecidos no ambiente padrão têm a *função* type . Uma função é um pedaço de programa envolvido em um valor. Esses valores podem ser *aplicados* para executar o programa agrupado. Por exemplo, em um ambiente de navegador, a ligação `prompt`contém uma função que mostra uma pequena caixa de diálogo solicitando a entrada do usuário. É usado assim:

```js
prompt("Enter passcode");
```

![Uma janela de prompt](C:\Users\suporte-01\Desktop\Eloquent Javascript 3ed\img\prompt.png)

A execução de uma função é chamada de *invocação* , *chamada* ou *aplicação* . Você pode chamar uma função colocando parênteses após uma expressão que produz um valor de função. Geralmente você usará diretamente o nome da ligação que contém a função. Os valores entre parênteses são dados ao programa dentro da função. No exemplo, a `prompt`função usa a string que fornecemos como texto a ser exibido na caixa de diálogo. Os valores dados às funções são chamados *argumentos* . Funções diferentes podem precisar de um número diferente ou diferentes tipos de argumentos.

A `prompt`função não é muito usada na programação moderna da Web, principalmente porque você não tem controle sobre a aparência da caixa de diálogo resultante, mas pode ser útil em programas e experimentos de brinquedo.

## A função console.log

Nos exemplos, eu costumava `console.log`gerar valores. A maioria dos sistemas JavaScript (incluindo todos os navegadores modernos e Node.js) fornece uma `console.log`função que grava seus argumentos em *algum* dispositivo de saída de texto. Nos navegadores, a saída chega ao console do JavaScript. Essa parte da interface do navegador está oculta por padrão, mas a maioria dos navegadores a abre quando você pressiona F12 ou, no Mac, comando - opção -I. Se isso não funcionar, procure nos itens um item chamado Developer Tools ou similar.

Ao executar os exemplos (ou seu próprio código) nas páginas deste livro, a `console.log`saída será mostrada após o exemplo, em vez de no console JavaScript do navegador.

```js
let x = 30;
console.log("the value of x is", x);
// → the value of x is 30
```

Embora os nomes de ligação não possam conter caracteres de ponto final, `console.log`ele possui um. Isso ocorre porque `console.log`não é uma ligação simples. Na verdade, é uma expressão que recupera a `log`propriedade do valor mantido pela `console`ligação. Vamos descobrir exatamente o que isso significa no [capítulo 4](https://eloquentjavascript.net/04_data.html#properties) .

## Retornar valores

Mostrar uma caixa de diálogo ou escrever texto na tela é um *efeito colateral* . Muitas funções são úteis devido aos efeitos colaterais que produzem. As funções também podem produzir valores; nesse caso, elas não precisam ter um efeito colateral para serem úteis. Por exemplo, a função `Math.max`pega qualquer quantidade de argumentos numéricos e devolve o máximo.

```js
console.log(Math.max(2, 4));
// → 4
```

Quando uma função produz um valor, é dito que ele *retorna* esse valor. Qualquer coisa que produza um valor é uma expressão em JavaScript, o que significa que chamadas de função podem ser usadas em expressões maiores. Aqui, uma chamada para `Math.min`, que é o oposto de `Math.max`, é usada como parte de uma expressão positiva:

```js
console.log(Math.min(2, 4) + 100);
// → 102
```

O [próximo capítulo](https://eloquentjavascript.net/03_functions.html) explica como escrever suas próprias funções.

## Controle de fluxo

Quando seu programa contém mais de uma instrução, as instruções são executadas como se fossem uma história, de cima para baixo. Este programa de exemplo possui duas instruções. O primeiro pede ao usuário um número e o segundo, que é executado após o primeiro, mostra o quadrado desse número.

```js
let theNumber = Number(prompt("Pick a number"));
console.log("Your number is the square root of " +
            theNumber * theNumber);
```

A função `Number`converte um valor em um número. Precisamos dessa conversão porque o resultado de `prompt`é um valor de sequência e queremos um número. Existem funções semelhantes chamadas `String`e `Boolean`que convertem valores para esses tipos.

Aqui está a representação esquemática trivial do fluxo de controle linear:

![Fluxo de controle trivial](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/controlflow-straight.svg)

## Execução condicional

Nem todos os programas são estradas retas. Por exemplo, podemos criar uma estrada de ramificação, onde o programa leva a ramificação apropriada com base na situação em questão. Isso é chamado de *execução condicional* .

![Fluxo de controle condicional](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/controlflow-if.svg))

A execução condicional é criada com a `if`palavra - chave em JavaScript. No caso simples, queremos que algum código seja executado se, e somente se, uma determinada condição for mantida. Podemos, por exemplo, querer mostrar o quadrado da entrada apenas se a entrada for realmente um número.

```js
let theNumber = Number(prompt("Pick a number"));
if (!Number.isNaN(theNumber)) {
  console.log("Your number is the square root of " +
              theNumber * theNumber);
}
```

Com esta modificação, se você digitar "papagaio", nenhuma saída será mostrada.

A `if`palavra-chave executa ou ignora uma instrução, dependendo do valor de uma expressão booleana. A expressão decisiva é gravada após a palavra-chave, entre parênteses, seguida pela instrução a ser executada.

A `Number.isNaN`função é uma função JavaScript padrão que retorna `true`apenas se o argumento é fornecido `NaN`. A `Number`função retorna `NaN`quando você fornece uma string que não representa um número válido. Assim, a condição se traduz em "a menos que `theNumber`não seja um número, faça isso".

A declaração após o `if`é colocada entre chaves ( `{`e `}`) neste exemplo. Os colchetes podem ser usados para agrupar qualquer número de instruções em uma única instrução, chamada de *bloco* . Você também pode tê-los omitido nesse caso, já que eles contêm apenas uma única instrução, mas para evitar ter que pensar se são necessários, a maioria dos programadores de JavaScript os usa em todas as instruções agrupadas como esta. Seguiremos principalmente essa convenção neste livro, com exceção de uma frase ocasional.

```js
if (1 + 1 == 2) console.log("It's true");
// → It's true
```

Geralmente, você não terá apenas o código que é executado quando uma condição é verdadeira, mas também o código que lida com o outro caso. Esse caminho alternativo é representado pela segunda seta no diagrama. Você pode usar a `else`palavra - chave, juntamente com `if`, para criar dois caminhos de execução alternativos separados.

```js
let theNumber = Number(prompt("Pick a number"));
if (!Number.isNaN(theNumber)) {
  console.log("Your number is the square root of " +
              theNumber * theNumber);
} else {
  console.log("Hey. Why didn't you give me a number?");
}
```

Se você tiver mais de dois caminhos para escolher, você pode “cadeia” múltiplas `if`/ `else`pares juntos. Aqui está um exemplo:

```js
let num = Number(prompt("Pick a number"));

if (num < 10) {
  console.log("Small");
} else if (num < 100) {
  console.log("Medium");
} else {
  console.log("Large");
}
```

O programa primeiro verificará se `num`é menor que 10. Se for, escolhe essa ramificação, mostra `"Small"`e está concluída. Caso contrário, é necessário o `else`ramo, que contém um segundo `if`. Se a segunda condição ( `< 100`) for mantida, isso significa que o número está entre 10 e 100 e `"Medium"`é mostrado. Caso contrário, o segundo e o último `else`ramo serão escolhidos.

O esquema para este programa é mais ou menos assim:

![Aninhado se o fluxo de controle](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/controlflow-nested-if.svg)

## enquanto e fazer loops

Considere um programa que produz todos os números pares de 0 a 12. Uma maneira de escrever isso é a seguinte:

```js
console.log(0);
console.log(2);
console.log(4);
console.log(6);
console.log(8);
console.log(10);
console.log(12);
```

Isso funciona, mas a idéia de escrever um programa é fazer com que algo *menos* funcione, não mais. Se precisássemos de todos os números pares menores que 1.000, essa abordagem seria impraticável. O que precisamos é de uma maneira de executar um pedaço de código várias vezes. Essa forma de fluxo de controle é chamada de *loop* .

![Fluxo de controle de loop](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/controlflow-loop.svg)

O fluxo de controle em loop permite voltar a algum ponto do programa em que estávamos antes e repeti-lo com o estado atual do programa. Se combinarmos isso com uma ligação que conta, podemos fazer algo assim:

```js
let number = 0;
while (number <= 12) {
  console.log(number);
  number = number + 2;
}
// → 0
// → 2
//   … etcetera
```

Uma declaração começando com a palavra-chave `while`cria um loop. A palavra `while`é seguida por uma expressão entre parênteses e depois uma declaração, muito parecida `if`. O loop continua inserindo essa instrução, desde que a expressão produza um valor que seja fornecido `true`quando convertido em booleano.

A `number`ligação demonstra como uma ligação pode acompanhar o progresso de um programa. Toda vez que o loop se repete, `number`obtém um valor 2 a mais do que o valor anterior. No início de cada repetição, é comparado com o número 12 para decidir se o trabalho do programa está concluído.

Como um exemplo que realmente faz algo útil, agora podemos escrever um programa que calcula e mostra o valor de 2 10 (2 à 10ª potência). Usamos duas ligações: uma para acompanhar nosso resultado e outra para contar quantas vezes multiplicamos esse resultado por 2. O loop testa se a segunda ligação já atingiu 10 e, se não, atualiza as duas ligações.

```js
let result = 1;
let counter = 0;
while (counter < 10) {
  result = result * 2;
  counter = counter + 1;
}
console.log(result);
// → 1024
```

O contador também poderia ter iniciado `1`e verificado `<= 10`, mas por razões que se tornarão aparentes no [Capítulo 4](https://eloquentjavascript.net/04_data.html#array_indexing) , é uma boa idéia se acostumar a contar a partir de 0.

Um `do`loop é uma estrutura de controle semelhante a um `while`loop. Ele difere apenas em um ponto: um `do`loop sempre executa seu corpo pelo menos uma vez e começa a testar se deve parar somente após a primeira execução. Para refletir isso, o teste aparece após o corpo do loop.

```js
let yourName;
do {
  yourName = prompt("Who are you?");
} while (!yourName);
console.log(yourName);
```

Este programa forçará você a inserir um nome. Ele perguntará repetidamente até obter algo que não seja uma string vazia. A aplicação do `!`operador converterá um valor no tipo booleano antes de negá-lo, e todas as cadeias, exceto a `""`conversão em `true`. Isso significa que o loop continua girando até você fornecer um nome não vazio.

## Recuo do código

Nos exemplos, adicionei espaços na frente de declarações que fazem parte de alguma declaração maior. Esses espaços não são necessários - o computador aceitará o programa sem problemas. De fato, mesmo as quebras de linha nos programas são opcionais. Você pode escrever um programa como uma única linha longa, se quiser.

O papel desse recuo dentro dos blocos é destacar a estrutura do código. No código em que novos blocos são abertos dentro de outros, pode ser difícil ver onde um bloco termina e o outro começa. Com o recuo adequado, a forma visual de um programa corresponde à forma dos blocos dentro dele. Gosto de usar dois espaços para cada bloco aberto, mas os gostos diferem - algumas pessoas usam quatro espaços e outras usam caracteres de tabulação. O importante é que cada novo bloco adicione a mesma quantidade de espaço.

```js
if (false != true) {
  console.log("That makes sense.");
  if (1 < 2) {
    console.log("No surprise there.");
  }
}
```

A maioria dos programas de edição de código (incluindo o deste livro) ajudará recuando automaticamente as novas linhas na quantidade adequada.

## Loop For

Muitos loops seguem o padrão mostrado nos `while`exemplos. Primeiro, uma ligação de "contador" é criada para rastrear o progresso do loop. Em seguida, vem um `while`loop, geralmente com uma expressão de teste que verifica se o contador atingiu seu valor final. No final do corpo do loop, o contador é atualizado para acompanhar o progresso.

Como esse padrão é muito comum, JavaScript e linguagens semelhantes fornecem uma forma um pouco mais curta e abrangente, o `for`loop.

```js
for (let number = 0; number <= 12; number = number + 2) {
  console.log(number);
}
// → 0
// → 2
//   … etcetera
```

Este programa é exatamente equivalente ao exemplo [anterior](https://eloquentjavascript.net/02_program_structure.html#loops) de impressão de números pares. A única mudança é que todas as instruções relacionadas ao "estado" do loop são agrupadas depois `for`.

Os parênteses após uma `for`palavra - chave devem conter dois pontos e vírgulas. A parte antes do primeiro ponto e vírgula *inicializa* o loop, geralmente definindo uma ligação. A segunda parte é a expressão que *verifica* se o loop deve continuar. A parte final *atualiza* o estado do loop após cada iteração. Na maioria dos casos, isso é mais curto e mais claro que uma `while`construção.

Este é o código que calcula 2 10 usando em `for`vez de `while`:

```js
let result = 1;
for (let counter = 0; counter < 10; counter = counter + 1) {
  result = result * 2;
}
console.log(result);
// → 1024
```

## Quebrando um loop

Produzir a condição de loop `false`não é a única maneira de um loop terminar. Existe uma declaração especial chamada `break`que tem o efeito de saltar imediatamente para fora do loop anexado.

Este programa ilustra a `break`declaração. Ele encontra o primeiro número que é maior ou igual a 20 e divisível por 7.

```js
for (let current = 20; ; current = current + 1) {
  if (current % 7 == 0) {
    console.log(current);
    break;
  }
}
// → 21
```

Usar o `%`operador restante ( ) é uma maneira fácil de testar se um número é divisível por outro número. Se for, o restante de sua divisão é zero.

A `for`construção no exemplo não possui uma parte que verifique o final do loop. Isso significa que o loop nunca irá parar, a menos que a `break`instrução interna seja executada.

Se você remover essa `break`instrução ou escrever acidentalmente uma condição final que sempre produz `true`, seu programa ficará preso em um *loop infinito* . Um programa preso em um loop infinito nunca terminará a execução, o que geralmente é uma coisa ruim.

Se você criar um loop infinito em um dos exemplos dessas páginas, geralmente será perguntado se deseja interromper o script após alguns segundos. Se isso falhar, você precisará fechar a guia em que está trabalhando ou, em alguns navegadores, fechar o navegador inteiro, para se recuperar.

A `continue`palavra-chave é semelhante a `break`, pois influencia o progresso de um loop. Quando `continue`é encontrado em um corpo de loop, o controle salta para fora do corpo e continua com a próxima iteração do loop.

## Atualizando Ligações Sucintamente

Especialmente ao fazer um loop, um programa geralmente precisa "atualizar" uma ligação para manter um valor com base no valor anterior dessa ligação.

```js
counter = counter + 1;
```

JavaScript fornece um atalho para isso.

```js
counter += 1;
```

Atalhos semelhantes funcionam para muitos outros operadores, como `result *= 2`dobrar `result`ou `counter -= 1`contar para baixo.

Isso nos permite encurtar um pouco mais o nosso exemplo de contagem.

```js
for (let number = 0; number <= 12; number += 2) {
  console.log(number);
}
```

Para `counter += 1`e `counter -= 1`, existem equivalentes ainda mais curtos: `counter++`e `counter--`.

## Despachando um valor com o switch

Não é incomum que o código fique assim:

```js
if (x == "value1") action1();
else if (x == "value2") action2();
else if (x == "value3") action3();
else defaultAction();
```

Existe uma construção chamada `switch`que se destina a expressar esse "despacho" de uma maneira mais direta. Infelizmente, a sintaxe que o JavaScript utiliza para isso (que é herdada da linha de linguagens de programação C / Java) é um pouco estranha - uma cadeia de `if`instruções pode parecer melhor. Aqui está um exemplo:

```js
switch (prompt("What is the weather like?")) {
  case "rainy":
    console.log("Remember to bring an umbrella.");
    break;
  case "sunny":
    console.log("Dress lightly.");
  case "cloudy":
    console.log("Go outside.");
    break;
  default:
    console.log("Unknown weather type!");
    break;
}
```

Você pode colocar qualquer número de `case`etiquetas dentro do bloco aberto por `switch`. O programa começará a executar no rótulo que corresponde ao valor que `switch`foi fornecido ou `default`se nenhum valor correspondente for encontrado. Ele continuará executando, mesmo em outros rótulos, até atingir uma `break`instrução. Em alguns casos, como no `"sunny"`caso do exemplo, isso pode ser usado para compartilhar algum código entre os casos (recomenda-se sair para o dia ensolarado e nublado). Mas tenha cuidado - é fácil esquecer isso `break`, o que fará com que o programa execute o código que você não deseja executar.

## Capitalização

Os nomes de encadernação podem não conter espaços, mas geralmente é útil usar várias palavras para descrever claramente o que a encadernação representa. Essas são as suas escolhas para escrever um nome de ligação com várias palavras:

```
fuzzylittleturtle
fuzzy_little_turtle
FuzzyLittleTurtle
fuzzyLittleTurtle
```

O primeiro estilo pode ser difícil de ler. Eu gosto bastante da aparência dos sublinhados, embora esse estilo seja um pouco doloroso para digitar. As funções padrão do JavaScript, e a maioria dos programadores de JavaScript, seguem o estilo inferior - elas capitalizam cada palavra, exceto a primeira. Não é difícil se acostumar com pequenas coisas assim, e o código com estilos de nomes mistos pode ser difícil de ler, por isso seguimos esta convenção.

Em alguns casos, como a `Number`função, a primeira letra de uma ligação também é maiúscula. Isso foi feito para marcar essa função como um construtor. O que é um construtor ficará claro no [Capítulo 6](https://eloquentjavascript.net/06_object.html#constructors) . Por enquanto, o importante não deve ser incomodado por essa aparente falta de consistência.

## Comentários

Freqüentemente, o código bruto não transmite todas as informações que você deseja que um programa transmita aos leitores humanos, ou as transmite de maneira tão enigmática que as pessoas talvez não as entendam. Em outros momentos, convém incluir algumas idéias relacionadas como parte do seu programa. É para isso que servem os *comentários* .

Um comentário é um pedaço de texto que faz parte de um programa, mas é completamente ignorado pelo computador. O JavaScript tem duas maneiras de escrever comentários. Para escrever um comentário de linha única, você pode usar dois caracteres de barra ( `//`) e depois o texto do comentário depois dele.

```js
let accountBalance = calculateBalance(account);
// It's a green hollow where a river sings
accountBalance.adjust();
// Madly catching white tatters in the grass.
let report = new Report();
// Where the sun on the proud mountain rings:
addToReport(accountBalance, report);
// It's a little valley, foaming like light in a glass.
```

Um `//`comentário vai apenas até o final da linha. Uma seção de texto entre `/*`e `*/`será ignorada por inteiro, independentemente de conter quebras de linha. Isso é útil para adicionar blocos de informações sobre um arquivo ou parte do programa.

```js
/*
  I first found this number scrawled on the back of an old notebook.
  Since then, it has often dropped by, showing up in phone numbers
  and the serial numbers of products that I've bought. It obviously
  likes me, so I've decided to keep it.
*/
const myNumber = 11213;
```

## Sumário

Agora você sabe que um programa é construído com instruções, que às vezes contêm mais instruções. As instruções tendem a conter expressões, que podem ser construídas a partir de expressões menores.

A colocação de instruções uma após a outra fornece um programa que é executado de cima para baixo. Você pode introduzir perturbações no fluxo de controle usando condicional ( `if`, `else`e `switch`) e looping ( `while`, `do`e `for`declarações).

As ligações podem ser usadas para arquivar dados sob um nome e são úteis para rastrear o estado do seu programa. O ambiente é o conjunto de ligações definidas. Os sistemas JavaScript sempre colocam várias ligações padrão úteis em seu ambiente.

Funções são valores especiais que encapsulam uma parte do programa. Você pode invocá-los escrevendo `functionName(argument1, argument2)`. Essa chamada de função é uma expressão e pode produzir um valor.

## Exercícios

Se você não souber como testar suas soluções para os exercícios, consulte a [Introdução](https://eloquentjavascript.net/00_intro.html) .

Cada exercício começa com uma descrição do problema. Leia esta descrição e tente resolver o exercício. Se você tiver problemas, considere ler as dicas após o exercício. As soluções completas para os exercícios não estão incluídas neste livro, mas você pode encontrá-las online em [*https://eloquentjavascript.net/code*](https://eloquentjavascript.net/code#2) . Se você quiser aprender alguma coisa com os exercícios, recomendo analisar as soluções somente depois de resolvê-lo, ou pelo menos depois de atacá-lo por tempo suficiente para ter uma leve dor de cabeça.

### Dar laços em um triângulo

Escreva um loop que faça sete chamadas `console.log`para gerar o seguinte triângulo:

```
#
##
###
####
#####
######
#######
```

Pode ser útil saber que você pode encontrar o comprimento de uma string escrevendo `.length`depois dela.

```js
let abc = "abc";
console.log(abc.length);
// → 3
```

A maioria dos exercícios contém um código que você pode modificar para resolver o exercício. Lembre-se de que você pode clicar nos blocos de código para editá-los.

```js
// Your code here.
```

Você pode começar com um programa que imprima os números de 1 a 7, que você pode derivar fazendo algumas modificações no [exemplo de impressão de números pares,](https://eloquentjavascript.net/02_program_structure.html#loops) fornecido anteriormente no capítulo, onde o `for`loop foi introduzido.

Agora considere a equivalência entre números e seqüências de caracteres de hash. Você pode passar de 1 para 2 adicionando 1 ( `+= 1`). Você pode ir de `"#"`para `"##"`adicionando um caractere ( `+= "#"`). Assim, sua solução pode acompanhar de perto o programa de impressão numérica.

### FizzBuzz

Escreva um programa que use `console.log`para imprimir todos os números de 1 a 100, com duas exceções. Para números divisíveis por 3, imprima em `"Fizz"`vez do número e, para números divisíveis por 5 (e não 3), imprima `"Buzz"`.

Quando você tiver esse trabalho, modifique seu programa para imprimir `"FizzBuzz"`números divisíveis por 3 e 5 (e ainda imprimir `"Fizz"`ou `"Buzz"`para números divisíveis por apenas um deles).

(Na verdade, essa é uma pergunta de entrevista que eliminou uma porcentagem significativa de candidatos a programadores. Portanto, se você a resolveu, seu valor no mercado de trabalho subiu.)

```js
// Your code here.
```

Examinar os números é claramente um trabalho em loop, e selecionar o que imprimir é uma questão de execução condicional. Lembre-se do truque de usar o `%`operador restante ( ) para verificar se um número é divisível por outro número (tem o restante zero).

Na primeira versão, há três resultados possíveis para cada número; portanto, você precisará criar uma cadeia `if`/ `else if`/ `else`.

A segunda versão do programa tem uma solução simples e inteligente. A solução simples é adicionar outro "ramo" condicional para testar com precisão a condição especificada. Para uma solução inteligente, crie uma sequência contendo a palavra ou palavras a serem impressas e imprima essa palavra ou o número, se não houver uma palavra, potencialmente fazendo bom uso do `||`operador.

### Tabuleiro de xadrez

Escreva um programa que crie uma string que represente uma grade 8 × 8, usando caracteres de nova linha para separar linhas. Em cada posição da grade existe um espaço ou um caractere "#". Os personagens devem formar um tabuleiro de xadrez.

Passar essa string para `console.log`deve mostrar algo como isto:

```
 # # # #
# # # # 
 # # # #
# # # # 
 # # # #
# # # # 
 # # # #
# # # #
```

Quando você tiver um programa que gere esse padrão, defina uma ligação `size = 8`e altere o programa para que funcione para qualquer um `size`, produzindo uma grade da largura e altura especificadas.

```js
// Your code here.
```

Você pode criar a string iniciando com uma vazia ( `""`) e adicionando caracteres repetidamente. Um caractere de nova linha é gravado `"\n"`.

Para trabalhar com duas dimensões, você precisará de um loop dentro de um loop. Coloque chaves nos corpos dos dois loops para facilitar a visualização de onde eles começam e terminam. Tente recuar adequadamente esses corpos. A ordem dos loops deve seguir a ordem em que construímos a string (linha por linha, da esquerda para a direita, de cima para baixo). Portanto, o loop externo lida com as linhas e o loop interno lida com os caracteres de uma linha.

Você precisará de duas associações para acompanhar seu progresso. Para saber se deve colocar um espaço ou um sinal de hash em uma determinada posição, você pode testar se a soma dos dois contadores é par ( `% 2`).

O término de uma linha adicionando um caractere de nova linha deve ocorrer após a construção da linha. Faça isso após o loop interno, mas dentro do loop externo.