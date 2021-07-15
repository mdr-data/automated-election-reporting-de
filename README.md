# automated-election-reporting-de

automated-election-reporting-de automates the election reporting by generating articles based on election data.

____

This repo is a reduced and abstract version of the code we used for reporting the state results on constituency and municipality level during the state election in Sachsen-Anhalt in June 2021. The aim is to document our methods and approach. Additionaly, [this text](https://www.mdr.de/nachrichten/sachsen-anhalt/landtagswahl/warum-wir-automatisierte-wahlberichte-fuer-gemeinden-und-wahlkreise-anbieten-100.html) provides more information regarding our motivation.

**! Caution !**  This repo uses randomly generated sample data which does not reflect any election results.

To generate more general examples from the local election results our direct candidates are taken from the Star Trek universe. Most of the information about characters and the Quadrants is based on ```memory-alpha.wiki```and ```memory-alpha.fandom.com```.

The final articles are in German and some grammar rules are highly adapted to the election context.



The main components of the script are:

- the **article structure** - determines the final structure of an article. It can contain fixed text, as well as text modules.
  In this repo two example structures are included:
  - ```templates/preelection.txt``` for an article before the election started
  - ```templates/article.txt``` for the counting and final election results
- the **text modules** - contains the actual text with variables, e.g. ```The winning party is the *${winner}".```
  Also, the required processing steps to ensure the grammatical correctness of the text snippet is listed.
  An example is noted in ```templates/text_modules/counting.yaml```
- the **data** - is the basis for the extraction of all relevant election metrics such as the turnout, the party with the highest percentage of list share, the name of the winning candidate, etc. The information is extracted, assessed and stored as parameters which are used in the text modules. Besides the actual election results there is background information on the candidates or special information about a constituency. This information is researched and verified by editorial staff before the election. 
  This repo contains randomly generated sample data including fictional candidates from the Star Trek universe. The data is stored in ```data/```

These three components ensure a certain flexibilty. By combining certain article structures with different text modules, it allows us to easily generate articles at different stages of an election: from the preelection phase over the counting stage up to the preliminary and final results.



## Getting Started with docker-compose

To quickly get started start ```docker-compose``` with:

```
docker-compose build
docker-compose up
```

A ```./results``` folder is created which contains one article file for each constituency.



## Local Setup 

For Linux and Mac users there is also an easy way to setup the code within a virtual environment. It only requires ``` python3.9 ``` to be installed. To setup the virtual environment and install all dependencies run:

```
./run install
```



### Generating texts

The main script is called *script.py*. It generates an article for each consituency listed in the sample data files which can be found in the */data/sample/* folder. Start the script with:

```
./run run
```

A possible flag is *--init* or *-i*. It initializes the articles with the preelection text. By default it is set to false.
```
./run run --init
```

A results folder containing a ```txt``` file for each consituency is created.


### Testing

For local testing use: ```./run test```



## How to use the code in the real world 

The aim of this repo is to demonstrate a potential way to approach automated election reporting in German. It is easy to adapt the article structure or text modules to fit in a specific context.
However, to use this code in the "real world", e.g. for election reporting at a state election, the following things have to be added:

- A reliable and verified data source, e.g. the data of the corresponding election organizers, as well as a way to put them into the required csv format which is showcased in ```./data/sample/```
- An article structure that fits the required format for publication, e.g. ```html, xml, md``` or ```txt``` files.
- A way of importing the generated articles into your Content Management System.
- Text is of course only part of the story. Add graphs to your articles to make it more accessible and nicer to read.



And this is how it could look: 

<img src="./data/sample.png" title="sample article" width="800"/>



More examples for automated election reporting with this script can be found here:

- Constituency level: https://www.mdr.de/nachrichten/sachsen-anhalt/landtagswahl/wahlkreisergebnis/index.html
- Municipality level: https://www.mdr.de/nachrichten/sachsen-anhalt/landtagswahl/gemeindeergebnis/gemeinden-von-a-z-100.html

It might be that after a certain period of time the linked articles are not available any more due to the German Interstate Broadcasting Treaty.



## Detailed Process

1. sample data is loaded from *./data/sample/*

2. previous progress state for each region is loaded. e.g. previous progress state: 5 districts are counted
   current progress is loaded and stored as csv in ```./src/data/progess_....csv``` for the next iteration
   
3. If not state results are present, the entire script is stopped!

4. results on state level are computed, e.g. winning party, turnout, 

5. For each constituency:
   1. If the constituency is on the blacklist, the next steps are skipped and for this region no article is created
   2. If all data in this constituency is None, the next steps are skipped and for this region no article is created
   3. a status is set: ```init``` ```counting``` or ```final```
   4. current progress is determined, e.g. 7 districts are counted
      if previous progress == current progress, the next steps are skipped as the article with this progress state already exists
   5. cleanup data: remove results for party "Sonstige" as well as all information for any row in which the column with percentage of votes (*in %*) is None, as this party might not be represented in a specific municipality
   6. process election results to get all required parameter (e.g. winning party, turnout, number of invalid votes)
      On constituency level the results are computed for direct share (Erststimmen) and list share (Zweitstimmen). The corresponding parameter contain respectively the prefix ```list_``` or ```direct_```
   7. parameter are assessed such that specific text modules change dependent on the election outcome (e.g. if the difference to a previous election is positive it's a "win" while if it is negative it's a "lost".)
   8. parameters are formatted:
      - numbers are converted to strings containing a comma instead of a point to indicate decimals, e.g. ```14.5 --> "14,5"```
      - for numbers with *,0* the decimals are removed, e.g. ```16,0 -> 16```
      - integer between 0 and 12 are changed into written numbers, e.g.``` 4 --> vier, 9 --> neun ``` 
   9. Based on the status the respective article structure, e.g. ```article.txt``` or ```preelection.txt``` in ```./src/templates/``` and text modules, e.g. ```counting.yaml, final.yaml``` in ```./src/templates/text_modules/``` are loaded
   10. For each text module the corresponding apply the specified postprocessing steps. This could be:
      - gender: switch the gender of a sentence
        e.g. ```XY, Direktkandidat der Föderation --> XY, Direktkandidatin der Föderation```
      - pluralize party name
        e.g. ```Die Q führen --> Die Qs führen```
      - pluralize verb dependen on party
        e.g. ```Tellariten kommt auf Platz 2 --> Tellariten kommen auf Platz 2```
      - singularize word after 1
        e.g. ```ein Verlust von einen Prozentpunkte --> ein Verlust von einem Prozentpunkt```

   11. A validity check is performed: e.g. the teaser length is checked to not exceed a certain number of characters or it is checked whether the winning direct candidate is listed. Only if all checks are passed the next step is executed
   12. article is saved in ```./results/```





___



## Team 

- Redaktion: Martin Paul, Luca Deutschländer, Thomas Vorreyer, Manuel Mohr, Frank Rugullis

- Umsetzung: Natalie Widmann, Jan Sledz, Mike Czekalla, Uwe Kunzelmann, Karl Liebich

  

[Impressum Mitteldeutscher Rundfunk](https://www.mdr.de/impressum/index.html)

[Impressum ida](https://www.ida.me/impressum/)

___

## Legal Notice

###Lizenz
Python (Source-Code oder aufbereitet) ist bei Beibehaltung des Lizenztextes unter der MIT License frei nutzbar und weiterverbreitbar.

[Lizenztext](https://github.com/mdr-data/automated-election-reporting-de/blob/main/LICENSE)

Für Grafiken wird kein Nutzungsrecht eingeräumt.

###Urheberrecht
Copyright Mitteldeutscher Rundfunk & ida

###Gewähleistungsausschluss
Es besteht keinerlei Gewährleistung für das Programm, soweit dies gesetzlich zulässig ist. Sofern nicht anderweitig schriftlich bestätigt, stellen die Urheberrechtsinhaber und/oder Dritte das Programm so zur Verfügung, „wie es ist“, ohne irgendeine Gewährleistung. Das volle Risiko bezüglich Qualität und Leistungsfähigkeit des Programms liegt bei Ihnen.
