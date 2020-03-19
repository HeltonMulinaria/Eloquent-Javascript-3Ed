# Capítulo 5 Funções de ordem superior

> Tzu-li e Tzu-ssu estavam se gabando do tamanho de seus programas mais recentes. 'Duzentas mil linhas', disse Tzu-li, 'sem contar os comentários!' Tzu-ssu respondeu: 'Pssh, o meu já é quase um *milhão de* linhas'. Mestre Yuan-Ma disse: 'Meu melhor programa tem quinhentas linhas'. Ao ouvir isso, Tzu-li e Tzu-ssu foram esclarecidos.
>
> *—Mestre Yuan-Ma, O Livro de Programação*

> Existem duas maneiras de construir um design de software: uma maneira é tornar tão simples que obviamente não há deficiências, e a outra maneira é torná-lo tão complicado que não há deficiências óbvias.
>
> *—CAR Hoare, 1980 Palestra do Prêmio ACM Turing*

![Cartas de diferentes scripts](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/chapter_picture_5.jpg)

Um programa grande é um programa caro, e não apenas pelo tempo que leva para construir. Tamanho quase sempre envolve complexidade, e complexidade confunde os programadores. Programadores confusos, por sua vez, introduzem erros ( *bugs* ) nos programas. Um programa grande oferece muito espaço para ocultar esses bugs, dificultando sua localização.

Vamos voltar brevemente aos dois últimos exemplos de programas na introdução. O primeiro é independente e tem seis linhas.

```js
let total = 0, count = 1;
while (count <= 10) {
  total += count;
  count += 1;
}
console.log(total);
```

O segundo se baseia em duas funções externas e tem uma linha de comprimento.

```js
console.log(sum(range(1, 10)));
```

Qual deles tem maior probabilidade de conter um bug?

Se contarmos o tamanho das definições de `sum`e `range`, o segundo programa também será grande - ainda maior que o primeiro. Mas, ainda assim, eu argumentaria que é mais provável que esteja correto.

É mais provável que esteja correto porque a solução é expressa em um vocabulário que corresponde ao problema que está sendo resolvido. Somando um intervalo de números não se trata de loops e contadores. É sobre intervalos e somas.

As definições desse vocabulário (as funções `sum`e `range`) ainda envolverão loops, contadores e outros detalhes incidentais. Mas, como estão expressando conceitos mais simples que o programa como um todo, é mais fácil acertar.

## Abstração

No contexto da programação, esses tipos de vocabulários são geralmente chamados de *abstrações* . As abstrações ocultam detalhes e nos permitem falar sobre problemas em um nível mais alto (ou mais abstrato).

Como analogia, compare essas duas receitas para sopa de ervilha. O primeiro é assim:

> Coloque 1 xícara de ervilhas secas por pessoa em um recipiente. Adicione água até que as ervilhas estejam bem cobertas. Deixe as ervilhas em água por pelo menos 12 horas. Retire as ervilhas da água e coloque-as em uma panela. Adicione 4 xícaras de água por pessoa. Tampe a panela e mantenha as ervilhas fervendo por duas horas. Tome meia cebola por pessoa. Corte em pedaços com uma faca. Adicione-o às ervilhas. Pegue um talo de aipo por pessoa. Corte em pedaços com uma faca. Adicione-o às ervilhas. Pegue uma cenoura por pessoa. Corte em pedaços. Com uma faca! Adicione-o às ervilhas. Cozinhe por mais 10 minutos.

E esta é a segunda receita:

> Por pessoa: 1 xícara de ervilhas secas, meia cebola picada, um talo de aipo e uma cenoura.
>
> Deixe as ervilhas de molho por 12 horas. Cozinhe por 2 horas em 4 xícaras de água (por pessoa). Pique e adicione os legumes. Cozinhe por mais 10 minutos.

O segundo é mais curto e fácil de interpretar. Mas você precisa entender mais algumas palavras relacionadas à culinária, como *molho* , *cozinhe* , *pique* e, eu acho, *vegetal* .

Ao programar, não podemos confiar em todas as palavras que precisamos estar esperando por nós no dicionário. Assim, podemos cair no padrão da primeira receita - elaborar as etapas precisas que o computador deve executar, uma a uma, cego aos conceitos de nível superior que eles expressam.

É uma habilidade útil, na programação, perceber quando você está trabalhando em um nível de abstração muito baixo.

## Abstraindo repetição

Funções simples, como as vimos até agora, são uma boa maneira de criar abstrações. Mas às vezes eles ficam aquém.

É comum que um programa faça algo um determinado número de vezes. Você pode escrever um `for`loop para isso, assim:

```js
for (let i = 0; i < 10; i++) {
  console.log(i);
}
```

Podemos abstrair "fazer algo *N* vezes" como uma função? Bem, é fácil escrever uma função que chama `console.log` *N* vezes.

```js
function repeatLog(n) {
  for (let i = 0; i < n; i++) {
    console.log(i);
  }
}
```

Mas e se quisermos fazer algo diferente de registrar os números? Como “fazer algo” pode ser representado como uma função e funções são apenas valores, podemos passar nossa ação como um valor de função.

```js
function repeat(n, action) {
  for (let i = 0; i < n; i++) {
    action(i);
  }
}

repeat(3, console.log);
// → 0
// → 1
// → 2
```

Não precisamos passar uma função predefinida para `repeat`. Frequentemente, é mais fácil criar um valor de função no local.

```js
let labels = [];
repeat(5, i => {
  labels.push(`Unit ${i + 1}`);
});
console.log(labels);
// → ["Unit 1", "Unit 2", "Unit 3", "Unit 4", "Unit 5"]
```

Isso é estruturado um pouco como um `for`loop - primeiro descreve o tipo de loop e, em seguida, fornece um corpo. No entanto, o corpo agora é gravado como um valor de função, que é colocado entre parênteses da chamada para `repeat`. É por isso que ele deve ser fechado com a chave de fechamento *e o* parêntese de fechamento. Em casos como este exemplo, em que o corpo é uma única expressão pequena, você também pode omitir os chavetas e escrever o loop em uma única linha.

## Funções de ordem superior

Funções que operam em outras funções, tomando-as como argumentos ou retornando-as, são chamadas *de funções de ordem superior* . Como já vimos que funções são valores regulares, não há nada particularmente notável no fato de que tais funções existem. O termo vem da matemática, onde a distinção entre funções e outros valores é levada mais a sério.

Funções de ordem superior nos permitem abstrair *ações* , não apenas valores. Eles vêm de várias formas. Por exemplo, podemos ter funções que criam novas funções.

```js
function greaterThan(n) {
  return m => m > n;
}
let greaterThan10 = greaterThan(10);
console.log(greaterThan10(11));
// → true
```

E podemos ter funções que mudam outras funções.

```js
function noisy(f) {
  return (...args) => {
    console.log("calling with", args);
    let result = f(...args);
    console.log("called with", args, ", returned", result);
    return result;
  };
}
noisy(Math.min)(3, 2, 1);
// → calling with [3, 2, 1]
// → called with [3, 2, 1] , returned 1
```

Podemos até escrever funções que fornecem novos tipos de fluxo de controle.

```js
function unless(test, then) {
  if (!test) then();
}

repeat(3, n => {
  unless(n % 2 == 1, () => {
    console.log(n, "is even");
  });
});
// → 0 is even
// → 2 is even
```

Existe um método de matriz interno,, `forEach`que fornece algo como um loop `for`/ `of`como uma função de ordem superior.

```js
["A", "B"].forEach(l => console.log(l));
// → A
// → B
```

## Conjunto de dados de script

Uma área em que funções de ordem superior brilham é o processamento de dados. Para processar dados, precisaremos de alguns dados reais. Este capítulo usará um conjunto de dados sobre scripts - sistemas de escrita como latim, cirílico ou árabe.

Você se lembra do Unicode do [capítulo 1](https://eloquentjavascript.net/01_values.html#unicode) , o sistema que atribui um número a cada caractere na linguagem escrita? A maioria desses caracteres está associada a um script específico. O padrão contém 140 scripts diferentes - 81 ainda estão em uso hoje e 59 são históricos.

Embora eu possa ler fluentemente apenas caracteres latinos, aprecio o fato de as pessoas estarem escrevendo textos em pelo menos 80 outros sistemas de escrita, muitos dos quais eu nem reconheceria. Por exemplo, aqui está uma amostra da caligrafia tâmil:

![Caligrafia tâmil](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/tamil.png)

O conjunto de dados de exemplo contém algumas informações sobre os 140 scripts definidos no Unicode. Está disponível na [sandbox de codificação](https://eloquentjavascript.net/code#5) deste capítulo como `SCRIPTS`encadernação. A ligação contém uma matriz de objetos, cada um dos quais descreve um script.

```js
{
  name: "Coptic",
  ranges: [[994, 1008], [11392, 11508], [11513, 11520]],
  direction: "ltr",
  year: -200,
  living: false,
  link: "https://en.wikipedia.org/wiki/Coptic_alphabet"
}
```

Esse objeto nos diz o nome do script, os intervalos Unicode atribuídos a ele, a direção na qual ele é gravado, o tempo de origem (aproximado), se ele ainda está em uso e um link para mais informações. A direção pode ser `"ltr"`da esquerda para a direita, `"rtl"`da direita para a esquerda (da maneira como os textos em árabe e hebraico são escritos) ou `"ttb"`de cima para baixo (como na escrita mongol).

A `ranges`propriedade contém uma matriz de intervalos de caracteres Unicode, cada um dos quais é uma matriz de dois elementos que contém um limite inferior e um limite superior. Quaisquer códigos de caracteres dentro desses intervalos são atribuídos ao script. O limite inferior é inclusivo (o código 994 é um caractere copta) e o limite superior é não inclusivo (o código 1008 não é).

## Filtrando matrizes

Para localizar os scripts no conjunto de dados que ainda estão em uso, a seguinte função pode ser útil. Ele filtra os elementos em uma matriz que não passam no teste.

```js
 function filter(array, test) {
  let passed = [];
  for (let element of array) {
    if (test(element)) {
      passed.push(element);
    }
  }
  return passed;
}

console.log(filter(SCRIPTS, script => script.living));
// → [{name: "Adlam", …}, …]
```

A função usa o argumento nomeado `test`, um valor da função, para preencher uma “lacuna” no cálculo - o processo de decidir quais elementos coletar.

Observe como a `filter`função, em vez de excluir elementos da matriz existente, cria uma nova matriz apenas com os elementos que passam no teste. Esta função é *pura* . Não modifica a matriz que é fornecida.

Como `forEach`, `filter`é um método de matriz padrão. O exemplo definiu a função apenas para mostrar o que faz internamente. A partir de agora, usaremos assim:

```js
console.log(SCRIPTS.filter(s => s.direction == "ttb"));
// → [{name: "Mongolian", …}, …]
```

## Transformando com mapa

Digamos que tenhamos uma matriz de objetos representando scripts, produzidos filtrando a `SCRIPTS`matriz de alguma forma. Mas queremos um conjunto de nomes que seja mais fácil de inspecionar.

O `map`método transforma uma matriz aplicando uma função a todos os seus elementos e construindo uma nova matriz a partir dos valores retornados. A nova matriz terá o mesmo comprimento que a matriz de entrada, mas seu conteúdo será *mapeado* para um novo formulário pela função.

```js
function map(array, transform) {
  let mapped = [];
  for (let element of array) {
    mapped.push(transform(element));
  }
  return mapped;
}

let rtlScripts = SCRIPTS.filter(s => s.direction == "rtl");
console.log(map(rtlScripts, s => s.name));
// → ["Adlam", "Arabic", "Imperial Aramaic", …]
```

Like `forEach`e `filter`, `map`é um método de matriz padrão.

## Resumindo com reduzir

Outra coisa comum a fazer com matrizes é calcular um único valor a partir delas. Nosso exemplo recorrente, somando uma coleção de números, é uma instância disso. Outro exemplo é encontrar o script com mais caracteres.

A operação de ordem superior que representa esse padrão é chamada de *redução* (às vezes também chamada de *dobra* ). Ele cria um valor retirando repetidamente um único elemento da matriz e combinando-o com o valor atual. Ao somar números, você começaria com o número zero e, para cada elemento, adicionaria isso à soma.

Os parâmetros para `reduce`são, além da matriz, uma função combinada e um valor inicial. Essa função é um pouco menos direta que `filter`e `map`, portanto, observe-a de perto:

```js
function reduce(array, combine, start) {
  let current = start;
  for (let element of array) {
    current = combine(current, element);
  }
  return current;
}

console.log(reduce([1, 2, 3, 4], (a, b) => a + b, 0));
// → 10
```

O método de matriz padrão `reduce`, que obviamente corresponde a essa função, possui uma conveniência adicional. Se sua matriz contiver pelo menos um elemento, você poderá deixar de lado o `start`argumento. O método pegará o primeiro elemento da matriz como seu valor inicial e começará a reduzir no segundo elemento.

```js
console.log([1, 2, 3, 4].reduce((a, b) => a + b));
// → 10
```

Para usar `reduce`(duas vezes) para encontrar o script com mais caracteres, podemos escrever algo como isto:

```js
function characterCount(script) {
  return script.ranges.reduce((count, [from, to]) => {
    return count + (to - from);
  }, 0);
}

console.log(SCRIPTS.reduce((a, b) => {
  return characterCount(a) < characterCount(b) ? b : a;
}));
// → {name: "Han", …}
```

A `characterCount`função reduz os intervalos atribuídos a um script, somando seus tamanhos. Observe o uso de desestruturação na lista de parâmetros da função redutora. A segunda chamada para `reduce`então usa isso para encontrar o maior script comparando repetidamente dois scripts e retornando o maior.

O script Han possui mais de 89.000 caracteres atribuídos a ele no padrão Unicode, tornando-o de longe o maior sistema de gravação no conjunto de dados. Han é um script (às vezes) usado para texto em chinês, japonês e coreano. Esses idiomas compartilham muitos caracteres, embora eles tendam a escrevê-los de maneira diferente. O Unicode Consortium (com sede nos EUA) decidiu tratá-los como um único sistema de gravação para salvar códigos de caracteres. Isso se chama *unificação Han* e ainda deixa algumas pessoas muito zangadas.

## Composabilidade

Considere como teríamos escrito o exemplo anterior (encontrando o maior script) sem funções de ordem superior. O código não é muito pior.

```js
let biggest = null;
for (let script of SCRIPTS) {
  if (biggest == null ||
      characterCount(biggest) < characterCount(script)) {
    biggest = script;
  }
}
console.log(biggest);
// → {name: "Han", …}
```

Existem mais algumas ligações e o programa tem quatro linhas a mais. Mas ainda é muito legível.

As funções de ordem superior começam a brilhar quando você precisa *compor* operações. Como exemplo, vamos escrever um código que encontre o ano médio de origem para scripts vivos e mortos no conjunto de dados.

```js
function average(array) {
  return array.reduce((a, b) => a + b) / array.length;
}

console.log(Math.round(average(
  SCRIPTS.filter(s => s.living).map(s => s.year))));
// → 1165
console.log(Math.round(average(
  SCRIPTS.filter(s => !s.living).map(s => s.year))));
// → 204
```

Portanto, os scripts mortos no Unicode são, em média, mais antigos que os vivos. Esta não é uma estatística terrivelmente significativa ou surpreendente. Mas espero que você concorde que o código usado para calculá-lo não é difícil de ler. Você pode vê-lo como um pipeline: começamos com todos os scripts, filtramos os vivos (ou mortos), tiramos os anos deles, calculamos a média deles e arredondamos o resultado.

Definitivamente, você também pode escrever esse cálculo como um grande loop.

```js
let total = 0, count = 0;
for (let script of SCRIPTS) {
  if (script.living) {
    total += script.year;
    count += 1;
  }
}
console.log(Math.round(total / count));
// → 1165
```

Mas é mais difícil ver o que estava sendo computado e como. E como os resultados intermediários não são representados como valores coerentes, seria muito mais trabalhoso extrair algo parecido `average`em uma função separada.

Em termos do que o computador está realmente fazendo, essas duas abordagens também são bem diferentes. O primeiro criará novas matrizes durante a execução `filter`e `map`, enquanto o segundo calcula apenas alguns números, fazendo menos trabalho. Geralmente, você pode pagar pela abordagem legível, mas se estiver processando matrizes enormes e fazendo isso muitas vezes, o estilo menos abstrato pode valer a velocidade extra.

## Strings e códigos de caracteres

Um uso do conjunto de dados seria descobrir qual script um pedaço de texto está usando. Vamos passar por um programa que faz isso.

Lembre-se de que cada script possui uma matriz de intervalos de códigos de caracteres associados a ele. Portanto, dado um código de caractere, poderíamos usar uma função como esta para encontrar o script correspondente (se houver):

```js
function characterScript(code) {
  for (let script of SCRIPTS) {
    if (script.ranges.some(([from, to]) => {
      return code >= from && code < to;
    })) {
      return script;
    }
  }
  return null;
}

console.log(characterScript(121));
// → {name: "Latin", …}
```

O `some`método é outra função de ordem superior. Ele pega uma função de teste e informa se essa função retorna true para qualquer um dos elementos da matriz.

Mas como obtemos os códigos de caracteres em uma string?

No [capítulo 1](https://eloquentjavascript.net/01_values.html) , mencionei que as strings JavaScript são codificadas como uma sequência de números de 16 bits. Estes são chamados de *unidades de código* . Inicialmente, um código de caractere Unicode deveria caber nessa unidade (o que fornece um pouco mais de 65.000 caracteres). Quando ficou claro que isso não seria suficiente, muitas pessoas recusaram a necessidade de usar mais memória por personagem. Para resolver essas preocupações, o UTF-16, o formato usado pelas seqüências de caracteres JavaScript, foi inventado. Ele descreve os caracteres mais comuns usando uma única unidade de código de 16 bits, mas usa um par de duas dessas unidades para outras.

Hoje, o UTF-16 é geralmente considerado uma má idéia. Parece quase intencionalmente projetado para convidar erros. É fácil escrever programas que fingem que as unidades e caracteres de código são a mesma coisa. E se o seu idioma não usar caracteres de duas unidades, isso parecerá funcionar perfeitamente. Mas assim que alguém tenta usar esse programa com alguns caracteres chineses menos comuns, ele quebra. Felizmente, com o advento do emoji, todo mundo começou a usar caracteres de duas unidades, e o ônus de lidar com esses problemas é distribuído de maneira mais justa.

Infelizmente, operações óbvias em strings JavaScript, como obter o comprimento da `length`propriedade e acessar seu conteúdo usando colchetes, lidam apenas com unidades de código.

```js
// Two emoji characters, horse and shoe
let horseShoe = "🐴👟";
console.log(horseShoe.length);
// → 4
console.log(horseShoe[0]);
// → (Invalid half-character)
console.log(horseShoe.charCodeAt(0));
// → 55357 (Code of the half-character)
console.log(horseShoe.codePointAt(0));
// → 128052 (Actual code for horse emoji)
```

O `charCodeAt`método do JavaScript fornece uma unidade de código, não um código completo de caracteres. O `codePointAt`método, adicionado posteriormente, fornece um caractere Unicode completo. Então, poderíamos usar isso para obter caracteres de uma string. Mas o argumento passado `codePointAt`ainda é um índice para a sequência de unidades de código. Portanto, para atropelar todos os caracteres de uma sequência, ainda precisamos lidar com a questão de saber se um caractere ocupa uma ou duas unidades de código.

No [capítulo anterior](https://eloquentjavascript.net/04_data.html#for_of_loop) , mencionei que um loop `for`/ `of`também pode ser usado em strings. Assim `codePointAt`, esse tipo de loop foi introduzido em um momento em que as pessoas estavam cientes dos problemas com o UTF-16. Quando você o usa para fazer um loop sobre uma string, ele fornece caracteres reais, não unidades de código.

```js
let roseDragon = "🌹🐉";
for (let char of roseDragon) {
  console.log(char);
}
// → 🌹
// → 🐉
```

Se você tiver um caractere (que será uma sequência de uma ou duas unidades de código), poderá usá `codePointAt(0)`-lo para obter seu código.

## Reconhecendo texto

Temos uma `characterScript`função e uma maneira de fazer um loop correto sobre os caracteres. O próximo passo é contar os caracteres que pertencem a cada script. A seguinte abstração de contagem será útil lá:

```js
function countBy(items, groupName) {
  let counts = [];
  for (let item of items) {
    let name = groupName(item);
    let known = counts.findIndex(c => c.name == name);
    if (known == -1) {
      counts.push({name, count: 1});
    } else {
      counts[known].count++;
    }
  }
  return counts;
}

console.log(countBy([1, 2, 3, 4, 5], n => n > 2));
// → [{name: false, count: 2}, {name: true, count: 3}]
```

A `countBy`função espera uma coleção (qualquer coisa que possamos repetir com `for`/ `of`) e uma função que calcule o nome de um grupo para um determinado elemento. Ele retorna uma matriz de objetos, cada um com um nome para um grupo e informa o número de elementos encontrados nesse grupo.

Ele usa outro método de matriz - `findIndex`. Este método é um pouco parecido `indexOf`, mas em vez de procurar um valor específico, ele encontra o primeiro valor para o qual a função especificada retorna verdadeira. Como `indexOf`, retorna -1 quando nenhum elemento é encontrado.

Usando `countBy`, podemos escrever a função que nos diz quais scripts são usados em um pedaço de texto.

```js
function textScripts(text) {
  let scripts = countBy(text, char => {
    let script = characterScript(char.codePointAt(0));
    return script ? script.name : "none";
  }).filter(({name}) => name != "none");

  let total = scripts.reduce((n, {count}) => n + count, 0);
  if (total == 0) return "No scripts found";

  return scripts.map(({name, count}) => {
    return `${Math.round(count * 100 / total)}% ${name}`;
  }).join(", ");
}

console.log(textScripts('英国的狗说"woof", 俄罗斯的狗说"тяв"'));
// → 61% Han, 22% Latin, 17% Cyrillic
```

A função primeiro conta os caracteres pelo nome, usando `characterScript`para atribuir um nome a eles e retornando à sequência `"none"`para caracteres que não fazem parte de nenhum script. A `filter`chamada elimina a entrada `"none"`da matriz resultante, pois não estamos interessados nesses caracteres.

Para poder calcular porcentagens, primeiro precisamos do número total de caracteres que pertencem a um script, com o qual podemos calcular `reduce`. Se nenhum desses caracteres for encontrado, a função retornará uma sequência específica. Caso contrário, ele transforma as entradas de contagem em seqüências de caracteres legíveis `map`e as combina com `join`.

## Sumário

Ser capaz de passar valores de função para outras funções é um aspecto profundamente útil do JavaScript. Ele nos permite escrever funções que modelam cálculos com "lacunas" neles. O código que chama essas funções pode preencher as lacunas fornecendo valores de função.

As matrizes fornecem vários métodos úteis de ordem superior. Você pode usar o `forEach`loop sobre os elementos em uma matriz. O `filter`método retorna uma nova matriz que contém apenas os elementos que passam na função de predicado. A transformação de uma matriz colocando cada elemento através de uma função é feita com `map`. Você pode usar `reduce`para combinar todos os elementos em uma matriz em um único valor. O `some`método testa se algum elemento corresponde a uma determinada função de predicado. E `findIndex`encontra a posição do primeiro elemento que corresponde a um predicado.

## Exercícios

### Achatamento

Use o `reduce`método em combinação com o `concat`método para “achatar” uma matriz de matrizes em uma única matriz que possua todos os elementos das matrizes originais.

```js
let arrays = [[1, 2, 3], [4, 5], [6]];
// Your code here.
// → [1, 2, 3, 4, 5, 6]
```

### Seu próprio loop

Escreva uma função de ordem superior `loop`que forneça algo como uma `for`declaração de loop. É preciso um valor, uma função de teste, uma função de atualização e uma função do corpo. A cada iteração, ele primeiro executa a função de teste no valor atual do loop e para se isso retornar falso. Em seguida, chama a função body, fornecendo o valor atual. Por fim, ele chama a função de atualização para criar um novo valor e começa desde o início.

Ao definir a função, você pode usar um loop regular para fazer o loop real.

```js
// Your code here.

loop(3, n => n > 0, n => n - 1, console.log);
// → 3
// → 2
// → 1
```

### Tudo

Análoga ao `some`método, matrizes também têm um `every`método. Este retorna true quando a função especificada retorna true para *todos os* elementos da matriz. De certa forma, `some`é uma versão do `||`operador que atua em matrizes e `every`é como o `&&`operador.

Implemente `every`como uma função que utiliza uma matriz e uma função de predicado como parâmetros. Escreva duas versões, uma usando um loop e outra usando o `some`método

```js
function every(array, test) {
  // Your code here.
}

console.log(every([1, 3, 5], n => n < 10));
// → true
console.log(every([2, 4, 16], n => n < 10));
// → false
console.log(every([], n => n < 10));
// → true
```

Como o `&&`operador, o `every`método pode parar de avaliar outros elementos assim que encontrar um que não corresponda. Portanto, a versão baseada em loop pode sair do loop - com `break`ou `return`- assim que for executada em um elemento para o qual a função predicada retorne false. Se o loop for executado até o fim sem encontrar esse elemento, sabemos que todos os elementos corresponderam e devemos retornar true.

Para construir `every`em cima `some`, podemos aplicar *as leis de De Morgan* , que afirmam que `a && b`é igual `!(!a || !b)`. Isso pode ser generalizado para matrizes, onde todos os elementos na matriz correspondem se não houver nenhum elemento na matriz que não corresponda.

### Direção de escrita dominante

Escreva uma função que calcule a direção de escrita dominante em uma sequência de texto. Lembre-se de que cada objeto de script possui uma `direction`propriedade que pode ser `"ltr"`(da esquerda para a direita), `"rtl"`(da direita para a esquerda) ou `"ttb"`(de cima para baixo).

A direção dominante é a direção da maioria dos personagens que possuem um script associado a eles. As funções `characterScript`e `countBy`definidas anteriormente neste capítulo provavelmente são úteis aqui.

```js
function dominantDirection(text) {
  // Your code here.
}

console.log(dominantDirection("Hello!"));
// → ltr
console.log(dominantDirection("Hey, مساء الخير"));
// → rtl
```

Sua solução pode parecer muito com a primeira metade do `textScripts`exemplo. Você novamente precisa contar os caracteres por um critério baseado `characterScript`e depois filtrar a parte do resultado que se refere a caracteres desinteressantes (sem script).

É possível encontrar a direção com a maior contagem de caracteres `reduce`. Se não estiver claro como, consulte o exemplo anterior no capítulo, onde `reduce`foi usado para encontrar o script com mais caracteres.