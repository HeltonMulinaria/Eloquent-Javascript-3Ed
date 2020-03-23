# Capítulo 9 Expressões regulares

> Algumas pessoas, quando confrontadas com um problema, pensam 'eu sei, vou usar expressões regulares'. Agora eles tem dois problemas.
>
> *—Jamie Zawinski*

> Yuan-Ma disse: 'Quando você corta contra o grão da madeira, é necessária muita força. Quando você programa contra o problema, é necessário muito código. '
>
> *—Mestre Yuan-Ma, O Livro de Programação*

![Um diagrama ferroviário](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/chapter_picture_9.jpg)

As ferramentas e técnicas de programação sobrevivem e se espalham de maneira caótica e evolutiva. Nem sempre são os bonitos ou brilhantes que vencem, mas os que funcionam bem o suficiente no nicho certo ou que são integrados a outra peça de tecnologia bem-sucedida.

Neste capítulo, discutirei uma dessas ferramentas, *expressões regulares* . Expressões regulares são uma maneira de descrever padrões em dados de string. Eles formam uma linguagem pequena e separada que faz parte do JavaScript e de muitas outras linguagens e sistemas.

Expressões regulares são terrivelmente estranhas e extremamente úteis. Sua sintaxe é enigmática, e a interface de programação que o JavaScript fornece para eles é desajeitada. Mas eles são uma ferramenta poderosa para inspecionar e processar seqüências de caracteres. O entendimento adequado das expressões regulares fará de você um programador mais eficaz.

## Criando uma expressão regular

Uma expressão regular é um tipo de objeto. Ele pode ser construído com o `RegExp`construtor ou gravado como um valor literal colocando um padrão em `/`caracteres de barra ( ).

```js
let re1 = new RegExp("abc");
let re2 = /abc/;
```

Ambos os objetos de expressão regular representam o mesmo padrão: *um* caractere a seguido de *b* seguido de *c* .

Ao usar o `RegExp`construtor, o padrão é gravado como uma sequência normal, portanto, as regras usuais se aplicam a barras invertidas.

A segunda notação, em que o padrão aparece entre caracteres de barra, trata as barras invertidas de maneira um pouco diferente. Primeiro, como uma barra finaliza o padrão, precisamos colocar uma barra invertida antes de qualquer barra que desejamos fazer *parte* do padrão. Além disso, as barras invertidas que não fazem parte dos códigos de caracteres especiais (como `\n`) serão *preservadas* , em vez de ignoradas como estão nas cadeias, e alteram o significado do padrão. Alguns caracteres, como pontos de interrogação e sinais de adição, têm significados especiais em expressões regulares e devem ser precedidos por uma barra invertida se pretendem representar o próprio personagem.

```JS
let eighteenPlus = /eighteen\+/;
```

## Testando partidas

Os objetos de expressão regular têm vários métodos. O mais simples é `test`. Se você passar uma string, ele retornará um booleano informando se a string contém uma correspondência do padrão na expressão.

```JS
console.log(/abc/.test("abcde"));
// → true
console.log(/abc/.test("abxde"));
// → false
```

Uma expressão regular que consiste apenas em caracteres não especiais simplesmente representa essa sequência de caracteres. Se *abc* ocorrer em qualquer lugar da string em que estamos testando (não apenas no início), `test`retornará `true`.

## Conjuntos de caracteres

Descobrir se uma string contém *abc também* pode ser feito com uma chamada para `indexOf`. Expressões regulares nos permitem expressar padrões mais complicados.

Digamos que queremos corresponder a qualquer número. Em uma expressão regular, colocar um conjunto de caracteres entre colchetes faz com que essa parte da expressão corresponda a qualquer um dos caracteres entre colchetes.

As duas expressões a seguir correspondem a todas as strings que contêm um dígito:

```js
console.log(/[0123456789]/.test("in 1992"));
// → true
console.log(/[0-9]/.test("in 1992"));
// → true
```

`-`Entre colchetes, um hífen ( ) entre dois caracteres pode ser usado para indicar um intervalo de caracteres, em que a ordem é determinada pelo número Unicode do caractere. Os caracteres de 0 a 9 ficam ao lado um do outro nesta ordem (códigos 48 a 57), portanto, `[0-9]`abrange todos eles e corresponde a qualquer dígito.

Vários grupos de caracteres comuns têm seus próprios atalhos internos. Os dígitos são um deles: `\d`significa a mesma coisa que `[0-9]`.

| `\d` | Qualquer caractere de dígito                                 |
| ---- | ------------------------------------------------------------ |
| `\w` | Um caractere alfanumérico ("caractere de palavra")           |
| `\s` | Qualquer caractere de espaço em branco (espaço, guia, nova linha e semelhante) |
| `\D` | Um caractere que *não* é um dígito                           |
| `\W` | Um caractere não alfanumérico                                |
| `\S` | Um caractere não-espaço em branco                            |
| `.`  | Qualquer caractere, exceto nova linha                        |

Assim, você pode combinar um formato de data e hora como 01-30-2003 15:20 com a seguinte expressão:

```js
let dateTime = /\d\d-\d\d-\d\d\d\d \d\d:\d\d/;
console.log(dateTime.test("01-30-2003 15:20"));
// → true
console.log(dateTime.test("30-jan-2003 15:20"));
// → false
```

Parece completamente horrível, não é? Metade é de barras invertidas, produzindo um ruído de fundo que dificulta a identificação do padrão real expresso. Veremos uma versão ligeiramente melhorada dessa expressão [posteriormente](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2009%20-%20Express%C3%B5es%20regulares.md#date_regexp_counted) .

Esses códigos de barra invertida também podem ser usados dentro de colchetes. Por exemplo, `[\d.]`significa qualquer dígito ou caractere de ponto. Mas o próprio período, entre colchetes, perde seu significado especial. O mesmo vale para outros caracteres especiais, como `+`.

Para *inverter* um conjunto de caracteres - ou seja, para expressar que deseja corresponder a qualquer caractere, *exceto* os do conjunto -, você pode escrever um `^`caractere de sinal de intercalação ( ) após o colchete de abertura.

```js
let notBinary = /[^01]/;
console.log(notBinary.test("1100100010100110"));
// → false
console.log(notBinary.test("1100100010200110"));
// → true
```

## Repetindo partes de um padrão

Agora sabemos como combinar um único dígito. E se quisermos combinar um número inteiro - uma sequência de um ou mais dígitos?

Quando você coloca um sinal de adição ( `+`) após algo em uma expressão regular, isso indica que o elemento pode ser repetido mais de uma vez. Assim, `/\d+/`corresponde a um ou mais caracteres de dígito.

```js
console.log(/'\d+'/.test("'123'"));
// → true
console.log(/'\d+'/.test("''"));
// → false
console.log(/'\d*'/.test("'123'"));
// → true
console.log(/'\d*'/.test("''"));
// → true
```

A estrela ( `*`) tem um significado semelhante, mas também permite que o padrão corresponda a zero tempo. Algo com uma estrela depois que nunca impede a correspondência de um padrão - ele corresponderá a zero instâncias se não encontrar nenhum texto adequado para corresponder.

Um ponto de interrogação torna *opcional* uma parte de um padrão , o que significa que pode ocorrer zero vezes ou uma vez. No exemplo a seguir, é permitido que o caractere *u* ocorra, mas o padrão também corresponde quando está ausente.

```js
let neighbor = /neighbou?r/;
console.log(neighbor.test("neighbour"));
// → true
console.log(neighbor.test("neighbor"));
// → true
```

Para indicar que um padrão deve ocorrer um número preciso de vezes, use chaves. Colocar `{4}`um elemento, por exemplo, exige que ele ocorra exatamente quatro vezes. Também é possível especificar um intervalo desta maneira: `{2,4}`significa que o elemento deve ocorrer pelo menos duas e no máximo quatro vezes.

Aqui está outra versão do padrão de data e hora que permite dias, meses e horas com um e dois dígitos. Também é um pouco mais fácil de decifrar.

```js
let dateTime = /\d{1,2}-\d{1,2}-\d{4} \d{1,2}:\d{2}/;
console.log(dateTime.test("1-30-2003 8:45"));
// → true
```

Você também pode especificar intervalos abertos ao usar chaves, omitindo o número após a vírgula. Então, `{5,}`significa cinco ou mais vezes.

## Agrupando subexpressões

Para usar um operador como `*`ou `+`em mais de um elemento por vez, é necessário usar parênteses. Uma parte de uma expressão regular entre parênteses conta como um único elemento no que diz respeito aos operadores que a seguem.

```js
let cartoonCrying = /boo+(hoo+)+/i;
console.log(cartoonCrying.test("Boohoooohoohooo"));
// → true
```

O primeiro e o segundo `+`caracteres se aplicam apenas ao segundo *o* no *boo* e *hoo* , respectivamente. O terceiro `+`se aplica a todo o grupo `(hoo+)`, correspondendo a uma ou mais seqüências como essa.

O `i`no final da expressão no exemplo torna essa expressão regular sem distinção entre maiúsculas e minúsculas, permitindo que ela corresponda ao *B* maiúsculo na sequência de entrada, mesmo que o padrão seja todo em minúscula.

## Jogos e grupos

O `test`método é a maneira absolutamente mais simples de corresponder a uma expressão regular. Ele informa apenas se foi compatível e nada mais. Expressões regulares também têm um `exec`método (execute) que retornará `null`se nenhuma correspondência for encontrada e retornará um objeto com informações sobre a correspondência.

```js
let match = /\d+/.exec("one two 100");
console.log(match);
// → ["100"]
console.log(match.index);
// → 8
```

Um objeto retornado `exec`possui uma `index`propriedade que nos diz *onde* na sequência a correspondência bem-sucedida começa. Fora isso, o objeto se parece (e de fato é) uma matriz de cadeias, cujo primeiro elemento é a cadeia que foi correspondida. No exemplo anterior, esta é a sequência de dígitos que estávamos procurando.

Os valores de sequência têm um `match`método que se comporta de maneira semelhante.

```js
console.log("one two 100".match(/\d+/));
// → ["100"]
```

Quando a expressão regular contém subexpressões agrupadas entre parênteses, o texto que corresponde a esses grupos também será exibido na matriz. A partida inteira é sempre o primeiro elemento. O próximo elemento é a parte correspondida pelo primeiro grupo (aquele cujo parêntese de abertura vem primeiro na expressão), depois o segundo grupo e assim por diante.

```js
let quotedText = /'([^']*)'/;
console.log(quotedText.exec("she said 'hello'"));
// → ["'hello'", "hello"]
```

Quando um grupo não acaba sendo correspondido (por exemplo, quando seguido por um ponto de interrogação), sua posição na matriz de saída é mantida `undefined`. Da mesma forma, quando um grupo é correspondido várias vezes, apenas a última correspondência termina na matriz.

```js
console.log(/bad(ly)?/.exec("bad"));
// → ["bad", undefined]
console.log(/(\d)+/.exec("123"));
// → ["123", "3"]
```

Grupos podem ser úteis para extrair partes de uma sequência. Se não queremos apenas verificar se uma string contém uma data, mas também extraí-la e construir um objeto que a represente, podemos colocar parênteses em torno dos padrões de dígitos e escolher diretamente a data do resultado de `exec`.

Mas primeiro faremos um breve desvio, no qual discutiremos a maneira integrada de representar valores de data e hora em JavaScript.

## A classe Date

JavaScript tem uma classe padrão para representar datas - ou melhor, pontos no tempo. É chamado `Date`. Se você simplesmente criar um objeto de data usando `new`, obterá a data e a hora atuais.

```js
console.log(new Date());
// → Mon Nov 13 2017 16:19:11 GMT+0100 (CET)
```

Você também pode criar um objeto para um horário específico.

```js
console.log(new Date(2009, 11, 9));
// → Wed Dec 09 2009 00:00:00 GMT+0100 (CET)
console.log(new Date(2009, 11, 9, 12, 59, 59, 999));
// → Wed Dec 09 2009 12:59:59 GMT+0100 (CET)
```

O JavaScript usa uma convenção em que os números dos meses começam em zero (portanto, 11 de dezembro), mas os números dos dias começam em um. Isso é confuso e bobo. Seja cuidadoso.

Os últimos quatro argumentos (horas, minutos, segundos e milissegundos) são opcionais e considerados zero quando não fornecidos.

Os carimbos de data e hora são armazenados como o número de milissegundos desde o início de 1970, no fuso horário UTC. Isso segue uma convenção definida pelo "tempo Unix", que foi inventada nessa época. Você pode usar números negativos para horários anteriores a 1970. O `getTime`método em um objeto de data retorna esse número. É grande, como você pode imaginar.

```js
console.log(new Date(2013, 11, 19).getTime());
// → 1387407600000
console.log(new Date(1387407600000));
// → Thu Dec 19 2013 00:00:00 GMT+0100 (CET)
```

Se você der ao `Date`construtor um único argumento, esse argumento será tratado como uma contagem de milissegundos. Você pode obter a contagem atual de milissegundos criando um novo `Date`objeto e chamando `getTime`-o ou chamando a `Date.now`função.

Objetos Date fornecer métodos tais como `getFullYear`, `getMonth`, `getDate`, `getHours`, `getMinutes`, e `getSeconds`para extrair seus componentes. Além disso, `getFullYear`há também o `getYear`que dá o ano menos 1900 ( `98`ou `119`) e é praticamente inútil.

Colocando parênteses em torno das partes da expressão em que estamos interessados, agora podemos criar um objeto de data a partir de uma string.

```js
function getDate(string) {
  let [_, month, day, year] =
    /(\d{1,2})-(\d{1,2})-(\d{4})/.exec(string);
  return new Date(year, month - 1, day);
}
console.log(getDate("1-30-2003"));
// → Thu Jan 30 2003 00:00:00 GMT+0100 (CET)
```

A `_`ligação (sublinhado) é ignorada e usada apenas para ignorar o elemento de correspondência completa na matriz retornada por `exec`.

## Limites de palavras e strings

Infelizmente, `getDate`também será útil extrair a data sem sentido 00-1-3000 da string `"100-1-30000"`. Uma correspondência pode acontecer em qualquer lugar da string, portanto, neste caso, ela começará no segundo caractere e terminará no penúltimo caractere.

Se queremos reforçar que a correspondência deve abranger toda a cadeia, podemos adicionar os marcadores `^`e `$`. O sinal de intercalação corresponde ao início da sequência de entrada, enquanto o cifrão corresponde ao final. Portanto, `/^\d+$/`corresponde a uma string que consiste inteiramente de um ou mais dígitos, a `/^!/`qualquer string que comece com um ponto de exclamação e `/x^/`não corresponde a nenhuma string (não pode haver um *x* antes do início da string).

Se, por outro lado, queremos apenas garantir que a data comece e termine no limite de uma palavra, podemos usar o marcador `\b`. Um limite de palavra pode ser o início ou o fim da sequência ou qualquer ponto da sequência que tenha um caractere de palavra (como em `\w`) de um lado e um caractere não-palavra do outro.

```js
console.log(/cat/.test("concatenate"));
// → true
console.log(/\bcat\b/.test("concatenate"));
// → false
```

Observe que um marcador de limite não corresponde a um caractere real. Apenas reforça que a expressão regular corresponde apenas quando uma determinada condição é mantida no local em que aparece no padrão.

## Padrões de escolha

Digamos que queremos saber se um pedaço de texto contém não apenas um número, mas um número seguido por uma das palavras *porco* , *vaca* ou *galinha* ou qualquer uma de suas formas plurais.

Poderíamos escrever três expressões regulares e testá-las, mas há uma maneira melhor. O caractere de barra vertical ( `|`) indica uma escolha entre o padrão à esquerda e o padrão à direita. Então eu posso dizer o seguinte:

```js
let animalCount = /\b\d+ (pig|cow|chicken)s?\b/;
console.log(animalCount.test("15 pigs"));
// → true
console.log(animalCount.test("15 pigchickens"));
// → false
```

Os parênteses podem ser usados para limitar a parte do padrão ao qual o operador de tubulação se aplica e você pode colocar vários operadores próximos um do outro para expressar uma escolha entre mais de duas alternativas.

## A mecânica da correspondência

Conceitualmente, quando você usa `exec`ou `test`, o mecanismo de expressão regular procura uma correspondência em sua string tentando corresponder à expressão primeiro desde o início da string, depois a partir do segundo caractere, e assim por diante, até encontrar uma correspondência ou atingir o valor. fim da cadeia. Ele retornará a primeira correspondência encontrada ou não encontrará nenhuma correspondência.

Para fazer a correspondência real, o mecanismo trata uma expressão regular como um diagrama de fluxo. Este é o diagrama para a expressão de gado no exemplo anterior:

![Visualização de / \ b \ d + (porco | vaca | frango) s? \ B /](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/re_pigchickens.svg)

Nossa expressão corresponde se pudermos encontrar um caminho do lado esquerdo do diagrama para o lado direito. Mantemos uma posição atual na string e toda vez que movemos uma caixa, verificamos se a parte da string após nossa posição atual corresponde a essa caixa.

Portanto, se tentarmos corresponder `"the 3 pigs"`da posição 4, nosso progresso no fluxograma ficaria assim:

- Na posição 4, há um limite de palavras, para que possamos passar da primeira caixa.
- Ainda na posição 4, encontramos um dígito, para que também possamos passar da segunda caixa.
- Na posição 5, um caminho volta para antes da segunda caixa (dígito), enquanto o outro avança pela caixa que contém um único caractere de espaço. Há um espaço aqui, não um dígito, então devemos seguir o segundo caminho.
- Agora estamos na posição 6 (o início dos *porcos* ) e no ramo de três vias no diagrama. Não vemos *vaca* ou *galinha* aqui, mas vemos *porco* , então pegamos esse galho.
- Na posição 9, após o ramo de três vias, um caminho ignora a caixa *s* e vai direto para o limite final da palavra, enquanto o outro caminho corresponde a um *s* . Há um caractere *s* aqui, não um limite de palavra, então passamos pela caixa *s* .
- Estamos na posição 10 (o final da string) e podemos corresponder apenas a um limite de palavras. O final de uma sequência conta como um limite de palavra, portanto, percorremos a última caixa e correspondemos a essa sequência com êxito.

## Backtracking

A expressão regular corresponde a um número binário seguido de a *b* , um número hexadecimal (ou seja, base 16, com as letras de *a* a *f* representando os dígitos de 10 a 15) seguido de um *h* , ou um número decimal regular sem sufixo personagem. Este é o diagrama correspondente:`/\b([01]+b|[\da-f]+h|\d+)\b/`

![Visualização de / \ b ([01] + b | \ d + | [\ da-f] + h) \ b /](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/re_number.svg)

Ao corresponder a essa expressão, muitas vezes acontece que a ramificação superior (binária) é inserida, mesmo que a entrada realmente não contenha um número binário. Ao combinar a string `"103"`, por exemplo, fica claro apenas nos 3 que estamos no ramo errado. A string *não* corresponder à expressão, não apenas o ramo que estão atualmente em.

Então o jogador *recua* . Ao entrar em um ramo, ele se lembra de sua posição atual (neste caso, no início da cadeia, logo após a primeira caixa de limite do diagrama), para que ele possa voltar e tentar outro ramo, se o atual não funcionar. . Para a sequência `"103"`, após encontrar o caractere 3, ela começará a tentar a ramificação para números hexadecimais, que falham novamente porque não há *h* após o número. Por isso, tenta o número decimal ramo. Este se encaixa, e uma partida é relatada, afinal.

O jogador para assim que encontra uma partida completa. Isso significa que, se várias ramificações puderem corresponder a uma sequência, apenas a primeira (ordenada por onde as ramificações aparecem na expressão regular) será usada.

O retorno também acontece para operadores de repetição como + e `*`. Se você combinar `/^.*x/`contra `"abcxe"`, a `.*`parte vai primeiro tentar consumir toda a string. O mecanismo perceberá que precisa de um *x* para corresponder ao padrão. Como não há *x* após o final da sequência, o operador em estrela tenta corresponder a um caractere a menos. Mas a correspondência não encontrar um *x* depois de `abcx`qualquer um, por isso recua novamente, combinando o operador estrela para apenas `abc`. *Agora* ele encontra *x* onde precisa e relata uma correspondência bem-sucedida das posições 0 a 4.

É possível escrever expressões regulares que farão *muito* retorno. Esse problema ocorre quando um padrão pode corresponder a uma parte da entrada de várias maneiras diferentes. Por exemplo, se ficarmos confusos ao escrever uma expressão regular de número binário, poderemos escrever acidentalmente algo como `/([01]+)+b/`.

![Visualização de / ([01] +) + b /](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/re_slow.svg)

Se isso tentar corresponder a uma série longa de zeros e outros sem caractere *b* à direita , o correspondente primeiro percorrerá o loop interno até ficar sem dígitos. Em seguida, percebe que não há *b* , então recua uma posição, percorre o loop externo uma vez e desiste novamente, tentando voltar ao loop interno mais uma vez. Ele continuará tentando todas as rotas possíveis através desses dois loops. Isso significa que a quantidade de trabalho *dobra* com cada caractere adicional. Para apenas algumas dezenas de caracteres, a correspondência resultante levará praticamente uma eternidade.

## O método de substituição

Os valores `replace`de sequência têm um método que pode ser usado para substituir parte da sequência por outra.

```js
console.log("papa".replace("p", "m"));
// → mapa
```

O primeiro argumento também pode ser uma expressão regular. Nesse caso, a primeira correspondência da expressão regular é substituída. Quando uma `g`opção (para *global* ) é adicionada à expressão regular, *todas as* correspondências na sequência serão substituídas, não apenas a primeira.

```js
console.log("Borobudur".replace(/[ou]/, "a"));
// → Barobudur
console.log("Borobudur".replace(/[ou]/g, "a"));
// → Barabadar
```

Teria sido sensato se a escolha entre substituir uma correspondência ou todas as correspondências fosse feita através de um argumento adicional `replace`ou fornecendo um método diferente `replaceAll`. Mas, por algum motivo infeliz, a escolha depende de uma propriedade da expressão regular.

O poder real de usar expressões regulares `replace`vem do fato de podermos nos referir a grupos correspondentes na string de substituição. Por exemplo, digamos que tenhamos uma grande sequência contendo os nomes das pessoas, um nome por linha, no formato `Lastname, Firstname`. Se quisermos trocar esses nomes e remover a vírgula para obter um `Firstname Lastname`formato, podemos usar o seguinte código:

```js
console.log(
  "Liskov, Barbara\nMcCarthy, John\nWadler, Philip"
    .replace(/(\w+), (\w+)/g, "$2 $1"));
// → Barbara Liskov
//   John McCarthy
//   Philip Wadler
```

O `$1`e `$2`na cadeia de substituição se referem aos grupos entre parênteses no padrão. `$1`é substituído pelo texto que corresponde ao primeiro grupo, `$2`ao segundo e assim por diante, até `$9`. A partida inteira pode ser referida com `$&`.

É possível passar uma função - em vez de uma string - como o segundo argumento para `replace`. Para cada substituição, a função será chamada com os grupos correspondentes (assim como toda a correspondência) como argumentos, e seu valor de retorno será inserido na nova sequência.

Aqui está um pequeno exemplo:

```js
let s = "the cia and fbi";
console.log(s.replace(/\b(fbi|cia)\b/g,
            str => str.toUpperCase()));
// → the CIA and FBI
```

Aqui está um mais interessante:

```js
let stock = "1 lemon, 2 cabbages, and 101 eggs";
function minusOne(match, amount, unit) {
  amount = Number(amount) - 1;
  if (amount == 1) { // only one left, remove the 's'
    unit = unit.slice(0, unit.length - 1);
  } else if (amount == 0) {
    amount = "no";
  }
  return amount + " " + unit;
}
console.log(stock.replace(/(\d+) (\w+)/g, minusOne));
// → no lemon, 1 cabbage, and 100 eggs
```

Isso pega uma string, localiza todas as ocorrências de um número seguido por uma palavra alfanumérica e retorna uma string em que cada ocorrência é decrementada por uma.

O `(\d+)`grupo termina como `amount`argumento para a função e o `(\w+)`grupo fica vinculado `unit`. A função é convertida `amount`em um número - que sempre funciona desde que corresponde `\d+`- e faz alguns ajustes no caso de restar apenas um ou zero.

## Greed

É possível usar `replace`para escrever uma função que remove todos os comentários de uma parte do código JavaScript. Aqui está uma primeira tentativa:

```js
function stripComments(code) {
  return code.replace(/\/\/.*|\/\*[^]*\*\//g, "");
}
console.log(stripComments("1 + /* 2 */3"));
// → 1 + 3
console.log(stripComments("x = 10;// ten!"));
// → x = 10;
console.log(stripComments("1 /* a */+/* b */ 1"));
// → 1  1
```

A parte antes do operador *ou* corresponde a dois caracteres de barra seguidos por qualquer número de caracteres que não sejam de nova linha. A parte dos comentários multilinhas está mais envolvida. Usamos `[^]`(qualquer caractere que não esteja no conjunto vazio de caracteres) como uma maneira de corresponder a qualquer caractere. Não podemos usar apenas um ponto aqui porque os comentários do bloco podem continuar em uma nova linha, e o caractere do ponto não corresponde aos caracteres da nova linha.

Mas a saída para a última linha parece ter dado errado. Por quê?

A `[^]*`parte da expressão, como descrevi na seção sobre retorno, primeiro corresponderá o máximo possível. Se isso fizer com que a próxima parte do padrão falhe, o jogador retrocede um caractere e tenta novamente a partir daí. No exemplo, o correspondente tenta primeiro corresponder ao resto da string e depois volta a partir daí. Ele encontrará uma ocorrência de `*/`depois de retornar quatro caracteres e corresponder a isso. Não era isso que queríamos - a intenção era corresponder a um único comentário, não ir até o final do código e encontrar o final do último comentário do bloco.

Devido a esse comportamento, dizemos os operadores de repetição ( `+`, `*`, `?`e `{}`) são *gananciosos* , o que significa que corresponder tanto quanto eles podem e backtrack de lá. Se você colocar um ponto de interrogação depois deles ( `+?`, `*?`, `??`, `{}?`), tornam-se não greedy e começar por correspondência tão pouco quanto possível, combinando mais somente quando o padrão restante não se encaixa no jogo menor.

E é exatamente isso que queremos neste caso. Ao fazer com que a estrela corresponda ao menor número de caracteres que nos leva a `*/`, consumimos um comentário em bloco e nada mais.

```js
function stripComments(code) {
  return code.replace(/\/\/.*|\/\*[^]*?\*\//g, "");
}
console.log(stripComments("1 /* a */+/* b */ 1"));
// → 1 + 1
```

Muitos bugs em programas de expressão regular podem ser atribuídos sem querer a um operador ganancioso, no qual um não-remediado funcionaria melhor. Ao usar um operador de repetição, considere a variante não remediada primeiro.

## Criando dinamicamente objetos RegExp

Há casos em que você pode não saber o padrão exato com o qual precisa corresponder ao escrever seu código. Digamos que você queira procurar o nome do usuário em um pedaço de texto e coloque-o em caracteres sublinhados para destacá-lo. Como você saberá o nome apenas quando o programa estiver em execução, não será possível usar a notação baseada em barra.

Mas você pode criar uma string e usar o `RegExp`construtor nisso. Aqui está um exemplo:

```js
let name = "harry";
let text = "Harry is a suspicious character.";
let regexp = new RegExp("\\b(" + name + ")\\b", "gi");
console.log(text.replace(regexp, "_$1_"));
// → _Harry_ is a suspicious character.
```

Ao criar os `\b`marcadores de limite, precisamos usar duas barras invertidas, porque as escrevemos em uma sequência normal, não em uma expressão regular fechada. O segundo argumento para o `RegExp`construtor contém as opções para a expressão regular - nesse caso, `"gi"`global e sem distinção entre maiúsculas e minúsculas.

Mas e se o nome for `"dea+hl[]rd"`porque nosso usuário é um adolescente nerd? Isso resultaria em uma expressão regular sem sentido que não corresponderá realmente ao nome do usuário.

Para contornar isso, podemos adicionar barras invertidas antes de qualquer caractere que tenha um significado especial.

```js
let name = "dea+hl[]rd";
let text = "This dea+hl[]rd guy is super annoying.";
let escaped = name.replace(/[\\[.+*?(){|^$]/g, "\\$&");
let regexp = new RegExp("\\b" + escaped + "\\b", "gi");
console.log(text.replace(regexp, "_$&_"));
// → This _dea+hl[]rd_ guy is super annoying.
```

## O método de pesquisa

O `indexOf`método em strings não pode ser chamado com uma expressão regular. Mas há outro método,, `search`que espera uma expressão regular. Como `indexOf`, retorna o primeiro índice no qual a expressão foi encontrada ou -1 quando não foi encontrada.

```js
console.log("  word".search(/\S/));
// → 2
console.log("    ".search(/\S/));
// → -1
```

Infelizmente, não há como indicar que a partida deve começar em um determinado deslocamento (como podemos com o segundo argumento `indexOf`), o que geralmente seria útil.

## A propriedade lastIndex

Da `exec`mesma forma, o método não fornece uma maneira conveniente de iniciar a pesquisa a partir de uma determinada posição na string. Mas fornece uma *em* forma conveniente.

Objetos de expressão regular têm propriedades. Uma dessas propriedades é `source`, que contém a string da qual a expressão foi criada. Outra propriedade é `lastIndex`, que controla, em algumas circunstâncias limitadas, onde a próxima partida começará.

Essas circunstâncias são que a expressão regular deve ter a opção global ( `g`) ou pegajosa ( `y`) ativada e a correspondência deve ocorrer através do `exec`método Novamente, uma solução menos confusa teria sido permitir que um argumento extra fosse passado `exec`, mas a confusão é um recurso essencial da interface de expressão regular do JavaScript.

```js
let pattern = /y/g;
pattern.lastIndex = 3;
let match = pattern.exec("xyzzy");
console.log(match.index);
// → 4
console.log(pattern.lastIndex);
// → 5
```

Se a partida foi bem-sucedida, a chamada para `exec`atualizar automaticamente a `lastIndex`propriedade para apontar após a partida. Se nenhuma correspondência foi encontrada, `lastIndex`é definido como zero, que também é o valor que ela possui em um objeto de expressão regular recém-construído.

A diferença entre as opções global e aderente é que, quando ativada, a correspondência será bem-sucedida apenas se for iniciada diretamente em `lastIndex`, enquanto que com global, ela buscará uma posição em que uma correspondência possa começar.

```js
let global = /abc/g;
console.log(global.exec("xyz abc"));
// → ["abc"]
let sticky = /abc/y;
console.log(sticky.exec("xyz abc"));
// → null
```

Ao usar um valor de expressão regular compartilhada para várias `exec`chamadas, essas atualizações automáticas da `lastIndex`propriedade podem causar problemas. Sua expressão regular pode estar acidentalmente iniciando em um índice que sobrou de uma chamada anterior.

```js
let digit = /\d/g;
console.log(digit.exec("here it is: 1"));
// → ["1"]
console.log(digit.exec("and now: 1"));
// → null
```

Outro efeito interessante da opção global é que ela altera a maneira como o `match`método nas strings funciona. Quando chamado com uma expressão global, em vez de retornar uma matriz semelhante à retornada por `exec`, `match`encontrará *todas as* correspondências do padrão na string e retornará uma matriz contendo as strings correspondentes.

```js
console.log("Banana".match(/an/g));
// → ["an", "an"]
```

Portanto, seja cauteloso com expressões regulares globais. Os casos em que são necessários - chamadas `replace`e locais onde você deseja usar explicitamente `lastIndex`- geralmente são os únicos lugares em que você deseja usá-los.

### Looping sobre correspondências

Uma coisa comum a se fazer é varrer todas as ocorrências de um padrão em uma string, de uma maneira que nos permita acessar o objeto de correspondência no corpo do loop. Podemos fazer isso usando `lastIndex`e `exec`.

```js
let input = "A string with 3 numbers in it... 42 and 88.";
let number = /\b\d+\b/g;
let match;
while (match = number.exec(input)) {
  console.log("Found", match[0], "at", match.index);
}
// → Found 3 at 14
//   Found 42 at 33
//   Found 88 at 40
```

Isso faz uso do fato de que o valor de uma expressão de atribuição ( `=`) é o valor atribuído. Portanto, usando como condição na instrução, executamos a correspondência no início de cada iteração, salvamos o resultado em uma ligação e paramos de repetir quando não há mais correspondências.`match = number.exec(input)``while`

## Analisando um arquivo INI

Para concluir o capítulo, examinaremos um problema que exige expressões regulares. Imagine que estamos escrevendo um programa para coletar automaticamente informações sobre nossos inimigos da Internet. (Na verdade, não escreveremos esse programa aqui, apenas a parte que lê o arquivo de configuração. Desculpe.) O arquivo de configuração fica assim:

```js
searchengine=https://duckduckgo.com/?q=$1
spitefulness=9.7

; comments are preceded by a semicolon...
; each section concerns an individual enemy
[larry]
fullname=Larry Doe
type=kindergarten bully
website=http://www.geocities.com/CapeCanaveral/11451

[davaeorn]
fullname=Davaeorn
type=evil wizard
outputdir=/home/marijn/enemies/davaeorn
```

As regras exatas para esse formato (que é um formato amplamente usado, geralmente chamado de arquivo *INI* ) são as seguintes:

- Linhas em branco e linhas iniciando com ponto e vírgula são ignoradas.
- Linhas agrupadas `[`e `]`iniciam uma nova seção.
- Linhas contendo um identificador alfanumérico seguido por um `=`caractere adicionam uma configuração à seção atual.
- Qualquer outra coisa é inválida.

Nossa tarefa é converter uma string como esta em um objeto cujas propriedades contêm strings para configurações escritas antes do primeiro cabeçalho da seção e subobjetos para seções, com esses subobjetos mantendo as configurações da seção.

Como o formato precisa ser processado linha por linha, dividir o arquivo em linhas separadas é um bom começo. Vimos o `split`método no [capítulo 4](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2004%20-%20Estruturas%20de%20dados%20objetos%20e%20matrizes.md#split) . Alguns sistemas operacionais, no entanto, usam não apenas um caractere de nova linha para separar linhas, mas um caractere de retorno de carro seguido por uma nova linha ( `"\r\n"`). Dado que o `split`método também permite uma expressão regular como argumento, podemos usar uma expressão regular como `/\r?\n/`dividir de uma maneira que permita ambas `"\n"`e `"\r\n"`entre linhas.

```js
function parseINI(string) {
  // Start with an object to hold the top-level fields
  let result = {};
  let section = result;
  string.split(/\r?\n/).forEach(line => {
    let match;
    if (match = line.match(/^(\w+)=(.*)$/)) {
      section[match[1]] = match[2];
    } else if (match = line.match(/^\[(.*)\]$/)) {
      section = result[match[1]] = {};
    } else if (!/^\s*(;.*)?$/.test(line)) {
      throw new Error("Line '" + line + "' is not valid.");
    }
  });
  return result;
}

console.log(parseINI(`
name=Vasilis
[address]
city=Tessaloniki`));
// → {name: "Vasilis", address: {city: "Tessaloniki"}}
```

O código passa pelas linhas do arquivo e cria um objeto. As propriedades na parte superior são armazenadas diretamente nesse objeto, enquanto as propriedades encontradas nas seções são armazenadas em um objeto de seção separado. Os `section`pontos de ligação no objeto para a seção atual.

Existem dois tipos de linhas significativas - cabeçalhos de seção ou linhas de propriedade. Quando uma linha é uma propriedade regular, ela é armazenada na seção atual. Quando é um cabeçalho de seção, um novo objeto de seção é criado e `section`definido para apontar para ele.

Observe o uso recorrente de `^`e `$`para garantir que a expressão corresponda à linha inteira, não apenas a parte dela. Deixar esses resultados de fora em código que funciona principalmente, mas se comporta de maneira estranha para alguma entrada, o que pode ser um bug difícil de rastrear.

O padrão é semelhante ao truque de usar uma atribuição como condição para . Geralmente, você não tem certeza de que sua chamada será bem-sucedida; portanto, você pode acessar o objeto resultante apenas dentro de uma instrução que testa isso. Para não quebrar a agradável cadeia de formulários, atribuímos o resultado da correspondência a uma ligação e usamos imediatamente essa atribuição como teste para a declaração.`if (match = string.match(...))``while``match``if``else if``if`

Se uma linha não é um cabeçalho de seção ou uma propriedade, a função verifica se é um comentário ou uma linha vazia usando a expressão `/^\s*(;.*)?$/`. Você vê como isso funciona? A parte entre parênteses corresponderá a comentários e `?`garante que também corresponda a linhas contendo apenas espaço em branco. Quando uma linha não corresponde a nenhum dos formulários esperados, a função lança uma exceção.

## Personagens internacionais

Devido à implementação simplista inicial do JavaScript e ao fato de que essa abordagem simplista foi posteriormente definida como comportamento padrão, as expressões regulares do JavaScript são bastante estúpidas quanto a caracteres que não aparecem no idioma inglês. Por exemplo, no que diz respeito às expressões regulares do JavaScript, um "caractere de palavra" é apenas um dos 26 caracteres do alfabeto latino (maiúsculas ou minúsculas), dígitos decimais e, por algum motivo, o caractere sublinhado. Coisas como *E* ou *β* , que definitivamente são caracteres de texto, não irá corresponder `\w`(e *vai* coincidir com maiúscula `\W`, a categoria diferente de palavra).

Por um estranho acidente histórico, `\s`(espaço em branco) não apresenta esse problema e corresponde a todos os caracteres que o padrão Unicode considera espaço em branco, incluindo itens como o espaço não separável e o separador de vogal mongol.

Outro problema é que, por padrão, expressões regulares funcionam em unidades de código, conforme discutido no [Capítulo 5](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2005%20-%20Fun%C3%A7%C3%B5es%20de%20ordem%20superior.md#code_units) , não em caracteres reais. Isso significa que os caracteres compostos por duas unidades de código se comportam de maneira estranha.

```js
console.log(/🍎{3}/.test("🍎🍎🍎"));
// → false
console.log(/<.>/.test("<🌹>"));
// → false
console.log(/<.>/u.test("<🌹>"));
// → true
```

O problema é que o 🍎 na primeira linha é tratado como duas unidades de código e a `{3}`parte é aplicada apenas à segunda. Da mesma forma, o ponto corresponde a uma única unidade de código, não às duas que compõem o emoji rosa.

Você deve adicionar uma `u`opção (para Unicode) à sua expressão regular para fazer com que ele trate esses caracteres corretamente. O comportamento errado continua sendo o padrão, infelizmente, porque alterar isso pode causar problemas no código existente que depende dele.

Embora isso tenha sido apenas padronizado e ainda não seja amplamente suportado no momento da redação, é possível usar `\p`em uma expressão regular (que deve ter a opção Unicode ativada) para corresponder a todos os caracteres aos quais o padrão Unicode atribui um determinado propriedade.

```js
console.log(/\p{Script=Greek}/u.test("α"));
// → true
console.log(/\p{Script=Arabic}/u.test("α"));
// → false
console.log(/\p{Alphabetic}/u.test("α"));
// → true
console.log(/\p{Alphabetic}/u.test("!"));
// → false
```

O Unicode define várias propriedades úteis, embora encontrar a que você precisa nem sempre seja trivial. Você pode usar a `\p{Property=Value}`notação para corresponder a qualquer caractere que tenha o valor fornecido para essa propriedade. Se o nome da propriedade for deixado de fora, como em `\p{Name}`, assume-se que o nome seja uma propriedade binária como `Alphabetic`ou uma categoria como `Number`.

## Sumário

Expressões regulares são objetos que representam padrões em cadeias. Eles usam sua própria linguagem para expressar esses padrões.

| `/abc/`    | Uma sequência de caracteres                                  |
| ---------- | ------------------------------------------------------------ |
| `/[abc]/`  | Qualquer caractere de um conjunto de caracteres              |
| `/[^abc]/` | Qualquer caractere que *não esteja* em um conjunto de caracteres |
| `/[0-9]/`  | Qualquer caractere em um intervalo de caracteres             |
| `/x+/`     | Uma ou mais ocorrências do padrão `x`                        |
| `/x+?/`    | Uma ou mais ocorrências, sem remediação                      |
| `/x*/`     | Zero ou mais ocorrências                                     |
| `/x?/`     | Zero ou uma ocorrência                                       |
| `/x{2,4}/` | Duas a quatro ocorrências                                    |
| `/(abc)/`  | Um grupo                                                     |
| `/a|b|c/`  | Qualquer um dos vários padrões                               |
| `/\d/`     | Qualquer caractere de dígito                                 |
| `/\w/`     | Um caractere alfanumérico ("caractere de palavra")           |
| `/\s/`     | Qualquer caractere de espaço em branco                       |
| `/./`      | Qualquer caractere, exceto novas linhas                      |
| `/\b/`     | Um limite de palavras                                        |
| `/^/`      | Início da entrada                                            |
| `/$/`      | Fim da entrada                                               |

Uma expressão regular possui um método `test`para testar se uma determinada sequência corresponde a ela. Ele também possui um método `exec`que, quando uma correspondência é encontrada, retorna uma matriz contendo todos os grupos correspondentes. Essa matriz possui uma `index`propriedade que indica onde a correspondência começou.

As strings têm um `match`método para combiná-las com uma expressão regular e um `search`método para procurar uma, retornando apenas a posição inicial da correspondência. Seu `replace`método pode substituir correspondências de um padrão por uma sequência ou função de substituição.

Expressões regulares podem ter opções, que são gravadas após a barra de fechamento. A `i`opção torna a correspondência entre maiúsculas e minúsculas. A `g`opção torna a expressão *global* , o que, entre outras coisas, faz com que o `replace`método substitua todas as instâncias, e não apenas a primeira. A `y`opção a torna pegajosa, o que significa que ela não procurará adiante e pulará parte da string ao procurar uma correspondência. A `u`opção ativa o modo Unicode, que corrige vários problemas no manuseio de caracteres que ocupam duas unidades de código.

Expressões regulares são uma ferramenta afiada com uma alça estranha. Eles simplificam tremendamente algumas tarefas, mas podem rapidamente se tornar incontroláveis quando aplicados a problemas complexos. Parte de saber usá-los é resistir ao desejo de tentar calçar coisas que eles não conseguem expressar de maneira limpa.

## Exercícios

É quase inevitável que, durante o trabalho nesses exercícios, você fique confuso e frustrado com o comportamento inexplicável de alguma expressão regular. Às vezes, ajuda a inserir sua expressão em uma ferramenta on-line como [*https://debuggex.com*](https://www.debuggex.com/) para verificar se a visualização corresponde ao que você pretendia e experimentar a maneira como ela responde a várias seqüências de entrada.

### Regexp golf

*Código golf* é um termo usado para o jogo de tentar expressar um programa específico com o menor número possível de caracteres. Da mesma forma, *regexp golf* é a prática de escrever a menor expressão regular possível para corresponder a um determinado padrão, e *somente* esse padrão.

Para cada um dos itens a seguir, escreva uma expressão regular para testar se alguma das substrings especificadas ocorre em uma sequência. A expressão regular deve corresponder apenas a seqüências contendo uma das subseqüências descritas. Não se preocupe com os limites das palavras, a menos que seja mencionado explicitamente. Quando sua expressão funcionar, veja se você pode diminuí-la.

1. *carro* e *gato*
2. *pop* e *prop*
3. *furão* , *balsa* e *ferrari*
4. Qualquer palavra que termine em *ious*
5. Um caractere de espaço em branco seguido de um ponto, vírgula, dois pontos ou ponto e vírgula
6. Uma palavra com mais de seis letras
7. Uma palavra sem a letra *e* (ou *E* )

Consulte a tabela no [resumo](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2009%20-%20Express%C3%B5es%20regulares.md#summary_regexp) do [capítulo](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2009%20-%20Express%C3%B5es%20regulares.md#summary_regexp) para obter ajuda. Teste cada solução com algumas seqüências de teste.

```js
// Fill in the regular expressions

verify(/.../,
       ["my car", "bad cats"],
       ["camper", "high art"]);

verify(/.../,
       ["pop culture", "mad props"],
       ["plop", "prrrop"]);

verify(/.../,
       ["ferret", "ferry", "ferrari"],
       ["ferrum", "transfer A"]);

verify(/.../,
       ["how delicious", "spacious room"],
       ["ruinous", "consciousness"]);

verify(/.../,
       ["bad punctuation ."],
       ["escape the period"]);

verify(/.../,
       ["hottentottententen"],
       ["no", "hotten totten tenten"]);

verify(/.../,
       ["red platypus", "wobbling nest"],
       ["earth bed", "learning ape", "BEET"]);


function verify(regexp, yes, no) {
  // Ignore unfinished exercises
  if (regexp.source == "...") return;
  for (let str of yes) if (!regexp.test(str)) {
    console.log(`Failure to match '${str}'`);
  }
  for (let str of no) if (regexp.test(str)) {
    console.log(`Unexpected match for '${str}'`);
  }
}
```

### Estilo de citação

Imagine que você escreveu uma história e usou aspas simples para marcar partes do diálogo. Agora você deseja substituir todas as aspas de diálogo por aspas duplas, mantendo as aspas simples usadas em contrações como *não são* .

Pense em um padrão que distinga esses dois tipos de uso de cotação e faça uma chamada para o `replace`método que faz a substituição adequada.

```js
let text = "'I'm the cook,' he said, 'it's my job.'";
// Change this call.
console.log(text.replace(/A/g, "B"));
// → "I'm the cook," he said, "it's my job."
```

A solução mais óbvia é substituir apenas aspas por um caractere não-palavra em pelo menos um lado - algo assim `/\W'|'\W/`. Mas você também precisa levar em consideração o início e o fim da linha.

Além disso, você deve garantir que a substituição também inclua os caracteres que foram correspondidos pelo `\W`padrão para que eles não sejam descartados. Isso pode ser feito colocando-os entre parênteses e incluindo seus grupos na sequência de substituição ( `$1`, `$2`). Grupos que não correspondem são substituídos por nada.

### Números novamente

Escreva uma expressão que corresponda apenas aos números no estilo JavaScript. Ele deve suportar um sinal de menos *ou* mais opcional na frente do número, do ponto decimal e da notação do expoente - `5e-3`ou - `1E10`novamente com um sinal opcional na frente do expoente. Observe também que não é necessário que haja dígitos na frente ou depois do ponto, mas o número não pode ser um ponto sozinho. Ou seja, `.5`e `5.`são números JavaScript válidos, mas um ponto solitário *não é* .

```js
// Fill in this regular expression.
let number = /^...$/;

// Tests:
for (let str of ["1", "-1", "+15", "1.55", ".5", "5.",
                 "1.3e2", "1E-4", "1e+12"]) {
  if (!number.test(str)) {
    console.log(`Failed to match '${str}'`);
  }
}
for (let str of ["1a", "+-1", "1.2.3", "1+1", "1e4.5",
                 ".5.", "1f5", "."]) {
  if (number.test(str)) {
    console.log(`Incorrectly accepted '${str}'`);
  }
}
```

Primeiro, não esqueça a barra invertida na frente do período.

A correspondência do sinal opcional na frente do número, bem como na frente do expoente, pode ser feita com `[+\-]?`ou `(\+|-|)`(mais, menos ou nada).

A parte mais complicada do exercício é o problema de combinar ambos `"5."`e `".5"`sem também `"."`. Para isso, uma boa solução é usar o `|`operador para separar os dois casos - um ou mais dígitos seguidos opcionalmente por um ponto e zero ou mais dígitos *ou* um ponto seguido por um ou mais dígitos.

Por fim, para tornar o *e* maiúsculas *e* minúsculas, adicione uma `i`opção à expressão regular ou use `[eE]`.