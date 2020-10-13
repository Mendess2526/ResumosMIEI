# Mínimos Quadrados
## Polinómios Ortogonais
```matlab
x = [1.5 2 3 4]
y = [4.9 3.3 2 1.5]

[p,s] = polyfit(x,y,1)

polyval(p,2.5)

%Residuo do erro
(s.normr)^2
```

## Modelo não polinomial linear
```matlab
x = [1.5 2 3 4]
y = [4.9 3.3 2 1.5]

foo = @(c,x) c(1)./x + c(2).*x
[c,resnorm,residual,exitflag,out] = lsqcurvefit(foo,[1 1],x,y)

%Residuo do erro
resnorm
```
