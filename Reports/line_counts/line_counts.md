Get line and file counts for the TeDDi corpora by genre
================
Steven Moran
10 December, 2021

``` r
library(tidyverse)
library(knitr)
```

# Overview

Calculate line counts from database version of the TeDDi corpus. These
include total line counts, total file counts, for each language and for
each genre represented by each language.

Then compare them with the [pre-database Python
script](../progress/line_counts.py) that counts these figures directly
from the text files.

# Generate line and files counts from the database

Load the R serialized version of the TeDDi database.

``` r
# load('../../Database/test.RData') # for testing
load('../../Database/TeDDi.Rdata') # full database
```

First, remove all NAs (NULLs) in the `clc_line.text` field in the
database, i.e. any empty lines due to issues such as tags without text,
so that we get correct counts:

``` r
clc_line <- clc_line %>% filter(!is.na(text))
```

Then, get the total line counts per
file.

``` r
total_lines_per_file <- clc_line %>% select(file_id, id) %>% distinct() %>% group_by(file_id) %>% summarize(total_lines = n())
```

Now add in the file metadata. This gives us everything we need to
caculate the genre specific number of lines and files per language, and
also their
totals.

``` r
file_metadata <- clc_file %>% select(id, language_name_wals, genre_broad)
all <- left_join(total_lines_per_file, file_metadata, by=c("file_id"="id"))
rm(total_lines_per_file)
all %>% head() %>% kable()
```

| file\_id | total\_lines | language\_name\_wals | genre\_broad |
| -------: | -----------: | :------------------- | :----------- |
|        1 |           91 | Abkhaz               | professional |
|        2 |          256 | Acoma                | non-fiction  |
|        3 |         7799 | Alamblak             | non-fiction  |
|        4 |         9088 | Amele                | non-fiction  |
|        5 |         7957 | Apurinã              | non-fiction  |
|        6 |        31102 | Arabic (Egyptian)    | non-fiction  |

Total lines per
language.

``` r
total_lines <- all %>% select(language_name_wals, total_lines) %>% group_by(language_name_wals) %>% summarize(db_total_lines = sum(total_lines))
total_lines %>% head() %>% kable()
```

| language\_name\_wals | db\_total\_lines |
| :------------------- | ---------------: |
| Abkhaz               |               91 |
| Acoma                |              256 |
| Alamblak             |             7799 |
| Amele                |             9088 |
| Apurinã              |             7957 |
| Arabic (Egyptian)    |            31102 |

Total lines by language and
genre.

``` r
total_lines_per_genre <- all %>% select(language_name_wals, genre_broad, total_lines) %>% group_by(language_name_wals, genre_broad) %>% summarize(genre_lines=sum(total_lines))
```

    ## `summarise()` has grouped output by 'language_name_wals'. You can override using the `.groups` argument.

``` r
total_lines_per_genre %>% head() %>% kable()
```

| language\_name\_wals | genre\_broad | genre\_lines |
| :------------------- | :----------- | -----------: |
| Abkhaz               | professional |           91 |
| Acoma                | non-fiction  |          256 |
| Alamblak             | non-fiction  |         7799 |
| Amele                | non-fiction  |         9088 |
| Apurinã              | non-fiction  |         7957 |
| Arabic (Egyptian)    | non-fiction  |        31102 |

Total files per
language.

``` r
total_files <- all %>% select(language_name_wals, file_id) %>% group_by(language_name_wals) %>% summarize(db_total_files = n())
total_files %>% head() %>% kable()
```

| language\_name\_wals | db\_total\_files |
| :------------------- | ---------------: |
| Abkhaz               |                1 |
| Acoma                |                1 |
| Alamblak             |                1 |
| Amele                |                1 |
| Apurinã              |                1 |
| Arabic (Egyptian)    |                1 |

Total files per
genre.

``` r
total_files_per_genre <- all %>% select(file_id, language_name_wals, genre_broad) %>% group_by(language_name_wals, genre_broad) %>% summarize(genre_files = n())
```

    ## `summarise()` has grouped output by 'language_name_wals'. You can override using the `.groups` argument.

``` r
total_files_per_genre %>% head() %>% kable()
```

| language\_name\_wals | genre\_broad | genre\_files |
| :------------------- | :----------- | -----------: |
| Abkhaz               | professional |            1 |
| Acoma                | non-fiction  |            1 |
| Alamblak             | non-fiction  |            1 |
| Amele                | non-fiction  |            1 |
| Apurinã              | non-fiction  |            1 |
| Arabic (Egyptian)    | non-fiction  |            1 |

Now convert the output of grouping and counting by genres to wide format
for merging. Note there will be many NA
cells.

``` r
total_files_per_genre_wide <- spread(total_files_per_genre, key = genre_broad, value = genre_files)
total_files_per_genre_wide %>% head() %>% kable()
```

| language\_name\_wals | conversation | fiction | grammar | non-fiction | professional |
| :------------------- | -----------: | ------: | ------: | ----------: | -----------: |
| Abkhaz               |           NA |      NA |      NA |          NA |            1 |
| Acoma                |           NA |      NA |      NA |           1 |           NA |
| Alamblak             |           NA |      NA |      NA |           1 |           NA |
| Amele                |           NA |      NA |      NA |           1 |           NA |
| Apurinã              |           NA |      NA |      NA |           1 |           NA |
| Arabic (Egyptian)    |           NA |      NA |      NA |           1 |           NA |

Now for the lines per
genre.

``` r
total_lines_per_genre_wide <- spread(total_lines_per_genre, key = genre_broad, value = genre_lines)
total_lines_per_genre_wide %>% head() %>% kable()
```

| language\_name\_wals | conversation | fiction | grammar | non-fiction | professional |
| :------------------- | -----------: | ------: | ------: | ----------: | -----------: |
| Abkhaz               |           NA |      NA |      NA |          NA |           91 |
| Acoma                |           NA |      NA |      NA |         256 |           NA |
| Alamblak             |           NA |      NA |      NA |        7799 |           NA |
| Amele                |           NA |      NA |      NA |        9088 |           NA |
| Apurinã              |           NA |      NA |      NA |        7957 |           NA |
| Arabic (Egyptian)    |           NA |      NA |      NA |       31102 |           NA |

Finally, rename the columns to match the [pre-database Python script’s
output](../progress/line_counts.csv).

``` r
total_files_per_genre_wide <- total_files_per_genre_wide %>% rename(db_conversation_files = conversation, db_fiction_files = fiction, db_grammar_files = grammar, db_non_fiction_files = 'non-fiction', db_professional_files = professional)

total_lines_per_genre_wide <- total_lines_per_genre_wide %>% rename(db_conversation_lines = conversation, db_fiction_lines = fiction, db_grammar_lines = grammar, db_non_fiction_lines = 'non-fiction', db_professional_lines = professional)
```

Combine the data frames counts and add in the missing values,
i.e. languages without data as indicated in the
[langInfo\_TeDDi.csv](../../LangInfo/langInfo_TeDDi.csv) index.

``` r
db_report <- left_join(total_files, total_lines)
```

    ## Joining, by = "language_name_wals"

``` r
db_report <- left_join(db_report, total_files_per_genre_wide)
```

    ## Joining, by = "language_name_wals"

``` r
db_report <- left_join(db_report, total_lines_per_genre_wide)
```

    ## Joining, by = "language_name_wals"

``` r
index <- read.csv('../../LangInfo/langInfo_TeDDi.csv')
missing.languages <- index %>% filter(is.na(name)) %>% select(name_wals)
missing.languages <- missing.languages %>% rename(name=name_wals)

db_report <- db_report %>% add_row(language_name_wals=missing.languages$name)
```

Lastly, let’s add the TeDDi corpus names for convenience for matching
with the progress report.

``` r
corpus_names <- index %>% select(name_wals, name)
db_report <- left_join(db_report, corpus_names, by=c("language_name_wals" = "name_wals"))
```

How’s it
look?

``` r
kable(db_report)
```

| language\_name\_wals      | db\_total\_files | db\_total\_lines | db\_conversation\_files | db\_fiction\_files | db\_grammar\_files | db\_non\_fiction\_files | db\_professional\_files | db\_conversation\_lines | db\_fiction\_lines | db\_grammar\_lines | db\_non\_fiction\_lines | db\_professional\_lines | name                        |
| :------------------------ | ---------------: | ---------------: | ----------------------: | -----------------: | -----------------: | ----------------------: | ----------------------: | ----------------------: | -----------------: | -----------------: | ----------------------: | ----------------------: | :-------------------------- |
| Abkhaz                    |                1 |               91 |                      NA |                 NA |                 NA |                      NA |                       1 |                      NA |                 NA |                 NA |                      NA |                      91 | Abkhaz\_abk                 |
| Acoma                     |                1 |              256 |                      NA |                 NA |                 NA |                       1 |                      NA |                      NA |                 NA |                 NA |                     256 |                      NA | Acoma\_kjq                  |
| Alamblak                  |                1 |             7799 |                      NA |                 NA |                 NA |                       1 |                      NA |                      NA |                 NA |                 NA |                    7799 |                      NA | Alamblak\_amp               |
| Amele                     |                1 |             9088 |                      NA |                 NA |                 NA |                       1 |                      NA |                      NA |                 NA |                 NA |                    9088 |                      NA | Amele\_aey                  |
| Apurinã                   |                1 |             7957 |                      NA |                 NA |                 NA |                       1 |                      NA |                      NA |                 NA |                 NA |                    7957 |                      NA | Apurina\_apu                |
| Arabic (Egyptian)         |                1 |            31102 |                      NA |                 NA |                 NA |                       1 |                      NA |                      NA |                 NA |                 NA |                   31102 |                      NA | Arabic\_Egyptian\_arz       |
| Arapesh (Mountain)        |                1 |             7834 |                      NA |                 NA |                 NA |                       1 |                      NA |                      NA |                 NA |                 NA |                    7834 |                      NA | Arapesh\_Mountain\_ape      |
| Asmat                     |                2 |               30 |                       2 |                 NA |                 NA |                      NA |                      NA |                      30 |                 NA |                 NA |                      NA |                      NA | Asmat\_tml                  |
| Bagirmi                   |                1 |               10 |                      NA |                 NA |                  1 |                      NA |                      NA |                      NA |                 NA |                 10 |                      NA |                      NA | Bagirmi\_bmi                |
| Barasano                  |                1 |             7608 |                      NA |                 NA |                 NA |                       1 |                      NA |                      NA |                 NA |                 NA |                    7608 |                      NA | Barasano\_bsn               |
| Basque                    |              670 |           585726 |                      NA |                 NA |                 NA |                     669 |                       1 |                      NA |                 NA |                 NA |                  585633 |                      93 | Basque\_eus                 |
| Berber (Middle Atlas)     |                1 |               92 |                      NA |                 NA |                 NA |                      NA |                       1 |                      NA |                 NA |                 NA |                      NA |                      92 | Berber\_MiddleAtlas\_tzm    |
| Burmese                   |                2 |            31019 |                      NA |                 NA |                 NA |                       1 |                       1 |                      NA |                 NA |                 NA |                   30928 |                      91 | Burmese\_mya                |
| Burushaski                |                1 |               55 |                      NA |                 NA |                  1 |                      NA |                      NA |                      NA |                 NA |                 55 |                      NA |                      NA | Burushaski\_bsk             |
| Canela-Krahô              |                1 |              690 |                      NA |                 NA |                 NA |                       1 |                      NA |                      NA |                 NA |                 NA |                     690 |                      NA | CanelaKraho\_ram            |
| Chamorro                  |                2 |             8016 |                      NA |                 NA |                 NA |                       1 |                       1 |                      NA |                 NA |                 NA |                    7924 |                      92 | Chamorro\_cha               |
| Chukchi                   |                1 |             1144 |                      NA |                 NA |                 NA |                       1 |                      NA |                      NA |                 NA |                 NA |                    1144 |                      NA | Chukchi\_ckt                |
| Daga                      |                1 |             7856 |                      NA |                 NA |                 NA |                       1 |                      NA |                      NA |                 NA |                 NA |                    7856 |                      NA | Daga\_dgz                   |
| Dani (Lower Grand Valley) |                1 |               23 |                       1 |                 NA |                 NA |                      NA |                      NA |                      23 |                 NA |                 NA |                      NA |                      NA | Dani\_LowerGrandValley\_dni |
| English                   |             1384 |          1375247 |                      NA |                117 |                 NA |                    1266 |                       1 |                      NA |             494919 |                 NA |                  880236 |                      92 | English\_eng                |
| Fijian                    |                2 |             8012 |                      NA |                 NA |                 NA |                       1 |                       1 |                      NA |                 NA |                 NA |                    7916 |                      96 | Fijian\_fij                 |
| Finnish                   |             2575 |          1910223 |                      NA |                161 |                 NA |                    2413 |                       1 |                      NA |             653116 |                 NA |                 1257011 |                      96 | Finnish\_fin                |
| French                    |             1338 |          1326114 |                      NA |                126 |                 NA |                    1211 |                       1 |                      NA |             531889 |                 NA |                  794134 |                      91 | French\_fra                 |
| Georgian                  |              218 |           156787 |                      NA |                 NA |                 NA |                     217 |                       1 |                      NA |                 NA |                 NA |                  156696 |                      91 | Georgian\_kat               |
| German                    |             1631 |          1517069 |                      NA |                152 |                 NA |                    1478 |                       1 |                      NA |             626298 |                 NA |                  890679 |                      92 | German\_deu                 |
| Gooniyandi                |                3 |              107 |                       3 |                 NA |                 NA |                      NA |                      NA |                     107 |                 NA |                 NA |                      NA |                      NA | Gooniyandi\_gni             |
| Grebo                     |                1 |              451 |                      NA |                 NA |                 NA |                       1 |                      NA |                      NA |                 NA |                 NA |                     451 |                      NA | Grebo\_gry                  |
| Greek (Modern)            |             1587 |          1473221 |                      NA |                165 |                 NA |                    1420 |                       2 |                      NA |             540205 |                 NA |                  932832 |                     184 | Greek\_Modern\_ell          |
| Greenlandic (West)        |                2 |             6377 |                      NA |                 NA |                 NA |                       1 |                       1 |                      NA |                 NA |                 NA |                    6286 |                      91 | Greenlandic\_West\_kal      |
| Guaraní                   |                1 |               83 |                      NA |                 NA |                 NA |                      NA |                       1 |                      NA |                 NA |                 NA |                      NA |                      83 | Guarani\_gug                |
| Guraní                    |                1 |             7896 |                      NA |                 NA |                 NA |                       1 |                      NA |                      NA |                 NA |                 NA |                    7896 |                      NA | NA                          |
| Hausa                     |                3 |             8113 |                      NA |                 NA |                 NA |                       1 |                       2 |                      NA |                 NA |                 NA |                    7931 |                     182 | Hausa\_hau                  |
| Hebrew (Modern)           |             1636 |           997606 |                      NA |                  5 |                 NA |                    1630 |                       1 |                      NA |               8733 |                 NA |                  988784 |                      89 | Hebrew\_Modern\_heb         |
| Hindi                     |               73 |           137510 |                      NA |                 NA |                 NA |                      72 |                       1 |                      NA |                 NA |                 NA |                  137416 |                      94 | Hindi\_hin                  |
| Hixkaryana                |                1 |             1203 |                      NA |                 NA |                 NA |                       1 |                      NA |                      NA |                 NA |                 NA |                    1203 |                      NA | Hixkaryana\_hix             |
| Hmong Njua                |                1 |               91 |                      NA |                 NA |                 NA |                      NA |                       1 |                      NA |                 NA |                 NA |                      NA |                      91 | HmongNjua\_hnj              |
| Imonda                    |                1 |               52 |                      NA |                 NA |                 NA |                       1 |                      NA |                      NA |                 NA |                 NA |                      52 |                      NA | Imonda\_imn                 |
| Indonesian                |             1227 |          1081709 |                      NA |                 NA |                 NA |                    1226 |                       1 |                      NA |                 NA |                 NA |                 1081617 |                      92 | Indonesian\_ind             |
| Jakaltek                  |                1 |             7918 |                      NA |                 NA |                 NA |                       1 |                      NA |                      NA |                 NA |                 NA |                    7918 |                      NA | Jakaltek\_jac               |
| Japanese                  |             1068 |           856773 |                      NA |                 25 |                 NA |                    1042 |                       1 |                      NA |              23015 |                 NA |                  833667 |                      91 | Japanese\_jpn               |
| Kannada                   |                1 |               89 |                      NA |                 NA |                 NA |                      NA |                       1 |                      NA |                 NA |                 NA |                      NA |                      89 | Kannada\_kan                |
| Kayardild                 |                1 |                6 |                      NA |                 NA |                  1 |                      NA |                      NA |                      NA |                 NA |                  6 |                      NA |                      NA | Kayardild\_gyd              |
| Kewa                      |                1 |             9393 |                      NA |                 NA |                 NA |                       1 |                      NA |                      NA |                 NA |                 NA |                    9393 |                      NA | Kewa\_kew                   |
| Khalkha                   |                2 |             8022 |                      NA |                 NA |                 NA |                       1 |                       1 |                      NA |                 NA |                 NA |                    7932 |                      90 | Khalkha\_khk                |
| Khoekhoe                  |                1 |             7956 |                      NA |                 NA |                 NA |                       1 |                      NA |                      NA |                 NA |                 NA |                    7956 |                      NA | Khoekhoe\_naq               |
| Kiowa                     |                1 |               14 |                      NA |                 NA |                 NA |                       1 |                      NA |                      NA |                 NA |                 NA |                      14 |                      NA | Kiowa\_kio                  |
| Korean                    |              854 |           799353 |                      NA |                 NA |                 NA |                     853 |                       1 |                      NA |                 NA |                 NA |                  799261 |                      92 | Korean\_kor                 |
| Kutenai                   |                1 |               11 |                      NA |                 NA |                 NA |                       1 |                      NA |                      NA |                 NA |                 NA |                      11 |                      NA | Kutenai\_kut                |
| Lango                     |                1 |             7958 |                      NA |                 NA |                 NA |                       1 |                      NA |                      NA |                 NA |                 NA |                    7958 |                      NA | Lango\_laj                  |
| Lavukaleve                |                2 |              228 |                      NA |                 NA |                 NA |                       2 |                      NA |                      NA |                 NA |                 NA |                     228 |                      NA | Lavukaleve\_lvk             |
| Luvale                    |                1 |               90 |                      NA |                 NA |                 NA |                      NA |                       1 |                      NA |                 NA |                 NA |                      NA |                      90 | Luvale\_lue                 |
| Makah                     |                5 |               54 |                       3 |                 NA |                 NA |                       2 |                      NA |                      49 |                 NA |                 NA |                       5 |                      NA | Makah\_myh                  |
| Malagasy                  |                2 |            31471 |                      NA |                 NA |                 NA |                       1 |                       1 |                      NA |                 NA |                 NA |                   31385 |                      86 | Malagasy\_plt               |
| Mandarin                  |              784 |           973906 |                      NA |                115 |                 NA |                     667 |                       2 |                      NA |             214838 |                 NA |                  758884 |                     184 | Mandarin\_cmn               |
| Mapudungun                |                2 |             8066 |                      NA |                 NA |                 NA |                       1 |                       1 |                      NA |                 NA |                 NA |                    7922 |                     144 | Mapudungun\_arn             |
| Martuthunira              |                7 |              641 |                       7 |                 NA |                 NA |                      NA |                      NA |                     641 |                 NA |                 NA |                      NA |                      NA | Martuthunira\_vma           |
| Maung                     |                1 |              598 |                      NA |                 NA |                 NA |                       1 |                      NA |                      NA |                 NA |                 NA |                     598 |                      NA | Maung\_mph                  |
| Maybrat                   |                1 |               65 |                      NA |                 NA |                 NA |                       1 |                      NA |                      NA |                 NA |                 NA |                      65 |                      NA | Maybrat\_ayz                |
| Mixtec (Chalcatongo)      |                1 |             7957 |                      NA |                 NA |                 NA |                       1 |                      NA |                      NA |                 NA |                 NA |                    7957 |                      NA | Mixtec\_Chalcatongo\_mig    |
| Ngiyambaa                 |               16 |              237 |                      11 |                 NA |                 NA |                       5 |                      NA |                     217 |                 NA |                 NA |                      20 |                      NA | Ngiyambaa\_wyb              |
| Oromo (Harar)             |                1 |             7956 |                      NA |                 NA |                 NA |                       1 |                      NA |                      NA |                 NA |                 NA |                    7956 |                      NA | Oromo\_Harar\_hae           |
| Otomi (Mezquital)         |                1 |               91 |                      NA |                 NA |                 NA |                      NA |                       1 |                      NA |                 NA |                 NA |                      NA |                      91 | NA                          |
| Otomí (Mezquital)         |                3 |                3 |                      NA |                 NA |                  1 |                       2 |                      NA |                      NA |                 NA |                  1 |                       2 |                      NA | Otomi\_Mezquital\_ote       |
| Paiwan                    |                3 |              196 |                       2 |                 NA |                 NA |                       1 |                      NA |                     132 |                 NA |                 NA |                      64 |                      NA | Paiwan\_pwn                 |
| Persian                   |             1175 |           997917 |                      NA |                  1 |                 NA |                    1173 |                       1 |                      NA |               1018 |                 NA |                  996809 |                      90 | Persian\_pes                |
| Pirahã                    |               17 |              744 |                      17 |                 NA |                 NA |                      NA |                      NA |                     744 |                 NA |                 NA |                      NA |                      NA | Piraha\_myp                 |
| Quechua (Imbabura)        |                1 |             7786 |                      NA |                 NA |                 NA |                       1 |                      NA |                      NA |                 NA |                 NA |                    7786 |                      NA | Quechua\_Imbabura\_qvi      |
| Rama                      |                2 |              165 |                      NA |                 NA |                  1 |                       1 |                      NA |                      NA |                 NA |                128 |                      37 |                      NA | Rama\_rma                   |
| Rapanui                   |                3 |              189 |                      NA |                 NA |                 NA |                       3 |                      NA |                      NA |                 NA |                 NA |                     189 |                      NA | Rapanui\_rap                |
| Russian                   |             1586 |          1011996 |                      NA |                  4 |                 NA |                    1581 |                       1 |                      NA |               5826 |                 NA |                 1006078 |                      92 | Russian\_rus                |
| Sango                     |                2 |             8049 |                      NA |                 NA |                 NA |                       1 |                       1 |                      NA |                 NA |                 NA |                    7956 |                      93 | Sango\_sag                  |
| Sanuma                    |                1 |             7798 |                      NA |                 NA |                 NA |                       1 |                      NA |                      NA |                 NA |                 NA |                    7798 |                      NA | Sanuma\_xsu                 |
| Spanish                   |             1475 |          1455092 |                      NA |                119 |                 NA |                    1355 |                       1 |                      NA |             555446 |                 NA |                  899554 |                      92 | Spanish\_spa                |
| Swahili                   |                3 |             8037 |                      NA |                 NA |                 NA |                       2 |                       1 |                      NA |                 NA |                 NA |                    7944 |                      93 | Swahili\_swh                |
| Tagalog                   |              100 |           150994 |                      NA |                 55 |                 NA |                      44 |                       1 |                      NA |             108206 |                 NA |                   42692 |                      96 | Tagalog\_tgl                |
| Thai                      |             1002 |           820769 |                      NA |                 NA |                 NA |                    1001 |                       1 |                      NA |                 NA |                 NA |                  820679 |                      90 | Thai\_tha                   |
| Tiwi                      |               10 |              162 |                      NA |                 NA |                 NA |                      10 |                      NA |                      NA |                 NA |                 NA |                     162 |                      NA | Tiwi\_tiw                   |
| Turkish                   |             1923 |          1168047 |                      NA |                 NA |                 NA |                    1922 |                       1 |                      NA |                 NA |                 NA |                 1167955 |                      92 | Turkish\_tur                |
| Vietnamese                |              861 |           721814 |                      NA |                 NA |                 NA |                     859 |                       2 |                      NA |                 NA |                 NA |                  721628 |                     186 | Vietnamese\_vie             |
| Warao                     |               15 |              703 |                      NA |                 NA |                 NA |                      15 |                      NA |                      NA |                 NA |                 NA |                     703 |                      NA | Warao\_wba                  |
| Wari’                     |                2 |              394 |                       2 |                 NA |                 NA |                      NA |                      NA |                     394 |                 NA |                 NA |                      NA |                      NA | Wari\_pav                   |
| Wichí                     |                1 |            30448 |                      NA |                 NA |                 NA |                       1 |                      NA |                      NA |                 NA |                 NA |                   30448 |                      NA | Wichi\_mzh                  |
| Wichita                   |                2 |              146 |                       2 |                 NA |                 NA |                      NA |                      NA |                     146 |                 NA |                 NA |                      NA |                      NA | Wichita\_wic                |
| Yagua                     |                2 |             7559 |                      NA |                 NA |                 NA |                       1 |                       1 |                      NA |                 NA |                 NA |                    7467 |                      92 | Yagua\_yad                  |
| Yaqui                     |                1 |             7935 |                      NA |                 NA |                 NA |                       1 |                      NA |                      NA |                 NA |                 NA |                    7935 |                      NA | Yaqui\_yaq                  |
| Yoruba                    |                2 |            30857 |                      NA |                 NA |                 NA |                       1 |                       1 |                      NA |                 NA |                 NA |                   30767 |                      90 | Yoruba\_yor                 |
| Zoque (Copainalá)         |                1 |              147 |                      NA |                 NA |                 NA |                       1 |                      NA |                      NA |                 NA |                 NA |                     147 |                      NA | Zoque\_Copainala\_zoc       |
| Zulu                      |                1 |               92 |                      NA |                 NA |                 NA |                      NA |                       1 |                      NA |                 NA |                 NA |                      NA |                      92 | Zulu\_zul                   |
| Karok                     |               NA |               NA |                      NA |                 NA |                 NA |                      NA |                      NA |                      NA |                 NA |                 NA |                      NA |                      NA | NA                          |
| Koasati                   |               NA |               NA |                      NA |                 NA |                 NA |                      NA |                      NA |                      NA |                 NA |                 NA |                      NA |                      NA | NA                          |
| Koyraboro Senni           |               NA |               NA |                      NA |                 NA |                 NA |                      NA |                      NA |                      NA |                 NA |                 NA |                      NA |                      NA | NA                          |
| Krongo                    |               NA |               NA |                      NA |                 NA |                 NA |                      NA |                      NA |                      NA |                 NA |                 NA |                      NA |                      NA | NA                          |
| Lakhota                   |               NA |               NA |                      NA |                 NA |                 NA |                      NA |                      NA |                      NA |                 NA |                 NA |                      NA |                      NA | NA                          |
| Lezgian                   |               NA |               NA |                      NA |                 NA |                 NA |                      NA |                      NA |                      NA |                 NA |                 NA |                      NA |                      NA | NA                          |
| Mangarrayi                |               NA |               NA |                      NA |                 NA |                 NA |                      NA |                      NA |                      NA |                 NA |                 NA |                      NA |                      NA | NA                          |
| Meithei                   |               NA |               NA |                      NA |                 NA |                 NA |                      NA |                      NA |                      NA |                 NA |                 NA |                      NA |                      NA | NA                          |
| Maricopa                  |               NA |               NA |                      NA |                 NA |                 NA |                      NA |                      NA |                      NA |                 NA |                 NA |                      NA |                      NA | NA                          |
| Slave                     |               NA |               NA |                      NA |                 NA |                 NA |                      NA |                      NA |                      NA |                 NA |                 NA |                      NA |                      NA | NA                          |
| Oneida                    |               NA |               NA |                      NA |                 NA |                 NA |                      NA |                      NA |                      NA |                 NA |                 NA |                      NA |                      NA | NA                          |
| Supyire                   |               NA |               NA |                      NA |                 NA |                 NA |                      NA |                      NA |                      NA |                 NA |                 NA |                      NA |                      NA | NA                          |
| Tukang Besi               |               NA |               NA |                      NA |                 NA |                 NA |                      NA |                      NA |                      NA |                 NA |                 NA |                      NA |                      NA | NA                          |

Let’s write the table to disk.

``` r
write_csv(db_report, 'line_counts.csv')
```

Let’s clean up of the workspace.

``` r
rm(list = ls())
```

# Compare the database vs the Python reports

Read in Olga’s report (Python script’s output that examines the corpus
files directly) from the [progress](../progress/) report.

``` r
report <- read.csv('../progress/line_counts.csv')
```

Read in the database
    report.

``` r
db_report <- read_csv('line_counts.csv')
```

    ## Rows: 101 Columns: 14

    ## ── Column specification ───────────────────────────────────────────────────────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr  (2): language_name_wals, name
    ## dbl (12): db_total_files, db_total_lines, db_conversation_files, db_fiction_...

    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

Combine the reports for
comparison.

``` r
combined_reports <- left_join(report, db_report, by=c('language' = 'name'))
combined_reports$delta_total_lines <- abs(combined_reports$total_lines-combined_reports$db_total_lines)
```

Reorder the report to make it easier to
compare.

``` r
combined_reports <- combined_reports %>% select(language, total_files, db_total_files, fiction_files, db_fiction_files ,non.fiction_files, db_non_fiction_files, conversation_files, db_conversation_files, professional_files, db_professional_files, grammar_files, db_grammar_files, fiction_lines, db_fiction_lines, non.fiction_lines, db_non_fiction_lines, conversation_lines, db_conversation_lines, professional_lines, db_professional_lines, grammar_lines, db_grammar_lines, total_lines, db_total_lines, delta_total_lines)
```

The combined
reports.

``` r
kable(combined_reports)
```

| language                    | total\_files | db\_total\_files | fiction\_files | db\_fiction\_files | non.fiction\_files | db\_non\_fiction\_files | conversation\_files | db\_conversation\_files | professional\_files | db\_professional\_files | grammar\_files | db\_grammar\_files | fiction\_lines | db\_fiction\_lines | non.fiction\_lines | db\_non\_fiction\_lines | conversation\_lines | db\_conversation\_lines | professional\_lines | db\_professional\_lines | grammar\_lines | db\_grammar\_lines | total\_lines | db\_total\_lines | delta\_total\_lines |
| :-------------------------- | -----------: | ---------------: | -------------: | -----------------: | -----------------: | ----------------------: | ------------------: | ----------------------: | ------------------: | ----------------------: | -------------: | -----------------: | -------------: | -----------------: | -----------------: | ----------------------: | ------------------: | ----------------------: | ------------------: | ----------------------: | -------------: | -----------------: | -----------: | ---------------: | ------------------: |
| Abkhaz\_abk                 |            1 |                1 |              0 |                 NA |                  0 |                      NA |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |              0 |                 NA |                  0 |                      NA |                   0 |                      NA |                  91 |                      91 |              0 |                 NA |           91 |               91 |                   0 |
| Acoma\_kjq                  |            1 |                1 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |              0 |                 NA |                256 |                     256 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |          256 |              256 |                   0 |
| Alamblak\_amp               |            1 |                1 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |              0 |                 NA |               7799 |                    7799 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |         7799 |             7799 |                   0 |
| Amele\_aey                  |            1 |                1 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |              0 |                 NA |               9086 |                    9088 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |         9086 |             9088 |                   2 |
| Apurina\_apu                |            1 |                1 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |              0 |                 NA |               7957 |                    7957 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |         7957 |             7957 |                   0 |
| Arabic\_Egyptian\_arz       |            1 |                1 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |              0 |                 NA |              31101 |                   31102 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |        31101 |            31102 |                   1 |
| Arapesh\_Mountain\_ape      |            1 |                1 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |              0 |                 NA |               7834 |                    7834 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |         7834 |             7834 |                   0 |
| Asmat\_tml                  |            2 |                2 |              0 |                 NA |                  0 |                      NA |                   2 |                       2 |                   0 |                      NA |              0 |                 NA |              0 |                 NA |                  0 |                      NA |                  30 |                      30 |                   0 |                      NA |              0 |                 NA |           30 |               30 |                   0 |
| Bagirmi\_bmi                |            1 |                1 |              0 |                 NA |                  0 |                      NA |                   0 |                      NA |                   0 |                      NA |              1 |                  1 |              0 |                 NA |                  0 |                      NA |                   0 |                      NA |                   0 |                      NA |             10 |                 10 |           10 |               10 |                   0 |
| Barasano\_bsn               |            1 |                1 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |              0 |                 NA |               7608 |                    7608 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |         7608 |             7608 |                   0 |
| Basque\_eus                 |          670 |              670 |              0 |                 NA |                669 |                     669 |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |              0 |                 NA |             585315 |                  585633 |                   0 |                      NA |                  93 |                      93 |              0 |                 NA |       585408 |           585726 |                 318 |
| Berber\_MiddleAtlas\_tzm    |            1 |                1 |              0 |                 NA |                  0 |                      NA |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |              0 |                 NA |                  0 |                      NA |                   0 |                      NA |                  92 |                      92 |              0 |                 NA |           92 |               92 |                   0 |
| Burmese\_mya                |            2 |                2 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |              0 |                 NA |              30928 |                   30928 |                   0 |                      NA |                  91 |                      91 |              0 |                 NA |        31019 |            31019 |                   0 |
| Burushaski\_bsk             |            1 |                1 |              0 |                 NA |                  0 |                      NA |                   0 |                      NA |                   0 |                      NA |              1 |                  1 |              0 |                 NA |                  0 |                      NA |                   0 |                      NA |                   0 |                      NA |             55 |                 55 |           55 |               55 |                   0 |
| CanelaKraho\_ram            |            1 |                1 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |              0 |                 NA |                915 |                     690 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |          915 |              690 |                 225 |
| Chamorro\_cha               |            2 |                2 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |              0 |                 NA |               7924 |                    7924 |                   0 |                      NA |                  92 |                      92 |              0 |                 NA |         8016 |             8016 |                   0 |
| Chukchi\_ckt                |            1 |                1 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |              0 |                 NA |               1144 |                    1144 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |         1144 |             1144 |                   0 |
| Cree\_Plains\_crk           |            1 |               NA |              0 |                 NA |                  1 |                      NA |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |              0 |                 NA |                 10 |                      NA |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |           10 |               NA |                  NA |
| Daga\_dgz                   |            1 |                1 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |              0 |                 NA |               7856 |                    7856 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |         7856 |             7856 |                   0 |
| Dani\_LowerGrandValley\_dni |            1 |                1 |              0 |                 NA |                  0 |                      NA |                   1 |                       1 |                   0 |                      NA |              0 |                 NA |              0 |                 NA |                  0 |                      NA |                  23 |                      23 |                   0 |                      NA |              0 |                 NA |           23 |               23 |                   0 |
| English\_eng                |         1384 |             1384 |            117 |                117 |               1266 |                    1266 |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |         494919 |             494919 |             879902 |                  880236 |                   0 |                      NA |                  92 |                      92 |              0 |                 NA |      1374913 |          1375247 |                 334 |
| Fijian\_fij                 |            2 |                2 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |              0 |                 NA |               7916 |                    7916 |                   0 |                      NA |                  96 |                      96 |              0 |                 NA |         8012 |             8012 |                   0 |
| Finnish\_fin                |         2575 |             2575 |            161 |                161 |               2413 |                    2413 |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |         653115 |             653116 |            1256986 |                 1257011 |                   0 |                      NA |                  96 |                      96 |              0 |                 NA |      1910197 |          1910223 |                  26 |
| French\_fra                 |         1338 |             1338 |            126 |                126 |               1211 |                    1211 |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |         531889 |             531889 |             794046 |                  794134 |                   0 |                      NA |                  91 |                      91 |              0 |                 NA |      1326026 |          1326114 |                  88 |
| Georgian\_kat               |          218 |              218 |              0 |                 NA |                217 |                     217 |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |              0 |                 NA |             156696 |                  156696 |                   0 |                      NA |                  91 |                      91 |              0 |                 NA |       156787 |           156787 |                   0 |
| German\_deu                 |         1631 |             1631 |            152 |                152 |               1478 |                    1478 |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |         626281 |             626298 |             890592 |                  890679 |                   0 |                      NA |                  92 |                      92 |              0 |                 NA |      1516965 |          1517069 |                 104 |
| Gooniyandi\_gni             |            1 |                3 |              0 |                 NA |                  0 |                      NA |                   1 |                       3 |                   0 |                      NA |              0 |                 NA |              0 |                 NA |                  0 |                      NA |                  20 |                     107 |                   0 |                      NA |              0 |                 NA |           20 |              107 |                  87 |
| Greek\_Modern\_ell          |         1587 |             1587 |            165 |                165 |               1420 |                    1420 |                   0 |                      NA |                   2 |                       2 |              0 |                 NA |         540201 |             540205 |             932752 |                  932832 |                   0 |                      NA |                 184 |                     184 |              0 |                 NA |      1473137 |          1473221 |                  84 |
| Greenlandic\_West\_kal      |            2 |                2 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |              0 |                 NA |               6286 |                    6286 |                   0 |                      NA |                  91 |                      91 |              0 |                 NA |         6377 |             6377 |                   0 |
| Guarani\_gug                |            2 |                1 |              0 |                 NA |                  1 |                      NA |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |              0 |                 NA |               7896 |                      NA |                   0 |                      NA |                  83 |                      83 |              0 |                 NA |         7979 |               83 |                7896 |
| Hausa\_hau                  |            3 |                3 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   2 |                       2 |              0 |                 NA |              0 |                 NA |               7931 |                    7931 |                   0 |                      NA |                 182 |                     182 |              0 |                 NA |         8113 |             8113 |                   0 |
| Hebrew\_Modern\_heb         |         1636 |             1636 |              5 |                  5 |               1630 |                    1630 |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |           8733 |               8733 |             988620 |                  988784 |                   0 |                      NA |                  89 |                      89 |              0 |                 NA |       997442 |           997606 |                 164 |
| Hindi\_hin                  |           73 |               73 |              0 |                 NA |                 72 |                      72 |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |              0 |                 NA |             137411 |                  137416 |                   0 |                      NA |                  94 |                      94 |              0 |                 NA |       137505 |           137510 |                   5 |
| Hixkaryana\_hix             |            1 |                1 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |              0 |                 NA |               1077 |                    1203 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |         1077 |             1203 |                 126 |
| HmongNjua\_hnj              |            1 |                1 |              0 |                 NA |                  0 |                      NA |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |              0 |                 NA |                  0 |                      NA |                   0 |                      NA |                  91 |                      91 |              0 |                 NA |           91 |               91 |                   0 |
| Imonda\_imn                 |            1 |                1 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |              0 |                 NA |                 52 |                      52 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |           52 |               52 |                   0 |
| Indonesian\_ind             |         1227 |             1227 |              0 |                 NA |               1226 |                    1226 |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |              0 |                 NA |            1081188 |                 1081617 |                   0 |                      NA |                  92 |                      92 |              0 |                 NA |      1081280 |          1081709 |                 429 |
| Jakaltek\_jac               |            1 |                1 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |              0 |                 NA |               7918 |                    7918 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |         7918 |             7918 |                   0 |
| Japanese\_jpn               |         1068 |             1068 |             25 |                 25 |               1042 |                    1042 |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |          23015 |              23015 |             833657 |                  833667 |                   0 |                      NA |                  91 |                      91 |              0 |                 NA |       856763 |           856773 |                  10 |
| Kannada\_kan                |            1 |                1 |              0 |                 NA |                  0 |                      NA |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |              0 |                 NA |                  0 |                      NA |                   0 |                      NA |                  89 |                      89 |              0 |                 NA |           89 |               89 |                   0 |
| Kayardild\_gyd              |            1 |                1 |              0 |                 NA |                  0 |                      NA |                   0 |                      NA |                   0 |                      NA |              1 |                  1 |              0 |                 NA |                  0 |                      NA |                   0 |                      NA |                   0 |                      NA |              6 |                  6 |            6 |                6 |                   0 |
| Kewa\_kew                   |            1 |                1 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |              0 |                 NA |               9393 |                    9393 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |         9393 |             9393 |                   0 |
| Khalkha\_khk                |            2 |                2 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |              0 |                 NA |               7932 |                    7932 |                   0 |                      NA |                  90 |                      90 |              0 |                 NA |         8022 |             8022 |                   0 |
| Khoekhoe\_naq               |            1 |                1 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |              0 |                 NA |               7956 |                    7956 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |         7956 |             7956 |                   0 |
| Kiowa\_kio                  |            1 |                1 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |              0 |                 NA |                 14 |                      14 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |           14 |               14 |                   0 |
| Korean\_kor                 |          854 |              854 |              0 |                 NA |                853 |                     853 |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |              0 |                 NA |             799041 |                  799261 |                   0 |                      NA |                  92 |                      92 |              0 |                 NA |       799133 |           799353 |                 220 |
| Kutenai\_kut                |            1 |                1 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |              0 |                 NA |                 11 |                      11 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |           11 |               11 |                   0 |
| Lango\_laj                  |            1 |                1 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |              0 |                 NA |               7958 |                    7958 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |         7958 |             7958 |                   0 |
| Lavukaleve\_lvk             |            2 |                2 |              0 |                 NA |                  2 |                       2 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |              0 |                 NA |                228 |                     228 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |          228 |              228 |                   0 |
| Luvale\_lue                 |            1 |                1 |              0 |                 NA |                  0 |                      NA |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |              0 |                 NA |                  0 |                      NA |                   0 |                      NA |                  90 |                      90 |              0 |                 NA |           90 |               90 |                   0 |
| Makah\_myh                  |            5 |                5 |              0 |                 NA |                  2 |                       2 |                   3 |                       3 |                   0 |                      NA |              0 |                 NA |              0 |                 NA |                  5 |                       5 |                  49 |                      49 |                   0 |                      NA |              0 |                 NA |           54 |               54 |                   0 |
| Malagasy\_plt               |            2 |                2 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |              0 |                 NA |              31385 |                   31385 |                   0 |                      NA |                  86 |                      86 |              0 |                 NA |        31471 |            31471 |                   0 |
| Mandarin\_cmn               |          784 |              784 |            115 |                115 |                667 |                     667 |                   0 |                      NA |                   2 |                       2 |              0 |                 NA |         214838 |             214838 |             757871 |                  758884 |                   0 |                      NA |                 184 |                     184 |              0 |                 NA |       972893 |           973906 |                1013 |
| Mapudungun\_arn             |            2 |                2 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |              0 |                 NA |               7922 |                    7922 |                   0 |                      NA |                 144 |                     144 |              0 |                 NA |         8066 |             8066 |                   0 |
| Martuthunira\_vma           |            7 |                7 |              0 |                 NA |                  0 |                      NA |                   7 |                       7 |                   0 |                      NA |              0 |                 NA |              0 |                 NA |                  0 |                      NA |                 641 |                     641 |                   0 |                      NA |              0 |                 NA |          641 |              641 |                   0 |
| Maung\_mph                  |            1 |                1 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |              0 |                 NA |                598 |                     598 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |          598 |              598 |                   0 |
| Maybrat\_ayz                |            1 |                1 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |              0 |                 NA |                 65 |                      65 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |           65 |               65 |                   0 |
| Mixtec\_Chalcatongo\_mig    |            1 |                1 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |              0 |                 NA |               7957 |                    7957 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |         7957 |             7957 |                   0 |
| Ngiyambaa\_wyb              |           11 |               16 |              0 |                 NA |                  0 |                       5 |                  11 |                      11 |                   0 |                      NA |              0 |                 NA |              0 |                 NA |                  0 |                      20 |                 217 |                     217 |                   0 |                      NA |              0 |                 NA |          217 |              237 |                  20 |
| Oromo\_Harar\_hae           |            1 |                1 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |              0 |                 NA |               7956 |                    7956 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |         7956 |             7956 |                   0 |
| Otomi\_Mezquital\_ote       |            4 |                3 |              0 |                 NA |                  2 |                       2 |                   0 |                      NA |                   1 |                      NA |              1 |                  1 |              0 |                 NA |                 68 |                       2 |                   0 |                      NA |                  91 |                      NA |            309 |                  1 |          468 |                3 |                 465 |
| Persian\_pes                |         1175 |             1175 |              1 |                  1 |               1173 |                    1173 |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |           1018 |               1018 |             996673 |                  996809 |                   0 |                      NA |                  90 |                      90 |              0 |                 NA |       997781 |           997917 |                 136 |
| Piraha\_myp                 |            1 |               17 |              0 |                 NA |                  1 |                      NA |                   0 |                      17 |                   0 |                      NA |              0 |                 NA |              0 |                 NA |                744 |                      NA |                   0 |                     744 |                   0 |                      NA |              0 |                 NA |          744 |              744 |                   0 |
| Quechua\_Imbabura\_qvi      |            1 |                1 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |              0 |                 NA |               7786 |                    7786 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |         7786 |             7786 |                   0 |
| Rama\_rma                   |            2 |                2 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   0 |                      NA |              1 |                  1 |              0 |                 NA |                 37 |                      37 |                   0 |                      NA |                   0 |                      NA |            128 |                128 |          165 |              165 |                   0 |
| Rapanui\_rap                |            3 |                3 |              0 |                 NA |                  3 |                       3 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |              0 |                 NA |                189 |                     189 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |          189 |              189 |                   0 |
| Russian\_rus                |         1586 |             1586 |              4 |                  4 |               1581 |                    1581 |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |           5816 |               5826 |            1005673 |                 1006078 |                   0 |                      NA |                  92 |                      92 |              0 |                 NA |      1011581 |          1011996 |                 415 |
| Sango\_sag                  |            2 |                2 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |              0 |                 NA |               7956 |                    7956 |                   0 |                      NA |                  93 |                      93 |              0 |                 NA |         8049 |             8049 |                   0 |
| Sanuma\_xsu                 |            1 |                1 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |              0 |                 NA |               7798 |                    7798 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |         7798 |             7798 |                   0 |
| Spanish\_spa                |         1475 |             1475 |            119 |                119 |               1355 |                    1355 |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |         550742 |             555446 |             898339 |                  899554 |                   0 |                      NA |                  92 |                      92 |              0 |                 NA |      1449173 |          1455092 |                5919 |
| Swahili\_swh                |            3 |                3 |              0 |                 NA |                  2 |                       2 |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |              0 |                 NA |               7944 |                    7944 |                   0 |                      NA |                  93 |                      93 |              0 |                 NA |         8037 |             8037 |                   0 |
| Tagalog\_tgl                |          100 |              100 |             55 |                 55 |                 44 |                      44 |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |         108206 |             108206 |              42692 |                   42692 |                   0 |                      NA |                  96 |                      96 |              0 |                 NA |       150994 |           150994 |                   0 |
| Thai\_tha                   |         1002 |             1002 |              0 |                 NA |               1001 |                    1001 |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |              0 |                 NA |             816979 |                  820679 |                   0 |                      NA |                  90 |                      90 |              0 |                 NA |       817069 |           820769 |                3700 |
| Turkish\_tur                |         1923 |             1923 |              0 |                 NA |               1922 |                    1922 |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |              0 |                 NA |            1166063 |                 1167955 |                   0 |                      NA |                  92 |                      92 |              0 |                 NA |      1166155 |          1168047 |                1892 |
| Vietnamese\_vie             |          861 |              861 |              0 |                 NA |                859 |                     859 |                   0 |                      NA |                   2 |                       2 |              0 |                 NA |              0 |                 NA |             721451 |                  721628 |                   0 |                      NA |                 186 |                     186 |              0 |                 NA |       721637 |           721814 |                 177 |
| Warao\_wba                  |            1 |               15 |              0 |                 NA |                  1 |                      15 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |              0 |                 NA |                 43 |                     703 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |           43 |              703 |                 660 |
| Wichi\_mzh                  |            1 |                1 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |              0 |                 NA |              30448 |                   30448 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |        30448 |            30448 |                   0 |
| Wichita\_wic                |            2 |                2 |              0 |                 NA |                  0 |                      NA |                   2 |                       2 |                   0 |                      NA |              0 |                 NA |              0 |                 NA |                  0 |                      NA |                 146 |                     146 |                   0 |                      NA |              0 |                 NA |          146 |              146 |                   0 |
| Yagua\_yad                  |            2 |                2 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |              0 |                 NA |               7467 |                    7467 |                   0 |                      NA |                  92 |                      92 |              0 |                 NA |         7559 |             7559 |                   0 |
| Yaqui\_yaq                  |            1 |                1 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |              0 |                 NA |               7935 |                    7935 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |         7935 |             7935 |                   0 |
| Yoruba\_yor                 |            2 |                2 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |              0 |                 NA |              30767 |                   30767 |                   0 |                      NA |                  90 |                      90 |              0 |                 NA |        30857 |            30857 |                   0 |
| Zoque\_Copainala\_zoc       |            1 |                1 |              0 |                 NA |                  1 |                       1 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |              0 |                 NA |                147 |                     147 |                   0 |                      NA |                   0 |                      NA |              0 |                 NA |          147 |              147 |                   0 |
| Zulu\_zul                   |            1 |                1 |              0 |                 NA |                  0 |                      NA |                   0 |                      NA |                   1 |                       1 |              0 |                 NA |              0 |                 NA |                  0 |                      NA |                   0 |                      NA |                  92 |                      92 |              0 |                 NA |           92 |               92 |                   0 |

Write the comparison report to
disk.

``` r
write.csv(combined_reports, file="compared_reports.csv", quote=FALSE, row.names=FALSE)
```

Clean up.

``` r
rm(list = ls())
```
