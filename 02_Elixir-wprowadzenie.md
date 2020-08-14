
![Elixir wallpaper](https://wallup.net/wp-content/uploads/2017/11/23/434614-code-elixir-programming-748x420.jpg)

# Czym jest Elixir, Erlang, i Phoenix

## Elixir

[Elixir](https://elixir-lang.org/) to dynamiczny, wspolbiezny, funkcyjny jezyk najlepiej nadajacy sie do pisania skalowalnych aplikacji, ktore -- dzieki architekturze jezyka -- sa latwe w utrzymaniu.  Skladnia Elixir jest silnie zainspirowana Ruby, ale sam jezyk wykorzystuje [Erlang VM (Virtual Machine) (maszyne wirtualna Erlanga)](https://en.wikipedia.org/wiki/BEAM_(Erlang_virtual_machine)), i pozwala na pisanie rozproszonych aplikacji o niskich czasach oczekiwania oraz wysokiej odpornosci na awarie (dzieki systemowi supervisorkow, o ktorych pozniej). Jezyk powstal w 2011 z reki José Valim czerpiac inspiracje z jezykow takich jak Erlang, Ruby, oraz Clojure. Elixir wykorzystywany jest do pisania aplikacji sieciowych, APIs, przetwarzania danych, i swietnie nadaje sie do internetu rzeczy ([IoT (Internet of Things)](https://en.wikipedia.org/wiki/Internet_of_things))

## Erlang

[Erlang](https://pl.wikipedia.org/wiki/Erlang_(j%C4%99zyk_programowania)), tak jak Elixir, jest dynamicznym, wspolbieznym, funkcyjnym jezykiem. Jezyk zostal zaprojektowany przez Joe Armstronga w 1986 roku pracujacym w firmie Ericsson. Stal sie wolnym oprogramowaniem w 1998.

## Phoenix

[Phoenix](https://www.phoenixframework.org/) jest najbardziej popularnym frameworkiem w ekosystemie Elixir. Framework jest oparty o zasady MVC (Model-View-Controller), pozwala na pisanie aplikacji sieciowych wykorzystujacych protokoly HTTP i WebSocket, APIs, oraz interaktywnych aplikacji bez potrzeby uzycia JavaScriptu ([LiveView](https://www.youtube.com/watch?v=U_Pe8Ru06fM)).

## Supervisors

System supervisorkow (nadzorcow) czyli supervisor tree pozwala na restartowanie tych procesow, ktore przestaly dzialac. Jako, ze aplikacje Elixirowe skladaja sie z setek, a nawet tysiecy malych procesow, supervisorki gwarantuja, ze aplikacje wirtualnie nigdy nie przestana dzialac.  
Ten warsztat skupia sie na podstawach Elixir i Phoenix, wobec czego nie bedziemy sie supervisorami zajmowac, ale warto wiedziec o ich istnieniu. Wiecej na ich temat mozesz znalezc w [dokumentacji Elixir](https://elixir-lang.org/getting-started/mix-otp/supervisor-and-application.html).



# Istotne koncepty funkcyjnego programowania używane w Elixir

## Funkcje i moduly 

Elixirowe aplikacje zbudowane sa wylacznie z funkcji, ktore pogrupowane sa w modulach. Prosty przyklad:

```elixir
defmodule MyFirstModule do

  def my_first_function() do
    IO.puts "Hello world!"
  end

end
```

`defmodule` tworzy nowy modul, natomiast `def` nowa funkcje. Nazwy modulow zaczynaja sie z duzej litery i trzymaja sie konwencji *camel case*, natomiast nazwy funkcji z malej, przy konwencji *snake case*.  
Funkcja zawsze zwraca wartosc z ostatniej linijki.

## Pattern matching

Pattern matching jest jedna z najsilniejszych stron Elixir, i temat jest dosc obszerny, ale dzis skupimy sie jedynie na pattern matching, ktorego bedziemy konkretnie uzywali w Phoenix'owej aplikacji typu CRUD. Elixirowa dokumentacja ma sporo informacji na temat pattern matching. [[link]](https://elixir-lang.org/getting-started/pattern-matching.html)

W kontrolerach (controller), ktore bedziemy pisali w Phoenix bardzo czesto pojawi sie ten wzor:

```elixir
def one(conn, %{"id" => id}) do
  # jakis kod, ktory wykorzystuje id z poprzedniej linijki
end
```

Alternatywnie do `%{"id" => id}` moglibysmy napisac `params` (i pozniej uzywac `params.id`) lub `%{"id" => id} = params` (ta opcja pozwala uzywac zarowno `id`, jak i `params`).  
Dlaczego piszemy argument funkcji `one` w ten sposob? To wlasnie jest pattern matching. Jesli wewnatrz `params` nie bedzie string `"id"`, to funkcja `one` nie zostanie przez Elixir uzyta przy probie jej wywolania.  
Co ciekawsze, mozemy funkcje `one` napisac wiele razy, za kazdym razem uzywajac innej wersji do innego wzoru, ktory match'ujemy.

```elixir
def one(conn, %{}) do
  # jakis kod, ktory zadziala tylko jesli pusta mapa (%{}) zostanie przeslana przy wywolaniu funkcji
end

def one(conn, %{"id" => id}) do
  # jakis kod, ktory wykorzystuje id z poprzedniej linijki
end
```

W ten sposob, jesli `params` bedzie pusta mapa (`%{}`) wywolana zostanie pierwsza wersja funkcji `one`. Natomiast, jesli w `params` bedzie klucz `"id"`, to zostanie wywolana wersja druga.  
Notka: jesli druga wersja `one` otrzymalaby mape `%{"id" => 44, "name" => "Messiah"}` to funkcja zadziala prawidlowo, pomimo tego, ze nie pattern match'ujemy klucza `"name"`.

## Pipelines

Napisze to tylko raz: pipelines are fun! 😁  
Pipelines (tudziez, pipe operator: `|>`) to jedna z moich ulubionych cech funkcyjnych jezykow, ktora pozwala nam *pipe'owac* dane z funkcji do funkcji, transformujac je.  
  
Zobaczmy jak to wyglada w praktyce:

```elixir
"Yeti"
|> String.upcase                    # "YETI"
|> String.splitter("", trim: true)  # Stream, z ktorego musimy wziac tylko jakas czesc
|> Enum.take(4)                     # ["Y", "E", "T", "I"]
|> List.to_tuple                    # {"Y", "E", "T", "I"}
|> is_tuple                         # true
```

Proste i przejrzyste :)  
Pipe operator mozemy uzywac w dowolnej ilosci, a typowe uzycia, ktore czesto nie wychodza poza 2x sa zazwyczaj latwe do zrozumienia.

## Wspolbieznosc (Concurrency)

Nie bedziemy sie dzis zajmowac concurrency per se, ale dobrze o niej wiedziec, bo to jedna z silnych stron Elixir (i Erlang). Najprosciej rzecz ujmujac, aplikacje zbudowne w Elixir skladaja sie z wielu procesow, a model procesow to model Aktorow (Actor model), ktory pozwala odizolowanym procesom komunikowac sie ze soba za pomoca wiadomosci (messages). Procesy moga sie *linkowac* oraz monitorowac, a czestym sposobem ich uzycia sa GenServers (Generic Servers), ktore dzialaja na zasadzie petli oraz zestawu funkcyji typu callback. Za kazdym razem, gdy callback jest wywolany, stan GenServera moze zostac update'owany, a wysylajacy moze otrzymac jakas wiadomosc (np. potwierdzenie jakiegos dzialania).  
  
Wiecej na temat concurrency i GenServers [tutaj](https://elixirschool.com/en/lessons/advanced/concurrency/).


## Funkcje wyzszego rzedu (Higher order functions)

Choc nie ekskluzywne do jezykow funkcyjnych, funkcje wyzszego rzedu sa zawsze ich czescia, i bedziemy ich uzywac sporo (choc niekonieczne dzis 😅).  
  
Typowymi funkcjami wyzszego rzedu (a wiec takimi, ktore jako parametr przyjmuja inna funkcje) sa `map` (ktora transformuje kazdy element w liscie), `filter` (ktory zwraca tylko czesc listy), oraz `reduce` (ktora redukuje liste).

```elixir
#map
Enum.map [1,2,3], (fn n -> n * n)                 # [1, 4, 9]

#filter
Enum.filter [1,2,3], (fn n -> n > 1)              # [2, 3]

#reduce
Enum.reduce [1,2,3], 0, fn n, acc -> n + acc end  # 6

```

Te funkcje zawsze biora liste oraz funkcje, ktora zadziala dla kazdego elementu listy (jest to forma *loopowania* w jezykach funkcyjnych, za ktorymi stoi [tail recursion](https://en.wikipedia.org/wiki/Tail_call), czyli funkcja, ktora wywoluje sama siebie; tail recursion ([rekurencja ogonowa](https://pl.wikipedia.org/wiki/Rekurencja_ogonowa)) jest zazwyczaj nielatwa do zrozumienia dla nowicjuszy, i nie bedziemy jej sami dzis pisali, wobec tego temat pozostawie przy tej krotkiej notce; fajnie, ze bedziecie wiedziec o jej istnieniu, ale na potrzeby dzisiejszej aplikacji, nie musimy sie w temat dalej zaglebiac 😁).


## Niezmiennosc danych (Immutability of data)

Jedna z najprzyjemniejszych cech funkcyjnego jezyka jest niezmiennosc danych (immutability). Elixir zamiast zmieniac dane, kopiuje je, transformuje, a nastepnie zwraca. W Elixir, co prawda, mozliwym jest zapisanie innych danych do zmiennej o tej samej nazwie, ale alokacja w pamieci nie zezwala na mutacje danych (a wiec dwa razy zmienna `x` bedzie zapisana w innych lokacjach pamieci). Wezmy prosty przyklad:

```elixir
x = 1      # 1
y = x + 1  # 2
x = 2      # 2
z = x + 1  # 2
y          # 2
```

Co zas sie tyczy zwracania skopiowanych danych, to swietnie mozna to zrozumiec na przykladzie listy lub mapy.

```elixir
list = [1,2,3]
[0 | list]  # kompletnie nowa lista, ktora wyglad tak: [0,1,2,3]

map = %{id: 1}
%{map | id: 2} # kompletnie nowa mapa: %{id: 2}
```

Warto pamietac, ze w powyzszym przykladzie, zmiana zarowno `list`, jak i `map` nie zmienila w zaden sposob oryginalnych wartosci zmiennych.


## Deklaratywny kod

W odroznieniu od jezykow imperatywnych, Elixir (podobnie jak inne funkcyjne jezyki) 
jest jezykiem deklaratywnym. Jaka jest roznica pomiedzy imperatywnym a deklaratywnym (imperative vs declarative)?
Najprosciej rzecz ujmujac, imperatywny jezyk zmusza programiste do pisania w stylu JAK (HOW). Natomiast deklaratywny
jezyk pozwala na pisanie w stylu CO (WHAT). A wiec nie "jak mam to zrobic?", ale "co mam zrobic?" (gdyby komputer nas pytal 😉).  
  
Wysmienitym przykladem jest uzywanie funkcji `Enum.map`, ktora jest deklaratywna alternatywa dla petli typu for loop w jezykach imperatywnych, i
wyglada nastepujaco (drugi argument to anonimowa funkcja (anonymous function)):

```elixir
Enum.map [1,2,3], fn number -> number * number end   # [1,4,9]
```

`Enum.map` bierze liste liczb (`[1,2,3]`), i aplikuje anonimowa funkcje do kazdej z nich. Dla celow wylacznie edukacyjnych, tu jest kod, ktory dokona tego samego:

```elixir
def dbl_number(number), do: number * number

[dbl_number 1, dbl_number 2, dbl_number 3]  # [1,4,9]
```

Nie interesuje nas JAK `Enum.map` zmienia kazdy element podanej listy. Obchodzi nas jedynie, CO ma zostac zrobione.  
  
Deklaratywny styl pozwala na pisanie mniejszej ilosci kodu, ktory w dodatku jest bardziej przejrzysty.




# Typy danych

## Atom

Atom to typ danych, ktorego wartoscia jest jego wlasne imie.

```elixir
:hello
:world
:ok
:batman
:my_name_is_wayne
:bruce_wayne
```

Czesto uzywanymi atoms sa `:ok` oraz `:error`, szczegolnie przy uzyciu pattern-matching.


## String

String pozwala nam napisac jakis ciag znakow, ktory bierzemy w cudzyslow (np. `"hello"`).

```elixir
greeting = "Hello world!"
```

Strings mozemy laczyc (concatenate) za pomoca funkcji `<>`.

```elixir
def greeting(name), do: "Hello " <> name <> "!"
```

Poniewaz funkcja `<>` zyje sobie w module [`Kernel`](https://hexdocs.pm/elixir/1.3.3/Kernel.html#%3C%3E/2), mozemy jej uzyc w nastepujacy sposob:

```elixir
Kernel.<>("h", "i")  # "hi"
``` 


## Integer

Integer to liczba calkowita.

```elixir
4
```

Integer poddaje sie typowym dzialaniom matematycznym.

```elixir
4 + 4
4 - 4
4 * 4
4 / 4
```

Poniewaz `+`, `-`, etc. to w Elixir funkcje (`Kernel.+`, `Kernel.-`, etc.), mozemy tez uzywac je w ten sposob:

```elixir
Kernel.+ 4, 4
Kernel.- 4, 4
```


## Float

Float to liczba zmiennoprzecinkowa.

```elixir
4.5
```

Podobnie jak integer, float poddaje sie typowym dzialaniom matematycznym.

```elixir
4.0 + 4.5
4.0 - 4.5
4.0 * 4.5
4.0 / 4.5
```


## Boolean

Boolean to typ danych, ktory ma tylko dwie mozliwe wartosci: `true` (prawdziwa) lub `false` (nieprawdziwa).

```elixir
1 == 1           # true
"hi" == "hi"     # true
1 == 2           # false
"hi" == "hello"  # false

[] == []         # true
%{} == %{}       # true
```

W rzeczywistosci, w Elixir booleans `true` i `false` to atoms `:true` i `:false`.

```elixir
true == :true    # true
false == :false  # true
```


## Map

Map to typ danych, w ktorym trzymamy dane w formie par klucz=>wartosc (key=>value pairs). Mapy tworzymy za pomoca skladni `%{}`. Kluczem w mapach moga byc rozne typy danych: `String`, `Integer`, `Float`, `Atom`, `List`, `Map`, `Boolean`. Klucz od wartosci oddzielamy z pomoca `=>`. Natomiast, pary oddzielamy z uzyciem `,`.

```elixir
%{}  # pusta map
%{1984 => "George Orwell"}
%{[1,2,3] => :three_numbers}
%{"title" => "Citizen Kane", "year" => 1941, "players" => ["Orson Welles", "Joseph Cotten", "Dorothy Comingore"]}
```

Poniewaz Elixir jest dynamicznym jezykiem, klucze w mapach moga sie roznic pod wzgledem typu danych.

```elixir
%{"name" => "Alchemist", 33 => :age, [false, false] => [:married, :loves_oop]}
```

Co interesujace, jesli kluczem jest atom, Elixir uzywa odrobine innej skladni.

```elixir
%{title: "The Assassination of Julius Caesar", year: 2017, band: "Ulver", label: "House of Mythology"}

%{name: "Joseph K"} == %{:name => "Joseph K"}  # true
```

Bedziemy wykorzystywac mapy czesto do gromadzenia danych dotyczacych np. uzytkownikow, postow, czy przedmiotow w naszej aplikacji.

```elixir
users = 
    [ %{id: 0, name: "Elmo"}
    , %{id: 1, name: "Kermit"}
    , %{id: 2, name: "Yoda"}
    ]
```

W jaki sposob mozna dobrac sie (😁) do danych wewnatrz map? Innymi slowy: getter.

```elixir
person = %{ name: "Joe", age: 53 }

person.name    # "Joe"
person[:name]  # "Joe"
person.age     # 53
person[:age]   # 53

person2 = %{ "name" => "Joe", "age" => 53 }

person2["name"]  # "Joe"
person2["age"]   # 53
```

A w jaki sposob mozemy cos zmienic w mapie? Innymi slowy: update lub setter. Sluzy do tego specjalny syntax: `{ map | field => new_value, field2 => new_value, etc. }`

```elixir
updated_person = 
    { person | name: "Jane", age: 25 }  
    # jesli chcemy update'owac wiecej niz jedno pole (key), mozemy je oddzielic przecinkami

-- teraz zobaczmy, co jest wewnatrz updated_person
updated_person.name  # "Jane"
updated_person.age   # 25
```

Nie tylko, ze nasz Joe zmienil plec, to jeszcze odmlodnial o 28 lat!! 😁  
  
Mozemy rowniez uzywac specjalnych funkcji z modulu `Map` (`get` oraz `put`).

```elixir
Map.get(person, :name)     # "Joe"
Map.put(person, :age, 54)  # %{ name: "Joe", age: 54 }
Map.put(person, :langs, ["Elixir", "Elm"])  # %{ name: "Joe", age: 54, langs: ["Elixir", "Elm"] }
```

Zauwaz, ze `Map.put/3` pozwala na dodanie nowego klucza do mapy, w trakcie gdy skladnia `{map | key: value}` pozwala jedynie zmienic wartosc juz istniejacego klucza.

## Tuple

Tuples sa bardzo podobne do list, z ta roznica, ze tuple (gdy modyfikowany) jest w calosci kopiowany. To sprawia, ze czytanie tuples jest tanie, ale ich zmiana jest droga. Z tego powodu tuples sa najczesciej uzywane do komunikowania statusu i zwracania informacji kiedy funkcja moze zwrocic blad (zazwyczaj robiac cos w chaotycznym zewnetrznym swiecie, np. proszac o cos inny serwer albo robiac cos w lokalnej bazie danych).

```elixir
# prosty tuple
{:say, "Hi", true, 69}

# typowy przyklad z pracy z Ecto (ORM pomiedzy Phoenix a baza danych)
case MyRepo.insert %Post{title: "Ecto is great"} do
  {:ok, struct}       -> "Inserted with success"
  {:error, changeset} -> "Something went wrong"
end
```

Powyzszy wzor jest bardzo typowy. Funkcja albo zwroci `{:ok, ...}` albo `{:error, ...}`. Wykorzystanie `case ... do` pozwala nam na pattern matching tego, co zwroci funkcja `MyRepo.insert`. Wiecej na temat Ecto nauczymy sie budujac CRUD-owa aplikacje.

## List

List pozwala nam zgromadzic jakies wartosci. Np. w naszej aplikacji sieciowej typu CRUD bedziemy pracowali z listami map.

```elixir
strings = ["every", "thing", "is", "a", "function", ":)"]

bools = [true, true, false]

ints = [1,2,3]

floats = [1.0, 2.1, 3.2, 4.8]

# Elixir jest dynamiczny, wobec czego mozemy mieszac rozne typy danych
mixed = ["String", :atom, true, 1337, 3.14, {:tuple, :of, :four, :elements}]

# lista map
people =
    [ %{ id: 0 , name: "Hideto Matsumoto", tool: "guitar" },
      %{ id: 1 , name: "Richard Wright", tool: "keyboard"}
    ]
```

Nie bedziemy dzis modelowac danych w ten sposob, ale warto wiedziec, ze mozna miec liste zlozona z wielu list :-)

```elixir
list_of_lists = [ [1,2,3,4], [5,6], [7,8,9] ]
```



## Funkcje nazwane i funkcje anonimowe

### Nazwane

W Elixir "wszystko" jest funkcja. Dlatego tez duzo czesciej bedziemy uzywali nazwanych funkcji, anizeli anonimowych.

Nazwane funkcje maja nastepujacy syntax:
```elixir
def nazwa_funkcji(argument1, argument2, ...) do
  # kod wewnatrz funkcji (tutaj mozemy uzywac argumentow)
end
```

lub skrocona wersja:

```elixir
def nazwa_funkcji(argument1, argument2, ...), do: kod wewnatrz funkcji (tutaj mozemy uzywac argumentow)
```

Kilka prostych zasad:
1. Nazwa funkcji musi byc w formie snake_case (tzn. `to_jest_moja_funkcja`).
2. Argumenty sa opcjonalne, i moze ich byc wiele.
3. Ostatnia linijka wewnatrz funkcji zwraca jakas wartosc (return)

Ogolna zasada funkcyjnego programowania jest to, ze lubimy miec duzo malych funkcji, z ktorych zbudowany bedzie nasz program. Male funkcje sa latwiejsze do zrozumienia i testowania (zwlaszcza przy unit testing) 😃

Przyklad tworzenia i uzywania funkcji, oraz uzywania funkcji z Elixir standard library.

```elixir
defmodule ToyingAround do

    # 0-argumentowa funkcja
    def empty_user do
        %{ id: 0 , name: "Jack", trades: ["all"] }
    end

    # 2-argumentowa funkcja
    def add_two_nums(num1, num2), do: num1 + num2

end

# uzycie
ToyingAround.add_two_nums 13, 12  # 25
```

Uzywanie funkcji z [Elixir standard library](https://hexdocs.pm/elixir/Kernel.html#content) wyglada podobnie.

```elixir
# moduly zaczynaja sie z wielkiej litery, i poprzedzaja nazwe funkcji
String.length "Zool"  # 4
```


### Anonimowa

Anonimowe funkcje wygladaja nastepujaco:

```elixir
fn number -> number + 1 end
```

W powyzszym przykladzie funkcje zaczynamy od `fn`, pozniej `number` jest argumentem funkcji, po ktorym musi  nastapic symbol strzalki (`->`), a nastepnie kod, ktory moze uzyc argumentu `number` i zwrocic jakas wartosc (w naszym przypadku `number + 1`).

Anonimowe funkcje sa najczesciej uzywane przy dzialaniach na listach.

```elixir
Enum.map [1,2,3], fn number -> number + 1 end  # [2,3,4]
```

## Conditionals oraz pattern matching

Conditionals w Elixir mozemy pisac na trzy sposoby: 
1. `case`
1. `cond`
1. `if` oraz `unless`

### Case

Prosty przyklad:
```elixir
data = %User{"username" => "Winnie the Pooh", "occupation" => "taoistic wiseman"}
response = Repo.insert(data)   # wkladamy dane do bazy danych, i dostajemy odpowiedz
case response do
  {:ok, struct}       -> "Sukces!"         
  {:error, changeset} -> "Oopppssss..."    
  # struct to zwrocone dane, ktore wlasnie wsadzilismy do bazy danych
  # changeset ma w sobie klucz errors, z ktorego mozemy wydobyc http status oraz wiadomosc
end
```

`case` pozwala nam uzyc jakiegos kodu w zaleznosci od tego, ktora z galezi (`->`) zostanie aktywowana. Sprobujmy zobaczyc prostszy przyklad:

```elixir
case true do
  true -> 1
  false -> 0
end

# rezultat: 1
```

Mozemy czasem pattern matchowac na string albo int albo float, co daje nieskonczone mozliwosci. W takim wypadku, bedziemy uzywac `_` dla powiedzenia Elixirowi "dla wszystkich innych, uzyj tej galezi".

```elixir
case 69 do
  1984 -> "Blair or Well"
  69 -> "yin yang"
  _ -> "Nothing here..."
end

# rezultat: "yin yang"
```

## Cond

`cond` uzywamy kiedy chcemy sprawdzic przeciwko roznym *conditions*.

```elixir
n = 5

cond do
  rem(3,n) == 0 -> "Fizz"
  rem(5,n) == 0 -> "Buzz"
end
```

## If oraz unless

Dosc standardowo, mozemy rowniez uzywac `if ... else` lub `unless`.

```elixir
if true do
  :big
else
  :bang
end

unless true do
  :theory
end
```



# Mix CLI i praca z konsola/terminalem

## Podstawowe komendy

- `mix new` -- tworzy nowy projekt. Dodanie flagi `--sup` stworzy projekt z drzewkiem supervisorowym. 

- `mix deps.get` -- sciaga *dependencies* projektu.

- `mix compile` -- kompiluje Elixirowy projekt do plikow `.beam`.

- `mix test` -- uruchamia testy zyjace w folderze `test` projektu.

- `mix` -- uruchamia projekt.

- `iex` -- interaktywny Elixir (REPL).

- `iex -S mix` -- interaktywny Elixir (REPL) w polaczeniu z dzialajacym projektem. Pozwala wywolywac funkcje z modulow projektu. Np. `iex(1)> MyFirstExApp.hello()` (rezultat: `:world`). 
  
## Phoenix

- `mix phx.new` -- tworzy nowy projekt Phoenix. (Sprawdz `mix help phx.new` dla roznych opcji.)

- `mix phx.server` -- uruchamia serwer Phoenix, ktory domyslnie slucha na `localhost:4000`.

- `iex -S mix phx.server` -- uruchamia serwer Phoenix oraz interaktywny Elixir.

- `mix ecto.create` -- tworzy baze danych (domyslnie Postgresql).

- `mix ecto.migrate` -- odpala migracje.

- `mix ecto.reset` -- czysci baze danych, tworzy nowa, odpala migracje oraz `seeds.exs`.

- `mix phx.gen.html` -- generuje controller, views, oraz context dla HTML. 
  
- `mix phx.routes` -- wyswietla liste routes projektu.

## Pomoc 

Jesli potrzebujesz dodatkowych informacji, smialo skorzystaj z pomocy: `mix help`. Informacje na temat konkretnej komendy mozesz otrzymac np. tak: `mix help phx.new`.


# Pierwsza aplikacja

## Hello

1. Zacznijmy od stworzenia nowego projektu. Nazwiemy go `hello_world` (wygenerowany modul bedzie nazywal sie `HelloWorld`). W consoli/terminalu uzyjmy nastepujacej komendy: `mix new hello_world --sup` (`--sup` stworzy dla nas supervisorka).
1. Po stworzeniu nowego projektu, Elixir zacheci nas do przejscia do katalogu projektu i odpaleniu testow. Sprobujmy :) `cd hello_world`, a potem `mix test`. Elixir stworzyl dla nas troche plikow, modul `HelloWorld`, funkcje `hello` (ubiegajac nas!), i test dla tej funkcji. Modul znajdziemy w `lib/hello_world.ex`, a testy w `test/`.
1. Sprawdzmy jeszcze `lib/hello_world/application.ex`, w ktorej zyje nasz supervisor.
1. Teraz, napiszmy pierwsza funkcje, ktora zwroci `Hello, John!` jesli argument dla niej bedzie `John`.
1. W `lib/hello_world/hello_world.ex` zrobmy tak: 
```elixir
defmodule HelloWorld do

  ...

  def hello(name) do
    "Hello, " <> name <> "!"
  end

end
```

6. Teraz, odpalmy nasza mala appke wraz z interaktywnym Elixirem: `iex -S mix`
6. Nastepnie, wpiszmy `HelloWorld.hello("John")`. Iex powinien zwrocic `"Hello, John!".`
6. Sprobujmy tez `HelloWorld.hello`, ktore zwroci `:world`. W Elixir mozemy miec kilka funkcji o tej samej nazwie, ktore moga przyjmowac rozne ilosci argumentow albo pattern matchowac na inne wartosci.
6. Swietnie, mamy malenka aplikacje :-] Teraz zerknijmy do testow. Co prawda na potrzeby tego warsztatu nie bedziemy pisac testow w naszej Phoenix'owej aplikacji CRUD, ale warto o nich wspomniec. Testowanie w Elixir jest proste i przyjemne.
6. Testy zyja sobie w `test/hello_world_test.exs`, ktore obecnie wyglada tak:
```elixir
defmodule HelloWorldTest do
  use ExUnit.Case
  doctest HelloWorld

  test "greets the world" do
    assert HelloWorld.hello() == :world
  end
end
```
11. Dodajmy nowy test:
```elixir
defmodule HelloWorldTest do

  ..

  test "greets a person" do
    assert HelloWorld.hello("John") == "Hello, John!"
  end

  test "Fails to greet a person" do
    assert HelloWorld.hello("") == "Hello, !"
  end
end
```
12. `mix test`, zeby odpalic testy. Powinnismy zobaczyc `1 doctest, 3 tests, 0 failures`.


# Architektura MVC (Model-View-Controller)

![MVC](https://image.slidesharecdn.com/modelviewcontrollermvc-140211001124-phpapp01/95/model-view-controller-mvc-6-638.jpg?cb=1392077579)

Phoenix framework wykorzystuje architekture [Model-View-Controller](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) [[pl]](https://pl.wikipedia.org/wiki/Model-View-Controller).  
  
W Phoenix wykorzystanie MVC wyglada nastepujaco:
1. Kazdy request od usera/innego-zrodla zaczyna sie od `Endpoint`. W swojej aplikacji znajdziesz modul `MyAppWeb.Endpoint` (w katalogu `/my_app/lib/my_app_web/`), w ktorym znajduje sie cala lista `plugow` (plugi to taki Elixirowy *middleware* (albo zwyczajnie funkcje, przez ktore request musi przejsc zanim dotrze do routera)), przez ktore request musi najpierw przejsc.
1. Nastepnym krokiem jest `Router`, ktory tez ma w sobie rozne `plugi`, i zadecyduje o przeslaniu request do odpowiedniej funkcji w jakims kontrolerze. Przyklad:
```elixir
get "/", PageController, :index
```
`get` to funkcja dla HTTP metody 'GET'. `"/"` to root route. `PageController` to modul, do ktorego przekazany zostanie request. `:index` to funkcja w module ,ktora bedzie wygladala tak: 

```elixir
def index(conn, params) do 
  ... 
end
```

3. W koncu, request trafi do powiedniej funkcji w podanym module, gdzie cos sie wydarzy (zapis do bazy danych, req/res do innego serwera, transformacja danych, itd.).
3. Ostatnia linijka funkcji bedzie zwroceniem jakiejs wartosci lub funkcji z funkcji. Np.:
```elixir
conn
|> status(200)
|> json(%{status: "success", msg: "Operacja w bazie danych zakonczyla sie pelnym sukcesem 😎"})
```

Zauwaz, jak poprzedni przyklad wykorzystuje pipe operator (`|>`). Magic! 😋  
  
Jezeli nie zwracamy JSON (co nalezy raczej do aplikacji typu API), to zwrocimy jakis template (czyli dynamiczny HTML):
```elixir
def index(conn, _params) do
  render(conn, "index.html")
end
```

Notka: `_params` nie jest uzyte w funkcji, wobec czego zamiast `params` uzywamy `_params`. Znak `_` oznacza, ze parametr nie zostanie uzyty w funkcji.  
Notka: Nieuzyte parametry, ktore nie maja przed soba `_` zostana wypomniane nam przez kompilator, ale jedynie w formie ostrzezenia (warning). Aplikacja skompiluje sie poprawnie.

# Aplikacja sieciowa typu CRUD (Create, Read, Update, Delete)

Wiemy juz jak stworzyc Elixirowa aplikacje (`mix new`), ale Phoenix ma swoj wlasny zestaw mix tasks.

1. Do stworzenia aplikacji uzyj `mix phx.new my_app`. W ten sposob dostaniesz appke o nazwie `MyApp`.
1. Potwierdz sciagniecie *dependencies*, i zrob sobie szybka kawe, bo sciaganie deps dla node packages moze troche potrwac 😅. W zaleznosci od Twojej *connection*, naturalnie.
1. OK. Kolejny krok to wejscie do katalogu appki i stworzenie bazy danych: `ecto.create`. Upewnij sie, ze Twoj Postresql dziala w tle. A jesli komenda nie zadziala, to sprawdz w `/confid/dev.exs`, czy dane do bazy danych sie zgadzaja.
1. Teraz odpal appke: `mix phx.server`
1. Reszte aplikacji skodujemy razem. Moja appke wrzuce na github w oddzielnej repo, a tutaj umieszcze link po zakonczeniu warsztatu 😇



# Uzyteczne linki

* [Elixir Guides](https://elixir-lang.org/getting-started/introduction.html)
* [Elixir Docs](https://elixir-lang.org/docs.html)
* [Phoenix Guides](https://hexdocs.pm/phoenix)
* [Phoenix Docs](https://hexdocs.pm/phoenix/Phoenix.html)
* [Elixir Slack](https://elixir-slackin.herokuapp.com/)
* [Elixir School [en]](https://elixirschool.com/en/)
* [Elixir School [pl]](https://elixirschool.com/pl/)
* [Awesome Elixir](https://gitlab.com/kidakadeshia/awesome-elixir)
* [Functional Works: Elixir Jobs](https://functional.works-hub.com/elixir-jobs)
