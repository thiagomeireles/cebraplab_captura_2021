# Tutorial 5: Manipulação de texto no R: pacote _stringr_

## Material para o tutorial

Nossa primeira tarefa será obter um conjunto de textos com o qual trabalharemos. Classicamente, tutoriais de R sobre strings e mineração de texto utilizam "corpus" (já veremos o que é isso) de literatura moderna.

Para tornar nosso exemplo mais interessante, vamos recuperar as notícias da Folha de São Paulo que criamos a partir do Tutorial 3. A primeira ferramenta que veremos neste tutorial é o pacote *stringr*. Além de carregarmos o pacote *strigr*, também utilizaremos os pacotes *rvest* e *dplyr*. Para abrir o arquivo com os dados que utilizaremos, também utilizaremos o pacote *readr*:

```{r}
library(dplyr)
library(rvest)
library(stringr)
library(readr)
```

Feito isso, vamos utliizar os dados coletados no Tutorial 3 com as 300 primeiras entradas na busca de notícias da Folha pelo termo "coronavirus". Para não realizarmos o scrap mais uma vez, vamos abrir um ".csv" que está disponível na página do curso no GitHub com a função *read_delim*:

```{r}
dados_noticias <- read_delim("https://raw.githubusercontent.com/thiagomeireles/cebraplab_captura_2021/main/dados/dados_noticias.csv", 
                             delim = ",", locale = locale(encoding = "WINDOWS-1252"))
```

Aproveitemos para ver algumas funções novas do pacote *stringr*. Várias delas, como veremos, são semelhantes a funções de outros pacotes com as quais já trabalhamos. Há, porém, algumas vantagens ao utilizá-las: bugs e comportamentos inesperados corrigidos, uso do operador "pipe", nomes intuitivos e sequência de argumentos intuitivos.

Para isso, vamos aproveitar e adequar a variável de data retirando o horário da publicação? O primeiro passo é criar uma nova variável somente com a informação da data. Para isso, utilizaremos a função *str_extract_all*. Ela passa por todos os valores do vetor indicado e extrai somente o texto que indicamos. Aqui informaremos que queremos todo o texto até determinado padrão, no caso " às" que será nossa "âncora", mantendo somente a data.

```{r}
dados_noticias <- dados_noticias %>% 
  mutate(dia = str_extract_all(datahora, ".+?(?= às)")) %>% 
  select(-datahora)
```

Outra função bastante importante é a *str_replace_all* que substitui no texto um padrão por outro, respectivamente na sequência de argumentos. Seu uso é semelhante à função *gsub*, mas os argumentos estão em ordem intuitiva. Primeiro o vetor, o padrão que queremos substituir e a substituição. Vamos facilitar nossa vida substituindo o ".mai." pelo número do mês (05), o que permitirá a conversão para data.

```{r}
dados_noticias <- dados_noticias %>% 
  mutate(dia = str_replace_all(dia, ".mai.", "/05/"))
```

O último passo é transformar as variáveis em tipo "date". Para isso utilizaremos a função *strptime* indicando qual a ordem apresentada na variável (dia, mês e ano):

```{r}
dados_noticias <- dados_noticias %>% 
  mutate(dia = strptime(dia, "%d/%m/%Y"))
```

## Funcionalidades do *stringr*

Antes de observar as funcionalidades do pacote *stringr*, vamos extrair o texto das pubicações para um vetor com o qual vamos trabalhar a partir de agora no tutorial.

```{r}
noticias <- dados_noticias$texto
```

Qual é o tamanho de cada publicação? Vamos aplicar *str_length* para descobrir dos 100 primeiros textos. Seu uso é semelhante ao da função *nchar*:

```{r}
len_noticias <- str_length(noticias[1:100])
len_noticias
```

Vamos agora observar quais são os noticias que mencionam os termos "Bolsonaro" e "Lula". Para tanto, usamos *str_detect*

```{r}
str_detect(noticias, "Bolsonaro")
str_detect(noticias, "Lula")
```

Poderíamos usar o vetor lógico resultante para gerar um subconjunto dos noticias, apenas com aqueles nos quais as palavras "Bolsonaro" e "Lula" são mencionadas. Mais simples, porém, é utilizar a função *str_subset*, que funciona tal qual *str_detect*, mas resulta num subconjunto em lugar de um vetor lógico:

```{r}
noticias_bolsonaro <- str_subset(noticias, "Bolsonaro")
noticias_lula      <- str_subset(noticias, "Lula")
```

Se quisessemos apenas a posição no vetor dos noticias que contêm "Bolsonaro", *str_which* faria o trabalho:

```{r}
str_which(noticias, "Bolsonaro")
str_which(noticias, "Lula")
```

Voltando ao vetor completo, quantas vezes "Bolsonaro" é mencionada em cada noticias? E Lula? Qual é o máximo de menções a "Bolsonaro" em uma única publicação? E Lula

```{r}
str_count(noticias, "Bolsonaro")
max(str_count(noticias, "Bolsonaro"))
str_count(noticias, "Lula")
max(str_count(noticias, "Lula"))
```

Vamos fazer uma substituição nos noticias. No lugar de "Bolsonaro" colocaremos a expressão "Bolsonaro, atual presidente do Brasil,". E no lugar de "Lula", "Lula, potencial candidato em 2022," Podemos fazer a substituição com *str_replace* ou com *str_replace_all*. A diferença entre ambas é que *str_replace* substitui apenas a primeira ocorrênca encontrada, enquanto *str_replace_all* substitui todas as ocorrências. Façamos para as três primeiras notícias de cada caso.

```{r}
str_replace(noticias_bolsonaro[1:3], "Bolsonaro", "Bolsonaro, atual presidente do Brasil,")
str_replace_all(noticias_bolsonaro[1:3], "Bolsonaro", "Bolsonaro, atual presidente do Brasil,")

str_replace(noticias_lula[1:3], "Lula", "Lula, potencial candidato em 2022,")
str_replace_all(noticias_lula[1:3], "Lula", "Lula, potencial candidato em 2022,")
```

Em vez de substituir, queremos conhecer a posição das ocorrências de "Bolsonaro" e de "Lula". Com *str_locate* e *str_locate_all*, respectivamente para a primeira ocorrência e todas as ocorrências, obtemos a posição de começo e fim do padrão buscado:

```{r}
str_locate(noticias_bolsonaro, "Bolsonaro")
str_locate_all(noticias_bolsonaro, "Bolsonaro")

str_locate(noticias_lula, "Lula")
str_locate_all(noticias_lula, "Lula")
```

Finalmente, notemos que os noticias começam sempre mais ou menos da mesma forma. Vamos retirar os 100 primeiros caracteres de cada publicação para observá-los. Usamos a função *str_sub*, semelhante à função *substr*, para extrair um padaço de uma string:

```{r}
str_sub(noticias[1:100], 1, 130)
```

As posições para extração de exerto podem ser variáveis. Por exemplo, vamos usar "len_noticias" que criamos acima para extrair os 50 últimos caracteres de cada publicação:

```{r}
str_sub(noticias[1:100], (len_noticias - 50), len_noticias)
```

Infelizmente, não há tempo suficiente para entrarmos neste tutorial em um tema extremamante útil: expressões regulares. Expressões regulares, como podemos deduzir pelo nome, são expressões que nos permite localizar -- e, portanto, substituir, extrair, parear, etc -- sequências de caracteres com determinadas caraterísticas -- por exemplo, "quaisquer caracteres entre parênteses", ou "qualquer sequência entre espaços que comece com 3 letras e termine com 4 números" (placa de automóvel).

Você pode ler um pouco sobre expressões regulares no R [aqui](https://rstudio-pubs-static.s3.amazonaws.com/74603_76cd14d5983f47408fdf0b323550b846.html) em aula se terminar os três tutoriais de hoje. Com o uso de expressões regulares, outras são bastante úteis, como *str_extract*, *str_match* e *str_match_all*.

Expressões regulares não são simples de trabalhar, assim é sempre bom ter um guia para nos ajudar. O essencial pode ser encontrando nesta [Cheatsheet](https://github.com/rstudio/cheatsheets/raw/master/strings.pdf), consulte sempre que precisar.

## Nuvem de Palavras

Com a função *wordcloud* do pacote de mesmo nome, podemos rapidamente visualizar as palavras discursadas tendo o tamanho como função da frequência (vamos limitar a 50 palavras):

```{r, eval=FALSE}
install.packages(c("slam", "tm", "wordcloud"))
```

```{r}
library(wordcloud)
wordcloud(noticias, max.words = 50)
wordcloud(noticias_lula, max.words = 50)
wordcloud(noticias_bolsonaro, max.words = 50)
```

Estão longe de muito bonitas e com diversas palavras que não fazem muito sentido, não? Voltaremos a fazer nuvens de palavras depois de aprendermos outras maneiras de trabalharmos o texto como dado no R.
