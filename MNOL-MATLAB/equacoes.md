# Resolver sistemas de equações
* definir a função
* utilizar o comando **fsolve**
```Matlab
op = optimset('tolfun',TolFun_Value,'tolx',TolX_Value);
x1 = Aproximação_inicial;
format long;
[x,fval,exitflag, output] = fsolve('fun2', x1, op)
```
NOTA: tolx é o critério de paragem

- - - -

# Resolver equação
* definir a função
* utilizar o comando **fsolve**
```Matlab
op = optimset('tolx',TolX_Value);
x1 = Aproximação_inicial;
format long;
[x,fval,exitflag, output] = fsolve('fun2', x1, op)
```
