# [Cebrap.lab 2021] Captura de dados com R

## Informações básicas

### Instrutor: 
	
[Thiago Meireles](https://thiagomeireles.github.io/)

### Data, Hora e Local

De 7 a 11 de junho de 2021, das 18h30 às 21h30.

Os encontros serão nos dias 7, 9 e 11 de junho via Zoom.

## Apresentação

O laboratório apresenta as principais ferramentas de captura de dados na Internet e análise quantitativa de texto utilizando R. Além de ser um software livre voltado para estatística computacional e análise de dados, R é uma linguagem focada na aplicação de funções que, entre outras possibilidades, permite a captura de dados de forma automatizada na internet.  A partir de informações disponíveis em portais de notícias, apresentaremos esse processo de raspagem de dados de páginas web (especialmente de tabelas e de páginas construídas em html) e construção de bases de dados com textos de Internet tratados como informações quantitativas, o que permitirá introduzir algumas das práticas de mineração de texto. Faremos um exercício empírico partindo de uma questão de pesquisa que conduzirá a experimentação, de forma a capacitar os participantes com ferramentas e procedimentos que depois poderão ser usadas para a construção de suas próprias bases de dados. Para participação no curso, espera-se conhecimento prévio da linguagem R ou uma preparação de nivelamento por meio de tutoriais indicados antes do início das aulas.

Esse repositório será alimentado ao longo do curso com roteiros de aula e tutoriais atualizados tentado atender as particularidades da turma.

### Dinâmica das aulas

As aulas terão conteúdo expositivo sobre conceitos e ferramentas básicas utilizados durante o curso, mas a maior parte do tempo será dedicada à realização de tutoriais assistidos. Trabalharemos em dupla, cada um em seu computador. O professor acompanhará o andamento de cada dupla, tirando as dúvidas (sim, elas surgirão).

### Presença e avaliação

Em todos os roteiros teremos links para a sala do Zoom e para a lista virtual.

O requisito para a emissão de certificado é a presença em dois dos três encontros virtuais.

No entanto, ressalto a importância das atividades de terça e quinta-feira. O primeiro por ser um desafio de colocar em prática com seu material o que veremos na segunda-feira. O segundo por dar uma visão mais ampla sobre *text mining* com os quais trabalharemos no último encontro.

## Requisitos

### Preparação

A participação no curso requer uma exposição prévia à linguagem R e ao ambiente de tabalho do RStudio.

Caso não tenha nenhum contato com a linguagem, é mandatória a realização de um [roteiro de tutoriais de preparação](https://github.com/thiagomeireles/cebraplab_captura_2021/blob/main/roteiros/pre_curso/01_basico.md) antes do início das aulas. 

Ainda que tenha conhecimento básico das estruturas da linguagem, é fortemente recomendado que tambem o façam.

O tempo estimado para o tutorial é de *aproximadamente 4 horas*.

Além disso, um [Tutorial de manipulação de dados com dplyr](https://github.com/thiagomeireles/cebraplab_captura_2021/blob/main/tutoriais/pre_curso/Tutorial_05.md) é mandatório e tem tempo estimado para realização em *aproximadamente 2 horas*.

### Equipamento

Como a maior parte do curso é baseada em tutoriais em que vocês aprenderão "colocando a mão na massa", é mandatório que acompanhem as aulas no computador.

### Softwares

Foi preparado um [Roteiro de instalação](https://github.com/thiagomeireles/cebraplab_captura_2021/blob/main/roteiros/pre_curso/00_instalacao.md) onde estão as instruções para a instalação dos softwares necessários.

## Objetivos

Os participantes, ao fim do curso, serão capazes de:
- Coletar dados de sites de estrutura mais simples, como jornais e legislativos brasileiros;
- Realizar tarefas relacionadas a mineração de texto a partir de diferentes abordagens
- Produzir gráficos e grafos mais simples a partir dos dados coletados
- Entender e aplicar conceitos básicos de *text mining*

## Roteiros e tutoriais

### Roteiros

Todas os dias de curso terão roteiros a cumprir. Pouco antes de cada encontro, as linhas abaixo serão preenchidas com links com as descrições do que esperamos em cada dia de curso e como o faremos.

[Dia 1](https://github.com/thiagomeireles/cebraplab_captura_2021/blob/main/roteiros/dia_01.md) - O básico da raspagem de dados

[Dia 2](https://github.com/thiagomeireles/cebraplab_captura_2021/blob/main/roteiros/dia_02.md) - Desafios de raspagem de dados

[Dia 3](https://github.com/thiagomeireles/cebraplab_captura_2021/blob/main/roteiros/dia_03.md) - Introdução à manipulação de textos como dados

[Dia 4](https://github.com/thiagomeireles/cebraplab_captura_2021/blob/main/roteiros/dia_04.md) - A pesquisa quantitativa com texto

[Dia 5](https://github.com/thiagomeireles/cebraplab_captura_2021/blob/main/roteiros/dia_05.md) - *Text mining* em R


### Tutoriais

Os links para os tutoriais estarão abaixo antes de cada aula.

[Tutorial 1](https://github.com/thiagomeireles/cebraplab_captura_2021/blob/main/tutoriais/Tutorial_01.md): Páginas com tabelas. Versão [Rmd](https://github.com/thiagomeireles/cebraplab_captura_2021/blob/main/tutoriais/Tutorial_01.Rmd)

[Tutorial 2](https://github.com/thiagomeireles/cebraplab_captura_2021/blob/main/tutoriais/Tutorial_02.md): Realizar a extração de qualquer conteúdo de uma página utilizando os "caminhos" dos elementos da página no código html - Introdução ao XPath. Versão [Rmd](https://github.com/thiagomeireles/cebraplab_captura_2021/blob/main/tutoriais/Tutorial_02.Rmd)

[Tutorial 3](https://github.com/thiagomeireles/cebraplab_captura_2021/blob/main/tutoriais/Tutorial_03.md): Extrair informações de uma sequência páginas (ex. portal de notícias) - Captura de notícias da Folha. Versão [Rmd](https://github.com/thiagomeireles/cebraplab_captura_2021/blob/main/tutoriais/Tutorial_03.Rmd)

[Tutorial 4](https://github.com/thiagomeireles/cebraplab_captura_2021/blob/main/tutoriais/Tutorial_04.md): Captura de notícias do DataFolha\. Versão [Rmd](https://github.com/thiagomeireles/cebraplab_captura_2021/blob/main/tutoriais/Tutorial_04.Rmd) 

[Tutorial 5](https://github.com/thiagomeireles/cebraplab_captura_2021/blob/main/tutoriais/Tutorial_05.md): Mineração de Texto - pacote *stringr*. Versão [Rmd](https://github.com/thiagomeireles/cebraplab_captura_2021/blob/main/tutoriais/Tutorial_05.Rmd)

[Tutorial 6](https://github.com/thiagomeireles/cebraplab_captura_2021/blob/main/tutoriais/Tutorial_06.md): Mineração de Texto - pacote *tm*. Versão [Rmd](https://github.com/thiagomeireles/cebraplab_captura_2021/blob/main/tutoriais/Tutorial_06.Rmd)

[Tutorial 7](https://github.com/thiagomeireles/cebraplab_captura_2021/blob/main/tutoriais/Tutorial_07.md): Mineração de Texto - pacote *tidytext*. Versão [Rmd](https://github.com/thiagomeireles/cebraplab_captura_2021/blob/main/tutoriais/Tutorial_07.Rmd)

[Tutorial 8](https://github.com/thiagomeireles/cebraplab_captura_2021/blob/main/tutoriais/Tutorial_08.md) Texto como dados e o pacote *quanteda*. Versão [Rmd](https://github.com/thiagomeireles/cebraplab_captura_2021/blob/main/tutoriais/Tutorial_08.Rmd)

## Referências

- Grolemund, Garrett (2014). Hands-On Programming with R. Ed: O'Reilly Media. Disponível gratuitamente [aqui](https://rstudio-education.github.io/hopr/)
- Wichkam, Hadley e Grolemund, Garrett (2016). R for Data Science. Ed: O'Reilly Media. Disponível gratuitamente [aqui](http://r4ds.had.co.nz/data-visualisation.html)
- Wichkam, Hadley (2014). Advanced R. Ed: Chapman and Hall/CRC. Disponível gratuitamente Disponível gratuitamente [aqui](http://adv-r.had.co.nz/)
- Gillespie, Colin e Lovelace, Robin (2016). Efficient R programming. Ed: O'Reilly Media. Disponível gratuitamente [aqui](https://csgillespie.github.io/efficientR/)