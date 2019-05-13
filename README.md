# Zadanie Newsletter 10

Rozwiązania do zadania z Newslettera #10 - 08 maja 2019

Do Newslettera można zapisać się na stronie: https://mailtrain.javastart.pl/subscription/tc5pDEzUq

Omawiany kod:

```
class StringTask {
    public static void main(String[] args) {
        String string1 = "Hello".concat("World");           // Linia-1
        String string2 = new String("HelloWorld");          // Linia-2
        String string3 = string1.intern();                  // Linia-3
        System.out.println(string1 == string3);             // Linia-4
    }
}
```
#### Dlaczego jeżeli w poniższym kodzie usunę Linia-2 to wynik Linia-4 zmieni się z false na true?


Spróbuję to wyjaśnić przedstawiając swoją teorię na ten temat. Wydaje mi się, że pomocne do zrozumienia tego przypadku jest wiedza o tym, że 
`String string2 = new String("HelloWorld")`
nie tworzy jednego obiektu w pamięci ale dwa obiekty w pamięci. Jeden obiekt jest tworzony w String Pool i jeden poza String Pool.  `string2` to referencja do obiektu nie będącego w String Pool. 

Co robi metoda `intern()?` Metoda ta pobiera wartość Stringa ze String Pool. W jaki sposób pobiera określa to dokumentacja:
>     * <p>
>     * When the intern method is invoked, if the pool already contains a
>     * string equal to this {@code String} object as determined by
>     * the {@link #equals(Object)} method, then the string from the pool is
>     * returned. Otherwise, this {@code String} object is added to the
>     * pool and a reference to this {@code String} object is returned.
>     * <p>
Czyli jeśli w String Pool jest już dany String (w naszym przypadku "HelloWorld") to zostanie zwrócony, jeśli go nie ma w String Pool to ten String jest dodawany do String Pool i referencja do tego dodanego obiektu jest zwracana.
Jeszcze dokładniej co zwraca `public native String intern();`:
>     * @return  a string that has the same contents as this string, but is
>     *          guaranteed to be from a pool of unique strings.

Co ciekawe, referencje do obiektów z wykomentawaną linią 2 czy bez wykomentowanej linii 2 po kompilacji zawsze są takie same:
```
System.out.println("Hashcode string1 " + Integer.toHexString(string1.hashCode()));
System.out.println("Hashcode string3 " + Integer.toHexString(string3.hashCode()));
```

> true                                                    
> Hashcode string1 1a2fa200              
> Hashcode string3 1a2fa200                                                   

```
System.out.println("Hashcode string1 " + Integer.toHexString(string1.hashCode()));
System.out.println("Hashcode string2 " + Integer.toHexString(string2.hashCode()));
System.out.println("Hashcode string3 " + Integer.toHexString(string3.hashCode()));
```

> false                                  
> Hashcode string1 1a2fa200                                   
> Hashcode string2 1a2fa200                                 
> Hashcode string3 1a2fa200                                         

Dlatego poleceniem `javap -c StringTask.class` można wykonać disassemblację kodu i podejrzeć zachowanie w pamięci:

Z Linia-2:

![with_Linia-2](https://user-images.githubusercontent.com/26818304/57659160-9c301900-75e1-11e9-8fc9-33166610b537.PNG)

Bez Linia-2:

![without_Linia-2](https://user-images.githubusercontent.com/26818304/57659159-9c301900-75e1-11e9-93cf-8314a88407f5.PNG)


Niestety dość ciężko jest mi zrozumieć poszczególne kody, ale przykładowo __astore_X__ (X - cyfra) oznacza przypisanie referencji utworzonej w puli do zmiennej lokalnej, jak widać w przypadku kompilacji bez Linia-2 przypisanie referencji utworzonej w puli do zmiennej lokalnej odbywa się dwa razy: astore_1 i astore_2. W przypadku gdy zwracany jest false występują astore_1, astore_2 i astore_3. 

### Moja teoria:
* z Linia-2:
Referencja string3 wskazuje na obszar w pamięci na obiekt "HelloWorld" który jest zwracany przez metodę `intern()` ze String Pool (gdzie są przechowywane unikalne elementy w celu oszczędzenia pamięci) zgodnie z tym co powyżej. Jednak w STACK (na stosie) dołożony mamy nowy obiekt "HelloWorld" z inną referencją (`new String("HelloWorld")`) i to on jest wykorzystany (LIFO). Finalnie więc referencje `string1` i `string3` są różne, wskazują różne miejsce w pamięci na "HelloWorld".

* bez Linia-2:
W przypadku gdy nie mamy linii:
```String string2 = new String("HelloWorld");          // Linia-2```
metoda `intern()` zwraca Stringa ze String Pool i referencje są identyczne.
