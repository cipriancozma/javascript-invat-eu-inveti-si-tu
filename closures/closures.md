# Closures

Un closure este o funcție care „capturează” valorile din jur care existau la momentul în care s-a născut.
Closure-urile sunt funcții care au acces la scope-ul părinte chiar și atunci când părintele nu mai există.

```js
function closureEx(){
  var a = 'ceva';
  var b = 1000;
  return function(valoare){
    switch (valoare) {
      case 'a': return a;
      case 'b': return b;
    }
  };
};

var cheama = closureEx();
cheama('a'); // ceva
```

Un closure, adică codul din funcția care face closure-ul conține:
- variabile și obiectele care aparțin de scope-ul global,
- variabilele și obiectele care aparțin scope-ului funcției părinte, plus parametrii acesteia,
- toate variabilele declarate după ce funcția (care face closure-ul) a fost declarată.

Closure-ul este mecanismul prin care JavaScript permite ascunderea datelor - încapsularea datelor.

Un closure este mecanismul prin care putem obține o încapsulare dinamică a stări scope-ului, care, atenție, poate fi modificat atâta vreme cât closure-ul există.

## Dependințe cognitive
- funcții
- scope

## Alonjă

Înțelegerea modului în care se face closure vă va ajuta în înțelegerea lui Revealing Module Pattern.
Formează baza funcțională a callback-urilor și a temporizărilor. Amble cazuri implică funcții care sunt apelate asincron la un moment nespecificat din viitor. De regulă, se folosesc astfel de funcții pentru accesarea datelor externe.
Va ajuta la înțelegerea felului în care Event Loop funcționează. Pentru că un clojure memorează contextul, poate prelua controlul de la Event Loop pentru a încheia execuția unui callback, de exemplu.
Funcțiile de nivel înalt și closure-urile formează coloana vertebrală a programării funcționale.

## Definiții

Un closure este un tip special de obiect care combină două lucruri: o funcție și mediul în care funcția a fost creată. Mediul constă din oricare variabilă care exista in-scope la momentul la care closure-ul a fost creat (MDN - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures).
Acest obiect are acces la metodele prototype-ului funcției și la argumente precum call(), apply().

Closure este atunci când o FUNCȚIE ține minte scope-ul lexical chiar și atunci când este executată în afara acelui scope lexical (Kyle Simpson).

Un closure este un obiect special care combină două lucruri: o funcție și mediul în care aceasta a fost declarată.
Mediul, adică scope-ul lexical constă din toate variabilele locale care erau în-scope la momentul în care s-a creat closure-ul.

Un closure este o structură de date care leagă o funcție de mediul preexistent la momentul declarării.


## Mantre

- în JavaScript, variabilele sunt cuprinse în scope pe baza closure-ului în care se află.
- JavaScript are un **scope lexical** generat la faza de compilare.
- closure-uri generează doar funcțiile.
- Un closure permite accesarea variabilelor definite în funcția container.
- Toate variabilele din scope, chiar dacă sunt declarate după ce funcția container a fost declarată, sunt incluse în closure.
- De fiecare dată când funcția externă este apelată, funcția internă este definită din nou. Codul funcției interne va fi identic, dar scope chain-ul asociat va fi diferit.
- **Un closure nu poate accesa `this` al funcției container**. În acest scop se folosește salvarea lui this într-o variabilă `var self = this;`

## Analiză

### Cazul 1 - funcție internă care folosește variabilele din scope

- ai o funcție iar în interior există altă funcție
- funcția internă are acces la tot ce-i în scope
- funcția internă se execută la când se execută și funcția container

```js
function localizare(loc){
  var localitate = 'Rotbav';

  function undeSunt(){
    console.log('Te afli la ' + localitate + ', ' + loc);
  };

  undeSunt();
};

localizare('la biserica întărită');
```

### Cazul 2 - Funcțiile interne pot referi variabile definite în funcția container chiar dacă aceasta a returnat deja.

```js
function localizare(loc){
  var localitate = 'Rotbav';

  function undeSunt(){
    console.log('Te afli la ' + localitate + ', ' + loc);
  };

  return undeSunt();
};

var intrebare = localizare('la bibliotecă');
```

După cum se observă, funcția undeSunt() este executată în afara scope-ului lexical.
Astfel, este explicabil cum face „closure” (prinde ca într-o plasă de pescuit) peste scope-ul înconjurător. Se poate apela la un moment ulterior în program.

Deci, un closure este creat atunci când salvezi o funcție care are acces la contextul de execuție curent într-o variabilă în afara contextului de execuție curent.

### Cazul 3 - Returnarea unei funcții care face closure

Acest caz este util pentru a înțelege mai târziu șabloanele - patterns așa cum sunt Module cu a sa variantă Revealing Module.

```js
function adunare(x) {
  return function(y) {
    return x + y;
  };
}

var adunare1 = adunare(5);
console.log(adunare1(2));  // 7
```

adunare() este cazul prezentat de Mozilla. Este perfect pentru a înțelege felul în care closure-ul se formează în primă fază returnându-se funcția internă iar aceasta la rândul său returnând variabilele din mediu.
- adunare() generează scope-ul iar parametrul x devine variabilă locală atribuindu-se valoarea 5 la momentul invocării;
- adunare returnează o funcție iar variabila add5 devine referință către funcția internă. Astfel, adunare(5) este o funcție a cărui parametru y devine variabilă locală în scope-ul creat de ea.
- adunare1(2) atribuie variabilei interne valoarea 2 și returnează suma dintre variabila locală și variabila peste care s-a făcut closure din funcția container.

Spune documentația Mozilla că „în esență, adunare este o fabrică de funcții”.

### Cazul 4 - Closure cu o referință către funcția internă - execuția funcției interne din extern, nu prin execuția containerului.

```js
var externa = 'ceva extern';
var referintaCatreIntern;

function container(){
  var valContainer = 'ceva din container';

  function interna(){
    console.log('Am acces la ' + externa + ' și la ' + valContainer);
  };

  // încarci referința către funcția internă în variabila din afara containerului.
  referintaCatreIntern = interna;
};

container(); // creează closure-ul
referintaCatreIntern(); Am acces la ceva extern și la ceva din container

// din interiorul funcției interne poți executa funcția container; este echivalentul seriei de execuție de mai sus
referintaCatreIntern(container());
```

### Cazul 5 - Crearea și accesarea membrilor unui obiect creat cu un constructor

John Resig aduce acest caz de closure format de crearea unui obiect pe baza unei funcții constructor.

Funcțiile cu rol de constuctori pot defini întern metodele, iar acestea devin niște metode tip „accessor” sau „getter”, care te ajută să ajungi din scope-ul extern la valorile din constructor.

```js
function Sablon(){
  var cantitate = 10;

  this.getCantitate = function(){
    return cantitate;
  };

  this.incrementare = function(){
    cantitate++;
  };
};

var obiect = new Sablon();

obiect.getCantitate(); // 10
obiect.incrementare(); // undefined
obiect.getCantitate(); // 11
```
Pentru a ajunge la cantitate este nevoie de metode de acces („accessors”). Acesta este și unul din cazurile de realizare a unui closure.

## Referințe către obiecte care nu fac closure.

Exemplu de referințe către obiecte care nu este un closure (prezentat de Kyle Simpson):

```js
var obiect = (function(){
  var o = {test: "test"};
  return {obj: o} // obj ține o referință către obiectul o
})();

console.log(obiect.obj.test); // => test
```

## Utilitate

Closure-ul poate simula variabile private.
Este baza lui MODULE PATTERN prin care se realizează încapsularea și/sau ascunderea datelor.

Cu ajutorul closure-urilor se poate scrie cod care să ruleze într-un mediu izolat, adică într-un closure. Acest closure, acest model de incapsulare a funcționalităților, durează câtă vreme rulează aplicația și oferă o zonă privată care poate memora o stare. Un exemplu este o funcție anonimă executată imediat.

```js
// exemplu de closure anonim
(function () {
	// toate variabilele și funcțiile se află doar în acest scope
	// se pot accesa toate variabilele globale
}());
```

Pornind de la acest model, se poat rula cod extern:

```js
(function($, Mootools){
  // acum ai acces la globalele jQuery ca prescurtarea $ și la Mootools
}(jQuery, Mootools));
```

Acest model stă la baza șablonului Module.

## Erori
Este considerată a fi o eroare crearea de closure-uri în bucle.