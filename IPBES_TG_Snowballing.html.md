---
title: Technical Guideline Series
subtitle: Snowballing for Scientific Literature Search and Analysis
date: today
author:
  - name: 
        family: Krug
        given: Rainer M.
    id: rmk
    orcid: 0000-0002-7490-0066
    email: Rainer.Krug@Senckenberg.de, Rainer@Krugs.de
    affiliation: 
      - name: Senckenberg
        city: Frankfurt (Main)
        url: https://www.senckenberg.de/en/institutes/sbik-f/
    roles: [author, editor]
  - name: 
        family: Niamir
        given: Aidin 
    id: an
    orcid: 0000-0003-4511-3407
    email: Aidin.Niamir@Senckenberg.de
    affiliation: 
      - name: Senckenberg
        city: Frankfurt (Main)
        url: https://www.senckenberg.de/en/institutes/sbik-f/
    roles: [editor]
abstract: > 
    In addition to the typical literature search using search terms, one can conduct a snowballing search, which is searching, starting from a set of identified key-papers, the publications cited in the key papers as well as the publications citing the key-paper. This search strategy identifies related articles using citation relationships (not keywords) building a citation network focused on the topic of the key-papers  Here we will discuss how a snowballing searc hcan be achieved with code examples and mention some shortcomings and advantages of this approach. Furthermore, we will outline some analysis approaches and possibilities of a citation network without going into too much detail 
keyword: "literature search, snowballing, citation network"
license: "CC BY"
copyright: 
  holder: No idea
  year: 2023
citation: 
  type: report
  container-title: IPBES Technical Guidelines on Snowballing
  doi: xxxxx
doi: xxxxx
version: 0.0.2

format:
    html:
        toc: true
        toc-depth: 4
        toc_expand: true
        embed-resources: true
        code-fold: false
        code-summary: 'Show the code'
        keep-md: true
---






Prepared by 
- [Rainer M. Krug](mailto:Rainer.Krug@Senckenberg.de) - IPBES Task Force on Knowledge and Data. 

Reviewed by 
- [Aidin Niamir](mailto:Aidin.Niamir@Senckenberg.de) - IPBES Technical Support Unit for Knowledge and Data
- [Yanina Sica](mailto:Yanina.Sica@Senckenberg.de) - IPBES Technical Support Unit for Knowledge and Data

<!-- - IPBES Task Force on Knowledge and Data**  -->

*For any inquires please contact [Aidin.Niamir@senckenberg.de](mailto:aidin.niamir@senckenberg.de)* 

**Version:** 0.0.2

Last updated: 04 December, 2023


## Introduction

Snowballing in literature search and analysis refers to the approach to search for literature starting from a set of identified key-papers. These key-papers should cover the range of the topic which should be covered by the literature search, while containing  the most relevant papers of the topic. Starting from these key-papers, the papers which are cited in the key-papers (also called backward search) as well as the papers which are citing the key-papers (also called forward search) are identified.

Consequently, the selection of the key-papers is essential in determining the resulting coverage of the topic by the literature identified by the snowball search.


## Methods

### Databases suitable for Snowballing
In principle, snowballing can be done with most (if not all) literature databases. Examples are Web of Science,Scopus or PubMed which provide the possibility to search for papers from their web interface or using API services (e.g. https://dev.elsevier.com/sc_apis.html). When using their web interface, one can only search  for one article at the time, so it is very cumbersome. Using  the provided APIs makes  automating the process possible, but becomes easily extremely expensive and still laden with restrictions.
A good alternative for this type of search strategy is [OpenAlex](https://openalex.org), which has free and unlimited API access for all practical purposes. Therefore, the Snowball search can be scripted, and API clients are available for many different programming languages.
This technical guideline will focus particularly on the use of [OpenAlex](https://openalex.org) (the by the data tsu recommended data source for literature) through the R package [openalexR](https://docs.ropensci.org/openalexR/articles/A_Brief_Introduction_to_openalexR.html) which is perfectly suited for this task, in addition to being easy to use.

### Snowballing in R

#### Installation and loading of the package
The package [openalexR](https://docs.ropensci.org/openalexR/articles/A_Brief_Introduction_to_openalexR.html) 
is available from CRAN and can be installed using


::: {.cell}

```{.r .cell-code}
install.packages(openalexR)
```
:::


and loaded using


::: {.cell}

```{.r .cell-code}
library(openalexR)
```

::: {.cell-output .cell-output-stderr}

```
Thank you for using openalexR!
To acknowledge our work, please cite the package by calling `citation("openalexR")`.
To suppress this message, add `openalexR.message = suppressed` to your .Renviron file.
```


:::
:::


#### Snowballing
Let's assume, we have a list of DOIs of the key papers, which have been identified by 
experts with knowledge of the field and topic.


::: {.cell}

```{.r .cell-code}
dois <- c(
    "10.1016/j.marpol.2023.105710", "10.1007/s13280-014-0582-z",
    "10.1016/j.cosust.2022.101160", "10.1146/annurev-environ-102014-021340",
    "10.1016/j.eist.2016.09.001", "10.1016/j.cosust.2019.12.004"
)
```
:::


As the snowball search in [OpenAlex](https://openalex.org) is based on internal identifiers (OpenAlex ids), the DOIs need to be converted to these internal ids. This can be done using the function `oa_fetch()` of the package [openalexR](https://docs.ropensci.org/openalexR/articles/A_Brief_Introduction_to_openalexR.html) which can takes a list of DOIs and returns a list of works ([Works](https://docs.openalex.org/api-entities/works) are documents in [OpenAlex](https://openalex.org).) as provided by OpenAlex. The function has many other arguments, but I will show here on the use for this specific example. For more information, please refer to the help in R or the [documentation](https://docs.ropensci.org/openalexR/reference/index.html).


::: {.cell}

```{.r .cell-code}
key_works <- oa_fetch(
    entity = "works",
    doi = dois,
    verbose = FALSE
)

ids <- openalexR:::shorten_oaid(key_works$id)
```
:::


The resulting list, has two elements: 
    1. **`nodes`** which contains the individual publications, i.e., the works as returned by OpenAlex. These are the **nodes** of the citation network
    2. **`edges`** which contains the citations between the works or edges of the citation network, which form the **edges** of the citation network.

The edges only contain the citations from and to the key-papers, so the citation network does not include any citations between the newly identified works. We will come back to this later.
Another useful function is `snowball2df()` which flattens the object returned by `oa_snowball()` into a `tibble`. It is usually much easier to work with this flat structure


::: {.cell}

```{.r .cell-code}
fn <- file.path("data", "snowball.rds")
if (file.exists(fn)) {
    snowball <- readRDS(fn)
} else {
    snowball <- oa_snowball(
        identifier = ids,
        verbose = FALSE
    )
    saveRDS(snowball, fn)
}
```
:::


The resulting list, has two elements:
- **`nodes`** which contains the nodes of the citation network, i.e. the individual publications (or `works` in the terminology of OpenAlex) and
- **`edges`** which contains the edges of the citation network, i.e. the citations between the works.

The edges only contain the citations from and to the key papers, so the citation network does not include any citations between the newly identified works. We will come back to this later.

Another useful function is `snowball2df()` which flattens the object returned by `oa_snowball()` into a tibble.
It is usually much easier to work with this flat structure

::: {.cell}

```{.r .cell-code}
flat_snow <- snowball2df(snowball) |>
    tibble::as_tibble()
```
:::


#### Plotting the network
There ara many different packages in R to plot networks ([igraph](https://r.igraph.org), [ggraph](https://ggraph.data-imaginist.com) together with [tidygraph](https://tidygraph.data-imaginist.com) and others).

For now, I will only show how to plot the network using `ggraph` and `tidygraph` and not go into details how the plot can be tweaked.



::: {.cell}

```{.r .cell-code}
library(tidygraph)
```

::: {.cell-output .cell-output-stderr}

```

Attaching package: 'tidygraph'
```


:::

::: {.cell-output .cell-output-stderr}

```
The following object is masked from 'package:stats':

    filter
```


:::

```{.r .cell-code}
library(ggraph)
```

::: {.cell-output .cell-output-stderr}

```
Loading required package: ggplot2
```


:::

```{.r .cell-code}
fn <- file.path("figures", "cited_by_count.png")
if (!file.exists(fn)) {
 p_cb <- ggraph::ggraph(tidygraph::as_tbl_graph(snowball), 
        graph = , layout = "stress") + ggraph::geom_edge_link(ggplot2::aes(alpha = ggplot2::after_stat(index)), 
        show.legend = FALSE) + ggraph::geom_node_point(ggplot2::aes(fill = oa_input, 
        size = cited_by_count), shape = 21, color = "white") + 
        ggraph::geom_node_label(ggplot2::aes(filter = oa_input, 
            label = id), nudge_y = 0.2, size = 3) + ggraph::scale_edge_width(range = c(0.1, 
        1.5), guide = "none") + ggplot2::scale_size(range = c(3, 
        10), guide = "none") + ggplot2::scale_fill_manual(values = c("#a3ad62", 
        "#d46780"), na.value = "grey", name = "") + ggraph::theme_graph() + 
        ggplot2::theme(plot.background = ggplot2::element_rect(fill = "transparent", 
            colour = NA), panel.background = ggplot2::element_rect(fill = "transparent", 
            colour = NA), legend.position = "bottom") + ggplot2::guides(fill = "none") + 
        ggplot2::ggtitle(paste0("Cited by count"))
    ggplot2::ggsave(file.path("figures", "cited_by_count.pdf"), 
        plot = p_cb, device = cairo_pdf, width = 20, height = 15)
    ggplot2::ggsave(file.path("figures", "cited_by_count.png"), 
        plot = p_cb, width = 20, height = 15, bg = "white", dpi = 600)
}
```
:::

![](figures/cited_by_count.png)

There are many options on how these plots can be tweaked, but this is beyond the 
scope of this technical guideline to go into details. Please see the 
documentation of [ggraph](https://ggraph.data-imaginist.com) and 
[tidygraph](https://tidygraph.data-imaginist.com) the two packages for more information.

#### Supplementing the citation network

As mentioned above, the citation network does not include any links between the newly 
identified works, as these are included only through citing or cited by the key papers. This can be 
easily be changed, as the works returned from OpenAlex contain a field which lists all the ids 
of the works which are citing the work. This field is called `referenced_works` and can be used to 
supplement the citation network. 
We need to filter these works to only include the works
in the original citation network, and add the new edges to the `snowball$edges` object:


::: {.cell}

```{.r .cell-code}
library(tibble)

fn <- file.path("data", "snowball_supplemented.rds")
if (file.exists(fn)) {
    snowball_supplemented <- readRDS(fn)
} else {
    new_edges <- tibble(
        from = character(0),
        to = character(0)
    )

    ## all the works in the citation network
    works <- snowball$nodes$id

    ## check if the works in the `referenced_works` field are in the citation network
    ## and if yes, add them to new_edges 
    for (i in 1:nrow(snowball$nodes)) {
        from <- works[[i]]
        to <- gsub("https://openalex.org/", "", snowball$nodes$referenced_works[[i]])
        to_in_works <- to[to %in% works]
        if (length(to_in_works) > 0) {
            new_edges <- add_row(
                new_edges,
                tibble(
                    from = from,
                    to = to_in_works
                )
            )
        }
    }

    ## add the new edges to the citation network but only the unique ones
    snowball_supplemented <- snowball
    snowball_supplemented$edges <- add_row(snowball_supplemented$edges, new_edges) |>
        distinct()

    saveRDS(snowball_supplemented, fn)
}
```
:::


Now we can plot the supplemented citation network:


::: {.cell}

```{.r .cell-code}
fn <- file.path("figures", "supplemented_cited_by_count.png")
if (!file.exists(fn)) {
    p_cb <- ggraph::ggraph(tidygraph::as_tbl_graph(snowball_supplemented),
        graph = , layout = "stress"
    ) + ggraph::geom_edge_link(ggplot2::aes(alpha = ggplot2::after_stat(index)),
        show.legend = FALSE
    ) + ggraph::geom_node_point(ggplot2::aes(
        fill = oa_input,
        size = cited_by_count
    ), shape = 21, color = "white") +
        ggraph::geom_node_label(ggplot2::aes(
            filter = oa_input,
            label = id
        ), nudge_y = 0.2, size = 3) + ggraph::scale_edge_width(range = c(
            0.1,
            1.5
        ), guide = "none") + ggplot2::scale_size(range = c(
            3,
            10
        ), guide = "none") + ggplot2::scale_fill_manual(values = c(
            "#a3ad62",
            "#d46780"
        ), na.value = "grey", name = "") + ggraph::theme_graph() +
        ggplot2::theme(plot.background = ggplot2::element_rect(
            fill = "transparent",
            colour = NA
        ), panel.background = ggplot2::element_rect(
            fill = "transparent",
            colour = NA
        ), legend.position = "bottom") + ggplot2::guides(fill = "none") +
        ggplot2::ggtitle(paste0("Cited by count"))
    ggplot2::ggsave(file.path("figures", "supplemented_cited_by_count.pdf"),
        plot = p_cb, device = cairo_pdf, width = 20, height = 15
    )
    ggplot2::ggsave(file.path("figures", "supplemented_cited_by_count.png"),
        plot = p_cb, width = 20, height = 15, bg = "white", dpi = 600
    )
}
```
:::

![](figures/supplemented_cited_by_count.png)

As one can easily see, the number of citations (edges) is much higher, 
and the network is much more connected. A visual inspection is therefore not 
possible anymore, and one needs to use other methods to analyse the network. 
One possiblility is to identify clusters in the citation network.

## Other things to do with snowball search
Aspect which are not covered in this technical guidaline, but can be done with the results 
from the snowball seaarching are:

- Interactive graphs from the citation network using for example the package `networkD3`
- Clustering of the citation network using for example the package `igraph` to identify topical clusters, citation clusters, etc.
- Identify papers linking identified clsters 
- etc.


## Things to keep in mind when using snowballing for literature search

This is a (non complete) list of points one needs to consider when using snowballing for literature search. Not all points appl;y to all questions, and some are more relevant then others. They are listed in no particular order.

1. The selection of the key papers is essential for the resulting literature search. The key papers should cover the range of the topic which should be covered by the literature search, while containing  the most relevant papers of the topic.
2. The snowball search is based on citation relationships and not on keywords. This means, that the resulting literature search is based on the citation network of the key papers and not on the keywords of the papers. In other words, papers which are not cited by the key papers will not be included in the resulting literature search, even if they are highly relevant for the topic. 
3. The snowball covers (given a solid choice of key papers) the most relevant literature for the topic, but it does not discover topic areas which are relevant but not linked to the key-papers by citations. In other words, if a topic consists of two research areas which are not citing each other, keypapers need to be provided for both.
4. Snowball search contains less false positives (works not related to the topic) than a keyword based search, while a snowball search is less likely to find all works related.
5. Automated snowball search is limited to [OpenAlex](https://openalex.org) at the moment, which is a database of mainly scholary literature. This means, that grey literature is underrepresented in the snowball results. Neertheless, it is possible to do it manually, but the effort is much higher.
6 Snowball search is a very efficient (time saving) way to obtain a list of relevant literature of a topic, as keyword search usually requires rather complex search terms and numerous iterations to get down to a managable number of identified works.



<!-- 

#### Identifying clusters in the citation network

One approach to identify clusters in the citation network is to use the edge.betweenness
which is a measure which ??????.

This clustering is implemented in the [igraph](https://r.igraph.org) package.

HERE I NEED IDEAS!!!!


::: {.cell}

```{.r .cell-code}
library(igraph)

g <- igraph::graph_from_data_frame(
  d = snowball$edges, 
  # vertices = snowball$nodes,
  directed = TRUE
)
eb <- cluster_edge_betweenness(g)

layout <- layout.fruchterman.reingold(g)
plot(
  eb, g,
  layout = layout
)
```
:::


#### Interactive network chart
Interactive network graphs are also possible, and mainly useful for smaller networks and for exploring the network.


::: {.cell}

```{.r .cell-code}
library(networkD3)

p <- simpleNetwork(
  Data = snowball$edges,
  height="50px", width="50px",
  zoom = TRUE
)
p
```

::: {.cell-output-display}


```{=html}
<div class="forceNetwork html-widget html-fill-item-overflow-hidden html-fill-item" id="htmlwidget-a7bf6b43e9403f1c8f29" style="width:100%;height:464px;"></div>
<script type="application/json" data-for="htmlwidget-a7bf6b43e9403f1c8f29">{"x":{"links":{"source":[424,228,264,367,367,604,476,476,476,544,544,343,284,318,525,351,351,808,463,704,469,483,483,1083,713,713,272,399,658,658,474,474,278,363,858,230,371,702,587,286,811,270,653,391,419,417,282,303,520,271,440,440,329,663,311,311,311,736,425,352,578,458,262,237,277,334,724,836,618,618,618,618,332,309,281,373,441,594,388,304,661,914,639,536,754,753,225,386,460,398,561,561,287,790,855,855,445,629,629,336,507,626,374,656,250,925,344,312,861,861,861,861,971,1112,413,1166,1166,235,366,366,750,879,435,444,714,416,338,524,570,885,885,375,677,677,677,691,691,910,379,379,464,487,427,518,688,959,1502,307,326,226,479,648,341,405,1131,473,296,761,882,913,323,538,652,652,652,794,936,423,369,569,569,657,733,830,830,830,962,1256,346,437,530,623,796,852,274,387,519,519,571,606,616,893,1010,1010,350,401,431,446,620,682,697,839,847,943,1001,1001,488,503,511,568,665,665,890,1003,333,360,400,468,727,765,454,454,509,693,693,693,939,1087,321,438,528,566,642,660,780,824,1141,1182,1222,1222,330,410,555,609,627,867,867,907,1086,1235,283,308,349,619,803,829,917,1330,354,368,504,554,559,721,739,772,1119,1308,1308,1308,306,335,383,452,514,584,636,863,873,924,924,968,968,1145,449,449,490,601,645,749,786,1117,1129,1180,1180,392,471,647,692,845,1170,1170,339,356,432,632,751,860,1007,1008,1008,1079,289,358,358,377,377,396,456,459,567,574,731,776,792,802,1140,302,372,421,439,439,482,806,837,849,878,878,902,957,1063,1063,1125,1125,1250,1281,1281,1327,394,409,475,585,591,624,650,774,798,827,848,1004,1004,1004,1077,1184,328,376,385,498,592,621,628,804,857,904,992,1092,1226,256,288,359,359,404,521,593,617,785,805,823,844,954,1064,1196,1262,1418,251,260,320,320,397,433,433,523,535,597,611,715,717,784,784,813,826,875,876,949,949,961,961,965,1102,266,291,295,310,315,403,470,502,506,562,686,705,711,769,783,795,795,835,835,841,898,948,1133,1135,1148,1315,1382,1579,249,407,484,491,573,579,613,633,1042,1042,1084,1107,1219,1314,1316,327,390,390,485,565,625,638,638,734,752,818,818,868,868,918,918,982,1072,1101,1136,1204,1204,1204,1204,1307,1307,1307,1437,275,381,382,466,556,598,655,664,722,906,921,946,974,996,1115,1115,1115,1209,1266,1266,1375,1375,1404,293,300,305,337,501,513,513,563,581,614,622,675,678,698,707,728,728,728,767,833,833,846,846,892,899,927,935,958,963,990,997,997,997,1032,1142,1142,1142,1190,1201,1336,1341,1359,1373,1380,1388,1388,1388,1447,1469,313,467,481,510,542,560,641,641,668,671,671,716,725,740,782,799,819,870,915,960,986,1022,1058,1058,1071,1074,1121,1130,1188,1223,1223,1228,1228,1317,319,393,408,415,429,443,465,477,605,607,695,701,771,775,775,793,822,869,869,886,886,887,889,895,900,901,933,933,933,944,944,944,951,999,1006,1089,1099,1124,1124,1152,1193,1193,1193,1340,1369,1400,1400,1439,1453,1464,1464,1505,451,517,547,547,615,646,666,666,669,738,743,747,747,755,755,778,791,828,920,931,973,980,984,1048,1048,1068,1103,1113,1113,1116,1120,1128,1200,1200,1206,1212,1217,1236,1239,1284,1342,1344,1367,1376,1434,1521,1765,345,420,428,494,537,537,553,553,600,635,651,651,699,718,719,723,768,773,773,781,815,815,815,880,883,888,903,909,991,1002,1036,1036,1094,1137,1138,1181,1191,1197,1199,1238,1244,1246,1247,1249,1279,1279,1279,1319,1319,1338,1364,1364,1396,1398,1457,1498,1514,1553,1725,384,412,412,412,500,603,612,709,712,726,726,745,746,779,816,820,854,862,865,874,897,911,916,919,952,953,976,985,995,1016,1023,1023,1026,1026,1054,1061,1061,1080,1106,1122,1146,1195,1215,1272,1277,1277,1277,1305,1305,1305,1310,1322,1354,1356,1356,1424,1452,1455,1459,1465,1511,1511,1513,1515,1515,1565,414,564,564,706,708,744,744,789,789,809,809,809,853,872,896,923,926,941,945,994,1000,1013,1013,1027,1027,1030,1030,1066,1066,1108,1108,1108,1108,1134,1186,1192,1198,1202,1214,1220,1220,1225,1225,1242,1261,1299,1312,1323,1324,1329,1333,1333,1335,1358,1362,1366,1372,1374,1383,1397,1401,1402,1420,1428,1449,1450,1450,1473,1499,1518,1591,1601,1724,267,331,395,436,492,492,495,508,516,516,533,540,540,551,551,552,552,557,590,599,631,670,683,689,690,690,690,737,757,757,817,866,905,912,938,970,979,979,981,987,1062,1076,1078,1085,1090,1100,1105,1105,1123,1139,1227,1229,1240,1240,1240,1254,1255,1258,1264,1264,1267,1269,1297,1300,1301,1309,1318,1371,1390,1393,1393,1394,1394,1403,1405,1407,1411,1425,1425,1429,1445,1445,1445,1468,1478,1478,1492,1493,1493,1509,1527,1528,1540,1547,1557,1566,1583,1598,1600,1623,1652,1681,1726,355,430,448,461,478,493,493,532,532,572,576,589,589,595,634,640,643,654,673,673,673,684,694,710,742,760,787,801,908,930,947,955,967,977,993,1005,1020,1039,1039,1045,1045,1075,1081,1088,1091,1097,1132,1143,1147,1158,1162,1189,1189,1205,1207,1211,1224,1230,1231,1231,1237,1253,1260,1295,1296,1304,1306,1321,1325,1325,1332,1353,1387,1391,1399,1409,1431,1432,1433,1435,1436,1436,1441,1446,1448,1461,1461,1474,1477,1477,1487,1501,1507,1510,1516,1519,1520,1533,1541,1549,1552,1554,1555,1559,1559,1568,1573,1580,1584,1586,1617,1617,1617,1617,1618,1642,1646,1659,1672,1696,299,411,418,455,457,457,505,512,526,545,545,546,546,549,549,550,550,580,588,596,608,608,667,674,680,687,696,696,703,812,814,832,843,843,850,864,891,928,940,966,978,988,1025,1025,1033,1050,1050,1067,1070,1070,1073,1082,1082,1095,1096,1110,1110,1114,1127,1144,1144,1151,1151,1154,1155,1157,1157,1163,1169,1179,1185,1194,1203,1208,1210,1213,1221,1232,1243,1245,1248,1252,1257,1263,1268,1283,1298,1303,1311,1313,1320,1326,1328,1331,1337,1343,1345,1345,1345,1351,1361,1363,1368,1378,1378,1378,1379,1381,1384,1389,1395,1410,1410,1413,1415,1421,1430,1442,1444,1454,1460,1462,1475,1476,1479,1482,1485,1490,1490,1495,1496,1496,1508,1517,1522,1523,1529,1530,1531,1535,1536,1539,1539,1542,1546,1546,1548,1556,1561,1562,1567,1569,1570,1572,1572,1572,1572,1572,1596,1596,1597,1602,1602,1603,1603,1606,1610,1615,1620,1627,1628,1629,1629,1632,1650,1660,1666,1666,1669,1669,1685,1694,1698,1710,1721,1744,1766,298,316,317,317,364,389,422,434,442,462,480,480,486,499,531,539,539,543,543,548,548,558,575,577,583,610,610,637,649,659,672,679,681,685,700,700,720,729,730,730,732,735,741,748,759,766,788,788,797,800,807,810,821,825,825,831,831,834,838,838,840,842,851,859,871,877,884,894,894,922,929,932,934,937,942,950,956,964,969,969,972,975,983,989,998,1009,1009,1011,1011,1012,1012,1014,1014,1015,1015,1017,1017,1018,1018,1019,1019,1021,1021,1024,1024,1028,1028,1029,1029,1031,1031,1034,1034,1035,1035,1037,1037,1038,1038,1040,1040,1041,1041,1043,1043,1044,1044,1046,1046,1047,1047,1049,1049,1051,1051,1052,1052,1053,1053,1055,1055,1056,1056,1057,1057,1059,1060,1060,1065,1065,1069,1069,1093,1098,1104,1109,1111,1118,1149,1150,1153,1153,1156,1156,1159,1159,1160,1161,1164,1164,1165,1165,1167,1168,1171,1172,1173,1174,1175,1175,1176,1177,1178,1178,1178,1183,1187,1216,1218,1233,1234,1241,1251,1259,1265,1270,1270,1271,1273,1274,1275,1282,1282,1285,1285,1286,1287,1289,1290,1291,1292,1294,1302,1334,1339,1346,1347,1348,1349,1350,1352,1355,1360,1365,1370,1385,1386,1392,1406,1408,1412,1414,1423,1426,1427,1427,1438,1440,1443,1451,1456,1458,1463,1466,1467,1470,1471,1472,1472,1480,1481,1483,1484,1486,1488,1489,1491,1494,1497,1500,1503,1504,1506,1512,1524,1525,1525,1526,1532,1532,1534,1537,1538,1543,1544,1544,1545,1550,1551,1551,1558,1558,1560,1560,1563,1564,1571,1574,1575,1576,1577,1577,1578,1581,1582,1585,1587,1588,1589,1590,1592,1592,1593,1593,1593,1594,1595,1599,1604,1605,1607,1607,1607,1608,1609,1611,1612,1613,1614,1616,1619,1621,1622,1622,1624,1625,1626,1630,1631,1631,1633,1634,1635,1635,1636,1637,1638,1639,1640,1641,1643,1644,1645,1647,1648,1648,1649,1651,1653,1654,1655,1656,1656,1656,1657,1658,1661,1662,1663,1663,1664,1665,1667,1668,1670,1671,1673,1674,1674,1675,1676,1677,1678,1679,1680,1682,1683,1683,1684,1686,1687,1688,1689,1690,1691,1692,1693,1695,1697,1699,1700,1701,1702,1703,1704,1704,1704,1704,1705,1706,1707,1708,1708,1709,1711,1712,1713,1714,1715,1716,1717,1718,1719,1719,1720,1722,1723,1727,1728,1729,1730,1731,1732,1733,1734,1735,1736,1737,1738,1739,1740,1741,1742,1743,1745,1745,1746,1747,1748,1749,1750,1751,1752,1753,1754,1755,1756,1757,1758,1759,1759,1759,1760,1760,1761,1762,1763,1764,1767,1768,1769,1770,1771,1771,1772,1772,1772,1773,1774,1775,1776,1776,1777,1778,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,228,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,264,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,72,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,544,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1204,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622,1622],"target":[228,264,72,72,264,228,72,228,264,72,264,72,264,72,228,72,228,228,228,544,228,72,264,228,72,544,72,228,228,544,72,228,72,228,228,72,264,544,228,72,228,72,228,228,228,72,72,228,228,72,72,228,72,72,72,228,264,228,228,228,264,72,72,72,264,72,264,228,72,228,264,544,264,264,72,72,228,72,72,264,228,264,72,228,544,228,72,228,228,228,228,264,72,228,264,544,264,228,264,264,72,544,72,228,72,544,264,72,72,228,264,544,544,544,228,72,264,72,228,264,72,544,228,228,228,264,264,228,228,72,264,72,72,228,264,72,264,72,72,228,72,228,264,72,228,544,264,72,72,72,264,264,72,228,544,228,228,72,264,544,264,228,72,228,264,264,228,264,228,72,264,264,228,72,264,544,228,264,264,72,228,228,264,228,72,264,72,228,72,264,544,264,228,264,264,228,72,264,72,72,72,264,264,228,72,264,228,228,72,544,72,264,544,264,228,72,72,72,544,72,72,264,72,72,264,544,72,544,264,72,228,228,228,264,228,228,228,228,72,264,264,72,228,228,228,264,544,544,544,544,264,72,228,228,228,544,72,228,228,264,264,228,264,228,72,544,264,228,264,544,72,72,72,228,72,544,228,544,72,264,544,72,544,72,72,228,72,228,544,264,72,228,544,228,264,72,72,72,544,72,228,264,72,72,228,228,228,228,544,72,544,544,72,228,264,72,228,228,264,264,228,228,72,228,544,264,228,72,72,264,72,264,228,264,264,544,228,264,544,72,228,264,72,264,228,228,264,228,72,228,228,72,228,72,228,228,228,544,228,72,228,264,264,228,72,228,264,228,544,264,228,544,72,228,228,264,544,72,72,72,228,72,264,228,264,72,72,72,228,544,264,228,544,544,72,72,72,264,228,228,264,228,228,72,228,72,544,72,264,72,228,72,72,228,264,72,264,72,544,72,72,72,264,264,228,72,264,228,228,228,544,228,228,228,228,264,264,544,228,544,72,264,72,544,228,264,228,72,228,228,72,264,264,228,264,72,264,228,228,72,72,228,72,72,228,72,228,228,72,264,228,228,228,544,228,264,72,264,72,228,264,72,72,228,264,544,72,264,544,264,72,72,228,228,228,264,228,228,544,228,228,544,228,72,228,264,544,544,264,544,72,544,544,72,72,72,264,264,228,264,264,228,264,72,228,228,228,264,228,264,544,264,72,544,228,544,264,544,228,264,228,544,228,72,228,544,228,72,228,264,264,264,228,228,544,72,228,72,264,544,228,72,264,228,264,72,228,228,264,544,228,228,264,264,72,72,228,228,264,228,72,72,264,228,228,264,228,228,228,228,544,228,264,72,228,264,264,228,264,264,228,72,228,264,72,228,228,228,228,264,544,228,228,264,544,72,264,72,544,228,544,264,72,264,544,72,264,544,72,264,264,72,228,72,228,72,72,264,544,228,544,264,544,544,264,72,264,264,228,228,228,264,544,72,228,264,72,228,72,228,264,228,264,72,72,264,264,228,72,228,228,228,264,264,72,72,264,544,228,72,72,544,264,228,544,228,228,264,228,228,228,228,264,72,544,264,264,228,72,228,264,228,264,544,264,72,264,72,544,228,228,228,228,264,228,72,264,544,228,228,264,228,264,264,72,228,264,228,264,72,228,544,264,228,228,544,264,228,72,72,228,264,228,264,228,72,264,228,72,228,228,228,264,264,264,72,228,264,228,264,264,228,72,72,264,228,264,264,228,228,72,72,228,228,228,228,264,264,264,544,264,72,264,228,228,264,228,264,228,228,264,228,264,228,264,72,228,264,72,228,264,228,264,544,264,228,264,72,544,228,228,72,228,72,72,228,544,72,544,544,72,228,264,72,228,72,264,264,544,72,264,544,72,264,264,228,544,228,228,264,264,228,264,228,264,228,264,228,264,72,228,264,544,264,544,228,72,228,228,228,544,264,544,228,228,544,228,228,228,228,72,228,544,544,72,228,264,264,228,264,72,228,72,228,544,228,264,264,228,544,544,228,544,72,228,72,228,72,264,228,228,228,264,228,228,264,228,264,228,264,228,228,264,228,228,228,72,72,228,264,228,228,264,228,72,228,544,72,228,72,264,72,228,264,72,72,228,228,264,228,544,72,72,228,228,72,264,544,544,228,264,228,264,544,264,228,72,228,228,228,228,264,264,544,544,1204,264,72,228,544,72,264,228,72,264,544,264,264,544,228,72,264,228,544,228,544,264,228,228,228,228,228,228,228,544,264,264,264,228,228,264,72,264,228,264,264,264,72,228,228,264,228,264,264,72,228,264,72,228,228,228,544,228,228,72,264,72,228,264,228,228,228,544,228,264,72,264,228,264,544,228,544,264,228,544,72,72,72,544,264,72,72,544,544,72,544,228,228,544,264,228,544,264,264,72,264,72,72,228,228,72,228,544,544,544,228,72,264,228,228,72,72,264,264,72,544,228,228,544,228,72,264,72,544,544,228,228,264,228,544,1204,228,544,228,228,72,72,228,264,544,228,544,544,264,264,544,264,228,72,264,72,264,228,228,264,228,264,228,264,228,264,228,264,228,264,264,228,264,228,228,228,228,72,264,228,228,264,228,228,264,264,228,72,264,228,228,228,72,228,264,544,228,264,228,228,264,228,72,264,228,544,228,544,264,228,72,264,228,264,72,72,72,264,72,72,228,72,264,228,544,72,72,228,264,228,264,228,228,264,72,228,228,72,264,544,228,228,544,264,264,544,264,72,264,544,264,544,544,72,72,264,544,228,228,544,228,1204,264,544,264,264,228,544,544,72,228,544,228,264,228,264,264,228,228,1204,72,264,544,264,228,228,544,228,264,228,264,264,72,228,264,264,544,228,228,228,228,228,544,544,72,228,264,544,1204,264,544,72,72,264,264,544,544,228,72,544,228,228,544,1204,228,228,264,264,544,72,264,228,228,1204,264,544,228,228,72,72,72,264,228,72,228,72,72,72,228,264,264,72,264,228,264,228,264,228,264,264,228,264,72,228,264,228,228,264,228,72,228,72,228,544,264,228,228,264,228,228,264,228,228,72,72,264,72,264,228,228,228,72,264,72,264,228,72,264,228,72,228,264,228,72,72,72,264,228,228,228,228,264,228,72,72,228,72,544,544,264,544,228,228,228,264,228,264,228,264,228,264,228,264,228,264,228,264,228,264,228,264,228,264,228,264,228,264,228,264,228,264,228,264,228,264,228,264,228,264,228,264,228,264,228,264,228,264,228,264,228,264,228,264,228,264,228,264,228,264,228,264,228,264,72,228,264,228,264,228,264,264,72,72,228,228,72,228,72,228,264,228,264,72,264,72,72,72,264,72,264,72,72,228,72,72,72,72,264,544,228,72,228,264,544,228,228,264,264,228,228,228,228,228,228,264,228,228,72,228,228,264,72,264,72,228,72,228,544,72,264,72,544,264,228,72,72,544,228,228,228,228,228,228,264,228,544,228,228,228,72,544,544,228,544,544,228,228,544,72,228,72,228,72,544,544,544,1204,228,264,228,228,228,264,228,264,72,544,264,228,544,228,228,264,264,544,228,72,264,264,72,228,544,264,544,544,544,72,228,264,544,72,544,264,228,228,264,228,264,228,264,228,72,72,544,264,1204,264,264,264,544,228,264,544,72,72,264,228,228,72,264,544,228,544,544,72,228,264,72,264,228,72,264,228,228,72,544,228,264,228,544,228,544,544,544,264,264,264,264,228,228,1204,72,228,544,228,72,228,228,228,72,264,544,228,544,228,72,72,264,228,544,72,228,228,264,228,544,1204,264,72,544,228,544,264,228,228,544,264,544,264,228,228,264,264,228,228,72,544,264,228,228,228,72,72,264,544,1204,72,544,264,72,228,544,228,228,228,544,228,228,228,228,264,544,228,228,228,228,228,228,544,264,228,72,228,228,544,72,72,264,544,228,264,228,264,544,264,228,544,228,264,228,544,228,264,72,228,264,228,72,264,544,264,544,544,264,264,264,228,264,228,228,264,544,72,228,264,228,228,544,264,544,228,544,4,5,6,7,8,10,12,13,18,22,23,24,27,31,32,35,38,41,43,46,47,48,50,52,55,57,58,59,60,61,65,68,71,73,74,75,76,77,78,81,87,88,89,90,92,94,96,97,98,99,103,107,109,110,111,112,114,116,121,122,123,124,125,126,128,131,132,134,137,140,144,148,150,152,153,154,155,157,159,163,174,176,177,179,180,182,190,191,195,196,197,198,202,204,205,206,207,208,209,210,212,217,219,220,233,234,236,240,243,244,252,255,261,268,285,347,644,1280,1293,1377,1417,1422,1,14,15,21,30,31,33,34,35,37,39,40,44,53,58,63,64,66,83,84,86,88,91,93,100,101,103,104,105,106,113,115,118,129,131,133,136,141,146,149,152,156,160,163,165,167,168,169,171,178,181,187,189,190,192,199,200,201,205,207,212,213,216,221,231,232,239,240,241,242,245,269,273,1278,1779,2,3,16,17,20,21,25,26,28,29,36,38,42,44,45,49,53,54,56,60,67,70,79,85,102,108,113,117,120,126,127,130,142,145,151,158,164,168,169,170,172,175,178,183,185,188,207,211,214,215,218,223,232,257,259,273,602,758,764,1288,1416,1417,1419,31,51,60,62,65,66,69,81,88,95,103,113,119,128,129,131,135,138,139,143,147,161,162,166,168,169,173,178,184,186,192,193,194,203,222,224,240,248,253,276,279,280,290,294,297,301,314,322,324,325,329,340,353,357,361,365,367,370,378,402,406,447,489,9,11,19,51,65,82,83,128,168,169,229,232,240,246,258,271,329,342,343,351,361,367,380,402,424,440,450,472,474,476,515,529,534,582,586,629,630,704,762,763,777,809,819,830,835,856,863,875,881,890,944,946,1276,0,19,49,80,93,113,133,151,168,169,175,200,207,227,230,232,238,241,247,254,261,263,265,269,278,292,329,343,348,351,362,367,426,453,474,496,497,522,527,541,662,676,684,756,765,770,777,955,1126,1256,1305,1306,1357],"value":[1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1],"colour":["#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666","#666"]},"nodes":{"name":["W1478311351","W1498328630","W1503263161","W1505700478","W1523351240","W1545656083","W1560160683","W1593600276","W1600877975","W1672862333","W1748274567","W1749602744","W1764489390","W1786105803","W1788156339","W1890955996","W1927533673","W1954417695","W1964181949","W1964510407","W1964959681","W1968852409","W1968942643","W1970091959","W1970551484","W1973131774","W1973881518","W1974564128","W1974675643","W1974918392","W1975949752","W1976759885","W1976775578","W1978035589","W1978252907","W1979877174","W1981988708","W1982008313","W1982748232","W1984286412","W1985401316","W1985766189","W1989697355","W1990201348","W1990722542","W1991922076","W1993152396","W1994744422","W1996900848","W1998431836","W1998907019","W1999167944","W1999997471","W2000485484","W2001994011","W2002311592","W2002526147","W2003112502","W2007762019","W2009040832","W2009175240","W2009202666","W2009652527","W2010253870","W2010941927","W2011303397","W2011880877","W2014075123","W2015857016","W2016092462","W2016532953","W2017477218","W2020319093","W2020454475","W2022035466","W2024712186","W2025352501","W2025675057","W2025784700","W2026457385","W2028260379","W2030655414","W2034994370","W2037833995","W2040994515","W2046499434","W2047939329","W2048198984","W2048226523","W2049390389","W2049501389","W2050599067","W2051438597","W2052509076","W2053268102","W2053510621","W2057290175","W2058190339","W2058869237","W2059527695","W2060587030","W2060855669","W2061793514","W2067441067","W2067525094","W2068061833","W2070028837","W2070488366","W2070553586","W2072430631","W2075251296","W2076734701","W2077736798","W2078778096","W2079824627","W2081995659","W2082597100","W2082865867","W2082901040","W2084530896","W2087341478","W2088780071","W2089815742","W2090037659","W2092756265","W2093623622","W2094701299","W2095756714","W2096885696","W2097137610","W2098984318","W2099330597","W2099706748","W2099776967","W2100324077","W2101092961","W2101788157","W2103166694","W2103435246","W2104460807","W2105083736","W2105414963","W2105877423","W2106815820","W2107250248","W2108711327","W2110512393","W2113316348","W2113991966","W2115027623","W2115314385","W2116805608","W2117275101","W2117401019","W2117794600","W2119144351","W2119484624","W2119619016","W2120351977","W2120767947","W2121387700","W2122325078","W2122925767","W2122963205","W2124512119","W2124611226","W2125512042","W2126498825","W2127569725","W2127643778","W2127654085","W2127919389","W2128042445","W2128925770","W2129862350","W2130268098","W2130309539","W2130463305","W2130677668","W2131182446","W2131374581","W2131514269","W2132552845","W2132605719","W2132738192","W2132869153","W2135432480","W2135752489","W2139811688","W2141347543","W2142130708","W2142949159","W2144768886","W2145162648","W2146329391","W2146412061","W2147233604","W2147744624","W2148226521","W2148907477","W2149155737","W2149524014","W2150044210","W2150600289","W2152600059","W2154302927","W2154468733","W2156304718","W2157672947","W2158043276","W2160214035","W2160334111","W2160474160","W2160829291","W2161709017","W2165019077","W2167382412","W2168572117","W2168872380","W2169293230","W2170981157","W2171251976","W2171305096","W2174670974","W2180863964","W2185777446","W2191751149","W2193206597","W2204133637","W2216340177","W2231169545","W2243673883","W2274815048","W2278938776","W2289178561","W2302145134","W2308670362","W2312490292","W2313248704","W2313563931","W2315897902","W2334645172","W2342277544","W2363378650","W2381880499","W2415207269","W2429866240","W2430108188","W2438060767","W2461186558","W2464032901","W2465519120","W2468359603","W2469144556","W2473015971","W2486429399","W2490990160","W2494330206","W2497365301","W2499185134","W2505810543","W2516650870","W2517677353","W2519009793","W2520670351","W2522712802","W2528314521","W2532905222","W2543121299","W2544713334","W2549250279","W2552641290","W2554778377","W2556968239","W2558457320","W2560737727","W2583498283","W2590226842","W2598609934","W2607660811","W2614530326","W2619198014","W2625210731","W2695898996","W2726819335","W2742679029","W2751764714","W2754504873","W2754824141","W2756352938","W2756778465","W2759065239","W2760502374","W2761007043","W2761074613","W2762734952","W2765397101","W2765669570","W2765974464","W2766008982","W2766709859","W2766858724","W2767022457","W2767873334","W2768058319","W2768524866","W2768834448","W2769855804","W2770376730","W2771750633","W2771992487","W2772946006","W2774321232","W2775262821","W2777152579","W2777372386","W2777511983","W2780569657","W2781078111","W2782961903","W2783170155","W2783352927","W2785395942","W2786370255","W2786672560","W2787318954","W2788472604","W2789326571","W2789411649","W2790286726","W2791104480","W2791120664","W2791745770","W2792525775","W2796203944","W2796776641","W2797037420","W2797672600","W2799812352","W2799912658","W2800156564","W2800483169","W2800495074","W2800906030","W2801242023","W2801594204","W2801657569","W2802928520","W2803259405","W2804227365","W2804478633","W2804856255","W2805474745","W2805771335","W2805917037","W2807661149","W2807883405","W2808018930","W2808444906","W2808907262","W2809091571","W2809741249","W2810519509","W2810932621","W2811376825","W2882977976","W2882995622","W2883507226","W2883704421","W2884169568","W2884548543","W2884987576","W2885208612","W2885386636","W2886852065","W2888768695","W2888949708","W2890944349","W2891361490","W2891604155","W2891993120","W2892117428","W2892228308","W2892343072","W2895776887","W2895782389","W2896349174","W2898007154","W2898034033","W2898152519","W2898273287","W2898380854","W2898919128","W2899328354","W2899408887","W2899447482","W2899461901","W2899702864","W2899864787","W2900531799","W2901251982","W2901609995","W2901630300","W2901776898","W2901810672","W2902299301","W2902411700","W2902850633","W2903665391","W2904790386","W2904895936","W2904901044","W2904914354","W2904946774","W2905113469","W2905535531","W2905942963","W2906174863","W2906381065","W2906828572","W2906924500","W2908655165","W2908920100","W2909485101","W2910922638","W2911071331","W2911118964","W2911136666","W2911369981","W2911516462","W2911851653","W2912020157","W2912708088","W2912843034","W2913120575","W2913363232","W2913759170","W2913941725","W2914489634","W2917337303","W2918193929","W2919924471","W2920427734","W2920818998","W2920906396","W2922063765","W2923002946","W2927896138","W2932559193","W2938689122","W2939139118","W2939513370","W2939745984","W2941705808","W2941889181","W2942966791","W2943347494","W2944235954","W2944245860","W2944752532","W2945138307","W2945866350","W2946989723","W2947141751","W2947439912","W2947474565","W2948886343","W2949728395","W2949836107","W2952815940","W2953733166","W2953962031","W2956263215","W2960003280","W2960049244","W2965343096","W2966770423","W2967019924","W2967026235","W2967355043","W2967384638","W2969332410","W2969604753","W2969681925","W2969754811","W2970246714","W2970625172","W2971422873","W2971886249","W2972755783","W2975765346","W2976467917","W2976990520","W2977256519","W2977900975","W2979741480","W2980566250","W2980660234","W2981829174","W2981882415","W2982285225","W2982294811","W2983569614","W2983763493","W2984955818","W2985080647","W2985304397","W2986656870","W2986801444","W2987201834","W2987535147","W2988974583","W2989123953","W2989306646","W2990018663","W2990024397","W2990265920","W2990606490","W2990778964","W2990856540","W2991459440","W2993712353","W2994578413","W2994817612","W2994843752","W2995631322","W2995781992","W2995840337","W2995855915","W2996645988","W2997041690","W2998760175","W2998865753","W2998927069","W2999414134","W2999525506","W2999747982","W2999980947","W2999981690","W3000644249","W3000775429","W3001069219","W3001230374","W3001401140","W3001406994","W3002294434","W3002318919","W3002399360","W3002428806","W3002467440","W3002636653","W3002877507","W3003031541","W3003086173","W3003488560","W3004433408","W3004509705","W3004735014","W3005388122","W3005455928","W3005799683","W3006418270","W3006451697","W3007093530","W3007635864","W3007774730","W3008715491","W3008783510","W3008899623","W3008974412","W3009682821","W3010554738","W3010566419","W3010713366","W3010745840","W3011196735","W3011827088","W3012240312","W3012294133","W3012533812","W3013143008","W3013829534","W3013945982","W3014158956","W3014187200","W3014617198","W3015335586","W3015490194","W3015538832","W3015757473","W3015757537","W3017176764","W3017879913","W3018245946","W3018760090","W3019174547","W3019461108","W3019977729","W3020903952","W3021242715","W3021759073","W3021803035","W3022024007","W3023733639","W3024184641","W3024468757","W3025688008","W3025799229","W3026097854","W3026521874","W3026734769","W3027953923","W3028205489","W3028359224","W3028475724","W3030107733","W3030154364","W3030274770","W3030603415","W3031916030","W3033314184","W3033545166","W3034507732","W3034758791","W3035384710","W3036310791","W3036404422","W3036539042","W3036632615","W3036820224","W3036894024","W3037096845","W3037102096","W3037107829","W3037286788","W3037518329","W3037528992","W3037763485","W3038016020","W3038361518","W3040812790","W3041028737","W3041089638","W3041243423","W3042131687","W3042867568","W3043533584","W3043628321","W3043924842","W3044047259","W3044117470","W3044563663","W3044590679","W3044606135","W3044795343","W3047112015","W3047223579","W3047232868","W3047523542","W3047636069","W3047726629","W3047789540","W3048315630","W3048398814","W3048443380","W3048814354","W3052686670","W3055288645","W3076993331","W3077426680","W3080792037","W3080939497","W3081117878","W3081355632","W3081428722","W3081832022","W3082123197","W3082596907","W3082805163","W3083141864","W3083758046","W3084618394","W3087541015","W3088121216","W3088172808","W3088308333","W3088438996","W3088591189","W3088889790","W3088992157","W3089192400","W3089328591","W3089994602","W3090562667","W3090746157","W3091356589","W3091961594","W3092149276","W3092341362","W3092430417","W3092538383","W3092585017","W3092598152","W3092618446","W3092706105","W3092852606","W3092977391","W3093065594","W3093227945","W3093372480","W3093955271","W3094163676","W3094703748","W3096405865","W3097299616","W3097583761","W3097782402","W3097954874","W3099254212","W3103773452","W3104100189","W3104314734","W3106656463","W3107864692","W3107877676","W3107931619","W3108470736","W3108920563","W3108934821","W3109861943","W3109896851","W3110042739","W3110853477","W3111198152","W3111407736","W3111747339","W3112169905","W3112414810","W3112605623","W3112710440","W3112973828","W3113006106","W3113159930","W3113311354","W3114561587","W3114711665","W3115995168","W3116613772","W3117469597","W3117516523","W3119159328","W3119320733","W3119585017","W3119619653","W3120575302","W3120656309","W3120678254","W3121366987","W3122022709","W3122100396","W3122448827","W3122562367","W3123231979","W3124610766","W3124864973","W3125430603","W3125666390","W3125750968","W3126485686","W3126540447","W3128121759","W3128212381","W3128465816","W3128986180","W3129007391","W3129739550","W3129922141","W3129998045","W3130409719","W3131018395","W3131095702","W3131719476","W3131961909","W3132690998","W3132885109","W3133316011","W3133589388","W3133630643","W3133703462","W3133788982","W3133882938","W3134035333","W3134074448","W3134097909","W3134279974","W3134280505","W3134476194","W3134616987","W3134695887","W3135285129","W3135300960","W3135442753","W3135897697","W3136290048","W3136300347","W3136362508","W3136390452","W3136532039","W3136594673","W3136925335","W3136983769","W3137382114","W3137881469","W3138037336","W3138589873","W3138695700","W3140005073","W3140173160","W3140580853","W3142474638","W3142900105","W3143523822","W3143756877","W3145294107","W3145763325","W3149973932","W3150736260","W3152742205","W3154142437","W3154708647","W3154787182","W3155093370","W3155848752","W3156582688","W3156885831","W3157054384","W3157094372","W3157496375","W3157536452","W3157765617","W3157886499","W3157915071","W3157984007","W3158313001","W3158581844","W3158735115","W3159399995","W3159420401","W3159493665","W3159613248","W3159652746","W3159768335","W3159774868","W3159791574","W3159797986","W3159918457","W3160143515","W3161468670","W3161478110","W3162557622","W3162873606","W3163183782","W3163196467","W3163293840","W3163469341","W3164043159","W3164082586","W3164564563","W3164572476","W3164982415","W3165061839","W3165159569","W3165225186","W3165499901","W3165555279","W3165666348","W3165735596","W3165937310","W3166261275","W3166265825","W3167452584","W3167470870","W3167578458","W3167705214","W3168098983","W3168246180","W3168545913","W3168569806","W3168776436","W3169042901","W3169157822","W3169217612","W3169602024","W3169740594","W3169873184","W3170320801","W3170611971","W3171306319","W3171475923","W3172502979","W3172791791","W3173077530","W3173085514","W3173325888","W3173500528","W3173642325","W3174009970","W3174185763","W3174939480","W3175484419","W3175520319","W3176075508","W3176100365","W3176297977","W3176347993","W3176614132","W3176770037","W3177112528","W3177219454","W3178537392","W3178896880","W3178931638","W3178993613","W3179000515","W3179388216","W3179765034","W3179883605","W3180259255","W3181579571","W3181583185","W3181628318","W3182361951","W3182717750","W3183047539","W3183318554","W3183558312","W3183635124","W3183946456","W3184558592","W3184784420","W3184968753","W3185330243","W3185431341","W3185703352","W3185847870","W3186099625","W3186104796","W3186471346","W3186472566","W3187198887","W3187303295","W3187689958","W3187842267","W3188826447","W3189506607","W3189680882","W3189979379","W3189992120","W3190960735","W3190981597","W3192076633","W3192368730","W3192562479","W3192945965","W3193228038","W3194254635","W3194547367","W3194921748","W3195225450","W3195260545","W3195399118","W3195879536","W3196833906","W3196887549","W3197194340","W3197297261","W3197487643","W3197668355","W3197808683","W3198037878","W3198875804","W3199431575","W3199580753","W3199651616","W3199774222","W3200361458","W3200509928","W3200878629","W3201042577","W3201297065","W3201918263","W3202081012","W3202136200","W3202371052","W3202563772","W3203030364","W3203561869","W3204417470","W3204783158","W3205137648","W3205952153","W3205974906","W3206031208","W3206217297","W3206417499","W3206711039","W3207137384","W3207209831","W3207882204","W3208011952","W3208016689","W3208033597","W3208095140","W3208203214","W3208204058","W3208218443","W3208327148","W3208350832","W3208356281","W3208373214","W3208383205","W3208532918","W3208734847","W3208767091","W3208840663","W3208850642","W3208876657","W3208980584","W3209099959","W3209107499","W3209176845","W3209248539","W3209330851","W3209346120","W3209348067","W3209424139","W3209507730","W3209519557","W3209629045","W3209630450","W3209648119","W3209860170","W3209868854","W3209900028","W3209900536","W3209939328","W3210041600","W3210045867","W3210060645","W3210151124","W3210171707","W3210221790","W3210239920","W3210422835","W3210469006","W3210490262","W3210614560","W3210756867","W3210800110","W3210845664","W3210896637","W3210960422","W3210961117","W3211088822","W3211094746","W3211096923","W3211107216","W3211242015","W3211250973","W3211291801","W3211301114","W3211337524","W3211379842","W3211470715","W3211919505","W3212132794","W3212380292","W3212405773","W3213043282","W3213375358","W3213812569","W3214465557","W3214602988","W3214625971","W3215642678","W3216335708","W3216385143","W3216416295","W3216511818","W3216566484","W3216731046","W3217222404","W3217409729","W3217592632","W4200008819","W4200009588","W4200026162","W4200047345","W4200068637","W4200083829","W4200092205","W4200109127","W4200190024","W4200207680","W4200239263","W4200263675","W4200281976","W4200340794","W4200343243","W4200370890","W4200483750","W4200484119","W4200498744","W4200610439","W4205195074","W4205298166","W4205386374","W4205390205","W4205453198","W4205454498","W4205463385","W4205471327","W4205570166","W4205607722","W4205971113","W4206304011","W4206339275","W4206362528","W4206566002","W4206568419","W4206583929","W4206594704","W4206682245","W4206951778","W4207049889","W4207071011","W4207071765","W4207073941","W4210308169","W4210365787","W4210465482","W4210465876","W4210526291","W4210592839","W4210663889","W4210787793","W4210835971","W4210852342","W4210923778","W4210949675","W4210953700","W4210976589","W4210986670","W4211053276","W4211063333","W4211063667","W4211065261","W4211080504","W4211082682","W4211114055","W4211114668","W4211125823","W4211146221","W4211154645","W4211155907","W4211159604","W4211162346","W4211164408","W4211184025","W4211186823","W4211208068","W4211221075","W4211224434","W4211233163","W4211235006","W4211239384","W4212868069","W4212901334","W4212926313","W4212943877","W4212949671","W4213046288","W4213080549","W4213187577","W4213206316","W4213228981","W4213260963","W4213264847","W4213339724","W4213340832","W4213412522","W4214541982","W4214578355","W4214711221","W4214844716","W4214934798","W4214936463","W4220662981","W4220678790","W4220690316","W4220713469","W4220739771","W4220744947","W4220759081","W4220786370","W4220807642","W4220809332","W4220811274","W4220829048","W4220848157","W4220869245","W4220880853","W4220881889","W4220894287","W4220905390","W4220911583","W4220954174","W4220987564","W4220994745","W4221005180","W4221021933","W4221025235","W4221028911","W4221029450","W4221033686","W4221059619","W4221060081","W4221065664","W4221068901","W4221086117","W4221088875","W4221096131","W4221124346","W4223510495","W4224023408","W4224033921","W4224044965","W4224075461","W4224124012","W4224219895","W4224269837","W4224279189","W4224285666","W4224322256","W4224598172","W4225011975","W4225102694","W4225154123","W4225285098","W4225304554","W4225351101","W4225637229","W4226071621","W4226081676","W4226111950","W4226128796","W4226169891","W4226192540","W4226268434","W4226275529","W4226277020","W4226338847","W4229038651","W4229334188","W4229374930","W4229376168","W4229441609","W4229441840","W4229625899","W4229654731","W4232249741","W4233407710","W4233565295","W4234085755","W4234164483","W4234241406","W4234269322","W4234459924","W4234721561","W4236504206","W4236888567","W4239975309","W4240193250","W4243072808","W4245072227","W4245283308","W4245491582","W4245632730","W4246438352","W4246613436","W4246684142","W4247075867","W4248308599","W4249536693","W4249720755","W4280495043","W4280500339","W4280514808","W4280586148","W4280592441","W4280641053","W4280647530","W4281489757","W4281552532","W4281560219","W4281563038","W4281568170","W4281571651","W4281628495","W4281653023","W4281709886","W4281710712","W4281717819","W4281736161","W4281787380","W4281836523","W4281885643","W4281987821","W4282018109","W4282828301","W4282833285","W4282840432","W4282940551","W4282981207","W4283009889","W4283363279","W4283379951","W4283381916","W4283394293","W4283395001","W4283400098","W4283698427","W4283741139","W4283774529","W4284699386","W4284892665","W4284962095","W4284964405","W4285008508","W4285020279","W4285025873","W4285040192","W4285082199","W4285092513","W4285097006","W4285107485","W4285149905","W4285152151","W4285190825","W4285194147","W4285233005","W4285274514","W4285311970","W4285505900","W4285678398","W4285725766","W4285984322","W4286559524","W4286564629","W4286669197","W4286811047","W4287578207","W4287831418","W4288084840","W4288489135","W4288516331","W4289333333","W4289516045","W4289518034","W4289767465","W4289780417","W4290038907","W4290573464","W4290779358","W4291123816","W4291237335","W4291670496","W4292168260","W4292186320","W4292208952","W4292263645","W4292324276","W4292342637","W4292549450","W4292553291","W4292714548","W4292847270","W4293060808","W4293173604","W4293200063","W4293318319","W4293581846","W4293795836","W4294132090","W4294365732","W4294605381","W4294718664","W4294796646","W4295048309","W4295884542","W4295895107","W4296038580","W4296100798","W4296165045","W4296482638","W4296483029","W4296764821","W4296967546","W4297120075","W4297192813","W4297225318","W4297826887","W4297828728","W4297880926","W4297895597","W4298007793","W4299870202","W4300104998","W4300501739","W4300731913","W4300963159","W4301185084","W4301268333","W4302311804","W4303423609","W4303474785","W4304775582","W4304822863","W4304822885","W4304842799","W4306405781","W4306406424","W4306407284","W4306779834","W4306855706","W4306969509","W4307224017","W4307237982","W4307354982","W4307396023","W4307408812","W4307549195","W4307648576","W4308041350","W4308045222","W4308158291","W4308228583","W4308346989","W4308456483","W4308521580","W4308741572","W4309029850","W4309089725","W4309242652","W4309354786","W4309491483","W4309561404","W4309574461","W4309633996","W4309750817","W4309770604","W4309913737","W4309933646","W4309941200","W4310004272","W4310017060","W4310073084","W4310733373","W4310759543","W4311088171","W4311412383","W4311479060","W4311654161","W4311715871","W4312038284","W4312116117","W4312122534","W4312190462","W4312201874","W4312220370","W4312260928","W4312447909","W4312458269","W4312613234","W4312726804","W4312792496","W4312844681","W4312938750","W4313049871","W4313122098","W4313241558","W4313334680","W4313387518","W4313399531","W4313414228","W4313440074","W4313461244","W4313465468","W4313472658","W4313480485","W4313481872","W4313527924","W4313529889","W4313555680","W4313588222","W4313621217","W4313638467","W4313898292","W4315486843","W4315570859","W4315644617","W4315783807","W4316505620","W4317423299","W4317627182","W4317814988","W4317935309","W4318066372","W4318071612","W4318476017","W4318476073","W4318486493","W4318567403","W4318677292","W4318946806","W4319013622","W4319160922","W4319263634","W4319342133","W4319440183","W4319746340","W4319785861","W4319787226","W4319919549","W4319996861","W4320008723","W4320013855","W4320510601","W4320511159","W4320515164","W4320888114","W4321351209","W4321373311","W4321604156","W4321787171","W4322504062","W4322581595","W4322746270","W4323043886","W4323045244","W4323276227","W4323309951","W4323546382","W4323666157","W4323851612","W4327606281","W4327968170","W4327978114","W4328110970","W4328120212","W4360859671","W4360865006","W4361198950","W4361216735","W4361225200","W4361266957","W4361278354","W4361773290","W4361840561","W4362458024","W4362466947","W4362468681","W4362573725","W4362577846","W4362599850","W4362608046","W4362645672","W4362660655","W4363674223","W4364355690","W4364367538","W4364860378","W4364860902","W4365140447","W4365511853","W4365813041","W4365815367","W4366257324","W4366591173","W4366669444","W4366775653","W4366813469","W4367307370","W4367316381","W4367459499","W4367459547","W4367599260","W4367677259","W4367837770","W4368373179","W4368374023","W4372259606","W4376115229","W4376566994","W4376637286","W4376864124","W4377104677","W4377690785","W4377971731","W4377984312","W4377984328","W4378085674","W4378087134","W4378212146","W4378901586","W4378903301","W4379193904","W4379231521","W4379365271","W4379374909","W4379387540","W4379534598","W4379647013","W4379791006","W4380080643","W4380194016","W4380631406","W4380841926","W4381165065","W4381432119","W4381712984","W4381930659","W4381997185","W4382023783","W4382052147","W4382133137","W4382137375","W4382138162","W4382139804","W4382140290","W4382319530","W4382400090","W4382402546","W4382402714","W4382406513","W4382406847","W4382582248","W4382601333","W4382623225","W4382657780","W4382725281","W4382753383","W4382753692","W4382788216","W4382937920","W4383212383","W4383558737","W4383618863","W4383678381","W4383873755","W4383874131","W4384070456","W4384120855","W4384130040","W4384202279","W4384522274","W4384558028","W4384784532","W4384788317","W4384819983","W4384833550","W4385154090","W4385251802","W4385257329","W4385322577","W4385332655","W4385347874","W4385355044","W4385466559","W4385550057","W4385608567","W4385628461","W4385701892","W4385720409","W4385743490","W4385754818","W4385768722","W4385819158","W4385819176","W4385824529","W4385970676","W4385991123","W4386037730","W4386096609","W4386141768","W4386179497","W4386196652","W4386203510","W4386224548","W4386248332","W4386275988","W4386287111","W4386368923","W4386370410","W4386384026","W4386385551","W4386387592","W4386412177","W4386461207","W4386462147","W4386470223","W4386521405","W4386527182","W4386542324","W4386543811","W4386545929","W4386565980","W4386607500","W4386708360","W4386712952","W4386734346","W4386734502","W4386774793","W4386801684","W4386873139","W4386873159","W4386873169","W4386901751","W4386921987","W4386923936","W4387028897","W4387058677","W4387102443","W4387312916","W4387335706","W4387336036","W4387352964","W4387400490","W4387409188","W4387422679","W4387573925","W4387589867","W4387611178","W4387647664","W4387679757","W4387685995","W4387686367","W4387704608","W4387758439","W4387767631","W4387791666","W4387911242","W4387923511","W4387949583","W4387965663","W4387968987","W4388018251","W4388113259","W4388190566","W4388215428","W4388271941","W4388520253","W4388537657","W4388551062","W4388553120","W4388617609","W4388622467","W4388637004","W4388812476","W4388831847","W4388834956","W4388871093","W4388881876","W4388886539","W4388928848","W4389043197","W4389045751","W4389215747","W4389257303","W54650538"],"group":[1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1],"nodesize":[8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8,8]},"options":{"NodeID":"name","Group":"group","colourScale":"d3.scaleOrdinal(['#3182bd'])","fontSize":7,"fontFamily":"serif","clickTextSize":17.5,"linkDistance":50,"linkWidth":"'1.5px'.toString()","charge":-30,"opacity":0.6,"zoom":true,"legend":false,"arrows":false,"nodesize":true,"radiusCalculation":"d.nodesize","bounded":false,"opacityNoHover":1,"clickAction":null}},"evals":[],"jsHooks":[]}</script>
```


:::
:::



#### And another graph option (have to try it out)


#| eval: false

# https://datastorm-open.github.io/visNetwork/
library(visNetwork)

copub_network <-
    visNetwork(snowball$nodes, snowball$edges, width = "100%") |>
    ## set random seed for reproducibility
    visLayout(randomSeed = 123) |>
    visOptions(
        selectedBy = list(variable = "doi", highlight = TRUE),
        highlightNearest = list(enabled = TRUE, algorithm = "all")
    )
``` -->

