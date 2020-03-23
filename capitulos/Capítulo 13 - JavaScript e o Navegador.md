# Capítulo 13 JavaScript e o Navegador

> O sonho por trás da Web é um espaço comum de informações no qual nos comunicamos compartilhando informações. Sua universalidade é essencial: o fato de um link de hipertexto poder apontar para qualquer coisa, seja pessoal, local ou global, seja rascunho ou altamente polido.
>
> *—Tim Berners-Lee, A World Wide Web: Uma história pessoal muito curta*

![Imagem de uma central telefônica](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/chapter_picture_13.jpg)

Os próximos capítulos deste livro falarão sobre navegadores da web. Sem navegadores da web, não haveria JavaScript. Ou mesmo se houvesse, ninguém teria prestado atenção a isso.

A tecnologia da Web foi descentralizada desde o início, não apenas tecnicamente, mas também na maneira como evoluiu. Vários fornecedores de navegadores adicionaram novas funcionalidades de maneiras ad hoc e, às vezes, mal pensadas, que, às vezes, acabavam sendo adotadas por outros - e finalmente estabelecidas como padrões.

Isso é tanto benção quanto maldição. Por um lado, é empoderador não ter uma parte central controlando um sistema, mas que ela seja melhorada por várias partes trabalhando em colaboração frouxa (ou ocasionalmente hostilidade aberta). Por outro lado, a maneira casual como a Web foi desenvolvida significa que o sistema resultante não é exatamente um exemplo brilhante de consistência interna. Algumas partes são absolutamente confusas e mal concebidas.

## Redes e Internet

As redes de computadores existem desde a década de 1950. Se você colocar cabos entre dois ou mais computadores e permitir que eles enviem dados para lá e para cá através desses cabos, você poderá fazer todos os tipos de coisas maravilhosas.

E se conectar duas máquinas no mesmo prédio nos permite fazer coisas maravilhosas, conectar máquinas em todo o planeta deve ser ainda melhor. A tecnologia para começar a implementar essa visão foi desenvolvida na década de 1980 e a rede resultante é chamada *Internet* . Ele cumpriu sua promessa.

Um computador pode usar essa rede para disparar bits em outro computador. Para qualquer comunicação eficaz surgir dessa captura de bits, os computadores de ambos os lados devem saber o que os bits devem representar. O significado de qualquer sequência de bits depende inteiramente do tipo de coisa que ele está tentando expressar e do mecanismo de codificação usado.

Um *protocolo de rede* descreve um estilo de comunicação através de uma rede. Existem protocolos para envio de email, busca de email, compartilhamento de arquivos e até mesmo controle de computadores infectados por software malicioso.

Por exemplo, o HTTP ( *Hypertext Transfer Protocol* ) é um protocolo para recuperar recursos nomeados (pedaços de informações, como páginas da Web ou imagens). Ele especifica que o lado que faz a solicitação deve começar com uma linha como esta, nomeando o recurso e a versão do protocolo que está tentando usar:

```
GET /index.html HTTP / 1.1
```

Há muito mais regras sobre como o solicitante pode incluir mais informações na solicitação e como o outro lado, que retorna o recurso, empacota seu conteúdo. Veremos o HTTP com mais detalhes no [Capítulo 18](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2018%20-%20%20HTTP%20e%20formul%C3%A1rios.md) .

A maioria dos protocolos é construída sobre outros protocolos. O HTTP trata a rede como um dispositivo semelhante a um fluxo no qual você pode inserir bits e fazer com que eles cheguem ao destino correto na ordem correta. Como vimos no [Capítulo 11](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2011%20-%20Programa%C3%A7%C3%A3o%20ass%C3%ADncrona.md) , garantir essas coisas já é um problema bastante difícil.

O TCP ( *Transmission Control Protocol* ) é um protocolo que resolve esse problema. Todos os dispositivos conectados à Internet "falam" e a maior parte da comunicação na Internet é construída sobre ela.

Uma conexão TCP funciona da seguinte maneira: um computador deve estar aguardando ou *escutando* para que outros computadores comecem a conversar com ele. Para poder escutar diferentes tipos de comunicação ao mesmo tempo em uma única máquina, cada ouvinte tem um número (chamado de *porta* ) associado a ele. A maioria dos protocolos especifica qual porta deve ser usada por padrão. Por exemplo, quando queremos enviar um email usando o protocolo SMTP, espera-se que a máquina pela qual o enviamos esteja escutando na porta 25.

Outro computador pode estabelecer uma conexão conectando-se à máquina de destino usando o número da porta correto. Se a máquina de destino puder ser alcançada e estiver escutando nessa porta, a conexão será criada com sucesso. O computador de escuta é chamado de *servidor* e o computador de conexão é chamado de *cliente* .

Essa conexão atua como um tubo de duas vias através do qual os bits podem fluir - as máquinas nas duas extremidades podem colocar dados nela. Uma vez que os bits são transmitidos com sucesso, eles podem ser lidos novamente pela máquina do outro lado. Este é um modelo conveniente. Você poderia dizer que o TCP fornece uma abstração da rede.

## A Web

A *World Wide Web* (que não deve ser confundida com a Internet como um todo) é um conjunto de protocolos e formatos que nos permitem visitar páginas da Web em um navegador. A parte "Web" no nome refere-se ao fato de que essas páginas podem ser facilmente vinculadas entre si, conectando-se assim a uma enorme malha pela qual os usuários podem se mover.

Para se tornar parte da Web, tudo o que você precisa fazer é conectar uma máquina à Internet e escutá-la na porta 80 com o protocolo HTTP, para que outros computadores possam solicitar documentos.

Cada documento na Web é nomeado por um URL ( *Uniform Resource Locator* ), que se parece com isso:

```
   http://eloquentjavascript.net/13_browser.html
 |      |                      |               |
 protocol       server               path
```

A primeira parte nos diz que esse URL usa o protocolo HTTP (em oposição a, por exemplo, HTTP criptografado, que seria *https: //* ). Em seguida, vem a parte que identifica de qual servidor estamos solicitando o documento. Last é uma string de caminho que identifica o documento (ou *recurso* ) específico em que estamos interessados.

As máquinas conectadas à Internet obtêm um *endereço IP* , que é um número que pode ser usado para enviar mensagens para essa máquina e se parece com `149.210.142.219`ou `2001:4860:4860::8888`. Mas listas de números mais ou menos aleatórios são difíceis de lembrar e difíceis de digitar; portanto, você pode registrar um *nome de domínio* para um endereço ou conjunto de endereços específico. Registrei o *eloquentjavascript.net* para apontar para o endereço IP de uma máquina que eu controle e, portanto, posso usar esse nome de domínio para servir páginas da web.

Se você digitar esse URL na barra de endereços do navegador, o navegador tentará recuperar e exibir o documento nesse URL. Primeiro, o seu navegador precisa descobrir a que endereço o *eloquentjavascript.net* se refere. Em seguida, usando o protocolo HTTP, ele fará uma conexão com o servidor nesse endereço e solicitará o recurso */13_browser.html* . Se tudo correr bem, o servidor envia de volta um documento, que o seu navegador exibe na tela.

## HTML

HTML, que significa *Hypertext Markup Language* , é o formato de documento usado para páginas da web. Um documento HTML contém texto, além de *tags* que dão estrutura ao texto, descrevendo itens como links, parágrafos e títulos.

Um pequeno documento HTML pode ser assim:

```html
<!doctype html>
<html>
  <head>
    <meta charset="utf-8">
    <title>My home page</title>
  </head>
  <body>
    <h1>My home page</h1>
    <p>Hello, I am Marijn and this is my home page.</p>
    <p>I also wrote a book! Read it
      <a href="http://eloquentjavascript.net">here</a>.</p>
  </body>
</html>
```

As tags, entre colchetes angulares ( `<`e `>`os símbolos para *menor que* e *maior que* ), fornecem informações sobre a estrutura do documento. O outro texto é apenas texto simples.

O documento começa com ``, que instrui o navegador a interpretar a página como HTML *moderno* , em oposição a vários dialetos que estavam em uso no passado.

Os documentos HTML têm uma cabeça e um corpo. A cabeça contém informações *sobre* o documento e o corpo contém o próprio documento. Nesse caso, o chefe declara que o título deste documento é "Minha página inicial" e que usa a codificação UTF-8, que é uma maneira de codificar texto Unicode como dados binários. O corpo do documento contém um cabeçalho (que ``significa "cabeçalho 1" - ``para ``produzir subtítulos) e dois parágrafos ( ``).

Tags vêm em várias formas. Um elemento, tal como o corpo, um número, ou uma ligação, é iniciada por uma *marca de abertura* como ``e terminado por uma *marca de fechamento* como ``. Algumas tags de abertura, como a do link ( ``), contêm informações extras na forma de `name="value"`pares. Estes são chamados de *atributos* . Nesse caso, o destino do link é indicado com , onde significa "referência de hipertexto".`href="http://eloquentjavascript.net"``href`

Alguns tipos de tags não incluem nada e, portanto, não precisam ser fechados. A tag de metadados ``é um exemplo disso.

Para poder incluir colchetes angulares no texto de um documento, mesmo tendo um significado especial em HTML, outra forma de notação especial precisa ser introduzida. Um colchete angular de abertura simples é escrito como `<`("menor que") e um colchete de fechamento é escrito como `>`("maior que"). Em HTML, um `&`caractere " e comercial" ( ) seguido de um nome ou código de caractere e um ponto e vírgula ( `;`) é chamado de *entidade* e será substituído pelo caractere que ele codifica.

Isso é análogo ao modo como as barras invertidas são usadas nas strings JavaScript. Como esse mecanismo também confere aos caracteres e comercial um significado especial, eles precisam ser escapados como `&`. Os valores de atributo interno, que são colocados entre aspas duplas, `"`podem ser usados para inserir um caractere de cotação real.

O HTML é analisado de uma maneira notavelmente tolerante a erros. Quando faltam tags que deveriam estar lá, o navegador as reconstrói. A maneira como isso é feito foi padronizada, e você pode confiar em todos os navegadores modernos para fazê-lo da mesma maneira.

O documento a seguir será tratado exatamente como o mostrado anteriormente:

```HTML
<!doctype html>

<meta charset=utf-8>
<title>My home page</title>

<h1>My home page</h1>
<p>Hello, I am Marijn and this is my home page.
<p>I also wrote a book! Read it
  <a href=http://eloquentjavascript.net>here</a>.
```

Os ``, ``e ``as tags são ido completamente. O navegador sabe disso ``e ``pertence à cabeça e isso ``significa que o corpo foi iniciado. Além disso, não estou mais fechando explicitamente os parágrafos, pois a abertura de um novo parágrafo ou o encerramento do documento os fechará implicitamente. As aspas ao redor dos valores dos atributos também desapareceram.

Este livro geralmente omitir as ``, ``e ``marcas de exemplos para mantê-los curtos e livre da desordem. Mas *vou* fechar as tags e incluir aspas nos atributos.

Também geralmente omitirei o doctype e a `charset`declaração. Isso não deve ser tomado como incentivo para removê-los dos documentos HTML. Os navegadores costumam fazer coisas ridículas quando você os esquece. Você deve considerar o doctype e os `charset`metadados presentes implicitamente nos exemplos, mesmo quando eles não são realmente mostrados no texto.

## HTML e JavaScript

No contexto deste livro, a tag HTML mais importante é ``. Essa tag permite incluir um pedaço de JavaScript em um documento.

```HTML
<h1>Testing alert</h1>
<script>alert("hello!");</script>
```

Esse script será executado assim que sua ``tag for encontrada enquanto o navegador lê o HTML. Essa página abrirá uma caixa de diálogo quando aberta - a `alert`função se assemelha `prompt`, pois abre uma pequena janela, mas mostra apenas uma mensagem sem solicitar entrada.

Incluir programas grandes diretamente em documentos HTML geralmente é impraticável. A ``tag pode receber um `src`atributo para buscar um arquivo de script (um arquivo de texto que contém um programa JavaScript) de uma URL.

```html
<h1>Testing alert</h1>
<script src="code/hello.js"></script>
```

O arquivo *code / hello.js* incluído aqui contém o mesmo programa— `alert("hello!")`. Quando uma página HTML faz referência a outros URLs como parte dele - por exemplo, um arquivo de imagem ou um script - os navegadores da Web os recuperam imediatamente e os incluem na página.

Uma tag de script sempre deve ser fechada ``, mesmo que se refira a um arquivo de script e não contenha nenhum código. Se você esquecer isso, o restante da página será interpretado como parte do script.

Você pode carregar os módulos ES (consulte o [Capítulo 10](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2010%20-%20M%C3%B3dulos.md#es) ) no navegador, atribuindo um `type="module"`atributo à sua tag de script . Esses módulos podem depender de outros módulos usando URLs relativas a si mesmos como nomes de módulo nas `import`declarações.

Alguns atributos também podem conter um programa JavaScript. A ``tag mostrada a seguir (que aparece como um botão) tem um `onclick`atributo. O valor do atributo será executado sempre que o botão for clicado.

```html
<button onclick="alert('Boom!');">DO NOT PRESS</button>
```

Observe que eu tive que usar aspas simples para a sequência no `onclick`atributo porque aspas duplas já são usadas para citar todo o atributo. Eu também poderia ter usado `"`.

## Na caixa de areia

A execução de programas baixados da Internet é potencialmente perigosa. Você não sabe muito sobre as pessoas por trás da maioria dos sites que você visita, e elas não necessariamente significam bem. A execução de programas por pessoas que não são bem-intencionadas é como você infecta seu computador por vírus, seus dados são roubados e suas contas são invadidas.

No entanto, a atração da Web é que você pode navegar sem necessariamente confiar em todas as páginas visitadas. É por isso que os navegadores limitam severamente as coisas que um programa JavaScript pode fazer: ele não pode ver os arquivos no seu computador ou modificar qualquer coisa não relacionada à página da Web em que foi incorporado.

Isolar um ambiente de programação dessa maneira é chamado *sandboxing* , a ideia é que o programa esteja sendo reproduzido inofensivamente em uma sandbox. Mas você deve imaginar esse tipo específico de caixa de areia como tendo uma gaiola de barras de aço grossas sobre ela, para que os programas que estão tocando nela não possam realmente sair.

A parte difícil do sandboxing é permitir que os programas tenham espaço suficiente para serem úteis, mas ao mesmo tempo os impede de fazer algo perigoso. Muitas funcionalidades úteis, como a comunicação com outros servidores ou a leitura do conteúdo da área de transferência de copiar e colar, também podem ser usadas para executar tarefas problemáticas e que invadem a privacidade.

De vez em quando, alguém cria uma nova maneira de contornar as limitações de um navegador e fazer algo prejudicial, que varia desde vazar pequenas informações privadas até assumir o controle de toda a máquina na qual o navegador é executado. Os desenvolvedores do navegador respondem corrigindo o problema, e tudo está bem de novo - até que o próximo problema seja descoberto e, com sorte, publicado, em vez de ser explorado secretamente por alguma agência ou máfia do governo.

## Compatibilidade e guerra de navegadores

Nos estágios iniciais da Web, um navegador chamado Mosaic dominava o mercado. Depois de alguns anos, o saldo passou para o Netscape, que, por sua vez, foi amplamente substituído pelo Internet Explorer da Microsoft. Em qualquer ponto em que um único navegador fosse dominante, o fornecedor desse navegador teria o direito de inventar unilateralmente novos recursos para a Web. Como a maioria dos usuários usava o navegador mais popular, os sites simplesmente começavam a usar esses recursos - não importa os outros navegadores.	

Essa era a idade das trevas da compatibilidade, geralmente chamada de *guerra* dos *navegadores* . Os desenvolvedores da Web ficaram com não uma Web unificada, mas duas ou três plataformas incompatíveis. Para piorar as coisas, os navegadores em uso por volta de 2003 estavam cheios de bugs e, é claro, os bugs eram diferentes para cada navegador. A vida era difícil para as pessoas que escreviam páginas da web.

O Mozilla Firefox, um ramo sem fins lucrativos da Netscape, desafiou a posição do Internet Explorer no final dos anos 2000. Como a Microsoft não estava particularmente interessada em permanecer competitiva na época, o Firefox retirou muita participação de mercado. Na mesma época, o Google lançou seu navegador Chrome, e o navegador Safari da Apple ganhou popularidade, levando a uma situação em que havia quatro grandes players, em vez de um.

Os novos jogadores tinham uma atitude mais séria em relação aos padrões e melhores práticas de engenharia, dando-nos menos incompatibilidade e menos bugs. A Microsoft, vendo sua participação de mercado desmoronar, adotou essas atitudes em seu navegador Edge, que substitui o Internet Explorer. Se você está começando a aprender desenvolvimento web hoje, considere-se com sorte. As versões mais recentes dos principais navegadores se comportam de maneira bastante uniforme e possuem relativamente poucos bugs.