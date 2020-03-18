# Capítulo 6 A Vida Secreta dos Objetos

> Um tipo de dado abstrato é realizado escrevendo um tipo especial de programa […] que define o tipo em termos das operações que podem ser executadas nele.
>
> *—Barbara Liskov, Programação com tipos de dados abstratos*

![Imagem de um coelho com seu proto-coelho](https://eloquentjavascript.net/img/chapter_picture_6.jpg)

[O capítulo 4](https://eloquentjavascript.net/04_data.html) apresentou os objetos do JavaScript. Na cultura de programação, temos uma coisa chamada *programação orientada a objetos* , um conjunto de técnicas que usam objetos (e conceitos relacionados) como o princípio central da organização do programa.

Embora ninguém realmente concorde com sua definição precisa, a programação orientada a objetos modelou o design de muitas linguagens de programação, incluindo JavaScript. Este capítulo descreve como essas idéias podem ser aplicadas em JavaScript.

## Encapsulamento

A idéia central da programação orientada a objetos é dividir os programas em partes menores e responsabilizar cada parte pelo gerenciamento de seu próprio estado.

Dessa forma, alguns conhecimentos sobre o funcionamento de uma parte do programa podem ser mantidos *locais* nessa parte. Alguém que trabalha no restante do programa não precisa se lembrar ou mesmo estar ciente desse conhecimento. Sempre que esses detalhes locais são alterados, apenas o código diretamente ao seu redor precisa ser atualizado.

Diferentes partes desse programa interagem entre si por meio de *interfaces* , conjuntos limitados de funções ou ligações que fornecem funcionalidade útil em um nível mais abstrato, ocultando sua implementação precisa.

Essas partes do programa são modeladas usando objetos. Sua interface consiste em um conjunto específico de métodos e propriedades. As propriedades que fazem parte da interface são chamadas *públicas* . Os outros, que o código externo não deve tocar, são chamados de *particulares* .

Muitos idiomas fornecem uma maneira de distinguir propriedades públicas e privadas e impedir que códigos externos acessem completamente as particulares. O JavaScript, mais uma vez adotando a abordagem minimalista, não - ainda não pelo menos. Há trabalho em andamento para adicionar isso ao idioma.

Mesmo que a linguagem não tenha essa distinção embutida, os programadores de JavaScript *estão* usando com êxito essa ideia. Normalmente, a interface disponível é descrita na documentação ou nos comentários. Também é comum colocar um caractere sublinhado ( `_`) no início dos nomes das propriedades para indicar que essas propriedades são privadas.

Separar a interface da implementação é uma ótima ideia. Geralmente é chamado de *encapsulamento* .

## Métodos

Métodos nada mais são do que propriedades que mantêm valores de função. Este é um método simples:

```js
let rabbit = {};
rabbit.speak = function(line) {
  console.log(`The rabbit says '${line}'`);
};

rabbit.speak("I'm alive.");
// → The rabbit says 'I'm alive.'
```

Geralmente, um método precisa fazer algo com o objeto em que foi chamado. Quando uma função é chamada como método - procurada como uma propriedade e imediatamente chamada, como em `object.method()`-, a ligação chamada `this`em seu corpo aponta automaticamente para o objeto em que foi chamada.

```js
function speak(line) {
  console.log(`The ${this.type} rabbit says '${line}'`);
}
let whiteRabbit = {type: "white", speak};
let hungryRabbit = {type: "hungry", speak};

whiteRabbit.speak("Oh my ears and whiskers, " +
                  "how late it's getting!");
// → The white rabbit says 'Oh my ears and whiskers, how
//   late it's getting!'
hungryRabbit.speak("I could use a carrot right now.");
// → The hungry rabbit says 'I could use a carrot right now.'
```

Você pode pensar `this`em um parâmetro extra que é passado de uma maneira diferente. Se você deseja passá-lo explicitamente, pode usar o `call`método de uma função , que aceita o `this`valor como seu primeiro argumento e trata outros argumentos como parâmetros normais.

```js
speak.call(hungryRabbit, "Burp!");
// → The hungry rabbit says 'Burp!'
```

Como cada função tem sua própria `this`ligação, cujo valor depende da maneira como é chamada, você não pode se referir ao `this`escopo de quebra automática em uma função regular definida com a `function`palavra - chave.

As funções de seta são diferentes - elas não se vinculam, `this`mas podem ver a `this`ligação do escopo ao seu redor. Assim, você pode fazer algo como o seguinte código, que faz referência `this`de dentro de uma função local:

```js
function normalize() {
  console.log(this.coords.map(n => n / this.length));
}
normalize.call({coords: [0, 2, 3], length: 5});
// → [0, 0.4, 0.6]
```

Se eu tivesse escrito o argumento para `map`usar a `function`palavra - chave, o código não funcionaria.

## Prototypes

Assista de perto.

```
let empty = {};
console.log(empty.toString);
// → function toString(){…}
console.log(empty.toString());
// → [object Object]
```

Tirei uma propriedade de um objeto vazio. Magia!

Bem, na verdade não. Simplesmente retive informações sobre o funcionamento dos objetos JavaScript. Além do conjunto de propriedades, a maioria dos objetos também possui um *protótipo* . Um protótipo é outro objeto que é usado como uma fonte alternativa de propriedades. Quando um objeto obtém uma solicitação para uma propriedade que não possui, seu protótipo será pesquisado pela propriedade, depois o protótipo do protótipo e assim por diante.

Então, quem é o protótipo desse objeto vazio? É o grande protótipo ancestral, a entidade por trás de quase todos os objetos `Object.prototype`.

```js
console.log(Object.getPrototypeOf({}) ==
            Object.prototype);
// → true
console.log(Object.getPrototypeOf(Object.prototype));
// → null
```

Como você adivinha, retorna o protótipo de um objeto.`Object.getPrototypeOf`

As relações de protótipo dos objetos JavaScript formam uma estrutura em forma de árvore e, na raiz dessa estrutura, fica `Object.prototype`. Ele fornece alguns métodos que aparecem em todos os objetos, como, por exemplo `toString`, o que converte um objeto em uma representação de string.

Muitos objetos não têm diretamente `Object.prototype`como protótipo, mas têm outro objeto que fornece um conjunto diferente de propriedades padrão. Funções derivam de e matrizes derivam de .`Function.prototype``Array.prototype`

```js
console.log(Object.getPrototypeOf(Math.max) ==
            Function.prototype);
// → true
console.log(Object.getPrototypeOf([]) ==
            Array.prototype);
// → true
```

Esse objeto de protótipo, por si só, terá um protótipo, frequentemente `Object.prototype`, para que ele ainda indiretamente forneça métodos como `toString`.

Você pode usar `Object.create`para criar um objeto com um protótipo específico.

```js
let protoRabbit = {
  speak(line) {
    console.log(`The ${this.type} rabbit says '${line}'`);
  }
};
let killerRabbit = Object.create(protoRabbit);
killerRabbit.type = "killer";
killerRabbit.speak("SKREEEE!");
// → The killer rabbit says 'SKREEEE!'
```

Uma propriedade como `speak(line)`em uma expressão de objeto é uma maneira abreviada de definir um método. Ele cria uma propriedade chamada `speak`e atribui a ela uma função como valor.

O coelho “proto” atua como um recipiente para as propriedades que são compartilhadas por todos os coelhos. Um objeto de coelho individual, como o coelho assassino, contém propriedades que se aplicam apenas a si mesmo - nesse caso, seu tipo - e deriva propriedades compartilhadas de seu protótipo.

## Classes

O sistema de protótipo do JavaScript pode ser interpretado como uma abordagem um tanto informal de um conceito orientado a objetos chamado *classes* . Uma classe define a forma de um tipo de objeto - quais métodos e propriedades ele possui. Esse objeto é chamado de *instância* da classe.

Os protótipos são úteis para definir propriedades para as quais todas as instâncias de uma classe compartilham o mesmo valor, como métodos. Propriedades que diferem por exemplo, como a `type`propriedade de nossos coelhos , precisam ser armazenadas diretamente nos próprios objetos.

Portanto, para criar uma instância de uma determinada classe, você deve criar um objeto derivado do protótipo adequado, mas *também* deve garantir que ele próprio possua as propriedades que as instâncias dessa classe devem ter. É isso que uma função *construtora* faz.

```js
function makeRabbit(type) {
  let rabbit = Object.create(protoRabbit);
  rabbit.type = type;
  return rabbit;
}
```

O JavaScript fornece uma maneira de facilitar a definição desse tipo de função. Se você colocar a palavra-chave `new`na frente de uma chamada de função, a função será tratada como um construtor. Isso significa que um objeto com o protótipo correto é criado automaticamente, vinculado à `this`função e retornado no final da função.

O objeto de protótipo usado ao construir objetos é encontrado usando a `prototype`propriedade da função construtora.

```js
function Rabbit(type) {
  this.type = type;
}
Rabbit.prototype.speak = function(line) {
  console.log(`The ${this.type} rabbit says '${line}'`);
};

let weirdRabbit = new Rabbit("weird");
```

Os construtores (todas as funções, de fato) obtêm automaticamente uma propriedade denominada `prototype`, que, por padrão, mantém um objeto simples e vazio do qual deriva `Object.prototype`. Você pode substituí-lo por um novo objeto, se desejar. Ou você pode adicionar propriedades ao objeto existente, como o exemplo.

Por convenção, os nomes dos construtores são escritos em maiúsculas para que possam ser facilmente distinguidos de outras funções.

É importante entender a distinção entre a maneira como um protótipo é associado a um construtor (por meio de sua `prototype`propriedade) e a maneira como os objetos *têm* um protótipo (que pode ser encontrado com ). O protótipo real de um construtor é que construtores são funções. Sua *propriedade* mantém o protótipo usado para instâncias criadas por ele.`Object.getPrototypeOf``Function.prototype``prototype`

```js
console.log(Object.getPrototypeOf(Rabbit) ==
            Function.prototype);
// → true
console.log(Object.getPrototypeOf(weirdRabbit) ==
            Rabbit.prototype);
// → true
```

## Notação de Classe

Portanto, as classes JavaScript são funções construtoras com uma propriedade prototype. É assim que eles funcionam e, até 2015, era assim que você os escrevia. Hoje em dia, temos uma notação menos embaraçosa.

```js
class Rabbit {
  constructor(type) {
    this.type = type;
  }
  speak(line) {
    console.log(`The ${this.type} rabbit says '${line}'`);
  }
}

let killerRabbit = new Rabbit("killer");
let blackRabbit = new Rabbit("black");
```

A `class`palavra-chave inicia uma declaração de classe, o que nos permite definir um construtor e um conjunto de métodos em um único local. Qualquer número de métodos pode ser escrito dentro dos colchetes da declaração. O nomeado `constructor`é tratado especialmente. Ele fornece a função real do construtor, que será vinculada ao nome `Rabbit`. Os outros são empacotados no protótipo desse construtor. Portanto, a declaração de classe anterior é equivalente à definição de construtor da seção anterior. Parece melhor.

Atualmente, as declarações de classe permitem que apenas *métodos -* propriedades que mantêm funções - sejam adicionados ao protótipo. Isso pode ser um pouco inconveniente quando você deseja salvar um valor não funcional aqui. A próxima versão do idioma provavelmente melhorará isso. Por enquanto, você pode criar essas propriedades manipulando diretamente o protótipo após definir a classe.

Como `function`, `class`pode ser usado tanto em declarações quanto em expressões. Quando usado como uma expressão, ele não define uma ligação, mas apenas produz o construtor como um valor. Você tem permissão para omitir o nome da classe em uma expressão de classe.

```js
let object = new class { getWord() { return "hello"; } };
console.log(object.getWord());
// → hello
```

## Substituindo propriedades derivadas

Quando você adiciona uma propriedade a um objeto, esteja presente no protótipo ou não, a propriedade é adicionada ao *próprio* objeto . Se já havia uma propriedade com o mesmo nome no protótipo, essa propriedade não afetará mais o objeto, pois agora está oculta por trás da propriedade do próprio objeto.

```js
Rabbit.prototype.teeth = "small";
console.log(killerRabbit.teeth);
// → small
killerRabbit.teeth = "long, sharp, and bloody";
console.log(killerRabbit.teeth);
// → long, sharp, and bloody
console.log(blackRabbit.teeth);
// → small
console.log(Rabbit.prototype.teeth);
// → small
```

O diagrama a seguir descreve a situação após a execução desse código. Os protótipos `Rabbit`e `Object`ficam para trás `killerRabbit`como uma espécie de pano de fundo, onde propriedades que não são encontradas no próprio objeto podem ser consultadas.

![Esquema de protótipo de objeto de coelho](https://eloquentjavascript.net/img/rabbits.svg)

Substituir propriedades que existem em um protótipo pode ser útil. Como o exemplo dos dentes de coelho mostra, a substituição pode ser usada para expressar propriedades excepcionais em instâncias de uma classe de objetos mais genérica, enquanto permite que objetos não excepcionais obtenham um valor padrão de seu protótipo.

A substituição também é usada para fornecer aos protótipos de função e matriz padrão um `toString`método diferente do protótipo de objeto básico.

```js
console.log(Array.prototype.toString ==
            Object.prototype.toString);
// → false
console.log([1, 2].toString());
// → 1,2
```

A chamada `toString`de uma matriz fornece um resultado semelhante ao de uma chamada - coloca vírgulas entre os valores da matriz. Chamar diretamente com uma matriz produz uma sequência diferente. Essa função não sabe sobre matrizes, portanto, simplesmente coloca a palavra *objeto* e o nome do tipo entre colchetes.`.join(",")``Object.prototype.toString`

```
console.log(Object.prototype.toString.call([1, 2]));
// → [object Array]
```

## Maps

Vimos o *mapa de* palavras usado no [capítulo anterior](https://eloquentjavascript.net/05_higher_order.html#map) para uma operação que transforma uma estrutura de dados aplicando uma função a seus elementos. Por mais confuso que seja, na programação a mesma palavra também é usada para algo relacionado, mas bastante diferente.

Um *mapa* (substantivo) é uma estrutura de dados que associa valores (as chaves) a outros valores. Por exemplo, você pode mapear nomes para idades. É possível usar objetos para isso.

```js
let ages = {
  Boris: 39,
  Liang: 22,
  Júlia: 62
};

console.log(`Júlia is ${ages["Júlia"]}`);
// → Júlia is 62
console.log("Is Jack's age known?", "Jack" in ages);
// → Is Jack's age known? false
console.log("Is toString's age known?", "toString" in ages);
// → Is toString's age known? true
```

Aqui, os nomes de propriedades do objeto são os nomes das pessoas e os valores das propriedades são suas idades. Mas certamente não listamos ninguém chamado toString em nosso mapa. No entanto, como objetos simples derivam `Object.prototype`, parece que a propriedade está lá.

Como tal, usar objetos simples como mapas é perigoso. Existem várias maneiras possíveis de evitar esse problema. Primeiro, é possível criar objetos *sem* protótipo. Se você passar `null`para `Object.create`, o objeto resultante não será derivado `Object.prototype`e poderá ser usado com segurança como um mapa.

```js
console.log("toString" in Object.create(null));
// → false
```

Os nomes de propriedades do objeto devem ser cadeias de caracteres. Se você precisar de um mapa cujas chaves não possam ser facilmente convertidas em seqüências de caracteres, como objetos, não poderá usar um objeto como seu mapa.

Felizmente, o JavaScript vem com uma classe chamada `Map`que é escrita para esse fim exato. Ele armazena um mapeamento e permite qualquer tipo de chave.

```js
let ages = new Map();
ages.set("Boris", 39);
ages.set("Liang", 22);
ages.set("Júlia", 62);

console.log(`Júlia is ${ages.get("Júlia")}`);
// → Júlia is 62
console.log("Is Jack's age known?", ages.has("Jack"));
// → Is Jack's age known? false
console.log(ages.has("toString"));
// → false
```

Os métodos `set`, `get`e `has`são parte da interface do `Map`objecto. Escrever uma estrutura de dados que pode atualizar e pesquisar rapidamente um grande conjunto de valores não é fácil, mas não precisamos nos preocupar com isso. Outra pessoa fez isso por nós, e podemos passar por essa interface simples para usar o trabalho deles.

Se você tem um objeto claro que você precisa tratar como um mapa, por algum motivo, é útil saber que `Object.keys`retorna somente de um objeto *próprias* chaves, não aqueles no protótipo. Como alternativa ao `in`operador, você pode usar o `hasOwnProperty`método, que ignora o protótipo do objeto.

```js
console.log({x: 1}.hasOwnProperty("x"));
// → true
console.log({x: 1}.hasOwnProperty("toString"));
// → false
```

## Polimorfismo

Quando você chama a `String`função (que converte um valor em uma sequência) em um objeto, ela chama o `toString`método nesse objeto para tentar criar uma sequência significativa a partir dele. Mencionei que alguns dos protótipos padrão definem sua própria versão `toString`para que eles possam criar uma string que contenha informações mais úteis que `"[object Object]"`. Você também pode fazer isso sozinho.

```js
Rabbit.prototype.toString = function() {
  return `a ${this.type} rabbit`;
};

console.log(String(blackRabbit));
// → a black rabbit
```

Este é um exemplo simples de uma ideia poderosa. Quando um trecho de código é escrito para trabalhar com objetos que possuem uma certa interface - nesse caso, um `toString`método - qualquer tipo de objeto que suporte essa interface pode ser conectado ao código e funcionará.

Essa técnica é chamada *polimorfismo* . O código polimórfico pode funcionar com valores de diferentes formas, desde que eles suportem a interface esperada.

Mencionei no [Capítulo 4](https://eloquentjavascript.net/04_data.html#for_of_loop) que um loop `for`/ `of`pode fazer loop sobre vários tipos de estruturas de dados. Esse é outro caso de polimorfismo - esses loops esperam que a estrutura de dados exponha uma interface específica, o que as matrizes e cadeias de caracteres fazem. E também podemos adicionar essa interface aos seus próprios objetos! Mas antes que possamos fazer isso, precisamos saber o que são símbolos.

## Símbolos

É possível que várias interfaces usem o mesmo nome de propriedade para coisas diferentes. Por exemplo, eu poderia definir uma interface na qual o `toString`método deveria converter o objeto em um pedaço de fio. Não seria possível que um objeto estivesse em conformidade com essa interface e com o uso padrão de `toString`.

Isso seria uma péssima idéia e esse problema não é tão comum. A maioria dos programadores de JavaScript simplesmente não pensa nisso. Mas os designers de linguagem, cujo *trabalho* é pensar nessas coisas, nos forneceram uma solução de qualquer maneira.

Quando afirmei que os nomes de propriedades são strings, isso não era inteiramente preciso. Eles geralmente são, mas também podem ser *símbolos* . Símbolos são valores criados com a `Symbol`função Diferentemente das seqüências de caracteres, os símbolos recém-criados são únicos - você não pode criar o mesmo símbolo duas vezes.

```js
let sym = Symbol("name");
console.log(sym == Symbol("name"));
// → false
Rabbit.prototype[sym] = 55;
console.log(blackRabbit[sym]);
// → 55
```

A string que você passa `Symbol`é incluída quando você a converte em uma string e pode facilitar o reconhecimento de um símbolo quando, por exemplo, mostrá-lo no console. Mas não tem significado além disso - vários símbolos podem ter o mesmo nome.

Ser único e utilizável como nomes de propriedades torna os símbolos adequados para definir interfaces que podem viver pacificamente ao lado de outras propriedades, independentemente de seus nomes.

```js
const toStringSymbol = Symbol("toString");
Array.prototype[toStringSymbol] = function() {
  return `${this.length} cm of blue yarn`;
};

console.log([1, 2].toString());
// → 1,2
console.log([1, 2][toStringSymbol]());
// → 2 cm of blue yarn
```

É possível incluir propriedades de símbolo em expressões e classes de objetos usando colchetes ao redor do nome da propriedade. Isso faz com que o nome da propriedade seja avaliado, assim como a notação de acesso à propriedade entre colchetes, que nos permite fazer referência a uma ligação que contém o símbolo.

```js
let stringObject = {
  [toStringSymbol]() { return "a jute rope"; }
};
console.log(stringObject[toStringSymbol]());
// → a jute rope
```

## A interface do iterador

Espera-se que o objeto dado a um loop `for`/ `of`seja *iterável* . Isso significa que ele possui um método nomeado com o `Symbol.iterator`símbolo (um valor de símbolo definido pelo idioma, armazenado como uma propriedade da `Symbol`função).

Quando chamado, esse método deve retornar um objeto que fornece uma segunda interface, *iterador* . Essa é a coisa real que itera. Ele tem um `next`método que retorna o próximo resultado. Esse resultado deve ser um objeto com uma `value`propriedade que forneça o próximo valor, se houver, e uma `done`propriedade, que deve ser verdadeira quando não houver mais resultados e falsa caso contrário.

Note que o `next`, `value`e `done`nomes de propriedade são cadeias simples, e não símbolos. Somente `Symbol.iterator`, que provavelmente será adicionado a *muitos* objetos diferentes, é um símbolo real.

Nós podemos usar diretamente essa interface.

```js
let okIterator = "OK"[Symbol.iterator]();
console.log(okIterator.next());
// → {value: "O", done: false}
console.log(okIterator.next());
// → {value: "K", done: false}
console.log(okIterator.next());
// → {value: undefined, done: true}
```

Vamos implementar uma estrutura de dados iterável. Vamos construir uma classe de *matriz* , atuando como uma matriz bidimensional.

```js
class Matrix {
  constructor(width, height, element = (x, y) => undefined) {
    this.width = width;
    this.height = height;
    this.content = [];

    for (let y = 0; y < height; y++) {
      for (let x = 0; x < width; x++) {
        this.content[y * width + x] = element(x, y);
      }
    }
  }

  get(x, y) {
    return this.content[y * this.width + x];
  }
  set(x, y, value) {
    this.content[y * this.width + x] = value;
  }
}
```

A classe armazena seu conteúdo em uma única matriz de elementos *largura* × *altura* . Os elementos são armazenados linha por linha; portanto, por exemplo, o terceiro elemento na quinta linha é (usando a indexação com base em zero) armazenado na posição 4 × *largura* + 2.

A função construtora usa uma largura, uma altura e uma `element`função opcional que será usada para preencher os valores iniciais. Há `get`e `set`métodos para recuperar e elementos de atualização na matriz.

Quando looping sobre uma matriz, você está normalmente interessado na posição dos elementos, bem como os próprios elementos, por isso vamos ter a nossa iteradoras objetos produzir com `x`, `y`e `value`propriedades.

```js
class MatrixIterator {
  constructor(matrix) {
    this.x = 0;
    this.y = 0;
    this.matrix = matrix;
  }

  next() {
    if (this.y == this.matrix.height) return {done: true};

    let value = {x: this.x,
                 y: this.y,
                 value: this.matrix.get(this.x, this.y)};
    this.x++;
    if (this.x == this.matrix.width) {
      this.x = 0;
      this.y++;
    }
    return {value, done: false};
  }
}
```

A classe acompanha o progresso da iteração sobre uma matriz em suas propriedades `x`e `y`. O `next`método começa verificando se a parte inferior da matriz foi alcançada. Se não tem, ele *primeiro* cria o objeto mantendo o valor atual e *, em seguida,* atualiza sua posição, movendo-se para a próxima linha, se necessário.

Vamos configurar a `Matrix`classe para ser iterável. No decorrer deste livro, ocasionalmente usarei a manipulação de protótipos após o fato para adicionar métodos às classes, para que os trechos de código individuais permaneçam pequenos e independentes. Em um programa regular, onde não há necessidade de dividir o código em pequenos pedaços, você declara esses métodos diretamente na classe.

```
Matrix.prototype[Symbol.iterator] = function() {
  return new MatrixIterator(this);
};
```

Agora podemos fazer um loop sobre uma matriz com `for`/ `of`.

```
let matrix = new Matrix(2, 2, (x, y) => `value ${x},${y}`);
for (let {x, y, value} of matrix) {
  console.log(x, y, value);
}
// → 0 0 value 0,0
// → 1 0 value 1,0
// → 0 1 value 0,1
// → 1 1 value 1,1
```

## Getters, Setters e Statics

As interfaces geralmente consistem principalmente em métodos, mas também é bom incluir propriedades que mantêm valores não funcionais. Por exemplo, os `Map`objetos têm uma `size`propriedade que informa quantas chaves estão armazenadas neles.

Nem é necessário que esse objeto calcule e armazene essa propriedade diretamente na instância. Mesmo as propriedades acessadas diretamente podem ocultar uma chamada de método. Esses métodos são chamados de *getters* e são definidos escrevendo `get`na frente do nome do método em uma expressão de objeto ou declaração de classe.

```js
let varyingSize = {
  get size() {
    return Math.floor(Math.random() * 100);
  }
};

console.log(varyingSize.size);
// → 73
console.log(varyingSize.size);
// → 49
```

Sempre que alguém lê da `size`propriedade desse objeto , o método associado é chamado. Você pode fazer uma coisa semelhante quando uma propriedade é gravada, usando um *setter* .

```js
class Temperature {
  constructor(celsius) {
    this.celsius = celsius;
  }
  get fahrenheit() {
    return this.celsius * 1.8 + 32;
  }
  set fahrenheit(value) {
    this.celsius = (value - 32) / 1.8;
  }

  static fromFahrenheit(value) {
    return new Temperature((value - 32) / 1.8);
  }
}

let temp = new Temperature(22);
console.log(temp.fahrenheit);
// → 71.6
temp.fahrenheit = 86;
console.log(temp.celsius);
// → 30
```

A `Temperature`classe permite que você leia e escreva a temperatura em graus Celsius ou graus Fahrenheit, mas internamente armazena apenas Celsius e converte automaticamente de e para Celsius no `fahrenheit`getter e setter.

Às vezes, você deseja anexar algumas propriedades diretamente à sua função de construtor, e não ao protótipo. Esses métodos não terão acesso a uma instância de classe, mas podem, por exemplo, ser usados para fornecer maneiras adicionais de criar instâncias.

Dentro de uma declaração de classe, os métodos que foram `static`escritos antes do nome são armazenados no construtor. Portanto, a `Temperature`turma permite que você escreva para criar uma temperatura usando graus Fahrenheit.`Temperature.fromFahrenheit(100)`

## Herança

Algumas matrizes são conhecidas por serem *simétricas* . Se você espelhar uma matriz simétrica em torno de sua diagonal do canto superior esquerdo para o canto inferior direito, ela permanecerá a mesma. Em outras palavras, o valor armazenado em *x* , *y* é sempre o mesmo que em *y* , *x* .

Imagine que precisamos de uma estrutura de dados semelhante à `Matrix`que reforce o fato de que a matriz é e permanece simétrica. Poderíamos escrever do zero, mas isso envolveria a repetição de um código muito semelhante ao que já escrevemos.

O sistema de protótipo do JavaScript possibilita a criação de uma *nova* classe, semelhante à antiga, mas com novas definições para algumas de suas propriedades. O protótipo para a nova classe deriva do antigo protótipo, mas adiciona uma nova definição para, digamos, o `set`método.

Em termos de programação orientada a objetos, isso é chamado de *herança* . A nova classe herda propriedades e comportamento da classe antiga.

```js
class SymmetricMatrix extends Matrix {
  constructor(size, element = (x, y) => undefined) {
    super(size, size, (x, y) => {
      if (x < y) return element(y, x);
      else return element(x, y);
    });
  }

  set(x, y, value) {
    super.set(x, y, value);
    if (x != y) {
      super.set(y, x, value);
    }
  }
}

let matrix = new SymmetricMatrix(5, (x, y) => `${x},${y}`);
console.log(matrix.get(2, 3));
// → 3,2
```

O uso da palavra `extends`indica que essa classe não deve ser diretamente baseada no `Object`protótipo padrão, mas em alguma outra classe. Isso é chamado de *superclasse* . A classe derivada é a *subclasse* .

Para inicializar uma `SymmetricMatrix`instância, o construtor chama o construtor de sua superclasse por meio da `super`palavra - chave Isso é necessário porque se esse novo objeto se comportar (aproximadamente) como um `Matrix`, precisará das propriedades da instância que as matrizes possuem. Para garantir que a matriz seja simétrica, o construtor envolve a `element`função para trocar as coordenadas por valores abaixo da diagonal.

O `set`método novamente usa, `super`mas desta vez não para chamar o construtor, mas para chamar um método específico do conjunto de métodos da superclasse. Estamos redefinindo, `set`mas queremos usar o comportamento original. Porque `this.set`se refere ao *novo* `set` método, chamar isso não funcionaria. Dentro dos métodos da classe, `super`fornece uma maneira de chamar métodos como eles foram definidos na superclasse.

A herança nos permite criar tipos de dados ligeiramente diferentes dos existentes, com relativamente pouco trabalho. É uma parte fundamental da tradição orientada a objetos, ao lado de encapsulamento e polimorfismo. Mas enquanto os dois últimos agora são geralmente considerados idéias maravilhosas, a herança é mais controversa.

Enquanto o encapsulamento e o polimorfismo podem ser usados para *separar* partes do código umas das outras, reduzindo o emaranhado de todo o programa, a herança basicamente une as classes, criando *mais* confusão. Ao herdar de uma classe, você geralmente precisa saber mais sobre como ela funciona do que simplesmente usá-la. A herança pode ser uma ferramenta útil, e eu a uso de vez em quando em meus próprios programas, mas não deve ser a primeira ferramenta a ser alcançada e você provavelmente não deve procurar ativamente oportunidades para construir hierarquias de classes (árvores genealógicas) de aulas).

## A instância do operador

Ocasionalmente, é útil saber se um objeto foi derivado de uma classe específica. Para isso, o JavaScript fornece um operador binário chamado `instanceof`.

```js
console.log(
  new SymmetricMatrix(2) instanceof SymmetricMatrix);
// → true
console.log(new SymmetricMatrix(2) instanceof Matrix);
// → true
console.log(new Matrix(2, 2) instanceof SymmetricMatrix);
// → false
console.log([1] instanceof Array);
// → true
```

O operador verá através de tipos herdados, então a `SymmetricMatrix`é uma instância de `Matrix`. O operador também pode ser aplicado a construtores padrão, como `Array`. Quase todo objeto é uma instância de `Object`.

## Sumário

Portanto, os objetos fazem mais do que apenas manter suas próprias propriedades. Eles têm protótipos, que são outros objetos. Eles agirão como se tivessem propriedades que não têm, desde que o protótipo possua essa propriedade. Objetos simples têm `Object.prototype`como protótipo.

Os construtores, que são funções cujos nomes geralmente começam com uma letra maiúscula, podem ser usados com o `new`operador para criar novos objetos. O protótipo do novo objeto será o objeto encontrado na `prototype`propriedade do construtor. Você pode fazer bom uso disso colocando as propriedades que todos os valores de um determinado tipo compartilham em seu protótipo. Existe uma `class`notação que fornece uma maneira clara de definir um construtor e seu protótipo.

Você pode definir getters e setters para chamar secretamente métodos sempre que a propriedade de um objeto for acessada. Métodos estáticos são métodos armazenados no construtor de uma classe, em vez de seu protótipo.

O `instanceof`operador pode, dado um objeto e um construtor, informar se esse objeto é uma instância desse construtor.

Uma coisa útil a fazer com os objetos é especificar uma interface para eles e dizer a todos que eles devem conversar com seu objeto somente através dessa interface. O restante dos detalhes que compõem seu objeto agora estão *encapsulados* , ocultos atrás da interface.

Mais de um tipo pode implementar a mesma interface. O código gravado para usar uma interface sabe automaticamente como trabalhar com qualquer número de objetos diferentes que fornecem a interface. Isso é chamado de *polimorfismo* .

Ao implementar várias classes que diferem apenas em alguns detalhes, pode ser útil escrever as novas classes como *subclasses* de uma classe existente, *herdando* parte de seu comportamento.

## Exercícios

### Um tipo de vetor

Escreva uma classe `Vec`que represente um vetor no espaço bidimensional. Leva `x`e `y`parâmetros (números), que devem ser salvos em propriedades com o mesmo nome.

Dê ao `Vec`protótipo dois métodos, `plus`e `minus`, que tomam outro vetor como parâmetro e retornam um novo vetor que possui a soma ou a diferença dos valores de *x* e *y* dos dois vetores ( `this`e o parâmetro) .

Adicione uma propriedade getter `length`ao protótipo que calcula o comprimento do vetor - ou seja, a distância do ponto ( *x* , *y* ) da origem (0, 0).

```js
// Your code here.

console.log(new Vec(1, 2).plus(new Vec(2, 3)));
// → Vec{x: 3, y: 5}
console.log(new Vec(1, 2).minus(new Vec(2, 3)));
// → Vec{x: -1, y: -1}
console.log(new Vec(3, 4).length);
// → 5
```

Volte ao `Rabbit`exemplo da classe se não tiver certeza de como as `class`declarações são.

A adição de uma propriedade getter ao construtor pode ser feita colocando a palavra `get`antes do nome do método. Para calcular a distância de (0, 0) a (x, y), você pode usar o teorema de Pitágoras, que diz que o quadrado da distância que procuramos é igual ao quadrado da coordenada x mais o quadrado de a coordenada y. Portanto, √ (x 2 + y 2 ) é o número desejado e `Math.sqrt`é a maneira como você calcula uma raiz quadrada em JavaScript.

### Grupos

O ambiente JavaScript padrão fornece outra estrutura de dados chamada `Set`. Como uma instância de `Map`, um conjunto contém uma coleção de valores. Ao contrário `Map`, ele não associa outros valores a esses - apenas rastreia quais valores fazem parte do conjunto. Um valor pode fazer parte de um conjunto apenas uma vez - adicioná-lo novamente não tem efeito.

Escreva uma classe chamada `Group`(uma vez que `Set`já foi tirada). Como `Set`, ele tem `add`, `delete`e `has`métodos. Seu construtor cria um grupo vazio, `add`adiciona um valor ao grupo (mas apenas se ele ainda não é um membro), `delete`remove seu argumento do grupo (se era um membro) e `has`retorna um valor booleano indicando se o argumento é um membro do grupo.

Use o `===`operador, ou algo equivalente como `indexOf`, para determinar se dois valores são iguais.

Dê à classe um `from`método estático que use um objeto iterável como argumento e crie um grupo que contenha todos os valores produzidos pela iteração sobre ele.

```js
class Group {
  // Your code here.
}

let group = Group.from([10, 20]);
console.log(group.has(10));
// → true
console.log(group.has(30));
// → false
group.add(10);
group.delete(10);
console.log(group.has(10));
// → false
```

A maneira mais fácil de fazer isso é armazenar uma matriz de membros do grupo em uma propriedade de instância. Os métodos `includes`ou `indexOf`podem ser usados para verificar se um determinado valor está na matriz.

O construtor da sua classe pode definir a coleção de membros para uma matriz vazia. Quando `add`é chamado, ele deve verificar se o valor fornecido está na matriz ou adicioná-lo, por exemplo `push`, com , caso contrário.

A exclusão de um elemento de uma matriz, em `delete`, é menos direta, mas você pode usar `filter`para criar uma nova matriz sem o valor. Não esqueça de sobrescrever a propriedade que contém os membros com a versão recém-filtrada da matriz.

O `from`método pode usar um loop `for`/ `of`para obter os valores do objeto iterável e chamar `add`para colocá-los em um grupo recém-criado.

### Grupos iteráveis

Torne a `Group`aula do exercício anterior iterável. Consulte a seção sobre a interface do iterador, anteriormente neste capítulo, se você não souber mais a forma exata da interface.

Se você usou uma matriz para representar os membros do grupo, não retorne o iterador criado chamando o `Symbol.iterator`método na matriz. Isso funcionaria, mas derrota o objetivo deste exercício.

Não há problema se o seu iterador se comportar estranhamente quando o grupo for modificado durante a iteração.

```
// Your code here (and the code from the previous exercise)

for (let value of Group.from(["a", "b", "c"])) {
  console.log(value);
}
// → a
// → b
// → c
```

Provavelmente vale a pena definir uma nova classe `GroupIterator`. As instâncias do iterador devem ter uma propriedade que rastreie a posição atual no grupo. Toda vez que `next`é chamado, ele verifica se está pronto e, caso contrário, passa o valor atual e o retorna.

A `Group`própria classe obtém um método nomeado por `Symbol.iterator`isso, quando chamado, retorna uma nova instância da classe iteradora para esse grupo.

### Emprestando um método

No começo do capítulo, mencionei que um objeto `hasOwnProperty`pode ser usado como uma alternativa mais robusta ao `in`operador quando você deseja ignorar as propriedades do protótipo. Mas e se o seu mapa precisar incluir a palavra `"hasOwnProperty"`? Você não poderá mais chamar esse método porque a propriedade do próprio objeto oculta o valor do método.

Você consegue pensar em uma maneira de chamar `hasOwnProperty`um objeto que tenha sua própria propriedade com esse nome?

```js
let map = {one: true, two: true, hasOwnProperty: true};

// Fix this call
console.log(map.hasOwnProperty("one"));
// → true
```

Lembre-se de que os métodos existentes em objetos simples são originários `Object.prototype`.

Lembre-se também de que você pode chamar uma função com uma `this`ligação específica usando seu `call`método