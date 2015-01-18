#NoSQL Zadanie 3 Egzamin

## Spis treści
- [NoSQL Zadanie 3 Egzamin](#nosql-zadanie-3-egzamin)
    - [Spis treści](#spis-treści)
    - [Punkt pierwszy](#punkt-pierwszy)
    - [Punkt drugi](#punkt-drugi)

## Punkt pierwszy
```
Przygotować funkcje map i reduce, które:
- wyszukają wszystkie anagramy w pliku word_list.txt
```

Na początku pobrałem [word_list.txt](http://wbzyl.inf.ug.edu.pl/nosql/doc/data/word_list.txt). Jako, że jest to bardzo prosty plik, to wystarczy zmiana jego rozszerzenia na csv. Dodałem w edytorze w pierwszym wierszu "word" jako nagłówek, który posłuży mi jako nazwa kolumny w bazie.

Importujemy oczywiście plik do bazy:
```
mongoimport -d wordlist -c words --type csv --file word_list.csv --headerline
```

```
"imported 8199 objects"

> db.words.findOne()
{ "_id" : ObjectId("54a13aa1d542ead55c03dfc3"), "word" : "author" }

> db.words.stats()
{
        "ns" : "wordlist.words",
        "count" : 8199,
        "size" : 393600,
        "avgObjSize" : 48,
        "storageSize" : 696320,
        "numExtents" : 4,
        "nindexes" : 1,
        "lastExtentSize" : 524288,
        "paddingFactor" : 1,
        "systemFlags" : 1,
        "userFlags" : 1,
        "totalIndexSize" : 277984,
        "indexSizes" : {
                "_id_" : 277984
        },
        "ok" : 1
}
```

Teraz przechodząc już do sedna zadania, potrzebujemy funkcji map i reduce. Dla swojej wygody wykorzystałem również funkcję finalize.

```
m = function() {
	var ordered = this.word.split("").sort().join("");
	emit(ordered, this.word);
};

r = function(key, values) {
		result = {"anagramy": values, "ilosc": values.length};
		return result

};

f = function(key, values) {
	if (values.ilosc > 1) 
		return values
}

var res = db.words.mapReduce(
m, 
r, 
{
out: "anagrams", 
finalize: f

}
);
```

wywołanie zmiennej "res" pozwoli wyświetlić nam podstawowe informacje dotyczące wykonania się funkcji mapReduce. 

```
> res
{
        "result" : "anagrams",
        "timeMillis" : 603,
        "counts" : { 
                "input" : 8199,
                "emit" : 8199,
                "reduce" : 914,
                "output" : 7011
        },
        "ok" : 1
}
```

![mapReduce 1](https://cloud.githubusercontent.com/assets/1538320/5792180/f4bd062a-9f0b-11e4-902f-048e58688ada.png)

|Czas|
| ---- |
|603 ms|

Sama funkcja działa na prostej zasadzie grupowania słów, w oparciu o "słowo" zbudowane z liter w kolejności alfabetycznej. 

Jak zatem widać, udało nam się stworzyć 7011 grup anagramów. Dla potwierdzenia, możemy wykonać następującą komendę:

```
> db.anagrams.find().count()
7011
```

Jeśli chcemy wyświetlić tylko te grupy anagramów, które są anagramami nie tylko dla samch sobą, możemy wykonać następującą komendę:

```
> db.anagrams.find({$where: "this.value != null"})
{ "_id" : "aabdor", "value" : { "anagramy" : [ "abroad", "aboard" ], "ilosc" : 2 } }
{ "_id" : "aablst", "value" : { "anagramy" : [ "basalt", "tablas" ], "ilosc" : 2 } }
{ "_id" : "aabmnt", "value" : { "anagramy" : [ "bantam", "batman" ], "ilosc" : 2 } }
{ "_id" : "aacetv", "value" : { "anagramy" : [ "caveat", "vacate" ], "ilosc" : 2 } }
{ "_id" : "aacimn", "value" : { "anagramy" : [ "caiman", "maniac" ], "ilosc" : 2 } }
{ "_id" : "aaclrs", "value" : { "anagramy" : [ "rascal", "scalar" ], "ilosc" : 2 } }
{ "_id" : "aaclsu", "value" : { "anagramy" : [ "casual", "causal" ], "ilosc" : 2 } }
{ "_id" : "aadmrs", "value" : { "anagramy" : [ "madras", "dramas" ], "ilosc" : 2 } }
{ "_id" : "aaffir", "value" : { "anagramy" : [ "affair", "raffia" ], "ilosc" : 2 } }
{ "_id" : "aailmn", "value" : { "anagramy" : [ "animal", "lamina", "manila" ], "ilosc" : 3 } }
{ "_id" : "aailrt", "value" : { "anagramy" : [ "atrial", "lariat" ], "ilosc" : 2 } }
{ "_id" : "aailsv", "value" : { "anagramy" : [ "avails", "saliva" ], "ilosc" : 2 } }
{ "_id" : "aaimnr", "value" : { "anagramy" : [ "airman", "marina" ], "ilosc" : 2 } }
{ "_id" : "aainpt", "value" : { "anagramy" : [ "patina", "pinata" ], "ilosc" : 2 } }
{ "_id" : "aalmnu", "value" : { "anagramy" : [ "alumna", "manual" ], "ilosc" : 2 } }
{ "_id" : "aalrst", "value" : { "anagramy" : [ "altars", "astral", "tarsal" ], "ilosc" : 3 } }
{ "_id" : "aanrtt", "value" : { "anagramy" : [ "rattan", "tantra", "tartan" ], "ilosc" : 3 } }
{ "_id" : "abbder", "value" : { "anagramy" : [ "barbed", "dabber" ], "ilosc" : 2 } }
{ "_id" : "abbeir", "value" : { "anagramy" : [ "babier", "barbie" ], "ilosc" : 2 } }
{ "_id" : "abbelr", "value" : { "anagramy" : [ "barbel", "rabble" ], "ilosc" : 2 } }
...
```

By zliczyć całkowitą ilość takich grup anagramów, wykorzystujemy poniższy kod:

```
db.anagrams.find({$where: "this.value != null"}).count()
914
```

Najpopularniejszy anagram:
```
> db.anagrams.find().sort({"value.ilosc": -1}).limit(3)
{ "_id" : "aceprs", "value" : { "anagramy" : [ "capers", "crapes", "parsec", "pacers", "recaps", "scrape", "spacer" ], "
ilosc" : 7 } }
{ "_id" : "acerst", "value" : { "anagramy" : [ "carets", "crates", "caster", "caters", "recast", "reacts", "traces" ], "
ilosc" : 7 } }
{ "_id" : "adelst", "value" : { "anagramy" : [ "lasted", "salted", "slated", "staled", "deltas", "desalt" ], "ilosc" : 6
 } }
>
```


|Słowo|Ilość|
| ----| ----- |
|aceprs|7|
|acerst|7|
|adelst|6|

## Punkt drugi

```
"wyszukają najczęściej występujące słowa z Wikipedia data PL"
```

Po pobraniu "plwiki-latest-pages-articles.xml.bz2" skorzystałem ze skryptu [xml-dump-import-mongodb](http://jameslinden.com/dataset/wikipedia.org/xml-dump-import-mongodb/) (do tego potrzebne było skonfigurowanie środowiska, oraz zainstalowanie rozszerzenia php-mongo)

Dodatkowo, w związku z problemami podczas konfiguracji, potrzebna była drobna korekcja skryptu. Jego zmodyfikowana wersja znajduje się [na moim repo](https://github.com/Misiek92/NoSQLexam/blob/master/skrypt%20php).

Skrypt jest w pełni zautomatyzowany, zatem po dłuższej chwili chwili, mamy utworzoną już nową bazę z zaimportownymi danymi w czytelnej formie. Odpalenie, po skonfigurowaniu ograniczało się do:

```
D:\wamp\bin\php\php5.5.12>php d:\wamp\www\wiki\wikipedia.org-xmldump-mongodb.php
```
|Czas|
| ------------- |
|4h 21m|

Przykładowy rekord:
```
db.page.findOne()

      "_id" : 4,
      "title" : "Alergologia",
      "ns" : {
              "id" : 0,
              "name" : null
      },
      "redirect" : false,
      "revision" : {
              "id" : 41351093,
              "parent" : 35063052,
              "timestamp" : ISODate("2014-12-27T16:38:14Z"),
              "contributor" : {
                      "id" : 632110,
                      "username" : "Mishu57"
              },
              "minor" : false,
              "comment" : "/* Linki zewnętrzne */ commons",
              "sha1" : "5sgz8sbnobk66fyywx5p8v366s5p5zu",
              "length" : 601,
              "text" : "'''Alergologia''' - dziedzina [[medycyna|medycyny]] zajmująca się rozpoznawaniem i [[leczenie|
czeniem]] [[alergia|schorzeń alergicznych]].\n\n== Zobacz też ==\n{{wikisłownik|alergologia}}\n* [[alergen]]\n\n== Lin
 zewnętrzne ==\n{{commonscat|Allergology}}\n* [http://www.pta.med.pl/ Polskie Towarzystwo Alergologiczne]\n* [http://w
.alergologia.org/ Portal Lekarzy Alergologów 'alergologia.org']\n* [http://alergie.mp.pl/ Alergie.mp.pl, serwis wydawn
twa Medycyna Praktyczna]\n\n\n{{Zastrzeżenia|Medycyna}}\n\n{{Specjalności medyczne}}\n\n[[Kategoria:Alergologia| ]]\n[
ategoria:Specjalności lekarskie]]"
     }
     
```

Statystyki:
```
> db.page.stats()
{
        "ns" : "wikipedia.page",
        "count" : 1671883,
        "size" : 7767421936,
        "avgObjSize" : 4645,
        "storageSize" : 9305772016,
        "numExtents" : 23,
        "nindexes" : 1,
        "lastExtentSize" : 2146426864,
        "paddingFactor" : 1,
        "systemFlags" : 1,
        "userFlags" : 1,
        "totalIndexSize" : 46668608,
        "indexSizes" : {
                "_id_" : 46668608
        },
        "ok" : 1
}
```

Mając już zaimportowaną bazę danych wikipedii, napisałem następująca funkcję mapReduce:

```
m = function() {
	var tmp = this.revision.text.match(/\b(?!&)([a-zążśźęćółń]+)(?!;)\b/gi);
	if (tmp) {
		for (var i = 0; i < tmp.length; i++)
			emit(tmp[i], 1)
	}
};
r = function(key, values) {
	return Array.sum(values);
};
var res = db.page.mapReduce(m, r, {out: "words"});
```

|Czas|
| ------------- |
|8h 34m|

```
res

      "result" : "words",
      "timeMillis" : 23047471,
      "counts" : {
              "input" : 1671883,
              "emit" : 504002499,
              "reduce" : 61375131,
              "output" : 4427445
      },
      "ok" : 1
```
![mapReduce 2](https://cloud.githubusercontent.com/assets/1538320/5792181/f4c62f52-9f0b-11e4-913a-b1c740f4a144.png)

Poprawność regExpa:
![regExp](https://cloud.githubusercontent.com/assets/1538320/5792179/f4a12e96-9f0b-11e4-80f6-6bc01798bb35.png)

Pozostało sprawdzić, jakie słowa padają najczęściej. Wyniki nie zaskakują, bowiem spójniki pojawiają się najczęściej:

```
db.words.find().sort({"value":-1}).limit(10)
"_id" : "w", "value" : 13611027 }
"_id" : "i", "value" : 5736179 }
"_id" : "align", "value" : 4910634 }
"_id" : "na", "value" : 4488300 }
"_id" : "z", "value" : 4409467 }
"_id" : "ref", "value" : 4251272 }
"_id" : "data", "value" : 3381604 }
"_id" : "Kategoria", "value" : 3169207 }
"_id" : "do", "value" : 2830715 }
"_id" : "si", "value" : 2613568 }
```
![Wyniki popularnosci slow](https://cloud.githubusercontent.com/assets/1538320/5792182/f4cbaa04-9f0b-11e4-9002-3f81f70c7ed5.png)

