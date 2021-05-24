# The stuff Nestor doesn't tell you
Neste "resumo" vou esclarecer umas coisas importantes sobre Programação
Orientada a Objetos.


## Getters and Setters
O lixo dos getters/setters devem ser evitados ao máximo. Aliás, se a tua classe
for
```java
class Foo {
    private String bar;

    public String get_bar() {
        return bar;
    }

    public void set_bar(String bar) {
        this.bar = bar;
    }
}
```
O quão menos encapsulado ficava se `bar` fosse simplesmente `public`? É a mesma
coisa! Há duas principais razões para encapsulamento:

- Mudar a representação interna do objeto não devia ser uma breaking change que
implica mudar o código que o usa.
    - Se eu quiser mudar `Foo` para ter um `int` em vez de uma `String` vou ter
    de mudar o `getter` e o `setter`. Logo isto não está encapsulado. É uma
    breaking change.
- Impedir uso incorreto do objeto.
    - Nem preciso de explicar esta, se podes diretamente alterar os campos do
    objeto nada te impede de o fazer mal.

Efetivamente, este código é equivalente a `public String bar`. Mas deu mais
trabalho e não se ganhou nada.

Mas a solução é fazer tudo `public`? Não, é fazer coisas que fazem sentido e
mais nada.

Por exemplo, pegando no clássico `Point`.

```java
class Ponto {
    private int x;
    private int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
}
```

Qual parece ser o mais intuitivo de usar?
```java
public move(int x, int y) {
    this.x += x;
    this.y += y;
}
```
Ou

```java
public set_x(int x) {
    this.x = x;
}

public set_y(int y) {
    this.y = y;
}
```
?

Agora podias dizer "Mas e se eu quiser que o ponto passe a ter (x,y) coordenadas
e não quero fazer as contas para o mover ate ao sitio". E para isso relembro te
que tens um construtor: `p = new Point(x, y);`. Setters não servem para nada em
99% dos casos.

Claro que há situações em que um setter faz sentido, mas são raras e normalmente
é possível construir uma API melhor. Olhem para os métodos do `ArrayList`, por
exemplo. Quantos setters é que aquilo tem?

### Artigo
- [Getter and setters are evil][GettersSettersEvil]


## Herança
"Prefer composition over inheritance whenever possible"
    -- Some smart person probably.

Herança é considerado um dos maiores erros de programação orientada a objetos.
Eu podia falar aqui de todas as maneiras em que herança é má ideia, mas há gente
mais inteligente que eu que já falou sobre isso.

### Artigos
- [Why Extends is Evil][ExtendsisEvil]
- [Inheritance is a Procedural Technique for Code Reuse][InheritanceBad]


## Clones
Muito provavelmente já ouviram dizer que, quando querem retornar uma variável de
instância devem retornar um `.clone()` dessa variável de instância através de um
getter.

Isto previne que qualquer alteração que o caller faça à variável de
instância retornada não seja refletida no estado interno do objeto, mantendo
assim o objeto encapsulado. Esta técnica é um exemplo de um *software design*
denominado *[defensive programming][DefensiveProgramming]*.\
No entanto, usar clones, especialmente neste contexto, traz as suas desvantagens.

### Desvantagens
* **Desperdício de recursos:** sempre que queremos aceder à variável de
instância, estamos a criar uma nova cópia dessa variável, que gasta recursos.
Inicialmente, pode parecer um pequeno custo, mas pode sorrateiramente aumentar o
 consumo de memória (e tempo) de uma forma bastante significativa.\
Isto acontece, por exemplo, quando chamamos este tipo de getters num loop:
    ```java
    // A cloneable object that contains an ArrayList<String>.
    class Bar implements Cloneable {
        private ArrayList<String> strings;

        public Bar() {
            this.strings = new ArrayList<>();
        }

        public ArrayList<String> get_strings() {
            return (ArrayList<String>) this.strings.clone();  // inner clone
        }

        @Override
        public Bar clone() {
            try {
                var bar_clone = (Bar) super.clone();
                bar_clone.strings = (ArrayList<String>) this.strings.clone();
                return bar_clone;
            } catch (CloneNotSupportedException e) {
                throw new RuntimeException(e);  // hope for the best
            }
        }
    }

    // A Foo consists only of a Bar.
    class Foo {
        private Bar bar;

        public Foo() {
            this.bar = new Bar();
        }

        public Bar get_bar() {
            return this.bar.clone();  // outer clone
        }
    }

    // Takes a Bar and computes some result, but does not modify the Bar.
    void compute(Bar bar) { /* ... */ }

    // Calling code example:
    var all_foos = new ArrayList<Foo>(/* ... */);
    for (var foo : all_foos) {
        compute(foo.get_bar());  // each iteration will clone a Bar, which
    }                            // in turn will clone an ArrayList<String>
    ```

* **Dá origem a funções excessivamente complexas:** idealmente, o nome de uma
função deve descrever de forma clara e breve o seu propósito. `Foo.get_bar()`
está a sugerir ao caller que apenas vai *buscar* `bar`, mas na verdade está a
*criar* (uma cópia de) `bar`.\
Poderão argumentar "*Mas isso é encapsulamento! Não é suposto sabermos a origem
de `bar`, isso são detalhes de implementação que não deviam ser expostos!*".\
Ok, vamos assumir que são detalhes de implementação, e que não nos devemos
preocupar de onde vem `bar`; pode ou não ser uma cópia do original.\
Se, por acaso, o `bar` retornado for um clone, e o `.clone()` no `get_bar()`
lançar uma exceção, o que fazemos? Voltamos a lançar a exceção? Ignoramos a
exceção e escrevemos para um log?\
Efetivamente, uma implementação de `Foo.get_bar()` com base em `.clone()`s
introduz a necessidade de lidar com exceções. Uma função trivial como um getter
não devia criar esta responsabilidade no caller.\
Esta implementação de `get_bar()` é considerada uma
*[leaky abstraction][LeakyAbstractions]* -- a tentativa de abstração de
detalhes, neste caso a origem de `bar`, acaba por introduzir novos problemas
imprevistos. -- o tratamento de exceções.

* **Falsa imutabilidade**: Nós não estamos a fazer `.clone()` porque
queremos uma cópia, mas sim porque queremos garantir que a nossa variável de
instância não seja ilegalmente modificada pelo caller, ou, mais corretamente,
queremos indicar ao caller que não é suposto modificá-la.\
Imaginemos agora que `Bar` tem a seguinte função:
    ```java
    class Bar {
        public void clear_strings() {
            this.strings.clear();
        }
    }
    ```
    Esta função altera o estado interno de `Bar`. Agora, o método
`Foo.get_bar()` passou a retornar um objeto que permite modificações.\
Isto não é intuitivo.\
Estávamos a tentar transmitir ao caller que não é suposto modificar `Bar`, e
agora estamos a retornar um `Bar` que é modificável. Então agora podemos
modificar `Bar`? E será que estas alterações são refletidas no estado interno
de `Foo`?\
Teríamos de consultar a documentação de `Foo.get_bar()` para perceber o que está
realmente a acontecer.\
Não seria tão mais fácil se esta informação estivesse guardada no tipo retornado
pela `get_bar()`?

* **A mítica interface Cloneable**: Num mundo ~~perfeito~~ razoável, seria de
esperar que `Cloneable` fosse uma simples interface que contém a declaração de
um método chamado `.clone()`, *right*? Algo deste género:
    ```java
    public interface Cloneable {
        Object clone();
    }
    ```
    Assim, qualquer objeto que implementava esta interface tinha de definir um
método `clone()`.

    **NOPE!** Isto faz demasiado sentido.\
A interface `Cloneable` não declara nenhum método! A sua verdadeira
implementação é a seguinte:
    ```java
    public interface Cloneable {}
    ```
    Poderás pensar "*Espera aí, então isso significa que eu posso fazer com que
a minha class seja `Cloneable`, e não ter nenhum método `.clone()`!*".\
Yep, estás
completamente correto! Dizeres que a tua class implementa a interface
`Cloneable` não significa absolutamente nada, e não traz nenhuma garantia sobre
o facto de a tua class definir ou não um método `.clone()`! Hurray Java!\
`Cloneable` é uma interface [completamente *broken*][CloneableBroken]. Não a
uses.

* **Casts, casts everywhere:** certamente não tão importante como os outros
argumentos, mas usar `.clone()`s obriga-nos a escrever uma quantidade
considerável de casts, que acaba por gerar poluição visual no nosso código.


### A better way

Anteriormente, mencionei que não fazemos `.clone()` porque queremos uma cópia.
Fazemos `.clone()` para **simular** imutabilidade, para indicar que não
queremos alterações ao estado interno do nosso objeto.\
`.clone()` é apenas **uma** das formas de obter este resultado, que acaba por
ser um remendo que, como observamos, cria mais problemas do que aqueles
que resolve.

Existem alternativas, e no seguinte exemplo vou descrever uma delas.\
Esta alternativa permite-nos retornar a verdadeira variável de instância, sem
cópias intermediárias, sem exceções, sem broken APIs, e sem modificações
inesperadas pelo caller.


(Re) Introducing **const**.\
Provavelmente já ouviste falar em `const`. É uma keyword presente em várias
linguagens, que quando adicionada à declaração de uma variável, marca essa
variável como imutável. Isto significa que não lhe podes re-atribuir novos
valores com o operador `=`, e não a podes passar a funções que a modificam. Um
exemplo em C:
```c
#include <stdio.h>

void print_array(int const* a, size_t n) {  // 'a' marked as const, so it can't
    for (size_t i = 0; i < n; ++i) {        //  be modified by this function
        printf("%d\n", a[i]);
    }
}

void zero_array(int* a, size_t n) {   // 'a' isn't marked as const, so it can be
    for (size_t i = 0; i < n; ++i) {  // modified by this function
        a[i] = 0;  // mutation occurs here
    }
}

int main(void) {
    int const values[] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};  // immutable array
    print_array(values, 10);  // valid, not modifying 'values'
    zero_array(values, 10);  // invalid, modifying 'values'
}
```
Curiosamente, esta keyword já existe em C desde 1989. Considerando que Java foi
criado em 1996, parece que conseguimos regredir.

Adiante, a nossa estratégia aqui é, para cada class que definirmos, definimos
uma interface que só declara métodos que não modificam a class, e a class
implementa esta interface.\
Se a class se chamar `Bar`, vamos chamar à interface `BarView`. É uma *view* de
Bar. Podemos ver, mas não tocar.\
Sempre que queremos retornar um `Bar` que não deve ser modificado pelo caller,
em vez de retornar um `Bar`, retornamos uma `BarView`.
Aplicando este design ao exemplo anterior, obtemos o seguinte código:
```java
interface BarView {  // only declares methods that don't mutate Bar
    ArrayList<String> get_strings();
}

class Bar implements BarView {
    private ArrayList<String> strings;

    public Bar() {
        this.strings = new ArrayList<>();
    }

    public ArrayList<String> get_strings() {
        return (ArrayList<String>)
            this.strings.clone();  // a clone still happens if this function is
        }                          // called. We could, in practice, define an
                                   // ArrayListView, but there's a much simpler
                                   // solution (to be written soon TM).

    public void clear_strings() {
        this.strings.clear();
    }
}

class Foo {
    private Bar bar;

    public Foo() {
        this.bar = new Bar();
    }

    public BarView get_bar() {  // now returns an immutable view
        return this.bar;  // implicit cast from Bar to BarView
    }
}

// Takes a Bar and computes some result, but does not modify the Bar.
// Since it doesn't modify the Bar, we can pass a BarView instead.
void compute(BarView bar) { /* ... */ }

var all_foos = new ArrayList<Foo>(/* ... */);
for (var foo : all_foos) {
    compute(foo.get_bar());  // no more unnecessary clones :)
}
```
Conseguimos definir um método de garantir imutabilidade que:
* não faz cópias desnecessárias, por isso
* não precisa de lidar com exceções na criação das cópias;
* não aceita modificações a `Foo.bar`.\
Por exemplo, tentar chamar `Foo.get_bar().clear_strings()` dá um erro de
compilação:\
'`The method clear_strings() is undefined for the type BarView`', que faz todo o
sentido, porque `BarView` não define esse método.

### Nada disto seria necessário

Embora a solução descrita esteja a resolver o problema, está a resolver um
problema que *não devia existir*.\
Em programação orientada a objetos, os objetos fazem o trabalho por nós, não o
contrário. No exemplo anterior, estamos a "dissecar" um `Foo` através do método
`get_bar()`, e a fazer o "trabalho" com a função `compute()`. A abordagem
correta, em OOP, seria ter a função `compute()` como um método de instância de
`Foo`:
```java
class Bar {
    private ArrayList<String> strings;

    public Bar() {
        this.strings = new ArrayList<>();
    }

    public ArrayList<String> get_strings() {
        return (ArrayList<String>) this.strings.clone();
    }

    public void clear_strings() {
        this.strings.clear();
    }
}

class Foo {
    private Bar bar;

    public Foo() {
        this.bar = new Bar();
    }

    public void compute() { /* ... */ }
}

var all_foos = new ArrayList<Foo>(/* ... */);
all_foos.forEach(Foo::compute);  // functional flavour
```
Isto evita a necessidade de clones, getters, interfaces, e em geral melhora a
ergonomia do código. Conseguimos reduzir significativamente a quantidade de
*[boilerplate code][BoilerplateCode]*. O código ficou mais sucinto, explícito, e
legível.

No entanto, isto nem sempre é possível. Pode acontecer que `Bar` e `Foo` façam
parte de uma biblioteca externa que não possamos modificar. Neste caso, temos
que recorrer a uma destas soluções, e escolher aquela que nos trará menos dor no
futuro.

[GettersSettersEvil]:https://www.yegor256.com/2014/09/16/getters-and-setters-are-evil.html
[ExtendsisEvil]:https://www.javaworld.com/article/2073649/why-extends-is-evil.html
[InheritanceBad]:https://www.yegor256.com/2016/09/13/inheritance-is-procedural.html
[DefensiveProgramming]:https://en.wikipedia.org/wiki/Defensive_programming
[LeakyAbstractions]:https://en.wikipedia.org/wiki/Leaky_abstraction
[CloneableBroken]:https://www.artima.com/intv/bloch.html#part13
[BoilerplateCode]:https://en.wikipedia.org/wiki/Boilerplate_code
