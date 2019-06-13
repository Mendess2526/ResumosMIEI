# Hierarquia de classes

A hierarquia de classes tem imensas vantagens no que toca a manter a
sanidade de quem programa.

**Nota:** É assumido que há um getter definido corretamente para todas as
variáveis de instância declaradas.

## Situação exemplo
Imagina que precisamos de implementar um programa com alunos e professores.
Poderíamos então implementar as seguintes classes:
```Java
public class Aluno{
	private String num;
	private String nome;
	private String email;
	private ArrayList<String> cadeiras;

	public Aluno(String num, String nome, String email, List<String> cadeiras){
		this.num = num;
		this.nome = nome;
		this.email = email;
		this.cadeiras = new ArrayList<>(cadeiras);
		//Um dos construtores de arraylist permite passar uma Collection.
		//No entanto, é preciso ter cuidado pois este construtor não
		// clona os elementos da lista, ou seja, só pode ser usado para
		// listas de objetos imutáveis.
	}
	/* Getters, Setters, etc */
}

public class Prof{
	private String num;
	private String nome;
	private String email;
	private String departamento;

	public Prof(String num, String nome, String email, String departamento){
		this.num = num;
		this.nome = nome;
		this.email = email;
		this.departamento = departamento;
	}
	/* Getters, Setters, etc */
}
```
No entanto, temos assim muito código duplicado. Variáveis repetidas,
getters, setters, se for o caso, também teremos métodos iguais repetidos.

O equals e o toString vão ser extremamente parecidos também.

## Classe que agrupa o código comum
Uma primeira solução para o nosso problema é então agrupar as partes comuns
numa só classe.
```Java
public class Pessoa {
	private String num;
	private String nome;
	private String email;

	public Pessoa(String num, String nome, String email){
		this.num = num;
		this.nome = nome;
		this.email = email;
	}
	/* Getters, Setters, etc */
}
```
Resta-nos apenas relacionar as nossas antigas classes com esta.
Isto é muito simples. Mas tem alguns detalhes importantes a ter em conta,
que falarei já a seguir.
```Java
public class Aluno extends Pessoa {
	private ArrayList<String> cadeiras;

	public Aluno(String num, String nome, String email, List<String> cadeiras){
		super(num,nome,email);
		this.cadeiras = new ArrayList<>(cadeiras);
	}
	/* Getters, Setters, etc */
}

public class Prof extends Pessoa {
	private String departamento;

	public Prof(String num, String nome, String email, String departamento){
		super(num,nome,email);
		this.departamento = departamento;
	}
	/* Getters, Setters, etc */
}
```
A primeira palavra ('keyword') nova aqui é `super`. Esta serve para fazer
referência a símbolos (variáveis, métodos, etc) definidos na superclasse.

Assim a linha `super(...)` refere-se ao construtor da superclasse e evita-se
assim repetir código, visto que aquela parte do construtor é igual para
todas as subclasses de `Pessoa`.

Podemos também usar o super para chamar métodos da superclasse. O clássico
exemplo é o equals.
```Java
public class Aluno extends Pessoa {
	/* igual ao de cima */
	public boolean equals(Object o){
		if(this == o) return true;
		if(o == null || this.getClass() != o.getClass())
			return false;
		Aluno that = (Aluno) o;
		return super.equals(o)
		    && this.cadeiras.equals(that.getCadeiras());
	}
}
```
### Implicações desta construção do código
Pessoa é uma classe como outra qualquer. Podemos instancia-la e chamar-lhe
os métodos que tem definido.
```Java
	Pessoa p = new Pessoa("D99999","JBB","god@ghci.com");
	String num = p.getNum(); // funciona
	ArrayList<String> cadeiras = p.getCadeiras(); // Não funciona, este getter não
                                                     // está definido na classe Pessoa.
```
`Aluno` (e `Prof`) podem também ser instanciados. E podemos chamar métodos
 definidos na superclasse e métodos definidos na própria classe.
```Java
	Aluno a = new Aluno("A12345","Mendes","mendes@mymail.com", new ArrayList<>());
	String num = a.getNum(); // Funciona. O getter está definido na superclasse
				 // que esta estende.
	ArrayList<String> cadeiras = a.getCadeiras();
				// Funciona. O getter esta definido nesta classe.
```
Mas a mais interessante consequência é a seguinte:
```Java
	Pessoa p = new Prof("D54321","Nestor","nestor@clone.com","DI");
	String num = p.getNum(); //Funciona.
	String departamento = p.getDepartamento();
				// Não funciona. Um objeto do tipo Pessoa não tem
				// este getter definido. Apesar do "verdadeiro" tipo
				// da classe ser Prof.
	Prof prof = (Prof) p;
	String departamento = prof.getDepartamento();
				// Funciona. Após o cast a JVM já consegue encontrar
				// o método e chamá-lo.
```
Este "malabarismo" de casts pode dar a entender que instanciar objetos desta
forma é uma perda de tempo. Mas há uma muito boa razão para isto. Considere
o seguinte exemplo:
```Java
	ArrayList<Aluno> alunos = new ArrayList<>();
	alunos.add(new Aluno(...)); // Funciona.
	alunos.add(new Prof(...)); // Não funciona.

	ArrayList<Pessoa> pessoas = new ArrayList<>();
	pessoas.add(new Aluno(...)); // Funciona. Alunos são Pessoas.
	pessoas.add(new Prof(...)); // Funciona. Profs são Pessoas.
```
Aqui vemos uma das mais comuns utilizações deste paradigma. Outro exemplo é o seguinte:
```Java
public class Cartao{
	private Pessoa p;
	public Cartao(Pessoa p){
		this.p = p;
	}
}
```
Esta classe pode ter, na sua variável de instância, uma `Pessoa`, um `Aluno`
ou um `Prof`.

#### Atenção
```Java
	ArrayList<Pessoa> pessoas = new ArrayList<>();
	// Adicionam-se montes de cenas a lista.
	Pessoa p = pessoas.get(42);
	Prof prof = (Prof) p; // Não funciona caso o tipo "verdadeiro" de p não seja Prof.
```
Este código poderá lançar uma `ClassCastException`. Este tipo de erros não
podem ser detetados pelo compilador mas quando ocorrem crasham o programa.

Para evitar isto, simplesmente temos de fazer uma simples verificação:
```Java
	if(p instanceof Prof){
		Prof prof = (Prof) p;
		/* Fazer cenas de prof */
	}
```

## Classes abstratas
Até agora tudo bem, mas há um problema com a nossa aplicação. Podem haver
 instâncias de `Pessoa`, que não fazem sentido no nosso contexto. (Ou se é
Aluno ou se é Prof)

Este problema é facilmente resolvido adicionando uma palavra na declaração
de `Pessoa`.
```Java
public abstract class Pessoa {
	/* Tudo igual */
}
```
Tudo que foi escrito acima deste parágrafo continua a ser verdade, exceto
`new Pessoa(...)`, isto agora é inválido.

Outra vantagem é que agora podemos definir métodos abstratos.
```Java
public abstract class Pessoa {
	public abstract irParaAula(Aula a);
}
```
Agora todas as classes que estendem `Pessoa` são obrigadas a implementar o
método `irParaAula`.

Um método abstrato é declarado na classe abstrata e implementado nas
subclasses da mesma. Isto é, na classe abstrata dizemos que todas as suas
subclasses vão ter aquele comportamento (leia-se método) mas cada subclasse
é que escolhe como o implementa.

**Um método abstrato é obrigatório implementar.**

Isto é útil quando faz sentido que todas a subclasses implementem um certo
comportamento mas este depende de fatores que não conhecemos na superclasse,
por exemplo.

Em Java cada classe pode apenas estender uma superclasse. (C++, por exemplo,
permite herança múltipla)
