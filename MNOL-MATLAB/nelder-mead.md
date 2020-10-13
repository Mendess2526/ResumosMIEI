# Método de Nelder-Mead
```matlab
foo = @(x) max(abs(x(1)),abs(x(2)-1))

op = optimset('display','iter')

[x,y,ef,out] = fminsearch(foo,[1 1],op)
```
