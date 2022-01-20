---
title: "<center><div class='mytitle'>Analiza tržišta aparata za kavu</div></center>"
author: "<center><div class='mysubtitle'>Konkurenti: De`Longhi, LatteGo, Krups, Nespresso</div></center>"
date: "<center><div class='mysubtitle'></div></center>"
output:
  html_document:
    theme: yeti
    highlight: espresso
    toc: yes
    toc_depth: 4
    toc_float: yes
    keep_md: yes
    css: style.css
    includes:
      before_body: header.html
      after_body: footer.html
  
---
















## **Opće**
***
<br>


- Medijske objave o aparatima za kavu u Hrvatskoj (*LatteGo*, *De`Longhi*, *Krups* i *Nesspreso*)
<br><br>
- Cijeli medijski prostor u Hrvatskoj 
<br><br>
- Razdoblje od 2021-01-09 do 2022-11-01
<br><br>
- Podatci sa [mediatoolkit](https://www.mediatoolkit.com/) servisa
<br><br>
- 290 objava koje sadrže ukupno 8.980 riječi
<br><br>
- Izvještaj uključuje: *pregled medijskog prostora*, *analizu sentimenta*, *analizu sadržaja* i *tematsku analizu*
















## **Medijski prostor**
***
<br>

- Najvažniji mediji su Web, Facebook, Instagram i forum
<br><br>
- Nespresso dominira u medijskom prostoru, a DeLonghi slijedi
<br><br>
- Krups i LatteGo zaostaju i na sličnim pozicijama
<br><br>
- Medijske kampanje u dvomjesečnim ciklusima
<br><br>
- Facebook, forum, Instagram imaju jednaku dinamiku
<br><br>
- Brandovi se međusobno prate u medijskom prostoru

<br>
<button class="btn btn-primary" data-toggle="collapse" data-target="#Block6"> Pregled medijskog prostora </button>  
<div id="Block6" class="collapse">

![](Prez_files/figure-html/dekriptivnoDom-1.png)<!-- -->![](Prez_files/figure-html/dekriptivnoDom-2.png)<!-- -->![](Prez_files/figure-html/dekriptivnoDom-3.png)<!-- -->![](Prez_files/figure-html/dekriptivnoDom-4.png)<!-- -->

</div>
<br>

- Najčešće riječi su nazivi brandova (Nespresso,DeLonghi i LatteGo)
<br><br>
- Često se spominju i riječi *dom, namještaj, akcije i veliki saloni namještaja*
<br><br>
- Ovo ukazuje na *važnost "mitologije doma"* (manja važnost "mitologije okusa" i/ili "mitologije stila")
<br><br>
- Od prosinca zaoštrena konkurencija između Nespresso, LatteGo i Krups


<br>

<button class="btn btn-primary" data-toggle="collapse" data-target="#Block7"> Najčešće riječi u tekstu </button>  
<div id="Block7" class="collapse">

![](Prez_files/figure-html/dekriptivnoTxt-1.png)<!-- -->![](Prez_files/figure-html/dekriptivnoTxt-2.png)<!-- -->

</div>
<br>

- Općenito **pozitivan sentiment** medijskih objava
<br><br>
- Pozitivan sentiment na osnovi tematike vezane uz **dom, obitelj i kvalitetu života**
<br><br>


<button class="btn btn-primary" data-toggle="collapse" data-target="#Block8"> Najčešće riječi u tekstu </button>  
<div id="Block8" class="collapse">

![](Prez_files/figure-html/WCloutSent-1.png)<!-- -->

</div>
<br>


## **Analiza sentimenta**
***
<br>

- Pozitivan sentiment u periodu pred i za vrijeme blagdana
<br><br>
- Razdoblje za plasman proizvoda na tržište kada je dom u fokusu

<br>
<button class="btn btn-primary" data-toggle="collapse" data-target="#Block9"> Kretanje sentimenta kroz vrijeme </button>  
<div id="Block9" class="collapse">

![](Prez_files/figure-html/sentimentTempus-1.png)<!-- -->

</div>
<br>


- Riječi koje najviše doprinose pozitivnom sentimentu vezane uz **dom**, zdravlje i kvalitetu života
<br><br>
- Važnost nagradnih igara i darivanja


<br>
<button class="btn btn-primary" data-toggle="collapse" data-target="#Block10"> Doprinos sentimentu </button>  
<div id="Block10" class="collapse">

![](Prez_files/figure-html/doprinoSentimentu-1.png)<!-- -->

</div>
<br>



- Najbolje plasirane medijske objave ima DeLongi (najveći indeks pozitivnosti)
<br><br>
- Dobra medijska strategija;znatno bolja pozicija od tržišnog lidera Nespresso
<br><br>
- Tržišni lider Nespresso, LatteGo i Krups slični






<br>
<button class="btn btn-primary" data-toggle="collapse" data-target="#Block12"> Indeks pozitivnosti brandova </button>  
<div id="Block12" class="collapse">

<table class=" lightable-classic-2" style='font-family: "Arial Narrow", "Source Sans Pro", sans-serif; margin-left: auto; margin-right: auto;'>
 <thead>
  <tr>
   <th style="text-align:left;"> kword </th>
   <th style="text-align:right;"> pozWords </th>
   <th style="text-align:right;"> word </th>
   <th style="text-align:right;"> pozitivnostIndex </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> DeLonghi </td>
   <td style="text-align:right;"> 41 </td>
   <td style="text-align:right;"> 1196 </td>
   <td style="text-align:right;"> 3.428094 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Nespresso </td>
   <td style="text-align:right;"> 20 </td>
   <td style="text-align:right;"> 1083 </td>
   <td style="text-align:right;"> 1.846722 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> LatteGo </td>
   <td style="text-align:right;"> 10 </td>
   <td style="text-align:right;"> 546 </td>
   <td style="text-align:right;"> 1.831502 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Krups </td>
   <td style="text-align:right;"> 4 </td>
   <td style="text-align:right;"> 233 </td>
   <td style="text-align:right;"> 1.716738 </td>
  </tr>
</tbody>
</table>

</div>
<br>



## **Najvažniji pojmovi**
***
<br>

- Najbitnije riječi izračunate pomoću [IDF (inverse document frequency)](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.438.2284&rep=rep1&type=pdf) metode 
<br><br>
- Vidljiv **fokus DeLonghija na dom**, salone namještaja i općenito tematiku doma (saloni namještaja)
<br><br>
- **Krups pozicioniran u domenu bijele tehnike** (saloni bijele tehnike)
<br><br>
- **LatteGo jako ističe okus, kvalitetu i tehnologiju** (prepoznatljivost u okviru branda Phillips)
<br><br>
- **Nespresso gradi brand na okusu i stilu življenja** (socijalizacija i kultura pijenja kave)

<br>
<button class="btn btn-primary" data-toggle="collapse" data-target="#Block13"> Pregled najvažnijih riječi za brand </button>  
<div id="Block13" class="collapse">

![](Prez_files/figure-html/frekvencija-1.png)<!-- -->

</div>
<br>


- Jednaki zaključci na osnovi fraza (bigrami)


<br>
<button class="btn btn-primary" data-toggle="collapse" data-target="#Block14"> Pregled najvažnijih bigrama </button>  
<div id="Block14" class="collapse">

![](Prez_files/figure-html/nGRAMI-1.png)<!-- -->

</div>
<br>





- Korelacija između DeLonghi i pojmova dom i Bauhaus
<br><br>
- LatteGo jako vezan uz brand Phillips
<br><br>
- Nespresso fokusiran na kavu (kvaliteta, priprema, stil)





<br>
<button class="btn btn-primary" data-toggle="collapse" data-target="#Block15"> Pregled korelacije između branda i riječi </button>  
<div id="Block15" class="collapse">

![](Prez_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

</div>
<br>

## **Tematska analiza**
***
- Identificirane **četiri teme**
<br><br>
- Jedna tema se odnosi na uređaje, karakteristike i brandove (tehnički aspekt)
<br><br>
- Druga tema je dom, opremanje doma i udobnost življenja
<br><br>
- Treća tema je okus i užitak pijenja kave
<br><br>
- Četvta tema tema su ponude, akcije, konkurencija

<br>
<button class="btn btn-primary" data-toggle="collapse" data-target="#Block16"> Tematska analiza </button>  
<div id="Block16" class="collapse">

![](Prez_files/figure-html/TEME-1.png)<!-- -->

</div>
<br>

- Slični rezultati i na tematskoj analizi fraza
<br><br>
- Phillips se izdvaja kao zasebna tema

<br>
<button class="btn btn-primary" data-toggle="collapse" data-target="#Block17"> Tematska analiza -  fraze </button>  
<div id="Block17" class="collapse">

![](Prez_files/figure-html/TEMEbigram-1.png)<!-- -->

</div>
<br>


## **Zaključak**
***
<br>

- Web najbitniji medij

<br><br>

- Dijeljenje originalnog web sadržaja na mrežama

<br><br>

- Nespresso tržišni lider ali DeLonghi bolje pozicioniran u objavma
<br><br>

>  *Mitologija doma* bolje prolazi kod domaće publike (DeLonghi)


> *Mitologija okusa i stila* manje populrna kod domaće publike (Nespresso)

<br><br>

 - Važne teme: tehnički aspekti, opremanje doma, užitak konzumacije i okus, akcije i ponuda






















