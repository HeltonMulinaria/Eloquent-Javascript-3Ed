# Capítulo 11 Programação assíncrona

> Quem pode esperar em silêncio enquanto a lama assenta?
> Quem pode ficar parado até o momento da ação?
>
> *—Taoz Te Ching Laozi*

![Imagem de dois corvos em um galho](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/chapter_picture_11.jpg)

A parte central de um computador, a parte que executa as etapas individuais que compõem nossos programas, é chamada de *processador* . Os programas que vimos até agora são coisas que manterão o processador ocupado até que eles terminem seu trabalho. A velocidade na qual algo como um loop que manipula números pode ser executado depende praticamente da velocidade do processador.

Mas muitos programas interagem com coisas fora do processador. Por exemplo, eles podem se comunicar através de uma rede de computadores ou solicitar dados do disco rígido - o que é muito mais lento que obtê-los da memória.

Quando algo assim está acontecendo, seria uma pena deixar o processador ocioso - pode haver algum outro trabalho que ele possa fazer enquanto isso. Em parte, isso é tratado pelo seu sistema operacional, que alternará o processador entre vários programas em execução. Mas isso não ajuda quando queremos que um *único* programa possa progredir enquanto aguarda uma solicitação de rede.

## Assincronia

Em um modelo de programação *síncrona* , as coisas acontecem uma de cada vez. Quando você chama uma função que executa uma ação de longa execução, ela retorna apenas quando a ação é concluída e pode retornar o resultado. Isso interrompe seu programa pelo tempo que a ação leva.

Um modelo *assíncrono* permite que várias coisas aconteçam ao mesmo tempo. Quando você inicia uma ação, seu programa continua em execução. Quando a ação termina, o programa é informado e obtém acesso ao resultado (por exemplo, os dados lidos no disco).

Podemos comparar programação síncrona e assíncrona usando um pequeno exemplo: um programa que busca dois recursos da rede e combina resultados.

Em um ambiente síncrono, em que a função de solicitação retorna somente após o trabalho, a maneira mais fácil de executar esta tarefa é fazer as solicitações uma após a outra. Isso tem a desvantagem de que a segunda solicitação será iniciada apenas quando a primeira terminar. O tempo total gasto será pelo menos a soma dos dois tempos de resposta.

A solução para esse problema, em um sistema síncrono, é iniciar threads de controle adicionais. Um *encadeamento* é outro programa em execução cuja execução pode ser intercalada com outros programas pelo sistema operacional - como a maioria dos computadores modernos contém vários processadores, vários encadeamentos podem ser executados ao mesmo tempo em diferentes processadores. Um segundo encadeamento pode iniciar a segunda solicitação e, em seguida, os dois encadeamentos aguardam o retorno dos resultados, após o que são ressincronizados para combinar seus resultados.

No diagrama a seguir, as linhas grossas representam o tempo que o programa gasta executando normalmente e as linhas finas representam o tempo gasto aguardando a rede. No modelo síncrono, o tempo gasto pela rede faz *parte* da linha do tempo de um determinado encadeamento de controle. No modelo assíncrono, iniciar uma ação de rede conceitualmente causa uma *divisão* na linha do tempo. O programa que iniciou a ação continua em execução e a ação acontece ao lado, notificando o programa quando é concluído.

![Fluxo de controle para programação síncrona e assíncrona](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/control-io.svg)

Outra maneira de descrever a diferença é que a espera pela conclusão das ações está *implícita* no modelo síncrono, enquanto *explícita* , sob nosso controle, no assíncrono.

Assincronismo corta nos dois sentidos. Isso torna os programas de expressão que não se encaixam no modelo de controle linear mais fácil, mas também pode tornar os programas de expressão que seguem uma linha reta mais difíceis. Veremos algumas maneiras de resolver esse constrangimento mais adiante neste capítulo.

As duas importantes plataformas de programação JavaScript - browsers e Node.js - executam operações que podem demorar um pouco de forma assíncrona, em vez de depender de threads. Como a programação com threads é notoriamente difícil (entender o que um programa faz é muito mais difícil quando ele faz várias coisas ao mesmo tempo), isso geralmente é considerado uma coisa boa.

## Crow tech

A maioria das pessoas está ciente do fato de que os corvos são pássaros muito inteligentes. Eles podem usar ferramentas, planejar com antecedência, lembrar de coisas e até comunicar essas coisas entre si.

O que a maioria das pessoas não sabe é que são capazes de muitas coisas que mantêm bem escondidas de nós. Foi-me dito por um especialista respeitável (ainda que um pouco excêntrico) em corvídeos que a tecnologia dos corvos não está muito atrás da tecnologia humana, e eles estão alcançando.

Por exemplo, muitas culturas de corvos têm a capacidade de construir dispositivos de computação. Estes não são eletrônicos, como os dispositivos de computação humana, mas operam através da ação de pequenos insetos, uma espécie intimamente relacionada ao cupim, que desenvolveu uma relação simbiótica com os corvos. Os pássaros lhes fornecem comida e, em troca, os insetos constroem e operam suas complexas colônias que, com a ajuda das criaturas vivas dentro deles, realizam cálculos.

Tais colônias geralmente estão localizadas em grandes ninhos de vida longa. Os pássaros e os insetos trabalham juntos para construir uma rede de estruturas bulbosas de argila, escondidas entre os galhos do ninho, nas quais os insetos vivem e trabalham.

Para se comunicar com outros dispositivos, essas máquinas usam sinais de luz. Os corvos incorporam pedaços de material refletivo em hastes especiais de comunicação, e os insetos têm como objetivo refletir a luz em outro ninho, codificando dados como uma sequência de flashes rápidos. Isso significa que apenas os ninhos que possuem uma conexão visual ininterrupta podem se comunicar.

Nosso amigo, o especialista em corvid, mapeou a rede de ninhos de corvos na vila de Hières-sur-Amby, às margens do rio Ródano. Este mapa mostra os ninhos e suas conexões:

![Uma rede de ninhos de corvos em uma pequena vila](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/Hieres-sur-Amby.png)

Em um exemplo surpreendente de evolução convergente, os computadores crow executam JavaScript. Neste capítulo, escreveremos algumas funções básicas de rede para eles.

## Retornos de chamada

Uma abordagem da programação assíncrona é fazer com que as funções que executam uma ação lenta tenham um argumento extra, uma *função de retorno de chamada* . A ação é iniciada e, quando termina, a função de retorno de chamada é chamada com o resultado.

Como exemplo, a `setTimeout`função, disponível no Node.js e nos navegadores, aguarda um determinado número de milissegundos (um segundo é mil milissegundos) e depois chama uma função.

```js
setTimeout(() => console.log("Tick"), 500);
```

A espera geralmente não é um tipo de trabalho muito importante, mas pode ser útil ao fazer algo como atualizar uma animação ou verificar se algo está demorando mais do que um determinado período de tempo.

Executar várias ações assíncronas em uma linha usando retornos de chamada significa que você deve continuar transmitindo novas funções para lidar com a continuação do cálculo após as ações.

A maioria dos computadores crow nest possui uma lâmpada de armazenamento de dados de longo prazo, onde as informações são gravadas em galhos para que possam ser recuperadas mais tarde. A gravação ou a descoberta de um dado leva um momento; portanto, a interface para armazenamento de longo prazo é assíncrona e usa funções de retorno de chamada.

As lâmpadas de armazenamento armazenam pedaços de dados codificados em JSON em nomes. Um corvo pode armazenar informações sobre os lugares em que está oculta a comida sob o nome `"food caches"`, que pode conter uma variedade de nomes que apontam para outras partes de dados, descrevendo o cache real. Para procurar um cache de comida nas lâmpadas de armazenamento do ninho de *Big Oak* , um corvo poderia executar um código como este:

```jsx
import {bigOak} from "./crow-tech";

bigOak.readStorage("food caches", caches => {
  let firstCache = caches[0];
  bigOak.readStorage(firstCache, info => {
    console.log(info);
  });
});
```

(Todos os nomes e cadeias de caracteres foram traduzidos do idioma crow para o inglês.)

Esse estilo de programação é viável, mas o nível de indentação aumenta a cada ação assíncrona porque você acaba em outra função. Fazer coisas mais complicadas, como executar várias ações ao mesmo tempo, pode ficar um pouco estranho.

Os computadores Crow Nest são criados para se comunicar usando pares de solicitação-resposta. Isso significa que um ninho envia uma mensagem para outro ninho, que imediatamente envia uma mensagem de volta, confirmando o recebimento e possivelmente incluindo uma resposta a uma pergunta feita na mensagem.

Cada mensagem é marcada com um *tipo* , que determina como é tratada. Nosso código pode definir manipuladores para tipos de solicitação específicos e, quando uma solicitação é recebida, o manipulador é chamado para produzir uma resposta.

A interface exportada pelo módulo fornece funções baseadas em retorno de chamada para comunicação. Os ninhos têm um método que envia uma solicitação. Ele espera o nome do ninho de destino, o tipo da solicitação e o conteúdo da solicitação como seus três primeiros argumentos, e espera que uma função seja chamada quando uma resposta aparecer como seu quarto e último argumento.`"./crow-tech"``send`

```js
bigOak.send("Cow Pasture", "note", "Let's caw loudly at 7PM",
            () => console.log("Note delivered."));
```

Mas, para tornar os ninhos capazes de receber essa solicitação, primeiro precisamos definir um tipo de solicitação nomeado `"note"`. O código que lida com as solicitações deve ser executado não apenas neste computador-ninho, mas em todos os ninhos que podem receber mensagens desse tipo. Vamos assumir que um corvo sobrevoa e instala nosso código de manipulador em todos os ninhos.

```js
import {defineRequestType} from "./crow-tech";

defineRequestType("note", (nest, content, source, done) => {
  console.log(`${nest.name} received note: ${content}`);
  done();
});
```

A `defineRequestType`função define um novo tipo de solicitação. O exemplo adiciona suporte para `"note"`solicitações, que apenas envia uma nota para um determinado ninho. Nossa implementação chama `console.log`para que possamos verificar se a solicitação chegou. Os ninhos têm uma `name`propriedade que possui seu nome.

O quarto argumento fornecido ao manipulador,, `done`é uma função de retorno de chamada que ele deve chamar quando terminar a solicitação. Se tivéssemos usado o valor de retorno do manipulador como valor de resposta, isso significaria que um manipulador de solicitações não pode executar ações assíncronas. Uma função que executa trabalho assíncrono geralmente retorna antes que o trabalho seja concluído, tendo organizado para que um retorno de chamada seja chamado quando for concluído. Portanto, precisamos de algum mecanismo assíncrono - nesse caso, outra função de retorno de chamada - para sinalizar quando uma resposta está disponível.

De certa forma, a assincronicidade é *contagiosa* . Qualquer função que chama uma função que funciona de forma assíncrona deve ser ela própria assíncrona, usando um retorno de chamada ou mecanismo semelhante para fornecer seu resultado. Chamar um retorno de chamada é um pouco mais envolvente e propenso a erros do que simplesmente retornar um valor, portanto, a necessidade de estruturar grandes partes do seu programa dessa maneira não é boa.

## Promises

Trabalhar com conceitos abstratos geralmente é mais fácil quando esses conceitos podem ser representados por valores. No caso de ações assíncronas, você pode, em vez de organizar uma função a ser chamada em algum momento no futuro, retornar um objeto que represente esse evento futuro.

É para isso que serve a classe padrão `Promise`. Uma *promessa* é uma ação assíncrona que pode ser concluída em algum momento e produzir um valor. É capaz de notificar qualquer pessoa interessada quando seu valor estiver disponível.

A maneira mais fácil de criar uma promessa é ligar `Promise.resolve`. Essa função garante que o valor que você atribui seja envolvido em uma promessa. Se já é uma promessa, é simplesmente retornada - caso contrário, você recebe uma nova promessa que termina imediatamente com o seu valor como resultado.

```js
let fifteen = Promise.resolve(15);
fifteen.then(value => console.log(`Got ${value}`));
// → Got 15
```

Para obter o resultado de uma promessa, você pode usar seu `then`método. Isso registra uma função de retorno de chamada a ser chamada quando a promessa é resolvida e produz um valor. Você pode adicionar vários retornos de chamada a uma única promessa, e eles serão chamados, mesmo se você os adicionar depois que a promessa já tiver sido *resolvida* (concluída).

Mas isso não é tudo que o `then`método faz. Ele retorna outra promessa, que é resolvida com o valor que a função manipuladora retorna ou, se isso retorna uma promessa, aguarda essa promessa e depois resolve o resultado.

É útil pensar nas promessas como um dispositivo para mover valores para uma realidade assíncrona. Um valor normal está simplesmente lá. Um valor prometido é um valor que já *pode* estar lá ou pode aparecer em algum momento no futuro. Os cálculos definidos em termos de promessas agem sobre esses valores agrupados e são executados de forma assíncrona à medida que os valores se tornam disponíveis.

Para criar uma promessa, você pode usar `Promise`como construtor. Ele tem uma interface um tanto estranha - o construtor espera uma função como argumento, que chama imediatamente, passando uma função que pode ser usada para resolver a promessa. Funciona dessa maneira, em vez de, por exemplo, com um `resolve`método, para que apenas o código que criou a promessa possa resolvê-lo.

É assim que você criaria uma interface baseada em promessa para a `readStorage`função:

```js
function storage(nest, name) {
  return new Promise(resolve => {
    nest.readStorage(name, result => resolve(result));
  });
}

storage(bigOak, "enemies")
  .then(value => console.log("Got", value));
```

Essa função assíncrona retorna um valor significativo. Essa é a principal vantagem das promessas - elas simplificam o uso de funções assíncronas. Em vez de precisar repassar retornos de chamada, as funções baseadas em promessa são semelhantes às regulares: elas recebem a entrada como argumentos e retornam sua saída. A única diferença é que a saída ainda não está disponível.

## Failure

Os cálculos regulares de JavaScript podem falhar ao lançar uma exceção. Cálculos assíncronos geralmente precisam de algo assim. Uma solicitação de rede pode falhar ou algum código que faz parte da computação assíncrona pode gerar uma exceção.

Um dos problemas mais prementes com o estilo de retorno de chamada da programação assíncrona é que torna extremamente difícil garantir que as falhas sejam relatadas adequadamente aos retornos de chamada.

Uma convenção amplamente usada é que o primeiro argumento para o retorno de chamada é usado para indicar que a ação falhou e o segundo contém o valor produzido pela ação quando foi bem-sucedida. Essas funções de retorno de chamada sempre devem verificar se receberam uma exceção e garantir que quaisquer problemas que causem, incluindo exceções geradas por funções que chamam, sejam capturados e atribuídos à função correta.

Promessas tornam isso mais fácil. Eles podem ser resolvidos (a ação foi concluída com êxito) ou rejeitados (falhou). Os manipuladores de resolução (conforme registrados com `then`) são chamados apenas quando a ação é bem-sucedida e as rejeições são propagadas automaticamente para a nova promessa retornada por `then`. E quando um manipulador lança uma exceção, isso automaticamente faz com que a promessa produzida por sua `then`chamada seja rejeitada. Portanto, se algum elemento de uma cadeia de ações assíncronas falhar, o resultado de toda a cadeia será marcado como rejeitado e nenhum manipulador de sucesso será chamado além do ponto em que falhou.

Assim como resolver uma promessa fornece um valor, rejeitar um também fornece um, geralmente chamado de *motivo* da rejeição. Quando uma exceção em uma função de manipulador causa a rejeição, o valor da exceção é usado como o motivo. Da mesma forma, quando um manipulador retorna uma promessa que é rejeitada, essa rejeição flui para a próxima promessa. Existe uma `Promise.reject`função que cria uma nova promessa imediatamente rejeitada.

Para lidar explicitamente com essas rejeições, as promessas têm um `catch`método que registra um manipulador a ser chamado quando a promessa é rejeitada, semelhante à maneira como os `then`manipuladores lidam com a resolução normal. Também é muito parecido `then`com o fato de que ela retorna uma nova promessa, que é resolvida com o valor da promessa original se for resolvida normalmente e com o resultado do `catch`manipulador de outra forma. Se um `catch`manipulador gerar um erro, a nova promessa também será rejeitada.

Como atalho, `then`também aceita um manipulador de rejeição como um segundo argumento, para que você possa instalar os dois tipos de manipuladores em uma única chamada de método.

Uma função passada ao `Promise`construtor recebe um segundo argumento, juntamente com a função de resolução, que pode ser usada para rejeitar a nova promessa.

As cadeias de valores prometidos criadas por chamadas `then`e `catch`podem ser vistas como um pipeline através do qual valores ou falhas assíncronas se movem. Como essas cadeias são criadas pelo registro de manipuladores, cada link tem um manipulador de sucesso ou um manipulador de rejeição (ou ambos) associado a ele. Os manipuladores que não correspondem ao tipo de resultado (êxito ou falha) são ignorados. Mas aqueles que correspondem são chamados, e seu resultado determina que tipo de valor vem a seguir - sucesso quando retorna um valor que não é promissor, rejeição quando gera uma exceção e o resultado de uma promessa quando retorna um deles.

```js
new Promise((_, reject) => reject(new Error("Fail")))
  .then(value => console.log("Handler 1"))
  .catch(reason => {
    console.log("Caught failure " + reason);
    return "nothing";
  })
  .then(value => console.log("Handler 2", value));
// → Caught failure Error: Fail
// → Handler 2 nothing
```

Assim como uma exceção não capturada é manipulada pelo ambiente, os ambientes JavaScript podem detectar quando uma rejeição de promessa não é manipulada e relatam isso como um erro.

## Redes são difíceis

Ocasionalmente, não há luz suficiente para os sistemas de espelho dos corvos transmitirem um sinal ou algo está bloqueando o caminho do sinal. É possível que um sinal seja enviado, mas nunca recebido.

Como é, isso fará com que o retorno de chamada atribuído `send`nunca seja chamado, o que provavelmente fará com que o programa pare sem nem perceber que há um problema. Seria bom se, após um determinado período sem resposta, uma solicitação atingisse o *tempo limite* e informasse a falha.

Freqüentemente, falhas na transmissão são acidentes aleatórios, como o farol de um carro interferindo nos sinais de luz, e simplesmente repetir a solicitação pode causar êxito. Portanto, enquanto estamos nisso, vamos fazer com que nossa função de solicitação tente novamente o envio da solicitação algumas vezes antes que ela desista.

E, como estabelecemos que promessas são boas, também faremos com que nossa função de solicitação retorne uma promessa. Em termos do que eles podem expressar, retornos de chamada e promessas são equivalentes. Funções baseadas em retorno de chamada podem ser agrupadas para expor uma interface baseada em promessa e vice-versa.

Mesmo quando uma solicitação e sua resposta são entregues com êxito, a resposta pode indicar falha - por exemplo, se a solicitação tentar usar um tipo de solicitação que não foi definido ou se o manipulador gerar um erro. Para dar suporte a isso, `send`e `defineRequestType`siga a convenção mencionada anteriormente, onde o primeiro argumento passado para retornos de chamada é o motivo da falha, se houver, e o segundo é o resultado real.

Eles podem ser traduzidos para prometer resolução e rejeição pelo nosso invólucro.

```js
class Timeout extends Error {}

function request(nest, target, type, content) {
  return new Promise((resolve, reject) => {
    let done = false;
    function attempt(n) {
      nest.send(target, type, content, (failed, value) => {
        done = true;
        if (failed) reject(failed);
        else resolve(value);
      });
      setTimeout(() => {
        if (done) return;
        else if (n < 3) attempt(n + 1);
        else reject(new Timeout("Timed out"));
      }, 250);
    }
    attempt(1);
  });
}
```

Como as promessas podem ser resolvidas (ou rejeitadas) apenas uma vez, isso funcionará. A primeira vez `resolve`ou `reject`é chamado determina o resultado da promessa, e outras chamadas causadas por um pedido voltando depois de outro pedido terminado são ignorados.

Para criar um loop assíncrono, para as novas tentativas, precisamos usar uma função recursiva - um loop regular não nos permite parar e aguardar uma ação assíncrona. A `attempt`função faz uma única tentativa de enviar uma solicitação. Ele também define um tempo limite que, se nenhuma resposta retornar após 250 milissegundos, inicia a próxima tentativa ou, se essa foi a terceira tentativa, rejeita a promessa com uma instância `Timeout`como o motivo.

Repetir a cada quarto de segundo e desistir quando não houver resposta após três quartos de segundo é definitivamente um tanto arbitrário. É até possível, se a solicitação foi atendida, mas o manipulador está demorando um pouco mais para que as solicitações sejam entregues várias vezes. Escreveremos nossos manipuladores com esse problema em mente - mensagens duplicadas devem ser inofensivas.

Em geral, hoje não construiremos uma rede robusta e de classe mundial. Mas tudo bem - os corvos ainda não têm grandes expectativas quando se trata de computação.

Para nos isolarmos completamente dos retornos de chamada, iremos adiante e também definiremos um wrapper para `defineRequestType`que a função do manipulador retorne uma promessa ou valor simples e conecte-o até o retorno de chamada para nós.

```js
function requestType(name, handler) {
  defineRequestType(name, (nest, content, source,
                           callback) => {
    try {
      Promise.resolve(handler(nest, content, source))
        .then(response => callback(null, response),
              failure => callback(failure));
    } catch (exception) {
      callback(exception);
    }
  });
}
```

`Promise.resolve`é usado para converter o valor retornado por `handler`uma promessa, se ainda não estiver.

Observe que a chamada `handler`teve que ser agrupada em um `try`bloco para garantir que qualquer exceção que ela gerasse diretamente fosse dada ao retorno de chamada. Isso ilustra bem a dificuldade de lidar adequadamente com erros com retornos de chamada brutos - é fácil esquecer de rotear corretamente exceções como essa e, se você não fizer isso, as falhas não serão relatadas com o retorno de chamada correto. As promessas tornam isso principalmente automático e, portanto, menos propenso a erros.

## Coleções de promises

Cada computador ninho mantém uma variedade de outros ninhos dentro da distância de transmissão em sua `neighbors`propriedade. Para verificar quais delas estão acessíveis no momento, você pode escrever uma função que tente enviar uma `"ping"`solicitação (uma solicitação que simplesmente pede uma resposta) a cada uma delas e ver quais retornam.

Ao trabalhar com coleções de promessas em execução ao mesmo tempo, a `Promise.all`função pode ser útil. Ele retorna uma promessa que aguarda todas as promessas da matriz serem resolvidas e, em seguida, resolve para uma matriz dos valores que essas promessas produziram (na mesma ordem que a matriz original). Se qualquer promessa é rejeitada, o resultado `Promise.all`é ele próprio rejeitado.

```js
requestType("ping", () => "pong");

function availableNeighbors(nest) {
  let requests = nest.neighbors.map(neighbor => {
    return request(nest, neighbor, "ping")
      .then(() => true, () => false);
  });
  return Promise.all(requests).then(result => {
    return nest.neighbors.filter((_, i) => result[i]);
  });
}
```

Quando um vizinho não está disponível, não queremos que toda a promessa combinada falhe desde então, ainda não saberíamos nada. Portanto, a função que é mapeada sobre o conjunto de vizinhos para transformá-los em solicitação promete anexar manipuladores que fazem com êxito solicitações produzidas `true`e rejeitadas `false`.

No manipulador da promessa combinada, `filter`é usado para remover os elementos da `neighbors`matriz cujo valor correspondente é falso. Isto faz uso do facto de que `filter`passa o índice da matriz do elemento de corrente, como um segundo argumento para a sua função de filtragem ( `map`, `some`, e métodos de array de ordem superior semelhantes fazer o mesmo).

## Inundação de rede

O fato de os ninhos poderem conversar apenas com os vizinhos inibe bastante a utilidade dessa rede.

Para transmitir informações para toda a rede, uma solução é configurar um tipo de solicitação que é encaminhada automaticamente aos vizinhos. Esses vizinhos então o encaminham para seus vizinhos, até que toda a rede tenha recebido a mensagem.

```js
import {everywhere} from "./crow-tech";

everywhere(nest => {
  nest.state.gossip = [];
});

function sendGossip(nest, message, exceptFor = null) {
  nest.state.gossip.push(message);
  for (let neighbor of nest.neighbors) {
    if (neighbor == exceptFor) continue;
    request(nest, neighbor, "gossip", message);
  }
}

requestType("gossip", (nest, message, source) => {
  if (nest.state.gossip.includes(message)) return;
  console.log(`${nest.name} received gossip '${
               message}' from ${source}`);
  sendGossip(nest, message, source);
});
```

Para evitar o envio da mesma mensagem pela rede para sempre, cada ninho mantém uma série de sequências de fofocas que ele já viu. Para definir essa matriz, usamos a `everywhere`função - que executa código em todos os ninhos - para adicionar uma propriedade ao `state`objeto do ninho , que é onde manteremos o estado local do ninho.

Quando um ninho recebe uma mensagem de fofoca duplicada, o que é muito provável que ocorra com todo mundo reenviando cegamente, ele a ignora. Mas quando recebe uma nova mensagem, diz entusiasticamente a todos os seus vizinhos, exceto quem a enviou.

Isso fará com que uma nova fofoca se espalhe pela rede como uma mancha de tinta na água. Mesmo quando algumas conexões não estão funcionando no momento, se houver uma rota alternativa para um determinado ninho, as fofocas chegarão até lá.

Esse estilo de comunicação de rede é chamado de *inundação* - inunda a rede com um pedaço de informação até que todos os nós o tenham.

Podemos ligar `sendGossip`para ver um fluxo de mensagens pela vila.

```js
sendGossip(bigOak, "Kids with airgun in the park");
```

## Roteamento de mensagens

Se um determinado nó deseja conversar com um único outro nó, a inundação não é uma abordagem muito eficiente. Especialmente quando a rede é grande, isso levaria a muitas transferências de dados inúteis.

Uma abordagem alternativa é configurar uma maneira de as mensagens saltarem de nó em nó até atingirem seu destino. A dificuldade disso é que requer conhecimento sobre o layout da rede. Para enviar uma solicitação na direção de um ninho distante, é necessário saber qual ninho vizinho o aproxima mais do seu destino. Enviá-lo na direção errada não fará muito bem.

Como cada ninho conhece apenas seus vizinhos diretos, não possui as informações necessárias para calcular uma rota. De alguma forma, devemos espalhar as informações sobre essas conexões para todos os ninhos, de preferência de uma maneira que permita que ela mude com o tempo, quando ninhos são abandonados ou novos ninhos são construídos.

Podemos usar a inundação novamente, mas, em vez de verificar se uma determinada mensagem já foi recebida, agora verificamos se o novo conjunto de vizinhos para um determinado ninho corresponde ao conjunto atual que temos para ele.

```js
requestType("connections", (nest, {name, neighbors},
                            source) => {
  let connections = nest.state.connections;
  if (JSON.stringify(connections.get(name)) ==
      JSON.stringify(neighbors)) return;
  connections.set(name, neighbors);
  broadcastConnections(nest, name, source);
});

function broadcastConnections(nest, name, exceptFor = null) {
  for (let neighbor of nest.neighbors) {
    if (neighbor == exceptFor) continue;
    request(nest, neighbor, "connections", {
      name,
      neighbors: nest.state.connections.get(name)
    });
  }
}

everywhere(nest => {
  nest.state.connections = new Map;
  nest.state.connections.set(nest.name, nest.neighbors);
  broadcastConnections(nest, nest.name);
});
```

A comparação usa `JSON.stringify`porque `==`, em objetos ou matrizes, retornará true somente quando os dois tiverem exatamente o mesmo valor, que não é o que precisamos aqui. Comparar as strings JSON é uma maneira simples, mas eficaz, de comparar seu conteúdo.

Os nós começam imediatamente a transmitir suas conexões, o que deve, a menos que alguns ninhos sejam completamente inacessíveis, fornecer rapidamente a cada ninho um mapa do gráfico de rede atual.

Uma coisa que você pode fazer com os gráficos é encontrar rotas neles, como vimos no [Capítulo 7](https://eloquentjavascript.net/07_robot.html) . Se temos uma rota em direção ao destino de uma mensagem, sabemos em que direção enviá-la.

Essa `findRoute`função, que se assemelha bastante à `findRoute`do [Capítulo 7](https://eloquentjavascript.net/07_robot.html#findRoute) , procura uma maneira de alcançar um determinado nó na rede. Mas, em vez de retornar a rota inteira, apenas retorna o próximo passo. Que no próximo ninho em si vai, usando suas informações atuais sobre a rede, decidir onde *ele* envia a mensagem.

```js
function findRoute(from, to, connections) {
  let work = [{at: from, via: null}];
  for (let i = 0; i < work.length; i++) {
    let {at, via} = work[i];
    for (let next of connections.get(at) || []) {
      if (next == to) return via;
      if (!work.some(w => w.at == next)) {
        work.push({at: next, via: via || next});
      }
    }
  }
  return null;
}
```

Agora podemos criar uma função que pode enviar mensagens de longa distância. Se a mensagem for endereçada a um vizinho direto, ela será entregue como de costume. Caso contrário, ele é empacotado em um objeto e enviado para um vizinho mais próximo do destino, usando o `"route"`tipo de solicitação, o que fará com que esse vizinho repita o mesmo comportamento.

```js
function routeRequest(nest, target, type, content) {
  if (nest.neighbors.includes(target)) {
    return request(nest, target, type, content);
  } else {
    let via = findRoute(nest.name, target,
                        nest.state.connections);
    if (!via) throw new Error(`No route to ${target}`);
    return request(nest, via, "route",
                   {target, type, content});
  }
}

requestType("route", (nest, {target, type, content}) => {
  return routeRequest(nest, target, type, content);
});
```

Agora podemos enviar uma mensagem para o ninho na torre da igreja, que remove quatro saltos de rede.

```js
routeRequest(bigOak, "Church Tower", "note",
             "Incoming jackdaws!");
```

Construímos várias camadas de funcionalidade sobre um sistema de comunicação primitivo para facilitar o uso. Este é um bom modelo (embora simplificado) de como as redes de computadores reais funcionam.

Uma propriedade distintiva das redes de computadores é que elas não são confiáveis - abstrações construídas sobre elas podem ajudar, mas você não pode abstrair a falha na rede. Portanto, a programação de rede normalmente é muito importante para antecipar e lidar com falhas.

## Funções assíncronas

Para armazenar informações importantes, sabe-se que os corvos duplicam entre ninhos. Dessa forma, quando um falcão destrói um ninho, a informação não é perdida.

Para recuperar uma determinada informação que não possui em seu próprio bulbo de armazenamento, um computador ninho pode consultar outros ninhos aleatórios na rede até encontrar um que o possua.

```js
requestType("storage", (nest, name) => storage(nest, name));

function findInStorage(nest, name) {
  return storage(nest, name).then(found => {
    if (found != null) return found;
    else return findInRemoteStorage(nest, name);
  });
}

function network(nest) {
  return Array.from(nest.state.connections.keys());
}

function findInRemoteStorage(nest, name) {
  let sources = network(nest).filter(n => n != nest.name);
  function next() {
    if (sources.length == 0) {
      return Promise.reject(new Error("Not found"));
    } else {
      let source = sources[Math.floor(Math.random() *
                                      sources.length)];
      sources = sources.filter(n => n != source);
      return routeRequest(nest, source, "storage", name)
        .then(value => value != null ? value : next(),
              next);
    }
  }
  return next();
}
```

Porque `connections`é um `Map`, `Object.keys`não funciona nele. Ele tem um `keys` *método* , mas que retorna um iterador em vez de uma matriz. Um iterador (ou valor iterável) pode ser convertido em uma matriz com a `Array.from`função

Mesmo com promessas, esse é um código bastante estranho. Várias ações assíncronas são encadeadas de maneiras não óbvias. Novamente, precisamos de uma função recursiva ( `next`) para modelar o loop pelos ninhos.

E o que o código realmente faz é completamente linear - ele sempre espera que a ação anterior seja concluída antes de iniciar a próxima. Em um modelo de programação síncrona, seria mais fácil expressar.

A boa notícia é que o JavaScript permite escrever código pseudo-síncrono para descrever a computação assíncrona. Uma `async`função é uma função que retorna implicitamente uma promessa e que pode, em seu corpo, `await`outras promessas de uma maneira que *parece* síncrona.

Podemos reescrever `findInStorage`assim:

```js
async function findInStorage(nest, name) {
  let local = await storage(nest, name);
  if (local != null) return local;

  let sources = network(nest).filter(n => n != nest.name);
  while (sources.length > 0) {
    let source = sources[Math.floor(Math.random() *
                                    sources.length)];
    sources = sources.filter(n => n != source);
    try {
      let found = await routeRequest(nest, source, "storage",
                                     name);
      if (found != null) return found;
    } catch (_) {}
  }
  throw new Error("Not found");
}
```

Uma `async`função é marcada pela palavra `async`antes da `function`palavra - chave. Os métodos também podem ser feitos `async`escrevendo `async`antes do nome. Quando essa função ou método é chamado, ele retorna uma promessa. Assim que o corpo retorna algo, essa promessa é resolvida. Se lançar uma exceção, a promessa será rejeitada.

```js
findInStorage(bigOak, "events on 2017-12-21")
  .then(console.log);
```

Dentro de uma `async`função, a palavra `await`pode ser colocada na frente de uma expressão para aguardar a promessa de solução e só então continuar a execução da função.

Essa função não é mais, como uma função JavaScript comum, é executada do início à conclusão de uma só vez. Em vez disso, ele pode ser *congelado* em qualquer ponto que possua um `await`e pode ser retomado posteriormente.

Para código assíncrono não trivial, essa notação geralmente é mais conveniente do que usar diretamente promessas. Mesmo se você precisar fazer algo que não se encaixa no modelo síncrono, como executar várias ações ao mesmo tempo, é fácil combinar `await`com o uso direto de promessas.

## Generators

Essa capacidade das funções serem pausadas e retomadas novamente não é exclusiva das `async`funções. O JavaScript também possui um recurso chamado funções de *gerador* . Estes são semelhantes, mas sem as promessas.

Quando você define uma função com `function*`(colocando um asterisco após a palavra`function` ), ela se torna um gerador. Quando você chama um gerador, ele retorna um iterador, que já vimos no [Capítulo 6](https://eloquentjavascript.net/06_object.html) .

```js
function* powers(n) {
  for (let current = n;; current *= n) {
    yield current;
  }
}

for (let power of powers(3)) {
  if (power > 50) break;
  console.log(power);
}
// → 3
// → 9
// → 27
```

Inicialmente, quando você chama `powers`, a função é congelada no início. Toda vez que você chama `next`o iterador, a função é executada até atingir uma `yield`expressão, que a pausa e faz com que o valor produzido se torne o próximo valor produzido pelo iterador. Quando a função retorna (a que nunca aparece no exemplo), o iterador está pronto.

Escrever iteradores geralmente é muito mais fácil quando você usa funções de gerador. O iterador da `Group`classe (do exercício no [capítulo 6](https://eloquentjavascript.net/06_object.html#group_iterator) ) pode ser escrito com este gerador:

```js
Group.prototype[Symbol.iterator] = function*() {
  for (let i = 0; i < this.members.length; i++) {
    yield this.members[i];
  }
};
```

Não é mais necessário criar um objeto para manter o estado da iteração - os geradores salvam automaticamente seu estado local toda vez que produzem.

Tais `yield`expressões podem ocorrer apenas diretamente na função do gerador em si e não em uma função interna que você define dentro dela. O estado que um gerador salva, ao produzir, é apenas seu ambiente *local* e a posição em que produziu.

Uma `async`função é um tipo especial de gerador. Produz uma promessa quando chamada, que é resolvida quando retorna (termina) e rejeitada quando gera uma exceção. Sempre que produz (aguarda) uma promessa, o resultado dessa promessa (valor ou exceção lançada) é o resultado da `await`expressão.

## O event loop

Programas assíncronos são executados peça por peça. Cada peça pode iniciar algumas ações e programar o código a ser executado quando a ação terminar ou falhar. Entre essas partes, o programa fica ocioso, aguardando a próxima ação.

Portanto, os retornos de chamada não são chamados diretamente pelo código que os agendou. Se eu ligar `setTimeout`de dentro de uma função, essa função retornará quando a função de retorno de chamada for chamada. E quando o retorno de chamada retorna, o controle não retorna à função que o agendou.

O comportamento assíncrono ocorre em sua própria pilha de chamadas de função vazia. Esse é um dos motivos pelos quais, sem promessas, é difícil gerenciar exceções em código assíncrono. Como cada retorno de chamada começa com uma pilha quase vazia, seus `catch`manipuladores não estarão na pilha quando lançarem uma exceção.

```js
try {
  setTimeout(() => {
    throw new Error("Woosh");
  }, 20);
} catch (_) {
  // This will not run
  console.log("Caught!");
}
```

Não importa quão próximos eventos ocorram - como tempos limite ou solicitações de entrada -, um ambiente JavaScript executará apenas um programa por vez. Você pode pensar nisso como um grande loop em *torno do* seu programa, chamado *loop de eventos* . Quando não há nada a ser feito, esse loop é interrompido. Porém, à medida que os eventos chegam, eles são adicionados a uma fila e seu código é executado um após o outro. Como duas coisas não são executadas ao mesmo tempo, o código de execução lenta pode atrasar a manipulação de outros eventos.

Este exemplo define um tempo limite, mas fica suspenso até depois do tempo pretendido, causando atraso no tempo limite.

```js
let start = Date.now();
setTimeout(() => {
  console.log("Timeout ran at", Date.now() - start);
}, 20);
while (Date.now() < start + 50) {}
console.log("Wasted time until", Date.now() - start);
// → Wasted time until 50
// → Timeout ran at 55
```

As promessas sempre são resolvidas ou rejeitadas como um novo evento. Mesmo se uma promessa já tiver sido resolvida, aguardá-la fará com que seu retorno de chamada seja executado após a conclusão do script atual, e não imediatamente.

```js
Promise.resolve("Done").then(console.log);
console.log("Me first!");
// → Me first!
// → Done
```

Nos próximos capítulos, veremos vários outros tipos de eventos que são executados no loop de eventos.

## Erros assíncronos

Quando seu programa é executado de forma síncrona, de uma só vez, não há mudanças de estado, exceto aquelas que o próprio programa faz. Para programas assíncronos, isso é diferente - eles podem ter *lacunas* na execução durante as quais outro código pode ser executado.

Vejamos um exemplo. Um dos hobbies de nossos corvos é contar o número de filhotes que nascem por toda a vila todos os anos. Os ninhos armazenam essa contagem em suas lâmpadas de armazenamento. O código a seguir tenta enumerar as contagens de todos os ninhos para um determinado ano:

```js
function anyStorage(nest, source, name) {
  if (source == nest.name) return storage(nest, name);
  else return routeRequest(nest, source, "storage", name);
}

async function chicks(nest, year) {
  let list = "";
  await Promise.all(network(nest).map(async name => {
    list += `${name}: ${
      await anyStorage(nest, name, `chicks in ${year}`)
    }\n`;
  }));
  return list;
}
```

A `async name =>`parte mostra que as funções das setas também podem ser feitas `async`colocando a palavra `async`na frente delas.

O código não parece suspeito de imediato ... ele mapeia a `async`função de seta sobre o conjunto de ninhos, criando uma série de promessas e, em seguida, usa `Promise.all`para esperar por todas elas antes de retornar a lista que eles constroem.

Mas está seriamente quebrado. Ele sempre retornará apenas uma única linha de saída, listando o ninho que foi mais lento para responder.

```js
chicks(bigOak, 2017).then(console.log);
```

Você pode descobrir por quê?

O problema está no `+=`operador, que pega o valor *atual*`list` no momento em que a instrução começa a ser executada e, quando `await`termina, define a `list`ligação como esse valor mais a sequência adicionada.

Mas entre o momento em que a instrução começa a ser executada e o tempo em que termina, há um espaço assíncrono. A `map`expressão é executada antes de qualquer coisa ter sido adicionada à lista; portanto, cada um dos `+=`operadores inicia a partir de uma sequência vazia e termina, quando sua recuperação de armazenamento termina, definindo `list`uma lista de linha única - o resultado da adição de sua linha à sequência vazia .

Isso poderia ter sido facilmente evitado retornando as linhas das promessas mapeadas e chamando `join`o resultado de `Promise.all`, em vez de criar a lista alterando uma ligação. Como sempre, calcular novos valores é menos propenso a erros do que alterar os valores existentes.

```js
async function chicks(nest, year) {
  let lines = network(nest).map(async name => {
    return name + ": " +
      await anyStorage(nest, name, `chicks in ${year}`);
  });
  return (await Promise.all(lines)).join("\n");
}
```

Erros como esse são fáceis de cometer, principalmente quando se usa `await`, e você deve estar ciente de onde ocorrem as lacunas no seu código. Uma vantagem da assincronicidade *explícita* do JavaScript (seja por meio de retornos de chamada, promessas ou `await`) é que identificar essas lacunas é relativamente fácil.

## Sumário

A programação assíncrona torna possível expressar a espera de ações de longa execução sem congelar o programa durante essas ações. Os ambientes JavaScript geralmente implementam esse estilo de programação usando retornos de chamada, funções chamadas quando as ações são concluídas. Um loop de eventos agenda esses retornos de chamada para serem chamados quando apropriado, um após o outro, para que sua execução não se sobreponha.

A programação de forma assíncrona é facilitada por promessas, objetos que representam ações que podem ser concluídas no futuro e `async`funções, que permitem gravar um programa assíncrono como se fosse síncrono.

## Exercícios

### Rastreando o bisturi

Os corvos da vila possuem um bisturi velho que eles ocasionalmente usam em missões especiais - por exemplo, para cortar portas ou embalagens de tela. Para poder rastreá-lo rapidamente, toda vez que o bisturi é movido para outro ninho, uma entrada é adicionada ao armazenamento do ninho que o possuía e do ninho que o levou, sob o nome `"scalpel"`, com seu novo local como valor.

Isso significa que encontrar o bisturi é uma questão de seguir a trilha de navegação das entradas de armazenamento, até encontrar um ninho onde ele aponte para o próprio ninho.

Escreva uma `async`função `locateScalpel`que faça isso, começando no ninho em que é executado. Você pode usar a `anyStorage`função definida anteriormente para acessar o armazenamento em ninhos arbitrários. O bisturi está em movimento há tempo suficiente para que você possa assumir que todo ninho tem uma `"scalpel"`entrada em seu armazenamento de dados.

Em seguida, escreva a mesma função novamente sem usar `async`e `await`.

As falhas de solicitação são exibidas corretamente como rejeições da promessa retornada nas duas versões? Quão?

```js
async function locateScalpel(nest) {
  // Your code here.
}

function locateScalpel2(nest) {
  // Your code here.
}

locateScalpel(bigOak).then(console.log);
// → Butcher Shop
```

Isso pode ser feito com um único loop que pesquisa nos ninhos, avançando para o próximo quando encontrar um valor que não corresponda ao nome do ninho atual e retornando o nome quando encontrar um valor correspondente. Na `async`função, um regular `for`ou `while`loop pode ser usado.

Para fazer o mesmo em uma função simples, você precisará criar seu loop usando uma função recursiva. A maneira mais fácil de fazer isso é fazer com que a função retorne uma promessa chamando `then`a promessa que recupera o valor do armazenamento. Dependendo se esse valor corresponde ao nome do ninho atual, o manipulador retorna esse valor ou uma promessa adicional criada chamando a função de loop novamente.

Não se esqueça de iniciar o loop chamando a função recursiva uma vez da função principal.

Na `async`função, promessas rejeitadas são convertidas em exceções por `await`. Quando uma `async`função lança uma exceção, sua promessa é rejeitada. Então isso funciona.

Se você implementou a não `async`função, conforme descrito anteriormente, a maneira como `then`funciona também causa automaticamente uma falha no resultado prometido. Se uma solicitação falhar, o manipulador transmitido `then`não será chamado e a promessa retornada será rejeitada pelo mesmo motivo.

### Construindo Promise.all

Dada uma matriz de promessas, `Promise.all`retorna uma promessa que aguarda a conclusão de todas as promessas da matriz. Em seguida, obtém êxito, produzindo uma matriz de valores de resultados. Se uma promessa na matriz falhar, a promessa retornada `all`também falhará, com o motivo da falha da promessa que falhou.

Implemente algo assim como uma função regular chamada `Promise_all`.

Lembre-se de que, depois que uma promessa foi bem-sucedida ou falhou, ela não pode ser bem-sucedida ou falhar novamente, e chamadas adicionais para as funções que a resolvem são ignoradas. Isso pode simplificar a maneira como você lida com o fracasso de sua promessa.

```js
function Promise_all(promises) {
  return new Promise((resolve, reject) => {
    // Your code here.
  });
}

// Test code.
Promise_all([]).then(array => {
  console.log("This should be []:", array);
});
function soon(val) {
  return new Promise(resolve => {
    setTimeout(() => resolve(val), Math.random() * 500);
  });
}
Promise_all([soon(1), soon(2), soon(3)]).then(array => {
  console.log("This should be [1, 2, 3]:", array);
});
Promise_all([soon(1), Promise.reject("X"), soon(3)])
  .then(array => {
    console.log("We should not get here");
  })
  .catch(error => {
    if (error != "X") {
      console.log("Unexpected failure:", error);
    }
  });
```

A função passada ao `Promise`construtor terá que chamar `then`cada uma das promessas na matriz especificada. Quando um deles é bem-sucedido, duas coisas precisam acontecer. O valor resultante precisa ser armazenado na posição correta de uma matriz de resultados, e devemos verificar se essa foi a última promessa pendente e concluir nossa própria promessa, se foi.

O último pode ser feito com um contador que é inicializado com o comprimento da matriz de entrada e do qual subtraímos 1 toda vez que uma promessa é bem-sucedida. Quando chega a 0, terminamos. Certifique-se de levar em consideração a situação em que a matriz de entrada está vazia (e, portanto, nenhuma promessa será resolvida).

Lidar com a falha requer um pouco de reflexão, mas acaba sendo extremamente simples. Apenas passe a `reject`função da promessa de empacotamento para cada uma das promessas da matriz como `catch`manipulador ou como um segundo argumento para `then`que uma falha em um deles desencadeie a rejeição de toda a promessa do empacotador.