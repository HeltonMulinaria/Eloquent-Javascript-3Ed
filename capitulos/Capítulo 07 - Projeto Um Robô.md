# Capítulo 7 Projeto: Um Robô

> [...] a questão de saber se as máquinas podem pensar [...] é tão relevante quanto a questão de saber se os submarinos podem nadar.
>
> *—Edsger Dijkstra, As ameaças à ciência da computação*

![Imagem de um robô de entrega de pacotes](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/chapter_picture_7.jpg)

Nos capítulos de "projeto", pararei de abordar você com uma nova teoria por um breve momento e, em vez disso, trabalharemos juntos em um programa. A teoria é necessária para aprender a programar, mas a leitura e a compreensão dos programas reais são igualmente importantes.

Nosso projeto neste capítulo é construir um autômato, um pequeno programa que executa uma tarefa em um mundo virtual. Nosso autômato será um robô de entrega de correspondência, retirando e entregando encomendas.

## Meadowfield

A vila de Meadowfield não é muito grande. É composto por 11 lugares com 14 estradas entre eles. Pode ser descrito com este conjunto de estradas:

```js
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

![A aldeia de Meadowfield](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/village2x.png)

A rede de estradas da vila forma um *gráfico* . Um gráfico é uma coleção de pontos (lugares da vila) com linhas entre eles (estradas). Este gráfico será o mundo pelo qual nosso robô se moverá.

A matriz de strings não é muito fácil de trabalhar. O que nos interessa são os destinos que podemos alcançar a partir de um determinado local. Vamos converter a lista de estradas em uma estrutura de dados que, para cada local, nos diz o que pode ser alcançado a partir daí.

```js
function buildGraph(edges) {
  let graph = Object.create(null);
  function addEdge(from, to) {
    if (graph[from] == null) {
      graph[from] = [to];
    } else {
      graph[from].push(to);
    }
  }
  for (let [from, to] of edges.map(r => r.split("-"))) {
    addEdge(from, to);
    addEdge(to, from);
  }
  return graph;
}

const roadGraph = buildGraph(roads);
```

Dada uma matriz de arestas, `buildGraph`cria um objeto de mapa que, para cada nó, armazena uma matriz de nós conectados.

Ele usa o `split`método para ir das seqüências de caracteres de estrada, que têm o formato `"Start-End"`, para matrizes de dois elementos contendo o início e o fim como sequências separadas.

## A tarefa

Nosso robô estará se movendo pela vila. Existem pacotes em vários lugares, cada um endereçado a outro local. O robô pega os pacotes quando se trata deles e os entrega quando chega aos seus destinos.

O autômato deve decidir, em cada ponto, para onde ir a seguir. Concluiu sua tarefa quando todas as parcelas foram entregues.

Para poder simular esse processo, precisamos definir um mundo virtual que possa descrevê-lo. Este modelo nos diz onde está o robô e onde estão as parcelas. Quando o robô decidiu se mudar para algum lugar, precisamos atualizar o modelo para refletir a nova situação.

Se você está pensando em termos de programação orientada a objetos, seu primeiro impulso pode ser começar a definir objetos para os vários elementos do mundo: uma classe para o robô, uma para uma parcela, talvez uma para lugares. Eles poderiam conter propriedades que descrevem seu estado atual, como a pilha de parcelas em um local, que poderíamos mudar ao atualizar o mundo.

Isto está errado.

Pelo menos, geralmente é. O fato de algo parecer um objeto não significa automaticamente que ele deva ser um objeto no seu programa. A criação de classes reflexivas para cada conceito em seu aplicativo tende a deixar você com uma coleção de objetos interconectados, cada um com seu próprio estado interno de mudança. Esses programas geralmente são difíceis de entender e, portanto, fáceis de quebrar.

Em vez disso, vamos condensar o estado da vila até o conjunto mínimo de valores que a definem. Há a localização atual do robô e a coleção de parcelas não entregues, cada uma com uma localização atual e um endereço de destino. É isso aí.

E enquanto estamos nisso, vamos fazer com que não *alteremos* esse estado quando o robô se move, mas calculemos um *novo* estado para a situação após a mudança.

```js
class VillageState {
  constructor(place, parcels) {
    this.place = place;
    this.parcels = parcels;
  }

  move(destination) {
    if (!roadGraph[this.place].includes(destination)) {
      return this;
    } else {
      let parcels = this.parcels.map(p => {
        if (p.place != this.place) return p;
        return {place: destination, address: p.address};
      }).filter(p => p.place != p.address);
      return new VillageState(destination, parcels);
    }
  }
}
```

O `move`método é onde a ação acontece. Primeiro, ele verifica se há uma estrada que vai do local atual para o destino e, se não, retorna o estado antigo, pois essa não é uma movimentação válida.

Em seguida, cria um novo estado com o destino como o novo local do robô. Mas também precisa criar um novo conjunto de parcelas - as parcelas que o robô está carregando (que estão no local atual do robô) precisam ser movidas para o novo local. E as parcelas endereçadas ao novo local precisam ser entregues - isto é, precisam ser removidas do conjunto de parcelas não entregues. A chamada para `map`cuida da mudança e a chamada para `filter`entrega.

Os objetos do pacote não são alterados quando são movidos, mas recriados. O `move`método nos dá um novo estado de aldeia, mas deixa o antigo inteiramente intacto.

```js
let first = new VillageState(
  "Post Office",
  [{place: "Post Office", address: "Alice's House"}]
);
let next = first.move("Alice's House");

console.log(next.place);
// → Alice's House
console.log(next.parcels);
// → []
console.log(first.place);
// → Post Office
```

A movimentação faz com que a encomenda seja entregue, e isso é refletido no próximo estado. Mas o estado inicial ainda descreve a situação em que o robô está nos correios e o pacote não é entregue.

## Dados persistentes

As estruturas de dados que não são alteradas são chamadas *imutáveis* ou *persistentes* . Eles se comportam muito como strings e números, pois são quem são e permanecem assim, em vez de conter coisas diferentes em momentos diferentes.

No JavaScript, quase tudo *pode* ser alterado, portanto, trabalhar com valores que deveriam ser persistentes requer alguma restrição. Há uma função chamada `Object.freeze`que altera um objeto para que a gravação em suas propriedades seja ignorada. Você pode usar isso para garantir que seus objetos não sejam alterados, se quiser ter cuidado. O congelamento exige que o computador faça algum trabalho extra, e ter as atualizações ignoradas provavelmente confunde alguém, assim como fazê-lo fazer a coisa errada. Por isso, geralmente prefiro dizer às pessoas que um determinado objeto não deve ser bagunçado e espero que elas se lembrem dele.

```js
let object = Object.freeze({value: 5});
object.value = 10;
console.log(object.value);
// → 5
```

Por que estou me esforçando para não alterar objetos quando a linguagem obviamente está me esperando?

Porque isso me ajuda a entender meus programas. É sobre gerenciamento de complexidade novamente. Quando os objetos no meu sistema são coisas fixas e estáveis, posso considerar as operações isoladas - mudar para a casa de Alice a partir de um determinado estado inicial sempre produz o mesmo novo estado. Quando os objetos mudam com o tempo, isso adiciona uma nova dimensão de complexidade a esse tipo de raciocínio.

Para um sistema pequeno como o que estamos construindo neste capítulo, poderíamos lidar com essa complexidade extra. Mas o limite mais importante para que tipo de sistemas podemos construir é o quanto podemos entender. Qualquer coisa que facilite a compreensão do seu código torna possível a criação de um sistema mais ambicioso.

Infelizmente, embora seja mais fácil entender um sistema construído sobre estruturas de dados persistentes, *projetar* um, especialmente quando sua linguagem de programação não está ajudando, pode ser um pouco mais difícil. Vamos procurar oportunidades para usar estruturas de dados persistentes neste livro, mas também usaremos outras variáveis.

## Simulação

Um robô de entrega olha o mundo e decide em qual direção ele quer se mover. Como tal, poderíamos dizer que um robô é uma função que pega um `VillageState`objeto e retorna o nome de um local próximo.

Como queremos que os robôs sejam capazes de lembrar as coisas, para que possam fazer e executar planos, também passamos a memória deles e permitimos que eles retornem uma nova memória. Assim, a coisa que um robô retorna é um objeto que contém a direção em que deseja se mover e um valor de memória que será devolvido na próxima vez que for chamado.

```js
function runRobot(state, robot, memory) {
  for (let turn = 0;; turn++) {
    if (state.parcels.length == 0) {
      console.log(`Done in ${turn} turns`);
      break;
    }
    let action = robot(state, memory);
    state = state.move(action.direction);
    memory = action.memory;
    console.log(`Moved to ${action.direction}`);
  }
}
```

Considere o que um robô precisa fazer para "resolver" um determinado estado. Ele deve pegar todas as parcelas visitando todos os locais que possuem uma parcela e entregá-las visitando todos os locais para os quais uma parcela é endereçada, mas somente após a retirada.

Qual é a estratégia mais burra que poderia funcionar? O robô podia andar em uma direção aleatória a cada turno. Isso significa que, com grande probabilidade, acabará atingindo todas as parcelas e, em algum momento, também alcançará o local onde devem ser entregues.

Aqui está o que isso poderia parecer:

```js
function randomPick(array) {
  let choice = Math.floor(Math.random() * array.length);
  return array[choice];
}

function randomRobot(state) {
  return {direction: randomPick(roadGraph[state.place])};
}
```

Lembre-se de que `Math.random()`retorna um número entre zero e um - mas sempre abaixo de um. Multiplicar esse número pelo comprimento de uma matriz e depois aplicá `Math.floor`-la nos fornece um índice aleatório para a matriz.

Como esse robô não precisa se lembrar de nada, ignora seu segundo argumento (lembre-se de que funções JavaScript podem ser chamadas com argumentos extras sem efeitos negativos) e omite a `memory`propriedade em seu objeto retornado.

Para colocar esse robô sofisticado em funcionamento, primeiro precisamos de uma maneira de criar um novo estado com algumas parcelas. Um método estático (escrito aqui adicionando diretamente uma propriedade ao construtor) é um bom lugar para colocar essa funcionalidade.

```js
VillageState.random = function(parcelCount = 5) {
  let parcels = [];
  for (let i = 0; i < parcelCount; i++) {
    let address = randomPick(Object.keys(roadGraph));
    let place;
    do {
      place = randomPick(Object.keys(roadGraph));
    } while (place == address);
    parcels.push({place, address});
  }
  return new VillageState("Post Office", parcels);
};
```

Não queremos encomendas enviadas do mesmo local para o qual são endereçadas. Por esse motivo, o `do`loop continua escolhendo novos locais quando obtém um que é igual ao endereço.

Vamos começar um mundo virtual.

```js
runRobot(VillageState.random(), randomRobot);
// → Moved to Marketplace
// → Moved to Town Hall
// → …
// → Done in 63 turns
```

O robô leva muitas voltas para entregar as encomendas porque não está planejando muito bem o futuro. Abordaremos isso em breve.

Para uma perspectiva mais agradável da simulação, você pode usar a `runRobotAnimation`função disponível no [ambiente de programação deste capítulo](https://eloquentjavascript.net/code/#7) . Isso executa a simulação, mas, em vez de produzir texto, mostra o robô se movendo pelo mapa da vila.

```js
runRobotAnimation(VillageState.random(), randomRobot);
```

A maneira como `runRobotAnimation`é implementada continuará sendo um mistério por enquanto, mas depois de ler os [capítulos posteriores](https://eloquentjavascript.net/14_dom.html) deste livro, que discutem a integração do JavaScript em navegadores da web, você poderá adivinhar como funciona.

## A rota do caminhão de correio

Deveríamos ser capazes de fazer muito melhor do que o robô aleatório. Uma melhoria fácil seria dar uma dica do funcionamento da entrega de correio no mundo real. Se encontrarmos uma rota que passe por todos os lugares da vila, o robô poderá percorrer essa rota duas vezes, momento em que é garantido que será feito. Aqui está uma dessas rotas (a partir dos correios):

```js
const mailRoute = [
  "Alice's House", "Cabin", "Alice's House", "Bob's House",
  "Town Hall", "Daria's House", "Ernie's House",
  "Grete's House", "Shop", "Grete's House", "Farm",
  "Marketplace", "Post Office"
];
```

Para implementar o robô que segue a rota, precisamos fazer uso da memória do robô. O robô mantém o restante de sua rota na memória e solta o primeiro elemento a cada turno.

```js
function routeRobot(state, memory) {
  if (memory.length == 0) {
    memory = mailRoute;
  }
  return {direction: memory[0], memory: memory.slice(1)};
}
```

Este robô já é muito mais rápido. São necessárias 26 curvas no máximo (duas vezes a rota de 13 etapas), mas geralmente menos.

```js
runRobotAnimation(VillageState.random(), routeRobot, []);
```

## Encontrando o caminho

Ainda assim, eu realmente não ligaria cegamente seguindo um comportamento inteligente de rota fixa. O robô poderia trabalhar com mais eficiência se ajustasse seu comportamento ao trabalho real que precisa ser feito.

Para fazer isso, ele deve ser capaz de se mover deliberadamente em direção a uma determinada parcela ou em direção ao local onde uma parcela deve ser entregue. Fazer isso, mesmo quando a meta estiver a mais de um passo, exigirá algum tipo de função de busca de rota.

O problema de encontrar uma rota através de um gráfico é um *problema de pesquisa* típico . Podemos dizer se uma determinada solução (uma rota) é uma solução válida, mas não podemos calcular diretamente a solução da maneira que poderíamos para 2 + 2. Em vez disso, precisamos continuar criando soluções em potencial até encontrar uma que funcione.

O número de rotas possíveis através de um gráfico é infinito. Mas na busca de uma rota de *A* para *B* , estamos interessados apenas em aqueles que começam em *A* . Também não nos preocupamos com rotas que visitam o mesmo lugar duas vezes - essas definitivamente não são a rota mais eficiente em qualquer lugar. Portanto, isso reduz o número de rotas que o localizador de rotas deve considerar.

De fato, estamos principalmente interessados no caminho *mais curto* . Então, queremos ter certeza de observar rotas curtas antes de outras mais longas. Uma boa abordagem seria “aumentar” as rotas desde o ponto de partida, explorando todos os lugares acessíveis que ainda não foram visitados, até que uma rota atinja a meta. Dessa forma, exploraremos apenas rotas potencialmente interessantes e encontraremos a rota mais curta (ou uma das rotas mais curtas, se houver mais de uma) até a meta.

Aqui está uma função que faz isso:

```js
function findRoute(graph, from, to) {
  let work = [{at: from, route: []}];
  for (let i = 0; i < work.length; i++) {
    let {at, route} = work[i];
    for (let place of graph[at]) {
      if (place == to) return route.concat(place);
      if (!work.some(w => w.at == place)) {
        work.push({at: place, route: route.concat(place)});
      }
    }
  }
}
```

A exploração deve ser feita na ordem certa - os locais que foram alcançados primeiro devem ser explorados primeiro. Não podemos explorar imediatamente um lugar assim que o alcançamos, porque isso significaria que os lugares alcançados a *partir de lá* também seriam explorados imediatamente, e assim por diante, mesmo que haja outros caminhos mais curtos que ainda não foram explorados.

Portanto, a função mantém uma *lista de trabalho* . Este é um conjunto de lugares que devem ser explorados a seguir, juntamente com a rota que nos levou até lá. Começa apenas com a posição inicial e uma rota vazia.

A pesquisa então opera, pegando o próximo item da lista e explorando-o, o que significa que são examinadas todas as estradas que saem desse local. Se um deles é o objetivo, uma rota finalizada pode ser retornada. Caso contrário, se não tivermos visto esse lugar antes, um novo item será adicionado à lista. Se já analisamos isso antes, já que estamos analisando rotas curtas primeiro, descobrimos uma rota mais longa para esse local ou uma precisamente contanto que a existente, e não precisamos explorá-la.

Você pode visualmente imaginar isso como uma rede de rotas conhecidas que se arrastam para fora do local inicial, crescendo uniformemente por todos os lados (mas nunca se entrelaçando novamente). Assim que o primeiro encadeamento atingir o local do objetivo, ele será rastreado até o início, fornecendo a nossa rota.

Nosso código não lida com a situação em que não há mais itens de trabalho na lista de trabalho porque sabemos que nosso gráfico está *conectado* , o que significa que todos os locais podem ser alcançados em todos os outros locais. Sempre poderemos encontrar uma rota entre dois pontos, e a pesquisa não pode falhar.

```js
function goalOrientedRobot({place, parcels}, route) {
  if (route.length == 0) {
    let parcel = parcels[0];
    if (parcel.place != place) {
      route = findRoute(roadGraph, place, parcel.place);
    } else {
      route = findRoute(roadGraph, place, parcel.address);
    }
  }
  return {direction: route[0], memory: route.slice(1)};
}
```

Este robô usa seu valor de memória como uma lista de direções para se mover, assim como o robô que segue a rota. Sempre que essa lista estiver vazia, ela precisa descobrir o que fazer a seguir. Ele pega o primeiro pacote não entregue no conjunto e, se esse pacote ainda não tiver sido retirado, traça uma rota em direção a ele. Se o pacote *tiver* sido retirado, ele ainda precisará ser entregue; portanto, o robô cria uma rota em direção ao endereço de entrega.

Vamos ver como isso acontece.

```js
runRobotAnimation(VillageState.random(),
                  goalOrientedRobot, []);
```

Esse robô geralmente termina a tarefa de entregar 5 pacotes em cerca de 16 turnos. Isso é um pouco melhor do que `routeRobot`mas ainda definitivamente não é o ideal.

## Exercícios

### Medindo um robô

É difícil comparar objetivamente os robôs, apenas permitindo que eles resolvam alguns cenários. Talvez um robô tenha conseguido tarefas mais fáceis ou o tipo de tarefa em que é bom, enquanto o outro não.

Escreva uma função `compareRobots`que use dois robôs (e a memória inicial). Ele deve gerar 100 tarefas e permitir que cada um dos robôs resolva cada uma dessas tarefas. Quando concluído, deve gerar o número médio de etapas que cada robô executou por tarefa.

Por uma questão de justiça, certifique-se de atribuir cada tarefa aos dois robôs, em vez de gerar tarefas diferentes por robô.

```js
function compareRobots(robot1, memory1, robot2, memory2) {
  // Your code here
}

compareRobots(routeRobot, [], goalOrientedRobot, []);
```

Você precisará escrever uma variante da `runRobot`função que, em vez de registrar os eventos no console, retorne o número de etapas que o robô executou para concluir a tarefa.

Sua função de medição pode, em loop, gerar novos estados e contar as etapas que cada um dos robôs executa. Quando gera medições suficientes, pode ser usada `console.log`para gerar a média de cada robô, que é o número total de etapas executadas dividido pelo número de medições.

### Eficiência do robô

Você pode escrever um robô que conclua a tarefa de entrega mais rapidamente do que `goalOrientedRobot`? Se você observar o comportamento desse robô, que coisas obviamente estúpidas ele faz? Como isso poderia ser melhorado?

Se você resolveu o exercício anterior, convém usar sua `compareRobots`função para verificar se você melhorou o robô.

```js
// Your code here

runRobotAnimation(VillageState.random(), yourRobot, memory);
```



A principal limitação de `goalOrientedRobot`é que ele considera apenas uma parcela de cada vez. Muitas vezes, ele caminha para frente e para trás através da vila, porque a parcela que está olhando está do outro lado do mapa, mesmo se houver outras muito mais próximas.

Uma solução possível seria calcular rotas para todos os pacotes e, em seguida, pegar a mais curta. Resultados ainda melhores podem ser obtidos, se houver várias rotas mais curtas, preferindo as que vão buscar um pacote em vez de entregá-lo.Grupo persistente

A maioria das estruturas de dados fornecidas em um ambiente JavaScript padrão não é muito adequada para uso persistente. Matrizes têm métodos `slice`e `concat`permitem criar facilmente novas matrizes sem danificar a antiga. Mas `Set`, por exemplo, não possui métodos para criar um novo conjunto com um item adicionado ou removido.

Escreva uma nova classe `PGroup`, semelhante à `Group`classe do [Capítulo 6](https://eloquentjavascript.net/06_object.html#groups) , que armazena um conjunto de valores. Como `Group`, ele tem `add`, `delete`e `has`métodos.

Seu `add`método, no entanto, deve retornar uma *nova* `PGroup` instância com o membro especificado adicionado e deixar a antiga inalterada. Da mesma forma, `delete`cria uma nova instância sem um determinado membro.

A classe deve funcionar para valores de qualquer tipo, não apenas para cadeias. Ele *não* tem que ser eficiente quando usado com grandes quantidades de valores.

O construtor não deve fazer parte da interface da classe (embora você definitivamente queira usá-la internamente). Em vez disso, há uma instância vazia `PGroup.empty`, que pode ser usada como um valor inicial.

Por que você precisa de apenas um `PGroup.empty`valor, em vez de ter uma função que cria um novo mapa vazio toda vez?

```js
class PGroup {
  // Your code here
}

let a = PGroup.empty.add("a");
let ab = a.add("b");
let b = ab.delete("a");

console.log(b.has("b"));
// → true
console.log(a.has("b"));
// → false
console.log(b.has("a"));
// → false
```

A maneira mais conveniente de representar o conjunto de valores de membros ainda é uma matriz, pois é fácil copiar as matrizes.

Quando um valor é adicionado ao grupo, você pode criar um novo grupo com uma cópia da matriz original que possui o valor adicionado (por exemplo, usando `concat`). Quando um valor é excluído, você o filtra da matriz.

O construtor da classe pode pegar uma matriz como argumento e armazená-la como propriedade (apenas) da instância. Essa matriz nunca é atualizada.

Para adicionar uma propriedade ( `empty`) a um construtor que não é um método, você deve adicioná-la ao construtor após a definição da classe, como uma propriedade regular.

Você precisa de apenas uma `empty`instância, porque todos os grupos vazios são iguais e as instâncias da classe não mudam. Você pode criar muitos grupos diferentes a partir desse único grupo vazio sem afetá-lo.