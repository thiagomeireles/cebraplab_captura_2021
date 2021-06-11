# Tutorial 9: Modelos de análise de texto

## Carregando pacotes

Temos três novos pacotes novo neste tutorial, o *quanteda.textmodels*, o *quanteda.textstats* e o *stm*. Vamos instalá-lo: 

```{r, eval=FALSE}
install.packages("quanteda.textmodels")
install.packages("quanteda.textstats")
install.packages("stm")
```

Agora definimos os pacotes que queremos carregar e chamamos na biblioteca

```{r}
library(quanteda)
library(quanteda.textstats)
library(quanteda.textmodels) 
library(quanteda.textplots)
library(dplyr)
library(stringr)
library(readr)
library(ggplot2) 
library(viridis)
library(stm)
```

## Carregando os dados

Faremos o mesmo processo do Tutorial 8, carregando os dados que capturamos na Folha de São Paulo, criamos nosso corpus base e atribuimos os metadados com a função *docvars*:

```{r, message=FALSE, warning=FALSE}
dados_noticias <- read_delim("https://raw.githubusercontent.com/thiagomeireles/cebraplab_captura_2021/main/dados/dados_noticias2.csv", 
                             delim = ";", locale = locale(encoding = "WINDOWS-1252")) %>% 
  filter(!is.na(dia))

corpus_noticias <- corpus(dados_noticias$texto)

docvars(corpus_noticias, "titulo") <- dados_noticias$titulo
docvars(corpus_noticias, "dia")   <- dados_noticias$dia
```

## Análise dos textos: estatísticas textuais

Antes de criar nossa Matriz Documento-Recurso, vamos recriar nosso vetor de *stopwords*:

```{r}
stopwords_pt <- c(stopwords("portuguese"), "sobre", "após", "folha", "nesta", "u", "é", "ser", "r")
```

### Legibilidade dos textos

Um primeiro passo que faremos e é bastante importante é testar a legibilidade dos textos com a função [*textstat_readability*](https://www.rdocumentation.org/packages/quanteda.textstats/versions/0.94.1/topics/textstat_readability).

```{r}
head(textstat_readability(corpus_noticias),2)
```

De forma mais simples, quanto maior o score na escala de Flesch, mais fácil a leitura do texto. No caso dos dois primeiros que utilizamos como exemplo, a leitura é bastante complexa. Um texto de fácil compreensão possui score próximo a 80. Existem outras medidas que são apresentadas na documentação do link acima.

### Frequência dos termos

Aqui um primeiro passo é criar uma Matriz Documento-Recurso removendo as *stopwords*:

```{r}
dfm_noticias <- tokens(corpus_noticias, 
                       remove_punct = TRUE,
                       remove_numbers = TRUE,
                       remove_symbols = TRUE) %>% 
  tokens_tolower() %>% 
  dfm() %>% 
  dfm_remove(stopwords_pt)
```

Para observar a frequência de textos o pacote *quanteda.textstats* oferece a função [*textstat_frequency*](https://www.rdocumentation.org/packages/quanteda.textstats/versions/0.94.1/topics/textstat_frequency). Vamos fazer um gráfico com apenas os 20 termos mais frequentes, usando a função *filter* a partir da variável `rank`. A partir disso, utilizamos o *ggplot2*, reordenando o recurso (feature) no eixo do menos para o mais frequente, utilizando a frequência no eixo y e preenchendo com o próprio recurso (diferenciação de cores entre cada um). Mais uma vez utilizamos o *geom_bar* com a opção *stat* definida como `identity` porque fornecemos os valores do eixo y. A função *scale_fill_viridis_d* utiliza uma paleta de cores direcionada para daltônicos (isso é interessante saber, não?) e o "d" no final indica que é a opção para variáveis discretas. A função *coord_flip* troca o eixo x e y de posição. Por fim, de novidade, temos a *legend.position* como "none" para retirar a legenda e alteramos o tamanho da fonte do eixo "x" com a *asis.text*.

```{r}
textstat_frequency(dfm_noticias) %>% 
  filter(rank<=20) %>% 
  ggplot(aes(x = reorder(feature, rank), y = frequency, fill = feature)) +
  geom_bar(stat="identity") +
  scale_fill_viridis_d() + 
  labs(x = "Termos",
       y = "Frequência") +
  coord_flip() +                           # Inverte a coordenada entre eixos x e y
  theme_bw() + 
  theme(legend.position = "none",          # Remove a legenda
        axis.text=element_text(size=10))   # Altera o tamanho da fonte
```

### Análise de n-gramas

Antes de fazer a análise, criaremos um novo objeto da classe *token* a partir do `corpus_noticias` removendo as pontuações e os símbolos. A partir do *pipe* faremos duas etapas de remoção: na primeira removeremos as *stopwords_pt* (aqui especificadas no padrão) e na segunda os números a partir de uma [expressão regular](https://rstudio.com/wp-content/uploads/2016/09/RegExCheatsheet.pdf).

```{r}
tokens_noticias <- tokens(corpus_noticias, 
                           remove_punct = TRUE,
                           remove_numbers = TRUE,
                           remove_symbols = TRUE) %>% 
  tokens_tolower() %>% 
  tokens_remove(stopwords_pt) %>% 
  tokens_remove(pattern = '[[:digit:]]', valuetype = "regex", padding = TRUE) 
```

O pacote *quanteda.textstats* oferece a função [*textstat_collocations*](https://www.rdocumentation.org/packages/quanteda.textstats/versions/0.94.1/topics/textstat_collocations) para análise de bigramas e n-gramas. Aqui utilizaremos o objeto *token* criado trabalhando com n-gramas de 2 a 4 termos (opção *size*). A partir disso, selecionaremos as 20 combinações mais frequentes para fazermos um gráfico. No ggplot, na aesthetics utilizaremos a variável dos n-gramas, `collocation`, no eixo x de forma ordenada, e a contagem, `count`, no eixo y. Mais uma vez criamos o gráfico de barras preenchendo mais uma vez com os n-gramas. As outras opções são as mesmas utilizadas no gráfico anterior.

```{r}
textstat_collocations(tokens_noticias, size = 2:4, tolower = TRUE) %>%
  slice_max(count, n = 20) %>% 
  ggplot(aes(x=reorder(collocation, count), y = count)) +
  geom_bar(stat = "identity", aes(fill=collocation)) +
  scale_fill_viridis_d() +
  labs(x = "Termos",
       y ="Frequência") +
  coord_flip() +
  theme_bw() + 
  theme(legend.position = "none", 
        axis.text=element_text(size=10))
```

### Diferença entre textos

Primeiro criaremos uma nova matriz com grupos direcionados, como fizemos com "bolsonaro", "lula", "bolsonaro e lula" e "outros no último tutorial:

```{r}
dados_noticias <- dados_noticias %>% 
  mutate(texto = tolower(texto),
         politico = ifelse(grepl("jair bolsonaro", texto), "bolsonaro", NA),
         politico = ifelse(grepl("lula", texto), "lula", politico),
         politico = ifelse(grepl("jair bolsonaro", texto) & grepl("lula", texto), "bolsonaro e lula", politico),
         politico = ifelse(is.na(politico), "Outros", politico))

docvars(corpus_noticias, "politico") <- dados_noticias$politico

dfm_noticia_politicos <- tokens(corpus_noticias, 
                             remove_punct = TRUE,
                             remove_numbers = TRUE) %>% 
  dfm() %>% 
  dfm_remove(stopwords_pt) %>% 
  dfm_wordstem()
  
dfm_noticia_politicos <- dfm_group(dfm_noticia_politicos, groups = politico)
```

Para testar as diferenças entre os grupos de textos, utilizamos a função *textstat_keyness* que utiliza a técnica chamada [Keyness](https://www.rdocumentation.org/packages/quanteda.textstats/versions/0.94.1/topics/textstat_keyness) para identificar a diferença entre dois corpus ou elementos dentro de um mesmo corpus. Aqui utilizaremos os anos para diferenciar, tendo "lula" como alvo da análise (ou seja, comparando com as outras categorias).

```{r}
tstatkeyness <- textstat_keyness(dfm_noticia_politicos, target = 'lula')
head(tstatkeyness,15)
```

É importante entender que os resultados são sensíveis ao número de obserações tanto para as estatística de chi-quadrado como p-valor, quanto maior os corpus, maior a chance de encontrarmos resultados estatisticamente significantes.

Vamos dar uma olhada no gráfico gerado a partir do resultado obtido com a função [textplot_keyness](http://quanteda.io/reference/textplot_keyness.html) selecionando apenas os 15 casos de maior valor:

```{r}
textplot_keyness(tstatkeyness, n = 15)
```

### Semelhança entre textos

Para ver a semelhança entre os textos, utilizaremos a mesma matriz agrupada pelas categorias do exercício anterior.

Agora realizamos o teste de similaridade aplicado a essas pubicações com o [método de cosseno](https://sites.temple.edu/tudsc/2017/03/30/measuring-similarity-between-texts-in-python/#:~:text=The%20cosine%20similarity%20is%20the,the%20similarity%20between%20two%20documents.). Caso queiram pesquisar sobre os métodos aplicados, olhem a [documentação da função](https://www.rdocumentation.org/packages/quanteda.textstats/versions/0.94.1/topics/textstat_simil). A partir da nossa matriz, vamos comparar as notícias que comparem "lula" e "bolsonaro" com as demais e entre elas próprias.

```{r}
similaridade <- textstat_simil(dfm_noticia_politicos,
                               dfm_noticia_politicos[c("lula", "bolsonaro"), ],
                               margin = "documents",
                               method = "cosine")

head(similaridade)
```

De forma intuitiva, quanto maior o valor (de 0 a 1), mais semelhantes são os discursos. Podemos observar isso graficamente:

```{r}
dotchart(as.list(similaridade)$"lula", xlab = "Similaridade de cosseno (Lula)", pch = 19)
```

Outra possibilidade é a utilização dessas distâncias para produzir um dendograma clusterizando as edições. Utilizaremos *dfm* com as notícias agrupadas pelos políticos. A partir dela, utilizamos a funçao `dfm_trim()` para selecionar os recursos a partir de um "corte" na frequência. Aqui pegamos somente termos que apareçam ao menos 5 vezes e em 2 documentos. 

```{r, eval = FALSE}
trimDfm <- dfm_trim(dfm_noticia_politicos, min_termfreq = 5, min_docfreq = 2)

trimDfm
```

Agora obtemos as distâncias da *dfm* normalizadas com a função [textstat_dist](https://www.rdocumentation.org/packages/quanteda/versions/0.99.12/topics/textstat_dist).

```{r}
tstat_dist <- textstat_dist(dfm_weight(trimDfm, scheme = "prop"))
```

Em seguida, realizamos a clusterização hirárquica do objeto da distância:

```{r}
cluster_politico <- hclust(as.dist(tstat_dist))
```

E atribuímos rótulos com os nomes das edições:

```{r}
cluster_politico$labels <- docnames(dfm_noticia_politicos)
```

Por fim, plotamos o dendograma.

```{r, fig.width = 8, fig.height = 5}
plot(cluster_politico, ylab = "", xlab = "", sub = "",
     main = "Distância Euclidiana da frequência de tokens normalizada")
```

Perceba que é como uma "árvore" em que é possível diferenciar quais "ramos" estão próximos ou não. 

### Dispersão de palavras chave

Com o comando [*texplot_xray*](https://www.rdocumentation.org/packages/quanteda/versions/2.1.2/topics/textplot_xray) conseguimos selecionar padrões de palavras ao longo de um ou mais textos. Com isso, o formato do gráfico depende do número de objetos da classe *kwic* que chamamos. Se utilizarmos apenas um documento, as palavras-chave são chamadas uma abaixo da outra; se chamarmos múltiplos documentos, eles que são colocados um abaixo do outro e as palavras ficam lado a a lado. Aqui chamaremos por 5 palabras chave utilizando os primeiros 15 textos do corpus:

```{r}
textplot_xray(kwic(tokens(corpus_noticias[1:20]), "bolsonaro"), 
              kwic(tokens(corpus_noticias[1:20]), "vacina"))
```
### Diversidade léxica

Aqui veremos como observar a [diversidade léxica ou complexidade dos textos](https://textinspector.com/help/lexical-diversity/). Com a função *textstat_lexdiv* acessamos nossa matriz recurso para ver quantas palavras lexicalmente diferentes temos nos textos. Aqui mantivemos todas as medidas possíveis pela função, pode explorar um pouco mais sobre elas acessando sua [documentação](https://www.rdocumentation.org/packages/quanteda/versions/1.3.4/topics/textstat_lexdiv).

```{r}
head(textstat_lexdiv(dfm_noticias, measure = "all" ,drop=T))
```

### Conexões entre os termos (redes de palavras)

Para criar nossa rede de palavras, o primeiro passo é identificar quais são os recursos que queremos, vamos obter 40 e extrair seus nomes:

```{r}
top_feat <- names(topfeatures(dfm_noticias, n = 40))
head(top_feat)
```

Em um segundo passo, criaremos uma matriz de coocorrência de recursos com a função [*fcm*](https://www.rdocumentation.org/packages/quanteda/versions/2.1.2/topics/fcm) selecionando somente os 40 termos mais comuns que selecionamos no passo anterior.

```{r}
topfeatfcm <- fcm(dfm_noticias) %>% 
  fcm_select(., top_feat)
```

Por fim, criamos nosso gráfico de redes utilizando a função [*textplot_network*](https://www.rdocumentation.org/packages/quanteda/versions/2.1.2/topics/textplot_network) direcionada a redes de recursos coocorrentes. Lembre que o *set.seed* é para tornar replicável, enquanto as opções do gráfico dizem respeito à proporção de coocorrências (*min_freq*), a opacidade das ligações (*edge_alpha*) e o tamanho dos principais recursos coocorrentes (*edge_size*).

```{r}
set.seed(123)
textplot_network(topfeatfcm, min_freq = 0.1, edge_alpha = 0.2, edge_size = 2)
```
## Análise dos textos: estatísticas textuais: modelagem de texto

### Posições de Documentos

Aqui temos um exemplo de dimensionamento da posição de documentos não supervisionada utilizando nosso corpus completo com o Modelo "Wordfish". Para exemplos de modelagem de textos observem a [Documentação do Pacote *quanteda.textmodels*](https://www.rdocumentation.org/packages/quanteda.textmodels/versions/0.9.1). 

```{r}
wfm <- textmodel_wordfish(dfm_noticia_politicos)
textplot_scale1d(wfm, margin = "documents")
```

Também é possível fazer o gráfico a partir dos recursos, sendo possível destacar alguns que nos interessem. 

```{r}
textplot_scale1d(wfm, margin = "features",
                 highlighted = c("lula", "bolsonaro", "vacina", "cloroquina", "ivermectina"))
```

### Topic models

O *quanteda* oferece boas ferrametnas para *topic models*. Aqui retomamos os dados da Folha de São Paulo. Assim como fizemos anteriormente, aplicaremos a função `dfm_trim()` para realizar "cortes" nos nossos recursos. 

```{r}
quant_dfm <- dfm_trim(dfm_noticias, min_termfreq = 3, min_docfreq = 5)

quant_dfm
```

Agora podemos estimar um [*Structural Topic Model*](https://www.rdocumentation.org/packages/stm/versions/1.3.6/topics/stm) com o pacote `stm` e a função `stm()`. Vamos utilizar um exemplo de 10 tópicos, representado na opção `K` da função. Utilizamos o valor `FALSE` na opção `verbose` para que não imprima na tela as iterações realizadas durante os cálculos para modelagem realizados pela a função - a opção padrão é `TRUE`, a utilize se quiser observar cada um dos passos no console do seu RStudio.

```{r fig.width = 7, fig.height = 5}
my_lda_fit10 <- stm(quant_dfm, K = 10, verbose = FALSE)
plot(my_lda_fit10, main = "Principais Tópicos", xlab = "Proporções esperadas dos tópicos")    
```

São dois pontos sobre *topic modeling* que gostaria de ressaltar. O primeiro é que o número de tópicos é ajustável, sendo que precisamos "ajustar"  quantos necessários. Por exemplo, se alguns tópicos possuem termos que em copnjunto não fazem sentido, talvez indique a necessidade de um número maior de tópicos. O segundo ponto, mais intuitivo, é que os rótulos que pode encontrar nesse tipo de análise são feitos pelo próprio pesquisador.

## Terminamos(?)

Bem, espero que durante o curso tenham aproveitado bastante o conteúdo. O que vimos aqui fornece um bom conjunto de ferramentas para começar a explorar o mundo da análise quantitativa de texto e agora espero que sigam nessa explorando esse universo cada vez mais. Aqui nós vimos muito mais o aspecto de execução dos métodos, mas mais importante agora é identificar quais são os métodos de interesse e partir para leituras que auxiliem a entender melhor como utilizá-los.