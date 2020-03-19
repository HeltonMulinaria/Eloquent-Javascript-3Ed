# Capítulo 4 Estruturas de dados: objetos e matrizes

> Em duas ocasiões, me perguntaram: 'Ore, Sr. Babbage, se você colocar na máquina números errados, as respostas certas serão divulgadas?' [...] não sou capaz de apreender com razão o tipo de confusão de idéias que poderia provocar essa pergunta.
>
> *—Charles Babbage, passagens da vida de um filósofo (1864)*

![Imagem de um homem-esquilo](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/chapter_picture_4.jpg)

Números, booleanos e strings são os átomos dos quais as estruturas de dados são construídas. Muitos tipos de informações requerem mais de um átomo. *Objetos* nos permitem agrupar valores - incluindo outros objetos - para construir estruturas mais complexas.

Os programas que criamos até agora foram limitados pelo fato de estarem operando apenas em tipos de dados simples. Este capítulo apresentará estruturas básicas de dados. No final, você saberá o suficiente para começar a escrever programas úteis.

O capítulo trabalhará com um exemplo de programação mais ou menos realista, introduzindo conceitos que se aplicam ao problema em questão. O código de exemplo geralmente cria funções e ligações introduzidas anteriormente no texto.

## O homem-esquilo

De vez em quando, geralmente entre 20h e 22h, Jacques se transforma em um pequeno roedor peludo com uma cauda espessa.

Por um lado, Jacques está bastante feliz por não ter licantropia clássica. Se transformar em um esquilo causa menos problemas do que se transformar em um lobo. Em vez de se preocupar em comer acidentalmente o vizinho ( *isso* seria estranho), ele se preocupa em ser comido pelo gato do vizinho. Depois de duas ocasiões em que acordou em um galho precariamente magro na coroa de um carvalho, nu e desorientado, passou a trancar as portas e janelas de seu quarto à noite e a colocar algumas nozes no chão para se manter ocupado.

Isso cuida dos problemas dos gatos e das árvores. Mas Jacques prefere se livrar completamente de sua condição. As ocorrências irregulares da transformação fazem com que ele suspeite que elas possam ser desencadeadas por alguma coisa. Por um tempo, ele acreditou que isso acontecia apenas nos dias em que ele estava perto de carvalhos. Mas evitar carvalhos não impediu o problema.

Mudando para uma abordagem mais científica, Jacques começou a manter um registro diário de tudo o que faz em um determinado dia e se ele mudou de forma. Com esses dados, ele espera diminuir as condições que desencadeiam as transformações.

A primeira coisa que ele precisa é de uma estrutura de dados para armazenar essas informações.

## Conjuntos de dados

Para trabalhar com uma porção de dados digitais, primeiro precisamos encontrar uma maneira de representá-los na memória de nossa máquina. Digamos, por exemplo, que queremos representar uma coleção dos números 2, 3, 5, 7 e 11.

Poderíamos ser criativos com as strings - afinal, as strings podem ter qualquer comprimento, para que possamos colocar muitos dados nelas - e usá-las `"2 3 5 7 11"`como nossa representação. Mas isso é estranho. Você precisaria extrair os dígitos e convertê-los novamente em números para acessá-los.

Felizmente, o JavaScript fornece um tipo de dados especificamente para armazenar sequências de valores. É chamado de *matriz* e é escrito como uma lista de valores entre colchetes, separados por vírgulas.

```js
let listOfNumbers = [2, 3, 5, 7, 11];
console.log(listOfNumbers[2]);
// → 5
console.log(listOfNumbers[0]);
// → 2
console.log(listOfNumbers[2 - 1]);
// → 3
```

A notação para obter os elementos dentro de uma matriz também usa colchetes. Um par de colchetes imediatamente após uma expressão, com outra expressão dentro deles, procurará o elemento na expressão à esquerda que corresponde ao *índice* fornecido pela expressão entre colchetes.

O primeiro índice de uma matriz é zero, não um. Portanto, o primeiro elemento é recuperado com `listOfNumbers[0]`. A contagem baseada em zero tem uma longa tradição em tecnologia e, de certa forma, faz muito sentido, mas é preciso algum tempo para se acostumar. Pense no índice como a quantidade de itens a serem ignorados, contados desde o início da matriz.

## Properties

Vimos algumas expressões de aparência suspeita como `myString.length`(para obter o comprimento de uma string) e `Math.max`(a função máxima) nos capítulos anteriores. Essas são expressões que acessam uma *propriedade* de algum valor. No primeiro caso, temos acesso a `length`propriedade do valor `myString`. No segundo, acessamos a propriedade nomeada `max`no `Math`objeto (que é uma coleção de constantes e funções relacionadas à matemática).

Quase todos os valores de JavaScript têm propriedades. As exceções são `null`e `undefined`. Se você tentar acessar uma propriedade em um desses valores não, você receberá um erro.

```js
null.length;
// → TypeError: null has no properties
```

As duas principais maneiras de acessar propriedades no JavaScript são com um ponto e entre colchetes. Ambos `value.x`e `value[x]`acessam uma propriedade `value`- mas não necessariamente a mesma propriedade. A diferença está em como `x`é interpretado. Ao usar um ponto, a palavra após o ponto é o nome literal da propriedade. Ao usar colchetes, a expressão entre colchetes é *avaliada* para obter o nome da propriedade. Enquanto `value.x`busca a propriedade `value`denominada “x”, `value[x]`tenta avaliar a expressão `x`e usa o resultado, convertido em uma sequência, como o nome da propriedade.

Então, se você sabe que o imóvel em que você está interessado se chama *cor* , você diz `value.color`. Se você deseja extrair a propriedade nomeada pelo valor mantido na ligação `i`, você diz `value[i]`. Nomes de propriedades são cadeias de caracteres. Eles podem ser qualquer sequência, mas a notação de ponto funciona apenas com nomes que se parecem com nomes de ligação válidos. Portanto, se você deseja acessar uma propriedade chamada *2* ou *John Doe* , use colchetes: `value[2]`ou `value["John Doe"]`.

Os elementos em uma matriz são armazenados como propriedades da matriz, usando números como nomes de propriedades. Como você não pode usar a notação de ponto com números e geralmente deseja usar uma ligação que contenha o índice de qualquer maneira, é necessário usar a notação de colchete para alcançá-los.

A `length`propriedade de uma matriz nos diz quantos elementos ela possui. Esse nome de propriedade é um nome de ligação válido e sabemos seu nome com antecedência; portanto, para encontrar o comprimento de uma matriz, você normalmente escreve `array.length`porque é mais fácil escrever do que `array["length"]`.

## Métodos

Os valores da cadeia e da matriz contêm, além da `length`propriedade, várias propriedades que mantêm valores de função.

```
let doh = "Doh";
console.log(typeof doh.toUpperCase);
// → function
console.log(doh.toUpperCase());
// → DOH
```

Toda string tem uma `toUpperCase`propriedade. Quando chamado, ele retornará uma cópia da sequência na qual todas as letras foram convertidas em maiúsculas. Há também `toLowerCase`, indo para o outro lado.

Curiosamente, embora a chamada para `toUpperCase`não passe nenhum argumento, a função de alguma forma tem acesso à string `"Doh"`, o valor cuja propriedade chamamos. Como isso funciona é descrito no [Capítulo 6](https://eloquentjavascript.net/06_object.html#obj_methods) .

As propriedades que contêm funções são geralmente chamadas de *métodos* do valor a que pertencem, como em " `toUpperCase`é o método de uma string".

Este exemplo demonstra dois métodos que você pode usar para manipular matrizes:

```js
let sequence = [1, 2, 3];
sequence.push(4);
sequence.push(5);
console.log(sequence);
// → [1, 2, 3, 4, 5]
console.log(sequence.pop());
// → 5
console.log(sequence);
// → [1, 2, 3, 4]
```

O `push`método adiciona valores ao final de uma matriz e o `pop`método faz o contrário, removendo o último valor da matriz e retornando-o.

Esses nomes um tanto tolos são os termos tradicionais para operações em uma *pilha* . Uma pilha, na programação, é uma estrutura de dados que permite inserir valores nela e exibi-los novamente na ordem oposta, para que a última coisa que foi adicionada por último seja removida primeiro. Isso é comum na programação - você pode se lembrar da pilha de chamadas de função do [capítulo anterior](https://eloquentjavascript.net/03_functions.html#stack) , que é uma instância da mesma idéia.

## Objetos

De volta ao homem-esquilo. Um conjunto de entradas de log diárias pode ser representado como uma matriz. Mas as entradas não consistem em apenas um número ou uma string - cada entrada precisa armazenar uma lista de atividades e um valor booleano que indica se Jacques se transformou em um esquilo ou não. Idealmente, gostaríamos de agrupá-los em um único valor e, em seguida, colocar esses valores agrupados em uma matriz de entradas de log.

Os valores do *objeto de* tipo são coleções arbitrárias de propriedades. Uma maneira de criar um objeto é usar chaves como expressão.

```js
let day1 = {
  squirrel: false,
  events: ["work", "touched tree", "pizza", "running"]
};
console.log(day1.squirrel);
// → false
console.log(day1.wolf);
// → undefined
day1.wolf = false;
console.log(day1.wolf);
// → false
```

Dentro dos chavetas, há uma lista de propriedades separadas por vírgulas. Cada propriedade tem um nome seguido por dois pontos e um valor. Quando um objeto é escrito em várias linhas, o recuo, como no exemplo, ajuda na legibilidade. As propriedades cujos nomes não são nomes de ligação válidos ou números válidos devem ser citadas.

```
let descriptions = {
  work: "Went to work",
  "touched tree": "Touched a tree"
};
```

Isso significa que chaves têm *dois* significados em JavaScript. No início de uma instrução, eles iniciam um bloco de instruções. Em qualquer outra posição, eles descrevem um objeto. Felizmente, raramente é útil iniciar uma declaração com um objeto entre chaves, portanto a ambiguidade entre esses dois não é um grande problema.

Ler uma propriedade que não existe dará a você o valor `undefined`.

É possível atribuir um valor a uma expressão de propriedade com o `=`operador. Isso substituirá o valor da propriedade, se ela já existir, ou criará uma nova propriedade no objeto, se não existir.

Para retornar brevemente ao nosso modelo tentativo de ligações - as ligações de propriedades são semelhantes. Eles *compreendem* valores, mas outras ligações e propriedades podem estar mantendo esses mesmos valores. Você pode pensar em objetos como polvos com qualquer número de tentáculos, cada um com um nome tatuado.

O `delete`operador corta um tentáculo desse polvo. É um operador unário que, quando aplicado a uma propriedade de objeto, remove a propriedade nomeada do objeto. Isso não é algo comum de se fazer, mas é possível.

```js
let anObject = {left: 1, right: 2};
console.log(anObject.left);
// → 1
delete anObject.left;
console.log(anObject.left);
// → undefined
console.log("left" in anObject);
// → false
console.log("right" in anObject);
// → true
```

O `in`operador binário , quando aplicado a uma sequência e a um objeto, informa se esse objeto possui uma propriedade com esse nome. A diferença entre definir `undefined`e excluir uma propriedade é que, no primeiro caso, o objeto ainda *possui* a propriedade (apenas não possui um valor muito interessante), enquanto no segundo caso a propriedade não está mais presente e `in`retornará `false`.

Para descobrir quais propriedades um objeto possui, você pode usar a `Object.keys`função Você atribui um objeto a ele e ele retorna uma matriz de strings - os nomes das propriedades do objeto.

```js
console.log(Object.keys({x: 0, y: 0, z: 2}));
// → ["x", "y", "z"]
```

Há uma `Object.assign`função que copia todas as propriedades de um objeto para outro.

```js
let objectA = {a: 1, b: 2};
Object.assign(objectA, {b: 3, c: 4});
console.log(objectA);
// → {a: 1, b: 3, c: 4}
```

Matrizes, então, são apenas um tipo de objeto especializado para armazenar seqüências de coisas. Se você avaliar `typeof []`, ele produz `"object"`. Você pode vê-los como polvos longos e planos, com todos os seus tentáculos em uma fileira organizada, rotulados com números.

Representaremos o diário que Jacques mantém como uma variedade de objetos.

```js
let journal = [
  {events: ["work", "touched tree", "pizza",
            "running", "television"],
   squirrel: false},
  {events: ["work", "ice cream", "cauliflower",
            "lasagna", "touched tree", "brushed teeth"],
   squirrel: false},
  {events: ["weekend", "cycling", "break", "peanuts",
            "beer"],
   squirrel: true},
  /* and so on... */
];
```

## Mutabilidade

Em breve, chegaremos à programação *real* . Primeiro, há mais uma teoria para entender.

Vimos que os valores dos objetos podem ser modificados. Os tipos de valores discutidos nos capítulos anteriores, como números, seqüências de caracteres e booleanos, são todos *imutáveis* - é impossível alterar os valores desses tipos. Você pode combiná-los e derivar novos valores a partir deles, mas quando você obtém um valor de sequência específico, esse valor sempre permanecerá o mesmo. O texto dentro dele não pode ser alterado. Se você tiver uma sequência que contenha `"cat"`, não é possível que outro código altere um caractere em sua sequência para torná-la soletrada `"rat"`.

Objetos funcionam de maneira diferente. Você *pode* alterar suas propriedades, fazendo com que um valor de objeto único tenha conteúdo diferente em momentos diferentes.

Quando temos dois números, 120 e 120, podemos considerá-los precisamente o mesmo número, independentemente de se referirem aos mesmos bits físicos. Com objetos, há uma diferença entre ter duas referências ao mesmo objeto e dois objetos diferentes que contêm as mesmas propriedades. Considere o seguinte código:

```js
let object1 = {value: 10};
let object2 = object1;
let object3 = {value: 10};

console.log(object1 == object2);
// → true
console.log(object1 == object3);
// → false

object1.value = 15;
console.log(object2.value);
// → 15
console.log(object3.value);
// → 10
```

As ligações `object1`e `object2`compreendem o *mesmo* objeto, e é por isso que alterar `object1`também altera o valor de `object2`. Dizem que eles têm a mesma *identidade* . A ligação `object3`aponta para um objeto diferente, que inicialmente contém as mesmas propriedades, `object1`mas vive uma vida separada.

As ligações também podem ser alteráveis ou constantes, mas isso é separado da maneira como seus valores se comportam. Mesmo que os valores numéricos não sejam alterados, você pode usar uma `let`ligação para acompanhar um número alterado alterando o valor em que a ligação aponta. Da mesma forma, embora uma `const`ligação a um objeto não possa ela própria ser alterada e continue a apontar para o mesmo objeto, o *conteúdo* desse objeto pode ser alterado.

```js
const score = {visitors: 0, home: 0};
// This is okay
score.visitors = 1;
// This isn't allowed
score = {visitors: 1, home: 1};
```

Quando você compara objetos com o `==`operador JavaScript , ele é comparado por identidade: ele produzirá `true`apenas se os dois objetos tiverem exatamente o mesmo valor. A comparação de objetos diferentes retornará `false`, mesmo se eles tiverem propriedades idênticas. Não existe uma operação de comparação “profunda” incorporada ao JavaScript, que compara objetos por conteúdo, mas é possível escrevê-lo você mesmo (que é um dos [exercícios](https://eloquentjavascript.net/04_data.html#exercise_deep_compare) no final deste capítulo).

## O log do licantropo

Assim, Jacques inicia seu intérprete de JavaScript e configura o ambiente de que precisa para manter seu diário.

```
let journal = [];

function addEntry(events, squirrel) {
  journal.push({events, squirrel});
}
```

Observe que o objeto adicionado ao diário parece um pouco estranho. Em vez de declarar propriedades como `events: events`, apenas fornece um nome de propriedade. Essa é uma abreviação que significa a mesma coisa - se um nome de propriedade na notação entre colchetes não for seguido por um valor, seu valor será retirado da ligação com o mesmo nome.

Então, todas as noites às 22 horas - ou às vezes na manhã seguinte, depois de descer da prateleira de cima da estante - Jacques registra o dia.

```js
addEntry(["work", "touched tree", "pizza", "running",
          "television"], false);
addEntry(["work", "ice cream", "cauliflower", "lasagna",
          "touched tree", "brushed teeth"], false);
addEntry(["weekend", "cycling", "break", "peanuts",
          "beer"], true);
```

Uma vez que ele tenha pontos de dados suficientes, ele pretende usar as estatísticas para descobrir quais desses eventos podem estar relacionados às informações de esquilo.

*Correlação* é uma medida de dependência entre variáveis estatísticas. Uma variável estatística não é exatamente a mesma que uma variável de programação. Nas estatísticas, você normalmente tem um conjunto de *medidas* e cada variável é medida para cada medida. A correlação entre variáveis é geralmente expressa como um valor que varia de -1 a 1. Correlação zero significa que as variáveis não estão relacionadas. Uma correlação de um indica que os dois estão perfeitamente relacionados - se você conhece um, também conhece o outro. Um negativo também significa que as variáveis estão perfeitamente relacionadas, mas são opostas - quando uma é verdadeira, a outra é falsa.

Para calcular a medida de correlação entre duas variáveis booleanas, podemos usar o *coeficiente phi* ( *ϕ* ). Esta é uma fórmula cuja entrada é uma tabela de frequências que contém o número de vezes que as diferentes combinações das variáveis foram observadas. A saída da fórmula é um número entre -1 e 1 que descreve a correlação.

Poderíamos pegar o evento de comer pizza e colocá-lo em uma tabela de frequências como essa, em que cada número indica a quantidade de vezes que a combinação ocorreu em nossas medições:

![Comer pizza versus se transformar em um esquilo](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/pizza-squirrel.svg)

Se chamarmos essa tabela *n* , podemos calcular *ϕ* usando a seguinte fórmula:

- [ ]  Adicionar formatação correta da fórmula

  **ϕ =  n 11 n 00 - n 10 n 01**

  **√ n 1 • n 0 • n • 1 n • 0**

  

(Se, neste momento, você está colocando o livro de lado para se concentrar em um terrível flashback da aula de matemática da 10ª série - espere! Não pretendo torturá-lo com inúmeras páginas de notação enigmática - é apenas essa fórmula por enquanto. mesmo com este, tudo o que fazemos é transformá-lo em JavaScript.)

A notação *n* 01 indica o número de medições em que a primeira variável (esquilo) é falsa (0) e a segunda variável (pizza) é verdadeira (1). Na mesa da pizza, *n* 01 é 9.

O valor *n* 1 • refere-se à soma de todas as medições em que a primeira variável é verdadeira, que é 5 na tabela de exemplos. Da mesma forma, *n* • 0 refere-se à soma das medidas em que a segunda variável é falsa.

Portanto, para a mesa de pizza, a parte acima da linha de divisão (o dividendo) seria 1 × 76−4 × 9 = 40, e a parte abaixo (o divisor) seria a raiz quadrada de 5 × 85 × 10 × 80 ou √340000. Isto vem a *& Phi;* ≈ 0,069, que é pequena. Comer pizza não parece ter influência nas transformações.

## Correlação computacional

Podemos representar uma tabela dois por dois em JavaScript com uma matriz de quatro elementos ( `[76, 9, 4, 1]`). Também poderíamos usar outras representações, como uma matriz contendo duas matrizes de dois elementos ( `[[76, 9], [4, 1]]`) ou um objeto com nomes de propriedades como `"11"`e `"01"`, mas a matriz plana é simples e torna as expressões que acessam a tabela agradavelmente curtas. Vamos interpretar os índices para a matriz como números binários de dois bits, onde o dígito mais à esquerda (mais significativo) se refere à variável esquilo e o dígito mais à direita (menos significativo) se refere à variável de evento. Por exemplo, o número binário `10`refere-se ao caso em que Jacques se transformou em um esquilo, mas o evento (por exemplo, "pizza") não ocorreu. Isso aconteceu quatro vezes. E já que binário`10` é 2 em notação decimal, armazenaremos esse número no índice 2 da matriz.

Esta é a função que calcula o *φ* coeficiente de uma matriz tal:

```js
function phi(table) {
  return (table[3] * table[0] - table[2] * table[1]) /
    Math.sqrt((table[2] + table[3]) *
              (table[0] + table[1]) *
              (table[1] + table[3]) *
              (table[0] + table[2]));
}

console.log(phi([76, 9, 4, 1]));
// → 0.068599434
```

Esta é uma tradução directa do *φ* fórmula em JavaScript. `Math.sqrt`é a função de raiz quadrada, conforme fornecida pelo `Math`objeto em um ambiente JavaScript padrão. Temos que adicionar dois campos da tabela para obter campos como n 1, porque as somas de linhas ou colunas não são armazenadas diretamente em nossa estrutura de dados.

Jacques manteve seu diário por três meses. O conjunto de dados resultante está disponível na [sandbox de codificação](https://eloquentjavascript.net/code#4) deste capítulo, onde é armazenada na `JOURNAL`ligação e em um [arquivo para](https://eloquentjavascript.net/code/journal.js) download .

Para extrair uma tabela de duas por duas para um evento específico do diário, precisamos fazer um loop sobre todas as entradas e registrar quantas vezes o evento ocorre em relação às transformações de esquilo.

```js
function tableFor(event, journal) {
  let table = [0, 0, 0, 0];
  for (let i = 0; i < journal.length; i++) {
    let entry = journal[i], index = 0;
    if (entry.events.includes(event)) index += 1;
    if (entry.squirrel) index += 2;
    table[index] += 1;
  }
  return table;
}

console.log(tableFor("pizza", JOURNAL));
// → [76, 9, 4, 1]
```

As matrizes têm um `includes`método que verifica se um determinado valor existe na matriz. A função usa isso para determinar se o nome do evento em que está interessado faz parte da lista de eventos para um determinado dia.

O corpo do loop `tableFor`mostra em que caixa da tabela cada entrada do diário se encaixa, verificando se a entrada contém o evento específico no qual está interessada e se o evento acontece ao lado de um incidente de esquilo. O loop adiciona um à caixa correta na tabela.

Agora temos as ferramentas necessárias para calcular correlações individuais. O único passo restante é encontrar uma correlação para cada tipo de evento que foi registrado e ver se algo se destaca.

## Loops de matriz

Na `tableFor`função, há um loop como este:

```js
for (let i = 0; i < JOURNAL.length; i++) {
  let entry = JOURNAL[i];
  // Do something with entry
}
```

Esse tipo de loop é comum no JavaScript clássico - revisar as matrizes, um elemento de cada vez, é algo que surge muito e, para isso, é necessário executar um contador no comprimento da matriz e selecionar cada elemento por vez.

Existe uma maneira mais simples de escrever esses loops no JavaScript moderno.

```js
for (let entry of JOURNAL) {
  console.log(`${entry.events.length} events.`);
}
```

Quando um `for`loop se parece com isso, com a palavra `of`após uma definição de variável, ele fará um loop sobre os elementos do valor fornecido depois `of`. Isso funciona não apenas para matrizes, mas também para seqüências de caracteres e algumas outras estruturas de dados. Discutiremos *como* isso funciona no [Capítulo 6](https://eloquentjavascript.net/06_object.html) .

## A análise final

Precisamos calcular uma correlação para cada tipo de evento que ocorre no conjunto de dados. Para fazer isso, primeiro precisamos *encontrar* todos os tipos de eventos.

```js
function journalEvents(journal) {
  let events = [];
  for (let entry of journal) {
    for (let event of entry.events) {
      if (!events.includes(event)) {
        events.push(event);
      }
    }
  }
  return events;
}

console.log(journalEvents(JOURNAL));
// → ["carrot", "exercise", "weekend", "bread", …]
```

Examinando todos os eventos e adicionando aqueles que ainda não estão lá na `events`matriz, a função coleta todos os tipos de eventos.

Usando isso, podemos ver todas as correlações.

```js
for (let event of journalEvents(JOURNAL)) {
  console.log(event + ":", phi(tableFor(event, JOURNAL)));
}
// → carrot:   0.0140970969
// → exercise: 0.0685994341
// → weekend:  0.1371988681
// → bread:   -0.0757554019
// → pudding: -0.0648203724
// and so on...
```

A maioria das correlações parece estar perto de zero. Comer cenoura, pão ou pudim aparentemente não desencadeia a licantropia de esquilo. Ele *não* parecem ocorrer um pouco mais frequentemente nos fins de semana. Vamos filtrar os resultados para mostrar apenas correlações maiores que 0,1 ou menores que -0,1.

```js
for (let event of journalEvents(JOURNAL)) {
  let correlation = phi(tableFor(event, JOURNAL));
  if (correlation > 0.1 || correlation < -0.1) {
    console.log(event + ":", correlation);
  }
}
// → weekend:        0.1371988681
// → brushed teeth: -0.3805211953
// → candy:          0.1296407447
// → work:          -0.1371988681
// → spaghetti:      0.2425356250
// → reading:        0.1106828054
// → peanuts:        0.5902679812
// → amendoim: 0.5902679812
```

Aha! Existem dois fatores com uma correlação claramente mais forte que os outros. Comer amendoim tem um forte efeito positivo na chance de se transformar em um esquilo, enquanto escovar os dentes tem um efeito negativo significativo.

Interessante. Vamos tentar uma coisa.

```js
for (let entry of JOURNAL) {
  if (entry.events.includes("peanuts") &&
     !entry.events.includes("brushed teeth")) {
    entry.events.push("peanut teeth");
  }
}
console.log(phi(tableFor("peanut teeth", JOURNAL)));
// → 1
```

Esse é um resultado forte. O fenômeno ocorre precisamente quando Jacques come amendoim e falha em escovar os dentes. Se ele não fosse tão desleixado em relação à higiene dental, nunca teria notado sua aflição.

Sabendo disso, Jacques para de comer amendoim e descobre que suas transformações não voltam.

Por alguns anos, as coisas vão bem para Jacques. Mas em algum momento ele perde o emprego. Como ele mora em um país desagradável, onde não ter emprego significa não ter serviços médicos, ele é forçado a contratar emprego em um circo onde atua como *O Incrível Esquilo* , enchendo a boca com manteiga de amendoim antes de cada show.

Um dia, farto dessa existência lamentável, Jacques não consegue voltar à sua forma humana, pula através de uma fenda na tenda do circo e desaparece na floresta. Ele nunca mais é visto.

## Matriz adicional

Antes de terminar o capítulo, quero apresentar alguns outros conceitos relacionados a objetos. Começarei introduzindo alguns métodos de matriz geralmente úteis.

Vimos `push`e `pop`, que adicionam e removem elementos no final de uma matriz, [anteriormente](https://eloquentjavascript.net/04_data.html#array_methods) neste capítulo. Os métodos correspondentes para adicionar e remover itens no início de uma matriz são chamados `unshift`e `shift`.

```js
let todoList = [];
function remember(task) {
  todoList.push(task);
}
function getTask() {
  return todoList.shift();
}
function rememberUrgently(task) {
  todoList.unshift(task);
}
```

Esse programa gerencia uma fila de tarefas. Você adiciona tarefas ao final da fila chamando `remember("groceries")`e, quando estiver pronto para fazer alguma coisa, chama `getTask()`para obter (e remover) o item da frente da fila. A `rememberUrgently`função também adiciona uma tarefa, mas a adiciona à frente, em vez da parte traseira da fila.

Para procurar um valor específico, as matrizes fornecem um `indexOf`método. O método pesquisa na matriz do início ao fim e retorna o índice no qual o valor solicitado foi encontrado - ou -1, se não foi encontrado. Para pesquisar do final ao invés do início, existe um método semelhante chamado `lastIndexOf`.

```js
console.log([1, 2, 3, 2, 1].indexOf(2));
// → 1
console.log([1, 2, 3, 2, 1].lastIndexOf(2));
// → 3
```

Ambos `indexOf`e `lastIndexOf`usam um segundo argumento opcional que indica por onde começar a pesquisar.

Outro método fundamental de matriz é `slice`, que pega os índices de início e de fim e retorna uma matriz que possui apenas os elementos entre eles. O índice inicial é inclusivo, o índice final exclusivo.

```js
console.log([0, 1, 2, 3, 4].slice(2, 4));
// → [2, 3]
console.log([0, 1, 2, 3, 4].slice(2));
// → [2, 3, 4]
```

Quando o índice final não for fornecido, `slice`todos os elementos serão retirados após o índice inicial. Você também pode omitir o índice inicial para copiar toda a matriz.

O `concat`método pode ser usado para colar matrizes para criar uma nova matriz, semelhante ao que o `+`operador faz para seqüências de caracteres.

O exemplo a seguir mostra ambos `concat`e `slice`em ação. Ele pega uma matriz e um índice e retorna uma nova matriz que é uma cópia da matriz original com o elemento no índice especificado removido.

```js
function remove(array, index) {
  return array.slice(0, index)
    .concat(array.slice(index + 1));
}
console.log(remove(["a", "b", "c", "d", "e"], 2));
// → ["a", "b", "d", "e"]
```

Se você passar `concat`um argumento que não é uma matriz, esse valor será adicionado à nova matriz como se fosse uma matriz de um elemento.

## Strings e suas propriedades

Podemos ler propriedades como `length`e `toUpperCase`de valores de string. Mas se você tentar adicionar uma nova propriedade, ela não ficará.

```js
let kim = "Kim";
kim.age = 88;
console.log(kim.age);
// → undefined
```

Valores do tipo string, number e Boolean não são objetos e, embora o idioma não reclame se você tentar definir novas propriedades neles, na verdade não armazena essas propriedades. Como mencionado anteriormente, esses valores são imutáveis e não podem ser alterados.

Mas esses tipos têm propriedades internas. Todo valor de string possui vários métodos. Alguns muito úteis são `slice`e `indexOf`, que se assemelham aos métodos de matriz com o mesmo nome.

```js
console.log("coconuts".slice(4, 7));
// → nut
console.log("coconut".indexOf("u"));
// → 5
```

Uma diferença é que uma string `indexOf`pode procurar uma string contendo mais de um caractere, enquanto o método de matriz correspondente procura apenas um único elemento.

```js
console.log("one two three".indexOf("ee"));
// → 11
```

O `trim`método remove os espaços em branco (espaços, novas linhas, guias e caracteres semelhantes) do início e do fim de uma sequência.

```js
console.log("  okay \n ".trim());
// → okay
```

A `zeroPad`função do [capítulo anterior](https://eloquentjavascript.net/03_functions.html) também existe como método. É chamado `padStart`e usa o comprimento desejado e o caractere de preenchimento como argumentos.

```js
console . log ( String ( 6 ). padStart ( 3 , "0" ));
// → 006
```

Você pode dividir uma string em todas as ocorrências de outra string `split`e associá-la novamente `join`.

```js
let sentence = "Secretarybirds specialize in stomping";
let words = sentence.split(" ");
console.log(words);
// → ["Secretarybirds", "specialize", "in", "stomping"]
console.log(words.join(". "));
// → Secretarybirds. specialize. in. stomping
```

Uma sequência pode ser repetida com o `repeat`método, que cria uma nova sequência contendo várias cópias da sequência original, coladas.

```js
console.log("LA".repeat(3));
// → LALALA
```

Já vimos a `length`propriedade do tipo string . Acessar os caracteres individuais em uma string parece acessar elementos de array (com uma ressalva que discutiremos no [Capítulo 5](https://eloquentjavascript.net/05_higher_order.html#code_units) ).

```js
let string = "abc";
console.log(string.length);
// → 3
console.log(string[1]);
// → b
```

## Parâmetros de descanso

Pode ser útil para uma função aceitar qualquer número de argumentos. Por exemplo, `Math.max`calcula o máximo de *todos* os argumentos dados.

Para escrever uma função, coloque três pontos antes do último parâmetro da função, assim:

```js
function max(...numbers) {
  let result = -Infinity;
  for (let number of numbers) {
    if (number > result) result = number;
  }
  return result;
}
console.log(max(4, 1, 9, -2));
// → 9
```

Quando essa função é chamada, o *parâmetro rest* é vinculado a uma matriz que contém todos os argumentos adicionais. Se houver outros parâmetros antes, seus valores não farão parte dessa matriz. Quando, como em `max`, é o único parâmetro, ele mantém todos os argumentos.

Você pode usar uma notação de três pontos semelhante para *chamar* uma função com uma matriz de argumentos.

```js
let numbers = [5, 1, 7];
console.log(max(...numbers));
// → 7
```

Isso "espalha" a matriz na chamada de função, passando seus elementos como argumentos separados. É possível incluir uma matriz como essa junto com outros argumentos, como em .`max(9, ...numbers, 2)`

A notação de matriz de colchete permite que o operador de ponto triplo espalhe outra matriz na nova matriz.

```js
let words = ["never", "fully"];
console.log(["will", ...words, "understand"]);
// → ["will", "never", "fully", "understand"]
```

## O objeto Math

Como vimos, `Math`há um monte de funções utilitárias relacionadas a números, como `Math.max`(máximo), `Math.min`(mínimo) e `Math.sqrt`(raiz quadrada).

O `Math`objeto é usado como um contêiner para agrupar várias funcionalidades relacionadas. Existe apenas um `Math`objeto, e quase nunca é útil como um valor. Em vez disso, fornece um *espaço* para *nome* para que todas essas funções e valores não precisem ser ligações globais.

Ter muitas ligações globais "polui" o espaço para nome. Quanto mais nomes forem usados, maior será a probabilidade de substituir acidentalmente o valor de alguma ligação existente. Por exemplo, é improvável que você queira nomear algo `max`em um de seus programas. Como a `max`função interna do JavaScript está escondida com segurança dentro do `Math`objeto, não precisamos nos preocupar em substituí-lo.

Muitos idiomas o impedirão, ou pelo menos o alertarão, quando você estiver definindo uma ligação com um nome que já esteja sendo usado. O JavaScript faz isso para ligações que você declarou com `let`ou `const`mas - perversamente - não para ligações padrão nem para ligações declaradas com `var`ou `function`.

De volta ao `Math`objeto. Se você precisar fazer trigonometria, `Math`pode ajudar. Ele contém `cos`(co-seno), `sin`(seno), e `tan`(tangente), bem como as suas funções inversas, `acos`, `asin`, e `atan`, respectivamente. O número π (pi) - ou pelo menos a aproximação mais próxima que se encaixa em um número JavaScript - está disponível como `Math.PI`. Existe uma antiga tradição de programação de escrever os nomes dos valores constantes em todas as maiúsculas.

```js
function randomPointOnCircle(radius) {
  let angle = Math.random() * 2 * Math.PI;
  return {x: radius * Math.cos(angle),
          y: radius * Math.sin(angle)};
}
console.log(randomPointOnCircle(2));
// → {x: 0.3667, y: 1.966}
```

Se seno e cosseno não são algo com que você esteja familiarizado, não se preocupe. Quando eles são usados neste livro, no [Capítulo 14](https://eloquentjavascript.net/14_dom.html#sin_cos) , eu os explicarei.

O exemplo anterior usado `Math.random`. Essa é uma função que retorna um novo número pseudoaleatório entre zero (inclusive) e um (exclusivo) toda vez que você o chama.

```js
console.log(Math.random());
// → 0.36993729369714856
console.log(Math.random());
// → 0.727367032552138
console.log(Math.random());
// → 0.40180766698904335
```

Embora os computadores sejam máquinas determinísticas - eles sempre reagem da mesma maneira se recebem a mesma entrada - é possível fazê-los produzir números que parecem aleatórios. Para fazer isso, a máquina mantém algum valor oculto e, sempre que você solicita um novo número aleatório, realiza cálculos complicados nesse valor oculto para criar um novo valor. Ele armazena um novo valor e retorna um número derivado dele. Dessa forma, ele pode produzir números sempre novos e difíceis de prever de uma maneira que *pareça* aleatória.

Se queremos um número aleatório inteiro em vez de fracionário, podemos usar `Math.floor`(que arredonda para o número inteiro mais próximo) no resultado de `Math.random`.

```js
console.log(Math.floor(Math.random() * 10));
// → 2
```

Multiplicar o número aleatório por 10 nos dá um número maior ou igual a 0 e abaixo de 10. Como `Math.floor`arredondamentos para baixo, essa expressão produzirá, com igual chance, qualquer número de 0 a 9.

Existem também as funções `Math.ceil`(para “teto”, que arredondam para um número inteiro), `Math.round`(para o número inteiro mais próximo) e `Math.abs`, que recebe o valor absoluto de um número, o que significa que nega valores negativos, mas deixa positivos quando estamos.

## Desestruturação

Vamos voltar à `phi`função por um momento.

```js
function phi(table) {
  return (table[3] * table[0] - table[2] * table[1]) /
    Math.sqrt((table[2] + table[3]) *
              (table[0] + table[1]) *
              (table[1] + table[3]) *
              (table[0] + table[2]));
}
```

Uma das razões pelas quais essa função é difícil de ler é que temos uma ligação apontando para nossa matriz, mas preferimos muito ter ligações para os *elementos* da matriz, ou seja, `let n00 = table[0]`e assim por diante. Felizmente, existe uma maneira sucinta de fazer isso em JavaScript.

```js
function phi([n00, n01, n10, n11]) {
  return (n11 * n00 - n10 * n01) /
    Math.sqrt((n10 + n11) * (n00 + n01) *
              (n01 + n11) * (n00 + n10));
}
```

Isso também funciona para ligações criadas com `let`, `var`ou `const`. Se você souber que o valor que está vinculando é uma matriz, use colchetes para "olhar para dentro" do valor, vinculando seu conteúdo.

Um truque semelhante funciona para objetos, usando chaves em vez de colchetes.

```js
let {name} = {name: "Faraji", age: 23};
console.log(name);
// → Faraji
```

Observe que, se você tentar desestruturar `null`ou `undefined`, receberá um erro, da mesma forma que faria se tentasse acessar diretamente uma propriedade desses valores.

## JSON

Como as propriedades apenas apreendem seu valor, em vez de contê-lo, objetos e matrizes são armazenados na memória do computador como sequências de bits contendo os *endereços* - o local na memória - de seu conteúdo. Portanto, uma matriz com outra matriz dentro dela consiste em (pelo menos) uma região de memória para a matriz interna e outra para a matriz externa, contendo (entre outras coisas) um número binário que representa a posição da matriz interna.

Se você deseja salvar dados em um arquivo para mais tarde ou enviá-los para outro computador pela rede, é necessário converter esses emaranhados de endereços de memória em uma descrição que possa ser armazenada ou enviada. Você *poderia* enviar toda a memória do computador, juntamente com o endereço do valor em que está interessado, suponho, mas essa não parece ser a melhor abordagem.

O que podemos fazer é *serializar* os dados. Isso significa que é convertido em uma descrição simples. Um formato de serialização popular é chamado *JSON* (pronuncia-se "Jason"), que significa JavaScript Object Notation. É amplamente usado como armazenamento de dados e formato de comunicação na Web, mesmo em idiomas diferentes do JavaScript.

O JSON é semelhante à maneira do JavaScript de escrever matrizes e objetos, com algumas restrições. Todos os nomes de propriedades devem estar entre aspas duplas e apenas expressões de dados simples são permitidas - nenhuma chamada de função, vinculação ou qualquer coisa que envolva computação real. Comentários não são permitidos no JSON.

Uma entrada no diário pode parecer assim quando representada como dados JSON:

```js
{
  "squirrel": false,
  "events": ["work", "touched tree", "pizza", "running"]
}

```

O JavaScript fornece as funções `JSON.stringify`e `JSON.parse`a conversão de dados para e deste formato. O primeiro pega um valor JavaScript e retorna uma sequência codificada em JSON. O segundo pega essa string e a converte no valor que codifica.

```js
let string = JSON.stringify({squirrel: false,
                             events: ["weekend"]});
console.log(string);
// → {"squirrel":false,"events":["weekend"]}
console.log(JSON.parse(string).events);
// → ["weekend"]
```

## Sumário

Objetos e matrizes (que são um tipo específico de objeto) fornecem maneiras de agrupar vários valores em um único valor. Conceitualmente, isso nos permite colocar um monte de coisas relacionadas em uma bolsa e correr com ela, em vez de envolver nossos braços em torno de todas as coisas individuais e tentar segurá-las separadamente.

A maioria dos valores em JavaScript tem propriedades, sendo as exceções `null`e `undefined`. As propriedades são acessadas usando `value.prop`ou `value["prop"]`. Os objetos tendem a usar nomes para suas propriedades e armazenar mais ou menos um conjunto fixo deles. As matrizes, por outro lado, geralmente contêm quantidades variadas de valores conceitualmente idênticos e usam números (começando em 0) como os nomes de suas propriedades.

Não *são* algumas propriedades nomeadas em matrizes, tais como a `length`e um número de métodos. Métodos são funções que vivem em propriedades e (geralmente) agem sobre o valor de que são propriedades.

Você pode iterar sobre matrizes usando um tipo especial de `for`loop— `for (let element of array)`.

## Exercícios

### A soma de um intervalo

A [introdução](https://eloquentjavascript.net/00_intro.html) deste livro mencionou o seguinte como uma boa maneira de calcular a soma de um intervalo de números:

```js
console.log(sum(range(1, 10)));
```

Escreva uma `range`função que aceite dois argumentos, `start`e `end`, e retorne uma matriz contendo todos os números de `start`até (e inclusive) `end`.

Em seguida, escreva uma `sum`função que pegue uma matriz de números e retorne a soma desses números. Execute o programa de exemplo e veja se ele realmente retorna 55.

Como atribuição de bônus, modifique sua `range`função para usar um terceiro argumento opcional que indica o valor de "etapa" usado ao criar a matriz. Se nenhum passo for dado, os elementos aumentam em incrementos de um, correspondendo ao antigo comportamento. A chamada de função `range(1, 10, 2)`deve retornar `[1, 3, 5, 7, 9]`. Certifique-se de que ele também funcione com valores de etapa negativos, para que `range(5, 2, -1)`produza `[5, 4, 3, 2]`.

```js
// Your code here.

console.log(range(1, 10));
// → [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
console.log(range(5, 2, -1));
// → [5, 4, 3, 2]
console.log(sum(range(1, 10)));
// → 55
```

A criação de uma matriz é mais fácil, inicializando uma ligação em `[]`(uma matriz nova e vazia) e chamando repetidamente seu `push`método para adicionar um valor. Não se esqueça de retornar a matriz no final da função.

Como o limite final é inclusivo, você precisará usar o `<=`operador em vez de `<`verificar o final do seu loop.

O parâmetro step pode ser um parâmetro opcional que padroniza (usando o `=`operador) para 1.

Tendo `range`compreender valores negativos passo provavelmente o melhor é feito escrevendo dois loops e um para a contagem ascendente e um separados para contagem para baixo, porque a comparação que verifica se o circuito é necessidades acabados de ser `>=`, em vez de `<=`quando a contagem para baixo.

Também pode valer a pena usar uma etapa padrão diferente, ou seja, -1, quando o final do intervalo for menor que o início. Dessa forma, `range(5, 2)`retorna algo significativo, em vez de ficar preso em um loop infinito. É possível consultar os parâmetros anteriores no valor padrão de um parâmetro.

### Invertendo uma matriz

Matrizes têm um `reverse`método que altera a matriz invertendo a ordem em que seus elementos aparecem. Para este exercício, escreva duas funções `reverseArray`e `reverseArrayInPlace`. O primeiro, `reverseArray`pega uma matriz como argumento e produz uma *nova* matriz que possui os mesmos elementos na ordem inversa. O segundo `reverseArrayInPlace`,, faz o que o `reverse`método faz: *modifica* o array dado como argumento, revertendo seus elementos. Nenhum dos dois pode usar o `reverse`método padrão .

Voltando às notas sobre efeitos colaterais e funções puras no [capítulo anterior](https://eloquentjavascript.net/03_functions.html#pure) , qual variante você espera que seja útil em mais situações? Qual deles corre mais rápido?

```js
// Your code here.

console.log(reverseArray(["A", "B", "C"]));
// → ["C", "B", "A"];
let arrayValue = [1, 2, 3, 4, 5];
reverseArrayInPlace(arrayValue);
console.log(arrayValue);
// → [5, 4, 3, 2, 1]
```

Existem duas maneiras óbvias de implementar `reverseArray`. O primeiro é simplesmente passar o array de entrada da frente para trás e usar o `unshift`método no novo array para inserir cada elemento no início. O segundo é fazer um loop na matriz de entrada para trás e usar o `push`método A iteração de uma matriz para trás requer uma `for`especificação (um tanto estranha) , como .

```js
(let i = array.length - 1; i >= 0; i--)
```

Inverter a matriz no lugar é mais difícil. Você deve tomar cuidado para não sobrescrever os elementos necessários posteriormente. O uso `reverseArray`ou cópia de toda a matriz ( `array.slice(0)`é uma boa maneira de copiar uma matriz) funciona, mas está enganando.

O truque é *trocar* o primeiro e o último elementos, depois o segundo e o penúltimo, e assim por diante. Você pode fazer isso repetindo mais da metade do comprimento da matriz (use `Math.floor`para arredondar para baixo - você não precisa tocar no elemento do meio em uma matriz com um número ímpar de elementos) e trocando o elemento na posição pelo elemento `i`na posição . Você pode usar uma ligação local para segurar brevemente um dos elementos, substituí-lo por sua imagem espelhada e, em seguida, colocar o valor da ligação local no local onde a imagem espelhada costumava estar.`array.length - 1 - i`

### Uma lista

Objetos, como blobs genéricos de valores, podem ser usados para criar todos os tipos de estruturas de dados. Uma estrutura de dados comum é a *lista* (não deve ser confundida com a matriz). Uma lista é um conjunto aninhado de objetos, com o primeiro objeto mantendo uma referência ao segundo, o segundo ao terceiro e assim por diante.

```js
let list = {
  value: 1,
  rest: {
    value: 2,
    rest: {
      value: 3,
      rest: null
    }
  }
};
```

Os objetos resultantes formam uma cadeia, assim:

![Uma lista vinculada](https://eloquentjavascript.net/img/linked-list.svg)

Uma coisa boa das listas é que elas podem compartilhar partes de sua estrutura. Por exemplo, se eu criar dois novos valores `{value: 0, rest: list}`e `{value: -1, rest: list}`(com `list`referência à ligação definida anteriormente), eles serão listas independentes, mas compartilharão a estrutura que compõe seus três últimos elementos. A lista original também ainda é uma lista de três elementos válida.

Escreva uma função `arrayToList`que construa uma estrutura de lista como a mostrada quando fornecida `[1, 2, 3]`como argumento. Escreva também uma `listToArray`função que produz uma matriz a partir de uma lista. Em seguida, adicione uma função auxiliar `prepend`, que pega um elemento e uma lista e cria uma nova lista que adiciona o elemento à frente da lista de entrada e `nth`, que pega uma lista e um número e retorna o elemento na posição especificada na lista (com zero referente ao primeiro elemento) ou `undefined`quando não houver esse elemento.

Se você ainda não o fez, escreva também uma versão recursiva de `nth`.

```js
// Your code here.

console.log(arrayToList([10, 20]));
// → {value: 10, rest: {value: 20, rest: null}}
console.log(listToArray(arrayToList([10, 20, 30])));
// → [10, 20, 30]
console.log(prepend(10, prepend(20, null)));
// → {value: 10, rest: {value: 20, rest: null}}
console.log(nth(arrayToList([10, 20, 30]), 1));
// → 20
```

A criação de uma lista é mais fácil quando feita de trás para frente. Portanto, `arrayToList`poderia iterar a matriz para trás (consulte o exercício anterior) e, para cada elemento, adicionar um objeto à lista. Você pode usar uma ligação local para reter a parte da lista criada até o momento e usar uma atribuição como `list = {value: X, rest: list}`para adicionar um elemento.

Para executar uma lista (in `listToArray`e `nth`), uma `for`especificação de loop como esta pode ser usada:

```js
for (let node = list; node; node = node.rest) {}
```

Você pode ver como isso funciona? Toda iteração do loop `node`aponta para a sub-lista atual e o corpo pode ler sua `value`propriedade para obter o elemento atual. No final de uma iteração, `node`passa para a próxima sublist. Quando isso é nulo, chegamos ao final da lista e o loop está concluído.

A versão recursiva do `nth`vontade, da mesma forma, olha para uma parte cada vez menor da "cauda" da lista e, ao mesmo tempo, faz a contagem regressiva do índice até chegar a zero; nesse ponto, ele pode retornar a `value`propriedade do nó que está procurando. às. Para obter o elemento zeroth de uma lista, você simplesmente assume a `value`propriedade de seu nó principal. Para obter o elemento *N* + 1, você pega o *N* ésimo elemento da lista que está na `rest`propriedade desta lista .

### Comparação profunda

O `==`operador compara objetos por identidade. Mas às vezes você prefere comparar os valores de suas propriedades reais.

Escreva uma função `deepEqual`que aceite dois valores e retorne true somente se eles tiverem o mesmo valor ou forem objetos com as mesmas propriedades, em que os valores das propriedades são iguais quando comparados a uma chamada recursiva para `deepEqual`.

Para descobrir se os valores devem ser comparados diretamente (use o `===`operador para isso) ou se suas propriedades são comparadas, você pode usar o `typeof`operador. Se produz `"object"`para ambos os valores, você deve fazer uma comparação profunda. Mas você deve levar em consideração uma exceção boba: por causa de um acidente histórico, `typeof null`também produz `"object"`.

A `Object.keys`função será útil quando você precisar examinar as propriedades dos objetos para compará-las.

```js
// Your code here.

let obj = {here: {is: "an"}, object: 2};
console.log(deepEqual(obj, obj));
// → true
console.log(deepEqual(obj, {here: 1, object: 2}));
// → false
console.log(deepEqual(obj, {here: {is: "an"}, object: 2}));
// → true
```

Seu teste para saber se você está lidando com um objeto real será semelhante a `typeof x == "object" && x != null`. Cuidado para comparar propriedades apenas quando os *dois* argumentos forem objetos. Em todos os outros casos, você pode retornar imediatamente o resultado da aplicação `===`.

Use `Object.keys`para revisar as propriedades. Você precisa testar se os dois objetos têm o mesmo conjunto de nomes de propriedades e se essas propriedades têm valores idênticos. Uma maneira de fazer isso é garantir que os dois objetos tenham o mesmo número de propriedades (os comprimentos das listas de propriedades são os mesmos). E então, ao fazer um loop sobre uma das propriedades do objeto para compará-las, sempre verifique se o outro realmente tem uma propriedade com esse nome. Se eles tiverem o mesmo número de propriedades e todas as propriedades em uma também existirem na outra, elas terão o mesmo conjunto de nomes de propriedades.

É melhor retornar o valor correto da função retornando false imediatamente quando uma incompatibilidade for encontrada e retornando true no final da função.