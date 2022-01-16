---
title: "<center><div class='mytitle'>Analiza tržišta aparata za kavu</div></center>"
author: "<center><div class='mysubtitle'>Konkurenti: De`Longhi, LatteGo, Krups, Nespresso</div></center>"
date: "<center><div class='mysubtitle'></div></center>"
output:
  html_document:
    theme: yeti
    highlight: espresso
    code_folding: hide
    toc: yes
    toc_depth: 4
    toc_float: yes
    keep_md: yes
    css: style.css
    includes:
      before_body: header.html
      after_body: footer.html
  
---




## Analiza teksta


Analiza teksta dobiva na popularnosti zbog sve veće dostupnosti podataka i razvoja **user friendly** podrške za provedbu takve analize. Konceptualni pregled analize teksta ya sociologe je dostupan i u nedavno objavljenoj [knjizi](https://isi.f.bg.ac.rs/wp-content/uploads/2019/03/Zeljka_Manic_Analiza-sadr%C5%BEaja.pdf), koja se preporuča tek nakon savladavanja osnovnih tehničkih vještina i alata za obradu teksta. Provedba analize tekstualnih podataka je moguća na mnogo načina, a najšire korišten pristup je **bag-of-words** u kojem je frekvencija riječi polazište za analizu dok se (npr.) pozicija riječi u rečenici ili paragrafu zanemaruje. Bag of words pristup je ujedno i najjednostavniji (konceptualno i računarski) pa će biti korišten u ovom predavanju. 

Postupak analize teksta započinje *pripremom teksta (podataka)*, koja je često dosta zahtjevna i uključuje: **uvoz teksta**, **operacije sa riječima**, **uređivanje i tokenizaciju**, **izradu matrice pojmova**, **filtiranje i ponderiranje podataka**. Pri tome valja imati na umu da vrsta analize i korištena metoda određuju način na koji je potrebno pripremiti podatke za daljnu analizu te da svaka metoda ima svoje specifičnosti. Nakon pripreme podataka se vrši *analiza teksta (podataka)* metodama **nadziranog strojnog učenja**, **ne-nadziranog strojnog učenja**, **statistike na tekstualnim podatcima**, **analize riječnika**, **analize sentimenta**. *Napredne metode analize podataka* uključuju **NLP**, **analizu pozicije riječi i sintakse**...Sažeti prikaz *workflow-a* za analizu teksta izgleda ovako:


<div class="figure" style="text-align: center">
<img src="D:/LUKA/Academic/HS/NASTAVA/20-21/Obrada podataka/Foto/potupakAnalize.png" alt="Procedura za analizu teksta." width="700px" />
<p class="caption">Procedura za analizu teksta.</p>
</div>

### Software i korisni resursi

U ovom predavanju ćemo koristiti `tidytext` pristup (i istoimeni paket) za analizu tekstualnih podatka, detaljno opisan u knjizi [Text Mining with R](https://www.tidytextmining.com/). Ovaj paket služi kako bismo tekstualne podatke "uveli" u tidyverse ovir pomoću kojeg je moguće nestrukturirani tekst analizirati sa otprije poznatim alatima iz `dplyr` i `ggplot` paketa. Učitajmo potrebne pakete: 


```r
library(tidyverse)
library(tidytext)
library(data.table)
library(lubridate)
library(grid)
library(wordcloud)
library(reshape2)
library(igraph)
library(ggraph)
library(widyr)
library(topicmodels)
library(ggthemes)
library(DT)
library(kableExtra)
library(ggplot2)
library(ggthemes)
library(scales)
library(tidyverse)
library(httr)
library(lubridate)
library(dplyr)
library(data.table)
library(tidytext)
library(plotly)
library(readxl)
```


Prije opisa podataka koje ćemo koristiti valja naglasiti da `tidytext` pristup nije jedini način za rad s podatcima u R. Ovdje ga koristimo jer je kompatibilan sa pristupima koje smo do sada koristili u okviru ovog kolegija. Drugi paketi (pristupi) za rad sa tekstom u R su:

- `quanteda` je sveobuhvatan i funkcijama bogat paket, neophodan u za složeniju analizu teksta. Izvrstan tutorial je dostupan na [linku](https://tutorials.quanteda.io/).

- `text2vec` je izrazito koristan paket za ML algoritme sa tekstualnim podatcima. Posebno je pogodan za izradu *dtm* i *tcm* matrica. Paket je motiviran python-ovom Gensim knjižnicom, a tutorial je dostupan na [linku](http://text2vec.org/index.html).

- `stringr` paket je neophodan za manipulaciju *string* podataka u R i kao dio `tidyverse` svijeta će biti izrazito koristan u čišćenju i pripremi podataka. Vrlo je praktičan za rad sa [*regex-om*](https://en.wikipedia.org/wiki/Regular_expression)  i ima nekoliko izvrsnih funkcija za *pattern matching*. Službeni R Tutorial je dostupan na [linku](https://cran.r-project.org/web/packages/stringr/vignettes/stringr.html).

- `spacyr` je *wrapper* paket za spaCy knjižnicu iz python-a i omogućava provedbu naprednijih NLP modela (deep learning, speech tagging, tkoenization, parsing) u R. Također je kompatibilan sa quanteda i tidytext paketima. Tutorial je dostupan na [linku](https://spacyr.quanteda.io/articles/using_spacyr.html).  

- za one koji žele znati više mogu biti korisni i sljedeći resursi: [vodič za tekstualnu analizu u R](http://eprints.lse.ac.uk/86659/1/Benoit_Text%20analysis%20in%20R_2018.pdf) i [kolegij za obradu prirodnog teksta](https://github.com/BrbanMiro/Analiza-teksta) u najstajnju koji sadrži i mnoštvo referenci.


### Podatci

Svaka analiza (teksta)  počinje od podataka. Pribava tekstualnih podataka o specifičnim temama najčešće nije jednostavna. Najčešći je način preuzimanja podataka neki od dostupnih API servisa za novinske članke ili tekstualnih repozitorija ili servisi poput Twitter-a. No to često nije dovoljno ukolilko želimo analizirati specifičnu temu ili temu na specifičnom jeziku (npr. hrvatskom). Ovdje još valja napomeniti da je preuzimanje kvalitetnih tekstualnih podataka često moguće isključivo uz nadoplatu kao što je to slučaj člancima na hrvatskom jeziku kroz [webhose.io](https://webhose.io/) servis, [presscliping](https://www.pressclipping.hr/), [presscut](https://www1.presscut.hr/en/) i [mediatoolkit](https://www.mediatoolkit.com/)

U ovom ćemo predavanju analizirati tržište aparata za kavu u Hrvatskoj na osnovi osnovi svih tekstova objavljenih u svim domaćim medijima u perodu od  2021-01-09 do 2022-11-01. Članci su preuzeti strojno sa [mediatoolkit servisa](https://www.mediatoolkit.com/) i identificirani na način da sadrže riječ: *LatteGo*, *De`Longhi*, *Krups* i *Nesspreso*. Na taj je način prikupljeno 290 objava koje sadrže ukupno 8.980 riječi. Analiza teksta koju ćemo provesti uključuje nekoliko etapa: **čišćenje, uređivanje i prilagodbu podataka**, **dekriptivnu statistiku na tekstualnim podatcima**, **analizu sentimenta**, **analizu frekvencija** i **tematsku analizu**.


## Uvoz podataka

Podatci za analizu su prikupljeni na prethodno opisan način i dostupni u GitHub repozitoriju kolegija. Učitajmo i pregledajmo cjelopkupni podatkovni skup:

<br>
<button class="btn btn-primary" data-toggle="collapse" data-target="#Block1"> Pregled podataka </button>  
<div id="Block1" class="collapse">


```r
kava <- read_excel("D:/LUKA/Academic/HS/NASTAVA/20-21/WebObradaPodataka/Dta/kava.xlsx") #, encoding="UTF-8"
glimpse(kava)
```

```
## Rows: 289
## Columns: 45
## $ DATE                  <chr> "2022-01-10", "2022-01-08", "2022-01-07", "2022-~
## $ TIME                  <chr> "09:18:45", "18:32:43", "08:00:26", "18:00:00", ~
## $ TITLE                 <chr> "Hello.. #hello #monday #january #winter #day #w~
## $ FROM                  <chr> "anonymous_user", "Sarlo", "anonymous_user", "<U+0001D4EB><U+0001D4F8>~
## $ AUTHOR                <chr> "anonymous_user", "Sarlo", "anonymous_user", "<U+0001D4EB><U+0001D4F8>~
## $ URL                   <chr> "https://www.instagram.com/p/CYiubD4owlX/", "htt~
## $ URL_PHOTO             <chr> "https://mediatoolkit.com/img/50x50,sc,s-3IcNbqA~
## $ SOURCE_TYPE           <chr> "instagram", "twitter", "instagram", "twitter", ~
## $ GROUP_NAME            <chr> "Philips", "Philips", "Philips", "Philips", "Phi~
## $ KEYWORD_NAME          <chr> "Nespresso", "Nespresso", "LatteGo", "Nespresso"~
## $ FOUND_KEYWORDS        <chr> "nespresso", "Nespresso", "LatteGo, lattego", "n~
## $ LANGUAGES             <chr> "hr, et, no", "hr", "hr, bs", "hr, sk", "hr", "h~
## $ LOCATIONS             <chr> "EE, NO, HR", "HR", "HR, BA", "SK, HR", "HR", "H~
## $ TAGS                  <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, ~
## $ MANUAL_SENTIMENT      <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, ~
## $ AUTO_SENTIMENT        <chr> "neutral", "neutral", "positive", "neutral", "ne~
## $ MENTION_SNIPPET       <chr> "Hello.. #hello #monday #january #winter #day #w~
## $ REACH                 <dbl> 50, 0, 30, 22, NA, NA, NA, NA, 10, 77, 0, 50, 46~
## $ VIRALITY              <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, 0.0000000, 0~
## $ FOLLOWERS_COUNT       <dbl> 0, 9, 0, 449, NA, NA, NA, NA, 0, NA, NA, 259, NA~
## $ LIKE_COUNT            <dbl> 5, NA, 3, NA, NA, NA, NA, NA, 1, 0, 0, 3, 91, 0,~
## $ COMMENT_COUNT         <dbl> 0, NA, 0, NA, NA, NA, NA, NA, 0, 0, 0, 2, 16, 0,~
## $ SHARE_COUNT           <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, 0, 0, NA, 6,~
## $ TWEET_COUNT           <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, ~
## $ LOVE_COUNT            <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, ~
## $ WOW_COUNT             <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, ~
## $ HAHA_COUNT            <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, ~
## $ SAD_COUNT             <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, ~
## $ ANGRY_COUNT           <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, ~
## $ TOTAL_REACTIONS_COUNT <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, ~
## $ FAVORITE_COUNT        <dbl> NA, 0, NA, 0, NA, NA, NA, NA, NA, NA, NA, NA, NA~
## $ RETWEET_COUNT         <dbl> NA, 0, NA, 0, NA, NA, NA, NA, NA, NA, NA, NA, NA~
## $ VIEW_COUNT            <dbl> 0, NA, 0, NA, NA, NA, NA, NA, 0, NA, NA, 0, NA, ~
## $ DISLIKE_COUNT         <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, ~
## $ COMMENTS_COUNT        <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, ~
## $ LIKES                 <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, ~
## $ DISLIKES              <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, ~
## $ COUNT                 <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, ~
## $ REPOST_COUNT          <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, ~
## $ REDDIT_TYPE           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, ~
## $ REDDIT_SCORE          <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, ~
## $ INFLUENCE_SCORE       <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 3, 1, 1, 3, 1, 1, 3, ~
## $ TWEET_TYPE            <chr> NA, "ORIGINAL", NA, "ORIGINAL", NA, NA, NA, NA, ~
## $ TWEET_SOURCE_NAME     <chr> NA, "Twitter Web App", NA, "Twitter for Android"~
## $ TWEET_SOURCE_URL      <chr> NA, "https://mobile.twitter.com", NA, "http://tw~
```

</div>
<br>


Nakon što smo učitali podatke u radni prostor R, potrebno je učitati i druge podatke koji su nam potrebni za ovu  analizu. Osim **članaka**, potrebni su nam **leksikoni** i **stop riječi**. Leksikone ćemo preuzeti iz FER-ovog [repozitorija](http://meta-share.ffzg.hr/repository/browse/croatian-sentiment-lexicon/940fe19e6c6d11e28a985ef2e4e6c59eff8b12d75f284d58aacfa8d732467509/), a "stop riječi" ćemo napraviti sami. Ti su podatci trenutno pohranjeni na privatnoj MFiles bazi pa ćemo ih od tamo preuzeti na lokalno računalno:

<br>
<button class="btn btn-primary" data-toggle="collapse" data-target="#Block2"> Preuzimanje leksikona i stop riječi </button>  
<div id="Block2" class="collapse">



```r
## M-Files ----
# function to parse JSON from http conenctiion
parseJSON <- function(x) {
  xCon <- content(x, as = "text", type = "aplication/json", encoding = "UTF-8")
  xCon <- jsonlite::fromJSON(xCon, flatten = TRUE)
  xCon
}
# GET REST API function M-Files
mfiles_get <- function(token, resource){
  req <- GET(url = paste0('http://server.contentio.biz/REST', resource),
             add_headers('X-Authentication' = token, 'content-type' = "application/json"))
  result <- parseJSON(req)
  return(result)
}
# GET token M-Files
req <- POST(url = 'http://server.contentio.biz/REST/server/authenticationtokens.aspx', 
            config = add_headers('content-type' = "application/json"),
            body = list(Username = "msagovac", Password = "Wc8O10TaHz40",
                        VaultGuid = "{7145BCEB-8FE2-4278-AD3B-7AE70374FF8A}",
                        ComputerName  = "CT-VM-01"),
            encode = "json", verbose())
```

```r
token <- parseJSON(req)[[1]]
# M-FILES DOWNLOAD FILES
mfiles_downlaod <- function(objType, objId, fileId) {
  req <- GET(url = paste0('http://server.contentio.biz/REST/objects/', objType, '/', 
                          objId, '/latest/files/',fileId , '/content'),
             add_headers('X-Authentication' = token))
  reqCon <- content(req, as = "text", encoding = "UTF-8")
  if (is.na(reqCon)) {
    reqCon <- content(req, as = "raw", encoding = "UTF-8")
    reqCon <- rawToChar(reqCon, multiple = FALSE)
    reqCon <- iconv(reqCon, "", "UTF-8")
  }
  reqCon
}
mfiles_downlaod_txt <- function(objType, objId, fileId, ext = ".csv") {
  req <- GET(url = paste0('http://server.contentio.biz/REST/objects/', objType, '/', 
                          objId, '/latest/files/',fileId , '/content'),
             add_headers('X-Authentication' = token))
  reqCon <- httr::content(req)
  tempFileSave <- paste0(tempfile(), ext)
  writeBin(reqCon, tempFileSave)
  return(tempFileSave)
}
# GET classess, props and others
prop <- mfiles_get(token, "/structure/properties")
prop <- prop %>% 
  select(DataType, ID, Name, ObjectType) %>% 
  dplyr::arrange(Name)
objs <- mfiles_get(token, "/structure/objecttypes")
mfilesClass <- mfiles_get(token, "/structure/classes")
CroSentilex_n <- read.delim(mfiles_downlaod_txt("0", 136679, 136711, ext = ".txt"),
                            header = FALSE,
                            sep = " ",
                            stringsAsFactors = FALSE) %>% 
  rename(word = "V1", sentiment = "V2" ) %>%
  mutate(brija = "NEG")
CroSentilex_p <- read.delim(mfiles_downlaod_txt("0", 136681, 136713, ext = ".txt"),
                            header = FALSE,
                            sep = " ",
                            stringsAsFactors = FALSE) %>% 
  rename(word = "V1", sentiment = "V2" ) %>%
  mutate(brija = "POZ")
Crosentilex_sve <- rbind(setDT(CroSentilex_n), setDT(CroSentilex_p))
#head(Crosentilex_sve)
CroSentilex_Gold  <- read.delim2(mfiles_downlaod_txt("0", 136680, 136712, ext = ".txt"),
                                 header = FALSE,
                                 sep = " ",
                                 stringsAsFactors = FALSE) %>%
  rename(word = "V1", sentiment = "V2" ) 
CroSentilex_Gold[1,1] <- "dati"
CroSentilex_Gold$sentiment <- str_replace(CroSentilex_Gold$sentiment , "-", "1")
CroSentilex_Gold$sentiment <- str_replace(CroSentilex_Gold$sentiment , "\\+", "2")
CroSentilex_Gold$sentiment <- as.numeric(unlist(CroSentilex_Gold$sentiment))
#head(CroSentilex_Gold)
# leksikoni
stopwords_cro <- get_stopwords(language = "hr", source = "stopwords-iso")
my_stop_words <- tibble(
  word = c(
    "jedan","i", "za", "je", "ti","mp","50","4300","5400",
    "e","prvi", "dva","dvije","drugi","u","na","my",
    "tri","tre?i","pet","kod", "bit.ly", "pixie", "https","family.hr",
    "ove","ova",  "ovo","bez",
    "evo","oko",  "om", "ek",
    "mil","tko","?est", "sedam",
    "osam",   "?im", "zbog",
    "prema", "dok","zato", "koji", 
    "im", "?ak","me?u", "tek",
    "koliko", "tko","kod","poput", 
    "ba?", "dakle", "osim", "svih", 
    "svoju", "odnosno", "gdje",
    "kojoj", "ovi", "toga","ima","treba","sad","to","kad", "?e","ovaj","?ta","onda","ce","ko"
  ),
  lexicon = "lux"
)
stop_corpus <- my_stop_words %>%
  bind_rows(stopwords_cro)
```


</div>
<br>





## Prilagodba podataka

U sljedećem koraku ćemo stvoriti neke dodatne varijable korisne za analizu:


```r
kava %>%
   mutate(kword = case_when(grepl("latteg", MENTION_SNIPPET, ignore.case = TRUE) ~ "LatteGo",
                            grepl("longhi", MENTION_SNIPPET, ignore.case = TRUE) ~ "DeLonghi",
                            grepl("krups", MENTION_SNIPPET, ignore.case = TRUE) ~ "Krups",
                            grepl("Nespresso", MENTION_SNIPPET, ignore.case = TRUE) ~ "Nespresso")) -> kava
```


Potom pretvaramo podatke u `dataframe`, izabiremo varijable za analizu, specificiramo vremenski pečat članka kao datumsku varijablu, pripisujemo id svakom članku, izabiremo vremenski raspon analize i dodajemo numerički označitelj svakom članku: 


```r
# prilagodi podatke
newskava <- kava %>% 
  as.data.frame() %>%
  select(TITLE,MENTION_SNIPPET, DATE, SOURCE_TYPE, AUTHOR, FROM, kword) %>%  
  mutate(datum = as.Date(DATE,"%Y-%m-%d")) %>%
  mutate(clanak = 1:n()) 
```


<br>
<button class="btn btn-primary" data-toggle="collapse" data-target="#Block3"> Pregled uređenih podatka </button>  
<div id="Block3" class="collapse">


```r
# brzi pregled strukture podataka
glimpse(newskava)
```

```
## Rows: 289
## Columns: 9
## $ TITLE           <chr> "Hello.. #hello #monday #january #winter #day #work #l~
## $ MENTION_SNIPPET <chr> "Hello.. #hello #monday #january #winter #day #work #l~
## $ DATE            <chr> "2022-01-10", "2022-01-08", "2022-01-07", "2022-01-06"~
## $ SOURCE_TYPE     <chr> "instagram", "twitter", "instagram", "twitter", "forum~
## $ AUTHOR          <chr> "anonymous_user", "Sarlo", "anonymous_user", "<U+0001D4EB><U+0001D4F8><U+0001D4F8><U+0001D4F4><U+0001D4EA> <U+0001D4EB><U+0001D4F8>~
## $ FROM            <chr> "anonymous_user", "Sarlo", "anonymous_user", "<U+0001D4EB><U+0001D4F8><U+0001D4F8><U+0001D4F4><U+0001D4EA> <U+0001D4EB><U+0001D4F8>~
## $ kword           <chr> "Nespresso", "Nespresso", "LatteGo", "Nespresso", "Nes~
## $ datum           <date> 2022-01-10, 2022-01-08, 2022-01-07, 2022-01-06, 2022-~
## $ clanak          <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16,~
```

```r
# izgled podataka
# newskava %>%
#   sample_n(.,10)

datatable(newskava, rownames = FALSE, filter="top", options = list(pageLength = 5, scrollX=T) )
```

```{=html}
<div id="htmlwidget-328fc5967fcb62dc8793" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-328fc5967fcb62dc8793">{"x":{"filter":"top","filterHTML":"<tr>\n  <td data-type=\"character\" style=\"vertical-align: top;\">\n    <div class=\"form-group has-feedback\" style=\"margin-bottom: auto;\">\n      <input type=\"search\" placeholder=\"All\" class=\"form-control\" style=\"width: 100%;\"/>\n      <span class=\"glyphicon glyphicon-remove-circle form-control-feedback\"><\/span>\n    <\/div>\n  <\/td>\n  <td data-type=\"character\" style=\"vertical-align: top;\">\n    <div class=\"form-group has-feedback\" style=\"margin-bottom: auto;\">\n      <input type=\"search\" placeholder=\"All\" class=\"form-control\" style=\"width: 100%;\"/>\n      <span class=\"glyphicon glyphicon-remove-circle form-control-feedback\"><\/span>\n    <\/div>\n  <\/td>\n  <td data-type=\"character\" style=\"vertical-align: top;\">\n    <div class=\"form-group has-feedback\" style=\"margin-bottom: auto;\">\n      <input type=\"search\" placeholder=\"All\" class=\"form-control\" style=\"width: 100%;\"/>\n      <span class=\"glyphicon glyphicon-remove-circle form-control-feedback\"><\/span>\n    <\/div>\n  <\/td>\n  <td data-type=\"character\" style=\"vertical-align: top;\">\n    <div class=\"form-group has-feedback\" style=\"margin-bottom: auto;\">\n      <input type=\"search\" placeholder=\"All\" class=\"form-control\" style=\"width: 100%;\"/>\n      <span class=\"glyphicon glyphicon-remove-circle form-control-feedback\"><\/span>\n    <\/div>\n  <\/td>\n  <td data-type=\"character\" style=\"vertical-align: top;\">\n    <div class=\"form-group has-feedback\" style=\"margin-bottom: auto;\">\n      <input type=\"search\" placeholder=\"All\" class=\"form-control\" style=\"width: 100%;\"/>\n      <span class=\"glyphicon glyphicon-remove-circle form-control-feedback\"><\/span>\n    <\/div>\n  <\/td>\n  <td data-type=\"character\" style=\"vertical-align: top;\">\n    <div class=\"form-group has-feedback\" style=\"margin-bottom: auto;\">\n      <input type=\"search\" placeholder=\"All\" class=\"form-control\" style=\"width: 100%;\"/>\n      <span class=\"glyphicon glyphicon-remove-circle form-control-feedback\"><\/span>\n    <\/div>\n  <\/td>\n  <td data-type=\"character\" style=\"vertical-align: top;\">\n    <div class=\"form-group has-feedback\" style=\"margin-bottom: auto;\">\n      <input type=\"search\" placeholder=\"All\" class=\"form-control\" style=\"width: 100%;\"/>\n      <span class=\"glyphicon glyphicon-remove-circle form-control-feedback\"><\/span>\n    <\/div>\n  <\/td>\n  <td data-type=\"date\" style=\"vertical-align: top;\">\n    <div class=\"form-group has-feedback\" style=\"margin-bottom: auto;\">\n      <input type=\"search\" placeholder=\"All\" class=\"form-control\" style=\"width: 100%;\"/>\n      <span class=\"glyphicon glyphicon-remove-circle form-control-feedback\"><\/span>\n    <\/div>\n    <div style=\"display: none; position: absolute; width: 200px;\">\n      <div data-min=\"1630454400000\" data-max=\"1641772800000\"><\/div>\n      <span style=\"float: left;\"><\/span>\n      <span style=\"float: right;\"><\/span>\n    <\/div>\n  <\/td>\n  <td data-type=\"integer\" style=\"vertical-align: top;\">\n    <div class=\"form-group has-feedback\" style=\"margin-bottom: auto;\">\n      <input type=\"search\" placeholder=\"All\" class=\"form-control\" style=\"width: 100%;\"/>\n      <span class=\"glyphicon glyphicon-remove-circle form-control-feedback\"><\/span>\n    <\/div>\n    <div style=\"display: none; position: absolute; width: 200px;\">\n      <div data-min=\"1\" data-max=\"289\"><\/div>\n      <span style=\"float: left;\"><\/span>\n      <span style=\"float: right;\"><\/span>\n    <\/div>\n  <\/td>\n<\/tr>","data":[["Hello.. #hello #monday #january #winter #day #work #longday #nespresso #cafe #coffee #like #likesforlike #love #lovely #happiness #happyme #haveaniceday #always #week #induljonahét #imadom","@bdatdzutim43 Kupis Dolce Gusto i adapter za Nespresso na ebay-u (oko 10€) i imas oba.","Potrebna ti je prava gurmanska budilica koja će te dignuti iz kreveta? Izaberi espresso na svom LatteGo 5400 aparatu. Osim što će te razbuditi u trenu, espresso je niskokaloričan, a poboljšava i koncentraciju. Nije ni čudo što ga Talijani piju svaki dan! ☕ . . . #philips #philipshrvatska #philipshomeliving #coffee #philipscoffee #lattego","@dannyd1976 J/k I love my #nespresso machine 👍","Koji espresso aparat?","Nescafe Dolce Gusto","Koji espresso aparat?",null,"morning buzz #volluto #nespresso #caffeinefix #coffee #conradmanila","Philips aparati za kavu 2022","Apartman Neige d'or u Tignes FR7351.385.4","#autumn #season #Holiday #vacation #sunset #sunrise #sea #wave #happiness #halloween #party #life #Beach #yacht #sailing #coffee #nespresso #wine #ron #anesthesia #operation #Croatia #oman #zilina #slowmotion #hospital #timelapse #Christmasmarket","Nespresso Boutique u Zagrebu - idealna adresa za kupovinu poklona","3 recepta za zimsku kavu s kojima će blagdani biti slađi","3 recepta za zimsku kavu s kojima će blagdani biti slađi","Shopping vodič 2021: Last-minute božićni pokloni","Ovo su 3 recepta za pripremu jedinstvene zimsku kave! Kava odavno predstavlja puno više od napitka za razbuđivanje. Omiljeni napitci od kave uvijek su u...","True love 💓 #coffee #nespresso ☕☕☕","U Roses Designer's outlet centru u Svetom Križu Začretje otvorena je nova outlet trgovina Home&amp;cook tvrtke SEB. U njoj se po outlet cijenama prodaju...","3 recepta za zimsku kavu s kojima će blagdani biti slađi","3 recepta za zimsku kavu s kojima će blagdani biti slađi","Istražite PHILIPS blagdansku ponudu malih kućanskih aparata","Po jutarnjoj se kavi dan poznaje","Prvi jestivi poklon je ispečen, upakiran i sutra će na put do jedne drage mi osobe. Vjerujem da su sve vrste prazničnih keksića odlična ideja za poklon, a uz...","3 recepta za zimsku kavu s kojima će blagdani biti slađi - Magazin za turizam i gastronomiju","3 recepta za zimsku kavu s kojima će blagdani biti slađi","Kreativan blagdanski poklon koji će oduševiti sve ljubitelje dobre kave!","Omiljena kava uz samo jedan dodir.","3 recepta za zimsku kavu s kojima će blagdani biti slađi","#mycuppa #doitfluid #coffee #coffeeaddict #coffeegram #coffeelover #coffeetime #coffeeaddiction #expresso #nespresso #coffeeflavors morningjava #morningmust #morningmotivation #homecafe #homebarista #coffeetime #coffeeholic #coffeedaily #photooftheday #likes #follow","Otvorena je već druga Home&amp;Cook trgovina u Hrvatskoj","Nespresso limitirana blagdanska kolekcija","Johanna Ortiz unijela je ljepotu šume u Nespresso limitiranu blagdansku kolekciju","Johanna Ortiz unijela je ljepotu šume u Nespresso limitiranu blagdansku kolekciju","Tag #nespresso označava ova pitanja","Nespresso ima limitiranu blagdansku kolekciju i potpisuje je poznata modna dizajnerica","Johanna Ortiz unijela je ljepotu šume u Nespresso limitiranu blagdansku kolekciju","Dizajnerica Johanna Ortiz kreirala Nespresso limitiranu blagdansku kolekciju","Što je kolumbijska modna dizajnerica Johanna Ortiz dizajnirala za Nespresso? https://t.co/ymJ4FV6uGO #pitanje #odgovor","Dizajnerski komad koji ovog Božića želimo u svom domu","Johanna Ortiz unijela je ljepotu šume u Nespresso limitiranu blagdansku kolekciju - Magazin za turizam i gastronomiju","Arhiva turizam - Page 131 of 131 - Magazin za turizam i gastronomiju","RT @Fnac_Home ¡Concurso! Se acaba el año y seguro que aún tienes muchos cafés pendientes.\r\n\r\n✔️Síguenos\r\n✔️Haz RT + #DeLonghiFnacHome, dinos a qué amigo le debes un ☕️ y esta cafetera superautomática De'Longhi puede ser tuya 😉\r\n\r\nBBLL https://t.co/IhWzeceW0R\r\n\r\n¡El miércoles 15 anunciamos ganador! https://t.co/UTboAq7jUv https://terms.easypromosapp.com/t/36146","Ideje za božićne poklone koji će razveseliti sve foodieje","Ako vam je promaknula jučerašnja, posljednja epizoda u ovoj sezoni InDizajna, sada je možete pogledati :) My Istria De’Longhi Hrvatska Credo centar Meblo...","Philips aparati za kavu prosinac 2021","Ne propustite posljednju ovosezonsku epizodu vaše omiljene emisije 😍 Pogledajte u videu što vas očekuje u današnjem InDizajnu 👇 My Istria De’Longhi...","Koji espresso aparat?","Koji espresso aparat?","PAŽNJA! Povodom službenog otvorenja prodavaonice Home &amp; Cook imamo odlične popuste! 🍴 Dodatnih 10% popusta na blagajni za brendove: EMSA, Kaiser, Krups,...","i love u ! 😂#morningfix #nespresso","Ovih blagdana složite idealan Nespresso paket za najveće kavoljupce","Idejno rješenje za kupaonicu sa saunom ponuđeno je u tri varijante s različitim modelima keramike. Koju su vlasnici odabrali te kako izgleda kupaonica nakon...","Reposted from @paolotroilo54 WTF . . . . . . . . . . . . . . . #wtf #fingerpainting #contemporaryart #body #shoulders #art #privatecollection #museum #figurativeart #paintedwithfingers #nespresso","Shopping vodič: Pokloni koje naručiti iz udobnosti svog doma","Donosimo idealne poklone koje možete kupiti iz udobnosti vlastitog doma","Mali, lijep, funkcionalan i praktičan! Idealan poklon🎁 za blagdane🎄- Nespresso aparat za mirisnu ☕️kavu. &lt;3 &gt;&gt; https://bit.ly/3ECKzGf Izbor iz ponude,...","Mali, kompaktani, lagani i divnog dizajna - Nespresso aparati! Od sada uz 20% popusta na izdvojene modele i vrijedan poklon. 😀 Btw. posjeti nas u Arena...","SuperCard - 10 ideja za savršen poklon!","Akcijske cijene za Lavazza Nespresso kapsule na www.lovin.hr #Lavazza #Nespresso #Lovin #LovinWebshop #CoffeeMoment #CoffeeBreak #Akcija","Love my job 🎄 #nespresso #pinterest #sapin #sapindenoel #deco #decodeclasse #portre #peinture #peinture #dixdoigts #capsules #maitresse #maitressedecole #maitressedematernelle #imagination","Čini li nam se ili (shopping za) Božić dolazi sve ranije?","Subotnju epizodu emisije Tražimo dom s Mirjanom Mikulec sada možete pogledati i online :) Citroën Samsung BAUHAUS Hrvatska Momax Hrvatska OTP banka d.d....","Subotnju epizodu emisije Tražimo dom s Mirjanom Mikulec sada možete pogledati i online :) Citroën Samsung BAUHAUS Hrvatska Momax Hrvatska OTP banka d.d....","Započni tjedan sa stilom.. #CoffyLovers #thecoffyway #thecoffywaycroatia #coffeelover #coffybreak #coffetime #coffeeholic #coffeeshop #coffeeaddict #kapsule #kompatibilne #lavazza #lavazzapoint #lavazzaamodomio #lavazzafirma #juliusmeinl #nespresso #nescafedolcegusto #dolcegusto #kavauzrnu #mljevenakava #filterzakave #vrećice #dalmacija...","Pogodi što smo mi jutros pronašli u svojoj čizmici - Philips LatteGO 5400! Elegantno dizajnirani aparat koji spravlja čak 12 različitih vrsta kave prilagodljive arome i okusa. Disclaimer: Imamo jako velike čizmice. 😅 . . . #philips #philipshrvatska #philipshomeliving #coffee #philipscoffee #lattego","Essenza Plus Black - Nespresso Hrvatska","PRODAVAČ / TRGOVAC (m/ž) SEB mku &amp; p d.o.o. 📍Zagreb Groupe SEB posjeduje 30 brandova u sektoru malih kućanskih uređaja, posuđa i opreme. Svima dobro poznati...",null,"U današnjoj epizodi emisije Tražimo dom s Mirjanom Mikulec upoznat ćemo djevojku Martinu koja je sa zaručnikom u potrazi za stanom :) Njihov uvjet je budući...","Čini li nam se ili (shopping za) Božić dolazi sve ranije?","Konačno je došao trenutak da podijelimo s vama dugo željenu epizodu podcasta Iskreno o majčinstvu. Današnja tema su partnerski odnosi, stres i kako promijeniti svoj mindset. Ovu epizodu snimale smo uz savršenu šalicu kave iz Philips LatteGo 5400 aparata za kavu, te se ovim putem zahvaljujemo upravo Philipsu što nas podržava u glasnom progovoru...","Black Friday i dalje traje! #anangroup #blackfriday #capsule #nespresso #espresso #dolcegusto","Čini li nam se ili (shopping za) Božić dolazi sve ranije?","KUPILI SMO KAFE APARAT I SUPER JE","Martina i njezin zaručnik traže novi dom ❤🏠 Koje stanove su im pronašli agenti, saznajte u subotu prijepodne na RTL-u, u novoj epizodi Tražimo dom s...","Čini li nam se ili (shopping za) Božić dolazi sve ranije?","Čini li nam se ili (shopping za) Božić dolazi sve ranije?","Čini li nam se ili (shopping za) Božić dolazi sve ranije?","Evo gdje se kriju najbolje ideje za blagdanski shopping!","Čini li nam se ili (shopping za) Božić dolazi sve ranije?","Čini li nam se ili (shopping za) Božić dolazi sve ranije?","Čini li nam se ili (shopping za) Božić dolazi sve ranije?","Čini li nam se ili shopping za Božić dolazi sve ranije?","Zeleni dizajn budućnosti: Uređenjem interijera možete utjecati i na vaše zdravlje","Provjerite što je savršen božićni poklon za baš svakoga!","Sublime. #nespresso #vertuo #todoestabien","Ako ste u subotu propustili Kristininu potragu za novim domom, sada možete pogledati cijelu epizodu :) BAUHAUS Hrvatska Citroën Samsung De’Longhi Hrvatska...","Ako ste u subotu propustili Kristininu potragu za novim domom, sada možete pogledati cijelu epizodu :) BAUHAUS Hrvatska Citroën Samsung De’Longhi Hrvatska...","Kada temperature padaju, osjećamo potrebu što više ostati kod kuće. Za savršeno zimsko jutro preporučamo vam da isprobate Pumpkin Spice Latte. Sve što će vam trebati je - vaša najdraža kava iz našeg LatteGo aparata, pumpkin spice začin, pire od bundeve i malo cimeta. Kako bi upotpunili svoj užitak, pustite novi album od Adele i pokrijte se...","Kada temperature padaju, osećamo potrebu da što više ostanemo kod kuće. Za savršeno zimsko jutro preporučujemo vam da isprobate Pumpkin Spice Latte. Sve što vam je potrebno su - vaša najdraža kafa iz našeg LatteGo aparata, pumpkin spice začin, pire od bundeve i malo cimeta. Kako bi upotpunili svoj užitak, pustite novi album od Adele i pokrijte...","Čini li nam se ili (shopping za) Božić dolazi sve ranije?","Jučerašnja epizoda InDizajna već danas je online 👇💕 My Istria De’Longhi Hrvatska Perfecta Dreams Hespo Samsung BAUHAUS Hrvatska Lesnina XXXL TEXO Hrvatska...","Aparat za espresso kavu Philips EP4341/50 / Svijet-medija.hr: Aparati za espresso kavu","Čarolija adventa u Emmezeti - svaki dan novi popusti i pokloni!","Koji espresso aparat?","🖤Nespresso je napravio revoluciju u kreiranju aparata za kavu za \"po kući\" Ovog #CrnogPetka dodatno uštedite u Emmezeta centrima ili web shopu 🖤 👉👉...","Provedite prijepodne na RTL-u u društvu vaše omiljene emisije 📺🔝 U današnoj epizodi pogledajte kako smo poznatoj voditeljici Ani Radišić uredili spavaću...","Junakinja današnje epizode je Kristina koja želi novi dom u istočnom dijelu Zagreba 🏠 Koje nekretnine su joj pronašli agenti emisije Tražimo dom s Mirjanom...","Junakinja današnje epizode je Kristina koja želi novi dom u istočnom dijelu Zagreba 🏠 Koje nekretnine su joj pronašli agenti emisije Tražimo dom s Mirjanom...","Mali sneak peak u jednu sasvim posebnu kuću u Hrvatskoj :) 🔝 Kako izgleda ostatak interijera, kao i impresivan eskterijer, pogledajte u novoj epizodi...","Black Friday akcija do -37% Nespresso, izbor iz ponude - Aktualno","Postavili smo muškarcima nekoliko škakljivih pitanja. Njihovi odgovori pokazali su zašto je Movember važan","U novoj epizodi Tražimo dom s Mirjanom Mikulec upoznat ćemo Kristinu koja je u potrazi na novim domom :) Budite s nama u subotu prijepodne na RTL-u 📺⏰...","Krups Arabica EA811810 aparat za kavu","I love my NESPRESSO!","Apartman La Cormorane u Carnac FR2618.270.6","Crazy weekend 2021","Koji espresso aparat?","Kava – tamna, magična tekućina koja pretvara „pusti me na miru“ u „dobro jutro, ljubavi . Kava je puno više od napitka. To je moj trenutak kada sam sama sa sobom, kada plovim svojim mislima i nastojim ih uobličiti u riječi. To je taj osjećaj topline na usnicama i baršunastog osjeta na nepcu. To je taj trenutak kojeg nekada sebično čuvam za sebe....","Nespresso akcija prosinac 2021","Aparat za espresso kavu Philips EP3241/50 / Svijet-medija.hr: Aparati za espresso kavu","Krups","Gdje u Zagrebu popiti (i kupiti) specialty kavu?","Ako ste propustili pogledati subotnje izdanje Tražimo dom s Mirjanom Mikulec i Tinovu potragu za idealnim domom, sada to možete nadoknaditi 🏡⤵ De’Longhi...","Ako ste propustili pogledati subotnje izdanje Tražimo dom s Mirjanom Mikulec i Tinovu potragu za idealnim domom, sada to možete nadoknaditi 🏡⤵ De’Longhi...","☕ Sve ljubitelje dobre kave će obradovati vijest da smo imali dopunu Dolce Gusto Infinissima kafe aparata. 🎁 Uz svaku online narudžbu šaljemo vam na poklon Nescafé Cappuccino kapsule. ℹ Ponuda vrijedi do 29.11.2021. #ZADstoreKiseljak #Krups #Nescafe #BiH","Krema goodness 😋 #coffeecheers #nespresso","Sve nam je top, ali ove otvorene grede naročito 😍😍😍 My Istria De’Longhi Hrvatska","Cijeli mjesec je u znaku crne kave ☕️! Najbolje Nespresso aparate potražite u Emmezeti. &gt;&gt; https://bit.ly/3qZm2XZ","Ako niste stigli jučer, sada možete pogledati novu epizodu InDizajna :) My Istria De’Longhi Hrvatska Moj dom po mjeri BAUHAUS Hrvatska Lesnina XXXL Samsung...","Koji espresso aparat?","Alebrije loco 🤪 . . . . #art #arte #nespresso #capsulas #alebrijes #alebrijeloco #onlyart #happylife","Moze anon Zene znate li gdje mogu kupit kapsule za dolce gusto, ali da nisu nes ili starbucks nego malo jeftinije, ali da odgovaraju za picollo krups aparat....","Danas je s vama nova epizoda InDizajna, vidimo se u podne na RTL-u ⏰😍 Pogledajte, između ostaloga, uređenje moderne spavaće sobe i hodnika, nagradno...","Danas je s vama nova epizoda InDizajna, vidimo se u podne na RTL-u ⏰😍 Pogledajte, između ostaloga, uređenje moderne spavaće sobe i hodnika, nagradno...","Koji espresso aparat?","Koji espresso aparat?","Dizajneri se u procesu uređenja služe i moodboardima, inspirativnim kolažima kako bi se lakše definirao stil uređenja i predočilo kako elementi funkcioniraju...","Dizajneri se u procesu uređenja služe i moodboardima, inspirativnim kolažima kako bi se lakše definirao stil uređenja i predočilo kako elementi funkcioniraju...","Koji espresso aparat?","Koji espresso aparat?","Koji espresso aparat?","Tamne boje polako su se ušuljale u kuhinje 🖤 Na primjeru novog projekta studija Mirjane Mikulec možete vidjeti kako dominaciju tamnih tonova ublažiti i...","Nespresso kava i aparati",null,"Planirate (pre)urediti spavaću sobu i htjeli biste u nju smjestiti kutak za šminkanje? Inspirirajte se idejnim rješenjem by studio Mirjana Mikulec, a u...","Planirate (pre)urediti spavaću sobu i htjeli biste u nju smjestiti kutak za šminkanje? Inspirirajte se idejnim rješenjem by studio Mirjana Mikulec, a u...",null,"U novoj epizodi Tražimo dom s Mirjanom Mikulec očekuje nas Tin, mladi sportaš u potrazi za stanom 🏘 Budite s nama u subotu ujutro na RTL-u 📺 De’Longhi...","I love my Nespresso.","Koji espresso aparat?","Koji espresso aparat?","Ako niste uspjeli u prošlu subotu biti pred malim ekranima, sada možete pogledati propuštenu epizodu Tražimo dom s Mirjanom Mikulec 👇 Citroën BAUHAUS...","Koji espresso aparat?",null,null,null,"Zanima vas kako je teklo uređenje moderne kuhinje u tamnim tonovima i kako je tim Mirjane Mikulec uredio jednu dječju sobu? Pogledajte jučerašnju emisiju...","Zanima vas kako je teklo uređenje moderne kuhinje u tamnim tonovima i kako je tim Mirjane Mikulec uredio jednu dječju sobu? Pogledajte jučerašnju emisiju...","Koji espresso aparat?","U današnjoj epizodi InDizajna pogledajte između ostaloga, uređenje kuhinje u tamnim tonovima i jedne dječje sobe te nagradno preuređenje radno-gostinjske...","Aşkk ile..🤍🤍 🤍 🤍 🤍 #aşk#jetaime #monamour #coffee #cafe#hello #kahve#latte#flowers #mylove#lattemacchiato#photography #mylove#details @nespresso @karaca","Mlada tročlana obitelj nakon godina podstanarstva želi pronaći dom za sebe. Kako je tekla njihova potraga i kako su preuređene nekretnine koje su im ponudili...","JEDNOSTAVNI KEKSIĆI KOJI SE ŠPRICAJU - PRHKI KEKSIĆI OD ŽUMANJAKA - ŠTA RADITI SA ŽUMANCIMA?","Iznenadit će vas ono što povezuje ovih osam zagrebačkih stanova","Mali, kompaktani, lagani i divnog dizajna - Nespresso aparati! Od sada uz 20% popusta na izdvojene modele i vrijedan poklon. 😀 Izaberi svoj model ➡...","Koja je tvoja najdraža knjiga? 📖 Naša je svaka koju čitamo uz ukusnu šalicu kave iz LatteGo 5400 aparata. Zahvaljujući našem jednostavnom zaslonu, nećeš stići pročitati ni jednu stranicu već će te dočekati na izbor jedna topla, aromatična šalica espressa, cappuccina, macchiata - što god ti srce poželi. . . . #philips #philipshrvatska...","Tamne boje vratile su se u kuhinje, a najtraženije su crna i antracit siva 🖤 Upravo jednu takvu kuhinju moći ćete vidjeti u novoj epizodi InDizajna u...","Chat #16","Chat #16","Chat #16","U idućoj epizodi emisije Tražimo dom s Mirjanom Mikulec agenti su u potrazi za domom za tročlanu mladu obitelj :) 🏡 Budite s nama u subotu ujutro 👍 Citroën...","U idućoj epizodi emisije Tražimo dom s Mirjanom Mikulec agenti su u potrazi za domom za tročlanu mladu obitelj :) 🏡 Budite s nama u subotu ujutro 👍 Citroën...",null,null,"Black, just black… #nespresso #podlabs #blackcoffee #kopi-o","Uzmi trenutak za sebe i svoj omiljeni napitak. 🥰 Nespresso aparati za kavu stižu uz 20% popusta + poklon bon za kupnju kapsula! ☕","Nespresso promocija - Bijela tehnika i mali kućanski aparati - Promocije - POSEBNA PONUDA","Nespresso akcija listopad 2021","Kuća Feldkasten u Wildschönau AT6311.118.1","Ako ste propustili Aninu potragu za novim domom, sada to možete nadoknaditi 👇 BAUHAUS Hrvatska Svijet stolica Citroën Qualis salon namještaja - KLER...","☕ NOVO ➡ APARAT ZA KAVU PHILIPS EP3243/50 Uživajte u omiljenim napicima za posebne trenutke. Bez obzira na to želite li popiti espresso, kavu ili mlijeko, potpuno automatski aparat za espresso u tren oka će vam pripremiti savršen napitak, bez napora! Lako napravite aromatične napitke kao što su espresso, kava, cappuccino i latte macchiato...",null,null,"Kada imaš Nespresso Lattissima One-W #aparatzakavu, nije teško preživjeti #ponedjeljak. 😅🙂 Pronađi sve potrebne informacije vezane za proizvod na našem WEB SHOPU www.cialdissima-hrvatska.com ➡️ link u BIO! 👀🛒 #capsulalaboutiquedelcaffe #happymonday #dobrojutro #dobrojutrohrvatska #nespresso #nespressolattissima #kava #coffee","Promaknula vam je jučerašnja epizoda InDizajna? Ne brinite, sada ju možete pogledati kad god želite 👇 My Istria De’Longhi Hrvatska Modul Samsung BAUHAUS...","Promaknula vam je jučerašnja epizoda InDizajna? Ne brinite, sada ju možete pogledati kad god želite 👇 My Istria De’Longhi Hrvatska Modul Samsung BAUHAUS...","Koji espresso aparat?","OMILJENI BRENDOVI KOJE SVI KORISTITE! Cijene će divljati i u idućoj godini: Značajna poskupljenja već su krenula","Projet Nespresso #aftereffect #Illustrator #logo #Nespresso","U današnjoj epizodi InDizajna vodimo vas u luksuzu istarsku villu Merapi. Pogledajte uređenje dnevnog prostora u minimalističkom stilu i prostrane...","Junakinja današnje epizode Tražimo dom s Mirjanom Mikulec je Ana koja je u potrazi za većim stanom za svoju peteročlanu obitelj 🏠. Agenti su uspjeli pronaći...","Dnevni boravak središte je svakog doma i mnogima najomiljenija prostorija. Uređenje stoga planiramo do posljednjeg detalja 💛🖤 Kako izgleda ostatak ovog...","Nova epizoda Tražimo dom s Mirjanom Mikulec s vama je u subotu prijepodne na RTL-u :) Pogledajte je li Ana uspjela pronaći novi dom za svoju obitelj 🏠...","Nova epizoda Tražimo dom s Mirjanom Mikulec s vama je u subotu prijepodne na RTL-u :) Pogledajte je li Ana uspjela pronaći novi dom za svoju obitelj 🏠...",null,"Koji espresso aparat?","Krups aparat za kavu EA 815E70 Espresseria Auto Pisa S line","Kafa - sinonim za dobro jutro. LatteGo 5400 - sinonim za dobar život. Lakše nego ikad prije, napravi 12 ukusnih napitaka od svježih zrna kafe. Aromatični napitci kao što su espresso, kafa, cappuccino i latte macchiato čekaju te na jedan pritisak dugmeta. Jednom kad probaš kafu iz LatteGo aparata, nema povratka natrag! 😊 . . . #philips...","SARAH BUITENDACH: residing a wonderful period of fifty tones of gray","Kava - sinonim za dobro jutro. LatteGo 5400 - sinonim za dobar život. Lakše nego ikad prije, napravi 12 ukusnih napitaka od svježih zrna kave, aromatični napitci kao što su espresso, kava, cappuccino i latte macchiato čekaju te na jedan pritisak gumba. Jednom kad kušaš kavu iz LatteGo aparata, nema povratka natrag! 😊 . . . #philips...","Od svih kava koje možete pripremiti u toplini svog doma, Nespresso nam je najdraži. Nema tu previše filozofije. Jednostavno imaju premium proizvod i zato ih...","Kako je Nespresso postao najpopularnija kava na svijetu?","Moćan mališa. Detaljan test pogledajte na našem YouTube kanalu: https://www.youtube.com/watch?v=BaBXnKAwMZ0 #eponuda #kafa #dolcegusto #krups #nescafe...","Pridruži se Brad Pittu u ispijanju #Perfetto šalice kave koju za tebe priprema De'Longhi! 😍","Koji espresso aparat?","Apartman Bellevue Plage u Biarritz FR3450.165.3","SCENARIJ KAKAV SE VIĐA JEDNOM U 20 GODINA: Tri moćne svjetske kompanije već najavile dizanje cijena, ali to nije kraj",null,"Inflacija se zahuktava: tri moćne svjetske kompanije već najavile dizanje cijena! ‘Upravo gledamo scenarij koji se viđa jednom u dvadeset godina...‘","Vole li US-ovci popiti kavu ujutro?","Vole li US-ovci popiti kavu ujutro?","Inflacija se zahuktava: tri moćne svjetske kompanije već najavile nova dizanja cijena!","Inflacija se zahuktava: tri moćne svjetske kompanije već najavile dizanje cijena!","Akcija krups piccolo 3color 10mj 2021","Kuća Mariandl u Tamsweg im Lungau AT5580.200.1","Kuća Casa Miloni u Astano CH6999.125.1","Promaknula vam je prva epizoda nove sezone emisije Tražimo dom s Mirjanom Mikulec? Sada ju možete pogledati od početka do kraja 👇 BAUHAUS Hrvatska Citroën...","Promaknula vam je prva epizoda nove sezone emisije Tražimo dom s Mirjanom Mikulec? Sada ju možete pogledati od početka do kraja 👇 BAUHAUS Hrvatska Citroën...","Promaknula vam je prva epizoda nove sezone emisije Tražimo dom s Mirjanom Mikulec? Sada ju možete pogledati od početka do kraja 👇 BAUHAUS Hrvatska Citroën...",null,"Pet razloga zbog kojih volimo Nespresso","Zadovoljna.hr: Pet razloga zbog kojih volimo Nespresso","Trudnice zbor Vol. 49","Krups aparat za kavu Evidence EA891C10","Last #PumpkinSpiceCake #Nespresso #Vertuo Pod 👀. #Coffee #PortraitMode","Dobro jutro svima :) Danas vas očekuje nova sezona emisije Tražimo dom 🏠❤️ Vidimo se u 10.55 na RTL-u 📺 BAUHAUS Hrvatska Citroën Samsung Wimax doo Moj dom...","Drage pomozite, odgovaraju li koje jeftinike kapsule u ovaj aparat dolce gusto krups","Spremni za novu, 3. sezonu Tražimo dom? S vama smo od ove subote na RTL-u 🏠❤️📺 A ako ste u potrazi za idealnim domom i želite biti dio emisije, javite se...","Love love love my new toy! #nespresso #coffee #nespressocreatistapro","Apartman REX u Locarno CH6600.375.1","Aparati za kavu","Kava","Apartman Chesa Stè u Celerina CH7505.13.1","Još malo nas dijeli od nove sezone emisije Tražimo dom, gledajte nas od 16. listopada na RTL.hr! BAUHAUS Hrvatska Citroën Samsung Wimax doo Moj dom po mjeri...","Stiže nam treća sezona emisije Tražimo dom 💪🏘📺⏰ BAUHAUS Hrvatska Citroën Samsung Wimax doo Moj dom po mjeri OTP banka d.d. KARE Hrvatska Svijet stolica...","Stiže nam treća sezona emisije Tražimo dom 💪🏘📺⏰ BAUHAUS Hrvatska Citroën Samsung Wimax doo Moj dom po mjeri OTP banka d.d. KARE Hrvatska Svijet stolica...","Stiže nam treća sezona emisije Tražimo dom 💪🏘📺⏰ BAUHAUS Hrvatska Citroën Samsung Wimax doo Moj dom po mjeri OTP banka d.d. KARE Hrvatska Svijet stolica...","Stiže nam treća sezona emisije Tražimo dom 💪🏘📺⏰ BAUHAUS Hrvatska Citroën Samsung Wimax doo Moj dom po mjeri OTP banka d.d. KARE Hrvatska Svijet stolica...","Ovaj tjedan uživam u Ispirazione Novecento kavi iz limitirane edicije inspirirane tradicijom ispijanja espressa. S ovom kavom #Nespresso nas mirisom i okusom pokušava vratiti u Italiju 1940-ih, u atmosferu stand-up kafića tipičnih za taj period ! I mogu vam reći da su u tome i uspijeli! 🙌🏻 Posjetite Nespresso trgovine i uživajte u posebnoj...","Kako kod kuće napraviti savršenu kavu poput profesionalca","[NOVI LETAK] Veeeliki izbor sniženih S-Budget proizvoda u SPAR i INTERSPAR trgovinama! 😁 Odaberite slasne domaće pite koje se savršeno slažu uz espresso ili...","good morning pancake 🥞🍌 #homecooking #homesweethome #nespresso #cafe15☕","Doznali smo koju talijansku kavu piju ljudi sa stavom, a sada je dostupna i na našem tržištu!","Doznali smo koju talijansku kavu piju ljudi sa stavom, a sada je dostupna i na našem tržištu!","Nespresso kava i aparati","Kuća Villa REWIZ u Arzon FR2674.612.1","For invited users! You have violated the terms of the Facebook community. Please check your account properly on the site as you have violated our privacy...","Kava sa stavom - kava DIEMME","Nespresso kava i aparati","Tiramisu pohárkrém ☕️✨ #tiramisu #cake #cakedecorating #cakedesign #cakeart #cakephotography #coffee #nespresso #followforfollowback #likeforlikes","3 savjeta kako napraviti savršenu kavu poput profesionalca - Magazin za turizam i gastronomiju","Ciao, bella! Jeste li spremni na putovanje kroz vrijeme sa šalicom kave u ruci?","3 savjeta kako napraviti savršenu kavu poput profesionalca","Sta si u kavu stavila\"ljubav\" #mojakava# #nespressokava# #nespresso# #nespresso# #nespresso#","NAPOKON NAJBOLJE KREM ŠNITE I RECEPT KOJI UVIJEK USPIJEVA - LEGENDARNE KREMŠNITE - KREMPITA","Svjetski dan kave obilježite perfetto napitkom - PROGRESSIVE MAGAZIN","Svjetski dan kave obilježite perfetto napitkom","Novi Nespresso okusi odvest će vas u prve talijanske kafiće ili pak u moderno uređene barove! 🥰 Nespresso","Ona obilježava novi početak, ona unosi boju u naš život, ona budi naša osjetila... ona je kafa iz našeg LatteGo 5400 aparata! ☕ U nekoliko koraka možeš da prilagodiš okus od friških zrna i pretočiš želju u ukusnu šoljicu sa dodatkom svilenkasto glatke mliječne pjene. Sretan ti svjetski dan kafe! . . . #philips #philipsbosnaihercegovina...","Novi Nespresso okusi odvest će vas u prve talijanske kafiće ili pak u moderno uređene barove - PROGRESSIVE MAGAZIN","Ona označuje nov začetek, ona prinaša barvo v naše življenje, ona prebuja naše čute... ona je kava iz našega aparata LatteGo 5400! ☕ V nekaj korakih lahko prilagodite okus svežih zrn in pretočite željo v okusno skodelico z dodatkom svilnato gladke mlečne pene. Vesel svetovni dan kave! . . . #philips #philipsslovenija #philipscoffee #coffee...","Ciao, bella! Jeste li spremni na putovanje kroz vrijeme sa šalicom kave u ruci?","Ona obilježava novi početak, ona unosi boju u naš život, ona budi naša osjetila... ona je kava iz našeg LatteGo 5400 aparata! ☕ U nekoliko koraka možeš prilagoditi okus od svježih zrna i pretočiti želju u ukusnu šalicu sa dodatkom svilenkasto glatke mliječne pjene. Sretan ti svjetski dan kave! . . . #philips #philipshrvatska #philipshomeliving...","Povodom Međunarodnog dana kave Julius Meinl daruje bogati Starter paket","Svjetski dan kave obilježite perfetto napitkom","Uz tri savjeta napravi kavu poput profesionalca","Kako napraviti kavu poput profesionalca - fama news","Nespresso inissia i pixie akcija","Svjetski dan kave obilježite perfetto napitkom","Svjetski je dan kave! 😀 Uživaj u ispijanju kave u toplini svog doma uz vrhunske Philips, Nespresso, Krups, Russell Hobbs, Lavazza, DeLonghi i Gorenje...","Svjetski dan kave obilježite perfetto napitkom","Iz Nespressa su nam ponudili dva nova Limited Edition dodatka Inspirazione Italiana liniji - Inspirazione Millennio &amp; Inspirazione Novecento. Obje prožete talijanskom tradicijom, bogatog okusa i mirisnih nota - prva voćnih, a druga orašastih. Kako ja uvijek više naginjem bijelom nego crnom, tako je i ovaj put moj favorit ona ipak mrvicu nježnija...","Ciao, bella! Jeste li spremni na putovanje kroz vrijeme sa šalicom kave u ruci?","Sretan vam svjetski dan kave uz nove Nespresso okuse","Nespresso ima nove okuse i vode vas direktno u Italiju","Ciao, bella! Jeste li spremni na putovanje kroz vrijeme sa šalicom kave u ruci? - Ordinacija.hr","Jeste li spremni na putovanje kroz vrijeme sa šalicom kave u ruci?","Jeste li spremni na putovanje kroz vrijeme sa šalicom kave u ruci?","⏳ZADNJI DAN POPUSTA⏳ Nespresso Lavazza Favourite Mix ✔️100 KAPSULA ✔️Nespresso kompatibilne ✔️Kutija sadrži 5x20 kapsula 1. Espresso Vigoroso - Intenzitet 12 Jak i prirodan. Od brazilskih Arabica i Robusta pranih i prženih na visokim temperaturama, mješavina jakog karaktera i prirodne snage, stvorena je za intenzivnu i kremastu kavu, s...","Ciao, bella! Jeste li spremni na putovanje kroz vrijeme sa šalicom kave u ruci?","Ciao, bella! Jeste li spremni na putovanje kroz vrijeme sa šalicom kave u ruci? - Magazin za turizam i gastronomiju","Doznali smo koju talijansku kavu piju ljudi sa stavom, a sada je dostupna i na našem tržištu!","Cafe Frei u City Centar one East","Zalig ontbijtje zo ! @binahandmadebe #ontbijt #ceramic #keramiek #handmade #giveaway #happy #beautifulcoffee #nespressomoments #nespresso #cruesli #quakercruesli","#NESPRESSO Citiz&amp;milk Krups #VENTEArticleBEBE #DépotBébéKmayra #44385627 INSTAGRAM 👇 https://www.instagram.com/depot_kmayra page FACEBOOK 👇👇 https://www.facebook.com/pg/VENTE-Article-BEBE-244976682883641/posts","Ono kad ti je milka.hrvatska inspiracija za outfit 😁💜 podsjecam vas da jos uvijek traje nagradna igra u kojoj mozete osvojiti Panasonic televizore, Krups aparate za kavu i Tefal ekspres lonce 😊 nagrade su inspirirane novim limitiranim okusima, a moj najdrazi je “Our movie night in” 🎬💜 sve detalje nagradne igre te kako sudjelovati mozete...","Aparati za kafu","U jutarnjoj ludnici dok spremaš djecu za školu, nemoj zaboraviti da uhvatiš trenutak za sebe! 🥰 Započni dan na najbolji mogući način - napravi zdravi doručak, pripremi kafu iz Philips 5400 LatteGo aparata i spremno dočekaj sve dnevne obveze. . . . #philips #philipsbosnaihercegovina #philipshomeliving #coffee #lattego #philipscoffee","Jutranji mir in topla kava... ☕ Vsi vemo, kaj to pomeni, otroci so končno začeli hoditi v šolo! Za popolno jutranjo izkušnjo prelijte svojo kavo s svilnato gladko plastjo mlečne pene. Philips LatteGo z ustrezno hitrostjo ustvari kremasto peno idealne temperature in teksture, oblikovan pa je za delo z vsemi vrstami mleka. . . . #philips...","Jutarnji mir i topla kava... ☕ Svi znamo što to znači, djeca su napokon krenula u školu! Za potpuni jutarnji doživljaj prelijte svoju kavu svilenkasto glatkim slojem mliječne pjene. Philips LatteGo odgovarajućom brzinom stvara kremastu pjenu idealne temperature i teksture, a dizajniran je za rad sa svim vrstama mlijeka. . . . #philips...","[PHILIPS RASKOŠNA JESEN] Kava i toplo pecivo su razlog zašto nas vesele buđenja ujutro.☕️🥐 Sretni dobitnik našeg nagradnog natječaja imat će priliku uživati u savršenim jutrima uz Philips Espresso LatteGo aparate za kavu. 🥰 Više pročitajte na linku u opisu profila. 🤍 #mepasmall #lifestyleutrendu #philips #PhilipsHomeLiving #makelifebetter...","Series 3200 Popolnoma samodejni espresso kavni aparati EP3243/50 Popolnoma samodejni espresso kavni aparati","U jutarnjoj ludnici dok spremaš decu za školu, nemoj zaboraviti uhvatiti trenutak za sebe! 🥰 Započni dan na najbolji mogući način - napravi zdrav doručak, pripremi kafu iz Philips 5400 LatteGo aparata i spremno dočekaj sve dnevne obveze. . . . #philips #philipssrbija #philipshomeliving #coffee #lattego #philipscoffee","U jutarnjoj ludnici dok spremaš djecu za školu, nemoj zaboraviti uhvatiti trenutak za sebe! 🥰 Započni dan na najbolji mogući način - napravi zdravi doručak, pripremi kavu u LatteGo aparatu i spremno dočekaj sve dnevne obveze. . . . #philips #philipshrvatska #philipshomeliving #coffee #philipscoffee #lattego","Jutarnji mir i topla kafa... ☕ Svi znamo šta to znači, djeca su konačno krenula u školu! Za potpuni jutarnji doživljaj prelijte svoju kafu svilenkasto glatkim slojem mliječne pjene. Philips LatteGo odgovarajućom brzinom stvara kremastu pjenu idealne temperature i teksture, a dizajniran je za rad sa svim vrstama mlijeka. . . . #philips...","Gourmet - Stran 3 - Grazia Slovenija","Med jutranjo zmedo, medtem ko pripravljaš otroke za šolo ali vrtec, ne pozabi ujeti trenutek zase! 🥰 Začni dan na najboljši možen način - naredi si zdrav zajtrk, skuhaj kavo s Philipsovim aparatom 5400 LatteGo in pričakaj pripravljen vse dnevne obveznosti. . . . #philips #philipsslovenija #philipscoffee #coffee #coffeetime #lattego..."],["Hello.. #hello #monday #january #winter #day #work #longday #nespresso #cafe #coffee #like #likesforlike #love #lovely #happiness #happyme #haveaniceday #always #week #induljonahét #imadom","@bdatdzutim43 Kupis Dolce Gusto i adapter za Nespresso na ebay-u (oko 10€) i imas oba.","Izaberi espresso na svom LatteGo 5400 aparatu. Osim što će te razbuditi u trenu, espresso je niskokaloričan, a poboljšava ... #philips #philipshrvatska #philipshomeliving #coffee #philipscoffee #lattego","@dannyd1976 J/k I love my #nespresso machine 👍","Quote: roomko kaže: Prije kapsula sam imao poluautomatski aparat od nekih 1000-1500kn i mlinac od 300kn i kapsule (lavazza, illy, nespresso) su značajan napredak u odnosu na to. Nagađam da je rang 2-3k kn i dalje lošija kava od kapsula. Sad gledam da","Quote: chavez ding kaže: nakon par godina stajanja izvukao krups piccolo kp1002 i pusta vodu na onoj brtvi gdje dolazi spremnik. gdje u zgu mogu kupiti tu brtvu? https://www.nestle.hr/ask-nestle/ndg Na dnu je popis pa zovi redom ako netko ima odmah","Prije kapsula sam imao poluautomatski aparat od nekih 1000-1500kn i mlinac od 300kn i kapsule (lavazza, illy, nespresso) su značajan napredak u odnosu na to. Nagađam da je rang 2-3k kn i dalje lošija kava od kapsula. Sad gledam da opet odem na","nakon par godina stajanja izvukao krups piccolo kp1002 i pusta vodu na onoj brtvi gdje dolazi spremnik. gdje u zgu mogu kupiti tu brtvu?","morning buzz #volluto #nespresso #caffeinefix #coffee #conradmanila","LatteGo, sustav za mlijeko koji se najbrže čisti. Lako napravite aromatične napitke kao što su espresso, kava, cappuccino i latte Stavi u košaricu Lako napravite aromatične napitke kao što su espresso, kava i cappuccino Stavi u košaricu 5 ukusnih","aparat za kavu, Doze za mašinu za kavu (Nespresso) uz doplatu). Kada, odv. WC. Parketni pod. Balkonski namještaj. Vrlo lijep panoramski pogled na planine i skijašku stazu. Na raspolaganju: perilica za rublje, glačalo, sušilo za kosu. Internet","#autumn #season #Holiday #vacation #sunset #sunrise #sea #wave #happiness #halloween #party #life #Beach #yacht #sailing #coffee #nespresso #wine #ron #anesthesia #operation #Croatia #oman #zilina #slowmotion #hospital #timelapse #Christmasmarket","Nespresso Boutique u Zagrebu - idealna adresa za kupovinu poklona Nespresso Boutique u centru Zagreba osvojio nas je ponudom i interijerom, a pronašle smo i super božićne poklone za sve kavoljupce Imate li među ukućanima i prijateljima kavoljupca,","Pripremit ćete kavu pomoću De’Longhi aparata, a zatim je preliti u veću keramičku šalicu ili čašu. Jedna žličica javorovog ... Za najbolji okus, recepte isprobajte pripremajući De'Longhi aparatima za kavu.  Više infomacija provjerite putem poveznice","Pripremit ćete kavu pomoću De’Longhi aparata, a zatim je preliti u veću keramičku šalicu ili čašu. Jedna žličica javorovog sirupa i četvrtina žličice cimeta sljedeći su dodatak. Konačno, zapijenjena količina mlijeka po želji uz malo mljevenog cimeta","Nespresso Hrvatska: Lattissima One Silky White Nespresso Tekst: Krešimir Pavličić Foto: Notino, FABUspot, Afrodita, L’Occitane, L’Adria, Melvita, Freywille, Nespresso","Više na našem webu https://www.modnialmanah.com/ovo-su-3-recepta-za-pripremu-jedinstvene-zimsku-kave/ #modnialmanah #almapremerlzoko #coffee #kava De’Longhi Hrvatska #DeLonghi #gastro #recept","True love 💓 #coffee #nespresso ☕☕☕","U njoj se po outlet cijenama prodaju proizvodi brendova: Tefal, Rowenta, EMSA, Kaiser, Krups, Lagostina, Moulinex, Silit i WMF. Saznajte više...","Pripremit ćete kavu pomoću De’Longhi aparata, a zatim je preliti u veću keramičku šalicu ili čašu. Jedna žličica javorovog ... Za najbolji okus, recepte isprobajte pripremajući De'Longhi aparatima za kavu. Više infomacija provjerite putem poveznice:","Pripremit ćete kavu pomoću De’Longhi aparata, a zatim je preliti u veću keramičku šalicu ili čašu. Jedna žličica javorovog ... Izvor: Promo Za najbolji okus, recepte isprobajte pripremajući De'Longhi aparatima za kavu. Više infomacija provjerite","LatteGo napicima s mlijekom dodaje svilenkasto glatku pjenu, lako se postavlja i može se očistiti za samo 15 sekundi. 3.999,00 kn Dizajnirano za superiorno usisavanje prašine. Allergy Lock osigurava zadržavanje sitne prašine. Philips PowerPro Expert","To se zove superautomatski aparat za kavu poput De'Longhi Dinamica Plus koji ja koristim. Kava se u društvu često percipira kao napitak koji potiče produktivnost. Što Vi mislite o tome? Pojam jutarnje kave nije bez razloga postao nešto s čime se može","Uz kekse sam odlučio uključiti nekoliko novih limitiranih Nespresso prazničnih okusa jer ovi Cantuccini su najbolji upravo ... #darkovawebkuharica #nespresso #nespressomoments #nespressolovers #nespressofestive #giftsoftheforest #cantuccini","Pripremit ćete kavu pomoću De’Longhi aparata, a zatim je preliti u veću keramičku šalicu ili čašu. Jedna žličica javorovog sirupa i četvrtina žličice cimeta sljedeći su dodatak. Konačno, zapijenjena količina mlijeka po želji uz malo mljevenog cimeta","Pripremit ćete kavu pomoću De’Longhi aparata, a zatim je preliti u veću keramičku šalicu ili čašu. Jedna žličica javorovog sirupa i četvrtina žličice cimeta sljedeći su dodatak. Konačno, zapijenjena količina mlijeka po želji uz malo mljevenog cimeta","Brend koji je sredinom 1980-ih godina napravio pravu malu revoluciju kada je priprema kave u pitanju je Nespresso , a zahvaljujući kojem možemo uživati u omiljenom napitku bilo kada i bilo gdje. Po čemu su aparati za pripremu kave ovog brenda","Philips EP4341/50 Series 4300 LatteGo ima spremnik za mlijeko LatteGo, opciju od 8 napitaka za kavu i patentirane keramičke mlince.","Male 3 recepta za zimsku kavu s kojima će blagdani biti slađi Foto: De'Longhi Vrijeme blagdana pravi je trenutak za neke ... Pripremit ćete kavu pomoću De’Longhi aparata, a zatim je preliti u veću keramičku šalicu ili čašu. Jedna žličica javorovog","#mycuppa #doitfluid #coffee #coffeeaddict #coffeegram #coffeelover #coffeetime #coffeeaddiction #expresso #nespresso #coffeeflavors morningjava #morningmust #morningmotivation #homecafe #homebarista #coffeetime #coffeeholic #coffeedaily","U njoj se po outlet cijenama prodaju proizvodi brendova: Tefal, Rowenta, EMSA, Kaiser, Krups, Lagostina, Moulinex, Silit i WMF. To je već druga takva trgovina tvrtke SEB. Prva je bila otvorena prošle godine u Designer’s outlet centru u Zagrebu. U","Nespresso limitirana blagdanska kolekcija Ovoga Božića, Nespresso je u suradnji s kolumbijskom modnom dizajnericom ... Limitirana serija „Darovi šume\" je dostupna na stranici www.nespresso.hr i u Nespresso buticima u Ilici 19 i Arena Centru.","Johanna Ortiz unijela je ljepotu šume u Nespresso limitiranu blagdansku kolekciju Brend kave poznat po posvećenosti ... Limitirana serija „Darovi šume“ je dostupna na stranici www.nespresso.hr i u Nespresso buticima u Ilici 19 i Arena Centru.","Johanna Ortiz unijela je ljepotu šume u Nespresso limitiranu blagdansku kolekciju Ovoga Božića, Nespresso je u suradnji s ... Limitirana serija „Darovi šume“ je dostupna na stranici www.nespresso.hr i u Nespresso buticima u Ilici 19 i Arena Centru.","Tag #nespresso označava ova pitanja nespresso Tag #nespresso označava ova pitanja 1 odgovor 19 pregleda","Nespresso ima limitiranu blagdansku kolekciju i potpisuje je poznata modna dizajnerica Nespresso ima limitiranu blagdansku ... Limitirana serija „Darovi šume“ je dostupna na stranici www.nespresso.hr i u Nespresso buticima u Ilici 19 i Arena Centru.","Johanna Ortiz unijela je ljepotu šume u Nespresso limitiranu blagdansku kolekciju  Brend kave poznat po posvećenosti prirodi u suradnji s Johannom Ortiz lansirao je limitiranu kolekciju \"Darovi šume\" Ovoga Božića, Nespresso je u Johanna Ortiz unijela","Dizajnerica Johanna Ortiz kreirala Nespresso limitiranu blagdansku kolekciju Ovoga Božića, brend Nespresso je u suradnji s kolumbijskom modnom dizajnericom Johannom Ortiz lansirao Festive kolekciju, inspiriranu darovima šume. Kolekcija utjelovljuje","Što je kolumbijska modna dizajnerica Johanna Ortiz dizajnirala za Nespresso? https://t.co/ymJ4FV6uGO #pitanje #odgovor https://ift.tt/3dSa6PN","Dizajnerski komad koji ovog Božića želimo u svom domu Brend kave poznat po posvećenosti prirodi u suradnji s Johannom Ortiz lansirao je limitiranu kolekciju “Darovi šume” Ovoga Božića, Nespresso je u suradnji s kolumbijskom modnom dizajnericom","Johanna Ortiz unijela je ljepotu šume u Nespresso limitiranu blagdansku kolekciju - Magazin za turizam i gastronomiju ... Limitirana serija „Darovi šume“ je dostupna na stranici www.nespresso.hr i u Nespresso buticima u Ilici 19 i Arena Centru.","u Europi te su se tako pozicionirali kao jedina netehnološka tvrtka iz jugoistočne [...] vijesti 15/12/2021  Brend kave poznat po posvećenosti prirodi u suradnji s Johannom Ortiz lansirao je limitiranu kolekciju “Darovi šume” Ovoga Božića, Nespresso","✔️Síguenos ✔️Haz RT + #DeLonghiFnacHome, dinos a qué amigo le debes un ☕️ y esta cafetera superautomática De'Longhi puede ser tuya 😉 BBLL https://t.co/IhWzeceW0R ¡El miércoles 15 anunciamos ganador! https://t.co/UTboAq7jUv","Webshop: ChiaCups Studio View this post on Instagram A post shared by Filipa Sorko (@filipaasorko) View this post on Instagram A post shared by Filipa Sorko (@filipaasorko) Nespresso aparat za kavu Uz Nescafe Dolce Gusto aparat za kavu samo vas jedan","Ako vam je promaknula jučerašnja, posljednja epizoda u ovoj sezoni InDizajna, sada je možete pogledati :) My Istria De’Longhi Hrvatska Credo centar Meblo Trade Moj dom po mjeri Lesnina XXXL BAUHAUS Hrvatska TEXO Hrvatska","LatteGo, sustav za mlijeko koji se najbrže čisti. Lako napravite aromatične napitke kao što su espresso, kava, cappuccino i latte Stavi u košaricu 12 ukusnih napitaka od svježih zrna kave, lakše nego ikad prije. LatteGo, sustav za mlijeko koji se","Ne propustite posljednju ovosezonsku epizodu vaše omiljene emisije 😍 Pogledajte u videu što vas očekuje u današnjem InDizajnu 👇 My Istria De’Longhi Hrvatska Credo centar BAUHAUS Hrvatska Lesnina XXXL TEXO Hrvatska Meblo Trade Moj dom po","Krups ima fiksnu jedinicu, nalik Jura aparatima - dakle ne možete proprati rukom i izvaditi nego kem. tablete za rastapanje uljnih naslaga kasnije, što poskupljuje i nije odveć praktično. Philips/Saeco/Gaggia, Bosch/Siemens i DeLonghi imaju jedinice","Jel valjaju šta ovi Delonghi i Krups jel ima servisa u Zagrebu ? Gledao sam da neki Philipsovi aparati imaju taj tzv. filter vode što je meni ok, jer mi je voda jako tvrda. A neki imaju onaj prednji kotačić kojim reguliraš količinu kave ako se","🍴 Dodatnih 10% popusta na blagajni za brendove: EMSA, Kaiser, Krups, Moulinex, Rowenta, Silit i Tefal 🍴 Dodatnih 30% popusta na blagajni za brendove: Lagostina i WMF *Popust se neće primijeniti za proizvode koji su već na akciji. Požuri jer akcija","i love u ! 😂#morningfix #nespresso","Ovih blagdana složite idealan Nespresso paket za najveće kavoljupce Strastvene kavopije će se složiti – kada je u pitanju odabir darova za kavoljupce, jedino Nespresso puninom okusa i opojnošću mirisa zadovoljava i najstrože kriterije. Donosimo ideju","Koju su vlasnici odabrali te kako izgleda kupaonica nakon realizacije, pogledajte u posljednjoj epizodi ove sezone InDizajna u subotu prijepodne na RTL-u 📺🙌 My Istria De’Longhi Hrvatska Credo centar BAUHAUS Hrvatska Lesnina XXXL TEXO Hrvatska","Reposted from @paolotroilo54 WTF . . . . . . . . . . . . . . . #wtf #fingerpainting #contemporaryart #body #shoulders #art #privatecollection #museum #figurativeart #paintedwithfingers #nespresso","Tehnika Nespresso aparat za kavu Aparat za kavu jedan je od poklona koji će trajati gotovo vječno. Odličan je izbor za entuzijaste kave, a s novim dizajnom, koji daje dodir elegancije ritualima ispijanja kave širom svijeta, CitiZ će zadovoljiti i","Tehnika Nespresso aparat za kavu Aparat za kavu jedan je od poklona koji će trajati gotovo vječno. Odličan je izbor za entuzijaste kave, a s novim dizajnom, koji daje dodir elegancije ritualima ispijanja kave širom svijeta, CitiZ će zadovoljiti i","Mali, lijep, funkcionalan i praktičan! Idealan poklon🎁 za blagdane🎄- Nespresso aparat za mirisnu ☕️kavu. &lt;3 &gt;&gt; https://bit.ly/3ECKzGf Izbor iz ponude, akcija traje do 31.12.2021. ili isteka zaliha","Mali, kompaktani, lagani i divnog dizajna - Nespresso aparati! Od sada uz 20% popusta na izdvojene modele i vrijedan poklon. 😀 Btw. posjeti nas u Arena Parku u petak 16-20h ili subotu 12-18h na Nespresso promociji! Izaberi svoj model ➡","Oliver, Lijepa.hr, Tom Tailor, Nespresso, Next, Karting, Amazinga,… Dostupne su 3 različite kartice: • SuperCard Black za kupovinu kod više od 80 partnera, • Super E, za online kupovinu kod više od 80 partnera, i • SuperCard White s preko 30","Akcijske cijene za Lavazza Nespresso kapsule na www.lovin.hr #Lavazza #Nespresso #Lovin #LovinWebshop #CoffeeMoment #CoffeeBreak #Akcija","Love my job 🎄 #nespresso #pinterest #sapin #sapindenoel #deco #decodeclasse #portre #peinture #peinture #dixdoigts #capsules #maitresse #maitressedecole #maitressedematernelle #imagination","Za savršen početak dana Propali shopping planovi uz SuperCard nisu mogući kada tražite nešto za prijateljicu, sestru, majku, baku ili voljenu osobu, brata, tatu, djeda – jer Nespresso je upravo savršen odabir za sve kavoljupce! Ovo je idealno mjesto","Subotnju epizodu emisije Tražimo dom s Mirjanom Mikulec sada možete pogledati i online :) Citroën Samsung BAUHAUS Hrvatska Momax Hrvatska OTP banka d.d. De’Longhi Hrvatska","Subotnju epizodu emisije Tražimo dom s Mirjanom Mikulec sada možete pogledati i online :) Citroën Samsung BAUHAUS Hrvatska Momax Hrvatska OTP banka d.d. De’Longhi Hrvatska","#CoffyLovers #thecoffyway #thecoffywaycroatia #coffeelover #coffybreak #coffetime #coffeeholic #coffeeshop #coffeeaddict #kapsule #kompatibilne #lavazza #lavazzapoint #lavazzaamodomio #lavazzafirma #juliusmeinl #nespresso #nescafedolcegusto","Pogodi što smo mi jutros pronašli u svojoj čizmici - Philips LatteGO 5400! Elegantno dizajnirani aparat koji spravlja čak ... #philips #philipshrvatska #philipshomeliving #coffee #philipscoffee #lattego","Essenza Plus Black - Nespresso Hrvatska Prikladan za svaku kuhinju, Nespresso Essenza Plus je uvijek tu kada Vam zatreba. Bilo za Espresso, Lungo, Americano ili pak čaj s opcijom vruće vode. Ovdje ... 1 DODAJTE U KOŠARICU Prikladan za svaku kuhinju,","Svima dobro poznati TEFAL, ROWENTA, MOULINEX, KRUPS, LAGOSTINA, WMF, SILIT, KAISER...🤩 Sada imaš priliku poststai dio ovog velikog tima!😎 ZADUŽENJA: 🛒prezentacija, prodaja robe i savjetovanje kupaca 🛒nadopuna i slaganje robe 🛒rad na blagajni","od druge, vidjet cemo hoce li me zvati za booster Za BF sam bila i vise nego razumna, samo Nespresso aparat, tajice i G&amp;G skincare Poslano sa mog SM-G991B koristeći Tapatalk","De’Longhi Hrvatska","Za savršen početak dana Propali shopping planovi uz SuperCard nisu mogući kada tražite nešto za prijateljicu, sestru, majku, baku ili voljenu osobu, brata, tatu, djeda – jer Nespresso je upravo savršen odabir za sve kavoljupce! Ovo je idealno mjesto","Ovu epizodu snimale smo uz savršenu šalicu kave iz Philips LatteGo 5400 aparata za kavu, te se ovim putem zahvaljujemo upravo Philipsu što nas podržava u glasnom progovoru upravo o ovako važnim temama. Naša sugovornica je @snazna.mama","Black Friday i dalje traje! #anangroup #blackfriday #capsule #nespresso #espresso #dolcegusto","Za savršen početak dana Propali shopping planovi uz SuperCard nisu mogući kada tražite nešto za prijateljicu, sestru, majku, baku ili voljenu osobu, brata, tatu, djeda – jer Nespresso je upravo savršen odabir za sve kavoljupce! Ovo je idealno mjesto","KUPILI SMO KAFE APARAT I SUPER JE De'Longhi Magnifica S ECAM 22.110.B potpuno automatski aparat za kavu s mlaznicom za pjenjenje mlijeka za cappuccino i integriranim mlincem, s gumbima za izravni odabir espressa i okretnom kontrolom, funkcija 2","De’Longhi Hrvatska","  Za savršen početak dana Propali shopping planovi uz SuperCard nisu mogući kada tražite nešto za prijateljicu, sestru, majku, baku ili voljenu osobu, brata, tatu, djeda – jer Nespresso je upravo savršen odabir za sve kavoljupce! Ovo je idealno","Za savršen početak dana Propali shopping planovi uz SuperCard nisu mogući kada tražite nešto za prijateljicu, sestru, majku, baku ili voljenu osobu, brata, tatu, djeda – jer Nespresso je upravo savršen odabir za sve kavoljupce! Ovo je idealno mjesto","Za savršen početak dana Propali shopping planovi uz SuperCard nisu mogući kada tražite nešto za prijateljicu, sestru, majku, baku ili voljenu osobu, brata, tatu, djeda– jer Nespresso je upravo savršen odabir za sve kavoljupce! Ovo je idealno mjesto","Za savršen početak dana Propali shopping planovi uz SuperCard nisu mogući kada tražite nešto za prijateljicu, sestru, majku, baku ili voljenu osobu, brata, tatu, djeda – jer Nespresso je upravo savršen odabir za sve kavoljupce! Ovo je idealno mjesto","Za savršen početak dana Propali shopping planovi uz SuperCard nisu mogući kada tražite nešto za prijateljicu, sestru, majku, baku ili voljenu osobu, brata, tatu, djeda – jer Nespresso je upravo savršen odabir za sve kavoljupce! Ovo je idealno mjesto","  Objavu dijeli Nespresso (@nespresso) Osim vrhunskih aparata, šarene kapsule raznih aroma iz limitirane blagdanske serije, jamče kako će svako nepce imati odličan početak dana. Ako tome dodamo kako je Božić razdoblje ispunjeno prepoznatljivim","  Objavu dijeli Nespresso (@nespresso) Osim vrhunskih aparata, šarene kapsule raznih aroma iz limitirane blagdanske serije, jamče kako će svako nepce imati odličan početak dana. Ako tome dodamo kako je Božić razdoblje ispunjeno prepoznatljivim","Za savršen početak dana Propali shopping planovi uz SuperCard nisu mogući kada tražite nešto za prijateljicu, sestru, majku, baku ili voljenu osobu, brata, tatu, djeda – jer Nespresso je upravo savršen odabir za sve kavoljupce! Ovo je idealno mjesto","Fotografija Nespresso Grad Beč svojevrsni je pionir među europskim prijestolnicama kada je u pitanju poticanje ozelenjivanja privatnih stambenih i poslovnih zgrada. Kao dio svoje zelene strategije, naime, gradske vlasti u Beču izdašno sufinanciraju","Za savršen početak dana Propali shopping planovi uz SuperCard nisu mogući kada tražite nešto za prijateljicu, sestru, majku, baku ili voljenu osobu, brata, tatu, djeda – jer Nespresso je upravo savršen odabir za sve kavoljupce! Ovo je idealno mjesto","Sublime. #nespresso #vertuo #todoestabien","Ako ste u subotu propustili Kristininu potragu za novim domom, sada možete pogledati cijelu epizodu :) BAUHAUS Hrvatska Citroën Samsung De’Longhi Hrvatska JYSK Hrvatska","Ako ste u subotu propustili Kristininu potragu za novim domom, sada možete pogledati cijelu epizodu :) BAUHAUS Hrvatska Citroën Samsung De’Longhi Hrvatska JYSK Hrvatska","Sve što će vam trebati je - vaša najdraža kava iz našeg LatteGo aparata, pumpkin spice začin, pire od bundeve i malo ... #philips #philipshrvatska #philipshomeliving #coffee #philipscoffee #lattego","Sve što vam je potrebno su - vaša najdraža kafa iz našeg LatteGo aparata, pumpkin spice začin, pire od bundeve i malo ... #philips #philipssrbija #philipshomeliving #coffee #lattego #philipscoffee","Za savršen početak dana Propali shopping planovi uz SuperCard nisu mogući kada tražite nešto za prijateljicu, sestru, majku, baku ili voljenu osobu, brata, tatu, djeda – jer Nespresso je upravo savršen odabir za sve kavoljupce! Ovo je idealno mjesto","Jučerašnja epizoda InDizajna već danas je online 👇💕 My Istria De’Longhi Hrvatska Perfecta Dreams Hespo Samsung BAUHAUS Hrvatska Lesnina XXXL TEXO Hrvatska Meblo Trade Moj dom po mjeri","LatteGo napicima s mlijekom dodaje svilenkasto glatku pjenu, lako se postavlja i može se očistiti za samo 15 sekundi. Kako bi čišćenje bilo praktično, LatteGo, pladanj i spremnik za ostatke mljevene kave možete prati perilici posuđa. Time se štedi","s  partnerima: Acer, Agronom, Alcazar, Amazfit, Ariete, Beurer, Black&amp;Decker, Braun/OralB, Breville, Brother, Creative, Ecovacs, Electrolux, Franck, Gorenje, Haier, Inventor, Julius Meinl, Končar, Krups, Meanit, Motorola, MS Energy, Muvo, Nespresso,","https://www.sancta-domenica.hr/mali-...kava-crni.html https://www.sancta-domenica.hr/mali-...ica-style.html https://www.sancta-domenica.hr/mali-...sso-retro.html Imam aparat Krups na kapsule, ali kapsule baš i nisu povoljne, dok je aparat čisto ok.","🖤Nespresso je napravio revoluciju u kreiranju aparata za kavu za \"po kući\" Ovog #CrnogPetka dodatno uštedite u Emmezeta centrima ili web shopu 🖤 👉👉 https://bit.ly/BlackW_21 Članovi kluba s minimalno 5️⃣0️⃣ bodova dodatno štede! #CrniPetak","Otvaramo vam i vrata istarske vile okružene šumom 🏡 My Istria De’Longhi Hrvatska Perfecta Dreams Hespo Samsung BAUHAUS Hrvatska Lesnina XXXL TEXO Hrvatska Meblo Trade Moj dom po mjeri","Junakinja današnje epizode je Kristina koja želi novi dom u istočnom dijelu Zagreba 🏠 Koje nekretnine su joj pronašli agenti emisije Tražimo dom s Mirjanom Mikulec pogledajte u 10.30 na RTL-u :) BAUHAUS Hrvatska Citroën Samsung De’Longhi","Junakinja današnje epizode je Kristina koja želi novi dom u istočnom dijelu Zagreba 🏠 Koje nekretnine su joj pronašli agenti emisije Tražimo dom s Mirjanom Mikulec pogledajte u 10.30 na RTL-u :) BAUHAUS Hrvatska Citroën Samsung De’Longhi","Mali sneak peak u jednu sasvim posebnu kuću u Hrvatskoj :) 🔝 Kako izgleda ostatak interijera, kao i impresivan eskterijer, pogledajte u novoj epizodi InDizajna koja je s vama u nedjelju ujutro na RTL-u 🙌 My Istria De’Longhi Hrvatska Perfecta","Black Friday akcija do -37% Nespresso, izbor iz ponude - Aktualno Black Friday akcija do -37% Nespresso, izbor iz ponude ... godine skenirate QR kod ili posjetite stranicu www.nespresso.hr/hr/trade-reg, registrirate svoj aparat i priložite","Sadržaj je nastao u suradnji s tvrtkama Biobaza i De’Longhi .  ","U novoj epizodi Tražimo dom s Mirjanom Mikulec upoznat ćemo Kristinu koja je u potrazi na novim domom :) Budite s nama u subotu prijepodne na RTL-u 📺⏰ BAUHAUS Hrvatska Citroën Samsung De’Longhi Hrvatska JYSK Hrvatska","Krups Arabica EA811810 aparat za kavu Automatski espresso aparat u elegantnoj crno-srebrnoj boji uvijek će vam pripremiti izuzetno piće čistog okusa i divne arome. Krups Arabica EA811010 sa praktičnim i kompaktnim dizajnom nadopunjuju intuitivne","I love my NESPRESSO!","aparat za kavu, Doze za mašinu za kavu (Nespresso) uz doplatu). Tuš, odv. WC. Grijanje na plin. Balkon 6 m2. Balkonski namještaj. Lijep pogled u daljini na more. Na raspolaganju: glačalo, sušilo za kosu. Internet (wireless LAN [WLAN], besplatan).","720 x 1600 PIC Procesor: Octa Core 2.35GHz Glavna kamera: 48.0 MP + 5.0 MP + 2.0 MP + 2.0 MP EXPRESS LONAC SECURE 6 L Ekspres lonac zapremine 6 l Velika ručka za jednostavno rukovanje i lakše držanje Aparat za kavu Dolce gusto;Piccolo XS Krups Dolce","Krups zbog načina čišćenja..... Javim što se kupilo. lp","S mojim Philips LatteGo 5400 sam vas već upoznala, a naša ljubav još traje... Na portalu vas čeka moja priča o kavama i ljudima koji ih piju. Inspirirana stvarnim ljudima... . @philipshomeliving_hr . Foto: @chumko . . #jasamsupermama #supermamehr","Nespresso akcija prosinac 2021 U ponudi imamo 20 proizvoda na Nespresso akcija prosinac 2021 Nespresso akcija prosinac 2021 Aparat za kavu Pixie Titan Uključuje komplet od 14 kapsula Pixie je razvijen kao SMART model asortimana u iznenađujuće malom","LatteGo napicima s mlijekom dodaje svilenkasto glatku pjenu, lako se postavlja i može se očistiti za samo 15 sekundi. Sklop za kuhanje u srcu je svakog potpuno automatskog aparata za kavu i trebali biste ga redovito čistiti. Odvojivi sklop za kuhanje","Krups Krups Spremnik za vodu od 3 l Cijena Kapacitet spremnika za vodu: 1.2 l Automatsko gašenje nakon 5 min Boja: Crvena Spremnik za vodu od 1,7 l Cijena Spremnik za vodu od 800 ml 188,90 KM XL funkcija (do 300 ml) 269,90 KM XL funkcija (do 300 ml)","Od nedavno u ponudi imaju i kavu u kompostabilnim i biorazgradivim kapsulama koje su kompatibilne Nespresso aparatima što olakšava pripremu kave, ali je čini i ekološki održivijom.  Preslatki kafić na adresi Horvaćanska 23A poseban je i po tome što","Ako ste propustili pogledati subotnje izdanje Tražimo dom s Mirjanom Mikulec i Tinovu potragu za idealnim domom, sada to možete nadoknaditi 🏡⤵ De’Longhi Hrvatska Samsung JYSK Hrvatska BAUHAUS Hrvatska Citroën","Ako ste propustili pogledati subotnje izdanje Tražimo dom s Mirjanom Mikulec i Tinovu potragu za idealnim domom, sada to možete nadoknaditi 🏡⤵ De’Longhi Hrvatska Samsung JYSK Hrvatska BAUHAUS Hrvatska Citroën","#ZADstoreKiseljak #Krups #Nescafe #BiH","Krema goodness 😋 #coffeecheers #nespresso","Sve nam je top, ali ove otvorene grede naročito 😍😍😍 My Istria De’Longhi Hrvatska","Cijeli mjesec je u znaku crne kave ☕️! Najbolje Nespresso aparate potražite u Emmezeti. &gt;&gt; https://bit.ly/3qZm2XZ","Ako niste stigli jučer, sada možete pogledati novu epizodu InDizajna :) My Istria De’Longhi Hrvatska Moj dom po mjeri BAUHAUS Hrvatska Lesnina XXXL Samsung Perfecta Dreams Hespo Meblo Trade","Pijem iskljucivo espresso i trenutno koristim illiy forte i nespresso napoli kapsule. Sada imam dvojbe da li ici na full automatiku Phillips/De Longhi ili sa amazona uzet Sage Barista Pro , znaci zanima me cjenovni rang do cca max 5000kn i sa cime cu","Alebrije loco 🤪 . . . . #art #arte #nespresso #capsulas #alebrijes #alebrijeloco #onlyart #happylife","Moze anon Zene znate li gdje mogu kupit kapsule za dolce gusto, ali da nisu nes ili starbucks nego malo jeftinije, ali da odgovaraju za picollo krups aparat. Hvala","vama nova epizoda InDizajna, vidimo se u podne na RTL-u ⏰😍 Pogledajte, između ostaloga, uređenje moderne spavaće sobe i hodnika, nagradno preuređenje blagovaonice te prošetajte s nama do Istre gdje vas očekuje prekrasna vila 🏡 My Istria De’Longhi","vama nova epizoda InDizajna, vidimo se u podne na RTL-u ⏰😍 Pogledajte, između ostaloga, uređenje moderne spavaće sobe i hodnika, nagradno preuređenje blagovaonice te prošetajte s nama do Istre gdje vas očekuje prekrasna vila 🏡 My Istria De’Longhi","Quote: branee kaže: Ciljao sam na onaj Krups iz linka https://www.elipso.hr/mali-kucanski/.../KRUPS-EA8170/ prvenstveno radi dobre cijene. Nisam trenutno sklon nekom duplo skupljem aparatu... Kao velika mu je mana thermoblock i kamenac....zar","Ciljao sam na onaj Krups iz linka https://www.elipso.hr/mali-kucanski/.../KRUPS-EA8170/ prvenstveno radi dobre cijene. Nisam trenutno sklon nekom duplo skupljem aparatu... Kao velika mu je mana thermoblock i kamenac....zar je to zaista tako,","A kako izgleda realizacija ova dva moodboarda dnevnog boravka u stanovima koji su ponuđeni Tinu, pogledajte u današnjoj epizodi Tražimo dom s Mirjanom Mikulec Budite ispred malih ekrana u 11.20 📺⏰ De’Longhi Hrvatska Samsung JYSK Hrvatska","A kako izgleda realizacija ova dva moodboarda dnevnog boravka u stanovima koji su ponuđeni Tinu, pogledajte u današnjoj epizodi Tražimo dom s Mirjanom Mikulec Budite ispred malih ekrana u 11.20 📺⏰ De’Longhi Hrvatska Samsung JYSK Hrvatska","Quote: branee kaže: https://www.elipso.hr/mali-kucanski/.../KRUPS-EA8170/ Žena me nagovara, bliži mi se rodjendan a ja full zadovoljan sa sa svojim starim polu automatskim 10tak g. starim Gaggia Evolution aparatom. Espresso kao u dobrom kafiću,","@branee Imao sam slican krups model EA8010 koji je pa skoro isti aparat kao i ovaj koji ti hoces. Utisci o njemu su sledeci. Pravio je solidnu kafu.Mana mu je mlin sa samo tri pozicije stelovanja krupnoce mlina i samo ciscenje aparata ide preko onih","https://www.elipso.hr/mali-kucanski/.../KRUPS-EA8170/ Žena me nagovara, bliži mi se rodjendan a ja full zadovoljan sa sa svojim starim polu automatskim 10tak g. starim Gaggia Evolution aparatom. Espresso kao u dobrom kafiću, uvijek odličan i kako","polako su se ušuljale u kuhinje 🖤 Na primjeru novog projekta studija Mirjane Mikulec možete vidjeti kako dominaciju tamnih tonova ublažiti i vješto kombinirati sa svijetlijima, kao i zanimljivim uzorcima i materijalima Moj dom po mjeri De’Longhi","Koje su vam se pokazale najbolje Nespresso kapsule za cafe latte/latte macchiato ili cappuccino u kombinaciji s Aerochinom? Unaprijed hvala! Nije uvjet, ali ja preporučam neke jače i kremastije kave/kapsule. Broj 1 izbor mi je svakako Napoli, a onda","Pozdrav svima! Koje su vam se pokazale najbolje Nespresso kapsule za cafe latte/latte macchiato ili cappuccino u kombinaciji s Aerochinom? Unaprijed hvala!","Inspirirajte se idejnim rješenjem by studio Mirjana Mikulec, a u idućoj epizodi InDizajna pogledajte finalnu izvedbu 👍 Vidimo se u nedjelju ujutro 🙌📺⏰ My Istria De’Longhi Hrvatska Moj dom po mjeri BAUHAUS Hrvatska Lesnina XXXL Samsung","Inspirirajte se idejnim rješenjem by studio Mirjana Mikulec, a u idućoj epizodi InDizajna pogledajte finalnu izvedbu 👍 Vidimo se u nedjelju ujutro 🙌📺⏰ My Istria De’Longhi Hrvatska Moj dom po mjeri BAUHAUS Hrvatska Lesnina XXXL Samsung","Kolko pratim njihove cijene 15% na perilicu i sušilicu je ok popust, usisavači se mogu naći jeftinije, aparat za kavu nemam pojma (nespresso aparati rade sasvim pristojnu kavu za djelić cijene) i 15% na sredstva za njegu rublja je ok ako inače","U novoj epizodi Tražimo dom s Mirjanom Mikulec očekuje nas Tin, mladi sportaš u potrazi za stanom 🏘 Budite s nama u subotu ujutro na RTL-u 📺 De’Longhi Hrvatska Samsung JYSK Hrvatska BAUHAUS Hrvatska Citroën","I love my Nespresso.","Quote: BigDaddy kaže: Ova 3583,58 kn je odlična cijena, ako nekome nije problem naručiti izvana: Philips Series 4300 LatteGo EP4341/50 aparat za kavu sa LatteGo pjenilicom za mlijeko Ne sjećam se da sam vidio taj aparat tako jeftino. Vrhunska cijena,","Quote: BigDaddy kaže: Ova 3583,58 kn je odlična cijena, ako nekome nije problem naručiti izvana: Philips Series 4300 LatteGo EP4341/50 aparat za kavu sa LatteGo pjenilicom za mlijeko Ne sjećam se da sam vidio taj aparat tako jeftino. dobra cijena","Ako niste uspjeli u prošlu subotu biti pred malim ekranima, sada možete pogledati propuštenu epizodu Tražimo dom s Mirjanom Mikulec 👇 Citroën BAUHAUS Hrvatska Samsung Qualis salon namještaja - KLER Hrvatska family.hr De’Longhi Hrvatska Partas","Ova 3583,58 kn je odlična cijena, ako nekome nije problem naručiti izvana: Philips Series 4300 LatteGo EP4341/50 aparat za kavu sa LatteGo pjenilicom za mlijeko Ne sjećam se da sam vidio taj aparat tako jeftino.","ovaj Krups, zanimljivo nudi na \"akciji\" i Elipso ali za 700 kuna više ; https://www.elipso.hr/mali-kucanski/.../KRUPS-EA891C/ Ne znam čekati li crni petak, ili kupiti ovaj Krups na mall.hr, iskreno više sam naginjao Philipsu, nešto iz serije","Quote: mali grizzly kaže: Krups se hvali da jedini ima metalnu jedinicu (brew ... Krupsov automat, ali tvrdnja je eksplicitna https://www.majdic.at/krups ... i Elipso ali za 700 kuna više ; https://www.elipso.hr/mali-kucanski/.../KRUPS","Krups se hvali da jedini ima metalnu jedinicu (brew unit) - jednaku u profi liniji aparata, kao i u kućnoj liniji. Ne znam ... odnosno nisam našao prezentacije gdje se rasklapa Krupsov automat, ali tvrdnja je eksplicitna https://www.majdic.at/krups","Pogledajte jučerašnju emisiju InDizajna i saznajte sve iz prve ruke 👇 My Istria De’Longhi Hrvatska Moj dom po mjeri BAUHAUS Hrvatska Lesnina XXXL Samsung TECE Adria Kalcer Perfecta Dreams Hespo Meblo Trade","Pogledajte jučerašnju emisiju InDizajna i saznajte sve iz prve ruke 👇 My Istria De’Longhi Hrvatska Moj dom po mjeri BAUHAUS Hrvatska Lesnina XXXL Samsung TECE Adria Kalcer Perfecta Dreams Hespo Meblo Trade","Ostali su ti još Siemens, Bosch, Krups, Melitta i slični, za koje je podrška i dostupnost nešto manja. Ovdje na forumu ti \"svatko svoga konja hvali\", što je logično, i teško je za očekivati da ćeš naći nekoga tko baš ima iskustva s više brendova, pa","Pokazat ćemo vam i na što pripaziti pri preuređenju malog toaleta te kako sami možete izraditi stalak za voće i povrće 😍 Gledamo se danas prijepodne na RTL-u 📺⏰ My Istria De’Longhi Hrvatska Moj dom po mjeri BAUHAUS Hrvatska Lesnina XXXL","Aşkk ile..🤍🤍 🤍 🤍 🤍 #aşk#jetaime #monamour #coffee #cafe#hello #kahve#latte#flowers #mylove#lattemacchiato#photography #mylove#details @nespresso @karaca","preuređene nekretnine koje su im ponudili agenti, pogledajte u današnjoj epizodi Tražimo dom s Mirjanom Mikulec 🏠 Budite s nama na RTL-u u 11.20 👍 Citroën BAUHAUS Hrvatska Samsung Qualis salon namještaja - KLER Hrvatska family.hr De’Longhi","●PROIZVODI IZ VIDEA*: Krups mikser: https://amzn.to/30gfj0c Set staklenih zdijela: https://amzn.to/3BEyWMx _________________________________________________ ●ZAPRATITE ME I OVDJE: *Instagram: http://bit.ly/2vNzFOx *Facebook:","Čajna kuhinja opremljena je električnim kuhalom za vodu, mikrovalnom pećnicom, aparatom za pripremu kave Nespresso i hladnjakom. Foto: Duško Vlaović Neposredna okolica ispunjena je prekrasnom povijesnom arhitekturom, muzejima, barovima i izvrsnim","Mali, kompaktani, lagani i divnog dizajna - Nespresso aparati! Od sada uz 20% popusta na izdvojene modele i vrijedan poklon. 😀 Izaberi svoj model ➡ https://cutt.ly/JTfz007","📖 Naša je svaka koju čitamo uz ukusnu šalicu kave iz LatteGo 5400 aparata. Zahvaljujući našem jednostavnom zaslonu, nećeš ... #philips #philipshrvatska #philipshomeliving #coffee #philipscoffee #lattego","Tamne boje vratile su se u kuhinje, a najtraženije su crna i antracit siva 🖤 Upravo jednu takvu kuhinju moći ćete vidjeti u novoj epizodi InDizajna u nedjelju ujutro, a ovo je samo mali sneak peak u nju 🙂 My Istria De’Longhi Hrvatska Moj dom po","Sarah, dobro je da nije ništa samo trebaš izdržati Mi bismo uzeli Nespresso samo još uvijek kalkuliram koliko nam je i da li nam je isplativ. Ok, muž popije par nesica dnevno, 2-3 a ja popijem samo ujutro na poslu jednu 0.3 tursku, čistu i to je to.","Sarah, dobro je da nije ništa samo trebaš izdržati Mi bismo uzeli Nespresso samo još uvijek kalkuliram koliko nam je i da li nam je isplativ. Ok, muž popije par nesica dnevno, 2-3 a ja popijem samo ujutro na poslu jednu 0.3 tursku, čistu i to je to.","Apart za kavu imam Nespresso ali nosim kapsule na reciklazu u Nespresso. Sto oni poslije rade s njima, ne znam. Od svih aparata u stanu najvise volim masinu za sudje. IPL mi je odlican ali ga mrzim koristiti. Ali volim rezultate pa zivim s njim u","U idućoj epizodi emisije Tražimo dom s Mirjanom Mikulec agenti su u potrazi za domom za tročlanu mladu obitelj :) 🏡 Budite s nama u subotu ujutro 👍 Citroën BAUHAUS Hrvatska Samsung Qualis salon namještaja - KLER Hrvatska family.hr De’Longhi","U idućoj epizodi emisije Tražimo dom s Mirjanom Mikulec agenti su u potrazi za domom za tročlanu mladu obitelj :) 🏡 Budite s nama u subotu ujutro 👍 Citroën BAUHAUS Hrvatska Samsung Qualis salon namještaja - KLER Hrvatska family.hr De’Longhi","Quote: teodora15 kaže: Mi na vikendici imamo julius na kapsule, nedavno sam kupila u mulleru za 10 kn neku njihovu marku kapsula i ok su mi, inače kombiniram julius ili lavazza mogu i nespresso ići. Tu imam De longhi automatski s mlincem, i taj je","Mi na vikendici imamo julius na kapsule, nedavno sam kupila u mulleru za 10 kn neku njihovu marku kapsula i ok su mi, inače kombiniram julius ili lavazza mogu i nespresso ići. Tu imam De longhi automatski s mlincem, i taj je stvarno isplativ zavisi","Black, just black… #nespresso #podlabs #blackcoffee #kopi-o","Uzmi trenutak za sebe i svoj omiljeni napitak. 🥰 Nespresso aparati za kavu stižu uz 20% popusta + poklon bon za kupnju kapsula! ☕","Nespresso promocija - Bijela tehnika i mali kućanski aparati - Promocije - POSEBNA PONUDA Internet trgovina Harvey Norman Bijela tehnika i mali kućanski aparati Real email address is required to social networks Please enter your email address below","Nespresso akcija listopad 2021 U ponudi imamo 19 proizvoda na Nespresso akcija listopad 2021 Nespresso akcija listopad 2021 Aparat za kavu, Essenza Mini Crna Uključuje komplet od 14 kapsula Usredotočujući svoje znanje o kavi i stručnost na potpuno","aparat za kavu, Doze za mašinu za kavu (Nespresso)) s 1 kaučem za 2 osobe (180 cm, dužina 200 cm), kutom za blagovanje i TV-om sa sat. prijemnikom (plosnatim TV ekranom), el. grijanjem. Izlaz na terasu. Gornji kat: (strme stepenice) 1 otvorena velika","Ako ste propustili Aninu potragu za novim domom, sada to možete nadoknaditi 👇 BAUHAUS Hrvatska Svijet stolica Citroën Qualis salon namještaja - KLER Hrvatska Samsung De’Longhi Hrvatska family.hr OTP banka d.d.","LatteGo napicima s mlijekom dodaje svilenkasto glatku pjenu, lako se postavlja i može se očistiti za samo 15 sekundi. Aparat za kavu PHILIPS EP3243/50 dostupan u našoj trgovini i online na www.malisicshop.ba ☑ #philips #aparatzakavu #cofeemachine","Quote: A i debilli nespresso aparati, to nije potreba nego je luksuz... Jako puno Hrvata u Zagrebu vozi 3-5 km do posla. I tako nazad doma. Tramvaj mreza moze imati poboljsanja ali nije toliko losa. A i busevi su OK, osim neke bas periferne linije","A i debilli nespresso aparati, to nije potreba nego je luksuz... ok... sve 5, ali velim, Vaillant bojler za centralno grijanje i sanitarnu vodu košta 5 000 kn a mjesečna rata plina koju potroši košta 6200 kuna... (karikiram, ali kužiš...).... skuplja","Kada imaš Nespresso Lattissima One-W #aparatzakavu, nije teško preživjeti #ponedjeljak. 😅🙂 Pronađi sve potrebne ... 👀🛒 #capsulalaboutiquedelcaffe #happymonday #dobrojutro #dobrojutrohrvatska #nespresso #nespressolattissima #kava","Ne brinite, sada ju možete pogledati kad god želite 👇 My Istria De’Longhi Hrvatska Modul Samsung BAUHAUS Hrvatska Lesnina XXXL TEXO Hrvatska Meblo Trade KARE Hrvatska Moj dom po mjeri Kamini Kodrić","Ne brinite, sada ju možete pogledati kad god želite 👇 My Istria De’Longhi Hrvatska Modul Samsung BAUHAUS Hrvatska Lesnina XXXL TEXO Hrvatska Meblo Trade KARE Hrvatska Moj dom po mjeri Kamini Kodrić","Jel ima tko iskustva, te kakvi su u usporedbi tipa Philipsovima ili Krups? Ono neki najbolji omjeri cijene/kvalitete.","Situacija je slična i u Nestleu, čiji su najpoznatiji brendovi Nescafe, Maggi i Nespresso. Nestle je tijekom trećeg kvartala cijene svojih proizvoda podigao za 2,1 posto pozivajući se na rast troškova energije i transporta te sirovina. I iz njihovih","Projet Nespresso #aftereffect #Illustrator #logo #Nespresso","My Istria De’Longhi Hrvatska Modul Samsung BAUHAUS Hrvatska Lesnina XXXL TEXO Hrvatska Meblo Trade KARE Hrvatska Moj dom po mjeri Kamini Kodrić","📺⏰ BAUHAUS Hrvatska Svijet stolica Citroën Qualis salon namještaja - KLER Hrvatska Samsung De’Longhi Hrvatska family.hr OTP banka d.d. Foto: Darko Mihalić","Uređenje stoga planiramo do posljednjeg detalja 💛🖤 Kako izgleda ostatak ovog boravka, kao i blagovaonica, pogledajte u novoj epizodi InDizajna u nedjelju prijepodne na RTL-u :) My Istria De’Longhi Hrvatska Modul Samsung BAUHAUS Hrvatska","Tražimo dom s Mirjanom Mikulec s vama je u subotu prijepodne na RTL-u :) Pogledajte je li Ana uspjela pronaći novi dom za svoju obitelj 🏠 BAUHAUS Hrvatska Svijet stolica Citroën Qualis salon namještaja - KLER Hrvatska Samsung De’Longhi","Tražimo dom s Mirjanom Mikulec s vama je u subotu prijepodne na RTL-u :) Pogledajte je li Ana uspjela pronaći novi dom za svoju obitelj 🏠 BAUHAUS Hrvatska Svijet stolica Citroën Qualis salon namještaja - KLER Hrvatska Samsung De’Longhi","LatteGo je ok, ali GranAroma modeli mi se više sviđaju i izgledom, i mogućnošću izrade 2 mliječna napitka istovremeno.","https://edigital.hr/elektricni-apara...lijeka-p878131 Rekao bih da je to \"pljunuti\" Philips EP5441/50 ali bez LatteGo pjenjača za mlijeko, nego s običnim. Taj model nisam vidio na HR tržištu pa budi oprezan jer ćeš ga u slučaju kvara pokrivenog","Krups aparat za kavu EA 815E70 Espresseria Auto Pisa S line Najmanji aparat za kavu s LCD zaslonom za pripremu espressa ... Espresso punog okusa iz kompletno automatskog KRUPS aparata za kavu. Pritisak od 15 bara i hidrauličko istezanje","LatteGo 5400 - sinonim za dobar život. Lakše nego ikad prije, napravi 12 ukusnih napitaka od svježih zrna kafe. Aromatični ... #philips #philipsbosnaihercegovina #philipshomeliving #coffee #lattego #philipscoffee","Suave, beautiful, rocking salt-and-pepper beards, sipping Nespresso because they appear, glistening damp, from an icy plunge in pond Como (okay, okay, that is merely myself getting carried away), the 2 company directed us to start thinking about we","LatteGo 5400 - sinonim za dobar život. Lakše nego ikad prije, napravi 12 ukusnih napitaka od svježih zrna kave, aromatični ... #philips #philipshrvatska #philipshomeliving #coffee #philipscoffee #lattego","Od svih kava koje možete pripremiti u toplini svog doma, Nespresso nam je najdraži. Nema tu previše filozofije. Jednostavno imaju premium proizvod i zato ih i nazivaju Chanelom kave.","Kako je Nespresso postao najpopularnija kava na svijetu?","Moćan mališa. Detaljan test pogledajte na našem YouTube kanalu: https://www.youtube.com/watch?v=BaBXnKAwMZ0 #eponuda #kafa #dolcegusto #krups #nescafe #aparatzakafu #test #opis #review #recenzija","Pridruži se Brad Pittu u ispijanju #Perfetto šalice kave koju za tebe priprema De'Longhi! 😍","Pozdrav, imam nespresso aparat De Longhi EN265, kupio ga preko neta prije 10-ak dana. Sve ok sa kavom, ali mi onaj dodatak za mlijeko, Aeroccino, uopće ne funkcionira. On ima samo jedno okruglo dugme na sebi, ali ništa se ne događa kada ga pritisnem.","aparat za kavu, Doze za mašinu za kavu (Nespresso) uz doplatu). Tuš, odv. WC, duplim umivaonikom. El. grijanje. Vrlo lijep pogled na more. Na raspolaganju: perilica za rublje, sušilica, glačalo, sušilo za kosu. Internet (wireless LAN [WLAN],","Nestle koji prodaje Nescafe, Purinu, KitKat, Maggi i Nespresso je tijekom trećeg kvartala podigao cijene za 2,1 posto, a predviđaju povećanje inflacijskih pritisaka. Poskupljenja je najavio i Procter &amp; Gamble iz kojeg ističu kako su zastoji u lancima","Imam i Nespresso aparat, al vise skuplja prasinu nego je u upotrebi. Jedno vrijeme koristila sam french press, sad pauziram. Ponekad popodne popijem jos jednu, ako sam negdje u drustvu. Obicno imam problem kad sam negdje u kaficu sta naruciti, uvijek","Kompanija Nestle, koja prodaje Nescafe, Purinu, KitKat, Maggi i Nespresso, je tijekom trećeg kvartala cijene podigla za 2,1 posto. Objasnili su kako su zabilježili značajan rast troškova energije, transporta i sirovina. I oni su pokušavali odgoditi","imam i Nespresso aparat, za one kave koje pijemo popodne ili kad mi se hoće aromatizirane kave... imam i Dolce Gusto, ali to mi stoji i skuplja prašinu... imam i mlinac te french press... to kad hoću neku hipstersku kafu... ili kad mi dođe mama jer","imam i Nespresso aparat, za one kave koje pijemo popodne ili kad mi se hoće aromatizirane kave... imam i Dolce Gusto, ali to mi stoji i skuplja prašinu... imam i mlinac te french press... to kad hoću neku hipstersku kafu... ili kad mi dođe mama jer","Kompanija Nestle, koja prodaje Nescafe, Purinu, KitKat, Maggi i Nespresso, je tijekom trećeg kvartala cijene podigla za 2,1 posto. Objasnili su kako su zabilježili značajan rast troškova energije, transporta i sirovina. I oni su pokušavali odgoditi","Kompanija Nestle, koja prodaje Nescafe, Purinu, KitKat, Maggi i Nespresso, je tijekom trećeg kvartala cijene podigla za 2,1 posto. Objasnili su kako su zabilježili značajan rast troškova energije, transporta i sirovina. I oni su pokušavali odgoditi","Akcija krups piccolo 3color 10mj 2021 U ponudi imamo 3 proizvoda na Akcija krups piccolo 3color 10mj 2021 Navigacije Akcija krups piccolo 3color 10mj 2021 Otpratite ljeto i spremni dočekajte jesen uz Krups","aparat za kavu, Doze za mašinu za kavu (Nespresso)) s stolom za blagovanje. Odv. WC. Gornji kat: 1 bračna soba s umivaonikom. Izlaz na balkon. 1 soba s 1 x 2 kreveta na kat, 1 bračnim krevetom. Izlaz na balkon. Kada/tuš/WC. Parketni pod. Vrlo lijep","aparat za kavu, Doze za mašinu za kavu (Nespresso) uz doplatu) s stolom za blagovanje. Gornji kat: 2 sobe, svaka soba s 2 kreveta (90 cm, dužina 200 cm), umivaonikom. Mali dnevni boravak s umivaonikom, kabelskom TV i međunarodnim TV kanalima","KARE Hrvatska Svijet stolica Momax Hrvatska JYSK Hrvatska family.hr De’Longhi Hrvatska Qualis salon namještaja - KLER Hrvatska Partas Azzardo","KARE Hrvatska Svijet stolica Momax Hrvatska JYSK Hrvatska family.hr De’Longhi Hrvatska Qualis salon namještaja - KLER Hrvatska Partas Azzardo","KARE Hrvatska Svijet stolica Momax Hrvatska JYSK Hrvatska family.hr De’Longhi Hrvatska Qualis salon namještaja - KLER Hrvatska Partas Azzardo","Quote: oceaneyes kaže: @Ivancica mala povremeno popijem latte macchiato preko onog Krups aparata. Vise je ko vodica i djeluje mi povoljno na probavu Pravu kavu, u tom smislu kak ju spominjete, pijem gotovo nikad. Cure, moram se pozalit malo na svoje","Pet razloga zbog kojih volimo Nespresso Jutra postaju sve mračnija i hladnija, što ritual ispijanja prve kave u danu čini sve važnijim. Uživali u njoj kod kuće u toplom krevetu ili na poslu s kolegama, važno je samo jedno – da vas kava puninom svojeg","Zadovoljna.hr: Pet razloga zbog kojih volimo Nespresso Jutra postaju sve mračnija i hladnija, što ritual ispijanja prve kave u danu čini sve važnijim. Uživali u njoj kod Zadovoljna.hr: Pet razloga zbog kojih volimo Nespresso Zadovoljna.hr: Istina ili","@Ivancica mala povremeno popijem latte macchiato preko onog Krups aparata. Vise je ko vodica i djeluje mi povoljno na probavu Pravu kavu, u tom smislu kak ju spominjete, pijem gotovo nikad. Cure, moram se pozalit malo na svoje brige. Imam dosta","Krups aparat za kavu Evidence EA891C10 Automatski aparat za kavu ima inovativni sustav Quattro Force koji uvijek optimizira pripremljeni espresso. Krups nadopunjuje 2.1-litarski spremnik za vodu i OLED zaslon osjetljiv na dodir za ugodan rad. Krups","Last #PumpkinSpiceCake #Nespresso #Vertuo Pod 👀. #Coffee #PortraitMode","KARE Hrvatska Svijet stolica Momax Hrvatska JYSK Hrvatska family.hr De’Longhi Hrvatska Qualis salon namještaja - KLER Hrvatska Partas Azzardo","Drage pomozite, odgovaraju li koje jeftinike kapsule u ovaj aparat dolce gusto krups","KARE Hrvatska Svijet stolica Momax Hrvatska JYSK Hrvatska family.hr De’Longhi Hrvatska Qualis salon namještaja - KLER Hrvatska Partas Azzardo","Love love love my new toy! #nespresso #coffee #nespressocreatistapro","aparat za kavu, Doze za mašinu za kavu (Nespresso) uz doplatu). Izlaz na balkon. Tuš/WC. Grijanje. Veliki balkon, prema jugoistoku, mali balkon, prema sjeveru. Balkonski namještaj. Lijep pogled na jezero, planine i mjesto. Na raspolaganju: sušilo za","- Buzin, Većeslava Holjevca 62 Šibenik Supernova, Put Vida 6 Samobor ulica Kralja Petra Krešimira IV, 4a Koprivnica Supernova - Radnička cesta 8 Pula Max City - Stoja 14A Zoom Kompaktan superautomatski aparat za kavu s patentiranim De’Longhi","Nespresso kompatibilne. Itenzitet 4/10. Originalna mješavina donosi vam nadahnute priče s južne polutke o podrijetlu ... Sastavljena od zrna arabice južne i središnje Amerike i Stavi u košaricu Lavazza nespresso kapsule 10/1 Ristretto","aparat za kavu, Doze za mašinu za kavu (Nespresso) uz doplatu, set za fondue (sir)). Tuš/bidet/WC. Mali balkon. Balkonski namještaj. Vrlo lijep pogled na planine i krajolik. Na raspolaganju: sušilo za kosu. Internet (wireless LAN [WLAN], besplatan).","KARE Hrvatska Svijet stolica Momax Hrvatska JYSK Hrvatska family.hr De’Longhi Hrvatska Qualis salon namještaja - KLER Hrvatska Partas Azzardo","KARE Hrvatska Svijet stolica Momax Hrvatska JYSK Hrvatska family.hr De’Longhi Hrvatska Qualis salon namještaja - KLER Hrvatska Partas Azzardo","KARE Hrvatska Svijet stolica Momax Hrvatska JYSK Hrvatska family.hr De’Longhi Hrvatska Qualis salon namještaja - KLER Hrvatska Partas Azzardo","KARE Hrvatska Svijet stolica Momax Hrvatska JYSK Hrvatska family.hr De’Longhi Hrvatska Qualis salon namještaja - KLER Hrvatska Partas Azzardo","KARE Hrvatska Svijet stolica Momax Hrvatska JYSK Hrvatska family.hr De’Longhi Hrvatska Qualis salon namještaja - KLER Hrvatska Partas Azzardo","S ovom kavom #Nespresso nas mirisom i okusom pokušava vratiti u Italiju 1940-ih, u atmosferu stand-up kafića tipičnih za taj period ! I mogu vam reći da su u tome i uspijeli! 🙌🏻 Posjetite Nespresso trgovine i uživajte u posebnoj ponudi ovog","Uz De’Longhi superatomatske aparate za kavu ne treba vam puno truda jer aparati sve sami naprave. Nova Magnifica Evo dizajnirana je da zadovolji različite ukuse potrošača s mnogo različitih recepata dostupnih na jedan dodir. Utjelovljuje De`Longhi ","😁 Odaberite slasne domaće pite koje se savršeno slažu uz espresso ili lungo kavu koju možete pripremiti u Nespresso aparatima. Za doručak tu je novi kremasti sirni namaz, a za brzi ručak ukusni umaci u 3 okusa. Potražite ove i još puno, puno drugih","good morning pancake 🥞🍌 #homecooking #homesweethome #nespresso #cafe15☕"," Tu je još i „L' espresso perfetto“ linija s posebno biranim sortama kave u kapsulama kompatibilnim s Nespresso aparatima. A za one avanturističkog duha namijenjena je linija kave  „I Viaggi“  – s mljevenom kavom za sve namjene i biorazgradivim","Predstavljamo i posebnu liniju za domaćinstvo- „ Blu “- mljevena kava, kava u zrnu i kava za moku te „ Bianco “ – bez kofeina.  Tu je još i „ L' espresso perfetto “ linija s posebno biranim sortama kave u kapsulama kompatibilnim s Nespresso","Čini mi se da se te Nespresso verzije kapsula daju naći povoljnije od Dolce gusto.","aparat za kavu, Doze za mašinu za kavu (Nespresso) uz doplatu). Odv. WC. Gornji kat: 1 soba s kosim stropom s 1 franc. krevetom (140 cm, dužina 190 cm). 1 soba s kosim stropom s 1 kaučem za 1 osobu (90 cm, dužina 190 cm), 1 franc. krevetom (140 cm,","° ° ° ° ° ° ° ° ° ° ° ° ° ° ° ° ° Lájk Kokoon Beauty Kaltenberg Sörház &amp; Étterem YURKOV echosline.hu Deliciosa Bálna Terasz Carplove Horgász és Pihenőpark Várkert Bazár Csetreszke.hu Nespresso The Sweet by Vintage Garden StílusZsaru"," Tu je još i „L' espresso perfetto“ linija s posebno biranim sortama kave u kapsulama kompatibilnim s Nespresso aparatima.  A za one avanturističkog duha namijenjena je linija kave „I Viaggi“ – s mljevenom kavom za sve namjene i biorazgradivim","Pozdrav. Čime čistite Nespresso aparate ( imam Citiz )? Jel može ono sredstvo iz DM-a za par kuna?","Tiramisu pohárkrém ☕️✨ #tiramisu #cake #cakedecorating #cakedesign #cakeart #cakephotography #coffee #nespresso #followforfollowback #likeforlikes","Uz De’Longhi superatomatske aparate za kavu ne treba vam puno truda jer aparati sve sami naprave. Nova Magnifica Evo dizajnirana je da zadovolji različite ukuse potrošača s mnogo različitih recepata dostupnih na jedan dodir. Utjelovljuje De`Longhi ","Novi Nespresso okusi odvest će vas u prve talijanske kafiće ili pak u moderno uređene barove Ciao, bella! Jeste li spremni na putovanje kroz vrijeme sa šalicom kave u ruci? Novi Nespresso okusi odvest će vas u prve talijanske kafiće ili pak u moderno","Uz De’Longhi superatomatske aparate za kavu ne treba vam puno truda jer aparati sve sami naprave. Nova Magnifica Evo dizajnirana je da zadovolji različite ukuse potrošača s mnogo različitih recepata dostupnih na jedan dodir. Utjelovljuje De`Longhi ","Sta si u kavu stavila\"ljubav\" #mojakava# #nespressokava# #nespresso# #nespresso# #nespresso#","Puslice: https://bit.ly/3ogD0iW Čokoladni kuglof od bjelanjaka: https://bit.ly/3m2hYBS Kokos puslice: https://bit.ly/3oeDDcY ______________________________________________ ●PROIZVODI IZ VIDEA*: Obruč za torte: https://amzn.to/2WsFXBK Krups","Uz De’Longhi superatomatske aparate za kavu ne treba vam puno truda jer aparati sve sami naprave. Nova Magnifica Evo dizajnirana je da zadovolji različite ukuse potrošača s mnogo različitih recepata dostupnih na jedan dodir. Utjelovljuje De`Longhi ","Uz De’Longhi superatomatske aparate za kavu ne treba vam puno truda jer aparati sve sami naprave. Nova Magnifica Evo dizajnirana je da zadovolji različite ukuse potrošača s mnogo različitih recepata dostupnih na jedan dodir. Utjelovljuje De`Longhi","Novi Nespresso okusi odvest će vas u prve talijanske kafiće ili pak u moderno uređene barove! 🥰 Nespresso","ona je kafa iz našeg LatteGo 5400 aparata! ☕ U nekoliko koraka možeš da prilagodiš okus od friških zrna i pretočiš želju u ... #philips #philipsbosnaihercegovina #philipshomeliving #coffee #lattego #philipscoffee","Novi Nespresso okusi odvest će vas u prve talijanske kafiće ili pak u moderno uređene barove - PROGRESSIVE MAGAZIN Naslovnica NOVITETI Novi Nespresso okusi odvest će vas u prve talijanske kafiće ili pak... Novi Nespresso okusi odvest će vas u prve","ona je kava iz našega aparata LatteGo 5400! ☕ V nekaj korakih lahko prilagodite okus svežih zrn in pretočite željo v ... #philips #philipsslovenija #philipscoffee #coffee #coffeetime #lattego #coffeeperson #coffee #philipshomeliving","Novi Nespresso okusi odvest će vas u prve talijanske kafiće ili pak u moderno uređene barove Kada je u pitanju hrana, ponekad poželimo eksperimentirati s novim začinima i receptima, dok već sutradan žudimo za ponovnim otkrivanjem autentičnih okusa","ona je kava iz našeg LatteGo 5400 aparata! ☕ U nekoliko koraka možeš prilagoditi okus od svježih zrna i pretočiti želju u ... #philips #philipshrvatska #philipshomeliving #coffee #philipscoffee #lattego","Povodom Međunarodnog dana kave jednog čitateljarazveselit ćemo Julius Meinl Starter paketom koji sadrži Inspresso aparat za Nespresso kompatibilne kapsule, 5 vrsta biorazgradivih kapsula i dvije crvene espresso šalice. Uživajte u omiljenoj kavi! Ime","Uz De'Longhi superatomatske aparate za kavu ne treba vam puno truda jer aparati sve sami naprave. Nova Magnifica Evo dizajnirana je da zadovolji različite ukuse potrošača s mnogo različitih recepata dostupnih na jedan dodir. Utjelovljuje De'Longhi","je ritual koji nam omogućuje da zastanemo i stvorimo trenutak u kojem možemo istinski uživati”, rekao je Nik.  De'Longhi ... Uz De’Longhi superatomatske aparate za kavu ne treba vam puno truda jer aparati sve sami naprave. Nova Magnifica Evo","Burj Khalifa, najviša zgrada s visinom od 828 metara, odjevena je u zrna kave i logotip De’Longhi u čast Svjetskog dana kave i prvog aparata za kavu na svijetu. Nik Oroši, jedan od najuspješnijih hrvatskih barista, tvrdi kako se recept za savršenu","Nespresso inissia i pixie akcija U ponudi imamo 5 proizvoda na Nespresso inissia i pixie akcija Nespresso inissia i pixie akcija Akcija traje od 1.10. do 11.10.2021 ili do isteka zaliha. Aparat za kavu Pixie Titan Uključuje komplet od 14 kapsula","Uz De’Longhi superatomatske aparate za kavu ne treba vam puno truda jer aparati sve sami naprave. Nova Magnifica Evo dizajnirana je da zadovolji različite ukuse potrošača s mnogo različitih recepata dostupnih na jedan dodir. Utjelovljuje De`Longhi","Svjetski je dan kave! 😀 Uživaj u ispijanju kave u toplini svog doma uz vrhunske Philips, Nespresso, Krups, Russell Hobbs, Lavazza, DeLonghi i Gorenje aparate za kavu. Izaberi svoj model već od 399 kn ➡ https://cutt.ly/MEAWTUY","Uz De’Longhi superatomatske aparate za kavu ne treba vam puno truda jer aparati sve sami naprave. Nova Magnifica Evo dizajnirana je da zadovolji različite ukuse potrošača s mnogo različitih recepata dostupnih na jedan dodir. Utjelovljuje De`Longhi","❤ #nespresso #nespressomoments #InspirazioneItaliana *sadržaj je nastao u suradnji s @nespresso","Nespresso je u svoju paletu Ispirazione Italiana dodao dva nova, ograničena okusa – Ispirazione Novecento i Ispirazione Millennio Ja TRGOVAC Welcome! Log into your account your username Ciao, bella! Jeste li spremni na putovanje kroz vrijeme sa","Sretan vam svjetski dan kave uz nove Nespresso okuse Kada je u pitanju hrana, ponekad poželimo eksperimentirati s novim začinima i receptima, dok već sutradan žudimo za ponovnim otkrivanjem autentičnih okusa Sretan vam svjetski dan kave uz nove","Nespresso ima nove okuse i vode vas direktno u Italiju Nespresso ima nove okuse i vode vas direktno u Italiju Nespresso ima nove okuse i vode vas direktno u Italiju Tekst: BURO. 30.9.21. 17:00 Kada je u pitanju hrana, ponekad poželimo","Inspiriran umjetnošću talijanskih majstora prženja kave i okusima koje tako vješto stvaraju, Nespresso je u svoju paletu Ispirazione Italiana dodao dva nova, ograničena okusa – Ispirazione Novecento i Ispirazione Millennio – koji svojim značajkama","Inspiriran umjetnošću talijanskih majstora prženja kave i okusima koje tako vješto stvaraju, Nespresso je u svoju paletu Ispirazione Italiana dodao dva nova, ograničena okusa – Ispirazione Novecento i Ispirazione Millennio – koji svojim značajkama","Novi Nespresso okusi odvest će vas u prve talijanske kafiće ili pak u moderno uređene barove Miris kave bez premca... (Arhiva) Više o povijest Kada je u pitanju hrana, ponekad poželimo eksperimentirati s novim začinima i receptima, dok već sutradan","⏳ZADNJI DAN POPUSTA⏳ Nespresso Lavazza Favourite Mix ✔️100 KAPSULA ✔️Nespresso kompatibilne ✔️Kutija sadrži 5x20 ... #capsulalaboutiquedelcaffe #kava #coffee #lavazzapods #lavazzacaffè #nespresso #lavazza #popust #popusti #sale #akcija","Novi Nespresso okusi odvest će vas u prve talijanske kafiće ili pak u moderno uređene barove Kada je u pitanju hrana, ponekad poželimo eksperimentirati s Ciao, bella! Jeste li spremni na putovanje kroz vrijeme sa šalicom kave u ruci? by Super1 Relax","- Magazin za turizam i gastronomiju Novi Nespresso okusi odvest će vas u prve talijanske kafiće ili pak u moderno uređene barove Kada je u pitanju hrana, ponekad poželimo eksperimentirati s novim začinima i receptima, dok već sutradan žudimo za","Tu je još i „L' espresso perfetto“ linija s posebno biranim sortama kave u kapsulama kompatibilnim s Nespresso aparatima. A za one avanturističkog duha namijenjena je linija kave „I Viaggi “  – s mljevenom kavom za sve namjene i biorazgradivim","Cafe Frei u City Centar one East Zanima me jel prodaju kapsule za nespresso i jel melju kavu za đezvu. Također što bi preporučili za probat da je fora i nešto što se svaki dan ne pije u kafiću?","Zalig ontbijtje zo ! @binahandmadebe #ontbijt #ceramic #keramiek #handmade #giveaway #happy #beautifulcoffee #nespressomoments #nespresso #cruesli #quakercruesli","#NESPRESSO Citiz&amp;milk Krups #VENTEArticleBEBE #DépotBébéKmayra #44385627 INSTAGRAM 👇 https://www.instagram.com/depot_kmayra page FACEBOOK 👇👇 https://www.facebook.com/pg/VENTE-Article-BEBE-244976682883641/posts/","Ono kad ti je milka.hrvatska inspiracija za outfit 😁💜 podsjecam vas da jos uvijek traje nagradna igra u kojoj mozete osvojiti Panasonic televizore, Krups aparate za kavu i Tefal ekspres lonce 😊 nagrade su inspirirane novim limitiranim okusima, a","Pozdrav, Planiram kupovinu espresso aparata i za sada mi je prvi izbor philips lattego 4300. Da li neko zna da li mozda u nekim vecim marketima/prodavnicama moze da se nadje ovaj aparat na popustu? Hvala unapred","🥰 Započni dan na najbolji mogući način - napravi zdravi doručak, pripremi kafu iz Philips 5400 LatteGo aparata i spremno ... #philips #philipsbosnaihercegovina #philipshomeliving #coffee #lattego #philipscoffee","Philips LatteGo z ustrezno hitrostjo ustvari kremasto peno idealne temperature in teksture, oblikovan pa je za delo z ... #philips #philipsslovenija #philipscoffee #coffee #coffeetime #lattego #coffeeperson #coffee #philipshomeliving","Philips LatteGo odgovarajućom brzinom stvara kremastu pjenu idealne temperature i teksture, a dizajniran je za rad sa svim ... #philips #philipshrvatska #philipshomeliving #coffee #philipscoffee #lattego","[PHILIPS RASKOŠNA JESEN] Kava i toplo pecivo su razlog zašto nas vesele buđenja ujutro.☕️🥐 Sretni dobitnik našeg nagradnog natječaja imat će priliku uživati u savršenim jutrima uz Philips Espresso LatteGo aparate za kavu. 🥰 Više pročitajte na linku","LatteGo, s katerim lahko iz mleka naredite gladko in svilnato mlečno peno, enostavno nastavite in očistite v samo 15 sekundah* Zaslon na dotik Vsak trenutek si lahko privoščite katero koli od 5 kav, vključno s kapučinom Popestrite svoje trenutke s","🥰 Započni dan na najbolji mogući način - napravi zdrav doručak, pripremi kafu iz Philips 5400 LatteGo aparata i spremno ... #philips #philipssrbija #philipshomeliving #coffee #lattego #philipscoffee","🥰 Započni dan na najbolji mogući način - napravi zdravi doručak, pripremi kavu u LatteGo aparatu i spremno dočekaj sve ... #philips #philipshrvatska #philipshomeliving #coffee #philipscoffee #lattego","Philips LatteGo odgovarajućom brzinom stvara kremastu pjenu idealne temperature i teksture, a dizajniran je za rad sa svim ... #philips #philipsbosnaihercegovina #philipshomeliving #coffee #lattego #philipscoffee","Philips LatteGo Ste tudi vi zelo zavezani temu, kakšna kava vam najbolj odgovarja? Je to espresso, kapučino, latte macchiato, kava z mlekom, ameriška kava ali navadna kava?     Vsak ljubitelj kave je zelo zavezan popolni kombinaciji... Ni lepšega kot","🥰 Začni dan na najboljši možen način - naredi si zdrav zajtrk, skuhaj kavo s Philipsovim aparatom 5400 LatteGo in ... #philips #philipsslovenija #philipscoffee #coffee #coffeetime #lattego #coffeeperson #coffee #philipshomeliving"],["2022-01-10","2022-01-08","2022-01-07","2022-01-06","2022-01-05","2022-01-05","2022-01-04","2022-01-04","2022-01-04","2022-01-04","2021-12-28","2021-12-27","2021-12-24","2021-12-23","2021-12-23","2021-12-23","2021-12-23","2021-12-23","2021-12-22","2021-12-22","2021-12-22","2021-12-21","2021-12-21","2021-12-20","2021-12-20","2021-12-20","2021-12-20","2021-12-20","2021-12-20","2021-12-18","2021-12-17","2021-12-16","2021-12-16","2021-12-15","2021-12-15","2021-12-15","2021-12-15","2021-12-15","2021-12-15","2021-12-15","2021-12-15","2021-12-15","2021-12-14","2021-12-14","2021-12-13","2021-12-13","2021-12-12","2021-12-11","2021-12-11","2021-12-10","2021-12-10","2021-12-10","2021-12-10","2021-12-09","2021-12-09","2021-12-09","2021-12-09","2021-12-09","2021-12-09","2021-12-08","2021-12-07","2021-12-07","2021-12-07","2021-12-07","2021-12-06","2021-12-06","2021-12-06","2021-12-06","2021-12-04","2021-12-04","2021-12-04","2021-12-03","2021-12-03","2021-12-02","2021-12-02","2021-12-02","2021-12-02","2021-12-02","2021-12-02","2021-12-02","2021-12-02","2021-12-02","2021-12-02","2021-12-02","2021-12-02","2021-12-02","2021-12-02","2021-11-30","2021-11-30","2021-11-30","2021-11-30","2021-11-30","2021-11-29","2021-11-29","2021-11-29","2021-11-28","2021-11-28","2021-11-28","2021-11-27","2021-11-27","2021-11-26","2021-11-26","2021-11-25","2021-11-25","2021-11-25","2021-11-25","2021-11-25","2021-11-25","2021-11-24","2021-11-24","2021-11-24","2021-11-23","2021-11-23","2021-11-23","2021-11-23","2021-11-23","2021-11-23","2021-11-23","2021-11-22","2021-11-22","2021-11-22","2021-11-22","2021-11-22","2021-11-21","2021-11-21","2021-11-21","2021-11-20","2021-11-20","2021-11-20","2021-11-20","2021-11-20","2021-11-20","2021-11-19","2021-11-19","2021-11-19","2021-11-19","2021-11-19","2021-11-19","2021-11-18","2021-11-18","2021-11-16","2021-11-16","2021-11-16","2021-11-16","2021-11-16","2021-11-15","2021-11-15","2021-11-15","2021-11-15","2021-11-15","2021-11-14","2021-11-14","2021-11-13","2021-11-13","2021-11-12","2021-11-12","2021-11-12","2021-11-12","2021-11-12","2021-11-11","2021-11-11","2021-11-11","2021-11-11","2021-11-11","2021-11-11","2021-11-11","2021-11-11","2021-11-10","2021-11-10","2021-11-10","2021-11-09","2021-11-09","2021-11-09","2021-11-08","2021-11-08","2021-11-08","2021-11-08","2021-11-08","2021-11-07","2021-11-07","2021-11-07","2021-11-07","2021-11-06","2021-11-05","2021-11-04","2021-11-04","2021-11-03","2021-11-03","2021-11-01","2021-10-31","2021-10-29","2021-10-29","2021-10-29","2021-10-29","2021-10-27","2021-10-27","2021-10-26","2021-10-25","2021-10-22","2021-10-22","2021-10-22","2021-10-22","2021-10-22","2021-10-22","2021-10-22","2021-10-21","2021-10-20","2021-10-20","2021-10-20","2021-10-20","2021-10-20","2021-10-19","2021-10-19","2021-10-19","2021-10-19","2021-10-18","2021-10-17","2021-10-16","2021-10-15","2021-10-14","2021-10-14","2021-10-13","2021-10-11","2021-10-11","2021-10-08","2021-10-08","2021-10-08","2021-10-08","2021-10-08","2021-10-08","2021-10-07","2021-10-06","2021-10-06","2021-10-06","2021-10-05","2021-10-05","2021-10-04","2021-10-04","2021-10-04","2021-10-04","2021-10-03","2021-10-02","2021-10-02","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-09-30","2021-09-30","2021-09-30","2021-09-30","2021-09-30","2021-09-30","2021-09-30","2021-09-30","2021-09-30","2021-09-29","2021-09-29","2021-09-28","2021-09-27","2021-09-14","2021-09-13","2021-09-13","2021-09-13","2021-09-13","2021-09-10","2021-09-08","2021-09-08","2021-09-06","2021-09-03","2021-09-01"],["instagram","twitter","instagram","twitter","forum","forum","forum","forum","instagram","web","web","instagram","web","web","web","web","facebook","instagram","facebook","web","web","web","web","facebook","web","web","web","web","web","instagram","web","web","web","web","web","web","web","web","twitter","web","web","web","twitter","web","facebook","web","facebook","forum","forum","facebook","instagram","web","facebook","instagram","web","web","facebook","facebook","youtube","instagram","instagram","web","facebook","facebook","instagram","instagram","web","facebook","forum","facebook","web","instagram","instagram","web","youtube","facebook","web","web","web","web","web","web","web","web","web","web","instagram","facebook","facebook","instagram","instagram","web","facebook","web","web","forum","facebook","facebook","facebook","facebook","facebook","web","web","facebook","web","youtube","web","web","forum","instagram","web","web","web","web","facebook","facebook","instagram","instagram","facebook","facebook","facebook","forum","instagram","facebook","facebook","facebook","forum","forum","facebook","facebook","forum","forum","forum","facebook","forum","forum","facebook","facebook","forum","facebook","reddit","forum","forum","facebook","forum","forum","forum","forum","facebook","facebook","forum","facebook","instagram","facebook","youtube","web","facebook","instagram","facebook","forum","forum","forum","facebook","facebook","forum","forum","instagram","facebook","web","web","web","facebook","instagram","forum","forum","instagram","facebook","facebook","forum","web","instagram","facebook","facebook","facebook","facebook","facebook","forum","forum","web","instagram","web","instagram","facebook","web","facebook","facebook","forum","web","web","forum","web","forum","forum","web","web","web","web","web","facebook","facebook","facebook","forum","web","web","forum","web","instagram","facebook","facebook","facebook","instagram","web","web","web","web","facebook","facebook","facebook","facebook","facebook","instagram","web","facebook","instagram","web","web","forum","web","facebook","web","forum","instagram","web","web","web","instagram","youtube","web","web","facebook","instagram","web","instagram","web","instagram","web","web","web","web","web","web","facebook","web","instagram","web","web","web","web","web","web","instagram","web","web","web","reddit","instagram","instagram","instagram","forum","instagram","instagram","instagram","instagram","web","instagram","instagram","instagram","web","instagram"],["anonymous_user","Sarlo","anonymous_user","𝓫𝓸𝓸𝓴𝓪 𝓫𝓸𝓸𝓴𝓪 𝔂𝓪𝓪𝓪",null,null,null,null,"anonymous_user","E PLUS d.o.o ©",null,"petermurarik","@journalhr","Vizua.net",null,"@journalhr","Modni ALMAnah","anonymous_user","Webgradnja Home - Uredi svoj dom",null,null,"https://plus.google.com/101843740320019272553?",null,"Darkova Web Kuharica",null,null,null,null,"https://www.facebook.com/super1.hr","anonymous_user",null,null,null,null,null,null,"https://www.facebook.com/super1.hr",null,"Znatko",null,null,null,"Ruben Luna","@journalhr","In Dizajn s Mirjanom Mikulec","E PLUS d.o.o ©","In Dizajn s Mirjanom Mikulec",null,null,"Roses Fashion Outlet","anonymous_user","@zadovoljnahr","In Dizajn s Mirjanom Mikulec","anonymous_user","@journalhr","@journalhr","Emmezeta","Sancta Domenica","SelectBox Hrvatska","anonymous_user","anonymous_user",null,"In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","anonymous_user","anonymous_user",null,"posao.hr",null,"In Dizajn s Mirjanom Mikulec",null,"anonymous_user","anonymous_user",null,"Obiteljski kanal","In Dizajn s Mirjanom Mikulec",null,null,null,null,null,"Sretna Mama",null,null,null,"ŠibenikIN News Portal","anonymous_user","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","anonymous_user","anonymous_user",null,"In Dizajn s Mirjanom Mikulec","https://plus.google.com/101843740320019272553?",null,null,"Emmezeta","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec",null,"Telegram.hr","In Dizajn s Mirjanom Mikulec",null,"Lisette Brianna",null,"E PLUS d.o.o ©",null,"anonymous_user","E PLUS d.o.o ©","https://plus.google.com/101843740320019272553?",null,null,"In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","anonymous_user","anonymous_user","In Dizajn s Mirjanom Mikulec","Emmezeta","In Dizajn s Mirjanom Mikulec",null,"anonymous_user","Mamine tajne anonimne i javne","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec",null,null,"In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec",null,null,null,"In Dizajn s Mirjanom Mikulec",null,null,"In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec",null,"In Dizajn s Mirjanom Mikulec","vonrupenstein",null,null,"In Dizajn s Mirjanom Mikulec",null,null,null,null,"In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec",null,"In Dizajn s Mirjanom Mikulec","anonymous_user","In Dizajn s Mirjanom Mikulec","Hanuma kocht - Hanumina kuhinja",null,"Sancta Domenica","anonymous_user","In Dizajn s Mirjanom Mikulec",null,null,null,"In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec",null,null,"anonymous_user","eKupi.hr",null,"E PLUS d.o.o ©",null,"In Dizajn s Mirjanom Mikulec","anonymous_user",null,null,"anonymous_user","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec",null,null,"anonymous_user","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec",null,null,null,"anonymous_user","http://www.top-lista.hr/www/author/hdu/","anonymous_user","Buro247.hr",null,"ePonuda","eKupi.hr",null,null,null,null,null,null,null,null,null,"E PLUS d.o.o ©",null,null,"In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec",null,"@zadovoljnahr","https://www.facebook.com/",null,null,"anonymous_user","In Dizajn s Mirjanom Mikulec","Mamine tajne anonimne i javne","In Dizajn s Mirjanom Mikulec","anonymous_user",null,"E PLUS d.o.o ©","E PLUS d.o.o ©",null,"Tražimo dom s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","father_of_djordje",null,"SPAR Hrvatska","anonymous_user",null,null,null,null,"Messaging_Page.",null,null,"anonymous_user",null,null,"Vizua.net","anonymous_user","Hanuma kocht - Hanumina kuhinja",null,null,"Extravagant","anonymous_user",null,"anonymous_user",null,"anonymous_user",null,null,null,null,"E PLUS d.o.o ©","https://www.facebook.com/super1.hr","Sancta Domenica",null,"anonymous_user","@jatrgovac",null,null,null,null,null,"anonymous_user","https://www.facebook.com/super1.hr",null,"https://www.facebook.com/Glasistre.hr","frogitoto","anonymous_user","anonymous_user","skitnica",null,"anonymous_user","anonymous_user","anonymous_user","anonymous_user",null,"anonymous_user","anonymous_user","anonymous_user",null,"anonymous_user"],["anonymous_user","Sarlo","anonymous_user","𝓫𝓸𝓸𝓴𝓪 𝓫𝓸𝓸𝓴𝓪 𝔂𝓪𝓪𝓪","forum.hr","forum.hr","forum.hr","forum.hr","anonymous_user","elipso.hr","interhome.hr","petermurarik","journal.hr","cafe.hr","wish.hr","journal.hr","Modni ALMAnah","anonymous_user","Webgradnja Home - Uredi svoj dom","story.hr","elle.hr","svijet-medija.hr","instore.hr","Darkova Web Kuharica","menu.hr","zmaichek.com.hr","fashion.hr","mall.hr","telegram.hr","anonymous_user","zmaichek.com.hr","Metro-portal.hr","jutarnji.hr","zmaichek.com.hr","znatko.com","buro247.hr","telegram.hr","magme.hr","Znatko","vecernji.hr","menu.hr","menu.hr","Ruben Luna","journal.hr","In Dizajn s Mirjanom Mikulec","elipso.hr","In Dizajn s Mirjanom Mikulec","forum.hr","forum.hr","Roses Fashion Outlet","anonymous_user","dnevnik.hr","In Dizajn s Mirjanom Mikulec","anonymous_user","journal.hr","journal.hr","Emmezeta","Sancta Domenica","SelectBox Hrvatska","anonymous_user","anonymous_user","novilist.hr","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","anonymous_user","anonymous_user","nespresso.hr","posao.hr","forum.hr","In Dizajn s Mirjanom Mikulec","vecernji.hr","anonymous_user","anonymous_user","wish.hr","Obiteljski kanal","In Dizajn s Mirjanom Mikulec","zenskikutak.hr","amazonke.com","elegant.hr","suvremenazena.hr","suvremenazena.hr","sretnamama.hr","lipadona.com","fashion.hr","index.hr","sibenik.in","anonymous_user","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","anonymous_user","anonymous_user","guster.com.hr","In Dizajn s Mirjanom Mikulec","svijet-medija.hr","emmezeta.hr","forum.hr","Emmezeta","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","emmezeta.hr","telegram.hr","In Dizajn s Mirjanom Mikulec","mall.hr","Lisette Brianna","interhome.hr","elipso.hr","forum.hr","anonymous_user","elipso.hr","svijet-medija.hr","tehnomag.com","24sata.hr","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","anonymous_user","anonymous_user","In Dizajn s Mirjanom Mikulec","Emmezeta","In Dizajn s Mirjanom Mikulec","forum.hr","anonymous_user","Mamine tajne anonimne i javne","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","forum.hr","forum.hr","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","forum.hr","forum.hr","forum.hr","In Dizajn s Mirjanom Mikulec","forum.hr","forum.hr","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","forum.hr","In Dizajn s Mirjanom Mikulec","whatisthisthing","forum.hr","forum.hr","In Dizajn s Mirjanom Mikulec","forum.hr","forum.hr","forum.hr","forum.hr","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","forum.hr","In Dizajn s Mirjanom Mikulec","anonymous_user","In Dizajn s Mirjanom Mikulec","Hanuma kocht - Hanumina kuhinja","vecernji.hr","Sancta Domenica","anonymous_user","In Dizajn s Mirjanom Mikulec","forum.hr","forum.hr","forum.hr","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","forum.hr","forum.hr","anonymous_user","eKupi.hr","harveynorman.hr","elipso.hr","interhome.hr","In Dizajn s Mirjanom Mikulec","anonymous_user","forum.hr","forum.hr","anonymous_user","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","forum.hr","dnevno.hr","anonymous_user","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","forum.hr","forum.hr","mall.hr","anonymous_user","top-lista.hr","anonymous_user","Buro247.hr","buro247.hr","ePonuda","eKupi.hr","forum.hr","interhome.hr","dnevno.hr","forum.hr","slobodnadalmacija.hr","forum.hr","forum.hr","jutarnji.hr","jutarnji.hr","elipso.hr","interhome.hr","interhome.hr","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","forum.hr","dnevnik.hr","360hr.news","forum.hr","mall.hr","anonymous_user","In Dizajn s Mirjanom Mikulec","Mamine tajne anonimne i javne","In Dizajn s Mirjanom Mikulec","anonymous_user","interhome.hr","elipso.hr","elipso.hr","interhome.hr","Tražimo dom s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","In Dizajn s Mirjanom Mikulec","father_of_djordje","femina.hr","SPAR Hrvatska","anonymous_user","istarski.hr","istrain.hr","forum.hr","interhome.hr","Messaging_Page.","regionalexpress.hr","forum.hr","anonymous_user","menu.hr","story.hr","cafe.hr","anonymous_user","Hanuma kocht - Hanumina kuhinja","progressive.com.hr","zmaichek.com.hr","Extravagant","anonymous_user","progressive.com.hr","anonymous_user","hedonist-magazin.com","anonymous_user","tportal.hr","metro-portal.hr","24sata.hr","fama.com.hr","elipso.hr","telegram.hr","Sancta Domenica","instore.hr","anonymous_user","jatrgovac.com","extravagant.com.hr","buro247.hr","vecernji.hr","zmaichek.com.hr","metro-portal.hr","anonymous_user","telegram.hr","menu.hr","glasistre.hr","croatia","anonymous_user","anonymous_user","skitnica","ana.rs","anonymous_user","anonymous_user","anonymous_user","anonymous_user","philips.si","anonymous_user","anonymous_user","anonymous_user","grazia.si","anonymous_user"],["Nespresso","Nespresso","LatteGo","Nespresso","Nespresso","Krups","Nespresso","Krups","Nespresso","LatteGo","Nespresso","Nespresso","Nespresso","DeLonghi","DeLonghi","Nespresso","DeLonghi","Nespresso","Krups","DeLonghi","DeLonghi","LatteGo","DeLonghi","Nespresso","DeLonghi","DeLonghi","Nespresso","LatteGo","DeLonghi","Nespresso","Krups","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","DeLonghi","Nespresso","DeLonghi","LatteGo","DeLonghi","DeLonghi","DeLonghi","Krups","Nespresso","Nespresso","DeLonghi","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","DeLonghi","DeLonghi","Nespresso","LatteGo","Nespresso","Krups","Nespresso","DeLonghi","Nespresso","LatteGo","Nespresso","Nespresso","DeLonghi","DeLonghi","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","DeLonghi","DeLonghi","LatteGo","LatteGo","Nespresso","DeLonghi","LatteGo","Krups","Krups","Nespresso","DeLonghi","DeLonghi","DeLonghi","DeLonghi","Nespresso","DeLonghi","DeLonghi","Krups","Nespresso","Nespresso","Krups","Krups","LatteGo","Nespresso","LatteGo","Krups","Nespresso","DeLonghi","DeLonghi","Krups","Nespresso","DeLonghi","Nespresso","DeLonghi","DeLonghi","Nespresso","Krups","DeLonghi","DeLonghi","Krups","Krups","DeLonghi","DeLonghi","Krups","Krups","Krups","DeLonghi","Nespresso","Nespresso","DeLonghi","DeLonghi","Nespresso","DeLonghi","Nespresso","LatteGo","LatteGo","DeLonghi","LatteGo","Krups","Krups","Krups","DeLonghi","DeLonghi","Krups","DeLonghi","Nespresso","DeLonghi","Krups","Nespresso","Nespresso","LatteGo","DeLonghi","Nespresso","Nespresso","Nespresso","DeLonghi","DeLonghi","DeLonghi","DeLonghi","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","DeLonghi","LatteGo","Nespresso","Nespresso","Nespresso","DeLonghi","DeLonghi","Krups","Nespresso","Nespresso","DeLonghi","DeLonghi","DeLonghi","DeLonghi","DeLonghi","LatteGo","LatteGo","Krups","LatteGo","Nespresso","LatteGo","Nespresso","Nespresso","Krups","DeLonghi","DeLonghi","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Krups","Nespresso","Nespresso","DeLonghi","DeLonghi","DeLonghi","Krups","Nespresso","Nespresso","Krups","Krups","Nespresso","DeLonghi","Krups","DeLonghi","Nespresso","Nespresso","DeLonghi","Nespresso","Nespresso","DeLonghi","DeLonghi","DeLonghi","DeLonghi","DeLonghi","Nespresso","DeLonghi","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","DeLonghi","Nespresso","DeLonghi","Nespresso","Krups","DeLonghi","DeLonghi","Nespresso","LatteGo","Nespresso","LatteGo","Nespresso","LatteGo","Nespresso","DeLonghi","DeLonghi","DeLonghi","Nespresso","DeLonghi","DeLonghi","DeLonghi","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Nespresso","Krups","Krups","LatteGo","LatteGo","LatteGo","LatteGo","LatteGo","LatteGo","LatteGo","LatteGo","LatteGo","LatteGo","LatteGo"],["2022-01-10","2022-01-08","2022-01-07","2022-01-06","2022-01-05","2022-01-05","2022-01-04","2022-01-04","2022-01-04","2022-01-04","2021-12-28","2021-12-27","2021-12-24","2021-12-23","2021-12-23","2021-12-23","2021-12-23","2021-12-23","2021-12-22","2021-12-22","2021-12-22","2021-12-21","2021-12-21","2021-12-20","2021-12-20","2021-12-20","2021-12-20","2021-12-20","2021-12-20","2021-12-18","2021-12-17","2021-12-16","2021-12-16","2021-12-15","2021-12-15","2021-12-15","2021-12-15","2021-12-15","2021-12-15","2021-12-15","2021-12-15","2021-12-15","2021-12-14","2021-12-14","2021-12-13","2021-12-13","2021-12-12","2021-12-11","2021-12-11","2021-12-10","2021-12-10","2021-12-10","2021-12-10","2021-12-09","2021-12-09","2021-12-09","2021-12-09","2021-12-09","2021-12-09","2021-12-08","2021-12-07","2021-12-07","2021-12-07","2021-12-07","2021-12-06","2021-12-06","2021-12-06","2021-12-06","2021-12-04","2021-12-04","2021-12-04","2021-12-03","2021-12-03","2021-12-02","2021-12-02","2021-12-02","2021-12-02","2021-12-02","2021-12-02","2021-12-02","2021-12-02","2021-12-02","2021-12-02","2021-12-02","2021-12-02","2021-12-02","2021-12-02","2021-11-30","2021-11-30","2021-11-30","2021-11-30","2021-11-30","2021-11-29","2021-11-29","2021-11-29","2021-11-28","2021-11-28","2021-11-28","2021-11-27","2021-11-27","2021-11-26","2021-11-26","2021-11-25","2021-11-25","2021-11-25","2021-11-25","2021-11-25","2021-11-25","2021-11-24","2021-11-24","2021-11-24","2021-11-23","2021-11-23","2021-11-23","2021-11-23","2021-11-23","2021-11-23","2021-11-23","2021-11-22","2021-11-22","2021-11-22","2021-11-22","2021-11-22","2021-11-21","2021-11-21","2021-11-21","2021-11-20","2021-11-20","2021-11-20","2021-11-20","2021-11-20","2021-11-20","2021-11-19","2021-11-19","2021-11-19","2021-11-19","2021-11-19","2021-11-19","2021-11-18","2021-11-18","2021-11-16","2021-11-16","2021-11-16","2021-11-16","2021-11-16","2021-11-15","2021-11-15","2021-11-15","2021-11-15","2021-11-15","2021-11-14","2021-11-14","2021-11-13","2021-11-13","2021-11-12","2021-11-12","2021-11-12","2021-11-12","2021-11-12","2021-11-11","2021-11-11","2021-11-11","2021-11-11","2021-11-11","2021-11-11","2021-11-11","2021-11-11","2021-11-10","2021-11-10","2021-11-10","2021-11-09","2021-11-09","2021-11-09","2021-11-08","2021-11-08","2021-11-08","2021-11-08","2021-11-08","2021-11-07","2021-11-07","2021-11-07","2021-11-07","2021-11-06","2021-11-05","2021-11-04","2021-11-04","2021-11-03","2021-11-03","2021-11-01","2021-10-31","2021-10-29","2021-10-29","2021-10-29","2021-10-29","2021-10-27","2021-10-27","2021-10-26","2021-10-25","2021-10-22","2021-10-22","2021-10-22","2021-10-22","2021-10-22","2021-10-22","2021-10-22","2021-10-21","2021-10-20","2021-10-20","2021-10-20","2021-10-20","2021-10-20","2021-10-19","2021-10-19","2021-10-19","2021-10-19","2021-10-18","2021-10-17","2021-10-16","2021-10-15","2021-10-14","2021-10-14","2021-10-13","2021-10-11","2021-10-11","2021-10-08","2021-10-08","2021-10-08","2021-10-08","2021-10-08","2021-10-08","2021-10-07","2021-10-06","2021-10-06","2021-10-06","2021-10-05","2021-10-05","2021-10-04","2021-10-04","2021-10-04","2021-10-04","2021-10-03","2021-10-02","2021-10-02","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-10-01","2021-09-30","2021-09-30","2021-09-30","2021-09-30","2021-09-30","2021-09-30","2021-09-30","2021-09-30","2021-09-30","2021-09-29","2021-09-29","2021-09-28","2021-09-27","2021-09-14","2021-09-13","2021-09-13","2021-09-13","2021-09-13","2021-09-10","2021-09-08","2021-09-08","2021-09-06","2021-09-03","2021-09-01"],[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,67,68,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100,101,102,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,118,119,120,121,122,123,124,125,126,127,128,129,130,131,132,133,134,135,136,137,138,139,140,141,142,143,144,145,146,147,148,149,150,151,152,153,154,155,156,157,158,159,160,161,162,163,164,165,166,167,168,169,170,171,172,173,174,175,176,177,178,179,180,181,182,183,184,185,186,187,188,189,190,191,192,193,194,195,196,197,198,199,200,201,202,203,204,205,206,207,208,209,210,211,212,213,214,215,216,217,218,219,220,221,222,223,224,225,226,227,228,229,230,231,232,233,234,235,236,237,238,239,240,241,242,243,244,245,246,247,248,249,250,251,252,253,254,255,256,257,258,259,260,261,262,263,264,265,266,267,268,269,270,271,272,273,274,275,276,277,278,279,280,281,282,283,284,285,286,287,288,289]],"container":"<table class=\"display\">\n  <thead>\n    <tr>\n      <th>TITLE<\/th>\n      <th>MENTION_SNIPPET<\/th>\n      <th>DATE<\/th>\n      <th>SOURCE_TYPE<\/th>\n      <th>AUTHOR<\/th>\n      <th>FROM<\/th>\n      <th>kword<\/th>\n      <th>datum<\/th>\n      <th>clanak<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"pageLength":5,"scrollX":true,"columnDefs":[{"className":"dt-right","targets":8}],"order":[],"autoWidth":false,"orderClasses":false,"orderCellsTop":true,"lengthMenu":[5,10,25,50,100]}},"evals":[],"jsHooks":[]}</script>
```

</div>
<br>

U sljedećem koraku provodimo *tokenizaciju*, odnosno pretvaranje teksta na jedinice analize koje su u ovom slučaju su  riječi:

<br>
<button class="btn btn-primary" data-toggle="collapse" data-target="#Block4"> Pregled tokeniziranih objava </button>  
<div id="Block4" class="collapse">


```r
# tokenizacija

newskava %>% 
  unnest_tokens(word, MENTION_SNIPPET) -> newskava_token 

#newsCOVID_token$word <- stri_encode(newsCOVID_token$word, "", "UTF-8") # prilagodi encoding

datatable(newskava_token, rownames = FALSE, filter="top", options = list(pageLength = 5, scrollX=T) )
```

```{=html}
<div id="htmlwidget-fec5ee325d2ae7683008" style="width:100%;height:auto;" class="datatables html-widget"></div>
```

</div>
<br>

Potom valja očistiti riječi od brojeva i nepotrebnih riječi. Na tako uređenim podatcima ćemo potom napraviti deskriptivno-statistički pregled teksta.

<br>
<button class="btn btn-primary" data-toggle="collapse" data-target="#Block5"> Pregled tokeniziranih i uređenih riječi </button>  
<div id="Block5" class="collapse">


```r
## Ukloni "stop words", brojeve, veznike i pojedinačna slova

newskava_token %>% 
  anti_join(stop_corpus, by = "word") %>%
  mutate(word = gsub("\\d+", NA, word)) %>%
  mutate(word = gsub("^[a-zA-Z]$", NA, word)) %>% 
  drop_na(.)-> newskava_tokenTidy

datatable(newskava_tokenTidy, rownames = FALSE, filter="top", options = list(pageLength = 5, scrollX=T) )
```

```{=html}
<div id="htmlwidget-2c472d8400aa449e36eb" style="width:100%;height:auto;" class="datatables html-widget"></div>
```

</div>
<br>

Na tako uređenim podatcima ćemo napraviti deskriptivno-statistički pregled teksta:

<br>
<button class="btn btn-primary" data-toggle="collapse" data-target="#Block6"> Osnovni deskriptivni pregled teksta </button>  
<div id="Block6" class="collapse">


```r
## Vremenski raspon podatka
range(newskava_token$DATE)
```

```
## [1] "2021-09-01" "2022-01-10"
```

```r
## Najčešće riječi
newskava_tokenTidy %>%
  count(word, sort = T) %>%
  head(25)
```

```
##             word   n
## 1       hrvatska 147
## 2      nespresso  92
## 3      de’longhi  60
## 4        lattego  45
## 5            dom  35
## 6        bauhaus  33
## 7         coffee  31
## 8           kavu  31
## 9        philips  31
## 10       samsung  31
## 11          kave  21
## 12        istria  20
## 13       mikulec  19
## 14        aparat  18
## 15       citroën  18
## 16          jysk  18
## 17          kler  18
## 18    namještaja  18
## 19    pogledajte  18
## 20        qualis  18
## 21         salon  18
## 22        akcija  17
## 23         krups  17
## 24        možete  17
## 25 philipscoffee  17
```

```r
## Vizualizacija najčešćih riječi
newskava_tokenTidy %>%
  count(word, sort = T) %>%
  filter(n > 10) %>%
  mutate(word = reorder(word, n)) %>%
  ggplot(aes(word, n)) +
  geom_col() +
  xlab(NULL) +
  coord_flip() +
  theme_economist()
```

![](Prez_files/figure-html/dekriptivnoTxt-1.png)<!-- -->

```r
## Vizualizacija najčešćih riječi kroz vrijeme
newskava_tokenTidy %>%
   mutate(Datum = floor_date(datum, "day")) %>%
   group_by(Datum) %>%
   count(word) %>% 
   mutate(gn = sum(n)) %>%
   filter(word %in%  c("nespresso", "de’longhi", "lattego", "krups")) %>%
   ggplot(., aes(Datum,  n / gn)) + 
   geom_point() +
   ggtitle("Učestalost korištenja kroz vrijeme") +
   ylab("% ukupnih riječi") +
   geom_smooth() +
   facet_wrap(~ word, scales = "free_y") +
   scale_y_continuous(labels = scales::percent_format())+
   theme_economist()
```

![](Prez_files/figure-html/dekriptivnoTxt-2.png)<!-- -->

</div>
<br>

...i  deskriptivno-statistički pregled objava:

<br>
<button class="btn btn-primary" data-toggle="collapse" data-target="#Block7"> Osnovni deskriptivni pregled objava </button>  
<div id="Block7" class="collapse">


```r
## Broj domena
newskava_tokenTidy %>% 
  summarise(Domena = n_distinct(SOURCE_TYPE))
```

```
##   Domena
## 1      6
```

```r
## Broj objava po domeni

kava %>% 
 # drop_na(.) %>%
  group_by(SOURCE_TYPE) %>%
  summarise(n = n()) %>%
  arrange(desc(n)) %>% 
  head(20)
```

```
## # A tibble: 7 x 2
##   SOURCE_TYPE     n
##   <chr>       <int>
## 1 web           110
## 2 facebook       73
## 3 instagram      51
## 4 forum          44
## 5 youtube         5
## 6 twitter         4
## 7 reddit          2
```

```r
## Broj objava po brandu

kava %>% 
 # drop_na(.) %>%
  group_by(kword) %>%
  summarise(n = n()) %>%
  arrange(desc(n)) %>% 
  head(20)
```

```
## # A tibble: 4 x 2
##   kword         n
##   <chr>     <int>
## 1 Nespresso   135
## 2 DeLonghi     84
## 3 Krups        35
## 4 LatteGo      35
```

```r
## Broj članaka po domeni 

newskava %>% 
   mutate(Datum = floor_date(datum, "week")) %>%
   group_by(Datum, SOURCE_TYPE) %>%
   summarise(n = n()) %>%
   ungroup() %>%
   ggplot(., aes(Datum,  n)) + 
   geom_line() +
   ggtitle("Broj članaka o kafe aparatima kroz vrijeme") +
   ylab("Broj članaka") +
   geom_smooth() +
   facet_wrap(~ SOURCE_TYPE, scales = "free_y") +
   theme_economist()
```

![](Prez_files/figure-html/dekriptivnoDom-1.png)<!-- -->

```r
## Broj objava kroz vrijeme 

newskava %>% 
   mutate(Datum = floor_date(datum, "week")) %>%
   group_by(Datum, kword) %>%
   summarise(n = n()) %>%
   ungroup() %>%
   ggplot(., aes(Datum,  n)) + 
   geom_line() +
   ggtitle("Članci na najvažnijim portalima") +
   ylab("Broj objavljenih COVID članaka") +
   geom_smooth() +
   facet_wrap(~ kword, scales = "free_y") +
   theme_economist()
```

![](Prez_files/figure-html/dekriptivnoDom-2.png)<!-- -->

</div>
<br>

## Analiza sentimenta

Nakon uređivanja podataka i osnovnog pregleda ćemo provesti analizu **sentimenta**. Za analizu sentimenta je potrebno preuzeti leksikone koji su za hrvatski jezik napravljeni u okviru FER-ovog [Croatian Sentiment Lexicon](http://meta-share.ffzg.hr/repository/browse/croatian-sentiment-lexicon/940fe19e6c6d11e28a985ef2e4e6c59eff8b12d75f284d58aacfa8d732467509/). Analiza sentimenta i uključuje sentiment kroz vrijeme, doprinos riječi sentimentu, 'wordCloud' i analizu negativnosti brandova.

Pogledajmo prvo kako izgledaju leksikoni (koje smo učitali još na početku):


<br>
<button class="btn btn-primary" data-toggle="collapse" data-target="#Block8"> Pregled leksikona sentimenta </button>  
<div id="Block8" class="collapse">


```r
## Pregled leksikona (negativne riječi)
CroSentilex_n %>% sample_n(10)
```

```
##            word sentiment brija
##  1:    svemoćan  0.492110   NEG
##  2:      bungeo  0.119140   NEG
##  3:    nigerija  0.620410   NEG
##  4:   primjeran  0.245610   NEG
##  5:      raskol  0.647700   NEG
##  6:     ozvučen  0.081471   NEG
##  7:  tužilaštvo  0.570660   NEG
##  8: isprobavati  0.519730   NEG
##  9:  zavjetrina  0.392540   NEG
## 10:      urođen  0.400040   NEG
```

```r
## Pregled leksikona (pozitivne riječi)
CroSentilex_p %>% sample_n(10)
```

```
##            word sentiment brija
##  1:       limes   0.10660   POZ
##  2:     barbika   0.34586   POZ
##  3:  mihailović   0.25689   POZ
##  4:  neisplativ   0.26536   POZ
##  5:   solventan   0.14321   POZ
##  6:       fagot   0.30432   POZ
##  7:  efikasnost   0.57516   POZ
##  8:   podravkin   0.46439   POZ
##  9:   dramatika   0.53868   POZ
## 10: presretnuti   0.59598   POZ
```

```r
## Pregled leksikona (sve riječi)
Crosentilex_sve %>% sample_n(10)
```

```
##              word sentiment brija
##  1:    hipnotički   0.40183   POZ
##  2:   premišljati   0.11874   NEG
##  3:     farnesina   0.21157   POZ
##  4:       mission   0.18703   NEG
##  5:      tehnički   0.28185   POZ
##  6:       zašliti   0.11885   POZ
##  7:      ilegalan   0.36917   NEG
##  8:           med   0.45837   POZ
##  9: starozavjetan   0.34750   POZ
## 10:     nedoličan   0.37100   NEG
```

```r
## Pregled leksikona (crosentilex Gold)
CroSentilex_Gold %>% sample_n(10)
```

```
##            word sentiment
## 1        sličan         0
## 2     obrazovan         2
## 3     zahvatiti         0
## 4        zračan         0
## 5      ugroziti         1
## 6    španjolski         0
## 7  veleposlanik         0
## 8       veljača         0
## 9          naći         2
## 10    izvještaj         0
```

</div>
<br>


Provjerimo kretanje sentimenta u vremenu:


<br>
<button class="btn btn-primary" data-toggle="collapse" data-target="#Block9"> Pregled leksikona sentimenta </button>  
<div id="Block9" class="collapse">


```r
## Kretanje sentimenta u vremenu 

vizualiziraj_sentiment <- function(dataset, frq = "week") {

dataset %>%
  inner_join( Crosentilex_sve, by = "word") %>%
  filter(!is.na(word)) %>%
  select(word, brija, datum, sentiment) %>% 
  unique() %>%
  spread(. , brija, sentiment) %>%
  mutate(sentiment = POZ - NEG) %>%
  select(word, datum, sentiment) %>% 
  group_by(word) %>% 
  mutate(count = n()) %>%
  arrange(desc(count)) %>%
  mutate( score = sentiment*count) %>%
  ungroup() %>%
  group_by(datum) %>%
  arrange(desc(datum)) -> sm

 
sm %>%
  select(datum, score) %>%
  group_by(Datum = floor_date(datum, frq)) %>%
  summarise(Dnevni_sent = sum(score, na.rm = TRUE)) %>%
  ggplot(., aes(Datum, Dnevni_sent)) +
  geom_bar(stat = "identity") + 
  ggtitle(paste0("Sentiment kroz vrijeme/frekvencija podataka:", frq)) +
  ylab("SentimentScore") +
  theme_economist()-> gg_sentiment_kroz_vrijeme_qv


gg_sentiment_kroz_vrijeme_qv

}

vizualiziraj_sentiment(newskava_tokenTidy,"week")
```

![](Prez_files/figure-html/sentimentTempus-1.png)<!-- -->

</div>
<br>


Korisno je i promotriti koje riječi najviše doprinose sentimentu (pozitivnom, negativnom i neutralnom):

<br>
<button class="btn btn-primary" data-toggle="collapse" data-target="#Block10"> Pregled riječi prema doprinosu sentimentu </button>  
<div id="Block10" class="collapse">



```r
## Doprinos sentimentu
doprinos_sentimentu <- function(dataset, no = n) {
dataset %>%
  inner_join(CroSentilex_Gold, by = "word") %>% 
  count(word, sentiment,sort = TRUE) %>% 
  group_by(sentiment) %>%
  top_n(no) %>%
  ungroup() %>%
  mutate(sentiment = case_when(sentiment == 0 ~ "NEUTRALNO",
                                 sentiment == 1 ~ "NEGATIVNO",
                                 sentiment == 2 ~ "POZITIVNO")) %>%
  mutate(word = reorder(word, n)) %>%
  ggplot(aes(word, n, fill = sentiment)) +
  geom_col(show.legend = FALSE) +
  ggtitle( "Doprinos sentimentu") +
  labs( x = "Riječ", y = "Broj riječi") +
  facet_wrap(~ sentiment, scales = "free_y") +
  coord_flip() +
  theme_economist() -> gg_doprinos_sentimentu
  
 gg_doprinos_sentimentu
 
}


doprinos_sentimentu(newskava_tokenTidy,15)
```

![](Prez_files/figure-html/doprinoSentimentu-1.png)<!-- -->

</div>
<br>


Korisno je pogledati i WordCloud sentiment. Pogledajmo "obični" WordCloud prije toga:


```r
## WordCloud(vulgaris)
newskava_tokenTidy %>%
  anti_join(CroSentilex_Gold,by="word") %>% 
  count(word) %>% 
  arrange(desc(n)) %>%
  top_n(100) %>%
  with(wordcloud(word, n, max.words = 80)) 
```

![](Prez_files/figure-html/WCloud-1.png)<!-- -->

Ovako izgleda WordCloud koji sadržava i prikaz sentimenta:


```r
## ComparisonCloud
newskava_tokenTidy %>%
  inner_join(CroSentilex_Gold,by="word") %>% 
  count(word, sentiment) %>% 
  top_n(200) %>%
  mutate(sentiment = case_when(sentiment == 0 ~ "+/-",
                                 sentiment == 1 ~ "-",
                                 sentiment == 2 ~ "+")) %>%
  acast(word ~ sentiment, value.var = "n", fill = 0) %>%
  comparison.cloud(colors = c("firebrick3", "deepskyblue3","darkslategray"),
                   max.words = 120)
```

![](Prez_files/figure-html/WCloutSent-1.png)<!-- -->

Analiza sentimenta se može iskoristiti za pregled negativnosti pojedinih brandova:

<br>
<button class="btn btn-primary" data-toggle="collapse" data-target="#Block11"> Pregled brandova prema negativnosti </button>  
<div id="Block11" class="collapse">


```r
## Najnegativniji brandovi
wCount <- newskava_tokenTidy %>% 
  group_by(kword) %>%
  summarise(word = n())

CroSentilex_Gold_neg <- CroSentilex_Gold %>% filter(sentiment == 1)
CroSentilex_Gold_poz <- CroSentilex_Gold %>% filter(sentiment == 2)


newskava_tokenTidy %>% 
  semi_join(CroSentilex_Gold_neg, by= "word") %>%
  group_by(kword) %>% 
  summarise(negWords = n()) %>%
  left_join(wCount, by = "kword") %>%
  mutate(negativnostIndex = (negWords/word)*100) %>%
  arrange(desc(negativnostIndex))
```

```
## # A tibble: 2 x 4
##   kword     negWords  word negativnostIndex
##   <chr>        <int> <int>            <dbl>
## 1 LatteGo          1   547           0.183 
## 2 Nespresso        1  1086           0.0921
```

</div>
<br>

...također i pozitivnosti brandova:

<br>
<button class="btn btn-primary" data-toggle="collapse" data-target="#Block12"> Pregled brandova prema pozitivnosti </button>  
<div id="Block12" class="collapse">


```r
## Najpozitivniji brandovi

CroSentilex_Gold_poz <- CroSentilex_Gold %>% filter(sentiment == 2)

newskava_tokenTidy %>% 
  semi_join(CroSentilex_Gold_poz, by= "word") %>%
  group_by(kword) %>% 
  summarise(pozWords = n()) %>%
  left_join(wCount, by = "kword") %>%
  mutate(pozitivnostIndex = (pozWords/word)*100) %>%
  arrange(desc(pozitivnostIndex))  
```

```
## # A tibble: 4 x 4
##   kword     pozWords  word pozitivnostIndex
##   <chr>        <int> <int>            <dbl>
## 1 DeLonghi        41  1368             3.00
## 2 Nespresso       20  1086             1.84
## 3 LatteGo         10   547             1.83
## 4 Krups            4   233             1.72
```

</div>
<br>



## Analiza vaznosti pojmova

Nakon analize sentimenta je korisno analizirati i **najbitnije** riječi. To se radi pomoću [IDF (inverse document frequency)](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.438.2284&rep=rep1&type=pdf) metode. IDF metoda omogućuje identifikaciju važnih (ne nužno čestih) riječi u korpusu i može poslužiti za analizu najvažnijih pojmova po brandovima.


<br>
<button class="btn btn-primary" data-toggle="collapse" data-target="#Block13"> Pregled najvažnijih riječi </button>  
<div id="Block13" class="collapse">


```r
## Udio riječi po domenama

domenaWords <- newskava %>%
  unnest_tokens(word,MENTION_SNIPPET) %>% 
  count(kword, word, sort = T)
  
ukupnoWords <- domenaWords %>%
  group_by(kword) %>%
  summarise(totWords = sum(n))

domenaWords <- left_join(domenaWords, ukupnoWords)


# domenaWords %>% head(15)

# domenaWords %>% 
# ggplot(., aes(n/totWords, fill = domena)) +
#   geom_histogram(show.legend = FALSE) +
#   xlim(NA, 0.0009) +
#   facet_wrap(~domena, ncol = 2, scales = "free_y")

## Najbitnije riječi po domenma

idf <- domenaWords %>%
  bind_tf_idf(word, kword, n)

idf %>% head(30)
```

```
##        kword      word   n totWords          tf       idf      tf_idf
## 1  Nespresso nespresso 181     4143 0.043688149 0.2876821 0.012568297
## 2   DeLonghi  hrvatska 146     2607 0.056003069 0.6931472 0.038818369
## 3  Nespresso        za 144     4143 0.034757422 0.0000000 0.000000000
## 4  Nespresso         i 136     4143 0.032826454 0.0000000 0.000000000
## 5  Nespresso         u 115     4143 0.027757664 0.0000000 0.000000000
## 6  Nespresso        je 108     4143 0.026068067 0.0000000 0.000000000
## 7   DeLonghi         u  82     2607 0.031453778 0.0000000 0.000000000
## 8   DeLonghi de’longhi  72     2607 0.027617952 1.3862944 0.038286611
## 9    LatteGo   lattego  58     1088 0.053308824 1.3862944 0.073901721
## 10     Krups     krups  50     1142 0.043782837 0.6931472 0.030347950
## 11 Nespresso         s  47     4143 0.011344436 0.0000000 0.000000000
## 12  DeLonghi        za  45     2607 0.017261220 0.0000000 0.000000000
## 13 Nespresso        na  45     4143 0.010861694 0.0000000 0.000000000
## 14  DeLonghi         s  44     2607 0.016877637 0.0000000 0.000000000
## 15  DeLonghi         i  41     2607 0.015726889 0.0000000 0.000000000
## 16  DeLonghi        je  39     2607 0.014959724 0.0000000 0.000000000
## 17   LatteGo   philips  37     1088 0.034007353 0.6931472 0.023572101
## 18 Nespresso      kavu  37     4143 0.008930727 0.0000000 0.000000000
## 19 Nespresso      kave  36     4143 0.008689356 0.2876821 0.002499772
## 20  DeLonghi       dom  35     2607 0.013425393 1.3862944 0.018611547
## 21 Nespresso       ili  35     4143 0.008447985 0.2876821 0.002430334
## 22  DeLonghi   bauhaus  33     2607 0.012658228 1.3862944 0.017548030
## 23  DeLonghi        na  32     2607 0.012274645 0.0000000 0.000000000
## 24  DeLonghi   samsung  31     2607 0.011891063 1.3862944 0.016484513
## 25     Krups         i  30     1142 0.026269702 0.0000000 0.000000000
## 26   LatteGo        za  30     1088 0.027573529 0.0000000 0.000000000
## 27 Nespresso    aparat  28     4143 0.006758388 0.0000000 0.000000000
## 28     Krups        za  27     1142 0.023642732 0.0000000 0.000000000
## 29 Nespresso        uz  26     4143 0.006275646 0.0000000 0.000000000
## 30  DeLonghi      kavu  25     2607 0.009589567 0.0000000 0.000000000
```

```r
# idf %>% 
#   select(-totWords) %>%
#   arrange(desc(tf_idf))

idf %>%
  arrange(desc(tf_idf)) %>%
  mutate(word = factor(word, levels = rev(unique(word)))) %>% 
  mutate(domena = factor(kword)) %>%
  group_by(domena) %>% 
  top_n(10,tf_idf) %>% 
  ungroup() %>%
  ggplot(aes(word, tf_idf, fill = kword)) +
  geom_col(show.legend = FALSE) +
  labs(x = NULL, y = "tf-idf") +
  facet_wrap(~kword, scales = "free") +
  coord_flip() +
  theme_economist()
```

![](Prez_files/figure-html/frekvencija-1.png)<!-- -->

</div>
<br>


## nGrami

Do sada smo analizirali tekst na osnovi pojedinačnih riječi. Takav pristup ograničava  nalaze do kojih je moguće doći kada se tekst sagleda na osnovi fraza (dvije ili *n* riječi). U sljedećemo koraku ćemo tokenizirati tekst na bigrame (dvije riječi) kako bismo proveli frazeološku analizu. Korištenje bigrama otvara mogućnosti korištenja dodatnih pokazatelja pa ćemo provesti i analizu korelacije među riječima.

<br>
<button class="btn btn-primary" data-toggle="collapse" data-target="#Block14"> Pregled najvažnijih bigrama </button>  
<div id="Block14" class="collapse">


```r
## tokeniziraj na bigram

newskava_bigram <- newskava %>%
  unnest_tokens(bigram, MENTION_SNIPPET, token = "ngrams", n = 2)

## pregledaj podatke

# newskava_bigram %>% head(25)


## najvažniji bigrami

newskava_bigram %>%
  count(bigram, sort = T) %>%
  head(25)
```

```
##                      bigram  n
## 1                   za kavu 64
## 2        de’longhi hrvatska 44
## 3                 aparat za 36
## 4          bauhaus hrvatska 33
## 5              nespresso je 24
## 6                      je u 22
## 7          istria de’longhi 20
## 8                 my istria 20
## 9             jysk hrvatska 18
## 10            kler hrvatska 18
## 11          namještaja kler 18
## 12             qualis salon 18
## 13         salon namještaja 18
## 14                    dom s 16
## 15       hrvatska family.hr 16
## 16         mirjanom mikulec 16
## 17               s mirjanom 16
## 18              tražimo dom 16
## 19                   dom po 15
## 20                  moj dom 15
## 21                   za sve 15
## 22      family.hr de’longhi 14
## 23          hrvatska svijet 14
## 24             lesnina xxxl 14
## 25 philipshomeliving coffee 14
```

```r
newskava_bigram_sep <- newskava_bigram %>%
  separate(bigram, c("word1","word2"), sep = " ")

newskava_bigram_tidy <- newskava_bigram_sep %>%
  filter(!word1 %in% stop_corpus$word) %>%
  filter(!word2 %in% stop_corpus$word) %>%
  mutate(word1 = gsub("\\d+", NA, word1)) %>%
  mutate(word2 = gsub("\\d+", NA, word2)) %>%
  mutate(word1 = gsub("^[a-zA-Z]$", NA, word1)) %>%
  mutate(word2 = gsub("^[a-zA-Z]$", NA, word2)) %>% 
  drop_na(.)


newskava_bigram_tidy_bigram_counts <- newskava_bigram_tidy %>% 
  count(word1, word2, sort = TRUE)


#newsCOVID_bigram_tidy_bigram_counts

bigrams_united <- newskava_bigram_tidy %>%
  drop_na(.) %>%
  unite(bigram, word1, word2, sep = " ")

#bigrams_united

bigrams_united %>% 
  count(clanak,bigram,sort = T) -> topicBigram

# Najvažniji bigrami po brandovima

 bigram_tf_idf <- bigrams_united %>%
  count(kword, bigram) %>%
  bind_tf_idf(bigram, kword, n) %>%
  arrange(desc(tf_idf))

bigram_tf_idf %>%
  arrange(desc(tf_idf)) %>%
  mutate(bigram = factor(bigram, levels = rev(unique(bigram)))) %>% 
  group_by(kword) %>% 
  top_n(7) %>% 
  ungroup() %>%
  ggplot(aes(bigram, tf_idf, fill = kword)) +
  geom_col(show.legend = FALSE) +
  labs(x = NULL, y = "tf-idf") +
  facet_wrap(~kword, ncol = 2, scales = "free") +
  coord_flip() + 
  theme_economist()
```

![](Prez_files/figure-html/nGRAMI-1.png)<!-- -->

</div>
<br>





Provjerimo koje su riječi najviše korelirane sa izabranim ključnim riječima:


<br>
<button class="btn btn-primary" data-toggle="collapse" data-target="#Block15"> Pregled korelacije između branda i riječi </button>  
<div id="Block15" class="collapse">


```r
# Korelacije riječi ( R crash na T=30)

#newsCOVID_tokenTidy %>% 
#  filter(published == "2020-04-22") %>%
#  pairwise_count(word, domena, sort = T) %>%
#  filter_all(any_vars(!is.na(.))) -> pairsWords

newskava_tokenTidy %>% 
#  filter(datum > "2020-02-20") %>%
  group_by(word) %>%
  filter(n() > 20) %>%
  filter(!is.na(word)) %>%
  pairwise_cor(word,datum, sort = T) -> corsWords

#corsWords %>%
#  filter(item1 == "oporavak")

corsWords %>%
  filter(item1 %in% c("de’longhi", "krups", "lattego", "nespresso", "dom")) %>%
  group_by(item1) %>%
  top_n(10) %>%
  ungroup() %>%
  mutate(item2 = reorder(item2, correlation)) %>%
  ggplot(aes(item2, correlation)) +
  geom_bar(stat = "identity") +
  facet_wrap(~ item1, scales = "free") +
  coord_flip() + 
  theme_economist()
```

![](Prez_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

</div>
<br>

## Tematska analiza

Na kraju provodimo tematsku analizu kao najsloženiji dio do sada provedene analize. Pri tome koristimo LDA (Latent Dirichlet allocation) algoritam kako bismo pronašli najvažnije riječi u algoritamski identificiranim temama. Ovdje je važno primijetiti da prije provedbe LDA modela tokenizirane riječi treba pretvoriti u matricu pojmova (document term matrix) koju ćemo kasnije koristiti kao input za LDA algoritam.


<br>
<button class="btn btn-primary" data-toggle="collapse" data-target="#Block16"> Tematska (4) analiza </button>  
<div id="Block16" class="collapse">


```r
newskava_tokenTidy %>%
  count(clanak, word, sort = TRUE) %>%
  cast_dtm(clanak, word,n) -> dtm

newskava_LDA <- LDA(dtm, k = 4,  control = list(seed = 1234))

newskava_LDA_tidy <- tidy(newskava_LDA, matrix = "beta")
#newsCOVID_LDA_tidy

newskava_terms <- newskava_LDA_tidy %>%
  drop_na(.) %>%
  group_by(topic) %>%
  top_n(15, beta) %>%
  ungroup() %>%
  arrange(topic, -beta)

#newsCOVID_terms

newskava_terms %>%
  mutate(term = reorder_within(term, beta, topic)) %>%
  ggplot(aes(term, beta, fill = factor(topic))) +
  geom_col(show.legend = FALSE) +
  facet_wrap(~ topic, scales = "free") +
  coord_flip() +
  scale_x_reordered() + 
  theme_economist()
```

![](Prez_files/figure-html/TEME-1.png)<!-- -->

</div>
<br>

Tematsku analizu je moguće i napraviti na bigramski tokeniziranom tekstu. Tada je često moguće doći do preciznijih i kontekstualno relevantnijih uvida:


<br>
<button class="btn btn-primary" data-toggle="collapse" data-target="#Block17"> Tematska (4) analiza -  bigrami </button>  
<div id="Block17" class="collapse">


```r
# Bigrami 

topicBigram %>%
  cast_dtm(clanak, bigram,n) -> dtmB

newskava_LDA <- LDA(dtmB, k = 4,  control = list(seed = 1234))

newskava_LDA_tidy <- tidy(newskava_LDA, matrix = "beta")
#newsCOVID_LDA_tidy

newskava_terms <- newskava_LDA_tidy %>%
  drop_na(.) %>%
  group_by(topic) %>%
  top_n(10, beta) %>%
  ungroup() %>%
  arrange(topic, -beta)

#newsCOVID_terms


newskava_terms %>%
  mutate(term = reorder_within(term, beta, topic)) %>%
  ggplot(aes(term, beta, fill = factor(topic))) +
  geom_col(show.legend = FALSE) +
  facet_wrap(~ topic, scales = "free") +
  coord_flip() +
  scale_x_reordered() + 
  theme_economist()
```

![](Prez_files/figure-html/TEMEbigram-1.png)<!-- -->

</div>
<br>


## Zaključak

U ovom smo predavanju dali uvodni pregled mogućnosti analize teksta u okviru `tidytext` paketa. Riječ je o skupu alata koji omogućavaju "prilagodbu" teksta u *tidy* format i daljnu analizu s `tidyverse` alatima koje smo do sada već dobro upoznali. `tidytext` nije jedini dostupan okvir za analizu teksta u R, već postoji i niz drugih paketa (vidi na početku) koji omogućavaju korištenje naprednijih (algoritamkskih tehnika.

U predavanju su korišteni tekstovi objavljeni u svim hrvatskim medijima o proizvođačima aparata za kavu u razdoblju četiri mjeseca. Predavanje je imalo za cilj demonstrirati uvodne mogućnosti tekstualne analize te osnovnih tehnika i alata.

Analiza teksta (NLP) je trenutno (brzo) rastuće istraživačko područje sa sve većim brojem primjena, novih etodoloških pristupa i perspektiva. Dostupno je mnoštvo kvalitetnih i korisnih resursa pa se zainteresiranim studentima preporuča uključivanje u ovu (vrlo perspektivnu) istraživačku paradigmu.