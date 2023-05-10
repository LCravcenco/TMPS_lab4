# Chain of Responsibility
Chain of Responsibility este utilizat în clasele SearchHandler, TitleSearchHandler și AuthorSearchHandler. Aceste clase definesc un lanț de responsabilitate pentru căutarea cărților în bibliotecă în funcție de titlu și autor.
	
Clasa SearchHandler este clasa de bază pentru fiecare handler în lanțul de responsabilitate. Aceasta conține o referință către următorul handler din lanț și metoda search(), care, în cazul în care există un următor handler, îi va cere să efectueze căutarea.
	
Clasele TitleSearchHandler și AuthorSearchHandler sunt derivate din clasa SearchHandler. Fiecare dintre ele implementează o strategie de căutare diferită pentru a filtra cărțile în funcție de titlu sau autor.

În clasa BookCollection, sunt create instanțe ale TitleSearchHandler și AuthorSearchHandler și setate în lanțul de responsabilitate prin metoda setNext().

Astfel, prin utilizarea pattern-ului Chain of Responsibility, se realizează o descompunere eficientă a responsabilităților, permitând fiecărui obiect să se concentreze pe sarcina sa specifică, fără a fi nevoie să știe ce obiecte există înaintea și după el în lanțul de responsabilități.
```
class SearchHandler {
    constructor() {
        this.nextHandler = null;
    }

    setNext(handler) {
        this.nextHandler = handler;
    }

    search(books, query) {
        if (this.nextHandler) {
            return this.nextHandler.search(books, query);
        }
        return [];
    }
}

class TitleSearchHandler extends SearchHandler {
    search(books, query) {
        const result = books.filter(book => book.title.toLowerCase().includes(query.toLowerCase()));
        return result.length ? result : super.search(books, query);
    }
}

class AuthorSearchHandler extends SearchHandler {
    search(books, query) {
        const result = books.filter(book => book.author.toLowerCase().includes(query.toLowerCase()));
        return result.length ? result : super.search(books, query);
    }
}
```
# Command
Command are o metodă execute() care este responsabilă pentru executarea acțiunii dorite. Clasa de comandă este apoi transmisă unui obiect Executor, care poate fi o clasă specializată (în cazul nostru, o clasă BookCollection), pentru a executa acțiunea specificată. Acest proces este posibil datorită faptului că obiectul Executor are o metodă execute() corespunzătoare, care va fi apelată atunci când se face apel la metodă execute() pe obiectul de tip comandă.

În aplicație există două clase de comandă definite - AddBookCommand și DeleteBookCommand - care sunt utilizate pentru a adăuga și a șterge cărți din BookCollection. Obiectul Executor este clasa Library, care execută comenzile în numele BookCollection.

O altă caracteristică importantă a modelului Command este faptul că aceste comenzi pot fi stocate și apoi executate mai târziu, ceea ce poate fi foarte util în anumite scenarii, cum ar fi executarea mai multor acțiuni simultan sau reversarea acțiunilor efectuate.
	
```
class AddBookCommand {
    constructor(book) {
        this.book = book;
    }

    execute(bookCollection) {
        bookCollection.addBook(this.book);
    }
}

class DeleteBookCommand {
    constructor(index) {
        this.index = index;
    }

    execute(bookCollection) {
        bookCollection.books.splice(this.index, 1);
    }
}
```
În concluzie, modelul de proiectare Command este o modalitate eficientă și flexibilă de a separa acțiunile și comenzi din aplicație, permițând astfel crearea unor aplicații mai scalabile și mai ușor de întreținut.
# Iterator
Paternul Iterator este implementat în clasa Iterator care primește colecția de elemente și reține indexul curent în această colecție. Această clasă oferă două metode publice: hasNext(), care returnează adevărat dacă mai există un element în colecție și next(), care returnează elementul curent și trece la următorul.

Clasa BookCollection utilizează această clasă pentru a oferi o interfață de iterare asupra cărților sale. Metoda getIterator() din clasa BookCollection creează și returnează un nou obiect de tip Iterator pentru colecția sa de cărți. Aceasta înseamnă că clientul poate utiliza metodele hasNext() și next() ale obiectului Iterator pentru a parcurge fiecare carte din colecție, indiferent de implementarea exactă a clasei BookCollection.
	
În esență, iteratorul ne permite să separăm logica de parcurgere a elementelor de implementarea concretă a colecției, ceea ce face ca codul nostru să fie mai modular și mai ușor de întreținut.
```
class Iterator {
    constructor(books) {
        this.books = books;
        this.index = 0;
    }

    hasNext() {
        return this.index < this.books.length;
    }

    next() {
        return this.hasNext() ? this.books[this.index++] : null;
    }
}
```
# Interpreter
Paternul Interpreter este utilizat în clasa QueryInterpreter.
	
Clasa QueryInterpreter conține un obiect handlers care are ca proprietăți două instanțe ale clasei SearchHandler: title și author. Acestea sunt folosite pentru a căuta în funcție de câmpul selectat de căutare, adică titlu sau autor.

Metoda interpret primește un string query, împarte query-ul în cuvinte separate prin spații și extrage primul cuvânt, care este numele câmpului de căutare. Toate celelalte cuvinte sunt concatenate împreună pentru a forma termenul de căutare. Apoi, caută în obiectul handlers pentru a verifica dacă există un obiect care corespunde numeleui câmpului. Dacă există, va apela metoda search a obiectului găsit și va returna rezultatul căutării. În caz contrar, se va arunca o eroare pentru a indica faptul că este un câmp de căutare invalid.

Apoi, când utilizatorul introduce o interogare de căutare în formularul de căutare, clasa QueryInterpreter este utilizată pentru a interpreta interogarea și a obține rezultatele corecte. Aceste rezultate sunt afișate apoi într-o listă de rezultate.
```
class QueryInterpreter {
    constructor() {
        this.handlers = {
            title: new TitleSearchHandler(),
            author: new AuthorSearchHandler(),
        };
    }

    interpret(query) {
        const tokens = query.split(' ');
        const field = tokens[0];
        const searchTerm = tokens.slice(1).join(' ');

        if (this.handlers[field]) {
            return this.handlers[field].search(library.bookCollection.books, searchTerm);
        } else {
            throw new Error(`Invalid query field: ${field}`);
        }
    }
}
```