
### Note

Este trecho foi anotado durante os estudos e nao substitui a documentacao oficial.

Alterando o RottenPotatoes
==========================

# Introdução ao Rails: acrescentar recursos ao RottenPotatoes

Nesta tarefa você vai acrescentar um recurso a um aplicativo Rails existente e publicar o resultado num serviço de hospedagem na nuvem. Serão executados testes de integração contra a versão publicada.

Conselho geral: o trabalho mexe em várias partes do aplicativo. Use controle de versão a favor: faça commit com frequência, para não perder o que já funcionava. Assim sempre dá para voltar a uma revisão anterior ou comparar o que mudou desde o último commit bom.

**Em resumo: faça commits cedo e sempre.**

## Preparação: rodar o RottenPotatoes na máquina

O aplicativo inicial usado aqui está em outro repositório público. Faça um fork dele e clone o seu fork na máquina de desenvolvimento:

`git clone git@github.com:`seu-usuario`/rottenpotatoes-rails-intro`

Sempre que começar a mexer num projeto Rails, o primeiro passo é rodar o gerenciador de dependências para garantir que as bibliotecas estão instaladas. Entre na raiz do aplicativo e rode o comando de instalação sem o grupo de produção; essa opção só precisa ser informada na primeira vez, porque fica memorizada nas execuções seguintes.

Por fim, crie o banco local:

`rake db:migrate`

* Autoverificação: como o framework decide onde e como criar o banco de desenvolvimento? (dica: olhe a pasta de banco do projeto)

Isso cria o banco de desenvolvimento e executa as migrações que geram o esquema. Também gera o arquivo de esquema refletindo a versão mais recente. **Esse arquivo deve ir para o controle de versão.**

* Autoverificação: quais tabelas as migrações criaram?

...E insira os dados iniciais, aqueles que o aplicativo precisa para funcionar:

`rake db:seed`

* Autoverificação: quais dados foram inseridos e onde eles estão definidos? (dica: a listagem de tarefas do rake explica tanto a tarefa de dados iniciais quanto as demais)

Nesse ponto já dá para subir o servidor local e abrir a listagem de filmes no navegador. Se estiver usando um ambiente na nuvem, suba com porta e endereço informados e abra o link gerado.

## Preparação: publicar na plataforma de hospedagem

Se você já publicou lá antes, basta criar um novo contêiner de aplicação pelo utilitário de linha de comando. Se for a primeira vez, faça duas coisas: crie uma conta gratuita e configure as chaves de acesso seguro usadas na publicação. Os três comandos básicos são gerar a chave, entrar na conta e cadastrar a chave; a página da plataforma detalha cada passo.

Depois da configuração inicial, crie o contêiner de aplicação onde o projeto será publicado.

A plataforma atribui um nome aleatório ao aplicativo; depois de publicado, ele fica acessível no endereço formado por esse nome. Pelo site dá para trocar o nome.

Por fim, envie o código:

`git push heroku master`

(É normal aparecer o aviso de autenticidade do servidor na primeira vez — responda que sim; nas próximas não aparece mais.)

O aplicativo já está no ar? Ainda não: assim como rodamos as migrações e os dados iniciais localmente, é preciso criar o banco do lado da hospedagem:

`heroku run rake db:migrate`
e
`heroku run rake db:seed`

Agora dá para abrir o endereço do aplicativo. Existe um comando que abre o navegador direto nesse endereço, caso você tenha esquecido.

**Nota:** não siga adiante antes de concluir essa etapa com sucesso, ou não conseguirá terminar a tarefa!

# Parte 1: ordenar a lista de filmes (15 pontos)

Na página que lista todos os filmes, transforme os títulos das colunas de título e de data de lançamento em links. Clicar em um deles recarrega a lista ordenada de forma crescente por aquela coluna: por data, os lançamentos mais antigos primeiro; por título, em ordem alfabética. Para títulos que começam com caracteres não alfabéticos, a ordem deve seguir o comportamento de comparação padrão da linguagem.

Quando a lista é exibida com ordenação ativa, o cabeçalho escolhido deve aparecer com fundo amarelo, como na imagem. Para isso, defina variáveis no controlador que condicionem a classe de estilo do cabeçalho e acrescente a regra de estilo indicada ao arquivo de folha de estilos do aplicativo.

O resultado deve ficar parecido com a captura de tela do enunciado.

**Importante para a correção:**

O link de ordenação por título deve ter o identificador `title_header`, e o de data de lançamento o identificador `release_date_header`. A tabela que contém a lista deve ter o identificador `movies`, já definido no código inicial.

## Dicas e armadilhas:

* As telas atuais usam o auxiliar de rotas baseadas em recurso para gerar o endereço da listagem. Vale saber que, se você passar um hash de parâmetros extras para esse auxiliar, o framework os coloca no hash de parâmetros da requisição.

* Bancos de dados são ótimos em devolver coleções ordenadas por um ou mais atributos. Antes de ordenar na mão o que veio do banco, consulte a documentação do método de consulta e veja se o banco pode fazer esse trabalho.

* Nada de código nas telas! A tela não deve ordenar coleção nenhuma: o papel dela é só exibir. O controlador entrega à tela exatamente o que deve aparecer.

## Entrega

Essa parte é entregue depois de publicar na hospedagem e informar o endereço na parte 3.

Por enquanto, grave tudo o que fez e publique para conferir que funciona antes de seguir:

```
git commit -a -m "tarefa 2 parte 1 concluida"
git push heroku master
```

# Parte 2: filtrar a lista por classificação (15 pontos)

Melhore o aplicativo assim: no topo da listagem, acrescente caixas de seleção que permitam mostrar apenas filmes com determinadas classificações indicativas.

Com o botão de atualizar pressionado, a lista é reexibida apenas com os filmes cujas classificações estavam marcadas.

Isso exige alguns trechos de código. O formulário que gera as caixas já está pronto para ser incluído na tela de listagem:

```
= form_tag movies_path, :method => :get do
  Include:
  - @all_ratings.each do |rating|
    = rating
    = check_box_tag "ratings[#{rating}]"
  = submit_tag 'Refresh'
```

Só que esse código espera uma variável com a coleção de todas as classificações possíveis, algo como as quatro faixas usuais. Quem precisa preparar essa variável é o controlador — e, como os valores são responsabilidade do modelo, o ideal é que o controlador obtenha a coleção por meio de um método de classe do modelo.

Também é preciso descobrir quais caixas o usuário marcou e restringir a consulta ao banco com base nisso.

Sobre o primeiro ponto: ao ver o código-fonte da página com o formulário, note que as caixas têm nomes no formato de hash. Esse truque faz o framework agrupar os valores marcados num único hash, cujas chaves são só as caixas assinaladas. Ou seja, se o usuário marcar duas faixas, os parâmetros trarão um hash com essas duas chaves e valor padrão. Como os valores não interessam aqui, basta extrair as chaves do hash — caixas não marcadas nem aparecem nos parâmetros.

Sobre o segundo ponto: provavelmente você vai trocar a consulta que traz tudo por uma consulta com restrições, que aceita opções para limitar o resultado.

## Importante para a correção

* O formulário deve ter o identificador `ratings_form`.
* O botão de envio do filtro deve ter o identificador `ratings_submit`.
* Cada caixa deve ter um identificador no formato `ratings_` seguido da classificação, como `ratings_PG-13`, `ratings_G` e assim por diante.

## Dicas e armadilhas

Cuidado para não quebrar a ordenação por coluna já implementada! Ou seja, ordenar pelos cabeçalhos deve continuar funcionando, e se o usuário clicar no cabeçalho de título depois, o resultado deve vir ordenado (nesse momento ainda não precisa respeitar o filtro; isso fica para a parte 3).

Se o usuário marcar algumas caixas e reexibir a lista, as mesmas caixas devem voltar marcadas. Isso exige um pequeno ajuste no formulário fornecido.

Na primeira visita, todas as caixas devem vir marcadas, para o usuário ver todos os filmes. Por enquanto ignore o caso de desmarcar todas; ele aparece na próxima parte.

Lembrete: nada de código nas telas! Guarde numa variável de instância do controlador quais classificações foram realmente usadas no filtro e deixe-a disponível para a tela marcar as caixas corretas ao recarregar.

Essa parte também é entregue depois da publicação e da informação do endereço na parte 3. Mas dá para gravar tudo e publicar para conferir antes de seguir.

# Parte 3: lembrar as preferências de ordenação e filtro (70 pontos)

Até aqui o usuário ordena pelos cabeçalhos e filtra pelas caixas, e a propriedade REST foi preservada, porque o próprio endereço carrega os parâmetros que definem ordenação e filtro.

O último passo é memorizar isso. Se o usuário escolheu uma combinação de ordenação e filtro e depois abriu os detalhes de um filme, ao voltar para a lista as mesmas preferências devem valer.

(Sair da lista para ver detalhes é só um exemplo; as preferências devem valer não importa o caminho, de modo que qualquer visita à listagem já venha com elas aplicadas.)

A melhor forma de memorizar é usar o armazenamento de sessão. Ele funciona como a mensagem temporária de uma requisição, com a diferença de que o valor posto na sessão dura até ser limpo ou removido item a item. Assim, no método de listagem, dá para aplicar as preferências guardadas mesmo quando o endereço acessado não traz os parâmetros.

## Dicas e armadilhas

Se o usuário informar novas preferências na requisição, a sessão não deve sobrescrevê-las; as novas é que passam a ser lembradas.

Se o usuário desmarcar todas as caixas, use o que está na sessão, porque desmarcar tudo não faz sentido prático.

Para continuar REST, um endereço que resulta numa listagem ordenada ou filtrada deve conter os parâmetros correspondentes. Então, se a requisição vier sem eles e você precisar preencher a partir da sessão, o correto é redirecionar para o endereço já com os parâmetros. Existe um caso especial aqui: se a ação anterior tinha deixado uma mensagem temporária para exibir após o redirecionamento, esse redirecionamento extra apaga a mensagem, porque ela só sobrevive a um redirecionamento. Para resolver, preserve a mensagem antes do redirecionamento extra.

## Ao terminar esta parte

Publique do mesmo jeito de antes:

```
git commit -a -m "tarefa 2 parte 3 concluida"
git push heroku master
```

# Como entregar no final

Publicar o aplicativo na hospedagem até o prazo faz parte da avaliação. Mesmo com o código gravado e funcionando, é preciso publicar para receber a nota integral.

Quando tiver certeza de que funciona na hospedagem, envie num arquivo de texto contendo apenas o endereço do aplicativo publicado.

**Atenção**: use **http** e não **https**, ou seja, envie `http://seu-app.herokuapp.com` e **não** `https://seu-app.herokuapp.com`.