# Capítulo 18 HTTP e formulários

> A comunicação deve ter natureza [...] sem estado, de modo que cada solicitação do cliente para o servidor deva conter todas as informações necessárias para entender a solicitação e não pode tirar proveito de nenhum contexto armazenado no servidor.
>
> *—Roy Fielding, estilos arquitetônicos e design de arquiteturas de software baseadas em rede*

![Imagem de um formulário da Web em um pergaminho medieval](https://eloquentjavascript.net/img/chapter_picture_18.jpg)

O *Hypertext Transfer Protocol* , já mencionado no [Capítulo 13](https://eloquentjavascript.net/13_browser.html#web) , é o mecanismo pelo qual os dados são solicitados e fornecidos na World Wide Web. Este capítulo descreve o protocolo com mais detalhes e explica como o JavaScript do navegador tem acesso a ele.

## O protocolo

Se você digitar *eloquentjavascript.net/18_http.html* na barra de endereços do navegador, o navegador procurará primeiro o endereço do servidor associado ao *eloquentjavascript.net* e tentará abrir uma conexão TCP na porta 80, a porta padrão para o tráfego HTTP . Se o servidor existir e aceitar a conexão, o navegador poderá enviar algo como isto:

```http
GET /18_http.html HTTP/1.1
Host: eloquentjavascript.net
User-Agent: Your browser's name
```

Em seguida, o servidor responde, através da mesma conexão.

```http
HTTP/1.1 200 OK
Content-Length: 65585
Content-Type: text/html
Last-Modified: Mon, 08 Jan 2018 10:29:45 GMT

<!doctype html>
... the rest of the document
```

O navegador pega a parte da resposta após a linha em branco, seu *corpo* (que não deve ser confundido com a ``tag HTML ) e a exibe como um documento HTML.

As informações enviadas pelo cliente são chamadas de *solicitação* . Começa com esta linha:

```http
GET /18_http.html HTTP/1.1
```

A primeira palavra é o *método* da solicitação. `GET`significa que queremos *obter* o recurso especificado. Outros métodos comuns são `DELETE`excluir um recurso, `PUT`criar ou substituí-lo e `POST`enviar informações a ele. Observe que o servidor não é obrigado a executar todas as solicitações recebidas. Se você acessar um site aleatório e contá-lo para `DELETE`a página principal, ele provavelmente será recusado.

A parte após o nome do método é o caminho do *recurso ao qual* a solicitação se aplica. No caso mais simples, um recurso é simplesmente um arquivo no servidor, mas o protocolo não exige que seja. Um recurso pode ser qualquer coisa que possa ser transferida *como se* fosse um arquivo. Muitos servidores geram as respostas que produzem em tempo real. Por exemplo, se você abrir [*https://github.com/marijnh*](https://github.com/marijnh) , o servidor procurará em seu banco de dados um usuário chamado "marijnh" e, se encontrar um, gerará uma página de perfil para esse usuário.

Após o caminho do recurso, a primeira linha da solicitação menciona `HTTP/1.1`para indicar a versão do protocolo HTTP que está usando.

Na prática, muitos sites usam o HTTP versão 2, que suporta os mesmos conceitos da versão 1.1, mas é muito mais complicado para que possa ser mais rápido. Os navegadores alternam automaticamente para a versão de protocolo apropriada ao conversar com um determinado servidor, e o resultado de uma solicitação é o mesmo, independentemente da versão usada. Como a versão 1.1 é mais direta e fácil de brincar, focaremos nisso.

A resposta do servidor também começará com uma versão, seguida pelo status da resposta, primeiro como um código de status de três dígitos e depois como uma sequência legível por humanos.

```http
HTTP/1.1 200 OK
```

Os códigos de status começando com 2 indicam que a solicitação foi bem-sucedida. Os códigos que começam com 4 significam que havia algo errado com a solicitação. 404 é provavelmente o código de status HTTP mais famoso - significa que o recurso não pôde ser encontrado. Os códigos que começam com 5 significam que ocorreu um erro no servidor e a culpa não é da solicitação.

A primeira linha de uma solicitação ou resposta pode ser seguida por qualquer número de *cabeçalhos* . Essas são linhas no formulário `name: value`que especificam informações extras sobre a solicitação ou resposta. Esses cabeçalhos fizeram parte da resposta de exemplo:

```http
Content-Length: 65585
Content-Type: text/html
Last-Modified: Thu, 04 Jan 2018 14:05:30 GMT
```

Isso nos diz o tamanho e o tipo do documento de resposta. Nesse caso, é um documento HTML de 65.585 bytes. Também nos diz quando esse documento foi modificado pela última vez.

Para a maioria dos cabeçalhos, o cliente e o servidor são livres para decidir se devem incluí-los em uma solicitação ou resposta. Mas alguns são necessários. Por exemplo, o `Host`cabeçalho, que especifica o nome do host, deve ser incluído em uma solicitação porque um servidor pode estar servindo vários nomes de host em um único endereço IP e, sem esse cabeçalho, o servidor não saberá qual nome de host o cliente está tentando falar para.

Após os cabeçalhos, as solicitações e as respostas podem incluir uma linha em branco seguida por um corpo, que contém os dados que estão sendo enviados. `GET`e `DELETE`solicitações não enviam ao longo de qualquer dados, mas `PUT`e `POST`solicitações fazer. Da mesma forma, alguns tipos de resposta, como respostas de erro, não requerem um corpo.

## Navegadores e HTTP

Como vimos no exemplo, um navegador fará uma solicitação quando inserirmos um URL em sua barra de endereços. Quando a página HTML resultante faz referência a outros arquivos, como imagens e arquivos JavaScript, eles também são recuperados.

Um site moderadamente complicado pode incluir facilmente de 10 a 200 recursos. Para poder buscá-las rapidamente, os navegadores farão várias `GET`solicitações simultaneamente, em vez de aguardar as respostas uma por vez.

As páginas HTML podem incluir *formulários* , que permitem ao usuário preencher informações e enviá-las ao servidor. Este é um exemplo de um formulário:

```html
<form method="GET" action="example/message.html">
  <p>Name: <input type="text" name="name"></p>
  <p>Message:<br><textarea name="message"></textarea></p>
  <p><button type="submit">Send</button></p>
</form>
```

Este código descreve um formulário com dois campos: um pequeno solicitando um nome e outro maior para escrever uma mensagem. Quando você clica no botão Enviar, o formulário é *enviado* , o que significa que o conteúdo de seu campo é compactado em um HTTP pedido e o navegador navega para o resultado desse pedido.

Quando o atributo ``do elemento `method`é `GET`(ou é omitido), as informações no formulário são adicionadas ao final do `action`URL como uma *string de consulta* . O navegador pode fazer uma solicitação para este URL:

```http
GET /example/message.html?name=Jean&message=Yes%3F HTTP/1.1
```

O ponto de interrogação indica o final da parte do caminho da URL e o início da consulta. É seguido por pares de nomes e valores, correspondentes ao `name`atributo nos elementos do campo de formulário e o conteúdo desses elementos, respectivamente. Um caractere e comercial ( `&`) é usado para separar os pares.

A mensagem real codificada no URL é "Sim?", Mas o ponto de interrogação é substituído por um código estranho. Alguns caracteres nas cadeias de consulta devem ser escapados. O ponto de interrogação, representado como `%3F`, é um deles. Parece haver uma regra não escrita de que todo formato precisa de seu próprio modo de escapar de caracteres. Este, chamado *codificação de URL* , usa um sinal de porcentagem seguido por dois dígitos hexadecimais (base 16) que codificam o código de caractere. Nesse caso, 3F, que é 63 em notação decimal, é o código de um caractere de ponto de interrogação. O JavaScript fornece as funções `encodeURIComponent`e `decodeURIComponent`para codificar e decodificar esse formato.

```js
console.log(encodeURIComponent("Yes?"));
// → Yes%3F
console.log(decodeURIComponent("Yes%3F"));
// → Yes?
```

Se alterarmos o `method`atributo do formulário HTML no exemplo que vimos anteriormente `POST`, a solicitação HTTP feita para enviar o formulário usará o `POST`método e colocará a string de consulta no corpo da solicitação, em vez de adicioná-lo ao URL.

```http
POST /example/message.html HTTP/1.1
Content-length: 24
Content-type: application/x-www-form-urlencoded

name=Jean&message=Yes%3F
```

`GET`solicitações devem ser usadas para solicitações que não têm efeitos colaterais, mas simplesmente solicitam informações. Solicitações que alteram algo no servidor, por exemplo, criando uma nova conta ou postando uma mensagem, devem ser expressas com outros métodos, como `POST`. Softwares do lado do cliente, como um navegador, sabem que não devem fazer `POST`solicitações às cegas, mas geralmente fazem `GET`solicitações implicitamente - por exemplo, para buscar previamente um recurso que acredita que o usuário precisará em breve.

Voltaremos aos formulários e como interagir com eles a partir do JavaScript [mais adiante neste capítulo](https://eloquentjavascript.net/18_http.html#forms) .

## Fetch

A interface pela qual o JavaScript do navegador pode fazer solicitações HTTP é chamada `fetch`. Por ser relativamente novo, usa convenientemente promessas (o que é raro para interfaces de navegador).

```js
fetch("example/data.txt").then(response => {
  console.log(response.status);
  // → 200
  console.log(response.headers.get("Content-Type"));
  // → text/plain
});
```

A chamada `fetch`retorna uma promessa que é resolvida para um `Response`objeto que contém informações sobre a resposta do servidor, como seu código de status e seus cabeçalhos. Os cabeçalhos são agrupados em um `Map`objeto semelhante que trata suas chaves (os nomes dos cabeçalhos) como não fazem distinção entre maiúsculas e minúsculas, porque os nomes de cabeçalho não devem diferenciar maiúsculas de minúsculas. Isso significa e retornará o mesmo valor.`headers.get("Content-Type")``headers.get("content-TYPE")`

Observe que a promessa retornada por `fetch`resolve com êxito, mesmo que o servidor tenha respondido com um código de erro. Também *pode* ser rejeitado se houver um erro de rede ou se o servidor ao qual a solicitação está endereçada não puder ser encontrado.

O primeiro argumento para `fetch`é a URL que deve ser solicitada. Quando esse URL não começa com um nome de protocolo (como *http:)* , é tratado como *relativo* , o que significa que é interpretado em relação ao documento atual. Quando começa com uma barra (/), substitui o caminho atual, que é a parte após o nome do servidor. Quando isso não ocorre, a parte do caminho atual até e incluindo seu último caractere de barra é colocada na frente da URL relativa.

Para obter o conteúdo real de uma resposta, você pode usar seu `text`método. Como a promessa inicial é resolvida assim que os cabeçalhos da resposta são recebidos e porque a leitura do corpo da resposta pode demorar um pouco mais, isso também retorna uma promessa.

```js
fetch("example/data.txt")
  .then(resp => resp.text())
  .then(text => console.log(text));
// → This is the content of data.txt
```

Um método semelhante, chamado `json`, retorna uma promessa que é resolvida com o valor que você obtém ao analisar o corpo como JSON ou rejeita se não for JSON válido.

Por padrão, `fetch`usa o `GET`método para fazer sua solicitação e não inclui um corpo de solicitação. Você pode configurá-lo de maneira diferente passando um objeto com opções extras como segundo argumento. Por exemplo, esta solicitação tenta excluir `example/data.txt`:

```js
fetch("example/data.txt", {method: "DELETE"}).then(resp => {
  console.log(resp.status);
  // → 405
});
```

O código de status 405 significa "método não permitido", a maneira de um servidor HTTP dizer "Não posso fazer isso".

Para adicionar um corpo de solicitação, você pode incluir uma `body`opção. Para definir cabeçalhos, existe a `headers`opção. Por exemplo, essa solicitação inclui um `Range`cabeçalho, que instrui o servidor a retornar apenas parte de uma resposta.

```js
fetch("example/data.txt", {headers: {Range: "bytes=8-19"}})
  .then(resp => resp.text())
  .then(console.log);
// → the content
```

O navegador adicionará automaticamente alguns cabeçalhos de solicitação, como “Host” e os necessários para o servidor descobrir o tamanho do corpo. Mas adicionar seus próprios cabeçalhos geralmente é útil para incluir itens como informações de autenticação ou informar ao servidor qual formato de arquivo você deseja receber.

## HTTP sandboxing

Fazer solicitações HTTP em scripts de página da Web mais uma vez levanta preocupações sobre segurança. A pessoa que controla o script pode não ter os mesmos interesses que a pessoa em cujo computador está executando. Mais especificamente, se eu visitar o site *themafia.org* , não quero que seus scripts possam fazer uma solicitação ao *mybank.com* , usando informações de identificação do meu navegador, com instruções para transferir todo o meu dinheiro para uma conta aleatória.

Por esse motivo, os navegadores nos protegem, impedindo que scripts façam solicitações HTTP para outros domínios (nomes como *themafia.org* e *mybank.com* ).

Isso pode ser um problema irritante ao criar sistemas que desejam acessar vários domínios por razões legítimas. Felizmente, os servidores podem incluir um cabeçalho como este em sua resposta para indicar explicitamente ao navegador que não há problema em a solicitação vir de outro domínio:

```http
Access-Control-Allow-Origin: *
```

## Apreciando HTTP

Ao criar um sistema que requer comunicação entre um programa JavaScript em execução no navegador (lado do cliente) e um programa em um servidor (lado do servidor), existem várias maneiras diferentes de modelar essa comunicação.

Um modelo comumente usado é o de *chamadas de procedimento remoto* . Nesse modelo, a comunicação segue os padrões das chamadas de funções normais, exceto que a função está realmente sendo executada em outra máquina. Chamar isso envolve fazer uma solicitação ao servidor que inclua o nome e os argumentos da função. A resposta a essa solicitação contém o valor retornado.

Ao pensar em termos de chamadas de procedimento remoto, o HTTP é apenas um veículo de comunicação, e você provavelmente escreverá uma camada de abstração que o oculta totalmente.

Outra abordagem é construir sua comunicação em torno do conceito de recursos e métodos HTTP. Em vez de um procedimento remoto chamado `addUser`, você usa uma `PUT`solicitação para `/users/larry`. Em vez de codificar as propriedades desse usuário nos argumentos das funções, você define um formato de documento JSON (ou usa um formato existente) que representa um usuário. O corpo da `PUT`solicitação para criar um novo recurso é esse documento. Um recurso é buscado fazendo uma `GET`solicitação para a URL do recurso (por exemplo `/user/larry`), que retorna novamente o documento que representa o recurso.

Essa segunda abordagem facilita o uso de alguns dos recursos que o HTTP fornece, como suporte para recursos de armazenamento em cache (mantendo uma cópia no cliente para acesso rápido). Os conceitos usados no HTTP, que são bem projetados, podem fornecer um conjunto útil de princípios para projetar a interface do servidor.

## Segurança e HTTPS

Os dados que viajam pela Internet tendem a seguir uma longa e perigosa estrada. Para chegar ao seu destino, ele deve percorrer qualquer coisa, desde pontos de acesso Wi-Fi da cafeteria até redes controladas por várias empresas e estados. Em qualquer ponto do percurso, ele pode ser inspecionado ou até modificado.

Se for importante que algo permaneça secreto, como a senha da sua conta de email, ou que chegue ao seu destino sem modificações, como o número da conta para a qual você transfere dinheiro através do site do banco, o HTTP simples não é bom o suficiente.

O protocolo HTTP seguro, usado para URLs que começam com *https: //* , envolve o tráfego HTTP de uma maneira que dificulta a leitura e a adulteração. Antes de trocar dados, o cliente verifica se o servidor é quem afirma ser, solicitando que prove que possui um certificado criptográfico emitido por uma autoridade de certificação que o navegador reconhece. Em seguida, todos os dados que passam pela conexão são criptografados de forma a impedir a interceptação e a violação.

Assim, quando funciona corretamente, o HTTPS impede que outras pessoas se façam passar pelo site com o qual você está tentando conversar e bisbilhote sua comunicação. Não é perfeito, e houve vários incidentes em que o HTTPS falhou devido a certificados falsificados ou roubados e software quebrado, mas é *muito* mais seguro que o HTTP simples.

## Campos de formulário

Os formulários foram projetados originalmente para a Web pré-JavaScript para permitir que os sites enviem informações enviadas pelo usuário em uma solicitação HTTP. Esse design pressupõe que a interação com o servidor sempre ocorra navegando para uma nova página.

Mas seus elementos fazem parte do DOM, como o restante da página, e os elementos DOM que representam os campos do formulário suportam várias propriedades e eventos que não estão presentes em outros elementos. Isso possibilita inspecionar e controlar esses campos de entrada com programas JavaScript e executar ações como adicionar novas funcionalidades a um formulário ou usar formulários e campos como blocos de construção em um aplicativo JavaScript.

Um formulário da web consiste em qualquer número de campos de entrada agrupados em uma ``tag. O HTML permite vários estilos diferentes de campos, desde simples caixas de seleção ativadas / desativadas até menus e campos suspensos para entrada de texto. Este livro não tentará discutir de forma abrangente todos os tipos de campos, mas começaremos com uma visão geral aproximada.

Muitos tipos de campo usam a ``tag. O `type`atributo dessa tag é usado para selecionar o estilo do campo. Estes são alguns ``tipos comumente usados :

| `text`     | Um campo de texto de linha única                       |
| ---------- | ------------------------------------------------------ |
| `password` | Igual a `text`mas oculta o texto digitado              |
| `checkbox` | Um interruptor on / off                                |
| `radio`    | (Parte de) um campo de múltipla escolha                |
| `file`     | Permite que o usuário escolha um arquivo do computador |

Os campos do formulário não precisam necessariamente aparecer em uma ``tag. Você pode colocá-los em qualquer lugar da página. Esses campos sem formulário não podem ser enviados (apenas um formulário como um todo), mas, ao responder a entradas com JavaScript, muitas vezes não queremos enviar nossos campos normalmente.

```html
<p><input type="text" value="abc"> (text)</p>
<p><input type="password" value="abc"> (password)</p>
<p><input type="checkbox" checked> (checkbox)</p>
<p><input type="radio" value="A" name="choice">
   <input type="radio" value="B" name="choice" checked>
   <input type="radio" value="C" name="choice"> (radio)</p>
<p><input type="file"> (file)</p>
```

A interface JavaScript para esses elementos difere do tipo do elemento.

Os campos de texto de múltiplas linhas têm sua própria tag, ``principalmente porque o uso de um atributo para especificar um valor inicial de várias linhas seria estranho. A ``tag requer uma tag de fechamento correspondente e usa o texto entre os dois, em vez do atributo, como texto inicial.```value`

```html
<textarea>
one
two
three
</textarea>
```

Por fim, a ``tag é usada para criar um campo que permite ao usuário selecionar dentre várias opções predefinidas.

```html
<select>
  <option>Pancakes</option>
  <option>Pudding</option>
  <option>Ice cream</option>
</select>
```

Sempre que o valor de um campo de formulário for alterado, ele disparará um `"change"`evento.

## Focus

Diferentemente da maioria dos elementos em documentos HTML, os campos do formulário podem obter o *foco do teclado* . Quando clicados ou ativados de alguma outra maneira, eles se tornam o elemento atualmente ativo e o destinatário da entrada do teclado.

Assim, você pode digitar em um campo de texto apenas quando estiver focado. Outros campos respondem de maneira diferente aos eventos do teclado. Por exemplo, um ``menu tenta ir para a opção que contém o texto digitado pelo usuário e responde às teclas de seta movendo sua seleção para cima e para baixo.

Podemos controlar o foco do JavaScript com os métodos `focus`e `blur`. O primeiro move o foco para o elemento DOM no qual é chamado e o segundo remove o foco. O valor em corresponde ao elemento atualmente focado.`document.activeElement`

```html
<input type="text">
<script>
  document.querySelector("input").focus();
  console.log(document.activeElement.tagName);
  // → INPUT
  document.querySelector("input").blur();
  console.log(document.activeElement.tagName);
  // → BODY
</script>
```

Para algumas páginas, espera-se que o usuário deseje interagir com um campo de formulário imediatamente. O JavaScript pode ser usado para focar esse campo quando o documento é carregado, mas o HTML também fornece o `autofocus`atributo, que produz o mesmo efeito, deixando o navegador saber o que estamos tentando alcançar. Isso dá ao navegador a opção de desativar o comportamento quando não é apropriado, como quando o usuário coloca o foco em outra coisa.

Os navegadores tradicionalmente também permitem que o usuário mova o foco pelo documento pressionando a tecla Tab . Podemos influenciar a ordem na qual os elementos recebem foco com o `tabindex`atributo O exemplo de documento a seguir permitirá que o foco passe da entrada de texto para o botão OK, em vez de acessar o link de ajuda primeiro:

```html
<input type="text" tabindex=1> <a href=".">(help)</a>
<button onclick="console.log('ok')" tabindex=2>OK</button>
```

Por padrão, a maioria dos tipos de elementos HTML não pode ser focada. Mas você pode adicionar um `tabindex`atributo a qualquer elemento que o torne focável. Um `tabindex`de -1 faz com que as guias pulem um elemento, mesmo que normalmente seja focalizável.

## Campos desativados

Todos os campos do formulário podem ser *desativados* por meio de seus `disabled`atributos. É um atributo que pode ser especificado sem valor - o fato de estar presente desativa o elemento.

```html
<button>I'm all right</button>
<button disabled>I'm out</button>
```

Os campos desativados não podem ser focados ou alterados, e os navegadores os tornam cinzentos e desbotados.

Quando um programa está processando uma ação causada por algum botão ou outro controle que possa exigir comunicação com o servidor e, portanto, demorar um pouco, pode ser uma boa idéia desabilitar o controle até que a ação seja concluída. Dessa forma, quando o usuário fica impaciente e clica novamente, ele não repete acidentalmente sua ação.

## O formulário como um todo

Quando um campo está contido em um ``elemento, seu elemento DOM terá uma `form`propriedade vinculada ao elemento DOM do formulário. O ``elemento, por sua vez, possui uma propriedade chamada `elements`que contém uma coleção semelhante a uma matriz dos campos dentro dele.

O `name`atributo de um campo de formulário determina a maneira como seu valor será identificado quando o formulário for enviado. Também pode ser usado como um nome de propriedade ao acessar a `elements`propriedade do formulário , que atua tanto como um objeto de matriz (acessível por número) quanto como um mapa (acessível por nome).

```html
<form action="example/submit.html">
  Name: <input type="text" name="name"><br>
  Password: <input type="password" name="password"><br>
  <button type="submit">Log in</button>
</form>
<script>
  let form = document.querySelector("form");
  console.log(form.elements[1].type);
  // → password
  console.log(form.elements.password.type);
  // → password
  console.log(form.elements.name.form == form);
  // → true
</script>
```

Um botão com um `type`atributo de `submit`vontade, quando pressionado, faz com que o formulário seja enviado. Pressionar enter quando um campo de formulário é focado tem o mesmo efeito.

O envio de um formulário normalmente significa que o navegador navega para a página indicada pelo `action`atributo do formulário , usando `GET`uma `POST`solicitação ou uma . Mas antes que isso aconteça, um `"submit"`evento é disparado. Você pode manipular esse evento com JavaScript e impedir esse comportamento padrão chamando `preventDefault`o objeto de evento.

```html
<form action="example/submit.html">
  Value: <input type="text" name="value">
  <button type="submit">Save</button>
</form>
<script>
  let form = document.querySelector("form");
  form.addEventListener("submit", event => {
    console.log("Saving value", form.elements.value.value);
    event.preventDefault();
  });
</script>
```

A interceptação de `"submit"`eventos em JavaScript tem vários usos. Podemos escrever código para verificar se os valores inseridos pelo usuário fazem sentido e mostrar imediatamente uma mensagem de erro em vez de enviar o formulário. Ou podemos desativar a maneira regular de enviar o formulário inteiramente, como no exemplo, e fazer com que nosso programa lide com a entrada, possivelmente usando `fetch`para enviá-lo a um servidor sem recarregar a página.

## Campos de texto

Os campos criados por ``tags ou ``tags com um tipo de `text`ou `password`compartilham uma interface comum. Seus elementos DOM têm uma `value`propriedade que mantém seu conteúdo atual como um valor de sequência. Definir essa propriedade para outra sequência altera o conteúdo do campo.

As propriedades `selectionStart`e `selectionEnd`dos campos de texto nos fornecem informações sobre o cursor e a seleção no texto. Quando nada é selecionado, essas duas propriedades mantêm o mesmo número, indicando a posição do cursor. Por exemplo, 0 indica o início do texto, e 10 indica o cursor é após o 10 º personagem. Quando parte do campo é selecionada, as duas propriedades diferem, fornecendo o início e o fim do texto selecionado. Assim `value`, essas propriedades também podem ser gravadas.

Imagine que você está escrevendo um artigo sobre Khasekhemwy, mas tem dificuldade em escrever o nome dele. O código a seguir cria uma ``tag com um manipulador de eventos que, quando você pressiona F2, insere a string "Khasekhemwy" para você.

```html
<textarea></textarea>
<script>
  let textarea = document.querySelector("textarea");
  textarea.addEventListener("keydown", event => {
    // The key code for F2 happens to be 113
    if (event.keyCode == 113) {
      replaceSelection(textarea, "Khasekhemwy");
      event.preventDefault();
    }
  });
  function replaceSelection(field, word) {
    let from = field.selectionStart, to = field.selectionEnd;
    field.value = field.value.slice(0, from) + word +
                  field.value.slice(to);
    // Put the cursor after the word
    field.selectionStart = from + word.length;
    field.selectionEnd = from + word.length;
  }
</script>
```

A `replaceSelection`função substitui a parte atualmente selecionada do conteúdo de um campo de texto pela palavra especificada e move o cursor após essa palavra para que o usuário possa continuar digitando.

O `"change"`evento para um campo de texto não é acionado toda vez que algo é digitado. Em vez disso, é acionado quando o campo perde o foco após a alteração do conteúdo. Para responder imediatamente às alterações em um campo de texto, você deve registrar um manipulador para o `"input"`evento, que é acionado sempre que o usuário digita um caractere, exclui texto ou manipula o conteúdo do campo.

O exemplo a seguir mostra um campo de texto e um contador exibindo o comprimento atual do texto no campo:

```html
<input type="text"> length: <span id="length">0</span>
<script>
  let text = document.querySelector("input");
  let output = document.querySelector("#length");
  text.addEventListener("input", () => {
    output.textContent = text.value.length;
  });
</script>
```

## Checkboxes e Radio buttons

Um campo da caixa de seleção é uma alternância binária. Seu valor pode ser extraído ou alterado através de sua `checked`propriedade, que contém um valor booleano.

```html
<label>
  <input type="checkbox" id="purple"> Make this page purple
</label>
<script>
  let checkbox = document.querySelector("#purple");
  checkbox.addEventListener("change", () => {
    document.body.style.background =
      checkbox.checked ? "mediumpurple" : "";
  });
</script>
```

A ``tag associa um pedaço de documento a um campo de entrada. Clicar em qualquer lugar do rótulo ativará o campo, que o focaliza e alterna seu valor quando é uma caixa de seleção ou botão de opção.

Um botão de opção é semelhante a uma caixa de seleção, mas está implicitamente vinculado a outros botões de opção com o mesmo `name`atributo, para que apenas um deles possa estar ativo a qualquer momento.

```html
Color:
<label>
  <input type="radio" name="color" value="orange"> Orange
</label>
<label>
  <input type="radio" name="color" value="lightgreen"> Green
</label>
<label>
  <input type="radio" name="color" value="lightblue"> Blue
</label>
<script>
  let buttons = document.querySelectorAll("[name=color]");
  for (let button of Array.from(buttons)) {
    button.addEventListener("change", () => {
      document.body.style.background = button.value;
    });
  }
</script>
```

Os colchetes na consulta CSS fornecida `querySelectorAll`são usados para corresponder aos atributos. Selecciona elementos cujo `name`atributo é `"color"`.

## Campos Select

Campos de seleção são conceitualmente semelhantes aos botões de opção - eles também permitem ao usuário escolher entre um conjunto de opções. Porém, quando um botão de opção coloca o layout das opções sob nosso controle, a aparência de uma ``tag é determinada pelo navegador.

Os campos de seleção também têm uma variante mais semelhante a uma lista de caixas de seleção, em vez de caixas de rádio. Quando atribuído o `multiple`atributo, uma ``tag permitirá ao usuário selecionar qualquer número de opções, em vez de apenas uma única opção. Na maioria dos navegadores, isso será exibido diferentemente de um campo de seleção normal, que geralmente é desenhado como um controle *suspenso* que mostra as opções somente quando você o abre.

Cada ``tag tem um valor. Este valor pode ser definido com um `value`atributo. Quando isso não for fornecido, o texto dentro da opção contará como seu valor. A `value`propriedade de um ``elemento reflete a opção selecionada no momento. No `multiple`entanto, para um campo, essa propriedade não significa muito, pois fornecerá o valor de apenas *uma* das opções atualmente selecionadas.

As ``tags para um ``campo podem ser acessadas como um objeto semelhante a um array através da `options`propriedade do campo . Cada opção possui uma propriedade chamada `selected`, que indica se essa opção está selecionada no momento. A propriedade também pode ser gravada para selecionar ou desmarcar uma opção.

Este exemplo extrai os valores selecionados de um `multiple`campo de seleção e os utiliza para compor um número binário de bits individuais. Mantenha o controle (ou comando no Mac) pressionado para selecionar várias opções.

```html
<select multiple>
  <option value="1">0001</option>
  <option value="2">0010</option>
  <option value="4">0100</option>
  <option value="8">1000</option>
</select> = <span id="output">0</span>
<script>
  let select = document.querySelector("select");
  let output = document.querySelector("#output");
  select.addEventListener("change", () => {
    let number = 0;
    for (let option of Array.from(select.options)) {
      if (option.selected) {
        number += Number(option.value);
      }
    }
    output.textContent = number;
  });
</script>
```

## Campos de arquivo

Os campos de arquivo foram originalmente projetados como uma maneira de fazer upload de arquivos da máquina do usuário por meio de um formulário. Nos navegadores modernos, eles também fornecem uma maneira de ler esses arquivos de programas JavaScript. O campo atua como uma espécie de porteiro. O script não pode simplesmente começar a ler arquivos particulares no computador do usuário, mas se o usuário selecionar um arquivo nesse campo, o navegador interpreta essa ação para que o script possa ler o arquivo.

Um campo de arquivo geralmente se parece com um botão rotulado com algo como "escolher arquivo" ou "procurar", com informações sobre o arquivo escolhido ao lado.

```html
<input type="file">
<script>
  let input = document.querySelector("input");
  input.addEventListener("change", () => {
    if (input.files.length > 0) {
      let file = input.files[0];
      console.log("You chose", file.name);
      if (file.type) console.log("It has type", file.type);
    }
  });
</script>
```

A `files`propriedade de um elemento do campo de arquivo é um objeto semelhante a uma matriz (novamente, não uma matriz real) que contém os arquivos escolhidos no campo. Está inicialmente vazio. O motivo de não haver simplesmente uma `file`propriedade é que os campos de arquivo também suportam um `multiple`atributo, o que possibilita selecionar vários arquivos ao mesmo tempo.

Os objetos no `files`objeto têm propriedades como `name`(o nome do arquivo), `size`(o tamanho do arquivo em bytes, que são pedaços de 8 bits) e `type`(o tipo de mídia do arquivo, como `text/plain`ou `image/jpeg`).

O que não possui é uma propriedade que contém o conteúdo do arquivo. Chegar a isso é um pouco mais envolvido. Como a leitura de um arquivo do disco pode demorar, a interface deve ser assíncrona para evitar o congelamento do documento.

```html
<input type="file" multiple>
<script>
  let input = document.querySelector("input");
  input.addEventListener("change", () => {
    for (let file of Array.from(input.files)) {
      let reader = new FileReader();
      reader.addEventListener("load", () => {
        console.log("File", file.name, "starts with",
                    reader.result.slice(0, 20));
      });
      reader.readAsText(file);
    }
  });
</script>
```

A leitura de um arquivo é feita criando um `FileReader`objeto, registrando um `"load"`manipulador de eventos para ele e chamando seu `readAsText`método, fornecendo o arquivo que queremos ler. Depois que o carregamento termina, a `result`propriedade do leitor contém o conteúdo do arquivo.

`FileReader`s também dispara um `"error"`evento ao ler o arquivo falha por qualquer motivo. O próprio objeto de erro terminará na `error`propriedade do leitor . Essa interface foi projetada antes que as promessas se tornassem parte do idioma. Você poderia envolvê-lo em uma promessa como esta:

```js
function readFileText(file) {
  return new Promise((resolve, reject) => {
    let reader = new FileReader();
    reader.addEventListener(
      "load", () => resolve(reader.result));
    reader.addEventListener(
      "error", () => reject(reader.error));
    reader.readAsText(file);
  });
}
```

## Armazenando Dados do Cliente

Páginas HTML simples com um pouco de JavaScript podem ser um ótimo formato para "mini aplicativos" - pequenos programas auxiliares que automatizam tarefas básicas. Ao conectar alguns campos de formulário a manipuladores de eventos, você pode fazer qualquer coisa, desde conversão entre centímetros e polegadas até senhas de computação de uma senha mestra e de um nome de site.

Quando esse aplicativo precisa se lembrar de algo entre as sessões, você não pode usar ligações JavaScript - elas são descartadas toda vez que a página é fechada. Você pode configurar um servidor, conectá-lo à Internet e fazer com que seu aplicativo armazene algo lá. Veremos como fazer isso no [capítulo 20](https://eloquentjavascript.net/20_node.html) . Mas isso é muito trabalho e complexidade extra. Às vezes, basta manter os dados no navegador.

O `localStorage`objeto pode ser usado para armazenar dados de uma maneira que sobreviva às recargas da página. Este objeto permite que você arquive valores de cadeia de caracteres em nomes.

```js
localStorage.setItem("username", "marijn");
console.log(localStorage.getItem("username"));
// → marijn
localStorage.removeItem("username");
```

Um valor `localStorage`permanece em até que seja substituído, é removido com `removeItem`ou o usuário limpa seus dados locais.

Sites de domínios diferentes obtêm compartimentos de armazenamento diferentes. Isso significa que os dados armazenados em `localStorage`um determinado site podem, em princípio, ser lidos (e substituídos) apenas por scripts no mesmo site.

Os navegadores impõem um limite no tamanho dos dados em que um site pode armazenar `localStorage`. Essa restrição, juntamente com o fato de que encher os discos rígidos das pessoas com lixo não é realmente lucrativa, impede que o recurso consuma muito espaço.

O código a seguir implementa um aplicativo de anotação bruto. Ele mantém um conjunto de notas nomeadas e permite ao usuário editar notas e criar novas.

```html
Notes: <select></select> <button>Add</button><br>
<textarea style="width: 100%"></textarea>

<script>
  let list = document.querySelector("select");
  let note = document.querySelector("textarea");

  let state;
  function setState(newState) {
    list.textContent = "";
    for (let name of Object.keys(newState.notes)) {
      let option = document.createElement("option");
      option.textContent = name;
      if (newState.selected == name) option.selected = true;
      list.appendChild(option);
    }
    note.value = newState.notes[newState.selected];

    localStorage.setItem("Notes", JSON.stringify(newState));
    state = newState;
  }
  setState(JSON.parse(localStorage.getItem("Notes")) || {
    notes: {"shopping list": "Carrots\nRaisins"},
    selected: "shopping list"
  });

  list.addEventListener("change", () => {
    setState({notes: state.notes, selected: list.value});
  });
  note.addEventListener("change", () => {
    setState({
      notes: Object.assign({}, state.notes,
                           {[state.selected]: note.value}),
      selected: state.selected
    });
  });
  document.querySelector("button")
    .addEventListener("click", () => {
      let name = prompt("Note name");
      if (name) setState({
        notes: Object.assign({}, state.notes, {[name]: ""}),
        selected: name
      });
    });
</script>
```

O script obtém seu estado inicial a partir do `"Notes"`valor armazenado `localStorage`ou, se estiver ausente, cria um estado de exemplo que possui apenas uma lista de compras. A leitura de um campo que não existe a partir de `localStorage`renderá `null`. Passar `null`para `JSON.parse`fará com que ele analise a sequência `"null"`e retorne `null`. Assim, o `||`operador pode ser usado para fornecer um valor padrão em uma situação como esta.

O `setState`método garante que o DOM esteja mostrando um determinado estado e armazene o novo estado `localStorage`. Os manipuladores de eventos chamam essa função para passar para um novo estado.

O uso de `Object.assign`no exemplo destina-se a criar um novo objeto que é um clone do antigo `state.notes`, mas com uma propriedade adicionada ou substituída. `Object.assign`pega seu primeiro argumento e adiciona todas as propriedades de quaisquer outros argumentos a ele. Assim, atribuir a ele um objeto vazio fará com que ele preencha um objeto novo. A notação de colchetes no terceiro argumento é usada para criar uma propriedade cujo nome se baseia em algum valor dinâmico.

Há outro objeto, semelhante a `localStorage`, chamado `sessionStorage`. A diferença entre os dois é que o conteúdo de `sessionStorage`é esquecido no final de cada *sessão* , o que para a maioria dos navegadores significa sempre que o navegador é fechado.

## Sumário

Neste capítulo, discutimos como o protocolo HTTP funciona. Um *cliente* envia uma solicitação, que contém um método (geralmente `GET`) e um caminho que identifica um recurso. O *servidor* decide o que fazer com a solicitação e responde com um código de status e um corpo de resposta. Pedidos e respostas podem conter cabeçalhos que fornecem informações adicionais.

A interface pela qual o JavaScript do navegador pode fazer solicitações HTTP é chamada `fetch`. Fazer uma solicitação fica assim:

```js
fetch("/18_http.html").then(r => r.text()).then(text => {
  console.log(`The page starts with ${text.slice(0, 15)}`);
});
```

Os navegadores fazem `GET`solicitações para buscar os recursos necessários para exibir uma página da web. Uma página também pode conter formulários, que permitem que as informações inseridas pelo usuário sejam enviadas como uma solicitação para uma nova página quando o formulário é enviado.

O HTML pode representar vários tipos de campos de formulário, como campos de texto, caixas de seleção, campos de múltipla escolha e seletores de arquivos.

Esses campos podem ser inspecionados e manipulados com JavaScript. Eles acionam o `"change"`evento quando alterado, acionam o `"input"`evento quando o texto é digitado e recebem eventos do teclado quando têm o foco do teclado. Propriedades como `value`(para texto e campos selecionados) ou `checked`(para caixas de seleção e botões de opção) são usadas para ler ou definir o conteúdo do campo.

Quando um formulário é enviado, um `"submit"`evento é disparado nele. Um manipulador de JavaScript pode chamar `preventDefault`esse evento para desativar o comportamento padrão do navegador. Os elementos do campo de formulário também podem ocorrer fora de uma marca de formulário.

Quando o usuário seleciona um arquivo do sistema de arquivos local em um campo do seletor de arquivos, a `FileReader`interface pode ser usada para acessar o conteúdo desse arquivo a partir de um programa JavaScript.

Os objetos `localStorage`e `sessionStorage`podem ser usados para salvar informações de uma maneira que sobrevive às recarregamentos de páginas. O primeiro objeto salva os dados para sempre (ou até o usuário decidir limpá-los) e o segundo salva até que o navegador seja fechado.

## Exercícios

### Negociação de conteúdo

Uma das coisas que o HTTP pode fazer é chamada de *negociação de conteúdo* . O `Accept`cabeçalho da solicitação é usado para informar ao servidor que tipo de documento o cliente gostaria de obter. Muitos servidores ignoram esse cabeçalho, mas quando um servidor conhece várias maneiras de codificar um recurso, ele pode olhar para esse cabeçalho e enviar o que o cliente preferir.

O URL [*https://eloquentjavascript.net/author*](https://eloquentjavascript.net/author) está configurado para responder com texto sem formatação, HTML ou JSON, dependendo do que o cliente solicitar. Esses formatos são identificados pelos padronizados *tipos de mídia* `text/plain` , `text/html`e `application/json`.

Envie solicitações para buscar todos os três formatos deste recurso. Use a `headers`propriedade no objeto de opções passado `fetch`para definir o cabeçalho nomeado `Accept`para o tipo de mídia desejado.

Por fim, tente solicitar o tipo de mídia e veja qual código de status produz.`application/rainbows+unicorns`

```js
// Your code here.
```

Baseie seu código nos `fetch`exemplos [anteriores neste capítulo](https://eloquentjavascript.net/18_http.html#fetch) .

Solicitar um tipo de mídia falsa retornará uma resposta com o código 406, “Não aceitável”, que é o código que um servidor deve retornar quando não conseguir preencher o `Accept`cabeçalho.

### Um ambiente de trabalho JavaScript

Crie uma interface que permita às pessoas digitar e executar partes do código JavaScript.

Coloque um botão ao lado de um ``campo que, quando pressionado, use o `Function`construtor que vimos no [Capítulo 10](https://eloquentjavascript.net/10_modules.html#eval) para envolver o texto em uma função e chamá-lo. Converta o valor de retorno da função, ou qualquer erro que isso aconteça, em uma string e exiba-o abaixo do campo de texto.

```html
<textarea id="code">return "hi";</textarea>
<button id="button">Run</button>
<pre id="output"></pre>

<script>
  // Your code here.
</script>
```

Use ou para obter acesso aos elementos definidos no seu HTML. Um manipulador de eventos ou eventos no botão pode obter a propriedade do campo de texto e acessá -lo.`document.querySelector``document.getElementById``"click"``"mousedown"``value``Function`

Certifique-se de agrupar a chamada `Function`e a chamada para seu resultado em um `try`bloco para capturar as exceções que ela produz. Nesse caso, realmente não sabemos que tipo de exceção estamos procurando, então pegue tudo.

A `textContent`propriedade do elemento de saída pode ser usada para preenchê-lo com uma mensagem de sequência. Ou, se você deseja manter o conteúdo antigo, crie um novo nó de texto usando e acrescente-o ao elemento. Lembre-se de adicionar um caractere de nova linha ao final para que nem toda a saída apareça em uma única linha.`document.createTextNode`

### Jogo da vida de Conway

O Jogo da Vida de Conway é uma simulação simples que cria "vida" artificial em uma grade, cada célula viva ou não. Cada geração (turno), as seguintes regras são aplicadas:

- Qualquer célula viva com menos de dois ou mais de três vizinhos vivos morre.
- Qualquer célula viva com dois ou três vizinhos vivos vive para a próxima geração.
- Qualquer célula morta com exatamente três vizinhos vivos se torna uma célula viva.

Um *vizinho* é definido como qualquer célula adjacente, incluindo as adjacentes na diagonal.

Observe que essas regras são aplicadas a toda a grade de uma só vez, não um quadrado por vez. Isso significa que a contagem de vizinhos é baseada na situação no início da geração, e as mudanças que acontecem nas células vizinhas durante essa geração não devem influenciar o novo estado de uma determinada célula.

Implemente esse jogo usando a estrutura de dados que achar apropriada. Use `Math.random`para preencher a grade com um padrão aleatório inicialmente. Exiba-o como uma grade de campos da caixa de seleção, com um botão ao lado para avançar para a próxima geração. Quando o usuário marca ou desmarca as caixas de seleção, suas alterações devem ser incluídas ao calcular a próxima geração.

```html
<div id="grid"></div>
<button id="next">Next generation</button>

<script>
  // Your code here.
</script>
```

Para resolver o problema de fazer as mudanças conceitualmente acontecerem ao mesmo tempo, tente ver o cálculo de uma geração como uma função pura, que pega uma grade e produz uma nova grade que representa a próxima curva.

A representação da matriz pode ser feita da maneira mostrada no [capítulo 6](https://eloquentjavascript.net/06_object.html#matrix) . Você pode contar vizinhos ativos com dois loops aninhados, fazendo um loop sobre coordenadas adjacentes em ambas as dimensões. Tome cuidado para não contar células fora do campo e ignorar a célula no centro, cujos vizinhos estamos contando.

Garantir que as alterações nas caixas de seleção entrem em vigor na próxima geração pode ser feito de duas maneiras. Um manipulador de eventos pode observar essas alterações e atualizar a grade atual para refleti-las, ou você pode gerar uma nova grade a partir dos valores nas caixas de seleção antes de calcular a próxima curva.

Se você optar por manipular eventos, convém anexar atributos que identifiquem a posição à qual cada caixa de seleção corresponde, para que seja fácil descobrir qual célula alterar.

Para desenhar a grade de caixas de seleção, você pode usar um ``elemento (consulte o [Capítulo 14](https://eloquentjavascript.net/14_dom.html#exercise_table) ) ou simplesmente colocá-los todos no mesmo elemento e colocar elementos ` `(quebra de linha) entre as linhas.