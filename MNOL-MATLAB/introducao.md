# Introdução

## Variáveis
MATLAB e dinamicamente tipado o que significa que não se indica qual e o tipo da
variável. Desta forma, para declarar uma variável utiliza-se o operador `=`:
```matlab
>> x = 20
x =
 20
>> x = 'JBB'
x =
JBB
>> x = [10*2, pi]
x =
   20.0000    3.1416
```

- - - -

## Vetores
Para facilmente declarar um vetor pode-se utilizar o método de MATLAB em que se
indica o valor inicial e final do vetor e o incremento entre cada elemento.
```matlab
>> array = 1:2:9
array =
    1 3 5 7 9
```
Omitindo o campo relativo ao incremento assume-se que seja 1.
```matlab
>> array = 1:4
array =
    1 2 3 4
```

- - - -

## Matrizes
Para declarar uma matriz fila a fila, separando os elementos de cada fila com um
`;` ou com `\n` e a lista de elementos deve ser rodeada de `[]`
```matlab
>> A = [ 1:4 ; 5 6 7 8 ; 9:12 ; 14:17]
A = 
     1  2  3  4
     5  6  7  8
     9 10 11 12
    14 15 16 17
```
Para aceder a uma matriz utiliza-se o operador `()`, podendo ser indicado um
intervalo para obter uma slice da matriz
```matlab
>> A(2,3)
ans =
    7

>> A(2:4, 3:4)
ans =
     7  8
    11 12
    16 17
```

- - - -

## Funções pré-definidas importantes
| funcao      | descricão                              |
| ----------- | -------------------------------------- |
| ones(n)     | matriz de 1s tamanho nxn               |
| zeros(n)    | matriz nula de ordem n                 |
| eye(n)      | matriz identidade de ordem n           |
| (A)’        | transposta da matriz A                 |
| det(A)      | determinante da matriz A               |
| rank(A)     | característica da matriz A             |
| inv(A)      | inversa da matriz A                    |
| diag(A)     | diagonal da matriz A                   |
| triu(A)     | matriz triangular superior da matriz A |
| tril(A)     | matriz triangular inferior da matriz A |
| norm(A,1)   | norma 1 da matriz A                    | 
| norm(A,inf) | norma inf da matriz A                  | 
| A \ b       | resolução do sistema linear Ax = b     |

### Exemplo
```matlab
>> b=[1; 3; -6]
>> A=[1 -3 4; 2 5 -2; 3 8 10]
>> A\b %sistema linear Ax=b
ans =
2.2703
-0.6216
-0.7838
```

- - - -

## Definir Funções
```Matlab
function [output] = function_name(input)
output = operation_on_input
```
**NOTA:** definir na *Function Window*

### Exemplos
* definir uma função "normal"
```Matlab
function [f] = fun1(x)
f = x^2 + x +1
```

* definir um sistema de equações
```Matlab
function [f] = fun2(x)
f(1) = x
f(2) = x^2
```

**NOTA:** é possível definir lambdas com `@(x) operation_on_x`


## Alterar precisão
```matlab
format long  % 15 casas decimais
format short %  5 casas decimais
```
