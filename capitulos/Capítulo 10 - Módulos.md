# Capítulo 10 Módulos

> Escreva um código fácil de excluir, não fácil de estender.
>
> *—Tef, a programação é terrível*

![Imagem de um edifício construído a partir de peças modulares](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/chapter_picture_10.jpg)

O programa ideal tem uma estrutura cristalina. É fácil explicar como funciona e cada parte tem um papel bem definido.

Um programa real típico cresce organicamente. Novas peças de funcionalidade são adicionadas à medida que novas necessidades surgem. Estruturar - e preservar a estrutura - é um trabalho adicional. É um trabalho que será recompensado apenas no futuro, na *próxima* vez que alguém trabalhar no programa. Portanto, é tentador negligenciá-lo e permitir que as partes do programa fiquem profundamente enredadas.

Isso causa duas questões práticas. Primeiro, é difícil entender esse sistema. Se tudo pode tocar em tudo o mais, é difícil olhar isoladamente qualquer peça. Você é forçado a construir uma compreensão holística da coisa toda. Segundo, se você quiser usar alguma funcionalidade de um programa em outra situação, reescrevê-la pode ser mais fácil do que tentar separá-la de seu contexto.

A frase “grande bola de barro” é frequentemente usada para programas grandes e sem estrutura. Tudo fica junto, e quando você tenta escolher um pedaço, tudo se desfaz e suas mãos se sujam.

## Módulos

*Módulos* são uma tentativa de evitar esses problemas. Um módulo é um pedaço de programa que especifica em quais outros pedaços ele se baseia e em qual funcionalidade ele fornece para outros módulos usarem (sua *interface* ).

As interfaces de módulo têm muito em comum com as interfaces de objetos, como vimos no [Capítulo 6](https://eloquentjavascript.net/06_object.html#interface) . Eles fazem parte do módulo disponível para o mundo exterior e mantêm o restante em sigilo. Ao restringir a maneira como os módulos interagem entre si, o sistema se torna mais como LEGO, onde as peças interagem por meio de conectores bem definidos e menos como lama, onde tudo se mistura com tudo.

As relações entre os módulos são chamadas de *dependências* . Quando um módulo precisa de uma peça de outro módulo, diz-se que depende desse módulo. Quando esse fato é claramente especificado no próprio módulo, ele pode ser usado para descobrir quais outros módulos precisam estar presentes para poder usar um determinado módulo e carregar automaticamente dependências.

Para separar os módulos dessa maneira, cada um precisa de seu próprio escopo privado.

Apenas colocar seu código JavaScript em arquivos diferentes não atende a esses requisitos. Os arquivos ainda compartilham o mesmo espaço para nome global. Eles podem, intencionalmente ou acidentalmente, interferir nas ligações um do outro. E a estrutura de dependência permanece incerta. Podemos fazer melhor, como veremos mais adiante neste capítulo.

Projetar uma estrutura de módulo de encaixe para um programa pode ser difícil. Na fase em que você ainda está explorando o problema, tentando coisas diferentes para ver o que funciona, convém não se preocupar muito com isso, pois pode ser uma grande distração. Depois de ter algo sólido, é um bom momento para dar um passo atrás e organizá-lo.

## Pacotes

Uma das vantagens de criar um programa com partes separadas e ser capaz de executá-las por conta própria é que você poderá aplicar a mesma peça em programas diferentes.

Mas como você configura isso? Digamos que eu queira usar a `parseINI`função do [Capítulo 9](https://eloquentjavascript.net/09_regexp.html#ini) em outro programa. Se estiver claro do que a função depende (neste caso, nada), posso copiar todo o código necessário no meu novo projeto e usá-lo. Porém, se eu encontrar um erro nesse código, provavelmente o corrigirei em qualquer programa com o qual estou trabalhando no momento e esquecerei de corrigi-lo também no outro programa.

Depois de começar a duplicar o código, você rapidamente estará perdendo tempo e energia movendo cópias e mantendo-as atualizadas.

É aí que os *pacotes* entram. Um pacote é um pedaço de código que pode ser distribuído (copiado e instalado). Pode conter um ou mais módulos e possui informações sobre quais outros pacotes ele depende. Um pacote também geralmente vem com documentação explicando o que faz, para que as pessoas que não o escreveram ainda possam usá-lo.

Quando um problema é encontrado em um pacote ou um novo recurso é adicionado, o pacote é atualizado. Agora, os programas que dependem dele (que também podem ser pacotes) podem ser atualizados para a nova versão.

Trabalhar dessa maneira requer infraestrutura. Precisamos de um local para armazenar e encontrar pacotes e uma maneira conveniente de instalá-los e atualizá-los. No mundo JavaScript, essa infraestrutura é fornecida pelo NPM ( [*https://npmjs.org*](https://npmjs.org/) ).

O NPM é duas coisas: um serviço online no qual é possível fazer o download (e fazer upload) de pacotes e um programa (fornecido com o Node.js.) que ajuda a instalar e gerenciar os pacotes.

No momento da redação deste artigo, existem mais de meio milhão de pacotes diferentes disponíveis no NPM. Uma grande parte deles é lixo, devo mencionar, mas quase todos os pacotes úteis e publicamente disponíveis podem ser encontrados lá. Por exemplo, um analisador de arquivo INI, semelhante ao que construímos no [Capítulo 9](https://eloquentjavascript.net/09_regexp.html) , está disponível sob o nome do pacote `ini`.

[O capítulo 20](https://eloquentjavascript.net/20_node.html) mostrará como instalar esses pacotes localmente usando o `npm`programa de linha de comando.

Ter pacotes de qualidade disponíveis para download é extremamente valioso. Isso significa que muitas vezes podemos evitar a reinvenção de um programa que 100 pessoas escreveram antes e obter uma implementação sólida e bem testada ao pressionar algumas teclas.

É barato copiar um software; portanto, depois que alguém o escreve, distribui-lo a outras pessoas é um processo eficiente. Mas escrevê-lo em primeiro lugar *é* trabalho, e responder a pessoas que encontraram problemas no código, ou que desejam propor novos recursos, é ainda mais trabalho.

Por padrão, você possui os direitos autorais do código que escreve e outras pessoas podem usá-lo apenas com sua permissão. Mas como algumas pessoas são boas e a publicação de um bom software pode ajudar a torná-lo um pouco famoso entre os programadores, muitos pacotes são publicados sob uma licença que permite explicitamente que outras pessoas o usem.

A maioria dos códigos no NPM é licenciada dessa maneira. Algumas licenças exigem que você também publique o código que você constrói sobre o pacote sob a mesma licença. Outros são menos exigentes, exigindo apenas que você mantenha a licença com o código ao distribuí-lo. A comunidade JavaScript usa principalmente o último tipo de licença. Ao usar os pacotes de outras pessoas, verifique se você está ciente da licença deles.

## Módulos improvisados

Até 2015, a linguagem JavaScript não possuía sistema de módulos embutido. No entanto, as pessoas construíram grandes sistemas em JavaScript por mais de uma década e *precisavam de* módulos.

Então, eles projetaram seus próprios sistemas de módulos no topo da linguagem. Você pode usar funções JavaScript para criar escopos e objetos locais para representar interfaces de módulo.

Este é um módulo para ir entre nomes de dia e números (como retornado por `Date`'s `getDay`método). Sua interface consiste em `weekDay.name`e `weekDay.number`, e oculta sua ligação local `names`dentro do escopo de uma expressão de função que é chamada imediatamente.

```js
const weekDay = function() {
  const names = ["Sunday", "Monday", "Tuesday", "Wednesday",
                 "Thursday", "Friday", "Saturday"];
  return {
    name(number) { return names[number]; },
    number(name) { return names.indexOf(name); }
  };
}();

console.log(weekDay.name(weekDay.number("Sunday")));
// → Sunday
```

Esse estilo de módulos fornece isolamento, até certo ponto, mas não declara dependências. Em vez disso, apenas coloca sua interface no escopo global e espera que suas dependências, se houver, façam o mesmo. Por um longo tempo, essa foi a principal abordagem usada na programação da Web, mas atualmente está obsoleta.

Se queremos tornar as relações de dependência parte do código, teremos que assumir o controle do carregamento de dependências. Fazer isso requer a capacidade de executar seqüências de caracteres como código. JavaScript pode fazer isso.

## Avaliando dados como código

Existem várias maneiras de obter dados (uma sequência de códigos) e executá-los como parte do programa atual.

A maneira mais óbvia é o operador especial `eval`, que executará uma sequência no escopo *atual* . Geralmente, é uma má idéia, pois quebra algumas das propriedades que os escopos normalmente possuem, como é facilmente previsível a qual ligação um determinado nome se refere.

```js
const x = 1;
function evalAndReturnX(code) {
  eval(code);
  return x;
}

console.log(evalAndReturnX("var x = 2"));
// → 2
console.log(x);
// → 1
```

Uma maneira menos assustadora de interpretar dados como código é usar o `Function`construtor. São necessários dois argumentos: uma string contendo uma lista separada por vírgula de nomes de argumentos e uma string contendo o corpo da função. Ele agrupa o código em um valor de função para que ele obtenha seu próprio escopo e não faça coisas estranhas com outros escopos.

```js
let plusOne = Function("n", "return n + 1;");
console.log(plusOne(4));
// → 5
```

É exatamente isso que precisamos para um sistema de módulos. Podemos envolver o código do módulo em uma função e usar o escopo dessa função como escopo do módulo.

## CommonJS

A abordagem mais amplamente usada para módulos JavaScript parafusados é chamada de *módulos CommonJS* . O Node.js usa e é o sistema usado pela maioria dos pacotes no NPM.

O conceito principal nos módulos CommonJS é uma função chamada `require`. Quando você chama isso com o nome do módulo de uma dependência, ele garante que o módulo esteja carregado e retorne sua interface.

Como o carregador agrupa o código do módulo em uma função, os módulos obtêm automaticamente seu próprio escopo local. Tudo o que eles precisam fazer é chamar `require`para acessar suas dependências e colocar sua interface no objeto vinculado `exports`.

Este módulo de exemplo fornece uma função de formatação de data. Ele usa dois pacotes do NPM - `ordinal`para converter números em seqüências de caracteres como `"1st"`e `"2nd"`, e `date-names`obter os nomes em inglês por dias da semana e meses. Ele exporta uma única função, `formatDate`que pega um `Date`objeto e uma sequência de modelo.

A sequência do modelo pode conter códigos que direcionam o formato, como `YYYY`para o ano inteiro e `Do`para o dia ordinal do mês. Você pode atribuir a ele uma string como `"MMMM Do YYYY"`obter resultados como "22 de novembro de 2017".

```js
const ordinal = require("ordinal");
const {days, months} = require("date-names");

exports.formatDate = function(date, format) {
  return format.replace(/YYYY|M(MMM)?|Do?|dddd/g, tag => {
    if (tag == "YYYY") return date.getFullYear();
    if (tag == "M") return date.getMonth();
    if (tag == "MMMM") return months[date.getMonth()];
    if (tag == "D") return date.getDate();
    if (tag == "Do") return ordinal(date.getDate());
    if (tag == "dddd") return days[date.getDay()];
  });
};
```

A interface de `ordinal`é uma função única, enquanto `date-names`exporta um objeto contendo várias coisas - `days`e `months`são matrizes de nomes. A reestruturação é muito conveniente ao criar ligações para interfaces importadas.

O módulo adiciona sua função de interface para `exports`que os módulos que dependem dele tenham acesso a ele. Poderíamos usar o módulo assim:

```js
const {formatDate} = require("./format-date");

console.log(formatDate(new Date(2017, 9, 13),
                       "dddd the Do"));
// → Friday the 13th
```

Podemos definir `require`, na sua forma mais mínima, o seguinte:

```js
require.cache = Object.create(null);

function require(name) {
  if (!(name in require.cache)) {
    let code = readFile(name);
    let module = {exports: {}};
    require.cache[name] = module;
    let wrapper = Function("require, exports, module", code);
    wrapper(require, module.exports, module);
  }
  return require.cache[name].exports;
}
```

Nesse código, `readFile`é uma função inventada que lê um arquivo e retorna seu conteúdo como uma sequência. O JavaScript padrão não fornece essa funcionalidade - mas diferentes ambientes JavaScript, como o navegador e o Node.js, fornecem suas próprias maneiras de acessar arquivos. O exemplo apenas finge que `readFile`existe.

Para evitar carregar o mesmo módulo várias vezes, `require`mantém um armazenamento (cache) de módulos já carregados. Quando chamado, ele primeiro verifica se o módulo solicitado foi carregado e, se não, carrega-o. Isso envolve ler o código do módulo, agrupá-lo em uma função e chamá-lo.

A interface do `ordinal`pacote que vimos antes não é um objeto, mas uma função. Uma peculiaridade dos módulos CommonJS é que, embora o sistema de módulos crie um objeto de interface vazio para você (vinculado a `exports`), você pode substituí-lo por qualquer valor substituindo-o `module.exports`. Isso é feito por muitos módulos para exportar um único valor em vez de um objeto de interface.

Ao definir `require`, `exports`e `module`como parâmetros para a função do wrapper gerado (e passar os valores apropriados ao chamá-lo), o carregador garante que essas ligações estejam disponíveis no escopo do módulo.

A maneira como a string atribuída `require`é traduzida para um nome de arquivo ou endereço da Web real difere em diferentes sistemas. Quando começa com `"./"`ou `"../"`, geralmente é interpretado como relativo ao nome do arquivo do módulo atual. Assim seria o arquivo nomeado no mesmo diretório.`"./format-date"``format-date.js`

Quando o nome não é relativo, o Node.js procurará um pacote instalado com esse nome. No código de exemplo deste capítulo, interpretaremos nomes como se referindo a pacotes NPM. Entraremos em mais detalhes sobre como instalar e usar os módulos NPM no [Capítulo 20](https://eloquentjavascript.net/20_node.html) .

Agora, em vez de escrever nosso próprio analisador de arquivos INI, podemos usar um do NPM.

```js
const {parse} = require("ini");

console.log(parse("x = 10\ny = 20"));
// → {x: "10", y: "20"}
```

## Módulos ECMAScript

Os módulos CommonJS funcionam muito bem e, em combinação com o NPM, permitiram à comunidade JavaScript começar a compartilhar código em larga escala.

Mas eles permanecem um pouco como um truque de fita adesiva. A notação é um pouco estranha - as coisas que você adiciona `exports`não estão disponíveis no escopo local, por exemplo. E como `require`uma chamada de função normal está recebendo qualquer tipo de argumento, não apenas um literal de cadeia, pode ser difícil determinar as dependências de um módulo sem executar seu código.

É por isso que o padrão JavaScript de 2015 introduz seu próprio sistema de módulos diferentes. É geralmente chamado de *módulos ES* , onde *ES* significa ECMAScript. Os principais conceitos de dependências e interfaces permanecem os mesmos, mas os detalhes diferem. Por um lado, a notação agora está integrada no idioma. Em vez de chamar uma função para acessar uma dependência, você usa uma `import`palavra-chave especial .

```js
import ordinal from "ordinal";
import {days, months} from "date-names";

export function formatDate(date, format) { /* ... */ }
```

Da mesma forma, a `export`palavra-chave é usada para exportar coisas. Pode aparecer na frente de uma função, classe, ou definição de ligação ( `let`, `const`, ou `var`).

A interface de um módulo ES não é um valor único, mas um conjunto de ligações nomeadas. O módulo anterior se liga `formatDate`a uma função. Quando você importa de outro módulo, importa a *ligação* , e não o valor, o que significa que um módulo de exportação pode alterar o valor da ligação a qualquer momento, e os módulos que a importam verão seu novo valor.

Quando há uma ligação denominada `default`, ela é tratada como o principal valor exportado do módulo. Se você importar um módulo como `ordinal`no exemplo, sem chaves entre o nome da ligação, você obtém sua `default`ligação. Esses módulos ainda podem exportar outras ligações com nomes diferentes ao lado de sua `default`exportação.

Para criar uma exportação padrão, escreva `export default`antes de uma expressão, declaração de função ou declaração de classe.

```js
export default ["Winter", "Spring", "Summer", "Autumn"];
```

É possível renomear ligações importados usando a palavra `as`.

```js
import {days as dayNames} from "date-names";

console.log(dayNames.length);
// → 7
```

Outra diferença importante é que as importações do módulo ES acontecem antes do script de um módulo começar a ser executado. Isso significa que as `import`declarações podem não aparecer dentro de funções ou blocos e os nomes das dependências devem ser cadeias de caracteres entre aspas, não expressões arbitrárias.

No momento da redação deste artigo, a comunidade JavaScript está adotando o estilo deste módulo. Mas tem sido um processo lento. Levou alguns anos, depois que o formato foi especificado, para navegadores e Node.js começarem a suportá-lo. E, apesar de o apoiarem agora, esse suporte ainda tem problemas, e a discussão sobre como esses módulos devem ser distribuídos pelo NPM ainda está em andamento.

Muitos projetos são gravados usando módulos ES e depois convertidos automaticamente para outro formato quando publicados. Estamos em um período de transição no qual dois sistemas de módulos diferentes são usados lado a lado, e é útil poder ler e escrever código em qualquer um deles.

## Construção e agrupamento

De fato, muitos projetos JavaScript não são, tecnicamente, escritos em JavaScript. Existem ramais, como o dialeto de verificação de tipo mencionado no [Capítulo 8](https://eloquentjavascript.net/08_error.html#typing) , que são amplamente utilizados. As pessoas também costumam começar a usar extensões planejadas para o idioma muito antes de serem adicionadas às plataformas que realmente executam JavaScript.

Para tornar isso possível, eles *compilam* seu código, convertendo-o do dialeto JavaScript escolhido para JavaScript antigo simples - ou mesmo para uma versão anterior do JavaScript - para que navegadores antigos possam executá-lo.

A inclusão de um programa modular que consiste em 200 arquivos diferentes em uma página da web produz seus próprios problemas. Se a busca de um único arquivo pela rede leva 50 milissegundos, o carregamento do programa inteiro leva 10 segundos, ou talvez a metade, se você puder carregar vários arquivos simultaneamente. Isso é muito tempo perdido. Como a busca de um único arquivo grande tende a ser mais rápida do que a busca de muitos pequenos, os programadores da Web começaram a usar ferramentas que revertem seus programas (que dividem meticulosamente em módulos) de volta em um único arquivo grande antes de publicá-lo na Web. Essas ferramentas são chamadas de *empacotadores* .

E podemos ir mais longe. Além do número de arquivos, o *tamanho* dos arquivos também determina com que rapidez eles podem ser transferidos pela rede. Assim, a comunidade JavaScript inventou *minifiers* . Essas são ferramentas que usam um programa JavaScript e o tornam menor, removendo automaticamente comentários e espaços em branco, renomeando ligações e substituindo trechos de código por código equivalente que ocupa menos espaço.

Portanto, não é incomum que o código que você encontra em um pacote NPM ou que é executado em uma página da Web tenha passado por *vários* estágios de transformação - convertido do JavaScript moderno para o JavaScript histórico, do formato do módulo ES para o CommonJS, empacotado e minificado . Não entraremos em detalhes dessas ferramentas neste livro, pois elas tendem a ser chatas e mudam rapidamente. Lembre-se de que o código JavaScript que você executa geralmente não é o código como foi escrito.

## Design do módulo

Estruturar programas é um dos aspectos mais sutis da programação. Qualquer parte não trivial da funcionalidade pode ser modelada de várias maneiras.

Um bom design de programa é subjetivo - há trocas envolvidas e questões de gosto. A melhor maneira de aprender o valor do design bem estruturado é ler ou trabalhar em muitos programas e observar o que funciona e o que não funciona. Não assuma que uma bagunça dolorosa é "do jeito que está". Você pode melhorar a estrutura de quase tudo colocando mais atenção nela.

Um aspecto do design do módulo é a facilidade de uso. Se você estiver projetando algo que se destina a ser usado por várias pessoas - ou mesmo por você mesmo, em três meses, quando você não se lembrar mais das especificidades do que fez -, será útil que sua interface seja simples e previsível.

Isso pode significar seguir as convenções existentes. Um bom exemplo é o `ini`pacote. Este módulo imita o `JSON`objeto padrão fornecendo `parse`e `stringify`(para escrever um arquivo INI) funções e, assim `JSON`, converte entre strings e objetos simples. Portanto, a interface é pequena e familiar e, depois de trabalhar com ela uma vez, você provavelmente se lembrará de como usá-la.

Mesmo que não exista nenhuma função padrão ou pacote amplamente usado para imitar, você pode manter seus módulos previsíveis usando estruturas de dados simples e fazendo uma única coisa focada. Muitos dos módulos de análise de arquivos INI no NPM fornecem uma função que lê diretamente esse arquivo do disco rígido e o analisa, por exemplo. Isso impossibilita o uso desses módulos no navegador, onde não temos acesso direto ao sistema de arquivos, e adiciona complexidade que seria melhor resolvida ao *compor* o módulo com alguma função de leitura de arquivos.

Isso aponta para outro aspecto útil do design do módulo - a facilidade com que algo pode ser composto com outro código. Módulos focados que calculam valores são aplicáveis em uma ampla gama de programas do que módulos maiores que executam ações complicadas com efeitos colaterais. Um leitor de arquivo INI que insiste em ler o arquivo do disco é inútil em um cenário em que o conteúdo do arquivo vem de outra fonte.

De maneira semelhante, os objetos com estado são algumas vezes úteis ou até necessários, mas se algo puder ser feito com uma função, use-a. Vários leitores de arquivos INI no NPM fornecem um estilo de interface que exige que você primeiro crie um objeto, carregue o arquivo no objeto e, finalmente, use métodos especializados para obter os resultados. Esse tipo de coisa é comum na tradição orientada a objetos e é terrível. Em vez de fazer uma única chamada de função e seguir em frente, você deve executar o ritual de mover seu objeto por vários estados. E como os dados agora estão agrupados em um tipo de objeto especializado, todo código que interage com ele precisa saber sobre esse tipo, criando interdependências desnecessárias.

Freqüentemente, a definição de novas estruturas de dados não pode ser evitada - apenas algumas básicas são fornecidas pelo padrão da linguagem e muitos tipos de dados precisam ser mais complexos que uma matriz ou um mapa. Mas quando uma matriz é suficiente, use uma matriz.

Um exemplo de uma estrutura de dados um pouco mais complexa é o gráfico do [Capítulo 7](https://eloquentjavascript.net/07_robot.html) . Não existe uma maneira óbvia de representar um gráfico em JavaScript. Nesse capítulo, usamos um objeto cujas propriedades contêm matrizes de strings - os outros nós alcançáveis a partir desse nó.

Existem vários pacotes de busca de caminho diferentes no NPM, mas nenhum deles usa esse formato gráfico. Eles geralmente permitem que as arestas do gráfico tenham um peso, que é o custo ou a distância associada a ele. Isso não é possível em nossa representação.

Por exemplo, há o `dijkstrajs`pacote. Uma abordagem bem conhecida da busca de caminhos, bastante semelhante à nossa `findRoute`função, é chamada *algoritmo de Dijkstra* , em homenagem a Edsger Dijkstra, que a escreveu primeiro. O `js`sufixo é frequentemente adicionado aos nomes dos pacotes para indicar o fato de que eles são gravados em JavaScript. Este `dijkstrajs`pacote usa um formato gráfico semelhante ao nosso, mas, em vez de matrizes, usa objetos cujos valores de propriedade são números - os pesos das arestas.

Portanto, se quiséssemos usar esse pacote, teríamos que garantir que nosso gráfico fosse armazenado no formato esperado. Todas as arestas recebem o mesmo peso, já que nosso modelo simplificado trata cada estrada como tendo o mesmo custo (uma volta).

```js
const {find_path} = require("dijkstrajs");

let graph = {};
for (let node of Object.keys(roadGraph)) {
  let edges = graph[node] = {};
  for (let dest of roadGraph[node]) {
    edges[dest] = 1;
  }
}

console.log(find_path(graph, "Post Office", "Cabin"));
// → ["Post Office", "Alice's House", "Cabin"]
```

Isso pode ser uma barreira para a composição - quando vários pacotes estão usando estruturas de dados diferentes para descrever coisas semelhantes, é difícil combiná-las. Portanto, se você deseja projetar para composição, descubra quais estruturas de dados outras pessoas estão usando e, quando possível, siga o exemplo delas.

## Sumário

Os módulos fornecem estrutura para programas maiores, separando o código em partes com interfaces e dependências claras. A interface é a parte do módulo que é visível a partir de outros módulos, e as dependências são os outros módulos que ele utiliza.

Como o JavaScript historicamente não forneceu um sistema de módulos, o sistema CommonJS foi construído sobre ele. Em seguida, em algum momento ele *se* obter um built-in sistema, que agora convive desconfortavelmente com o sistema commonjs.

Um pacote é um pedaço de código que pode ser distribuído por si próprio. O NPM é um repositório de pacotes JavaScript. Você pode baixar todos os tipos de pacotes úteis (e inúteis) a partir dele.

## Exercícios

### Um robô modular

Estas são as ligações que o projeto do [Capítulo 7](https://eloquentjavascript.net/07_robot.html) cria:

```
estradas
buildGraph
roadGraph
VillageState
runRobot
randomPick
randomRobot
mailRoute
routeRobot
findRoute
goalOrientedRobot
```

Se você escrevesse esse projeto como um programa modular, quais módulos você criaria? Qual módulo dependeria de qual outro módulo e como seriam suas interfaces?

Quais peças provavelmente estarão disponíveis pré-escritas no NPM? Você prefere usar um pacote NPM ou escrevê-lo você mesmo?

Aqui está o que eu teria feito (mas, novamente, não existe uma maneira *correta* de projetar um determinado módulo):

O código usado para criar o gráfico de estrada está no `graph`módulo. Como eu prefiro usar `dijkstrajs`do NPM do que nosso próprio código de localização de caminhos, faremos isso construir o tipo de dados gráficos que se `dijkstrajs`espera. Este módulo exporta uma única função `buildGraph`,. Eu `buildGraph`aceitaria uma matriz de matrizes de dois elementos, em vez de seqüências contendo hífens, para tornar o módulo menos dependente do formato de entrada.

O `roads`módulo contém os dados brutos da estrada (a `roads`matriz) e a `roadGraph`ligação. Este módulo depende `./graph`e exporta o gráfico de estradas.

A `VillageState`turma mora no `state`módulo. Depende do `./roads`módulo, pois precisa ser capaz de verificar se existe uma determinada estrada. Também precisa `randomPick`. Como essa é uma função de três linhas, podemos colocá-lo no `state`módulo como uma função auxiliar interna. Mas `randomRobot`precisa disso também. Portanto, teríamos que duplicá-lo ou colocá-lo em seu próprio módulo. Como essa função existe no NPM no `random-item`pacote, uma boa solução é fazer com que ambos os módulos dependam disso. Também podemos adicionar a `runRobot`função a este módulo, pois ela é pequena e está intimamente relacionada ao gerenciamento de estado. O módulo exporta a `VillageState`classe e a `runRobot`função.

Finalmente, os robôs, juntamente com os valores dos quais eles dependem `mailRoute`, poderiam entrar em um `example-robots`módulo, que depende `./roads`e exporta as funções do robô. Para possibilitar `goalOrientedRobot`a localização de rotas, este módulo também depende `dijkstrajs`.

Ao descarregar algum trabalho para os módulos NPM, o código ficou um pouco menor. Cada módulo individual faz algo bastante simples e pode ser lido por conta própria. A divisão do código em módulos também costuma sugerir melhorias adicionais no design do programa. Nesse caso, parece um pouco estranho que `VillageState`os robôs dependam de um gráfico de estradas específico. Pode ser uma idéia melhor tornar o gráfico um argumento para o construtor do estado e fazer com que os robôs o leiam do objeto state - isso reduz dependências (o que sempre é bom) e possibilita a execução de simulações em mapas diferentes (o que é Melhor).

É uma boa ideia usar módulos NPM para coisas que poderíamos ter escrito por nós mesmos? Em princípio, sim - para coisas não triviais, como a função de busca de caminhos, é provável que você cometa erros e perca tempo escrevendo-os. Para pequenas funções `random-item`, como escrevê-las, é fácil. Mas adicioná-los sempre que você precisar deles tende a confundir seus módulos.

No entanto, você também não deve subestimar o trabalho envolvido em *encontrar* um pacote de NPM apropriado. E mesmo se você encontrar um, ele pode não funcionar bem ou pode estar faltando algum recurso necessário. Além disso, dependendo dos pacotes do NPM significa que você deve garantir que eles estejam instalados, distribuí-los com o seu programa e atualizar periodicamente.

Então, novamente, isso é uma troca, e você pode decidir de qualquer maneira, dependendo de quanto os pacotes o ajudarão.

### Módulo de estradas

Escreva um módulo CommonJS, com base no exemplo do [Capítulo 7](https://eloquentjavascript.net/07_robot.html) , que contém a matriz de estradas e exporta a estrutura de dados do gráfico que os representa como `roadGraph`. Deve depender de um módulo `./graph`, que exporte uma função `buildGraph`que é usada para criar o gráfico. Esta função espera uma matriz de matrizes de dois elementos (os pontos inicial e final das estradas).

```js
// Add dependencies and exports

const roads = [
  "Alice's House-Bob's House",   "Alice's House-Cabin",
  "Alice's House-Post Office",   "Bob's House-Town Hall",
  "Daria's House-Ernie's House", "Daria's House-Town Hall",
  "Ernie's House-Grete's House", "Grete's House-Farm",
  "Grete's House-Shop",          "Marketplace-Farm",
  "Marketplace-Post Office",     "Marketplace-Shop",
  "Marketplace-Town Hall",       "Shop-Town Hall"
];
```

Como este é um módulo CommonJS, é necessário usar `require`para importar o módulo gráfico. Isso foi descrito como exportar uma `buildGraph`função, que você pode selecionar do objeto de interface com uma `const`declaração de desestruturação .

Para exportar `roadGraph`, você adiciona uma propriedade ao `exports`objeto. Como `buildGraph`utiliza uma estrutura de dados que não corresponde exatamente `roads`, a divisão das cadeias de rodovias deve ocorrer no seu módulo.

### Dependências circulares

Uma dependência circular é uma situação em que o módulo A depende de B e B também, direta ou indiretamente, depende de A. Muitos sistemas de módulos simplesmente proíbem isso porque, qualquer que seja a ordem que você escolher para carregar esses módulos, não pode garantir que as dependências de cada módulo tenham foi carregado antes de ser executado.

Os módulos CommonJS permitem uma forma limitada de dependências cíclicas. Contanto que os módulos não substituam seu `exports`objeto padrão e não acessem a interface um do outro até que eles terminem de carregar, as dependências cíclicas estarão bem.

A `require`função fornecida [anteriormente neste capítulo](https://eloquentjavascript.net/10_modules.html#require) suporta esse tipo de ciclo de dependência. Você pode ver como ele lida com os ciclos? O que iria dar errado quando um módulo em um ciclo de *não* substituir o seu padrão `exports`objeto?

O truque é que `require`adiciona módulos ao cache *antes de* começar a carregar o módulo. Dessa forma, se qualquer `require`chamada feita enquanto estiver em execução tentar carregá-la, ela já será conhecida e a interface atual será retornada, em vez de começar a carregar o módulo mais uma vez (o que acabaria sobrecarregando a pilha).

Se um módulo sobrescrever seu `module.exports`valor, qualquer outro módulo que tenha recebido seu valor de interface antes de terminar o carregamento terá se apossado do objeto de interface padrão (que provavelmente está vazio), em vez do valor de interface pretendido.