## Fonti esterne di calcolo in DLV2

Come si definisce il significato degli atomi esterni? Ogni atomo esterno corrisponderà a una certa computazione, ma di che cosa si tratta? Definire esternamente la semantica degli atomi significa che per ognuno di essi è necessario implementare una funzione (programmazione imperativa) che specifica cosa deve accadere nel momento in cui si invoca o si utilizza quell'atomo all'interno del programma. Le funzioni devono essere scritte in Python. Nel momento in cui si scrive un programma in cui si utilizza un costrutto del tipo:

`&p(t₀, ..., tₙ; u₀, ..., uₘ)`

Insieme al programma, si fornisce la definizione di una funzione chiamata `p`, la quale avrà come argomenti gli _n_ parametri di input e restituirà gli _m_ parametri di output.
Tenendo presente che queste funzioni vengono successivamente utilizzate da DLV, specialmente da I-DLV, nella fase di grounding, tutti questi predicati esterni vengono valutati dalla parte di grounding stessa. In pratica, tali predicati non raggiungono direttamente il solver. È I-DLV che si occupa della loro valutazione. Si può affermare, quindi, che la valutazione è effettuata dalla fase di grounding. Di conseguenza, tali predicati non arrivano al solver, poiché una volta che I-DLV ha invocato la funzione Python necessaria per comprendere, ad esempio, le variabili di output, fornisce una risposta di tipo vero o falso, attribuendo un valore di verità all'atomo in questione. A titolo esemplificativo:

Nel momento in cui si scrive nel programma l'istruzione `x = y + 1`, questa è o vera o falsa. Pertanto, durante la valutazione, si saprà con certezza se è vero o falso. Lo stesso principio si applica a ciascuno di questi atomi speciali. Vengono valutati, possono restituire valori di output, ma insieme a questi valori di output si otterrà un valore di verità: vero o falso. Questo perché il loro valore di verità non dipende dai modelli, dalle interpretazioni, eccetera. È qualcosa che è definito esternamente, e una volta che è stato valutato, si conosce il suo valore di verità.

E quindi nel momento in cui si ha il valore di verita' si ha allora questo vuol dire che se nel corpo della regola si ha una cosa del tipo a :- &p(...) e l'atomo all'interno e' stato valutato come vero, allora non solo si prendono le variabili di output che erano state istanziate con il risultato della computazione, perche' magari servono all'interno della regola, ma si sapra' che quest'atomo e' vero ed essendo vero dalla regola si puo' fare sparire a :- &p ~~(...)~~ , cosi come si sa anche che se il risulato invece e' falso allora la regola viene cancellata ~~a :- &p (...)~~ perche' una regola con corpo falso e' inutile per la computazione. 


Di conseguenza, nel momento in cui si ha il valore di verità, significa che se all'interno del corpo di una regola c'è qualcosa del tipo a :- &p(...), e l'atomo è stato valutato come vero, non solo si considerano le variabili di output che sono state istanziate con il risultato della computazione (magari necessarie all'interno della regola), ma si sa anche che questo atomo è vero. Essendo vero, nella regola si può eliminare a :- &p ~~(...)~~, in quanto non serve più. Al contrario, se il risultato è falso, la regola ~~a :- &p (...)~~ viene eliminata, poiché una regola con un corpo falso è inutile per la computazione.

Nel momento in cui avviene la valutazione di questi built-in esterni e atomi speciali, essi vengono rimossi dalle regole. La loro valutazione si conclude con I-DLV.

---

### Atomi Esterni, Esempio

Ogni volta che non è possibile scrivere un predicato esterno che, in una regola, richieda, ad esempio, due argomenti in input e uno in output, e in un'altra regola richieda due argomenti in input e due in output, significa che se si utilizza un atomo esterno ed esso dovrà sempre avere la stessa arità in tutte le sue occorrenze. Al contrario, per i predicati normali, nello stesso programma è possibile avere due regole con lo stesso atomo, cioè:

`a :- p(x)`

`a :- p(x,y)`

Il nome del predicato è lo stesso, ma l'arità è diversa. In generale, i due "p" sono considerati come due predicati distinti perché la loro definizione sarebbe: il primo predicato "p/1" (p con arità 1) e il secondo predicato "p/2" (p con arità 2). Questa è una situazione che può verificarsi in ASP. Talvolta, ciò può causare confusione perché erroneamente potrebbe essere inserito un parametro in eccesso o si potrebbe dimenticare un parametro, risultando in un output diverso da quanto atteso. Questo avviene perché il linguaggio consente di avere predicati con lo stesso nome ma arità diverse, e vengono trattati come due predicati distinti.

Per quanto riguarda i predicati esterni, come ad esempio "&p" nell'esempio precedente, è importante notare che la loro arità deve essere costante in tutto il programma. Ciò significa che devono avere lo stesso numero di parametri, sia in input che in output, in tutte le occorrenze nel programma.

---

### Atomi Esterni, Esempio (stringa invertita)

**Python:**
```python
1 def sum ( X,Y ):
2    return X+Y
```

**ASP:**

```asp
compute sum(X,Y,Z) :- number(X), number(Y), &sum(X,Y;Z).
```


L'atomo esterno &sum(X, Y; Z) ha come nome di predicato sum e viene riconosciuto come atomo esterno grazie al simbolo &. Tra i suoi parametri, presenta un insieme di variabili di input (X, Y) e una sola variabile di output (Z).

---

**Python:**

```python
1 def rev(S):
2    return S[::-1]
```

**ASP:**

```asp
revWord(Y) :- word(X), &rev(X;Y)
```

L'esempio fornito mostra una funzione Python chiamata rev che inverte una stringa e un'applicazione corrispondente in ASP. La funzione Python ha un solo parametro di input e restituisce esattamente un valore, il che è coerente con la definizione di una funzione in ASP.

---

## Atomi Esterni: valori di ritorno

Generalmente la conversione da Python ad ASP-Core-2 e' implicita. I valori ritornati dalla funzione Python possono essere

- Numerici
- Stringhe
- Booleani

  valori tradotti in ASP:

  Interi -> costante numerica (Es: P(1))

  Tutti gli altri valori -> costanti simboliche (Es: P(rosso))

  Se non e' possibile -> stringhe costanti (Es: P("rosso"))

  ---

## Atomi Esterni: Personalizzare I Criteri Di Mappatura

C'e' la possibilita' di specificare come deve avvenire la conversione, se si vuol usare qualcosa di diverso dal default. Per poterlo fare e' necessario inserire una direttiva nel programma, del tipo:

`#external_precicate_conversion(&p,type: TO₁,...,TOₙ).`

Questa dice la conversione per il predicato esterno, che va chiamato proprio #external_precicate_conversion. Nella parentesi tonda, &p e' il predicto interessato dalla conversione, TO₁,...,TOₙ sono i tipi elencati per i vari parametri, sempre di output dove va rispettato l'ordine. 

Il tipo di conversione può essere:

﻿`@U_INT` intero unsigned
 
`@UT_INT` troncato ad un intero unsigned

`@T_INT` troncato ad un intero

`@UR_INT` arrotondato a un intero unsigned

`@R_INT` arrotondato a un intero

`@CONST` stringa senza virgolette

`@Q_CONST` stringa con virgolette

Esempio: forzare l'output del predicato esterno sum in una stringa quotata:

`#external_precicate_conversion(&sum,type: Q_CONST).`

---

## Atomi Esterni: Funzionali o Relazionali

Si puo' avere e utilizzare dei predicati esterni che sono funzionali oppure predicati che sono relazionali.

Quando si parla di relazione generalmente si parla di una tabella quindi ci sono piu' tuple da considerare. 

Quando si parla di qualcosa che è funzionale, succede sostanzialmente che ci si aspetta un solo valore di ritorno, una sola tupla, dalla funzione. Ad esempio, guardando l'esempio precedente con il built-in `rev` che fa il reverse di una stringa, si può immaginare che una volta passato un argomento alla funzione, cioè la stringa da invertire `&rev(X;Y)`, come risultato ci si aspetta un unico risultato, ovvero la stringa invertita. È importante tenere presente che un singolo built-in potrebbe restituire più valori insieme, ma sarebbe sempre un'unica tupla di valori, poiché la funzione Python può restituire una tupla di valori. Questa tupla viene quindi assegnata agli argomenti, utilizzando l'esempio precedente: `&p(t₀, ..., tₙ; u₀, ..., uₘ)`. In altre parole, se si hanno più argomenti di output, si potrebbe avere una funzione che restituisce una tupla di m elementi. Tuttavia, questa tupla sarà unica nei built-in funzionali, il che significa che una volta che i valori per t₀ e tₙ vengono fissati, si otterrà sempre e solo una sola tupla contenente i valori di output.

Si può avere la possibilità di definire dei built-in che sono relazionali perché il loro valore, quello che restituiscono una volta che vengono passati i parametri di input, non è sempre una sola tupla, ma come valore di ritorno si potrebbe avere una tabella che consiste di più tuple. Avere a che fare con un built-in relazionale è come se producesse (con un esempio):

`a(X) :- p(X,Y);`

Dove p(X,Y) ha la sua tabella di valori:

- p(1,2) <-- vero

- p(2,3) <-- vero

- p(3,4) <-- indefinito

Quando si va a fare l'istanziazione, bisogna stare attenti che tutti i suoi valori devono essere utilizzati per l'istanziazione e devono essere presi in considerazione.

Per i built-in in generale, se questi sono funzionali questo vuol dire che la loro valutazione restituisce sempre lo stesso valore, per esempio:

```asp
a(X,Y,H) :- p(X),q(Y),H=X+1
p(1).
q(1). q(2). q(2).
```

Quando ad X viene assegnato il valore 1, l'input per X in H=X+1 diventa noto, quindi la valutazione di questo built-in restituirà sempre 2.

Il sistema potrebbe anche supportare la definizione di atomi esterni che sono relazionali, ovvero la loro valutazione non comporta una sola tupla, ma coinvolge più tuple. Ad esempio, all'interno di una regola si potrebbe avere:

`a(Y) :- p(X), &H(X;Y)`

In corrispondenza dell'X in &H(X;Y), non solo un valore ma tanti valori potrebbero essere restituiti, come:

H(1,1)

H(1,2)

H(1,7)

H(2,3)

H(2,4)

Poiché viene restituito un elenco di tuple, sostanzialmente si popola una tabella. Quando si procede con l'istanziazione, è necessario fare attenzione poiché, nel caso in cui siano disponibili solo p(1) e p(2), assegnando p(1) a p(X), i possibili valori di Y utilizzati non saranno sempre e solo 1, ma saranno:

per p(1) 

H(1,1)

H(1,2)

H(1,7), quindi si avranno tre istanze, a(1), a(2), e a(7).

Quindi si otterranno tre istanze: a(1), a(2) e a(7). Una volta finite le istanze che si legano a X con valore 1, si cambierà il valore di X a 2 e si esamineranno le istanze che ora si legano a 2, come se fosse un atomo normale.

---

Per fare restituire un insieme, cioe' un elenco, di tuple si usa un built-in relazionale che come valore di ritorno restituisce una sequenza in cui ogni elemento e' una sequenza di tuple. Per esempio:

Si suppone si scrivere una regola che dato un numero calcola quali sono i suoi fattori primi.



**Python**

```python
# Definizione di una funzione chiamata 'cpf' che calcola i fattori primi di un numero 'n'
def cpf(n):
    # Inizializzazione di un contatore 'i' a 2
    i = 2
    # Inizializzazione di una lista 'factors' per memorizzare i fattori primi
    factors = []

    # Iterazione finché il quadrato di 'i' è minore o uguale a 'n'
    while i * i <= n:
        # Verifica se 'n' è divisibile per 'i'
        if n % i:
            # Se non è divisibile, incrementa 'i'
            i += 1
        else:
            # Se è divisibile, divide 'n' per 'i' e aggiunge 'i' alla lista dei fattori
            n //= i
            factors.append(i)

    # Se il numero residuo è maggiore di 1, aggiunge il residuo alla lista dei fattori
    if n > 1:
        factors.append(n)

    # Restituisce la lista dei fattori primi
    return factors

# Esempio di utilizzo della funzione con un numero arbitrario
numero_da_testare = 72
risultato = cpf(numero_da_testare)
print(f"I fattori primi di {numero_da_testare} sono: {risultato}")

```

**ASP**
```asp
prime_factor(X,Z) :- number(X),&cpf(X;Z).
```
`cpf` viene inteso com atomo relazionale. Ciò implica che anziché assegnare un unico valore a Z, viene effettivamente creata una tabella, un predicato chiamato cpf, con due termini. Il primo termine corrisponde ai valori di X, mentre il secondo corrisponde ai valori di Z individuati per X. Per illustrare questo concetto, è come se si popolasse una tabella nel seguente modo:

```asp
cpf(42,2)
cpf(42,3)
cpf(42,7)
```

Visto in questa forma, sembra essere un atomo normale con tre possibili istanze. Quando si istanzia la regola assegnando, ad esempio, 42 a X, si verifica se esiste un'istanza di cpf che lega con 42. Se ne esiste una, come ad esempio 2, allora viene utilizzata e si genera la prima istanza, che è (42, 2). Il processo continua perché ci possono essere altre istanze che legano con 42. Proseguendo, vengono generate ulteriori istanze, e si crea essenzialmente una tabella. La funzione restituisce un elenco di tuple, creando così una lista di valori (o tuple), o una sequenza di sequenze. Ogni sequenza restituita è considerata come una tupla per creare un'istanza di questo atomo cpf. Durante l'istanziazione, la valutazione di questo atomo non termina immediatamente perché è necessario esplorare tutti i valori della tabella.

---

**Predicati Built-In**

Come già vsto per le liste, il sistema fornisce già delle funzioni già implementate per alcune tipologie di problemi, per esempio l'append sulle liste, member delle liste, ecc. C'è ne sono altri che sono già pronti che somigliano a questi atomi esterni ma in realtà sono interni al sistema e realizzano determinati tipi di calcoli in maniera simile a quella che viene per gli atomi esterni ma sono già pronti, cioè la loro implementazione è integrata nel sistema quindi vengono presi e usati, come avvenuto per i built-in lista. 

Sintassi:

`&p(t₀, ..., tₙ; u₀, ..., uₘ)`

- n + m > 0
- &p e' il nome del built-in
- t₀, ..., tₙ sono termini di input
- u₀, ..., uₘ sono termini di output

  Possono essere Funzionali o Relazionali

  Simili agli atomo esterni ma la semantica e' predefinita. Non c'e' la necessita' di definire una funzione Python esternamente perche' sono gia' pronti. La funzione che gli implementa e' gia' integrata nel sistema.

**Built-In Aritmetici**

- &abs (X; Z) restituisce in Z il valore assoluto di X. X deve essere un intero;

- &int (X, Y; Z ) è un atomo relazionale e genera tutti gli interi Z tale che X <=Z<= Y. X e Y devono essere due interi e devono rispettare la condizione X <=Y; Per esempio:

  `p(X)|q(X) :- &int(1,10;X)` genera:

  ```asp
  p(1).
  q(1).
  p(2).
  q(2).
  .
  .
  .
  p(10).
  q(10).
  ```

- &mod (X, Y ; Z) calcola X%Y (resto della divisione) e memorizza il risultato in Z. Y deve essere un numero maggiore di 0; si può scrivere anche Z=X\Y

- &rand (X, Y; Z) genera un intero casuale Z tale che X<=Z <=Y. X e Y devono essere due interi e devono rispettare la condizione X <=Y;

- &sum (X, Y ; Z) calcola la somma di X e Y e memorizza il risultato in Z. Si può scrivere anche Z=X+Y

**Built-in per la manipolazione di stringhe**

- &append str (X,Y;Z) appende Y a X e memorizza la stringa risultante in Z; 

- &length str (X;Z) restituisce in Z la lunghezza della stringa X;

- &member str (X,Y;) verifica se il carattere X è contenuto all'interno di Y, non ha termini di output poiché si tratta di un atomo booleano

- &reverse_str(X;Z) restituisce in Z la stringa risultante dall'inversione della stringa X.

- &sub_str (X,Y,W;Z) genera una sottostringa di W a partire dalla posizione X fino alla posizione Y e memorizza la stringa risultante in Z. X e Y devono essere degli interi, posizioni valide per W e rispettare la condizione X <=Y;

- &to_qstr (X;Z) se necessario, trasforma X in una stringa fra apici.

---

C'e' la possibilita' di fare interagire il sistema con un database sia relazionale che non relazionale. Vedere l25p2 1:01
