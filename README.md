Project 1-JSON Vinette
================
Grant Swigart
6/7/2020

  - [JSON](#json)
      - [What is it, where does it get used, and why is it a good way to
        store
        data?](#what-is-it-where-does-it-get-used-and-why-is-it-a-good-way-to-store-data)
      - [What package should I use to read JSON data into
        R?](#what-package-should-i-use-to-read-json-data-into-r)
      - [What the puck. Lets look at some JSON
        data\!](#what-the-puck.-lets-look-at-some-json-data)
  - [Grahs and Visualizations](#grahs-and-visualizations)
      - [Top 10](#top-10)
      - [Summary Statistics by
        Position](#summary-statistics-by-position)
      - [Performance by Draft Pick](#performance-by-draft-pick)
      - [Number of Players by Country](#number-of-players-by-country)

# JSON

## What is it, where does it get used, and why is it a good way to store data?

JSON stands for Javascript Object Notation. It is a file type for
storing and transferring data. It was originally invented in the early
2000’s to help servers communicate with browsers.

Lets look at some examples of a JSON objects\!

This is what a JSON object typically looks like. Typically you can
access each part of the JSON data by calling the name of the larger
group, similair to a list in R. However, for us to work with this data
in R we need to convert this data in a usable format.

``` r
library(jsonlite)
hockey_list <- list(Wayne = list(city = "Edmonton",last_name = "Gretzky"), 
          Sidney= list(city = "Pittsburgh",last_name = "Crosby"),
          Gordie= list(city = "Detroit",last_name = "Howe"))
json<-toJSON(hockey_list, pretty = TRUE, auto_unbox = TRUE)
print(json)
```

    ## {
    ##   "Wayne": {
    ##     "city": "Edmonton",
    ##     "last_name": "Gretzky"
    ##   },
    ##   "Sidney": {
    ##     "city": "Pittsburgh",
    ##     "last_name": "Crosby"
    ##   },
    ##   "Gordie": {
    ##     "city": "Detroit",
    ##     "last_name": "Howe"
    ##   }
    ## }

## What package should I use to read JSON data into R?

There ae three main packages that read JSON data into R.

  - rjson
  - RJSONIO
  - jsonlite

One of the major benefits of jsonlite is that it better maps JSON data
into R into data types that are more heavely used such as dataframes,
lists, and matrices Also, it is better in maintaining the way that
missing values ae coded. It also provides more information than the
other packages when an error occurs. JSON lite is also effecient,
especially when compared to rjson.

``` r
json_data<-fromJSON(json,flatten=TRUE)
print(json_data)
```

    ## $Wayne
    ## $Wayne$city
    ## [1] "Edmonton"
    ## 
    ## $Wayne$last_name
    ## [1] "Gretzky"
    ## 
    ## 
    ## $Sidney
    ## $Sidney$city
    ## [1] "Pittsburgh"
    ## 
    ## $Sidney$last_name
    ## [1] "Crosby"
    ## 
    ## 
    ## $Gordie
    ## $Gordie$city
    ## [1] "Detroit"
    ## 
    ## $Gordie$last_name
    ## [1] "Howe"

Notice that jsonlite correctly maps the data to the original data
structure (list of 3 lists.). Now that we have this data in a usable
data structure we can use other tools to summarize and analyze the data.

## What the puck. Lets look at some JSON data\!

Here are the packages that we will use to analyze our data.

``` r
library(devtools)
library(tidyverse)
library(httr)
library(knitr)
```

We will be looking the data available from the [NHL records
API](https://gitlab.com/dword4/nhlapi/-/blob/master/records-api.md).An
API is a application program interface and is essentially software that
allows computers to communicate with one another. Think of how your
browser (chrome, edge, firefox) follows a url to access information from
a webpage. That webpage sends us HTML code that your computer breaks
down into an interface. In this case we want to use R to follow a URL to
request data from the NHL records server. The text data returned from
this survey is JSON structured. Using the above package we can then
convert this data in a R dataframe, list, or tibble which we can
analyze.

API’s usually have several paramaters that you can specify to access
different information. This API has several tables that you can access
tha have information on player records, team records, attendance, and
draft information. We can also filter some of this information according
to a specific team id we want to analyze.

Lets get aquanted with using an API. An API hase a base url that is used
for all data requests and then has parameters we want to set. In this
case we want to look at the New Jersey Devils which has a teamid of 1.
Additiaonlly we want to look at the skater records for this franchise.
After combining the text for this text, we use a the GET function to
submit our request. Printing this request we are able to see a status
code. A status coded of 200 means we requested data from a URL that
exists. If you receive a 404 this means the URL was not found.

``` r
base_url <- 'https://records.nhl.com/site/api'
id<-'1'
build_url=paste0(base_url,'/franchise-skater-records?cayenneExp=franchiseId=',id)
get_request<-GET(build_url)
print(get_request)
```

    ## Response [https://records.nhl.com/site/api/franchise-skater-records?cayenneExp=franchiseId=1]
    ##   Date: 2020-06-11 00:51
    ##   Status: 200
    ##   Content-Type: application/json
    ##   Size: 1.07 MB

Now that we have this request we want to turn the request object into
text. We do this by using the content function and specifying the output
and encoding. Then we use fromJSON to convert the data into a usable
object. The flatten option turns nested data frames into a single data
frame.

``` r
request_text<-content(get_request,"text",encoding='UTF-8')
request_list<-fromJSON(request_text,flatten=TRUE)
request_data<-request_list$data
print(head(request_data,5))
```

    ##      id activePlayer assists firstName franchiseId      franchiseName
    ## 1 16891        FALSE     712      Jean           1 Montréal Canadiens
    ## 2 16911        FALSE     688     Henri           1 Montréal Canadiens
    ## 3 16990        FALSE     422   Maurice           1 Montréal Canadiens
    ## 4 17000        FALSE     728       Guy           1 Montréal Canadiens
    ## 5 17025        FALSE      87     Chris           1 Montréal Canadiens
    ##   gameTypeId gamesPlayed goals lastName
    ## 1          2        1125   507 Beliveau
    ## 2          2        1258   358  Richard
    ## 3          2         978   544  Richard
    ## 4          2         961   518  Lafleur
    ## 5          2         523    88    Nilan
    ##                                                                             mostAssistsGameDates
    ## 1                                     1955-02-19, 1956-12-01, 1962-11-24, 1965-11-20, 1967-12-28
    ## 2                                                                         1963-01-12, 1964-02-01
    ## 3                                                                                     1954-01-09
    ## 4             1977-03-10, 1977-03-12, 1978-02-23, 1979-04-07, 1980-11-12, 1980-12-27, 1981-11-21
    ## 5 1981-12-12, 1983-01-06, 1983-11-23, 1985-02-24, 1986-02-01, 1986-10-25, 1986-10-30, 1987-04-05
    ##   mostAssistsOneGame mostAssistsOneSeason mostAssistsSeasonIds
    ## 1                  4                   58             19601961
    ## 2                  5                   52             19571958
    ## 3                  5                   36             19541955
    ## 4                  4                   80             19761977
    ## 5                  2                   16   19841985, 19861987
    ##                                                                                                       mostGoalsGameDates
    ## 1                                                                                     1955-11-05, 1959-03-07, 1969-02-11
    ## 2                                                             1957-10-17, 1959-03-14, 1961-03-11, 1965-02-24, 1967-03-19
    ## 3                                                                                                             1944-12-28
    ## 4                                                                                                             1975-01-26
    ## 5 1980-11-22, 1981-11-11, 1983-11-09, 1983-12-03, 1984-02-23, 1985-02-07, 1985-02-23, 1985-12-27, 1986-03-04, 1986-03-08
    ##   mostGoalsOneGame mostGoalsOneSeason mostGoalsSeasonIds
    ## 1                4                 47           19551956
    ## 2                3                 30           19591960
    ## 3                5                 50           19441945
    ## 4                4                 60           19771978
    ## 5                2                 21           19841985
    ##   mostPenaltyMinutesOneSeason mostPenaltyMinutesSeasonIds
    ## 1                         143                    19551956
    ## 2                          91                    19601961
    ## 3                         125                    19541955
    ## 4                          51                    19721973
    ## 5                         358                    19841985
    ##                  mostPointsGameDates mostPointsOneGame mostPointsOneSeason
    ## 1                         1959-03-07                 7                  91
    ## 2                         1957-10-17                 6                  80
    ## 3                         1944-12-28                 8                  74
    ## 4 1975-01-04, 1978-02-28, 1979-04-07                 6                 136
    ## 5             1983-12-03, 1985-02-23                 3                  37
    ##   mostPointsSeasonIds penaltyMinutes playerId points positionCode
    ## 1            19581959           1033  8445408   1219            C
    ## 2            19571958            932  8448320   1046            C
    ## 3            19541955           1287  8448321    966            R
    ## 4            19761977            381  8448624   1246            R
    ## 5            19841985           2248  8449883    175            R
    ##   rookiePoints seasons
    ## 1           34      20
    ## 2           40      20
    ## 3           11      18
    ## 4           64      14
    ## 5           15      10

Now that we understand how APIs work, let’s create a larger function to
request multiple different tables from the API. We need to include error
checks to make sure the user understands any request mistakes. We also
use simplier table names to make the requesting data easier.

``` r
get_nhl<-function(table='franchise',id=''){
  #Convert the table names and franchise id's into string to prevent data issues with logical checks. 
  table<-toString(table)
  id<-toString(id)
  #This list of available ids is taken from the franchise table. 
  available_id<-c("8","41","45","37","10","6","43","51","39","3","16","17","49","26","25","4","5","19","7","23","20","2","1","15","22","12","21","53","28","9","14","24","13","18","52","29","30","54")
  
  #Basic logical checks to see if there have been any incorrect API calls or combinations of ids with the wrong table. 
  if (id!='' & (!table %in% c('skater-records','goalie-records','season-records'))){
                stop('ERROR: Franchise Id is only used for these tables:\n
                     "skater-records","goalie-records","season-records"')
  }
  else if (id!='' & (!id %in% available_id)){
                stop(paste("ERROR: The available id's are ",paste0(available_id,collapse=" ")))
  }

  
  #This is the base URL for the API we want to access
  base_url <- 'https://records.nhl.com/site/api'
  #We then build a different url based upon the data table specified.
  if(table=='franchise'){
    build_url=paste0(base_url,'/franchise')
  }
  else if(table=='team-totals'){
    build_url=paste0(base_url,'/franchise-team-totals')
  }
  else if(table=='player'){
    build_url=paste0(base_url,'/player')
  }
  else if(table=='draft'){
    build_url=paste0(base_url,'/draft')
  }
  else if(table=='attendance'){
    build_url=paste0(base_url,'/attendance')
  }
  
  else if(table=='season-records' & id==''){
    build_url=paste0(base_url,'/franchise-season-records')
  }
  else if(table=='goalie-records' & id==''){
    build_url=paste0(base_url,'/franchise-goalie-records')
  }
  else if(table=='skater-records' & id==''){
    build_url=paste0(base_url,'/franchise-skater-records')
  }
  
  else if(table=='season-records' & id!=''){
    build_url=paste0(base_url,'/franchise-season-records?cayenneExp=franchiseId=',id)
  }
  else if(table=='goalie-records' & id!=''){
    build_url=paste0(base_url,'/franchise-goalie-records?cayenneExp=franchiseId=',id)
  }
  else if(table=='skater-records' & id!=''){
    build_url=paste0(base_url,'/franchise-skater-records?cayenneExp=franchiseId=',id)
  }
  
  else {
    stop('ERROR: The available tables are "franchise","team-totals", "season-records", "goalie-records", "skater-records", "player","draft","attendance"')
  }
  get_request<-GET(build_url)
  #Stop if the url request wa not to an acceptable UR?L. 
  if (get_request$status_code==404){
    stop('Status Code 404.\n Inccorect URL. This is likely due to askinig for an unavailable table.\n
        These are tha available tables. Use the "franchise" table to find the list of franchise ids.
         "team-totals","season-records","goalie-records","skater-records","player","draft","attendance"')
  }
  request_text<-content(get_request,"text",encoding='UTF-8')
  request_list<-fromJSON(request_text,flatten=TRUE)
  request_data<-request_list$data
  return(request_data)
}
```

Let’s request all the data from all the tables the above function can
request. We dont have to specify a table for the franchise table because
this is the default.

``` r
franchise<-get_nhl()
team_totals<-get_nhl(table='team-totals')
draft<-get_nhl(table='draft')
attendance<-get_nhl(table='attendance')
player<-get_nhl(table='player')
rec_season<-get_nhl('season-records')
rec_goalie<-get_nhl('goalie-records')
rec_skater<-get_nhl('skater-records')
```

Not we want to request the

# Grahs and Visualizations

## Top 10

Now that we have looked at our data lets view some graphs to identify
trends. The rec\_skater dataframe may has multiple records for the same
player. Lets calcualte some summary statistics for the total points,
assists, penalty minutes, games played, and other information.

``` r
summary_points<-rec_skater %>%
  mutate(full_name=paste(firstName,lastName)) %>%
  group_by(playerId,full_name,activePlayer,positionCode) %>%
  summarise(seasons=sum(seasons),
            total_points=sum(points),
            games_played=sum(gamesPlayed),
            rookiePoints=sum(rookiePoints),
            penaltyMinutes=sum(penaltyMinutes),
            total_assists=sum(assists),
            total_goals=sum(goals)) %>%
  mutate(ppg=round(total_points/games_played,2),
         ppg_label=paste(as.character(ppg),'ppg')) %>%
  arrange(desc(total_points))
```

Lets look at the players who have the most points scored(goals+assists)
of all times. Also lets look at effeciency by adding text that has the
points per game for each player .

``` r
ggplot(summary_points %>% head(10),aes(x=reorder(full_name,total_points),y=total_points))+
  geom_col()+ 
  coord_flip()+
  geom_text(aes(label=ppg_label), position=position_dodge(width=0.9), hjust=1.2,vjust=.2)+
  ylab('Points Scored')+
  xlab('Player')+
  ggtitle('Top 10 Points Scored All Time')
```

![](README_files/figure-gfm/top%2010%20points-1.png)<!-- -->

## Summary Statistics by Position

Next lets break some of the information accross position. Lets group out
data by position code and then take the top 10 highest scorers of each
position.

``` r
top_10_pos<-summary_points %>%
  group_by(positionCode) %>%
  top_n(10,wt=total_points)%>%
  select(positionCode,full_name,total_points,ppg,total_assists,total_goals) 

kable(top_10_pos%>% filter(positionCode=='L') %>% select(-positionCode),caption ="Top 10 Left Wing Scorers" )
```

    ## Adding missing grouping variables: `positionCode`

| positionCode | full\_name       | total\_points |  ppg | total\_assists | total\_goals |
| :----------- | :--------------- | ------------: | ---: | -------------: | -----------: |
| L            | Luc Robitaille   |          1394 | 0.97 |            726 |          668 |
| L            | Johnny Bucyk     |          1369 | 0.89 |            813 |          556 |
| L            | Brendan Shanahan |          1354 | 0.89 |            698 |          656 |
| L            | Dave Andreychuk  |          1338 | 0.82 |            698 |          640 |
| L            | Alex Ovechkin    |          1278 | 1.11 |            572 |          706 |
| L            | Bobby Hull       |          1170 | 1.10 |            560 |          610 |
| L            | Michel Goulet    |          1153 | 1.06 |            605 |          548 |
| L            | Frank Mahovlich  |          1103 | 0.93 |            570 |          533 |
| L            | Keith Tkachuk    |          1065 | 0.89 |            527 |          538 |
| L            | Ray Whitney      |          1064 | 0.80 |            679 |          385 |

Top 10 Left Wing Scorers

``` r
kable(top_10_pos%>% filter(positionCode=='R') %>% select(-positionCode),caption ="Top 10 Right Wing Scorers" )
```

    ## Adding missing grouping variables: `positionCode`

| positionCode | full\_name      | total\_points |  ppg | total\_assists | total\_goals |
| :----------- | :-------------- | ------------: | ---: | -------------: | -----------: |
| R            | Jaromir Jagr    |          1921 | 1.11 |           1155 |          766 |
| R            | Gordie Howe     |          1850 | 1.05 |           1049 |          801 |
| R            | Mark Recchi     |          1533 | 0.93 |            956 |          577 |
| R            | Teemu Selanne   |          1457 | 1.00 |            773 |          684 |
| R            | Jari Kurri      |          1398 | 1.12 |            797 |          601 |
| R            | Brett Hull      |          1391 | 1.10 |            650 |          741 |
| R            | Guy Lafleur     |          1353 | 1.20 |            793 |          560 |
| R            | Mike Gartner    |          1335 | 0.93 |            627 |          708 |
| R            | Jarome Iginla   |          1300 | 0.84 |            675 |          625 |
| R            | Dino Ciccarelli |          1200 | 0.97 |            592 |          608 |

Top 10 Right Wing Scorers

``` r
kable(top_10_pos%>% filter(positionCode=='C') %>% select(-positionCode),caption ="Top 10 Center Scorers" )
```

    ## Adding missing grouping variables: `positionCode`

| positionCode | full\_name    | total\_points |  ppg | total\_assists | total\_goals |
| :----------- | :------------ | ------------: | ---: | -------------: | -----------: |
| C            | Wayne Gretzky |          2857 | 1.92 |           1963 |          894 |
| C            | Mark Messier  |          1887 | 1.07 |           1193 |          694 |
| C            | Ron Francis   |          1798 | 1.04 |           1249 |          549 |
| C            | Marcel Dionne |          1771 | 1.31 |           1040 |          731 |
| C            | Steve Yzerman |          1755 | 1.16 |           1063 |          692 |
| C            | Mario Lemieux |          1723 | 1.88 |           1033 |          690 |
| C            | Joe Sakic     |          1641 | 1.19 |           1016 |          625 |
| C            | Phil Esposito |          1590 | 1.24 |            873 |          717 |
| C            | Joe Thornton  |          1509 | 0.92 |           1089 |          420 |
| C            | Stan Mikita   |          1467 | 1.05 |            926 |          541 |

Top 10 Center Scorers

``` r
kable(top_10_pos%>% filter(positionCode=='D') %>% select(-positionCode),caption ="Top 10 Defensive Scorers" )
```

    ## Adding missing grouping variables: `positionCode`

| positionCode | full\_name       | total\_points |  ppg | total\_assists | total\_goals |
| :----------- | :--------------- | ------------: | ---: | -------------: | -----------: |
| D            | Ray Bourque      |          1579 | 0.98 |           1169 |          410 |
| D            | Paul Coffey      |          1531 | 1.09 |           1135 |          396 |
| D            | Al MacInnis      |          1274 | 0.90 |            934 |          340 |
| D            | Phil Housley     |          1232 | 0.82 |            894 |          338 |
| D            | Larry Murphy     |          1217 | 0.75 |            929 |          288 |
| D            | Nicklas Lidstrom |          1142 | 0.73 |            878 |          264 |
| D            | Denis Potvin     |          1052 | 0.99 |            742 |          310 |
| D            | Brian Leetch     |          1028 | 0.85 |            781 |          247 |
| D            | Larry Robinson   |           958 | 0.69 |            750 |          208 |
| D            | Chris Chelios    |           948 | 0.57 |            763 |          185 |

Top 10 Defensive Scorers

## Performance by Draft Pick

Next lets see what the average ppg for each draft pick. We merged the
draft dataframe with our summary statistcs and then calculate our
average ppf. We expect to see less ppg for defensemen so we should
remove them from our analysis.

``` r
points_draft<-summary_points %>% 
  inner_join(draft,by = "playerId") %>%
  filter(positionCode %in% c('L','R','C'),
         seasons>1) %>%
  group_by(overallPickNumber,positionCode) %>%
  summarise(ppg=mean(ppg),N=n())
```

We create a bar chart to view the ppg for each pick and each player.
Notably to identify the trend lines lets add the geom\_smooth

``` r
ggplot(points_draft,aes(x=overallPickNumber,y=ppg))+
  facet_wrap(~positionCode)+
  geom_bar(stat='identity')+
  geom_smooth(method='loess')+
  ggtitle('Average PPG by Overall Pick Number and Position',subtitle='Seasons>1, Nondefensemen') +
  xlab('Pick number')+
  ylab('Points Per Game')
```

![](README_files/figure-gfm/PPG%20by%20Pick-1.png)<!-- -->

How many people are drafted from each differnt country? We do this by
using the table command on the county code variable to count the number
of players. Then we use the kable funciton to make it more pretty.

## Number of Players by Country

``` r
table(draft$countryCode) %>%kable()
```

| Var1 | Freq |
| :--- | ---: |
| AUS  |    2 |
| AUT  |   14 |
| BEL  |    2 |
| BHS  |    2 |
| BLR  |   26 |
| BRA  |    2 |
| BRN  |    1 |
| CAN  | 6239 |
| CHE  |   67 |
| CHN  |    1 |
| CZE  |  452 |
| DEN  |    1 |
| DEU  |   70 |
| DNK  |   27 |
| EST  |    2 |
| FIN  |  456 |
| FRA  |    9 |
| GBR  |   33 |
| HRV  |    1 |
| HTI  |    1 |
| HUN  |    3 |
| ITA  |    2 |
| JAM  |    1 |
| JPN  |    3 |
| KAZ  |   25 |
| KOR  |    3 |
| LTU  |    3 |
| LVA  |   36 |
| MKD  |    2 |
| MNE  |    1 |
| NGA  |    2 |
| NLD  |    1 |
| NOR  |   21 |
| POL  |    8 |
| PRY  |    1 |
| RUS  |  646 |
| SRB  |    1 |
| SVK  |  166 |
| SVN  |    6 |
| SWE  |  721 |
| THA  |    1 |
| TWN  |    1 |
| TZA  |    1 |
| UKR  |   34 |
| USA  | 2479 |
| UZB  |    1 |
| VEN  |    1 |
| ZAF  |    1 |
| ZWE  |    1 |

We can also see how the number of players has changed from year to year
by graphing the number of usa born and foreign born players over time. I
am a NBA fan so I wondered if similarly we would see an increase of
foreign born plavers over time. However, it seems clear that foreign
born players are drafted more frequentyly. Additionally, the number of
foreign born players looks to be more correlated with the total number
of draft picks than is the number of us born players.

``` r
draft_group<-draft %>%
  mutate(foreign=ifelse(countryCode=="USA",'usa','foreign'))%>%
  group_by(foreign,draftYear) %>%
  summarise(N=n()) %>%
  filter(!is.na(foreign)) 

draft_total<-draft %>%
              group_by(draftYear) %>%
              summarise(N=n(),foreign="total")

draft_country<-bind_rows(draft_group,draft_total)



ggplot(draft_country,aes(x=draftYear,color=foreign))+
  geom_line(aes(y=N,group=foreign))+
  ggtitle('Number of USA Players Drafted by Year')+
  xlab('Draft Year')+
  xlab('Nunmber of Players Drafted')
```

![](README_files/figure-gfm/drafted%20by%20country-1.png)<!-- -->

Are players from any one nation better? We then compae the distribution
of pgg using boxplots. The three distibutions look largely similair.
There appear to be more canadian players with a high points per game
compared to sweden or the united states. However, it should be noted
that there are many more canadian players than sweden or u.s. players.

``` r
country<-summary_points %>% 
  inner_join(draft,by = "playerId") %>%
  filter(countryCode %in% c('CZE','FIN','RUS','SWE','USA','CAN'))


ggplot(country %>% filter(seasons>1,countryCode %in% c('SWE','USA','CAN')),aes(y=ppg))+
  facet_wrap(~countryCode)+
  geom_boxplot()
```

![](README_files/figure-gfm/PPG%20vs%20Country-1.png)<!-- -->

Lastly, lets look at the number of goals and assists by country of
origin. Also, we can include the presence in the hockey hall of fame by
using a third dimension that shows through the shape of the point.

``` r
country_player<-country%>%
  inner_join(player,by=c("birthDate","firstName","height", "lastName"))

ggplot(country_player %>% filter(seasons>1),aes(x=total_assists,y=total_goals,color=countryCode,shape=inHockeyHof))+
  geom_point()+
  geom_smooth(method='lm',se=FALSE,aes(group=countryCode))+
  ggtitle('Career Points and Assists by Country and Presence in Hall of Fame')+
  xlab('Career Assists')+
  ylab('Career Points')
```

![](README_files/figure-gfm/CareerPoints-1.png)<!-- -->
