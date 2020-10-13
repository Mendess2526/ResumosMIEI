# Estimar valores
## Polinómio Interpolador de Newton
* obter os grau + 1 pontos em torno do ponto x (por proximidade)
* calcular o polinómio interpolador
```Matlab
pol = polyfit(points_X, points_Y, grau);
```
* calcular um ponto nesse polinómio
```Matlab
format long;
polyval(pol, point_X)
```

## Spline
### Spline Cúbica
```Matlab
sp = spline(points_X, points_Y, x)
```

### Spline Cúbica Completa
```Matlab
sp = spline(points_X, [declive_inferior points_Y declive_superior], x)
```

NOTAS(apenas utilizar se for explicitado que é necessário):
* se tiver função e não tiver declives
  * derivar função
  * calcular declive para pontos
* se não tiver função nem declives
  * calcular declive dos primeiros dois e últimos dois
  * remover o segundo ponto e último ponto dos vetores

### Segmento de Spline
```Matlab
sp = spline(points_X, points_Y)
% OU
sp = spline(points_X, [declive_inferior points_Y declive_superior])
sp.coefs
```
