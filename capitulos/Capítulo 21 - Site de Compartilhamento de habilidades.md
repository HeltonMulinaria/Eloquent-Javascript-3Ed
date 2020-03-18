# ProjetoCapítulo 21: Site de Compartilhamento de habilidades

> Se você tiver conhecimento, deixe que outras pessoas acendam suas velas.
>
> *—Margaret Fuller*

![Imagem de dois monociclos](https://eloquentjavascript.net/img/chapter_picture_21.jpg)

Uma reunião de *compartilhamento de habilidades* é um evento em que pessoas com interesses compartilhados se reúnem e fazem pequenas apresentações informais sobre o que sabem. Em uma reunião de compartilhamento de habilidades em jardinagem, alguém pode explicar como cultivar aipo. Ou em um grupo de compartilhamento de habilidades de programação, você pode dar uma passada e contar às pessoas sobre o Node.js.

Esses encontros - também chamados *de grupos de usuários* quando se trata de computadores - são uma ótima maneira de ampliar seu horizonte, aprender sobre novos desenvolvimentos ou simplesmente conhecer pessoas com interesses semelhantes. Muitas cidades grandes têm encontros com JavaScript. Eles geralmente são gratuitos, e eu achei os que visitei como amigáveis e acolhedores.

Neste capítulo final do projeto, nosso objetivo é criar um site para gerenciar as palestras realizadas em uma reunião de compartilhamento de habilidades. Imagine um pequeno grupo de pessoas que se encontra regularmente no escritório de um dos membros para conversar sobre andar de bicicleta. O organizador anterior das reuniões mudou-se para outra cidade e ninguém se adiantou para assumir essa tarefa. Queremos um sistema que permita aos participantes propor e discutir conversas entre si, sem um organizador central.

Assim como no [capítulo anterior](https://eloquentjavascript.net/20_node.html) , parte do código deste capítulo foi escrito para o Node.js, e é improvável que funcioná-lo diretamente na página HTML que você está visualizando. O código completo do projeto pode ser baixado em [*https://eloquentjavascript.net/code/skillsharing.zip*](https://eloquentjavascript.net/code/skillsharing.zip) .

## Projeto

Há uma parte do *servidor* nesse projeto, criada para o Node.js, e uma parte do *cliente* , criada para o navegador. O servidor armazena os dados do sistema e os fornece ao cliente. Ele também serve os arquivos que implementam o sistema do lado do cliente.

O servidor mantém a lista de conversas proposta para a próxima reunião e o cliente mostra essa lista. Cada palestra tem um nome de apresentador, um título, um resumo e uma série de comentários associados. O cliente permite que os usuários proponham novas conversas (adicionando-as à lista), excluam conversas e comentem sobre conversas existentes. Sempre que o usuário faz essa alteração, o cliente faz uma solicitação HTTP para informar o servidor sobre isso.

![Captura de tela do site de compartilhamento de habilidades](https://eloquentjavascript.net/img/skillsharing.png)

O aplicativo será configurado para mostrar uma exibição *ao vivo* das atuais conversas propostas e seus comentários. Sempre que alguém, em algum lugar, envia uma nova palestra ou adiciona um comentário, todas as pessoas que têm a página aberta em seus navegadores devem ver imediatamente a alteração. Isso representa um desafio - não há como um servidor Web abrir uma conexão com um cliente, nem há uma boa maneira de saber quais clientes estão visualizando um determinado site no momento.

Uma solução comum para esse problema é chamada de *sondagem longa* , que é uma das motivações para o design do Node.

## Votação longa

Para poder notificar imediatamente um cliente que algo mudou, precisamos de uma conexão com esse cliente. Como os navegadores da Web não aceitam tradicionalmente conexões e os clientes geralmente ficam atrás de roteadores que bloqueariam essas conexões de qualquer maneira, não é prático fazer o servidor iniciar essa conexão.

Podemos providenciar para que o cliente abra a conexão e mantenha-a por perto, para que o servidor possa usá-la para enviar informações quando necessário.

Mas uma solicitação HTTP permite apenas um fluxo simples de informações: o cliente envia uma solicitação, o servidor volta com uma única resposta, e é isso. Existe uma tecnologia chamada *WebSockets* , suportada por navegadores modernos, que possibilita abrir conexões para troca arbitrária de dados. Mas usá-los adequadamente é um pouco complicado.

Neste capítulo, usamos uma técnica mais simples - pesquisa longa - em que os clientes solicitam continuamente novas informações ao servidor usando solicitações HTTP regulares, e o servidor interrompe sua resposta quando não tem nada novo para relatar.

Desde que o cliente verifique constantemente se há uma solicitação de pesquisa aberta, ele receberá informações do servidor rapidamente depois que estiver disponível. Por exemplo, se Fatma tiver nosso aplicativo de compartilhamento de habilidades aberto em seu navegador, ele fará um pedido de atualizações e estará aguardando uma resposta para esse pedido. Quando Iman enviar uma palestra sobre Extreme Downhill Unicycle, o servidor notará que Fatma está aguardando atualizações e enviará uma resposta contendo a nova palestra para sua solicitação pendente. O navegador da Fatma receberá os dados e atualizará a tela para mostrar a palestra.

Para evitar que o tempo limite das conexões seja interrompido (devido à falta de atividade), as longas técnicas de pesquisa geralmente estabelecem um tempo máximo para cada solicitação, após o qual o servidor responderá de qualquer maneira, mesmo que não tenha nada a relatar, após o que o cliente responderá. inicie uma nova solicitação. A reinicialização periódica da solicitação também torna a técnica mais robusta, permitindo que os clientes se recuperem de falhas temporárias de conexão ou problemas no servidor.

Um servidor ocupado que usa sondagem longa pode ter milhares de solicitações em espera e, portanto, conexões TCP, abertas. O nó, que facilita o gerenciamento de muitas conexões sem criar um encadeamento de controle separado para cada um, é um bom ajuste para esse sistema.

## Interface HTTP

Antes de começarmos a projetar o servidor ou o cliente, vamos pensar no ponto em que eles tocam: a interface HTTP sobre a qual eles se comunicam.

Usaremos JSON como o formato do nosso corpo de solicitação e resposta. Como no servidor de arquivos do [Capítulo 20](https://eloquentjavascript.net/20_node.html#file_server) , tentaremos fazer bom uso dos métodos e cabeçalhos HTTP. A interface está centralizada no `/talks`caminho. Os caminhos que não começarem `/talks`serão usados para veicular arquivos estáticos - o código HTML e JavaScript do sistema do lado do cliente.

Uma `GET`solicitação para `/talks`retornar um documento JSON como este:

```http
[{"title": "Unituning",
  "presenter": "Jamal",
  "summary": "Modifying your cycle for extra style",
  "comments": []}]
```

A criação de uma nova conversa é feita fazendo uma `PUT`solicitação para um URL como `/talks/Unituning`, onde a parte após a segunda barra é o título da conversa. O `PUT`corpo da solicitação deve conter um objeto JSON que possui `presenter`e `summary`propriedades.

Como os títulos de conversa podem conter espaços e outros caracteres que podem não aparecer normalmente em uma URL, as cadeias de título devem ser codificadas com a `encodeURIComponent`função ao criar essa URL.

```js
console.log("/talks/" + encodeURIComponent("How to Idle"));
// → /talks/How%20to%20Idle
```

Uma solicitação para criar uma conversa sobre ociosidade pode ser algo como isto:

```http
PUT /talks/How%20to%20Idle HTTP/1.1
Content-Type: application/json
Content-Length: 92

{"presenter": "Maureen",
 "summary": "Standing still on a unicycle"}
```

Essas URLs também suportam `GET`solicitações para recuperar a representação JSON de uma conversa e `DELETE`solicitações para excluir uma conversa.

A adição de um comentário a uma conversa é feita com uma `POST`solicitação para uma URL como , com um corpo JSON que possui e propriedades.`/talks/Unituning/comments``author``message`

```http
POST /talks/Unituning/comments HTTP/1.1
Content-Type: application/json
Content-Length: 72

{"author": "Iman",
 "message": "Will you talk about raising a cycle?"}
```

Para oferecer suporte a pesquisas longas, as `GET`solicitações `/talks`podem incluir cabeçalhos extras que informam o servidor a atrasar a resposta se nenhuma informação nova estiver disponível. Usaremos um par de cabeçalhos normalmente destinados a gerenciar o cache: `ETag`e `If-None-Match`.

Os servidores podem incluir um `ETag`cabeçalho ("tag de entidade") em uma resposta. Seu valor é uma sequência que identifica a versão atual do recurso. Os clientes, quando mais tarde solicitarem esse recurso novamente, poderão fazer uma *solicitação condicional* incluindo um `If-None-Match`cabeçalho cujo valor contenha a mesma string. Se o recurso não tiver sido alterado, o servidor responderá com o código de status 304, que significa "não modificado", informando ao cliente que sua versão em cache ainda está atual. Quando a etiqueta não corresponde, o servidor responde normalmente.

Precisamos de algo assim, em que o cliente possa informar ao servidor qual versão da lista de conversas possui e o servidor responderá somente quando essa lista tiver sido alterada. Mas, em vez de retornar imediatamente uma resposta 304, o servidor deve interromper a resposta e retornar somente quando algo novo estiver disponível ou após um determinado período de tempo. Para distinguir solicitações de pesquisa longa de solicitações condicionais normais, fornecemos outro cabeçalho, `Prefer: wait=90`que informa ao servidor que o cliente está disposto a esperar até 90 segundos pela resposta.

O servidor manterá um número de versão que será atualizado toda vez que as conversações mudarem e o usará como `ETag`valor. Os clientes podem fazer solicitações como essa para serem notificadas quando as negociações mudarem:

```http
GET /talks HTTP/1.1
If-None-Match: "4"
Prefer: wait=90

(time passes)

HTTP/1.1 200 OK
Content-Type: application/json
ETag: "5"
Content-Length: 295

[....]
```

O protocolo descrito aqui não faz nenhum controle de acesso. Todos podem comentar, modificar conversas e até excluí-las. (Como a Internet está cheia de hooligans, colocar esse sistema online sem proteção adicional provavelmente não terminaria bem.)

## O servidor

Vamos começar criando a parte do servidor do programa. O código nesta seção é executado no Node.js.

### Encaminhamento

Nosso servidor usará `createServer`para iniciar um servidor HTTP. Na função que lida com uma nova solicitação, devemos distinguir entre os vários tipos de solicitações (conforme determinado pelo método e o caminho) que suportamos. Isso pode ser feito com uma longa cadeia de `if`declarações, mas existe uma maneira melhor.

Um *roteador* é um componente que ajuda a despachar uma solicitação para a função que pode lidar com isso. Você pode dizer ao roteador, por exemplo, que `PUT`solicitações com um caminho que corresponde à expressão regular ( seguida de um título de conversa) podem ser tratadas por uma determinada função. Além disso, ele pode ajudar a extrair as partes significativas do caminho (neste caso, o título da conversa), entre parênteses na expressão regular, e passá-las para a função manipulador.`/^\/talks\/([^\/]+)$/``/talks/`

Existem vários bons pacotes de roteadores no NPM, mas aqui vamos escrever um para ilustrar o princípio.

Isto é `router.js`, que iremos posteriormente a `require`partir do nosso módulo de servidor:

```js
const {parse} = require("url");

module.exports = class Router {
  constructor() {
    this.routes = [];
  }
  add(method, url, handler) {
    this.routes.push({method, url, handler});
  }
  resolve(context, request) {
    let path = parse(request.url).pathname;

    for (let {method, url, handler} of this.routes) {
      let match = url.exec(path);
      if (!match || request.method != method) continue;
      let urlParts = match.slice(1).map(decodeURIComponent);
      return handler(context, ...urlParts, request);
    }
    return null;
  }
};
```

O módulo exporta a `Router`classe. Um objeto roteador permite que novos manipuladores sejam registrados com o `add`método e pode resolver solicitações com seu `resolve`método.

O último retornará uma resposta quando um manipulador for encontrado, e `null`caso contrário. Ele tenta as rotas uma de cada vez (na ordem em que foram definidas) até encontrar uma correspondente.

As funções do manipulador são chamadas com o `context`valor (que será a instância do servidor no nosso caso), correspondem as strings para qualquer grupo definido na expressão regular e o objeto request. As strings precisam ser decodificadas por URL, pois o URL bruto pode conter `%20`códigos de estilo.

### Servindo arquivos

Quando uma solicitação corresponde a nenhum dos tipos de solicitação definidos em nosso roteador, o servidor deve interpretá-la como uma solicitação de um arquivo no `public`diretório Seria possível usar o servidor de arquivos definido no [Capítulo 20](https://eloquentjavascript.net/20_node.html#file_server) para servir esses arquivos, mas não precisamos nem queremos dar suporte `PUT`e `DELETE`solicitações em arquivos, e gostaríamos de ter recursos avançados, como suporte para armazenamento em cache. Então, vamos usar um servidor de arquivos estático sólido e bem testado do NPM.

Eu optei por `ecstatic`. Este não é o único servidor desse tipo no NPM, mas funciona bem e se ajusta aos nossos propósitos. O `ecstatic`pacote exporta uma função que pode ser chamada com um objeto de configuração para produzir uma função de manipulador de solicitação. Usamos a `root`opção para informar ao servidor onde ele deve procurar arquivos. A função de manipulador aceita `request`e `response`parâmetros e pode ser passado diretamente para `createServer`criar um servidor que serve *apenas* arquivos. Porém, queremos primeiro verificar as solicitações com as quais devemos lidar especialmente, para envolvê-lo em outra função.

```js
const {createServer} = require("http");
const Router = require("./router");
const ecstatic = require("ecstatic");

const router = new Router();
const defaultHeaders = {"Content-Type": "text/plain"};

class SkillShareServer {
  constructor(talks) {
    this.talks = talks;
    this.version = 0;
    this.waiting = [];

    let fileServer = ecstatic({root: "./public"});
    this.server = createServer((request, response) => {
      let resolved = router.resolve(this, request);
      if (resolved) {
        resolved.catch(error => {
          if (error.status != null) return error;
          return {body: String(error), status: 500};
        }).then(({body,
                  status = 200,
                  headers = defaultHeaders}) => {
          response.writeHead(status, headers);
          response.end(body);
        });
      } else {
        fileServer(request, response);
      }
    });
  }
  start(port) {
    this.server.listen(port);
  }
  stop() {
    this.server.close();
  }
}
```

Isso usa uma convenção semelhante ao servidor de arquivos do [capítulo anterior](https://eloquentjavascript.net/20_node.html) para respostas - manipuladores retornam promessas que são resolvidas para objetos que descrevem a resposta. Envolve o servidor em um objeto que também mantém seu estado.

### Fala como recursos

As conversas propostas são armazenadas na `talks`propriedade do servidor, um objeto cujos nomes de propriedade são os títulos das conversas. Eles serão expostos como recursos HTTP `/talks/[title]`, portanto, precisamos adicionar manipuladores ao roteador que implementem os vários métodos que os clientes podem usar para trabalhar com eles.

O manipulador para solicitações de que `GET`uma única conversa deve procurar a conversa e responder com os dados JSON da conversa ou com uma resposta de erro 404.

```js
const talkPath = /^\/talks\/([^\/]+)$/;

router.add("GET", talkPath, async (server, title) => {
  if (title in server.talks) {
    return {body: JSON.stringify(server.talks[title]),
            headers: {"Content-Type": "application/json"}};
  } else {
    return {status: 404, body: `No talk '${title}' found`};
  }
});
```

A exclusão de uma conversa é feita removendo-a do `talks`objeto.

```js
router.add("DELETE", talkPath, async (server, title) => {
  if (title in server.talks) {
    delete server.talks[title];
    server.updated();
  }
  return {status: 204};
});
```

O `updated`método, que definiremos [posteriormente](https://eloquentjavascript.net/21_skillsharing.html#updated) , notifica a espera de longas solicitações de pesquisa sobre a alteração.

Para recuperar o conteúdo de um corpo de solicitação, definimos uma função chamada `readStream`, que lê todo o conteúdo de um fluxo legível e retorna uma promessa que é resolvida em uma string.

```js
function readStream(stream) {
  return new Promise((resolve, reject) => {
    let data = "";
    stream.on("error", reject);
    stream.on("data", chunk => data += chunk.toString());
    stream.on("end", () => resolve(data));
  });
}
```

Um manipulador que precisa ler os corpos de solicitação é o `PUT`manipulador, usado para criar novas conversas. Ele deve verificar se os dados fornecidos possuem `presenter`e `summary`propriedades, que são cadeias de caracteres. Qualquer dado proveniente de fora do sistema pode ser um disparate, e não queremos corromper nosso modelo de dados interno ou travar quando solicitações ruins chegarem.

Se os dados parecerem válidos, o manipulador armazenará um objeto que representa a nova conversa no `talks`objeto, possivelmente substituindo uma conversa existente por esse título e depois chama novamente `updated`.

```js
router.add("PUT", talkPath,
           async (server, title, request) => {
  let requestBody = await readStream(request);
  let talk;
  try { talk = JSON.parse(requestBody); }
  catch (_) { return {status: 400, body: "Invalid JSON"}; }

  if (!talk ||
      typeof talk.presenter != "string" ||
      typeof talk.summary != "string") {
    return {status: 400, body: "Bad talk data"};
  }
  server.talks[title] = {title,
                         presenter: talk.presenter,
                         summary: talk.summary,
                         comments: []};
  server.updated();
  return {status: 204};
});
```

Adicionar um comentário a uma conversa funciona da mesma forma. Usamos `readStream`para obter o conteúdo da solicitação, validar os dados resultantes e armazená-los como um comentário quando parecer válido.

```js
router.add("POST", /^\/talks\/([^\/]+)\/comments$/,
           async (server, title, request) => {
  let requestBody = await readStream(request);
  let comment;
  try { comment = JSON.parse(requestBody); }
  catch (_) { return {status: 400, body: "Invalid JSON"}; }

  if (!comment ||
      typeof comment.author != "string" ||
      typeof comment.message != "string") {
    return {status: 400, body: "Bad comment data"};
  } else if (title in server.talks) {
    server.talks[title].comments.push(comment);
    server.updated();
    return {status: 204};
  } else {
    return {status: 404, body: `No talk '${title}' found`};
  }
});
```

Tentar adicionar um comentário a uma conversa inexistente retorna um erro 404.

### Suporte de pesquisa longo

O aspecto mais interessante do servidor é a parte que lida com pesquisas longas. Quando uma `GET`solicitação é solicitada `/talks`, pode ser uma solicitação regular ou uma solicitação de pesquisa longa.

Haverá vários locais em que precisamos enviar uma série de conversas para o cliente; portanto, primeiro definimos um método auxiliar que construa essa matriz e inclua um `ETag`cabeçalho na resposta.

```js
SkillShareServer.prototype.talkResponse = function() {
  let talks = [];
  for (let title of Object.keys(this.talks)) {
    talks.push(this.talks[title]);
  }
  return {
    body: JSON.stringify(talks),
    headers: {"Content-Type": "application/json",
              "ETag": `"${this.version}"`}
  };
};
```

O manipulador de si precisa de olhar para os cabeçalhos de solicitação para ver se `If-None-Match`e `Prefer`cabeçalhos estão presentes. O nó armazena cabeçalhos, cujos nomes são especificados para não fazer distinção entre maiúsculas e minúsculas, sob seus nomes em minúsculas.

```js
router.add("GET", /^\/talks$/, async (server, request) => {
  let tag = /"(.*)"/.exec(request.headers["if-none-match"]);
  let wait = /\bwait=(\d+)/.exec(request.headers["prefer"]);
  if (!tag || tag[1] != server.version) {
    return server.talkResponse();
  } else if (!wait) {
    return {status: 304};
  } else {
    return server.waitForChanges(Number(wait[1]));
  }
});
```

Se nenhuma etiqueta foi fornecida ou uma etiqueta que não corresponde à versão atual do servidor, o manipulador responde com a lista de conversas. Se a solicitação for condicional e as negociações não tiverem sido alteradas, consultamos o `Prefer`cabeçalho para ver se devemos atrasar a resposta ou responder imediatamente.

As funções de retorno de chamada para solicitações atrasadas são armazenadas na `waiting`matriz do servidor para que possam ser notificadas quando algo acontecer. O `waitForChanges`método também define imediatamente um cronômetro para responder com um status 304 quando a solicitação aguarda o tempo suficiente.

```js
SkillShareServer.prototype.waitForChanges = function(time) {
  return new Promise(resolve => {
    this.waiting.push(resolve);
    setTimeout(() => {
      if (!this.waiting.includes(resolve)) return;
      this.waiting = this.waiting.filter(r => r != resolve);
      resolve({status: 304});
    }, time * 1000);
  });
};
```

Registrar uma alteração com `updated`aumenta a `version`propriedade e ativa todas as solicitações de espera.

```js
SkillShareServer.prototype.updated = function() {
  this.version++;
  let response = this.talkResponse();
  this.waiting.forEach(resolve => resolve(response));
  this.waiting = [];
};
```

Isso conclui o código do servidor. Se criarmos uma instância `SkillShareServer`e a iniciarmos na porta 8000, o servidor HTTP resultante exibirá arquivos do `public`subdiretório ao lado de uma interface de gerenciamento de conversação na `/talks`URL.

```js
new SkillShareServer(Object.create(null)).start(8000);
```

## O cliente

A parte do lado do cliente do site de compartilhamento de habilidades consiste em três arquivos: uma pequena página HTML, uma folha de estilos e um arquivo JavaScript.

### HTML

É uma convenção amplamente usada para servidores da Web tentar servir um arquivo nomeado `index.html`quando uma solicitação é feita diretamente para um caminho que corresponde a um diretório. O módulo do servidor de arquivos que usamos `ecstatic`, suporta esta convenção. Quando uma solicitação é feita no caminho `/`, o servidor procura o arquivo ( sendo a raiz que fornecemos) e retorna esse arquivo, se encontrado.`./public/index.html``./public`

Portanto, se queremos que uma página apareça quando um navegador é apontado para o nosso servidor, devemos colocá-la . Este é o nosso arquivo de índice:`public/index.html`

```js
<!doctype html>
<meta charset="utf-8">
<title>Skill Sharing</title>
<link rel="stylesheet" href="skillsharing.css">

<h1>Skill Sharing</h1>

<script src="skillsharing_client.js"></script>
```

Ele define o título do documento e inclui uma folha de estilos, que define alguns estilos para, entre outras coisas, garantir que haja algum espaço entre as conversas.

Na parte inferior, ele adiciona um cabeçalho na parte superior da página e carrega o script que contém o aplicativo do lado do cliente.

### Ações

O estado do aplicativo consiste na lista de conversas e no nome do usuário, e nós o armazenaremos em um `{talks, user}`objeto. Não permitimos que a interface do usuário manipule diretamente o estado ou envie solicitações HTTP. Em vez disso, pode emitir *ações* que descrevem o que o usuário está tentando fazer.

A `handleAction`função executa tal ação e a faz acontecer. Como nossas atualizações de estado são muito simples, as alterações de estado são tratadas na mesma função.

```js
function handleAction(state, action) {
  if (action.type == "setUser") {
    localStorage.setItem("userName", action.user);
    return Object.assign({}, state, {user: action.user});
  } else if (action.type == "setTalks") {
    return Object.assign({}, state, {talks: action.talks});
  } else if (action.type == "newTalk") {
    fetchOK(talkURL(action.title), {
      method: "PUT",
      headers: {"Content-Type": "application/json"},
      body: JSON.stringify({
        presenter: state.user,
        summary: action.summary
      })
    }).catch(reportError);
  } else if (action.type == "deleteTalk") {
    fetchOK(talkURL(action.talk), {method: "DELETE"})
      .catch(reportError);
  } else if (action.type == "newComment") {
    fetchOK(talkURL(action.talk) + "/comments", {
      method: "POST",
      headers: {"Content-Type": "application/json"},
      body: JSON.stringify({
        author: state.user,
        message: action.message
      })
    }).catch(reportError);
  }
  return state;
}
```

Armazenaremos o nome do usuário `localStorage`para que ele possa ser restaurado quando a página for carregada.

As ações que precisam envolver o servidor fazem solicitações de rede, usando `fetch`a interface HTTP descrita anteriormente. Usamos uma função de wrapper `fetchOK`, que garante que a promessa retornada seja rejeitada quando o servidor retornar um código de erro.

```js
function fetchOK(url, options) {
  return fetch(url, options).then(response => {
    if (response.status < 400) return response;
    else throw new Error(response.statusText);
  });
}
```

Essa função auxiliar é usada para criar um URL para uma conversa com um determinado título.

```js
function talkURL(title) {
  return "talks/" + encodeURIComponent(title);
}
```

Quando a solicitação falha, não queremos que nossa página fique parada, sem fazer nada sem explicação. Então, definimos uma função chamada `reportError`, que pelo menos mostra ao usuário uma caixa de diálogo informando que algo deu errado.

```js
function reportError(error) {
  alert(String(error));
}
```

### Componentes de renderização

Usaremos uma abordagem semelhante à que vimos no [Capítulo 19](https://eloquentjavascript.net/19_paint.html) , dividindo o aplicativo em componentes. Mas como alguns componentes nunca precisam ser atualizados ou sempre são totalmente redesenhados quando atualizados, os definiremos não como classes, mas como funções que retornam diretamente um nó DOM. Por exemplo, aqui está um componente que mostra o campo em que o usuário pode inserir seu nome:

```js
function renderUserField(name, dispatch) {
  return elt("label", {}, "Your name: ", elt("input", {
    type: "text",
    value: name,
    onchange(event) {
      dispatch({type: "setUser", user: event.target.value});
    }
  }));
}
```

A `elt`função usada para construir elementos DOM é a que usamos no [Capítulo 19](https://eloquentjavascript.net/19_paint.html) .

Uma função semelhante é usada para render palestras, que incluem uma lista de comentários e um formulário para adicionar um novo comentário.

```js
function renderTalk(talk, dispatch) {
  return elt(
    "section", {className: "talk"},
    elt("h2", null, talk.title, " ", elt("button", {
      type: "button",
      onclick() {
        dispatch({type: "deleteTalk", talk: talk.title});
      }
    }, "Delete")),
    elt("div", null, "by ",
        elt("strong", null, talk.presenter)),
    elt("p", null, talk.summary),
    ...talk.comments.map(renderComment),
    elt("form", {
      onsubmit(event) {
        event.preventDefault();
        let form = event.target;
        dispatch({type: "newComment",
                  talk: talk.title,
                  message: form.elements.comment.value});
        form.reset();
      }
    }, elt("input", {type: "text", name: "comment"}), " ",
       elt("button", {type: "submit"}, "Add comment")));
}
```

O `"submit"`manipulador de eventos chama `form.reset`para limpar o conteúdo do formulário após criar uma `"newComment"`ação.

Ao criar partes do DOM moderadamente complexas, esse estilo de programação começa a parecer bastante confuso. Existe uma extensão JavaScript amplamente usada (não padrão) chamada *JSX,* que permite escrever HTML diretamente em seus scripts, o que pode tornar esse código mais bonito (dependendo do que você considera bonito). Antes de poder executar esse código, é necessário executar um programa em seu script para converter as chamadas de função pseudo-HTML em JavaScript, como as que usamos aqui.

Os comentários são mais simples de renderizar.

```js
function renderComment(comment) {
  return elt("p", {className: "comment"},
             elt("strong", null, comment.author),
             ": ", comment.message);
}
```

Por fim, o formulário que o usuário pode usar para criar uma nova conversa é renderizado da seguinte maneira:

```js
function renderTalkForm(dispatch) {
  let title = elt("input", {type: "text"});
  let summary = elt("input", {type: "text"});
  return elt("form", {
    onsubmit(event) {
      event.preventDefault();
      dispatch({type: "newTalk",
                title: title.value,
                summary: summary.value});
      event.target.reset();
    }
  }, elt("h3", null, "Submit a Talk"),
     elt("label", null, "Title: ", title),
     elt("label", null, "Summary: ", summary),
     elt("button", {type: "submit"}, "Submit"));
}
```

### Polling

Para iniciar o aplicativo, precisamos da lista atual de palestras. Como o carregamento inicial está intimamente relacionado ao longo processo de pesquisa - o `ETag`carregamento deve ser usado durante a pesquisa - escreveremos uma função que continua pesquisando o servidor `/talks`e chama uma função de retorno de chamada quando um novo conjunto de conversas estiver disponível.

```js
async function pollTalks(update) {
  let tag = undefined;
  for (;;) {
    let response;
    try {
      response = await fetchOK("/talks", {
        headers: tag && {"If-None-Match": tag,
                         "Prefer": "wait=90"}
      });
    } catch (e) {
      console.log("Request failed: " + e);
      await new Promise(resolve => setTimeout(resolve, 500));
      continue;
    }
    if (response.status == 304) continue;
    tag = response.headers.get("ETag");
    update(await response.json());
  }
}
```

Essa é uma `async`função para facilitar o loop e aguardar a solicitação. Ele executa um loop infinito que, em cada iteração, recupera a lista de conversas - normalmente ou, se essa não for a primeira solicitação, com os cabeçalhos incluídos que a tornam uma longa solicitação de pesquisa.

Quando uma solicitação falha, a função espera um momento e tenta novamente. Dessa forma, se sua conexão de rede desaparecer por um tempo e depois voltar, o aplicativo poderá se recuperar e continuar atualizando. A promessa resolvida via `setTimeout`é uma maneira de forçar a `async`função a esperar.

Quando o servidor retorna uma resposta 304, isso significa que um longo pedido de pesquisa atingiu o tempo limite; portanto, a função deve iniciar imediatamente a próxima solicitação. Se a resposta for uma resposta normal 200, seu corpo será lido como JSON e passado para o retorno de chamada e seu `ETag`valor de cabeçalho será armazenado para a próxima iteração.

### A aplicação

O componente a seguir une toda a interface do usuário:

```js
class SkillShareApp {
  constructor(state, dispatch) {
    this.dispatch = dispatch;
    this.talkDOM = elt("div", {className: "talks"});
    this.dom = elt("div", null,
                   renderUserField(state.user, dispatch),
                   this.talkDOM,
                   renderTalkForm(dispatch));
    this.syncState(state);
  }

  syncState(state) {
    if (state.talks != this.talks) {
      this.talkDOM.textContent = "";
      for (let talk of state.talks) {
        this.talkDOM.appendChild(
          renderTalk(talk, this.dispatch));
      }
      this.talks = state.talks;
    }
  }
}
```

Quando as conversas mudam, esse componente redesenha todos eles. Isso é simples, mas também um desperdício. Voltaremos a isso nos exercícios.

Podemos iniciar o aplicativo assim:

```js
function runApp() {
  let user = localStorage.getItem("userName") || "Anon";
  let state, app;
  function dispatch(action) {
    state = handleAction(state, action);
    app.syncState(state);
  }

  pollTalks(talks => {
    if (!app) {
      state = {user, talks};
      app = new SkillShareApp(state, dispatch);
      document.body.appendChild(app.dom);
    } else {
      dispatch({type: "setTalks", talks});
    }
  }).catch(reportError);
}

runApp();
```

Se você executar o servidor e abrir duas janelas do navegador para [*http: // localhost: 8000, uma*](http://localhost:8000/) ao lado da outra, poderá ver que as ações executadas em uma janela são imediatamente visíveis na outra.

## Exercícios

Os exercícios a seguir envolverão a modificação do sistema definido neste capítulo. Para trabalhar com eles, baixe o código primeiro ( [*https://eloquentjavascript.net/code/skillsharing.zip*](https://eloquentjavascript.net/code/skillsharing.zip) ), instale o Node [*https://nodejs.org*](https://nodejs.org/) e instale a dependência do projeto `npm install`.

### Persistência de disco

O servidor de compartilhamento de habilidades mantém seus dados puramente na memória. Isso significa que, quando falha ou é reiniciado por qualquer motivo, todas as conversas e comentários são perdidos.

Estenda o servidor para que ele armazene os dados de conversação em disco e recarregue automaticamente os dados quando forem reiniciados. Não se preocupe com eficiência - faça a coisa mais simples que funciona.

A solução mais simples que posso encontrar é codificar todo o `talks`objeto como JSON e despejá-lo em um arquivo `writeFile`. Já existe um método ( `updated`) chamado toda vez que os dados do servidor são alterados. Pode ser estendido para gravar os novos dados no disco.

Escolha um nome de arquivo, por exemplo `./talks.json`. Quando o servidor é iniciado, ele pode tentar ler esse arquivo e `readFile`, se for bem-sucedido, o servidor pode usar o conteúdo do arquivo como dados iniciais.

Cuidado, no entanto. O `talks`objeto começou como um objeto sem protótipo, para que o `in`operador pudesse ser usado com segurança. `JSON.parse`retornará objetos regulares com `Object.prototype`seu protótipo. Se você usar JSON como formato de arquivo, precisará copiar as propriedades do objeto retornado `JSON.parse`para um novo objeto sem protótipo.

### O campo de comentário é redefinido

O redesenho por atacado das conversas funciona muito bem porque geralmente você não pode dizer a diferença entre um nó DOM e sua substituição idêntica. Mas há exceções. Se você começar a digitar algo no campo de comentário para uma conversa em uma janela do navegador e, em outra, adicionar um comentário a essa conversa, o campo na primeira janela será redesenhado, removendo o conteúdo e o foco.

Em uma discussão acalorada, em que várias pessoas estão adicionando comentários ao mesmo tempo, isso seria irritante. Você pode encontrar uma maneira de resolvê-lo?

A melhor maneira de fazer isso é provavelmente criar objetos componentes `syncState`do talk , com um método, para que possam ser atualizados para mostrar uma versão modificada do talk. Durante a operação normal, a única maneira de alterar uma conversa é adicionando mais comentários, para que o `syncState`método seja relativamente simples.

A parte difícil é que, quando uma lista alterada de conversas chega, precisamos reconciliar a lista existente de componentes DOM com as conversas da nova lista - excluindo componentes cuja conversa foi excluída e atualizando componentes cuja conversa mudou.

Para fazer isso, pode ser útil manter uma estrutura de dados que armazene os componentes da conversa nos títulos da conversa, para que você possa descobrir facilmente se existe um componente para uma determinada conversa. Em seguida, você pode percorrer a nova matriz de conversas e, para cada uma delas, sincronizar um componente existente ou criar um novo. Para excluir componentes de conversas excluídas, você também precisará fazer um loop sobre os componentes e verificar se as conversas correspondentes ainda existem.