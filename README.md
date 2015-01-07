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

Mając już zaimportowaną bazę danych wikipedii, napisałem następująca funkcję mapReduce:

```
m = function() {
	var tmp = this.revision.text.match(/[a-zA-Ząśżźęćńół]+/g);
	if (tmp) {
		for (var i = 0; i < tmp.length; i++)
			emit(tmp[i], 1)
	}
};
r = function(key, values) {
	return Array.sum(values);
};
var res = db.page.mapReduce(m, r, {out: "slowa"});
```

|Czas|
| ------------- |
|8h 34m|

```

> res
{
        "result" : "slowa",
        "timeMillis" : 20780416,
        "counts" : {
                "input" : 1671883,
                "emit" : 596487211,
                "reduce" : 63921156,
                "output" : 4131470
        },
        "ok" : 1
}
```

Pozostało sprawdzić, jakie słowa padają najczęściej. Wyniki nie zaskakują, bowiem spójniki pojawiają się najczęściej:

```
> db.slowa.find().sort({"value":-1}).limit(10)
{ "_id" : "w", "value" : 20292468 }
{ "_id" : "i", "value" : 5780516 }
{ "_id" : "a", "value" : 5407771 }
{ "_id" : "align", "value" : 4910641 }
{ "_id" : "na", "value" : 4872846 }
{ "_id" : "z", "value" : 4775000 }
{ "_id" : "ref", "value" : 4264414 }
{ "_id" : "o", "value" : 3722458 }
{ "_id" : "data", "value" : 3514935 }
{ "_id" : "Kategoria", "value" : 3169267 }
```

Same wyniki nie są może najlepsze bowiem wyszukują również słowa wykorzystywane podczas formatowania tekstu, lecz jak by nie patrzeć, są to również słowa. Tutaj trzeba by opierać się na zmienie wyrażenia regularnego. Aktualnie wykorzystałem:

```
/[a-zA-Ząśżźęćńół]+/g
```

Można by próbować dalej
```
/\s[a-ząśżźęćńół]+/gi
```
lecz to i tak wciąż nie jest rozwiązanie idealne. Stworzenie idealnego regexa byłoby bardzo skomplikowane, wymagające wielu wyjątków, a i tak nie ma pewności czy ostateczny rezultat nie zmanipuluje wynikami. Zatem wyszedłem z założenia, że lepiej stworzyć prosty skrypt, który da wiarygodne wyniki lecz ze śmieciami, które później na etapie prezentacji danych można usunąć.



