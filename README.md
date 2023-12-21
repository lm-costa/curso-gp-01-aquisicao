
<!-- README.md is generated from README.Rmd. Please edit that file -->

# curso-gp-01-aquisicao

![](img/img-00.png) Arte conceitual do OCO2 Fonte:
<https://www.jpl.nasa.gov/images/pia18374-nasas-orbiting-carbon-observatory-oco-2-artist-concept>

## Motivação

O GES DISC (Global Earth Science Data Information System) é um
repositório de dados científicos de sensoriamento remoto da NASA. Ele
contém dados de uma ampla variedade de sensores, incluindo satélites,
aeronaves e estações terrestres. Os dados do GES DISC podem ser usados
para uma variedade de aplicações, incluindo pesquisa científica,
monitoramento ambiental e aplicações comerciais.

Este curso visa fornecer aos alunos do grupo de pesquisa em Ciência de
Dados do Departamento de Ciências Exatas da FCAV-UNESP as habilidades
necessárias para adquirir dados no GES DISC. O objetivo inicial é
ensinar ao aluno como pesquisar dados, como baixar dados e como preparar
os dados GES DISC para posterior uso em suas pesquisas.

Além, disso, devido ao fato desse curso ser introdutório, serão
abordados temas como criação de projetos no RStudio , reprodutibilidade
e endereçamento.

## Criação do Projeto

A criação de projetos no RStudio é uma prática fundamental para qualquer
pessoa que trabalhe com dados usando o R. Os projetos permitem que você
estruture seus dados, código e relatórios de forma organizada e
eficiente. Isso pode ajudar o aluno a economizar tempo e esforço, e
também pode tornar seu trabalho mais reprodutível.

### Reproducibilidade

A reprodutibilidade é a capacidade de repetir os resultados de um
experimento ou análise. É uma prática importante na ciência e em outras
áreas, pois garante que os resultados sejam confiáveis e sejam capazes
de ser verificados por outros pesquisadores.

A criação de projetos no RStudio pode ajudar a melhorar a
reprodutibilidade de seu trabalho. Isso ocorre porque os projetos
permitem que você armazene todos os dados e código necessários para
executar uma análise em um único local. Isso torna mais fácil
compartilhar seu trabalho com outros e garantir que eles possam
reproduzir seus resultados.

### Endereçamento relativo

O endereçamento relativo é uma técnica que permite que você refira a
arquivos e pastas em um projeto usando caminhos relativos. Isso pode
tornar seu código mais conciso e fácil de manter.

A criação de projetos no RStudio torna o endereçamento relativo mais
fácil. Isso ocorre porque os projetos fornecem um contexto para os
caminhos de arquivos e pastas. Por exemplo, oo seu projeto, você pode
criar uma pasta chamada data para armazenar seus dados. Você pode então
ler os dados da pasta data usando o endereçamento relativo:

``` r
# Criar um objeto para armazenar os dados
dados <- read.csv("data/dados.csv")

# Visualizar os dados
head(dados)
```

O código acima criará um objeto chamado `dados` que contém os dados da
pasta data. O caminho relativo `"data/dados.csv"` refere-se ao arquivo
`dados.csv` na pasta `data` dentro do projeto.

O caminho absoluto para o arquivo `dados.csv` seria diferente,
dependendo do local do projeto no seu computador. Por exemplo, se o
projeto estiver localizado na pasta `C:\Meus Documentos\MeuProjeto`, o
caminho absoluto para o arquivo dados.csv seria
`C:\Meus Documentos\MeuProjeto\data\dados.csv`.

Assim, o uso de endereçamento relativo tem várias vantagens, incluindo:

- Torna o código mais conciso e fácil de entender.
- Torna o código mais portátil, pois não depende do local do projeto no
  computador.
- Torna o código mais reprodutível, pois os caminhos relativos são
  consistentes entre diferentes computadores.

## Criação da conta

<https://disc.gsfc.nasa.gov/>

![](img/img-01.png)

## configura do Wget

<https://eternallybored.org/misc/wget/>

![](img/img-02.png)

## Escolha da Base (Nível-2)

## Download dos dados

### Coteúdo referente ao arquivo `script_download.R`.

``` r
url_filename <- dir("url/",pattern = ".txt") # nome do arquivo txt

urls <- read.table(paste0("url/",url_filename)) # leu as urls dentro do arquivo



n_urls <- nrow(urls) # numero de linhas no arquivo urls

n_split <- length(stringr::str_split(urls[1,1],"/",simplify=TRUE))

n_split

filenames_nc <- stringr::str_split_fixed(urls[,1],"/",n=Inf)[,n_split] # armazenando o nome dos arquivos
filenames_nc

#### download

for(i in 1:n_urls){
  repeat{
    dw <- try(download.file(urls[i,1],
                            paste0("data-raw/",filenames_nc[i]),
                            method="wget",
                            extra= c("--user=alan.panosso --password FMB675fmb675@")
    ))
    if(!(inherits(dw,"try-error")))
      break
  }
}
```

## Extração dos dados

### Conteúdo referente ao script `script_extracao.R`

``` r
files_names <- list.files("data-raw/",
                          pattern = "nc")


exm <- ncdf4::nc_open(
  paste0("data-raw/",files_names[1])
)
typeof(exm)

for(i in 1:length(files_names)){
  if(i==1){
    df <- ncdf4::nc_open(paste0("data-raw/",files_names[i]))
    if(df$ndims==0){

    }else{
      xco2 <- data.frame(
        "longitude"=ncdf4::ncvar_get(df,varid="longitude"),
        "latitude"=ncdf4::ncvar_get(df,varid="latitude"),
        "time"=ncdf4::ncvar_get(df,varid="time"),
        "xco2"=ncdf4::ncvar_get(df,varid="xco2"),
        "xco2_quality_flag"=ncdf4::ncvar_get(df,varid="xco2_quality_flag"),
        "xco2_incerteza"=ncdf4::ncvar_get(df,varid="xco2_uncertainty")
      ) |>
        dplyr::filter(xco2_quality_flag==0) # quality = 0 ==> obs sem nuvem
    }
    ncdf4::nc_close(df)
  }else{
    df_a <- ncdf4::nc_open(paste0("data-raw/",files_names[i]))
    if(df_a$ndims == 0){

    }else{
      xco2_a <- data.frame(
        "longitude"=ncdf4::ncvar_get(df_a,varid="longitude"),
        "latitude"=ncdf4::ncvar_get(df_a,varid="latitude"),
        "time" = ncdf4::ncvar_get(df_a,varid="time"),
        "xco2" = ncdf4::ncvar_get(df_a,varid="xco2"),
        "xco2_quality_flag"=ncdf4::ncvar_get(df_a,varid="xco2_quality_flag"),
        "xco2_incerteza"=ncdf4::ncvar_get(df_a,varid="xco2_uncertainty")
      ) |>
        dplyr::filter(xco2_quality_flag==0)
    }
    ncdf4::nc_close(df_a)
    xco2 <- rbind(xco2,xco2_a) # stack = empilhar as tabelas
  }
}

xco2 <- xco2 |>
  dplyr::mutate(
    date = as.Date.POSIXct(time)
  )
# install.packages("readr")
readr::write_rds(xco2, "data/arquivo_xco2.rds")
```

``` r
xco2 <- readr::read_rds("data/arquivo_xco2.rds")
xco2 |>
  dplyr::sample_n(1000) |>
  ggplot2::ggplot(ggplot2::aes(x=date,y=xco2)) +
  ggplot2::geom_point() +
  ggplot2::geom_line()
```

![](README_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

``` r

xco2 |>
  dplyr::sample_n(1000) |>
  ggplot2::ggplot(ggplot2::aes(x=xco2, y=..density..)) +
  ggplot2::geom_histogram(bins = 9,
                          color="black",
                          fill="gray") +
  ggplot2::geom_density(fill="red",alpha=0.05) +
  ggplot2::theme_bw()
```

![](README_files/figure-gfm/unnamed-chunk-5-2.png)<!-- -->

![](img/final-grupo.png)
