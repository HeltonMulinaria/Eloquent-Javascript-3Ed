# Introdução

> Pensamos que estamos criando o sistema para nossos próprios propósitos. Acreditamos que estamos fazendo isso à nossa própria imagem ... Mas o computador não é como nós. É uma projeção de uma parte muito pequena de nós mesmos: aquela porção dedicada à lógica, ordem, regra e clareza.
>
> *—Ellen Ullman, Perto da Máquina: Tecnofilia e seus descontentamentos*

![Imagem de uma chave de fenda e uma placa de circuito](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/img/chapter_picture_00.jpg)

Este é um livro sobre como instruir computadores. Atualmente, os computadores são tão comuns quanto as chaves de fenda, mas são um pouco mais complexos, e fazê-los fazer o que você quer que eles façam nem sempre é fácil.

Se a tarefa que você possui para o seu computador é comum e bem compreendida, como mostrar seu e-mail ou agir como uma calculadora, você pode abrir o aplicativo apropriado e começar a trabalhar. Mas para tarefas exclusivas ou abertas, provavelmente não há aplicativo.

É aí que entra a *programação* . A *programação* é o ato de construir um *programa* - um conjunto de instruções precisas que informam ao computador o que fazer. Como os computadores são bestas pedantes e idiotas, a programação é fundamentalmente entediante e frustrante.

Felizmente, se você pode superar esse fato e talvez até aproveitar o rigor de pensar em termos com os quais as máquinas burras podem lidar, a programação pode ser gratificante. Ele permite que você faça coisas em segundos que levariam uma *eternidade* à mão. É uma maneira de fazer com que sua ferramenta de computador faça coisas que não podia fazer antes. E fornece um exercício maravilhoso de pensamento abstrato.

A maior parte da programação é feita com linguagens de programação. Uma *linguagem de programação* é uma linguagem construída artificialmente usada para instruir computadores. É interessante que a maneira mais eficaz que encontramos de nos comunicar com um computador se apóia tão fortemente na maneira como nos comunicamos. Como as linguagens humanas, as linguagens de computador permitem que palavras e frases sejam combinadas de novas maneiras, possibilitando a expressão de conceitos sempre novos.

Em um ponto, as interfaces baseadas em linguagem, como as instruções do BASIC e do DOS dos anos 80 e 90, foram o principal método de interação com os computadores. Eles foram amplamente substituídos por interfaces visuais, que são mais fáceis de aprender, mas oferecem menos liberdade. As linguagens de computador ainda estão lá, se você souber onde procurar. Um desses idiomas, o JavaScript, está embutido em todos os navegadores modernos e, portanto, está disponível em quase todos os dispositivos.

Este livro tentará familiarizá-lo o suficiente com esse idioma para fazer coisas úteis e divertidas com ele.

## Na programação

Além de explicar o JavaScript, apresentarei os princípios básicos da programação. A programação, ao que parece, é difícil. As regras fundamentais são simples e claras, mas os programas criados sobre essas regras tendem a se tornar complexos o suficiente para introduzir suas próprias regras e complexidade. Você está construindo seu próprio labirinto, de certa forma, e você pode se perder nele.

Haverá momentos em que a leitura deste livro parecerá terrivelmente frustrante. Se você é novo na programação, haverá muito material novo para digerir. Grande parte desse material será *combinada* de maneira a exigir conexões adicionais.

Cabe a você fazer o esforço necessário. Quando estiver lutando para seguir o livro, não tire conclusões precipitadas sobre seus próprios recursos. Você está bem - você só precisa continuar assim. Faça uma pausa, releia algum material e leia e compreenda os programas e exercícios de exemplo. Aprender é um trabalho árduo, mas tudo o que você aprende é seu e facilitará o aprendizado subsequente.

> Quando a ação se torna inútil, colete informações; quando as informações se tornam inúteis, durma.
>
> Ursula K. Le Guin, a mão esquerda da escuridão

Um programa é muitas coisas. É um pedaço de texto digitado por um programador, é a força diretiva que faz o computador fazer o que faz, são dados na memória do computador, mas controla as ações executadas nessa mesma memória. As analogias que tentam comparar programas com objetos com os quais estamos familiarizados tendem a ficar aquém. Uma superficialmente adequada é a de uma máquina - muitas partes separadas tendem a estar envolvidas e, para fazer a coisa toda funcionar, precisamos considerar as maneiras pelas quais essas partes se interconectam e contribuem para a operação do todo.

Um computador é uma máquina física que atua como um host para essas máquinas imateriais. Os próprios computadores podem fazer coisas estupidamente simples. A razão pela qual eles são tão úteis é que eles fazem essas coisas a uma velocidade incrivelmente alta. Um programa pode engenhosamente combinar um número enorme dessas ações simples para fazer coisas muito complicadas.

Um programa é uma construção de pensamento. É de construção barata, sem peso e cresce facilmente sob nossas mãos de digitação.

Mas, sem cuidado, o tamanho e a complexidade de um programa ficarão fora de controle, confundindo até a pessoa que o criou. Manter os programas sob controle é o principal problema de programação. Quando um programa funciona, é bonito. A arte da programação é a habilidade de controlar a complexidade. O grande programa é moderado - simplificado em sua complexidade.

Alguns programadores acreditam que essa complexidade é melhor gerenciada usando apenas um pequeno conjunto de técnicas bem conhecidas em seus programas. Eles criaram regras estritas (“melhores práticas”) que prescrevem o formulário que os programas devem ter e permanecem cuidadosamente dentro de sua pequena zona segura.

Isso não é apenas chato, é ineficaz. Novos problemas geralmente exigem novas soluções. O campo da programação é jovem e ainda está se desenvolvendo rapidamente, e é variado o suficiente para ter espaço para abordagens totalmente diferentes. Existem muitos erros terríveis a serem cometidos no design do programa, e você deve prosseguir e cometê-los para que você os entenda. Um senso de como é um bom programa é desenvolvido na prática, não aprendido com uma lista de regras.

## Por que a linguagem é importante

No começo, no nascimento da computação, não havia linguagens de programação. Programas pareciam algo assim:

```
00110001 00000000 00000000
00110001 00000001 00000001
00110011 00000001 00000010
01010001 00001011 00000010
00100010 00000010 00001000
01000011 00000001 00000000
01000001 00000001 00000001
00010000 00000010 00000000
01100010 00000000 00000000
```

Que é um programa para adicionar os números de 1 a 10 juntos e imprimir o resultado: . Poderia rodar em uma máquina simples e hipotética. Para programar computadores antigos, era necessário definir grandes matrizes de switches na posição correta ou fazer furos em tiras de papelão e alimentá-los no computador. Você provavelmente pode imaginar como esse procedimento foi tedioso e propenso a erros. Até escrever programas simples exigia muita inteligência e disciplina. Os complexos eram quase inconcebíveis.`1 + 2 + ... + 10 = 55`

Obviamente, a inserção manual desses padrões arcanos de bits (aqueles e zeros) deu ao programador uma profunda sensação de ser um poderoso mago. E isso tem que valer alguma coisa em termos de satisfação no trabalho.

Cada linha do programa anterior contém uma única instrução. Pode ser escrito em inglês assim:

1. Armazene o número 0 no local 0 da memória.
2. Armazene o número 1 na localização da memória 1.
3. Armazene o valor do local da memória 1 no local da memória 2.
4. Subtraia o número 11 do valor no local da memória 2.
5. Se o valor no local da memória 2 for o número 0, continue com a instrução 9.
6. Adicione o valor do local da memória 1 ao local 0 da memória.
7. Adicione o número 1 ao valor da localização da memória 1.
8. Continue com a instrução 3.
9. Envie o valor do local de memória 0.

Embora isso já seja mais legível que a sopa de bits, ainda é um tanto obscuro. Usar nomes em vez de números para obter instruções e locais de memória ajuda.

```
 Set “total” to 0.
 Set “count” to 1.
[loop]
 Set “compare” to “count”.
 Subtract 11 from “compare”.
 If “compare” is zero, continue at [end].
 Add “count” to “total”.
 Add 1 to “count”.
 Continue at [loop].
[end]
 Output “total”.
```

Você pode ver como o programa funciona neste momento? As duas primeiras linhas fornecem os valores iniciais de duas localizações de memória: `total`serão usadas para criar o resultado da computação e `count`acompanharão o número que estamos vendo atualmente. As linhas usando `compare`são provavelmente as mais estranhas. O programa deseja ver se `count`é igual a 11 para decidir se pode parar de executar. Como nossa máquina hipotética é bastante primitiva, ela só pode testar se um número é zero e tomar uma decisão com base nisso. Portanto, ele usa o local da memória rotulado `compare`para calcular o valor `count - 11`e toma uma decisão com base nesse valor. As próximas duas linhas adicionam o valor de `count`ao resultado e aumentam `count`em 1 toda vez que o programa decide que`count` ainda não tem 11 anos.

Aqui está o mesmo programa em JavaScript:

```js
let total = 0, count = 1;
while (count <= 10) {
  total += count;
  count += 1;
}
console.log(total);
// → 55
```

Esta versão nos oferece mais algumas melhorias. Mais importante, não há necessidade de especificar a maneira como queremos que o programa volte mais e mais. A `while`construção cuida disso. Ele continua executando o bloco (entre chaves) abaixo dele, desde que a condição recebida seja mantida. Essa condição é `count <= 10`, o que significa " *contagem* é menor ou igual a 10". Não precisamos mais criar um valor temporário e compará-lo a zero, o que era apenas um detalhe desinteressante. Parte do poder das linguagens de programação é que elas podem cuidar de detalhes desinteressantes para nós.

No final do programa, após a conclusão da `while`construção, a `console.log`operação é usada para gravar o resultado.

Finalmente, eis a aparência do programa se tivéssemos as operações convenientes `range`e `sum`disponíveis, que respectivamente criam uma coleção de números dentro de um intervalo e calculam a soma de uma coleção de números:

```js
console.log(sum(range(1, 10)));
// → 55
```

A moral dessa história é que o mesmo programa pode ser expresso de maneira longa e curta, ilegível e legível. A primeira versão do programa foi extremamente obscuro, ao passo que este último é quase Inglês: `log`o `sum`do `range`de números de 1 a 10. (Veremos em [capítulos posteriores](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2004%20-%20Estruturas%20de%20dados%20objetos%20e%20matrizes.md) como definir operações como `sum`e `range`.)

Uma boa linguagem de programação ajuda o programador, permitindo que ele fale sobre as ações que o computador deve executar em um nível superior. Ajuda a omitir detalhes, fornece blocos de construção convenientes (como `while`e `console.log`), permite definir seus próprios blocos de construção (como `sum`e `range`) e facilita a composição desses blocos.

## O que é JavaScript?

O JavaScript foi introduzido em 1995 como uma maneira de adicionar programas a páginas da Web no navegador Netscape Navigator. A linguagem já foi adotada por todos os outros principais navegadores gráficos. Ele tornou possível os aplicativos da Web modernos - aplicativos com os quais você pode interagir diretamente sem precisar recarregar a página para cada ação. O JavaScript também é usado em sites mais tradicionais para fornecer várias formas de interatividade e inteligência.

É importante observar que o JavaScript não tem quase nada a ver com a linguagem de programação chamada Java. O nome semelhante foi inspirado em considerações de marketing, em vez de um bom julgamento. Quando o JavaScript estava sendo introduzido, a linguagem Java estava sendo muito comercializada e ganhando popularidade. Alguém pensou que era uma boa ideia tentar continuar com esse sucesso. Agora estamos presos ao nome.

Após sua adoção fora do Netscape, um documento padrão foi escrito para descrever a maneira como a linguagem JavaScript deveria funcionar, de modo que os vários softwares que alegavam oferecer suporte ao JavaScript estavam falando sobre o mesmo idioma. Isso é chamado de padrão ECMAScript, após a organização Ecma International que fez a padronização. Na prática, os termos ECMAScript e JavaScript podem ser usados de forma intercambiável - são dois nomes para o mesmo idioma.

Há quem diga coisas *terríveis* sobre JavaScript. Muitas dessas coisas são verdadeiras. Quando fui obrigado a escrever algo em JavaScript pela primeira vez, rapidamente passei a desprezá-lo. Aceitaria quase tudo que eu digitasse, mas interpretaria de uma maneira completamente diferente do que eu quis dizer. Isso tinha muito a ver com o fato de eu não ter idéia do que estava fazendo, é claro, mas há um problema real aqui: o JavaScript é ridiculamente liberal no que permite. A idéia por trás desse design era facilitar a programação em JavaScript para iniciantes. Na realidade, isso dificulta a localização de problemas em seus programas, porque o sistema não os indicará para você.

Essa flexibilidade também tem suas vantagens, no entanto. Isso deixa espaço para muitas técnicas impossíveis em linguagens mais rígidas e, como você verá (por exemplo, no [Capítulo 10](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2010%20-%20M%C3%B3dulos.md ), pode ser usado para superar algumas das deficiências do JavaScript. Depois de aprender o idioma corretamente e trabalhar com ele por um tempo, aprendi a *gostar do* JavaScript.

Houve várias versões do JavaScript. A versão 3 do ECMAScript foi a versão amplamente suportada no momento da ascensão do JavaScript ao domínio, aproximadamente entre 2000 e 2010. Durante esse tempo, estava em andamento o trabalho em uma versão ambiciosa 4, que planejava uma série de melhorias e extensões radicais no idioma. Mudar uma linguagem viva e amplamente usada de maneira tão radical acabou sendo politicamente difícil, e o trabalho na versão 4 foi abandonado em 2008, levando a uma versão 5 muito menos ambiciosa, que fez apenas algumas melhorias incontroversas, lançadas em 2009 Então, em 2015, saiu a versão 6, uma grande atualização que incluía algumas das ideias planejadas para a versão 4. Desde então, recebemos novas e pequenas atualizações todos os anos.

O fato de o idioma estar evoluindo significa que os navegadores precisam acompanhar constantemente; se você estiver usando um navegador antigo, talvez ele não seja compatível com todos os recursos. Os designers de idiomas têm o cuidado de não fazer alterações que possam interromper os programas existentes, para que novos navegadores ainda possam executar programas antigos. Neste livro, estou usando a versão 2017 do JavaScript.

Navegadores da Web não são as únicas plataformas nas quais o JavaScript é usado. Alguns bancos de dados, como MongoDB e CouchDB, usam JavaScript como linguagem de script e consulta. Várias plataformas para programação de desktop e servidor, principalmente o projeto Node.js. (o assunto do [Capítulo 20](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2020%20-%20Node.js.md ), fornecem um ambiente para programar JavaScript fora do navegador.

## Código e o que fazer com ele

*Código* é o texto que compõe os programas. A maioria dos capítulos deste livro contém bastante código. Acredito que ler e escrever códigos são partes indispensáveis para aprender a programar. Tente não apenas dar uma olhada nos exemplos - leia-os atentamente e entenda-os. Isso pode ser lento e confuso no começo, mas prometo que você entenderá rapidamente. O mesmo vale para os exercícios. Não assuma que você os entende até que você tenha realmente escrito uma solução funcional.

Eu recomendo que você tente suas soluções para exercícios em um intérprete de JavaScript real. Dessa forma, você receberá um feedback imediato sobre se o que está fazendo está funcionando e, espero, ficará tentado a experimentar e ir além dos exercícios.

Ao ler este livro no seu navegador, você pode editar (e executar) todos os programas de exemplo clicando neles.

Se você deseja executar os programas definidos neste livro fora do site do livro, serão necessários alguns cuidados. Muitos exemplos são independentes e devem funcionar em qualquer ambiente JavaScript. Porém, o código nos capítulos posteriores geralmente é escrito para um ambiente específico (o navegador ou o Node.js.) e pode ser executado apenas lá. Além disso, muitos capítulos definem programas maiores, e os trechos de código que aparecem neles dependem um do outro ou de arquivos externos. A [sandbox](https://eloquentjavascript.net/code) no site fornece links para arquivos Zip que contêm todos os scripts e arquivos de dados necessários para executar o código em um determinado capítulo.

## Visão geral deste livro

Este livro contém aproximadamente três partes. Os 12 primeiros capítulos discutem a linguagem JavaScript. Os próximos sete capítulos são sobre navegadores da Web e a maneira como o JavaScript é usado para programá-los. Por fim, dois capítulos são dedicados ao Node.js, outro ambiente para programar o JavaScript.

Ao longo do livro, há cinco *capítulos de projetos* , que descrevem programas de exemplo maiores para fornecer uma amostra da programação real. Em ordem de aparência, trabalharemos na construção de um [robô de entrega](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2007%20-%20Projeto%20Um%20Rob%C3%B4.md) , uma [linguagem de programação](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2012%20-%20Uma%20linguagem%20de%20programa%C3%A7%C3%A3o.md) , um [jogo de plataforma](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2016%20-%20%20Projeto%20Um%20Jogo%20de%20Plataforma.md) , um [programa de pintura de pixels](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2019%20%20-%20Projeto%20Um%20editor%20de%20pixel%20art.md) e um [site dinâmico](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2021%20-%20Site%20de%20Compartilhamento%20de%20habilidades.md) .

A parte do idioma do livro começa com quatro capítulos que apresentam a estrutura básica da linguagem JavaScript. Eles introduzem [estruturas de controle](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2002%20-%20Estrutura%20do%20Programa.md) (como a `while`palavra que você viu nesta introdução), [funções](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2003%20-%20Fun%C3%A7%C3%B5es.md) (escrevendo seus próprios blocos de construção) e [estruturas de dados](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2004%20-%20Estruturas%20de%20dados%20objetos%20e%20matrizes.md) . Depois disso, você poderá escrever programas básicos. Em seguida, os capítulos [5](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2005%20-%20Fun%C3%A7%C3%B5es%20de%20ordem%20superior.md) e [6](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2006%20-%20A%20Vida%20Secreta%20dos%20Objetos.md) introduzem técnicas para usar funções e objetos para escrever um código mais *abstrato* e manter a complexidade sob controle.

Após um [primeiro capítulo do projeto](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2007%20-%20Projeto%20Um%20Rob%C3%B4.md) , a parte do idioma do livro continua com capítulos sobre [tratamento de erros e correção de erros](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2008%20-%20Bugs%20e%20erros.md) , [expressões regulares](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2009%20-%20Express%C3%B5es%20regulares.md) (uma ferramenta importante para trabalhar com texto), [modularidade](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2010%20-%20M%C3%B3dulos.md) (outra defesa contra a complexidade) e [programação assíncrona](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2011%20-%20Programa%C3%A7%C3%A3o%20ass%C3%ADncrona.md) (lidando com eventos que levar tempo). O [segundo capítulo do projeto](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2012%20-%20Uma%20linguagem%20de%20programa%C3%A7%C3%A3o.md) conclui a primeira parte do livro.

A segunda parte, capítulos [13](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2013%20-%20JavaScript%20e%20o%20Navegador.md) a [19](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2019%20%20-%20Projeto%20Um%20editor%20de%20pixel%20art.md) , descreve as ferramentas às quais o JavaScript do navegador tem acesso. Você aprenderá a exibir coisas na tela (capítulos [14](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2014%20-%20O%20Modelo%20de%20Objeto%20do%20Documento.md) e [17](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2017%20-%20Desenho%20sobre%20tela.md) ), responder à entrada do usuário ( [capítulo 15](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2015%20-%20Tratamento%20de%20eventos.md) ) e comunicar-se pela rede ( [capítulo 18](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2018%20-%20%20HTTP%20e%20formul%C3%A1rios.md) ). Há novamente dois capítulos de projetos nesta parte.

Depois disso, o [capítulo 20](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2020%20-%20Node.js.md) descreve o Node.js e o [capítulo 21](https://github.com/HeltonMulinaria/Eloquent-Javascript-3Ed/blob/master/capitulos/Cap%C3%ADtulo%2021%20-%20Site%20de%20Compartilhamento%20de%20habilidades.md) cria um site pequeno usando essa ferramenta.

## Convenções tipográficas

Neste livro, o texto escrito em uma `monospaced`fonte representará elementos de programas - algumas vezes são fragmentos autossuficientes e outras apenas se referem a parte de um programa próximo. Os programas (dos quais você já viu alguns) são gravados da seguinte maneira:

```js
function factorial(n) {
  if (n == 0) {
    return 1;
  } else {
    return factorial(n - 1) * n;
  }
}
```

Às vezes, para mostrar a saída que um programa produz, a saída esperada é gravada depois dele, com duas barras e uma seta na frente.

```js
console.log(factorial(8));
// → 40320
```

Boa sorte!