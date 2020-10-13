# Quasi-Newton
Este método apenas calcula o mínimo. Assim, para calcular o máximo utilizamos a
seguinte formula:
```
max = -min(-f(x))
```

## Sem derivadas
```matlab
foo = @(x) 0.25*x(1)^4-0.5*x(1)^2+0.1*x(1)+0.5*x(2)^2
fminunc(foo)
% OU
[x,y,ef,out] = fminunc(@(x) 0.25*x(1)^4-0.5*x(1)^2+0.1*x(1)+0.5*x(2)^2,[-1 0.5])
```

> Se `ef < 0` não converge

> Se `ef = 0` atinge nº max iterações -> aumentar maxIter

> Se `ef > 0` converge

## Com derivadas
```matlab
function [f,g] = foo(x)
    f = 0.25*x(1)^4-0.5*x(1)^2+0.1*x(1)+0.5*x(2)^2;
    g(1) = x(1)^3 %derivada em x1
    g(2) = x(2)   %derivada em x2
end

op = optimset('gradobj','on')
[x,y,ef,out] = fminunc('foo',[-1 0.5],op)

```

### optimset options:
* 'tolfun',10
* 'tolx',10
* 'HessUpdate','DFP'
* 'MaxIter',10000
* 'MaxFunEvals',10000
