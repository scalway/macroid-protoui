macroid-protoui
=======================

this file will be translated very soon, be patient.

Podstawowy scenariusz
-----------------------

podstawowym założeniem tej biblioteki jest uproszczenie do minimum sposobu tworzenia widoków na androidzie. 
Aby utworzyć reprezentacje (Ui[View]) np. dla klasy Person należy napisać jedynie:

val personView:ViewOf[Person] = ViewOf[Person]

W ten sposób utworzyliśmy instancję klasy ViewOf[Person]. Możemy dostać się do jej widoku (_.ui lub _.view) lub specjalnego handlera (_.value) który pozwala ustawić jaką konkretne instancje klasy person wyświetlać.

oczywiście musi istnieć gdzieś instrukcja jak z klasy Person utworzyć View. Gdy spojrzymy na sygnature metody ViewOf[T].apply (użyliśmy jej powyżej) zobaczymy, iż posiada ona jako parametr domniemany, obiekt typu CanBeViewOf[Person]. Ten obiekt jest naszą instrukcją jak utworzyć widok dla klasy Person.  

def apply[T](implicit creator:CanBeViewOf[T], c: ActivityContext):ViewOf[T]

oznacza to, że aby powyższa linia się skompilowała potrzebujemy dostarczyć CanBeViewOf[T], co możemy zrobić:
- tworząc anonimowy obiekt dziedziczący po CanBeViewOf[T]: 
  implicit val `Person can be viewed` = new CanBeViewOf[Person] {
	def createView(trig: AbstractReactiveVarible[T])(implicit c: ActivityContext): Ui[View] = 
	w[TextView] <~ text("Hi")
  } 
 
- tworząc instancje klasy AutoView[T]:
  implicit val `Person can be auto viewed` = new AutoView[Person] ( implicit c => implicit t => 
		w[TextView] <~ react(p => text("Hi " + p.name))
  )
  
wytłumaczenia wymaga użyta metoda react(...). Pomaga ona w wiązani Tweaków z manipulatorami (np pozwala ustawić text widgetu tak by zawsze pokazywał imie osoby, nawet jeśli zmieni się ono w przyszłości). W sumie jest to jedna z najważniejszych właściwości biblioteki. 
	
pozostaje zapytać gdzie mamy je utworzyć?
- najprostszym rozwiązaniem jest utworzenie go w obiekcie towarzyszącym naszej klasy Person. Jest to rozwiązanie wygodne, gdyż to miejsce będzie przeszukiwane automatycznie (nie trzeba go importować).
  object Person {
	...
	implicit val `can be viewed` = ...
	...
  }
- w osobnym obiekcie np:
  object ViewBuilders {
	implicit val `person view` = AutoView[Person](...)
	implicit val `book view` = AutoView[Person](...) 
	...
  }

wtedy będziesz zmuszony zaimportować je w miejscu użycia ręcznie:
  import ViewBuilders._
	
-------------------
abstract
-------------------

Czym dokładnie jest instancja ViewOf[T]? Możemy traktować ją jako połączenie przepisu na widok i specjalnego manipulatora. 

Przepis na widok (Ui[View]) -  pozwala nam tworzyć instancje klasy View (uwaga: możemy stworzyć więcej niż jeden obiekt typu View, korzystając z pojedyńczej instancji ViewOf[T], ale zazwyczaj nie tego chcesz).

manipulator - pozwala zaaktualizować wszystkie widoki (View) stworzone za pomocą Ui[View]. gdy na manipulatorze wywołasz metodę update(personObj) wszystkie widoki odświerzą się aby poprawnie wyświetlić personObj.

------------------
praca z tablicami
------------------

Jeśli poprawnie zdefiniujesz CanBeViewOf[Person] w prezencie otrzymasz implementacje CanBeViewOf[Array[Person]] która wykorzysta twoją implementacje, aby utworzyć ListView. Oznacza to, że jeśli możesz zrobić:

  ViewOf[Person]

to możesz również:

  ViewOf[Array[Person]] 

Praca z tablicami jest jednak nieco bardziej skomplikowana niż praca z pojedyńczymi elementami i przydałby się nam bogatszy interface niż ten klasyczny. Zamiast ViewOf[List[Person]] możesz użyć ListOf[Person]. ListOf ma znacznie bogatszy interface od ViewOf i pozwala np. na zaaktualizowanie pojedyńczej komurki zamiast całej tablicy.

-----------------
więcej zmiennych
-----------------

możesz również użyć konstrukcji dla dwuch typów np:

	val view:ViewOf2 = ViewOf[Person, Style]

klasa ViewOf2 posiada dwa manipulatory do zarządzania niezależnie lewą i prawą zmienną. Wymaga:
	
	implicit val `person with style can be viewed` = new CanBeViewOf2 { ... }

lub

	implicit val `person with style can be auto viewed` = new AutoView2( (personRV, styleRV) => implicit c => ... )
	
------------------
a w przyszłości...
------------------

todo
