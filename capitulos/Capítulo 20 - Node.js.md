# Capítulo 20 Node.js

> Um aluno perguntou: 'Os programadores antigos usavam apenas máquinas simples e sem linguagens de programação, mas faziam programas bonitos. Por que usamos máquinas complicadas e linguagens de programação? '. Fu-Tzu respondeu: 'Os antigos construtores usavam apenas paus e argila, mas faziam cabanas bonitas'.
>
> *—Mestre Yuan-Ma, O Livro de Programação*

![Imagem de um poste de telefone](https://eloquentjavascript.net/img/chapter_picture_20.jpg)

Até agora, usamos a linguagem JavaScript em um único ambiente: o navegador. Este capítulo e o [próximo](https://eloquentjavascript.net/21_skillsharing.html) apresentarão brevemente o Node.js, um programa que permite aplicar suas habilidades de JavaScript fora do navegador. Com ele, você pode criar qualquer coisa, desde pequenas ferramentas de linha de comando até servidores HTTP que alimentam sites dinâmicos.

Esses capítulos têm como objetivo ensinar os principais conceitos usados pelo Node.js. e fornecer informações suficientes para a criação de programas úteis. Eles não tentam ser um tratamento completo ou mesmo completo da plataforma.

Enquanto você pode executar o código nos capítulos anteriores diretamente nessas páginas, porque era JavaScript bruto ou gravado para o navegador, os exemplos de código deste capítulo são escritos para o Node e geralmente não são executados no navegador.

Se você deseja acompanhar e executar o código neste capítulo, será necessário instalar o Node.js. versão 10.1 ou superior. Para fazer isso, acesse [*https://nodejs.org*](https://nodejs.org/) e siga as instruções de instalação do seu sistema operacional. Você também pode encontrar documentação adicional para o Node.js.

## Background

Um dos problemas mais difíceis com os sistemas de gravação que se comunicam pela rede é o gerenciamento de entrada e saída - isto é, a leitura e gravação de dados de e para a rede e o disco rígido. Mover dados leva tempo e agendá-los de maneira inteligente pode fazer uma grande diferença na rapidez com que um sistema responde ao usuário ou às solicitações da rede.

Nesses programas, a programação assíncrona geralmente é útil. Ele permite que o programa envie e receba dados de e para vários dispositivos ao mesmo tempo sem gerenciamento e sincronização de threads complicados.

O nó foi inicialmente concebido com o objetivo de tornar a programação assíncrona fácil e conveniente. O JavaScript se presta bem a um sistema como o Node. É uma das poucas linguagens de programação que não possui uma maneira embutida de fazer entrada e saída. Portanto, o JavaScript pode ser ajustado à abordagem excêntrica do Node para entrada e saída sem terminar com duas interfaces inconsistentes. Em 2009, quando o Node estava sendo projetado, as pessoas já estavam fazendo uma programação baseada em retorno de chamada no navegador, de modo que a comunidade em torno do idioma estava acostumada a um estilo de programação assíncrono.

## Linha de Comando do Node

Quando o Node.js é instalado em um sistema, ele fornece um programa chamado `node`, usado para executar arquivos JavaScript. Digamos que você tenha um arquivo `hello.js`contendo este código:

```js
deixe a  mensagem  =  "Olá mundo" ;
console . log ( mensagem );
```

Você pode executar a `node`partir da linha de comando como esta para executar o programa:

```bash
$ node hello.js
Hello world
```

O `console.log`método no Node faz algo semelhante ao que faz no navegador. Imprime um pedaço de texto. Mas no Node, o texto irá para o fluxo de saída padrão do processo, e não para o console JavaScript do navegador. Ao executar a `node`partir da linha de comando, isso significa que você vê os valores registrados no seu terminal.

Se você executar `node`sem fornecer um arquivo, ele fornecerá um prompt no qual você poderá digitar o código JavaScript e ver imediatamente o resultado.

```bash
$ node
> 1 + 1
2
> [-1, -2, -3].map(Math.abs)
[1, 2, 3]
> process.exit(0)
$
```

A `process`ligação, assim como a `console`ligação, está disponível globalmente no Node. Ele fornece várias maneiras de inspecionar e manipular o programa atual. O `exit`método finaliza o processo e pode receber um código de status de saída, que informa ao programa que iniciou `node`(nesse caso, o shell da linha de comando) se o programa foi concluído com êxito (código zero) ou encontrou um erro (qualquer outro código).

Para encontrar os argumentos de linha de comando fornecidos ao seu script, você pode ler `process.argv`, que é uma matriz de strings. Observe que ele também inclui o nome do `node`comando e o nome do seu script, portanto, os argumentos reais começam no índice 2. Se `showargv.js`contiver a instrução , você poderá executá-la assim:`console.log(process.argv)`

```bash
$ node showargv.js one --and two
["node", "/tmp/showargv.js", "one", "--and", "two"]
```

Todas as ligações padrão JavaScript globais, tais como `Array`, `Math`e `JSON`, também estão presentes no ambiente de Node. A funcionalidade relacionada ao navegador, como `document`ou `prompt`, não é.

## Módulos

Além das ligações que mencionei, como `console`e `process`, o Node coloca poucas ligações adicionais no escopo global. Se você deseja acessar a funcionalidade interna, é necessário solicitar ao sistema do módulo.

O sistema do módulo CommonJS, com base na `require`função, foi descrito no [Capítulo 10](https://eloquentjavascript.net/10_modules.html#commonjs) . Este sistema está embutido no Node e é usado para carregar qualquer coisa, desde módulos internos a pacotes baixados até arquivos que fazem parte do seu próprio programa.

Quando `require`é chamado, o Nó precisa resolver a sequência especificada para um arquivo real que ele pode carregar. Os nomes de caminhos que começam com `/`, `./`ou `../`são resolvidos em relação ao caminho do módulo atual, onde `.`representa o diretório atual, `../`um diretório acima e `/`a raiz do sistema de arquivos. Portanto, se você solicitar o arquivo , o Node tentará carregar o arquivo .`"./graph"``/tmp/robot/robot.js``/tmp/robot/graph.js`

A `.js`extensão pode ser omitida e o Node a adicionará se esse arquivo existir. Se o caminho necessário se referir a um diretório, o Node tentará carregar o arquivo nomeado `index.js`nesse diretório.

Quando uma sequência que não se parece com um caminho relativo ou absoluto é atribuída `require`, supõe-se que se refira a um módulo interno ou a um módulo instalado em um `node_modules`diretório. Por exemplo, `require("fs")`você fornecerá o módulo do sistema de arquivos interno do Node. E `require("robot")`pode tentar carregar a biblioteca encontrada em . Uma maneira comum de instalar essas bibliotecas é usando o NPM, ao qual retornaremos em um momento.`node_modules/robot/`

Vamos configurar um pequeno projeto que consiste em dois arquivos. O primeiro, chamado `main.js`, define um script que pode ser chamado na linha de comando para reverter uma string.

```js
const {reverse} = require("./reverse");

// Index 2 holds the first actual command line argument
let argument = process.argv[2];

console.log(reverse(argument));
```

O arquivo `reverse.js`define uma biblioteca para reverter seqüências de caracteres, que pode ser usada por essa ferramenta de linha de comando e por outros scripts que precisam de acesso direto a uma função de reversão de seqüência de caracteres.

```js
exports.reverse = function(string) {
  return Array.from(string).reverse().join("");
};
```

Lembre-se de que adicionar propriedades para adicioná- `exports`las à interface do módulo. Como o Node.js trata os arquivos como módulos CommonJS, `main.js`pode usar a `reverse`função exportada `reverse.js`.

Agora podemos chamar nossa ferramenta assim:

```bash
$ node main.js JavaScript
tpircSavaJ
```

## Instalando com NPM

O NPM, que foi introduzido no [Capítulo 10](https://eloquentjavascript.net/10_modules.html#modules_npm) , é um repositório online de módulos JavaScript, muitos dos quais foram escritos especificamente para o Node. Ao instalar o Node no seu computador, você também recebe o `npm`comando, que pode ser usado para interagir com este repositório.

O principal uso do NPM é o download de pacotes. Vimos o `ini`pacote no [capítulo 10](https://eloquentjavascript.net/10_modules.html#modules_ini) . Podemos usar o NPM para buscar e instalar esse pacote em nosso computador.

```
$ npm install ini
npm WARN enoent ENOENT: no such file or directory,
         open '/tmp/package.json'
+ ini@1.3.5
added 1 package in 0.552s

$ node
> const {parse} = require("ini");
> parse("x = 1\ny = 2");
{ x: '1', y: '2' }
```

Após a execução `npm install`, o NPM terá criado um diretório chamado `node_modules`. Dentro desse diretório, haverá um `ini`diretório que contém a biblioteca. Você pode abri-lo e ver o código. Quando chamamos `require("ini")`, essa biblioteca é carregada e podemos chamar sua `parse`propriedade para analisar um arquivo de configuração.

Por padrão, o NPM instala pacotes no diretório atual, e não em um local central. Se você está acostumado com outros gerenciadores de pacotes, isso pode parecer incomum, mas possui vantagens - coloca cada aplicativo no controle total dos pacotes que instala e facilita o gerenciamento de versões e a limpeza ao remover um aplicativo.

### Package files

No `npm install`exemplo, você pode ver um aviso sobre o fato de o `package.json`arquivo não existir. É recomendável criar um arquivo para cada projeto, manualmente ou executando `npm init`. Ele contém algumas informações sobre o projeto, como nome e versão, e lista suas dependências.

A simulação de robô do [Capítulo 7](https://eloquentjavascript.net/07_robot.html) , conforme modularizada no exercício do [Capítulo 10](https://eloquentjavascript.net/10_modules.html#modular_robot) , pode ter um `package.json`arquivo como este:

```json
{
  "author": "Marijn Haverbeke",
  "name": "eloquent-javascript-robot",
  "description": "Simulation of a package-delivery robot",
  "version": "1.0.0",
  "main": "run.js",
  "dependencies": {
    "dijkstrajs": "^1.0.1",
    "random-item": "^1.0.0"
  },
  "license": "ISC"
}
```

Quando você executa `npm install`sem nomear um pacote para instalar, o NPM instala as dependências listadas em `package.json`. Quando você instala um pacote específico que ainda não está listado como uma dependência, o NPM o adicionará `package.json`.

### Versões

Um `package.json`arquivo lista a versão e as versões do próprio programa para suas dependências. As versões são uma maneira de lidar com o fato de os pacotes evoluírem separadamente, e o código gravado para funcionar com um pacote como ele existia em um momento, pode não funcionar com uma versão modificada posterior do pacote.

O NPM exige que seus pacotes sigam um esquema chamado *controle de versão semântico* , que codifica algumas informações sobre quais versões são *compatíveis* (não quebre a interface antiga) no número da versão. Uma versão semântica consiste em três números, separados por pontos, como `2.3.0`. Sempre que novas funcionalidades são adicionadas, o número do meio deve ser incrementado. Sempre que a compatibilidade é interrompida, para que o código existente que usa o pacote possa não funcionar com a nova versão, o primeiro número deve ser incrementado.

Um caractere de intercalação ( `^`) na frente do número da versão para uma dependência em `package.json`indica que qualquer versão compatível com o número fornecido pode ser instalada. Portanto, por exemplo, significaria que qualquer versão maior ou igual a 2.3.0 e menor que 3.0.0 é permitida.`"^2.3.0"`

O `npm`comando também é usado para publicar novos pacotes ou novas versões de pacotes. Se você executar `npm publish`em um diretório que possui um `package.json`arquivo, ele publicará um pacote com o nome e a versão listados no arquivo JSON no registro. Qualquer pessoa pode publicar pacotes no NPM - embora apenas com um nome de pacote que ainda não esteja em uso, pois seria um pouco assustador se pessoas aleatórias pudessem atualizar os pacotes existentes.

Como o `npm`programa é um software que se comunica com um sistema aberto - o registro de pacotes - não há nada de exclusivo no que faz. Outro programa, `yarn`que pode ser instalado a partir do registro NPM, desempenha a mesma função que o `npm`uso de uma interface e estratégia de instalação um pouco diferentes.

Este livro não se aprofundará nos detalhes do uso do NPM. Consulte [*https://npmjs.org*](https://npmjs.org/) para obter mais documentação e uma maneira de procurar pacotes.

## O módulo do sistema de arquivos

Um dos módulos internos mais usados no Node é o `fs`módulo, que significa *sistema de arquivos* . Ele exporta funções para trabalhar com arquivos e diretórios.

Por exemplo, a função chamada `readFile`lê um arquivo e, em seguida, chama um retorno de chamada com o conteúdo do arquivo.

```js
let {readFile} = require("fs");
readFile("file.txt", "utf8", (error, text) => {
  if (error) throw error;
  console.log("The file contains:", text);
});
```

O segundo argumento para `readFile`indica a *codificação de caracteres* usada para decodificar o arquivo em uma sequência. Existem várias maneiras pelas quais o texto pode ser codificado para dados binários, mas a maioria dos sistemas modernos usa UTF-8. Portanto, a menos que você tenha motivos para acreditar que outra codificação é usada, passe `"utf8"`ao ler um arquivo de texto. Se você não passar uma codificação, o Node assumirá que você está interessado nos dados binários e fornecerá um `Buffer`objeto em vez de uma string. Este é um objeto semelhante a um array que contém números que representam os bytes (pedaços de dados de 8 bits) nos arquivos.

```js
const {readFile} = require("fs");
readFile("file.txt", (error, buffer) => {
  if (error) throw error;
  console.log("The file contained", buffer.length, "bytes.",
              "The first byte is:", buffer[0]);
});
```

Uma função semelhante,, `writeFile`é usada para gravar um arquivo no disco.

```js
const {writeFile} = require("fs");
writeFile("graffiti.txt", "Node was here", err => {
  if (err) console.log(`Failed to write file: ${err}`);
  else console.log("File written.");
});
```

Aqui não era necessário especificar a codificação `writeFile`- assumirá que, quando receber uma string para escrever, em vez de um `Buffer`objeto, ela deverá ser gravada como texto usando sua codificação de caracteres padrão, que é UTF-8.

O `fs`módulo contém muitas outras funções úteis: `readdir`retornará os arquivos em um diretório como uma matriz de seqüências de caracteres, `stat`recuperará informações sobre um arquivo, `rename`renomeará um arquivo, `unlink`removerá um e assim por diante. Consulte a documentação em [*https://nodejs.org*](https://nodejs.org/) para obter detalhes.

A maioria deles aceita uma função de retorno de chamada como o último parâmetro, que eles chamam com um erro (o primeiro argumento) ou com um resultado bem-sucedido (o segundo). Como vimos no [Capítulo 11](https://eloquentjavascript.net/11_async.html) , há desvantagens nesse estilo de programação - a maior delas é que o tratamento de erros se torna detalhado e propenso a erros.

Embora as promessas façam parte do JavaScript há algum tempo, no momento em que escrevemos, sua integração ao Node.js ainda está em andamento. Há um objeto `promises`exportado do `fs`pacote desde a versão 10.1 que contém a maioria das mesmas funções, `fs`mas usa promessas em vez de funções de retorno de chamada.

```js
const {readFile} = require("fs").promises;
readFile("file.txt", "utf8")
  .then(text => console.log("The file contains:", text));
```

Às vezes você não precisa de assincronismo, e isso apenas atrapalha. Muitas das funções `fs`também possuem uma variante síncrona, que tem o mesmo nome e é `Sync`adicionada ao final. Por exemplo, a versão síncrona de `readFile`é chamada `readFileSync`.

```js
const {readFileSync} = require("fs");
console.log("The file contains:",
            readFileSync("file.txt", "utf8"));
```

Observe que, enquanto uma operação síncrona está sendo executada, seu programa é totalmente parado. Se estiver respondendo ao usuário ou a outras máquinas na rede, ficar preso em uma ação síncrona pode produzir atrasos irritantes.

## O módulo HTTP

Outro módulo central é chamado `http`. Ele fornece funcionalidade para executar servidores HTTP e fazer solicitações HTTP.

Isso é tudo o que é necessário para iniciar um servidor HTTP:

```js
const {createServer} = require("http");
let server = createServer((request, response) => {
  response.writeHead(200, {"Content-Type": "text/html"});
  response.write(`
    <h1>Hello!</h1>
    <p>You asked for <code>${request.url}</code></p>`);
  response.end();
});
server.listen(8000);
console.log("Listening! (port 8000)");
```

Se você executar esse script em sua própria máquina, poderá apontar seu navegador da Web para [*http: // localhost: 8000 / hello*](http://localhost:8000/hello) para fazer uma solicitação ao seu servidor. Ele responderá com uma pequena página HTML.

A função passada como argumento para `createServer`é chamada sempre que um cliente se conecta ao servidor. As ligações `request`e `response`são objetos que representam os dados recebidos e enviados. O primeiro contém informações sobre a solicitação, como sua `url`propriedade, que informa para qual URL a solicitação foi feita.

Portanto, quando você abre essa página no seu navegador, ela envia uma solicitação para o seu próprio computador. Isso faz com que a função do servidor seja executada e devolva uma resposta, que você poderá ver no navegador.

Para enviar algo de volta, você chama métodos no `response`objeto. O primeiro, `writeHead`escreverá os cabeçalhos de resposta (consulte o [Capítulo 18](https://eloquentjavascript.net/18_http.html#headers) ). Você fornece o código de status (200 para "OK" neste caso) e um objeto que contém valores de cabeçalho. O exemplo define o `Content-Type`cabeçalho para informar ao cliente que enviaremos de volta um documento HTML.

Em seguida, o corpo da resposta real (o próprio documento) é enviado com `response.write`. Você pode chamar esse método várias vezes se desejar enviar a resposta peça por peça, por exemplo, para transmitir dados ao cliente assim que estiverem disponíveis. Finalmente, `response.end`sinaliza o fim da resposta.

A chamada para `server.listen`faz com que o servidor comece a aguardar conexões na porta 8000. É por isso que você precisa se conectar ao *localhost: 8000* para falar com esse servidor, em vez de apenas com o *host local* , que usaria a porta padrão 80.

Quando você executa esse script, o processo fica parado e aguarda. Quando um script está ouvindo eventos - nesse caso, conexões de rede - `node`não sai automaticamente quando chega ao final do script. Para fechá-lo, pressione o controle -C.

Um servidor da Web real geralmente faz mais do que o exemplo - examina o método da solicitação (a `method`propriedade) para ver qual ação o cliente está tentando executar e o URL da solicitação para descobrir qual recurso esta ação está sendo executada. em. Veremos um servidor mais avançado [posteriormente neste capítulo](https://eloquentjavascript.net/20_node.html#file_server) .

Para atuar como um *cliente* HTTP , podemos usar a `request`função no `http`módulo

```js
const {request} = require("http");
let requestStream = request({
  hostname: "eloquentjavascript.net",
  path: "/20_node.html",
  method: "GET",
  headers: {Accept: "text/html"}
}, response => {
  console.log("Server responded with status code",
              response.statusCode);
});
requestStream.end();
```

O primeiro argumento a `request`configurar a solicitação, informando ao Node qual servidor conversar, qual caminho solicitar do servidor, qual método usar e assim por diante. O segundo argumento é a função que deve ser chamada quando uma resposta chega. É fornecido um objeto que nos permite inspecionar a resposta, por exemplo, para descobrir seu código de status.

Assim como o `response`objeto que vimos no servidor, o objeto retornado por `request`permite transmitir dados para a solicitação com o `write`método e concluir a solicitação com o `end`método O exemplo não usa `write`porque as `GET`solicitações não devem conter dados em seu corpo.

Há uma `request`função semelhante no `https`módulo que pode ser usada para fazer solicitações para `https:`URLs.

Fazer solicitações com a funcionalidade bruta do Node é bastante detalhado. Existem pacotes de wrappers muito mais convenientes disponíveis no NPM. Por exemplo, `node-fetch`fornece a `fetch`interface baseada em promessa que conhecemos no navegador.

## Streams

Vimos duas instâncias de fluxos graváveis nos exemplos HTTP - ou seja, o objeto de resposta no qual o servidor pôde gravar e o objeto de solicitação que foi retornado `request`.

*Fluxos graváveis* são um conceito amplamente usado no Nó. Esses objetos têm um `write`método que pode ser passado uma string ou um `Buffer`objeto para gravar algo no fluxo. O `end`método deles fecha o fluxo e, opcionalmente, assume um valor para gravar no fluxo antes de fechar. Ambos os métodos também podem receber um retorno de chamada como argumento adicional, que eles chamarão quando a gravação ou o encerramento terminar.

É possível criar um fluxo gravável que aponte para um arquivo com a `createWriteStream`função do `fs`módulo. Em seguida, você pode usar o `write`método no objeto resultante para gravar o arquivo uma peça de cada vez, em vez de em uma tomada como com a imagem `writeFile`.

Fluxos legíveis são um pouco mais envolvidos. A `request`ligação que foi passada ao retorno de chamada do servidor HTTP e a `response`ligação passada ao retorno de chamada do cliente HTTP são fluxos legíveis - um servidor lê solicitações e depois escreve respostas, enquanto um cliente primeiro grava uma solicitação e depois lê uma resposta. A leitura de um fluxo é feita usando manipuladores de eventos, em vez de métodos.

Objetos que emitem eventos no Node têm um método chamado `on`semelhante ao `addEventListener`método no navegador. Você atribui a ele um nome de evento e, em seguida, uma função, e ela registrará essa função a ser chamada sempre que o evento especificado ocorrer.

Fluxos legíveis têm `"data"`e `"end"`eventos. O primeiro é acionado toda vez que os dados entram e o segundo é chamado sempre que o fluxo termina. Este modelo é mais adequado para *streaming de* dados que podem ser processados imediatamente, mesmo quando o documento inteiro ainda não está disponível. Um arquivo pode ser lido como um fluxo legível usando a `createReadStream`função de `fs`.

Este código cria um servidor que lê os corpos da solicitação e os transmite de volta ao cliente como texto em maiúsculas:

```js
const {createServer} = require("http");
createServer((request, response) => {
  response.writeHead(200, {"Content-Type": "text/plain"});
  request.on("data", chunk =>
    response.write(chunk.toString().toUpperCase()));
  request.on("end", () => response.end());
}).listen(8000);
```

O `chunk`valor passado para o manipulador de dados será um binário `Buffer`. Podemos converter isso em uma string decodificando-a como caracteres codificados em UTF-8 com seu `toString`método

O seguinte trecho de código, quando executado com o servidor de maiúsculas ativo, enviará uma solicitação a esse servidor e gravará a resposta recebida:

```js
const {request} = require("http");
request({
  hostname: "localhost",
  port: 8000,
  method: "POST"
}, response => {
  response.on("data", chunk =>
    process.stdout.write(chunk.toString()));
}).end("Hello server");
// → HELLO SERVER
```

O exemplo grava em `process.stdout`(a saída padrão do processo, que é um fluxo gravável) em vez de usar `console.log`. Não podemos usá- `console.log`lo porque ele adiciona um caractere de nova linha extra após cada pedaço de texto que ele escreve, o que não é apropriado aqui, pois a resposta pode aparecer como vários pedaços.

## Um servidor de arquivos

Vamos combinar nosso novo conhecimento sobre servidores HTTP e trabalhar com o sistema de arquivos para criar uma ponte entre os dois: um servidor HTTP que permite acesso remoto a um sistema de arquivos. Esse servidor tem todos os tipos de usos - permite que aplicativos da Web armazenem e compartilhem dados ou pode dar a um grupo de pessoas acesso compartilhado a vários arquivos.

Quando tratamos arquivos como HTTP recursos, os métodos HTTP `GET`, `PUT`e `DELETE`pode ser usado para ler, escrever e apagar os arquivos, respectivamente. Vamos interpretar o caminho na solicitação como o caminho do arquivo ao qual a solicitação se refere.

Provavelmente, não queremos compartilhar todo o nosso sistema de arquivos, portanto, interpretaremos esses caminhos como iniciando no diretório de trabalho do servidor, que é o diretório em que foi iniciado. Se eu executei o servidor no `/tmp/public/`(ou `C:\tmp\public\`no Windows), uma solicitação de `/file.txt`consulta deve se referir a (ou ).`/tmp/public/file.txt``C:\tmp\public\file.txt`

Construiremos o programa peça por peça, usando um objeto chamado `methods`para armazenar as funções que lidam com os vários métodos HTTP. Manipuladores de métodos são `async`funções que obtêm o objeto de solicitação como argumento e retornam uma promessa que é resolvida para um objeto que descreve a resposta.

```js
const {createServer} = require("http");

const methods = Object.create(null);

createServer((request, response) => {
  let handler = methods[request.method] || notAllowed;
  handler(request)
    .catch(error => {
      if (error.status != null) return error;
      return {body: String(error), status: 500};
    })
    .then(({body, status = 200, type = "text/plain"}) => {
       response.writeHead(status, {"Content-Type": type});
       if (body && body.pipe) body.pipe(response);
       else response.end(body);
    });
}).listen(8000);

async function notAllowed(request) {
  return {
    status: 405,
    body: `Method ${request.method} not allowed.`
  };
}
```

Isso inicia um servidor que retorna apenas 405 respostas de erro, que é o código usado para indicar que o servidor se recusa a manipular um determinado método.

Quando a promessa de um manipulador de solicitações é rejeitada, a `catch`chamada converte o erro em um objeto de resposta, se ainda não o for, para que o servidor possa enviar uma resposta de erro para informar ao cliente que falhou ao manipular a solicitação.

O `status`campo da descrição da resposta pode ser omitido; nesse caso, o padrão é 200 (OK). O tipo de conteúdo, na `type`propriedade, também pode ser deixado de lado; nesse caso, a resposta é assumida como texto sem formatação.

Quando o valor de `body`for um fluxo legível, ele terá um `pipe`método usado para encaminhar todo o conteúdo de um fluxo legível para um fluxo gravável. Caso contrário, presume-se que seja `null`(sem corpo), uma sequência ou um buffer e é passado diretamente para o `end`método da resposta .

Para descobrir qual caminho do arquivo corresponde a uma URL de solicitação, a `urlPath`função usa o `url`módulo interno do Node para analisar a URL. Ele pega seu nome de caminho, que será algo como , decodifica para se livrar dos códigos de escape -style e o resolve em relação ao diretório de trabalho do programa.`"/file.txt"``%20`

```js
const {parse} = require("url");
const {resolve, sep} = require("path");

const baseDirectory = process.cwd();

function urlPath(url) {
  let {pathname} = parse(url);
  let path = resolve(decodeURIComponent(pathname).slice(1));
  if (path != baseDirectory &&
      !path.startsWith(baseDirectory + sep)) {
    throw {status: 403, body: "Forbidden"};
  }
  return path;
}
```

Assim que você configura um programa para aceitar solicitações de rede, é necessário começar a se preocupar com a segurança. Nesse caso, se não tomarmos cuidado, é provável que exponhamos acidentalmente todo o sistema de arquivos à rede.

Caminhos de arquivo são cadeias de caracteres no nó. Para mapear essa string para um arquivo real, há uma quantidade não trivial de interpretação em andamento. Os caminhos podem, por exemplo, incluir `../`para se referir a um diretório pai. Portanto, uma fonte óbvia de problemas seria solicitações de caminhos como `/../secret_file`.

Para evitar esses problemas, `urlPath`usa a `resolve`função do `path`módulo, que resolve caminhos relativos. Em seguida, verifica se o resultado está *abaixo* do diretório de trabalho. A `process.cwd`função (onde `cwd`significa "diretório de trabalho atual") pode ser usada para encontrar esse diretório de trabalho. A `sep`ligação do `path`pacote é o separador de caminho do sistema - uma barra invertida no Windows e uma barra invertida na maioria dos outros sistemas. Quando o caminho não inicia com o diretório base, a função lança um objeto de resposta a erro, usando o código de status HTTP indicando que o acesso ao recurso é proibido.

Configuraremos o `GET`método para retornar uma lista de arquivos ao ler um diretório e retornar o conteúdo do arquivo ao ler um arquivo regular.

Uma pergunta complicada é que tipo de `Content-Type`cabeçalho devemos definir ao retornar o conteúdo de um arquivo. Como esses arquivos podem ser qualquer coisa, nosso servidor não pode simplesmente retornar o mesmo tipo de conteúdo para todos eles. O NPM pode nos ajudar novamente aqui. O `mime`pacote (indicadores de tipo de conteúdo como `text/plain`também são chamados de *tipos MIME* ) conhece o tipo correto para um grande número de extensões de arquivo.

O `npm`comando a seguir , no diretório em que reside o script do servidor, instala uma versão específica de `mime`:

```bash
$ npm install mime@2.2.0
```

Quando um arquivo solicitado não existe, o código de status HTTP correto a ser retornado é 404. Usaremos a `stat`função, que pesquisa informações sobre um arquivo, para descobrir se o arquivo existe e se é um diretório.

```js
const {createReadStream} = require("fs");
const {stat, readdir} = require("fs").promises;
const mime = require("mime");

methods.GET = async function(request) {
  let path = urlPath(request.url);
  let stats;
  try {
    stats = await stat(path);
  } catch (error) {
    if (error.code != "ENOENT") throw error;
    else return {status: 404, body: "File not found"};
  }
  if (stats.isDirectory()) {
    return {body: (await readdir(path)).join("\n")};
  } else {
    return {body: createReadStream(path),
            type: mime.getType(path)};
  }
};
```

Porque ele tem que tocar o disco e, portanto, pode demorar um pouco, `stat`é assíncrono. Como estamos usando promessas em vez de estilo de retorno de chamada, ele precisa ser importado de e `promises`não diretamente de `fs`.

Quando o arquivo não existe, `stat`lançará um objeto de erro com uma `code`propriedade de `"ENOENT"`. Esses códigos um tanto obscuros, inspirados no Unix, são como você reconhece os tipos de erro no Node.

O `stats`objeto retornado por `stat`nos informa várias coisas sobre um arquivo, como seu tamanho ( `size`propriedade) e sua data de modificação ( `mtime`propriedade). Aqui, estamos interessados na questão de saber se é um diretório ou um arquivo regular, o que o `isDirectory`método nos diz.

Usamos `readdir`para ler a matriz de arquivos em um diretório e devolvê-la ao cliente. Para arquivos normais, criamos um fluxo legível `createReadStream`e o retornamos como corpo, juntamente com o tipo de conteúdo que o `mime`pacote nos fornece para o nome do arquivo.

O código para lidar com `DELETE`solicitações é um pouco mais simples.

```js
const {rmdir, unlink} = require("fs").promises;

methods.DELETE = async function(request) {
  let path = urlPath(request.url);
  let stats;
  try {
    stats = await stat(path);
  } catch (error) {
    if (error.code != "ENOENT") throw error;
    else return {status: 204};
  }
  if (stats.isDirectory()) await rmdir(path);
  else await unlink(path);
  return {status: 204};
};
```

Quando uma resposta HTTP não contém nenhum dado, o código de status 204 ("sem conteúdo") pode ser usado para indicar isso. Como a resposta à exclusão não precisa transmitir nenhuma informação além do êxito da operação, é uma coisa sensata retornar aqui.

Você pode estar se perguntando por que tentar excluir um arquivo inexistente retorna um código de status de sucesso, em vez de um erro. Quando o arquivo que está sendo excluído não estiver lá, você poderá dizer que o objetivo da solicitação já foi cumprido. O padrão HTTP nos incentiva a fazer solicitações *idempotentes* , o que significa que fazer a mesma solicitação várias vezes produz o mesmo resultado que uma vez. De certa forma, se você tentar excluir algo que já se foi, o efeito que estava tentando fazer foi alcançado - a coisa não está mais lá.

Este é o manipulador de `PUT`solicitações:

```js
const {createWriteStream} = require("fs");

function pipeStream(from, to) {
  return new Promise((resolve, reject) => {
    from.on("error", reject);
    to.on("error", reject);
    to.on("finish", resolve);
    from.pipe(to);
  });
}

methods.PUT = async function(request) {
  let path = urlPath(request.url);
  await pipeStream(request, createWriteStream(path));
  return {status: 204};
};
```

Não precisamos verificar se o arquivo existe dessa vez; se existir, apenas o substituiremos. Novamente, usamos `pipe`para mover dados de um fluxo legível para um gravável, nesse caso, da solicitação para o arquivo. Mas como `pipe`não foi escrito para retornar uma promessa, temos que escrever um invólucro `pipeStream`, que cria uma promessa em torno do resultado da chamada `pipe`.

Quando algo der errado ao abrir o arquivo, `createWriteStream`ainda retornará um fluxo, mas esse fluxo disparará um `"error"`evento. O fluxo de saída para a solicitação também pode falhar, por exemplo, se a rede cair. Então, ligamos os `"error"`eventos dos dois fluxos para rejeitar a promessa. Quando `pipe`terminar, ele fechará o fluxo de saída, o que fará com que ele dispare um `"finish"`evento. Esse é o ponto em que podemos resolver a promessa com sucesso (não retornando nada).

O script completo para o servidor está disponível em [*https://eloquentjavascript.net/code/file_server.js*](https://eloquentjavascript.net/code/file_server.js) . Você pode fazer o download e, depois de instalar suas dependências, executá-lo com o Node para iniciar seu próprio servidor de arquivos. E, é claro, você pode modificá-lo e ampliá-lo para resolver os exercícios deste capítulo ou para experimentar.

A ferramenta de linha de comando `curl`, amplamente disponível em sistemas do tipo Unix (como macOS e Linux), pode ser usada para fazer solicitações HTTP. A sessão a seguir testa brevemente nosso servidor. A `-X`opção é usada para definir o método da solicitação e `-d`para incluir um corpo da solicitação.

```bash
$ curl http://localhost:8000/file.txt
File not found
$ curl -X PUT -d hello http://localhost:8000/file.txt
$ curl http://localhost:8000/file.txt
hello
$ curl -X DELETE http://localhost:8000/file.txt
$ curl http://localhost:8000/file.txt
File not found
```

A primeira solicitação de `file.txt`falha já que o arquivo ainda não existe. A `PUT`solicitação cria o arquivo e, eis que a próxima solicitação o recupera com êxito. Depois de excluí-lo com uma `DELETE`solicitação, o arquivo está novamente ausente.

## Sumário

Node é um sistema pequeno e agradável que nos permite executar o JavaScript em um contexto que não seja um navegador. Foi originalmente projetado para tarefas de rede para desempenhar o papel de um *nó* em uma rede. Mas ele se presta a todos os tipos de tarefas de script e, se você gosta de escrever JavaScript, automatizar tarefas com o Node funciona bem.

O NPM fornece pacotes para tudo o que você pode pensar (e algumas coisas que você provavelmente nunca pensaria), e permite buscar e instalar esses pacotes com o `npm`programa. O nó vem com vários módulos internos, incluindo o `fs`módulo para trabalhar com o sistema de arquivos e o `http`módulo para executar servidores HTTP e fazer solicitações HTTP.

Todas as entradas e saídas no Nó são feitas de forma assíncrona, a menos que você use explicitamente uma variante síncrona de uma função, como `readFileSync`. Ao chamar essas funções assíncronas, você fornece funções de retorno de chamada e o Node as chama com um valor de erro e (se disponível) um resultado quando estiver pronto.

## Exercícios

### Ferramenta de pesquisa

Nos sistemas Unix, existe uma ferramenta de linha de comando chamada `grep`que pode ser usada para procurar rapidamente uma expressão regular nos arquivos.

Escreva um script Node que possa ser executado a partir da linha de comando e atue de maneira semelhante `grep`. Ele trata seu primeiro argumento de linha de comando como uma expressão regular e trata quaisquer outros argumentos como arquivos a serem pesquisados. Ele deve gerar os nomes de qualquer arquivo cujo conteúdo corresponda à expressão regular.

Quando isso funcionar, estenda-o para que, quando um dos argumentos for um diretório, ele pesquise todos os arquivos desse diretório e seus subdiretórios.

Use as funções assíncronas ou síncronas do sistema de arquivos como achar melhor. Configurar as coisas para que várias ações assíncronas sejam solicitadas ao mesmo tempo pode acelerar um pouco, mas não uma quantidade enorme, pois a maioria dos sistemas de arquivos pode ler apenas uma coisa de cada vez.

Seu primeiro argumento de linha de comando, a expressão regular, pode ser encontrado em `process.argv[2]`. Os arquivos de entrada vêm depois disso. Você pode usar o `RegExp`construtor para ir de uma string para um objeto de expressão regular.

Fazer isso de forma síncrona, com `readFileSync`, é mais simples, mas se você usar `fs.promises`novamente para obter funções de retorno de promessa e escrever uma `async`função, o código será semelhante.

Para descobrir se algo é um diretório, você pode novamente usar `stat`(ou `statSync`) e o `isDirectory`método do objeto stats .

Explorar um diretório é um processo de ramificação. Você pode fazer isso usando uma função recursiva ou mantendo uma matriz de trabalho (arquivos que ainda precisam ser explorados). Para encontrar os arquivos em um diretório, você pode chamar `readdir`ou `readdirSync`. A estranha capitalização - a nomeação de funções do sistema de arquivos do Node é baseada livremente nas funções padrão do Unix, como `readdir`todas minúsculas, mas depois é adicionada `Sync`com uma letra maiúscula.

Para passar de um nome de arquivo lido `readdir`para um nome de caminho completo, você deve combiná-lo com o nome do diretório, colocando um caractere de barra ( `/`) entre eles.

### Criação de diretório

Embora o `DELETE`método em nosso servidor de arquivos possa excluir diretórios (usando `rmdir`), o servidor atualmente não fornece nenhuma maneira de *criar* um diretório.

Adicione suporte ao `MKCOL`método ("coleção de criação"), que deve criar um diretório chamando `mkdir`do `fs`módulo. `MKCOL`não é um método HTTP amplamente usado, mas existe para esse mesmo objetivo no padrão *WebDAV* , que especifica um conjunto de convenções sobre o HTTP que o tornam adequado para a criação de documentos.

Você pode usar a função que implementa o `DELETE`método como um blueprint para o `MKCOL`método. Quando nenhum arquivo for encontrado, tente criar um diretório com `mkdir`. Quando um diretório existe nesse caminho, você pode retornar uma resposta 204 para que as solicitações de criação de diretório sejam idempotentes. Se um arquivo não-diretório existir aqui, retorne um código de erro. O código 400 ("solicitação incorreta") seria apropriado.

### Um espaço público na web

Como o servidor de arquivos serve para qualquer tipo de arquivo e até inclui o `Content-Type`cabeçalho certo , você pode usá-lo para servir um site. Como ele permite que todos excluam e substituam arquivos, seria um tipo interessante de site: aquele que pode ser modificado, aprimorado e vandalizado por todos os que dedicam tempo para criar a solicitação HTTP correta.

Escreva uma página HTML básica que inclua um arquivo JavaScript simples. Coloque os arquivos em um diretório atendido pelo servidor de arquivos e abra-os no seu navegador.

Em seguida, como um exercício avançado ou mesmo um projeto de fim de semana, combine todo o conhecimento que você adquiriu com este livro para criar uma interface mais amigável para modificar o site - de *dentro* do site.

Use um formulário HTML para editar o conteúdo dos arquivos que compõem o site, permitindo que o usuário os atualize no servidor usando solicitações HTTP, conforme descrito no [Capítulo 18](https://eloquentjavascript.net/18_http.html) .

Comece tornando apenas um único arquivo editável. Em seguida, faça com que o usuário possa selecionar qual arquivo editar. Use o fato de que nosso servidor de arquivos retorna listas de arquivos ao ler um diretório.

Não trabalhe diretamente no código exposto pelo servidor de arquivos, pois se você cometer um erro, provavelmente os arquivos serão danificados. Em vez disso, mantenha seu trabalho fora do diretório acessível ao público e copie-o lá durante o teste.

Você pode criar um ``elemento para armazenar o conteúdo do arquivo que está sendo editado. Uma `GET`solicitação, usando `fetch`, pode recuperar o conteúdo atual do arquivo. Você pode usar URLs relativos como *index.html* , em vez de [*http: // localhost: 8000 / index.html*](http://localhost:8000/index.html) , para se referir a arquivos no mesmo servidor que o script em execução.

Em seguida, quando o usuário clicar em um botão (você pode usar um ``elemento e um `"submit"`evento), faça uma `PUT`solicitação na mesma URL, com o conteúdo do ``corpo da solicitação, para salvar o arquivo.

Em seguida, você pode adicionar um ``elemento que contenha todos os arquivos no diretório superior do servidor, adicionando ``elementos `GET`ao URL contendo as linhas retornadas por uma solicitação `/`. Quando o usuário seleciona outro arquivo (um `"change"`evento no campo), o script deve buscar e exibir esse arquivo. Ao salvar um arquivo, use o nome do arquivo atualmente selecionado.