# Capítulo 1 Valores, tipos e operadores

> Abaixo da superfície da máquina, o programa se move. Sem esforço, ele se expande e contrai. Em grande harmonia, os elétrons se dispersam e se reagrupam. As formas no monitor são apenas ondulações na água. A essência permanece invisivelmente abaixo.
>
> *—Mestre Yuan-Ma, O Livro de Programação*

![Imagem de um mar de bits](https://eloquentjavascript.net/img/chapter_picture_1.jpg)

Dentro do mundo do computador, existem apenas dados. Você pode ler dados, modificar dados, criar novos dados - mas o que não é um dado não pode ser mencionado. Todos esses dados são armazenados como longas sequências de bits e, portanto, são fundamentalmente semelhantes.

*Bits* são qualquer tipo de coisa de dois valores, geralmente descrita como zeros e uns. Dentro do computador, eles assumem formas como carga elétrica alta ou baixa, sinal forte ou fraco ou um ponto brilhante ou opaco na superfície de um CD. Qualquer informação discreta pode ser reduzida a uma sequência de zeros e uns e, portanto, representada em bits.

Por exemplo, podemos expressar o número 13 em bits. Funciona da mesma maneira que um número decimal, mas em vez de 10 dígitos diferentes, você tem apenas 2, e o peso de cada um aumenta por um fator de 2 da direita para a esquerda. Aqui estão os bits que compõem o número 13, com os pesos dos dígitos mostrados abaixo deles:

```
   0 0 0 0 1 1 0 1
 128 64 32 16 8 4 2 1
```

Esse é o número binário 00001101. Seus dígitos diferentes de zero representam 8, 4 e 1 e somam 13.

## Valores

Imagine um mar de pedaços - um oceano deles. Um computador moderno típico possui mais de 30 bilhões de bits em seu armazenamento volátil de dados (memória de trabalho). O armazenamento não volátil (o disco rígido ou equivalente) tende a ter ainda mais algumas ordens de magnitude.

Para poder trabalhar com essas quantidades de bits sem nos perdermos, precisamos separá-los em partes que representam informações. Em um ambiente JavaScript, esses blocos são chamados de *valores* . Embora todos os valores sejam feitos de bits, eles desempenham papéis diferentes. Todo valor tem um tipo que determina seu papel. Alguns valores são números, alguns valores são partes do texto, alguns valores são funções e assim por diante.

Para criar um valor, você deve apenas chamar seu nome. Isso é conveniente. Você não precisa reunir material de construção para seus valores ou pagar por eles. Você apenas pede um e , *whoosh* , você o tem. Eles não são realmente criados do nada, é claro. Todo valor deve ser armazenado em algum lugar e, se você quiser usar uma quantidade gigantesca deles ao mesmo tempo, poderá ficar sem memória. Felizmente, isso é um problema apenas se você precisar deles todos simultaneamente. Assim que você não usar mais um valor, ele se dissipará, deixando para trás seus bits para serem reciclados como material de construção para a próxima geração de valores.

Este capítulo apresenta os elementos atômicos dos programas JavaScript, ou seja, os tipos de valores simples e os operadores que podem agir sobre esses valores.

## Números

Os valores do tipo de *número* são, sem surpresa, valores numéricos. Em um programa JavaScript, eles são escritos da seguinte maneira:

```js
13
```

Use isso em um programa e fará com que o padrão de bits para o número 13 venha a existir na memória do computador.

O JavaScript usa um número fixo de bits, 64 deles, para armazenar um único valor numérico. Existem apenas tantos padrões que você pode criar com 64 bits, o que significa que o número de diferentes números que podem ser representados é limitado. Com *N* dígitos decimais, você pode representar 10 N números. Da mesma forma, com 64 dígitos binários, você pode representar 2 64 números diferentes, ou seja, cerca de 18 quintilhões (uns 18 com 18 zeros depois). Isso é muito.

A memória do computador costumava ser muito menor e as pessoas costumavam usar grupos de 8 ou 16 bits para representar seus números. Foi fácil *estourar* acidentalmente números tão pequenos - acabar com um número que não se encaixava no número de bits especificado. Hoje, mesmo os computadores que cabem no seu bolso têm muita memória, então você é livre para usar blocos de 64 bits e precisa se preocupar com o transbordamento apenas ao lidar com números verdadeiramente astronômicos.

Porém, nem todos os números inteiros com menos de 18 quintilhões se encaixam em um número JavaScript. Esses bits também armazenam números negativos, portanto, um bit indica o sinal do número. Uma questão maior é que números não integrais também devem ser representados. Para fazer isso, alguns dos bits são usados para armazenar a posição do ponto decimal. O número inteiro máximo real que pode ser armazenado é maior que 9 quadrilhões (15 zeros) - o que ainda é agradavelmente grande.

Os números fracionários são escritos usando um ponto.

```js
9,81
```

Para números muito grandes ou muito pequenos, você também pode usar notação científica adicionando um *e* (para *expoente* ), seguido pelo expoente do número.

```js
2.998e8
```

Ou seja, 2.998 × 10 8 = 299.800.000.

Cálculos com números inteiros (também chamados de *números inteiros* ) menores que os 9 quadrilhões mencionados acima são garantidos para serem sempre precisos. Infelizmente, os cálculos com números fracionários geralmente não são. Assim como π (pi) não pode ser expresso com precisão por um número finito de dígitos decimais, muitos números perdem alguma precisão quando apenas 64 bits estão disponíveis para armazená-los. É uma pena, mas causa problemas práticos apenas em situações específicas. O importante é estar ciente disso e tratar os números digitais fracionários como aproximações, não como valores precisos.

### Aritmética

A principal coisa a fazer com os números é aritmética. Operações aritméticas como adição ou multiplicação pegam dois valores numéricos e produzem um novo número a partir deles. Aqui está a aparência deles em JavaScript:

```js
100  +  4  *  11
```

Os símbolos `+`e `*`são chamados *operadores* . O primeiro significa adição e o segundo significa multiplicação. Colocar um operador entre dois valores o aplicará a esses valores e produzirá um novo valor.

Mas o exemplo significa "adicione 4 e 100 e multiplique o resultado por 11" ou a multiplicação é feita antes da adição? Como você deve ter adivinhado, a multiplicação acontece primeiro. Mas, como na matemática, você pode alterar isso colocando a adição entre parênteses.

```js
( 100  +  4 ) *  11
```

Para subtração, existe o `-`operador, e a divisão pode ser feita com o `/`operador.

Quando os operadores aparecem juntos sem parênteses, a ordem em que são aplicados é determinada pela *precedência* dos operadores. O exemplo mostra que a multiplicação vem antes da adição. O `/`operador tem a mesma precedência que `*`. Da mesma forma para `+`e `-`. Quando vários operadores com a mesma precedência aparecem ao lado do outro, como em `1 - 2 + 1`, eles são aplicados esquerda para a direita: `(1 - 2) + 1`.

Essas regras de precedência não são algo com que você deve se preocupar. Em caso de dúvida, basta adicionar parênteses.

Há mais um operador aritmético, que você pode não reconhecer imediatamente. O `%`símbolo é usado para representar a operação *restante* . `X % Y`é o restante da divisão `X`por `Y`. Por exemplo, `314 % 100`produz `14`e `144 % 12`dá `0`. A precedência do operador restante é a mesma da multiplicação e divisão. Você também verá esse operador com frequência chamado *módulo* .

### Números especiais

Existem três valores especiais no JavaScript que são considerados números, mas não se comportam como números normais.

Os dois primeiros são `Infinity`e `-Infinity`, que representam os infinitos positivo e negativo. `Infinity - 1`ainda está `Infinity`e assim por diante. Não confie demais na computação baseada no infinito. Não é matematicamente correcto, e vai levar rapidamente para o próximo número especial: `NaN`.

`NaN`significa "não é um número", mesmo que *seja* um valor do tipo de número. Você obterá esse resultado quando, por exemplo, tentar calcular `0 / 0`(zero dividido por zero) `Infinity - Infinity`ou qualquer número de outras operações numéricas que não produzam um resultado significativo.

## Strings

O próximo tipo de dados básico é a *sequência* . Strings são usadas para representar texto. Eles são escritos colocando o conteúdo entre aspas.

```js
`Down on the sea`
"Lie on the ocean"
'Float on the ocean'
```

Você pode usar aspas simples, aspas duplas ou reticulares para marcar as strings, desde que as aspas no início e no final da string sejam correspondentes.

Quase tudo pode ser colocado entre aspas, e o JavaScript criará um valor de string. Mas alguns personagens são mais difíceis. Você pode imaginar como pode ser difícil colocar aspas entre aspas. *As novas linhas* (os caracteres que você obtém ao pressionar enter ) podem ser incluídas sem escapar apenas quando a string é citada com reticências ( ```).

Para possibilitar a inclusão desses caracteres em uma seqüência de caracteres, a seguinte notação é usada: sempre que uma barra invertida ( `\`) é encontrada no texto entre aspas, isso indica que o caractere após um significado especial. Isso é chamado de *escape* do personagem. Uma citação precedida por uma barra invertida não terminará a sequência, mas fará parte dela. Quando um `n`caractere ocorre após uma barra invertida, ele é interpretado como uma nova linha. Da mesma forma, `t`após uma barra invertida significa um caractere de tabulação. Pegue a seguinte string:

```js
"This is the first line\nAnd this is the second"
```

O texto real contido é este:

This is the first line
And this is the second

Obviamente, há situações em que você deseja que uma barra invertida em uma string seja apenas uma barra invertida, não um código especial. Se duas barras invertidas se sucederem, elas serão recolhidas juntas e apenas uma será deixada no valor resultante da string. É assim que a string “ *Um caractere de nova linha é escrita como `"`\ n `"`.* " pode ser expressada:

```js
"A newline character is written like \"\\n\"."
```

As strings também precisam ser modeladas como uma série de bits para poder existir dentro do computador. A maneira como o JavaScript faz isso é baseada no padrão *Unicode* . Esse padrão atribui um número a praticamente todos os caracteres de que você precisa, incluindo caracteres do grego, árabe, japonês, armênio etc. Se tivermos um número para cada caractere, uma sequência poderá ser descrita por uma sequência de números.

E é isso que o JavaScript faz. Mas há uma complicação: a representação do JavaScript usa 16 bits por elemento de string, que pode descrever até 2 16 caracteres diferentes. Mas o Unicode define mais caracteres que isso - cerca do dobro, neste momento. Portanto, alguns caracteres, como muitos emoji, ocupam duas "posições de caracteres" em strings JavaScript. Voltaremos a isso no [capítulo 5](https://eloquentjavascript.net/05_higher_order.html#code_units) .

As seqüências não podem ser divididas, multiplicadas ou subtraídas, mas o `+`operador *pode* ser usado nelas. Não acrescenta, mas *concatena* - cola duas strings. A seguinte linha produzirá a sequência `"concatenate"`:

```js
"con" + "cat" + "e" + "nate"
```

Os valores de sequência possuem várias funções ( *métodos* ) associadas que podem ser usadas para executar outras operações nelas. Vou falar mais sobre isso no [capítulo 4](https://eloquentjavascript.net/04_data.html#methods) .

As strings escritas com aspas simples ou duplas se comportam da mesma maneira - a única diferença é em que tipo de aspas você precisa escapar dentro delas. Seqüências de caracteres entre aspas, normalmente chamadas de *modelo literal* , podem fazer mais alguns truques. Além de poderem estender linhas, eles também podem incorporar outros valores.

```js
`half of 100 is ${100 / 2}`
```

Quando você escreve algo dentro `${}`de um literal de modelo, seu resultado será calculado, convertido em uma string e incluído nessa posição. O exemplo produz " *metade de 100 é 50* ".

## Operadores unários

Nem todos os operadores são símbolos. Alguns são escritos como palavras. Um exemplo é o `typeof`operador, que produz um valor de sequência nomeando o tipo de valor que você atribui.

```js
console.log(typeof 4.5)
// → number
console.log(typeof "x")
// → string
```

Usaremos `console.log`no código de exemplo para indicar que queremos ver o resultado da avaliação de algo. Mais sobre isso no [próximo capítulo](https://eloquentjavascript.net/02_program_structure.html) .

Os outros operadores mostrados operavam com dois valores, mas `typeof`usam apenas um. Os operadores que usam dois valores são chamados de operadores *binários* , enquanto os que usam um são chamados de operadores *unários* . O operador negativo pode ser usado como um operador binário e como um operador unário.

```js
console.log(- (10 - 2))
// → -8
```

## Valores booleanos

Muitas vezes, é útil ter um valor que distinga entre apenas duas possibilidades, como "sim" e "não" ou "ligado" e "desligado". Para esse propósito, o JavaScript tem um tipo *booleano* , que possui apenas dois valores, verdadeiro e falso, que são escritos como essas palavras.

### Comparação

Aqui está uma maneira de produzir valores booleanos:

```js
console.log(3 > 2)
// → true
console.log(3 < 2)
// → false
```

Os sinais `>`e `<`são os símbolos tradicionais de "é maior que" e "é menor que", respectivamente. Eles são operadores binários. A aplicação deles resulta em um valor booleano que indica se eles são verdadeiros nesse caso.

As strings podem ser comparadas da mesma maneira.

```js
console.log("Aardvark" < "Zoroaster")
// → true
```

A maneira como as strings são ordenadas é mais ou menos alfabética, mas não é exatamente o que você esperaria ver em um dicionário: letras maiúsculas são sempre "menos" que letras minúsculas, portanto `"Z" < "a"`, caracteres não alfabéticos (!, - e assim por diante) também estão incluídos na encomenda. Ao comparar seqüências de caracteres, o JavaScript passa os caracteres da esquerda para a direita, comparando os códigos Unicode um por um.

Outros operadores semelhantes são `>=`(maior ou igual a), `<=`(menor ou igual a), `==`(igual a) e `!=`(não igual a).

```js
console.log("Itchy" != "Scratchy")
// → true
console.log("Apple" == "Orange")
// → false
```

Existe apenas um valor no JavaScript que não é igual a si mesmo e é `NaN`("não é um número").

```js
console.log(NaN == NaN)
// → false
```

`NaN`deve denotar o resultado de uma computação sem sentido e, como tal, não é igual ao resultado de *outras* computações sem sentido.

### Operadores lógicos

Existem também algumas operações que podem ser aplicadas aos próprios valores booleanos. O JavaScript suporta três operadores lógicos: *e* , *ou* , e *não* . Eles podem ser usados para "raciocinar" sobre os booleanos.

O `&&`operador representa lógico *e* . É um operador binário e seu resultado é verdadeiro apenas se os dois valores fornecidos a ele forem verdadeiros.

```js
console.log(true && false)
// → false
console.log(true && true)
// → true
```

O `||`operador indica lógico *ou* . Produz true se qualquer um dos valores dados a ela for true.

```js
console.log(false || true)
// → true
console.log(false || false)
// → false
```

*Not* é escrito como um ponto de exclamação ( `!`). É um operador unário que inverte o valor que lhe é dado - `!true`produz `false`e `!false`dá `true`.

Ao misturar esses operadores booleanos com aritméticos e outros, nem sempre é óbvio quando são necessários parênteses. Na prática, geralmente você pode obter com sabendo que os operadores que vimos até agora, `||`tem a menor precedência, então vem `&&`, então os operadores de comparação ( `>`, `==`e assim por diante), e depois o resto. Essa ordem foi escolhida de forma que, em expressões típicas como a seguinte, o menor número possível de parênteses seja necessário:

```js
1 + 1 == 2 && 10 * 10 > 50
```

O último operador lógico que discutirei não é unário, não binário, mas *ternário* , operando em três valores. Está escrito com um ponto de interrogação e dois pontos, assim:

```js
console.log(true ? 1 : 2);
// → 1
console.log(false ? 1 : 2);
// → 2
```

Esse é chamado de operador *condicional* (ou às vezes apenas o operador *ternário* , pois é o único operador desse tipo no idioma). O valor à esquerda do ponto de interrogação “escolhe” qual dos outros dois valores sairá. Quando é verdade, escolhe o valor do meio e, quando é falso, escolhe o valor à direita.

## Valores vazios

Existem dois valores especiais, escritos `null`e `undefined`, que são usados para denotar a ausência de um valor *significativo* . Eles próprios são valores, mas não carregam informações.

Muitas operações no idioma que não produzem um valor significativo (você verá mais adiante) resultam `undefined`simplesmente porque precisam gerar *algum* valor.

A diferença de significado entre `undefined`e `null`é um acidente do design do JavaScript, e isso não importa na maioria das vezes. Nos casos em que você realmente precisa se preocupar com esses valores, recomendo tratá-los como principalmente intercambiáveis.

## Conversão automática de tipo

Na introdução, mencionei que o JavaScript se esforça para aceitar quase todos os programas que você oferece, mesmo programas que fazem coisas estranhas. Isso é bem demonstrado pelas seguintes expressões:

```js
console.log(8 * null)
// → 0
console.log("5" - 1)
// → 4
console.log("5" + 1)
// → 51
console.log("five" * 2)
// → NaN
console.log(false == 0)
// → true
```

Quando um operador é aplicado ao tipo de valor "errado", o JavaScript converte discretamente esse valor para o tipo necessário, usando um conjunto de regras que geralmente não são o que você deseja ou espera. Isso é chamado de *coerção de tipo* . A `null`primeira expressão se torna `0`e `"5"`a segunda expressão se torna `5`(da string para o número). No entanto, na terceira expressão, `+`tenta a concatenação de cadeias antes da adição numérica, para que `1`seja convertida em `"1"`(de número para cadeia).

Quando algo que não é mapeado para um número de maneira óbvia (como `"five"`ou `undefined`) é convertido em um número, você obtém o valor `NaN`. Outras operações aritméticas `NaN`continuam produzindo `NaN`; portanto, se você encontrar uma delas em um local inesperado, procure conversões acidentais de tipo.

Ao comparar valores do mesmo tipo usando `==`, é fácil prever o resultado: você deve ser verdadeiro quando os dois valores forem iguais, exceto no caso de `NaN`. Mas quando os tipos diferem, o JavaScript usa um conjunto de regras complicado e confuso para determinar o que fazer. Na maioria dos casos, apenas tenta converter um dos valores para o tipo do outro valor. No entanto, quando `null`ou `undefined`ocorre em ambos os lados do operador, ele produz true somente se os dois lados forem um de `null`ou `undefined`.

```js
console.log(null == undefined);
// → true
console.log(null == 0);
// → false
```

Esse comportamento é frequentemente útil. Quando você deseja testar se um valor tem um valor real em vez de `null`ou `undefined`, você pode compará-lo `null`com o operador `==`(ou `!=`).

Mas e se você quiser testar se algo se refere ao valor exato `false`? Expressões como `0 == false`e `"" == false`também são verdadeiras devido à conversão automática de tipo. Quando você *não* deseja que nenhuma conversão de tipo aconteça, existem dois operadores adicionais: `===`e `!==`. O primeiro testa se um valor é *exatamente* igual ao outro e o segundo testa se não é exatamente igual. Então, `"" === false`é falso como esperado.

Eu recomendo usar os operadores de comparação de três caracteres na defensiva para impedir conversões inesperadas de tipo. Mas quando você tiver certeza de que os tipos de ambos os lados serão os mesmos, não haverá problema em usar os operadores mais curtos.

### Curto-circuito dos operadores lógicos

Os operadores lógicos `&&`e `||`lidam com valores de diferentes tipos de uma maneira peculiar. Eles converterão o valor no lado esquerdo para o tipo booleano para decidir o que fazer, mas, dependendo do operador e do resultado dessa conversão, retornarão o valor *original à* esquerda ou o valor à direita.

O `||`operador, por exemplo, retornará o valor à sua esquerda quando isso puder ser convertido em true e, caso contrário, retornará o valor à sua direita. Isso tem o efeito esperado quando os valores são booleanos e faz algo análogo a valores de outros tipos.

```js
console.log(null || "user")
// → user
console.log("Agnes" || "user")
// → Agnes
```

Podemos usar essa funcionalidade como uma maneira de recuperar um valor padrão. Se você tiver um valor que possa estar vazio, poderá colocá- `||`lo com um valor de substituição. Se o valor inicial puder ser convertido em falso, você receberá a substituição. As regras para a conversão de strings e números para booleana valores de estado que `0`, `NaN`e a cadeia vazia ( `""`) contam como `false`, enquanto todos os outros valores contar como `true`. Então , `0 || -1`produz `-1`e `"" || "!?"`produz `"!?"`.

O `&&`operador trabalha da mesma forma, mas o contrário. Quando o valor à esquerda é algo que se converte em false, ele retorna esse valor e, caso contrário, retorna o valor à direita.

Outra propriedade importante desses dois operadores é que a peça à direita é avaliada apenas quando necessário. No caso de `true || X`, não importa o que `X`seja - mesmo que seja um programa que faça algo *terrível* - o resultado será verdadeiro e `X`nunca será avaliado. O mesmo vale para `false && X`, que é falso e será ignorado `X`. Isso é chamado *de avaliação de curto-circuito* .

O operador condicional trabalha de maneira semelhante. Dos segundo e terceiro valores, apenas o que é selecionado é avaliado.

## Sumário

Analisamos quatro tipos de valores JavaScript neste capítulo: números, seqüências de caracteres, booleanos e valores indefinidos.

Esses valores são criados digitando seu nome ( `true`, `null`) ou valor ( `13`, `"abc"`). Você pode combinar e transformar valores com operadores. Vimos operadores binários para a aritmética ( `+`, `-`, `*`, `/`, e `%`), concatenação ( `+`), a comparação ( `==`, `!=`, `===`, `!==`, `<`, `>`, `<=`, `>=`) e lógica ( `&&`, `||`), bem como vários operadores unários ( `-`para negar um número, `!`para negar logicamente e `typeof`para encontrar o tipo de um valor) e um operador ternário ( `?:`) para escolher um dos dois valores com base em um terceiro valor.

Isso fornece informações suficientes para usar o JavaScript como uma calculadora de bolso, mas não muito mais. O [próximo capítulo](https://eloquentjavascript.net/02_program_structure.html) começará a vincular essas expressões em programas básicos.