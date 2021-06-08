# Tutorial 11 - Formulários na web

Durante a aula de webscraping, trabalhamos com exemplos nos quais retiramos informação de páginas em html, mas não enviamos nenhuma informação ao servidor com as quais estamos nos comunicando (exceto manualmente). Contudo, é muito comum nos depararmos com formulários em páginas que queremos raspar. Formulários, na maioria das vezes, aparecerão como caixas de consulta acompanhada de um botão.

Mecanismos de busca (Google, DuckDuckGo, etc) têm formulários nas suas páginas iniciais. Portais de notícia ou de Legislativos têm formulários de busca (como o que usamos manualmente no caso da Folha de São Paulo). Por vezes, mesmo para "passar" de página nos deparamos com um formulário.

Neste tutorial vamos aprender a preencher um formulário, enviá-lo ao servidor da página e capturar sua resposta.

Faremos um exemplo simples, o buscador da ALESP.

## _rvest_, formulários e ALESP

Vamos começar carregando os pacotes _rvest_, *httr*, _dplyr_, _tidyr_ e *stringr*:

```{r}
library(rvest)
library(httr)
library(dplyr)
library(tidyr)
library(stringr)
```

Nosso primeiro passo ao lidar com formulários será estabelecer uma conexão com o servidor, antes mesmo de capturar a página na qual o formulário está. Damos a essa conexão o nome de "session", e utilizamos a função _session_ para criá-la observamos a página em que preenchemos o formulário para a busca observável [aqui](https://www.al.sp.gov.br/alesp/pesquisa-proposicoes/). Vamos ver como funciona?

```{r}
alesp_url <- "https://www.al.sp.gov.br/alesp/pesquisa-proposicoes/"

alesp_session <- session(alesp_url)
```

Estabelecida a conexão, precisamos conhecer o formulário. Começamos, obviamente, obtendo o código HTML da página do formulário. Na sequência, vamos extrair do código da página os formulários. Como tabelas em HTML, fomrulários tem suas tags próprias e contamos no pacote _rvest_ com uma função que extrai uma lista contendo todos os formulários da página, a *html\_form*:

```{r}
alesp_pagina <- read_html(alesp_session)

alesp_form_list <- html_form(alesp_pagina)

alesp_form_list
```

No caso do buscador da ALESP, há três formulário na página. Com dois colchetes, extraímos o formulário que está na terceira posição da lista de formulários.

```{r}
alesp_form <- alesp_form_list[[3]]

class(alesp_form)

alesp_form
```

Examine o objeto que contém o formulário. Ele é um objeto da classe "form" e podemos observar todos os parâmetros que o compõe, ou seja, tudo aquilo que pode ser preenchido para envio ao servidor, ademais dos botões de submissão.

Vá para o navegador e inspecione a caixa de busca. Você observará que cada "campo" do formulário é uma tag "input". O atributo "type", define se será oculto ("hidden"), texto ("text"), seleção de novas entradas ("select") ou botão de submissão ("button").

Alguns "inputs" já contêm valores (no atributo "values"). No nosso exemplo, os botões são destacados. Temos, por exemplo, qual o número de linhas por página ("rowsPerPage") definida com 20 resultados.

O que nos interesse preencher, obviamente, é o "input" chamado "text". Para capturar mais dados, podemos indicar quantos resultados queremos por página com "rowsPerPage". Vamos manter a busca por "coronavirus" e limitar a 400 resultados.

Vamos, então, preencher os campos com a função _html\_form\_set_:

```{r}
alesp_form <- html_form_set(alesp_form,
                          'text' = "coronavirus",
                          'rowsPerPage' = 400)
```

Simples, não? Colocamos o objeto do formulário no primeiro parâmetro da função e os campos a serem preenchidos na sequência, tal como no exemplo.

Reexamine agora o formulário. Você verá que "text" está preenchido e "rowsPerPage" teve o número de resultados alterado:

```{r}
alesp_form
```

Legal! Agora vamos fazer a submissão do formulário. Na *session\_submit*, precismos informar a sessão que criamos (conexão com o servidor), o formulário que vamos submeter e o nome do botão de submissão. No nosso exemplo, no entando, o botão de submissão é identificado como "NULL". Veja o exemplo: 

```{r}
alesp_submission <- session_submit(x = alesp_session,
                                   form = alesp_form,
                                   submit = NULL)
```

Pronto! Agora basta raspar o resultado como já haviámos feito antes. A página que queremos raspar é o objeto que resulta da função _submit\_form_. Abra o [resultado de uma busca na ALESP](https://www.al.sp.gov.br/spl_consultas/consultaProposicoesAction.do;jsessionid=B99CD5DD1520AF721E286203683CD227?direction=inicio&lastPage=0&currentPage=0&act=detalhe&rowsPerPage=400&currentPageDetalhe=1&method=search&text=coronavirus&legislativeNumber=&legislativeYear=&anoDeExercicio=&strInitialDate=&strFinalDate=&advancedSearch=S) e tente entender o código abaixo.

```{r}
dados <- alesp_submission %>% 
  read_html()  %>% 
  html_nodes(xpath = '//*[@id="lista_resultado"]/table') %>% 
  html_table() %>% 
  magrittr::extract2(1) %>% 
  filter(Autor != "") %>%
  separate("Documento", into = c("Documento", "Ementa"), 
           sep = "\r\n\t\t\t\t\t\t\r\n\t\t\t\t\t\t\r\n\t\t\t\t\t") %>% 
  mutate(Documento = str_squish(Documento)) %>% 
  mutate(links = read_html(alesp_submission) %>% 
           html_nodes(xpath = '//*[@id="lista_resultado"]/table/tbody/tr/td/a[@style="color: #000000"]') %>% 
           html_attr(name = 'href'),
         links = paste0("https://www.al.sp.gov.br/", links))
```

## Obtendo informações sobre processos no STF

Diferentemente dos Executivos e Legislativos no Brasil, os órgãos do Judiciário, em diversas esferas, têm avançado bem pouco em transparência e no estabelecimento de política de dados abertos. O STF não é exceção. A despeito do grande número de pesquisa sobre o STF, ainda é preciso raspar os dados diretamente do potal do Tribunal para construir bases de dados sobre processos que lá tramitam.

Vamos rapidamnte ver um exemplo de como obter dados do STF.

O primeiro passo será encarar este formulário [aqui](http://www.stf.jus.br/portal/processo/pesquisarProcesso.asp). Queremos buscar os processos por seu número (afinal de contas, no futuro vamos pegar todos os processos de um determinado tipo de 1 até "n"). O formuláio está logo no centro da página.

Vamos proceder como fizemos no caso do buscador da ALESP. Começaremos estabelecendo uma sessão (conexão) com o servidor. A seguir, vamos raspar a página que contém o formulário e produzir uma lista de fomulários. No entanto, precisamos, aqui, indicar quais as versões de navegadores pelos quais podemos fazer a pesquisa. Para isso criaremos o vetor *uastring*:

```{r}
uastring <- "Mozilla/89.0 Chrome/91.0.4472.77"

stf_url <- "http://www.stf.jus.br/portal/processo/pesquisarProcesso.asp"

stf_session <- session(stf_url, user_agent(uastring))

stf_pagina <- read_html(stf_session)

stf_form_list <- html_form(stf_pagina)

stf_form_list
```

Veja que há três formulários diferentes na página. Como decidir entre eles? Precisamos examinar o código HTML. Em geral, inspecionando o campo da busca já teremos um indicativo de qual é o formulário que nos interessa.

Neste caso, queremos fazer a busca por número de processo e o campo de busca se chama "numero". Qual é o formulário que contém tal informação? O terceiro:

```{r}
stf_form <- stf_form_list[[3]]

stf_form
```

Escolhido o formulário, precisamos preenchê-lo. Aqui nos depararemos com um novo problema: como saber qual campo é de preenchimento obrigatório? Esta informação pode esta até visível na página, e esta será nossa primeira pista. No caso do STF, não está. "dropmsgoption", por exemplo, é um campo obrigatório que ainda não está preenchido no formulário que capturamos. Tentativa e erro é o recurso final e tente se colocar no lugar de quem criou o formulário para preenchê-lo.

Note que o campo "dropmsgoption" é uma tag "select" e não "input". As tags "select" vêm acompanhadas, em geral, de suas opções, que relacionam o texto da opção com seu código. No nosso caso, queremos o código "1!

Vamos, então, preencher os dois parâmetos do formulário. Por uma razão que não convém explicar, vamos omitir na nossa busca o nome do processo (ex: ADI, AC, Pet, etc). Buscaremos apenas o número.

```{r}
stf_form <- html_form_set(form = stf_form,
                          'dropmsgoption' = 1,
                          'numero' = "1")

stf_form                  
```

Note agora que o botão de preenchimento do formulário não tem nome. Não tem problema. A função _submit\_form_ é bastante inteligente e procurará o campo de submissão do formulário se você não preenhcer o parâmetro "submit" (e vai apontar o nome dele e m uma mensagem. Da mesma forma, se você escrever um nome inexistente, receberá uma mensagem de erro com todos os nomes possíveis dos campos de submissão.

```{r}
stf_submission <- session_submit(x = stf_session,
                                 form = stf_form)
```

Faça o mesmo processo manualmente. O resultado é uma tabela com os links para todos os diferentes tipos de processos com o número buscado. Podemos extrair a tabela (você já sabe fazer isso):

```{r}
tabela_processos <- stf_submission %>% 
  read_html() %>% 
  html_table() %>% 
  magrittr::extract2(., (1))
```

E os links dos processos (você também sabe fazer isso). Os links precisarão de um pouco de limpeza e utilizaremos a função _str\_sub_ do pacote _stringr_ para resolver este problema. Já criaremos a variável dentro do próprio *data_frame*.

```{r}
tabela_processos <- tabela_processos %>% 
  mutate(links = stf_submission %>% 
           read_html() %>% 
           html_nodes(xpath = "//table//a") %>% 
           html_attr(name = "href"),
         links = str_squish(links),
         links = paste0("https://www.stf.jus.br/portal/processo/", links))
```

Excelente! Podemos escolher o link de um processo usando a tabela (ADI, por exemplo):

```{r}
link_adi <- tabela_processos$links[str_detect(tabela_processos$Processo, "ADI")]
```

Podemos, primeiro, criar um loop para passar por páginas com o número dos processos e não somente 1.

No fim, é possível executar loops para extração nas páginas de cada um dos processos identificando os padrões de *xpath* para cada informação. Caso queira, utilize como desafio para treinar suas habilidades nesses loops de raspagem.

Dica: as páginas estão em tabelas, mas para cada tipo de processo a estrutura da tabela e dos *xpaths* varia.