# Listas ligadas

Agora, juntando o conhecimento de apontadores e estruturas, vamos introduzir a noção de lista
ligada.

## Definição
Uma lista ligada é uma forma alternativa de representar uma lista de valores. Contrariamente aos arrays,
 que possuem tamanho fixo, uma lista ligada tem comprimento variável.

A representação de uma lista ligada em memória é um conjunto de nodos, cada um com o valor desse
nodo e um apontador para o nodo seguinte.

```
+---+    +---+    +---+
| 3 |    | 5 |    | 7 |
| x----->| x----->| 0 |
+---+    +---+    +---+
```
Lista ligada com 3 elementos. Para marcar o fim de uma lista, coloca-se no ultimo nodo, um apontador
para `NULL` (na imagem representado por um `0`).


Em C podemos então definir uma lista da seguinte forma.

```C
typedef struct lligada {
    int val;
    struct lligada* prox;
} *LLigada;
```

## Utilização
Como exemplo inicial de como ler uma lista ligada. Vamos resolver alguns problemas simples de consulta

### 1. Print da lista
```C
void print_lista(LLigada lista) {
    for(LLigada cursor = lista; cursor != NULL; cursor = cursor->prox) {
        printf("%d\n", cursor->val);
    }
}
```
Este método de iterar por uma lista ligada é bastante comum, declara-se um *pointer* que vamos utilizar
como cursor para percorrer a lista, e em todas as iterações do ciclo, o cursor passa a apontar para
o próximo nodo.

### 2. Retornar o valor no índice X
```C
int list_at(LLigada lista, int indice) {
    for(LLigada cursor = lista; indice > 0 && cursor != NULL; cursor = cursor->prox) {
        indice--;
    }
    return cursor->val; // Atencao, esta linha pode causar um Segmentation Fault caso o indice
                        // seja maior ou igual ao tamanho da lista, tal como aceder fora da area
                        // reservada para um array pode causar um Seg Fault.
}
```
Como podemos ver, aceder a um elemento de uma lista ligada num determinado índice e mais
complexo do que num array. Para além de ser mais custoso. Este custo acrescido ocorre
porque, no caso do array, acedesse diretamente à posição de memória que contém o valor
pretendido. No caso da lista ligada, é necessário percorrer todos os elementos até chegar
à posição pretendida.

### 3. Adicionar um elemento ao inicio da lista.
```C
LLigada list_add(LLigada lista, int valor) {
    LLigada novo_nodo = malloc(sizeof(struct lligada));
    novo_nodo->val = valor;
    novo_nodo->prox = lista;
    return novo_nodo;
}
```
Por outro lado, adicionar um elemento "à cabeça" da lista, é muito mais simples
do que num array. Apenas temos de alocar o espaço para o nodo e colocar este novo
nodo a apontar para a antiga "cabeça".

Outra versão desta função involve o uso de um apontador para o apontador da cabeça.
```C
void list_add2(LLigada* lista, int valor) {
    LLigada novo_nodo = malloc(sizeof(struct lligada));
    novo_nodo->val = valor;
    novo_nodo->prox = *lista;
    *lista = novo_nodo;
}
```

Qual é a diferença? A forma como se utiliza cada uma.
```C
int main() {
    LLigada lista1 = NULL;
    lista1 = list_add(lista1, 1);
    lista1 = list_add(lista1, 2);
    lista1 = list_add(lista1, 3);
    print_lista(lista1);

    LLigada lista2 = NULL;
    list_add2(&lista2, 1);
    list_add2(&lista2, 2);
    list_add2(&lista2, 3);
    print_lista(lista2);
}
```
As duas listas que este programa produz são exactamente iguais.

### 4. Adicionar um elemento num determinado índice da lista
```C
LLigada list_add_index(LLigada lista, int valor, int indice) {
    LLigada anterior = NULL;
    LLigada r = lista;
    for(int i = 0; lista != NULL && i < indice; i++) {
        anterior = lista;
        lista = lista->prox;
    }
    LLigada novo_nodo = malloc(sizeof(struct lligada));
    novo_nodo->val = valor;

    //Verificar se se vai adicionar à cabeça da lista
    if(anterior == NULL) {
        novo_nodo->prox = lista;
        r = novo_nodo;
    }
    else {
        novo_nodo->prox = atual;
        anterior->prox = novo_nodo;
    }
    return r;
}
```
Também adicionar um elemento no meio de uma lista se torna mais simples
que num array, basta iterar pela lista até à posição onde queremos
adicionar o elemento, guardando o anterior, de forma a que seja possível
"ligar" o novo nodo aos elementos anteriores.
Embora o exemplo dado seja para adicionar um elemento numa dada posição,
com ligeiras alterações, é possível adaptá-lo a outros critérios, como
por exemplo, inserir um elemento numa lista ordenada, alterando a condição
de paragem do ``for`` para ir de encontro ao pretendido.

De igual modo, também é possível ter uma versão que recebe o apontador
para a cabeça da lista.
```C
void list_add_index2(LLigada* lista, int valor, int indice) {
    LLigada anterior = NULL;
    LLigada atual = *lista;
    for(int i = 0; atual != NULL && i < indice; i++) {
        anterior = atual;
        atual = atual->prox;
    }
    LLigada novo_nodo = malloc(sizeof(struct lligada));
    novo_nodo->val = valor;

    //Verificar se se vai adicionar à cabeça da lista
    if(anterior == NULL) {
        novo_nodo->prox = atual;
        *lista = novo_nodo;
    }
    else {
        novo_nodo->prox = atual;
        anterior->prox = novo_nodo;
    }
}
```

Para visualizar este algoritmo.
```
Passo 1: Alocar um nodo   ||  Passo 2: Ligar o novo      ||  Passo 3: Ligar o antigo
                          ||       ao nodo seguinte      ||        nodo ao novo nodo
                          ||                             ||
    +---+                 ||       +---+                 ||       +---+
    | 4 |                 ||       | 4 |                 ||       | 4 |
    | 0 |                 ||       | x----+              ||       | x----+
    +---+                 ||       +---+  |              ||       +---+  |
                          ||              v              ||         ^    v
+---+    +---+    +---+   ||   +---+    +---+    +---+   ||   +---+ |  +---+    +---+
| 3 |    | 5 |    | 7 |   ||   | 3 |    | 5 |    | 7 |   ||   | 3 | |  | 5 |    | 7 |
| x----->| x----->| 0 |   ||   | x----->| x----->| 0 |   ||   | x---+  | x----->| 0 |
+---+    +---+    +---+   ||   +---+    +---+    +---+   ||   +---+    +---+    +---+
```

### 5. Libertar a memória utilizada por uma lista
```C
void list_free(LLigada lista) {
    while(lista != NULL) {
        LLigada tmp = lista->prox;
        free(tmp);
        lista = tmp;
    }
}
```
Para libertar uma lista ligada, temos apenas de visitar todos os nodos e
libertá-los individualmente. Tendo cuidado de não libertar um nodo sem guardar
uma referencia para o próximo. Senão nunca mais conseguimos avançar.


## Discussão

As listas ligadas têm vantagens e desvantagens dependendo do problema a
resolver, em teoria <sup>[1]</sup> uma lista ligada pode ser mais eficiente
do que um array. Por exemplo, se temos de adicionar muitas vezes no inicio de
uma lista a operação é simples e em tempo constante, enquanto que para fazer o
mesmo num array, todos os elementos do array teriam de ser copiados para a
frente para fazer espaço para o novo elemento.

- [1] Na pratica, com as maquinas que temos agora, uma lista ligada é quase sempre
  a escolha errada. Mesmo no caso exemplo acima, há soluções mais eficientes do que
  uma lista ligada.

