# Capítulo 8 Bugs e erros

> A depuração é duas vezes mais difícil do que escrever o código em primeiro lugar. Portanto, se você escrever o código da maneira mais inteligente possível, por definição, você não é inteligente o suficiente para depurá-lo.
>
> *—Brian Kernighan e PJ Plauger, Os elementos do estilo de programação*

![Imagem de uma coleção de bugs](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/chapter_picture_8.jpg)

Falhas em programas de computador são geralmente chamadas de *bugs* . Faz com que os programadores se sintam bem ao imaginá-los como pequenas coisas que por acaso se arrastam para o nosso trabalho. Na realidade, é claro, nós os colocamos lá.

Se um programa é um pensamento cristalizado, você pode categorizar aproximadamente os erros nos causados por pensamentos confusos e causados por erros introduzidos ao converter um pensamento em código. O primeiro tipo é geralmente mais difícil de diagnosticar e corrigir do que o último.

## Língua

Muitos erros poderiam ser apontados para nós automaticamente pelo computador, se ele soubesse o suficiente sobre o que estamos tentando fazer. Mas aqui a folga do JavaScript é um obstáculo. Seu conceito de associações e propriedades é vago o suficiente para que raramente capte erros de digitação antes de executar o programa. E mesmo assim, ele permite que você faça coisas claramente absurdas sem reclamar, como a computação `true * "monkey"`.

Há algumas coisas das quais o JavaScript reclama. Escrever um programa que não segue a gramática do idioma fará com que o computador se queixe imediatamente. Outras coisas, como chamar algo que não é uma função ou procurar uma propriedade em um valor indefinido, causarão um erro ao ser relatado quando o programa tentar executar a ação.

Mas frequentemente, sua computação sem sentido produzirá meramente `NaN`(não um número) ou um valor indefinido, enquanto o programa continua feliz, convencido de que está fazendo algo significativo. O erro se manifestará apenas mais tarde, após o valor falso ter percorrido várias funções. Ele pode não provocar um erro, mas silenciosamente faz com que a saída do programa esteja errada. Encontrar a fonte de tais problemas pode ser difícil.

O processo de encontrar erros - erros - nos programas é chamado de *depuração* .

## Modo estrito

O JavaScript pode ser um *pouco* mais *rígido,* ativando *o modo estrito* . Isso é feito colocando a sequência `"use strict"`no topo de um arquivo ou corpo de função. Aqui está um exemplo:

```js
function canYouSpotTheProblem() {
  "use strict";
  for (counter = 0; counter < 10; counter++) {
    console.log("Happy happy");
  }
}

canYouSpotTheProblem();
// → ReferenceError: counter is not defined
```

Normalmente, quando você esquece de colocar sua `let`frente de ligação, como `counter`no exemplo, o JavaScript silenciosamente cria uma ligação global e a usa. No modo estrito, um erro é relatado. Isso é muito útil. Note-se, no entanto, que isso não funciona quando a ligação em questão já existe como uma ligação global. Nesse caso, o loop ainda substituirá silenciosamente o valor da ligação.

Outra mudança no modo estrito é que a `this`ligação mantém o valor `undefined`em funções que não são chamadas como métodos. Ao fazer essa chamada fora do modo estrito, `this`refere-se ao objeto de escopo global, que é um objeto cujas propriedades são as ligações globais. Portanto, se você acidentalmente chamar um método ou construtor incorretamente no modo estrito, o JavaScript produzirá um erro assim que tentar ler algo `this`, em vez de escrever feliz no escopo global.

Por exemplo, considere o seguinte código, que chama uma função de construtor sem a `new`palavra-chave para que a sua `this`vai *não* se referir a um objeto recém-construído:

```js
function Person(name) { this.name = name; }
let ferdinand = Person("Ferdinand"); // oops
console.log(name);
// → Ferdinand
```

Portanto, a chamada falsa foi `Person`bem - sucedida, mas retornou um valor indefinido e criou a ligação global `name`. No modo estrito, o resultado é diferente.

```js
"use strict";
function Person(name) { this.name = name; }
let ferdinand = Person("Ferdinand"); // forgot new
// → TypeError: Cannot set property 'name' of undefined
```

Somos informados imediatamente de que algo está errado. Isso é útil.

Felizmente, os construtores criados com a `class`notação sempre reclamarão se forem chamados sem `new`, tornando isso menos problemático, mesmo no modo não estrito.

O modo estrito faz mais algumas coisas. Ele não permite atribuir vários parâmetros a uma função com o mesmo nome e remove completamente certos recursos problemáticos da linguagem (como a `with`declaração, que é tão errada que não será discutida mais neste livro).

Em resumo, colocar `"use strict"`no topo do seu programa raramente dói e pode ajudá-lo a identificar um problema.

## Tipos

Alguns idiomas desejam conhecer os tipos de todas as suas ligações e expressões antes mesmo de executar um programa. Eles informarão imediatamente quando um tipo é usado de maneira inconsistente. O JavaScript considera os tipos apenas ao executar o programa e, mesmo assim, muitas vezes tenta converter implicitamente valores para o tipo que espera, por isso não ajuda muito.

Ainda assim, os tipos fornecem uma estrutura útil para falar sobre programas. Muitos erros advêm de se confundir sobre o tipo de valor que entra ou sai de uma função. Se você tiver essas informações escritas, é menos provável que fique confuso.

Você pode adicionar um comentário como o seguinte antes da `goalOrientedRobot`função do capítulo anterior para descrever seu tipo:

```js
// (VillageState, Array) → {direction: string, memory: Array}
function goalOrientedRobot(state, memory) {
  // ...
}
```

Existem várias convenções diferentes para anotar programas JavaScript com tipos.

Uma coisa sobre os tipos é que eles precisam apresentar sua própria complexidade para poder descrever código suficiente para serem úteis. O que você acha que seria o tipo de `randomPick`função que retorna um elemento aleatório de uma matriz? Você precisa de introduzir uma *variável do tipo* , *T* , que pode servir de qualquer tipo, de modo que você pode dar `randomPick`um tipo como `([T]) → T`(função de um conjunto de *T* s a uma *T* ).

Quando os tipos de um programa são conhecidos, é possível que o computador os *verifique* , apontando erros antes da execução do programa. Existem vários dialetos JavaScript que adicionam tipos ao idioma e os verificam. O mais popular é chamado [TypeScript](https://www.typescriptlang.org/) . Se você estiver interessado em adicionar mais rigor aos seus programas, recomendo que tente.

Neste livro, continuaremos usando código JavaScript bruto, perigoso e sem tipo.

## Teste

Se o idioma não ajudar muito a encontrar erros, teremos que encontrá-los da maneira mais difícil: executando o programa e ver se ele faz a coisa certa.

Fazer isso manualmente, de novo e de novo, é uma péssima idéia. Não é apenas irritante, mas também tende a ser ineficaz, pois leva muito tempo para testar exaustivamente tudo toda vez que você faz uma alteração.

Os computadores são bons em tarefas repetitivas e o teste é a tarefa repetitiva ideal. Teste automatizado é o processo de escrever um programa que testa outro programa. Escrever testes é um pouco mais trabalhoso do que testar manualmente, mas depois de fazer isso, você ganha uma espécie de superpotência: leva apenas alguns segundos para verificar se o seu programa ainda se comporta corretamente em todas as situações para as quais você escreveu testes. Quando você quebra algo, notará imediatamente, em vez de encontrá-lo aleatoriamente mais tarde.

Os testes geralmente assumem a forma de pequenos programas rotulados que verificam algum aspecto do seu código. Por exemplo, um conjunto de testes para o método (padrão, provavelmente já foi testado por outra pessoa) `toUpperCase`pode ser assim:

```js
function test(label, body) {
  if (!body()) console.log(`Failed: ${label}`);
}

test("convert Latin text to uppercase", () => {
  return "hello".toUpperCase() == "HELLO";
});
test("convert Greek text to uppercase", () => {
  return "Χαίρετε".toUpperCase() == "ΧΑΊΡΕΤΕ";
});
test("don't convert case-less characters", () => {
  return "مرحبا".toUpperCase() == "مرحبا";
});
```

Escrever testes como esse tende a produzir código bastante repetitivo e constrangedor. Felizmente, existem peças de software que ajudam a criar e executar coleções de testes ( *suítes de testes* ), fornecendo uma linguagem (na forma de funções e métodos) adequada para expressar testes e fornecendo informações informativas quando um teste falha. Estes são geralmente chamados de *corredores de teste* .

Algum código é mais fácil de testar do que outro código. Geralmente, quanto mais objetos externos com os quais o código interage, mais difícil é configurar o contexto no qual testá-lo. O estilo de programação mostrado no [capítulo anterior](https://eloquentjavascript.net/07_robot.html) , que usa valores persistentes independentes em vez de alterar objetos, tende a ser fácil de testar.

## Depuração

Depois que você perceber que algo está errado com o seu programa porque ele se comporta mal ou gera erros, o próximo passo é descobrir *qual é* o problema.

Às vezes é óbvio. A mensagem de erro apontará para uma linha específica do seu programa e, se você examinar a descrição do erro e essa linha de código, poderá ver o problema com frequência.

Mas não sempre. Às vezes, a linha que desencadeou o problema é simplesmente o primeiro lugar em que um valor irregular produzido em outro lugar é usado de maneira inválida. Se você já resolveu os exercícios dos capítulos anteriores, provavelmente já experimentou essas situações.

O programa de exemplo a seguir tenta converter um número inteiro em uma string em uma determinada base (decimal, binária etc.) escolhendo repetidamente o último dígito e dividindo o número para se livrar desse dígito. Mas a saída estranha que atualmente produz sugere que ele possui um bug.

```js
function numberToString(n, base = 10) {
  let result = "", sign = "";
  if (n < 0) {
    sign = "-";
    n = -n;
  }
  do {
    result = String(n % base) + result;
    n /= base;
  } while (n > 0);
  return sign + result;
}
console.log(numberToString(13, 10));
// → 1.5e-3231.3e-3221.3e-3211.3e-3201.3e-3191.3e-3181.3…
```

Mesmo que você já veja o problema, finja por um momento que não está. Sabemos que nosso programa está com defeito e queremos descobrir o porquê.

É aqui que você deve resistir ao desejo de começar a fazer alterações aleatórias no código para ver se isso o torna melhor. Em vez disso, *pense* . Analise o que está acontecendo e elabore uma teoria de por que isso pode estar acontecendo. Em seguida, faça observações adicionais para testar essa teoria - ou, se você ainda não tem uma teoria, faça observações adicionais para ajudá-lo a criar uma.

Colocar algumas `console.log`chamadas estratégicas no programa é uma boa maneira de obter informações adicionais sobre o que o programa está fazendo. Neste caso, queremos `n`levar os valores `13`, `1`e em seguida `0`. Vamos escrever seu valor no início do loop.

```
13
1.3
0,13
0,013
...
1.5e-323
```

*Certo* . Dividir 13 por 10 não produz um número inteiro. Em vez de `n /= base`, o que realmente queremos é que o número seja corretamente "deslocado" para a direita.`n = Math.floor(n / base)`

Uma alternativa ao uso `console.log`para espiar o comportamento do programa é usar os recursos de *depurador* do seu navegador. Os navegadores vêm com a capacidade de definir um *ponto* de *interrupção* em uma linha específica do seu código. Quando a execução do programa atinge uma linha com um ponto de interrupção, ela é pausada e você pode inspecionar os valores das ligações nesse ponto. Não entrarei em detalhes, pois os depuradores diferem de navegador para navegador, mas procure nas ferramentas de desenvolvedor do navegador ou pesquise na Web para obter mais informações.

Outra maneira de definir um ponto de interrupção é incluir uma `debugger`instrução (consistindo simplesmente nessa palavra-chave) em seu programa. Se as ferramentas de desenvolvedor do seu navegador estiverem ativas, o programa será pausado sempre que chegar a essa declaração.

## Propagação de erro

Infelizmente, nem todos os problemas podem ser evitados pelo programador. Se o seu programa se comunicar com o mundo externo de alguma forma, é possível obter informações incorretas, sobrecarregar-se com o trabalho ou causar falha na rede.

Se você estiver programando apenas para si mesmo, poderá ignorar esses problemas até que eles ocorram. Mas se você criar algo que será usado por mais alguém, geralmente deseja que o programa seja melhor do que apenas travar. Às vezes, a coisa certa a fazer é levar a má entrada rapidamente e continuar correndo. Em outros casos, é melhor relatar ao usuário o que deu errado e desistir. Mas em qualquer situação, o programa precisa fazer algo ativamente em resposta ao problema.

Digamos que você tenha uma função `promptNumber`que solicite ao usuário um número e o retorne. O que deve retornar se o usuário digitar “laranja”?

Uma opção é fazer com que ele retorne um valor especial. Escolhas comuns para esses valores são `null`, `undefined`ou -1.

```js
function promptNumber(question) {
  let result = Number(prompt(question));
  if (Number.isNaN(result)) return null;
  else return result;
}

console.log(promptNumber("How many trees do you see?"));
```

Agora, qualquer código que ligue `promptNumber`deve verificar se um número real foi lido e, na sua falta, deve se recuperar de alguma forma - talvez perguntando novamente ou preenchendo um valor padrão. Ou pode novamente retornar um valor especial para *o seu* interlocutor para indicar que ele não conseguiu fazer o que foi solicitado.

Em muitas situações, principalmente quando os erros são comuns e o chamador deve levá-los em consideração explicitamente, retornar um valor especial é uma boa maneira de indicar um erro. No entanto, tem suas desvantagens. Primeiro, e se a função já puder retornar todo tipo de valor possível? Nessa função, você terá que fazer algo como agrupar o resultado em um objeto para poder distinguir sucesso de falha.

```js
function lastElement(array) {
  if (array.length == 0) {
    return {failed: true};
  } else {
    return {element: array[array.length - 1]};
  }
}
```

O segundo problema com o retorno de valores especiais é que ele pode levar a códigos estranhos. Se um pedaço de código chama `promptNumber`10 vezes, ele deve verificar 10 vezes se `null`foi retornado. E se sua resposta à busca `null`é simplesmente retornar a `null`si mesma, os chamadores da função, por sua vez, precisam verificar isso e assim por diante.

## Exceções

Quando uma função não pode prosseguir normalmente, o que nós *gostamos* de fazer é apenas parar o que estamos fazendo e saltar imediatamente para um lugar que sabe como lidar com o problema. É isso que o *tratamento de exceção* faz.

As exceções são um mecanismo que possibilita que o código que ocorre com um problema *crie* (ou *lance* ) uma exceção. Uma exceção pode ser qualquer valor. Aumentar um deles lembra um retorno sobrecarregado de uma função: ele salta não apenas da função atual, mas também de seus chamadores, até a primeira chamada que iniciou a execução atual. Isso é chamado de *desenrolar a pilha* . Você pode se lembrar da pilha de chamadas de função mencionada no [Capítulo 3](https://eloquentjavascript.net/03_functions.html#stack) . Uma exceção diminui o zoom nessa pilha, descartando todos os contextos de chamada que encontra.

Se as exceções sempre diminuíssem até o final da pilha, elas não seriam muito úteis. Eles forneceriam apenas uma nova maneira de explodir seu programa. O poder deles reside no fato de que você pode definir "obstáculos" ao longo da pilha para *capturar* a exceção enquanto diminui o zoom. Depois de capturar uma exceção, você pode fazer algo para solucionar o problema e continuar executando o programa.

Aqui está um exemplo:

```js
function promptDirection(question) {
  let result = prompt(question);
  if (result.toLowerCase() == "left") return "L";
  if (result.toLowerCase() == "right") return "R";
  throw new Error("Invalid direction: " + result);
}

function look() {
  if (promptDirection("Which way?") == "L") {
    return "a house";
  } else {
    return "two angry bears";
  }
}

try {
  console.log("You see", look());
} catch (error) {
  console.log("Something went wrong: " + error);
}
```

A `throw`palavra-chave é usada para gerar uma exceção. A captura de um é feita envolvendo um pedaço de código em um `try`bloco, seguido pela palavra-chave `catch`. Quando o código no `try`bloco faz com que uma exceção seja levantada, o `catch`bloco é avaliado, com o nome entre parênteses vinculado ao valor da exceção. Depois que o `catch`bloco termina - ou se o `try`bloco termina sem problemas - o programa continua abaixo da `try/catch`declaração inteira .

Nesse caso, usamos o `Error`construtor para criar nosso valor de exceção. Este é um construtor JavaScript padrão que cria um objeto com uma `message`propriedade. Na maioria dos ambientes JavaScript, as instâncias desse construtor também coletam informações sobre a pilha de chamadas que existia quando a exceção foi criada, o chamado *rastreamento de pilha* . Essas informações são armazenadas na `stack`propriedade e podem ser úteis ao tentar depurar um problema: indica a função em que o problema ocorreu e quais funções fizeram a chamada com falha.

Observe que a `look`função ignora completamente a possibilidade de `promptDirection`dar errado. Essa é a grande vantagem das exceções: o código de tratamento de erros é necessário apenas no ponto em que o erro ocorre e no ponto em que é tratado. As funções intermediárias podem esquecer tudo.

Bem, quase...

## Limpeza após exceções

O efeito de uma exceção é outro tipo de fluxo de controle. Toda ação que pode causar uma exceção, que é praticamente toda chamada de função e acesso à propriedade, pode fazer com que o controle deixe seu código repentinamente.

Isso significa que quando o código tem vários efeitos colaterais, mesmo que seu fluxo de controle "regular" pareça que sempre acontece, uma exceção pode impedir que alguns deles ocorram.

Aqui está um código bancário muito ruim.

```js
const accounts = {
  a: 100,
  b: 0,
  c: 20
};

function getAccount() {
  let accountName = prompt("Enter an account name");
  if (!accounts.hasOwnProperty(accountName)) {
    throw new Error(`No such account: ${accountName}`);
  }
  return accountName;
}

function transfer(from, amount) {
  if (accounts[from] < amount) return;
  accounts[from] -= amount;
  accounts[getAccount()] += amount;
}
```

A `transfer`função transfere uma quantia em dinheiro de uma determinada conta para outra, solicitando o nome da outra conta no processo. Se for fornecido um nome de conta inválido, `getAccount`lança uma exceção.

Mas `transfer` *primeiro* remova o dinheiro da conta e *depois* ligue `getAccount`antes de adicioná-lo a outra conta. Se for interrompido por uma exceção nesse ponto, fará com que o dinheiro desapareça.

Esse código poderia ter sido escrito de forma um pouco mais inteligente, por exemplo, ligando `getAccount`antes de começar a movimentar dinheiro. Mas frequentemente problemas como esse ocorrem de maneiras mais sutis. Mesmo funções que não parecem lançar uma exceção podem fazê-lo em circunstâncias excepcionais ou quando contêm um erro de programador.

Uma maneira de resolver isso é usar menos efeitos colaterais. Novamente, um estilo de programação que calcula novos valores em vez de alterar os dados existentes ajuda. Se um pedaço de código parar de ser executado no meio da criação de um novo valor, ninguém nunca verá o valor pela metade e não haverá problema.

Mas isso nem sempre é prático. Portanto, há outro recurso que as `try`declarações têm. Eles podem ser seguidos por um `finally`bloco em vez de ou além de um `catch`bloco. Um `finally`bloco diz " aconteça o *que* acontecer, execute esse código depois de tentar executá-lo no `try`bloco".

```js
function transfer(from, amount) {
  if (accounts[from] < amount) return;
  let progress = 0;
  try {
    accounts[from] -= amount;
    progress = 1;
    accounts[getAccount()] += amount;
    progress = 2;
  } finally {
    if (progress == 1) {
      accounts[from] += amount;
    }
  }
}
```

Essa versão da função rastreia seu progresso e, ao sair, percebe que foi interrompida em um ponto em que havia criado um estado inconsistente do programa, repara os danos causados.

Observe que, embora o `finally`código seja executado quando uma exceção é lançada no `try`bloco, ele não interfere na exceção. Depois que o `finally`bloco é executado, a pilha continua desenrolando.

É difícil escrever programas que funcionem de maneira confiável, mesmo quando as exceções surgem em locais inesperados. Muitas pessoas simplesmente não se incomodam e, como as exceções geralmente são reservadas para circunstâncias excepcionais, o problema pode ocorrer tão raramente que nunca é percebido. Se isso é uma coisa boa ou realmente ruim, depende de quanto dano o software causará quando falhar.

## Captura seletiva

Quando uma exceção chega ao fundo da pilha sem ser detectada, ela é tratada pelo ambiente. O que isso significa difere entre os ambientes. Nos navegadores, uma descrição do erro geralmente é gravada no console do JavaScript (acessível através do menu Ferramentas ou Desenvolvedor do navegador). O Node.js, o ambiente JavaScript sem navegador que discutiremos no [Capítulo 20](https://eloquentjavascript.net/20_node.html) , é mais cuidadoso com a corrupção de dados. Ele interrompe todo o processo quando ocorre uma exceção não tratada.

Para erros de programadores, deixar o erro passar geralmente é o melhor que você pode fazer. Uma exceção não tratada é uma maneira razoável de sinalizar um programa interrompido, e o console JavaScript, em navegadores modernos, fornece algumas informações sobre quais chamadas de função estavam na pilha quando o problema ocorreu.

Para problemas que se *espera* que ocorram durante o uso rotineiro, travar com uma exceção não tratada é uma estratégia terrível.

Usos inválidos do idioma, como referenciar uma ligação inexistente, procurar uma propriedade `null`ou chamar algo que não é uma função, também resultarão no surgimento de exceções. Tais exceções também podem ser capturadas.

Quando um `catch`corpo é inserido, tudo o que sabemos é que *algo* em nosso `try`corpo causou uma exceção. Mas não sabemos o *que* fez ou *qual* exceção causou.

JavaScript (em uma omissão bastante flagrante) não fornece suporte direto para capturar seletivamente exceções: ou você captura todas elas ou não captura nenhuma. Isso torna tentador *supor* que a exceção que você recebe é a que você estava pensando quando escreveu o `catch`bloco.

Mas pode não ser. Alguma outra suposição pode ser violada ou você pode ter introduzido um bug que está causando uma exceção. Aqui está um exemplo que *tenta* continuar chamando `promptDirection`até obter uma resposta válida:

```js
for (;;) {
  try {
    let dir = promtDirection("Where?"); // ← typo!
    console.log("You chose ", dir);
    break;
  } catch (e) {
    console.log("Not a valid direction. Try again.");
  }
}
```

A `for (;;)`construção é uma maneira de criar intencionalmente um loop que não termina sozinho. Só saímos do circuito quando uma direção válida é dada. *Mas* nós escrevemos incorretamente `promptDirection`, o que resultará em um erro de "variável indefinida". Como o `catch`bloco ignora completamente seu valor de exceção ( `e`), supondo que ele saiba qual é o problema, ele trata incorretamente o erro de ligação como indicando entrada incorreta. Isso não apenas causa um loop infinito, como "enterra" a mensagem de erro útil sobre a ligação incorreta.

Como regra geral, não colete exceções generalizadas, a menos que seja com o objetivo de "rotear" elas para algum lugar - por exemplo, pela rede para informar outro sistema que nosso programa travou. E mesmo assim, pense cuidadosamente em como você pode estar escondendo informações.

Então, queremos capturar um tipo *específico* de exceção. Podemos fazer isso verificando no `catch`bloco se a exceção que recebemos é aquela em que estamos interessados e repetindo novamente. Mas como reconhecemos uma exceção?

Poderíamos comparar sua `message`propriedade com a mensagem de erro que esperamos. Mas essa é uma maneira instável de escrever código - estaríamos usando informações destinadas ao consumo humano (a mensagem) para tomar uma decisão programática. Assim que alguém alterar (ou traduzir) a mensagem, o código deixará de funcionar.

Em vez disso, vamos definir um novo tipo de erro e usá `instanceof`-lo para identificá-lo.

```js
class InputError extends Error {}

function promptDirection(question) {
  let result = prompt(question);
  if (result.toLowerCase() == "left") return "L";
  if (result.toLowerCase() == "right") return "R";
  throw new InputError("Invalid direction: " + result);
}
```

A nova classe de erro se estende `Error`. Ele não define seu próprio construtor, o que significa que ele herda o `Error`construtor, o que espera uma mensagem de string como argumento. De fato, ele não define nada - a classe está vazia. `InputError`objetos se comportam como `Error`objetos, exceto que eles têm uma classe diferente pela qual podemos reconhecê-los.

Agora o loop pode capturá-los com mais cuidado.

```js
for (;;) {
  try {
    let dir = promptDirection("Where?");
    console.log("You chose ", dir);
    break;
  } catch (e) {
    if (e instanceof InputError) {
      console.log("Not a valid direction. Try again.");
    } else {
      throw e;
    }
  }
}
```

Isso irá capturar apenas instâncias `InputError`e permitir exceções não relacionadas. Se você reintroduzir o erro de digitação, o erro de ligação indefinido será relatado corretamente.

## Asserções

*Asserções* são verificações dentro de um programa que verificam se algo está do jeito que deveria ser. Eles são usados não para lidar com situações que podem surgir em operação normal, mas para encontrar erros no programador.

Se, por exemplo, `firstElement`for descrito como uma função que nunca deve ser chamada em matrizes vazias, podemos escrevê-lo assim:

```js
function firstElement(array) {
  if (array.length == 0) {
    throw new Error("firstElement called with []");
  }
  return array[0];
}
```

Agora, em vez de retornar silenciosamente indefinido (o que você obtém ao ler uma propriedade de matriz que não existe), isso explodirá seu programa em voz alta assim que você o usar incorretamente. Isso torna menos provável que esses erros passem despercebidos e seja mais fácil encontrar sua causa quando eles ocorrem.

Eu não recomendo tentar escrever asserções para todo tipo possível de entrada incorreta. Isso daria muito trabalho e levaria a códigos muito barulhentos. Você vai querer reservá-los para erros fáceis de cometer (ou que você comete).

## Sumário

Erros e opiniões ruins são fatos da vida. Uma parte importante da programação é encontrar, diagnosticar e corrigir bugs. Os problemas podem ficar mais fáceis de serem notados se você tiver um conjunto de testes automatizado ou adicionar asserções aos seus programas.

Os problemas causados por fatores fora do controle do programa geralmente devem ser tratados com graciosidade. Às vezes, quando o problema pode ser tratado localmente, valores de retorno especiais são uma boa maneira de rastreá-los. Caso contrário, as exceções podem ser preferíveis.

O lançamento de uma exceção faz com que a pilha de chamadas seja desenrolada até o próximo `try/catch`bloco anexo ou até a parte inferior da pilha. O valor da exceção será fornecido ao `catch`bloco que o captura, que deve verificar se realmente é o tipo de exceção esperado e, em seguida, fazer algo com ele. Para ajudar a resolver o fluxo de controle imprevisível causado por exceções, os `finally`blocos podem ser usados para garantir que um pedaço de código *sempre seja* executado quando um bloco for concluído.

## Exercícios

### Repetir

Digamos que você tenha uma função `primitiveMultiply`que em 20% dos casos multiplique dois números e nos outros 80% dos casos crie uma exceção do tipo `MultiplicatorUnitFailure`. Escreva uma função que agrupe essa função desajeitada e continue tentando até que uma chamada seja bem-sucedida, após a qual ela retornará o resultado.

Certifique-se de lidar apenas com as exceções que você está tentando tratar.

```js
class MultiplicatorUnitFailure extends Error {}

function primitiveMultiply(a, b) {
  if (Math.random() < 0.2) {
    return a * b;
  } else {
    throw new MultiplicatorUnitFailure("Klunk");
  }
}

function reliableMultiply(a, b) {
  // Your code here.
}

console.log(reliableMultiply(8, 8));
// → 64
```

### A caixa trancada

Considere o seguinte objeto (bastante artificial):

```js
const box = {
  locked: true,
  unlock() { this.locked = false; },
  lock() { this.locked = true;  },
  _content: [],
  get content() {
    if (this.locked) throw new Error("Locked!");
    return this._content;
  }
};
```

É uma caixa com uma fechadura. Há uma matriz na caixa, mas você pode acessá-la somente quando a caixa estiver desbloqueada. `_content`É proibido acessar diretamente a propriedade privada .

Escreva uma função chamada `withBoxUnlocked`que aceite um valor de função como argumento, desbloqueie a caixa, execute a função e garanta que a caixa seja bloqueada novamente antes de retornar, independentemente de a função de argumento ter retornado normalmente ou ter gerado uma exceção.

```js
const box = {
  locked: true,
  unlock() { this.locked = false; },
  lock() { this.locked = true;  },
  _content: [],
  get content() {
    if (this.locked) throw new Error("Locked!");
    return this._content;
  }
};

function withBoxUnlocked(body) {
  // Your code here.
}

withBoxUnlocked(function() {
  box.content.push("gold piece");
});

try {
  withBoxUnlocked(function() {
    throw new Error("Pirates on the horizon! Abort!");
  });
} catch (e) {
  console.log("Error raised: " + e);
}
console.log(box.locked);
// → true
```

Para ganhar pontos extras, verifique se você telefona `withBoxUnlocked`quando a caixa já está desbloqueada, a caixa permanece desbloqueada.

Este exercício exige um `finally`bloco. Sua função deve primeiro desbloquear a caixa e depois chamar a função de argumento de dentro de um `try`corpo. O `finally`bloco depois deve travar a caixa novamente.

Para garantir que não bloqueemos a caixa quando ela ainda não estiver bloqueada, verifique sua trava no início da função e desbloqueie e bloqueie-a apenas quando ela for bloqueada.