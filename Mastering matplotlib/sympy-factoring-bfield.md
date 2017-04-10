
# Factoring vector equations

<!-- toc orderedList:0 depthFrom:1 depthTo:6 -->

- [Factoring vector equations](#factoring-vector-equations)
	- [Magnetic due to 2-wires](#magnetic-due-to-2-wires)
		- [General equation](#general-equation)
		- [Same $y$ and $I$](#same-y-and-i)
		- [Different $I$](#different-i)

<!-- tocstop -->

## Magnetic due to 2-wires

### General equation

\begin{align}
\vec{\mathbf{B}} = {\mu}I_1 \left( - \frac{y_1}{s_1^2} \hat{x} + \frac{x_1}{s_1^2}  \hat{y} \right)
                  +{\mu}I_2 \left( - \frac{y_2}{s_2^2} \hat{x} + \frac{x_2}{s_2^2}  \hat{y} \right)
\end{align}

### Same $y$ and $I$


```python
import sympy as sym
from IPython.display import Math
```


```python
syms = sym.symbols("\mu I1 I2 I xhat x1 x2 x yhat y s1 s2 d")
(m, I1, I2, I, xhat, x1, x2, x, yhat, y, s1, s2, d) = syms
```


```python
B = m * I * (-y/s1**2*xhat + (x-d)/s1**2*yhat) + m * I * (-y/s2**2*xhat + (x+d)/s2**2*yhat)
```

Quick sanity check:


```python
Math(sym.latex(sym.collect(B, (xhat, yhat))))
```




$$I \mu \left(- \frac{\hat{x} y}{s_{1}^{2}} + \frac{\hat{y}}{s_{1}^{2}} \left(- d + x\right)\right) + I \mu \left(- \frac{\hat{x} y}{s_{2}^{2}} + \frac{\hat{y}}{s_{2}^{2}} \left(d + x\right)\right)$$




```python
factor = sym.factor(B, (xhat, yhat, I, m))
factor
```




    -I*\mu*(xhat*(s1**2*y + s2**2*y) + yhat*(-d*s1**2 + d*s2**2 - s1**2*x - s2**2*x))/(s1**2*s2**2)




```python
Math(sym.latex(factor))
```




$$- \frac{I \mu}{s_{1}^{2} s_{2}^{2}} \left(\hat{x} \left(s_{1}^{2} y + s_{2}^{2} y\right) + \hat{y} \left(- d s_{1}^{2} + d s_{2}^{2} - s_{1}^{2} x - s_{2}^{2} x\right)\right)$$




```python
yhat2 = yhat*(-d*s1**2 + d*s2**2 - s1**2*x - s2**2*x)/(s1**2*s2**2)
Math(sym.latex(sym.factor(yhat2, (yhat,))))
```




$$\frac{\hat{y}}{s_{1}^{2} s_{2}^{2}} \left(- d s_{1}^{2} + d s_{2}^{2} - s_{1}^{2} x - s_{2}^{2} x\right)$$




```python
yhat3 = (-d*s1**2 + d*s2**2 - s1**2*x - s2**2*x)/(s1**2*s2**2)
Math(sym.latex(sym.factor(yhat3, (s1,s2))))
```




$$- \frac{1}{s_{1}^{2} s_{2}^{2}} \left(s_{1}^{2} \left(d + x\right) + s_{2}^{2} \left(- d + x\right)\right)$$



From these, we get

\begin{align}
\frac{s_1^2 y + s_2^2 y}{s_1^2 s_2^2} \hat{x}
\end{align}

\begin{align}
- \frac{\Big(s_{1}^{2} \left(x + d\right) + s_{2}^{2} \left(x - d\right)\Big)}{s_1^2 s_2^2} \hat{y}
\end{align}

and thus our equation for the magnetic field due to two wires with the same currents:

\begin{align}
\vec{\mathbf{B}} = {\mu}I \Bigg( - \bigg( \frac{y}{s_1^2} + \frac{y}{s_2^2} \bigg) \hat{x}
                 + \bigg( \frac{x - d}{s_1^2} + \frac{x + d}{s_2^2} \bigg) \hat{y} \Bigg)
\end{align}

### Different $I$


```python
B = m * I1 * (-y/s1**2*xhat + x1/s1**2*yhat) + m * I2 * (-y/s2**2*xhat + x2/s2**2*yhat)
```

Sanity check:


```python
Math(sym.latex(sym.collect(B, (xhat, yhat))))
```




$$I_{1} \mu \left(\frac{x_{1} \hat{y}}{s_{1}^{2}} - \frac{\hat{x} y}{s_{1}^{2}}\right) + I_{2} \mu \left(\frac{x_{2} \hat{y}}{s_{2}^{2}} - \frac{\hat{x} y}{s_{2}^{2}}\right)$$




```python
factor = sym.factor(B, (xhat, yhat))
factor
```




    -\mu*(xhat*(I1*s2**2*y + I2*s1**2*y) + yhat*(-I1*s2**2*x1 - I2*s1**2*x2))/(s1**2*s2**2)




```python
sym.latex(factor)
```




    '- \\frac{\\mu}{s_{1}^{2} s_{2}^{2}} \\left(\\hat{x} \\left(I_{1} s_{2}^{2} y + I_{2} s_{1}^{2} y\\right) + \\hat{y} \\left(- I_{1} s_{2}^{2} x_{1} - I_{2} s_{1}^{2} x_{2}\\right)\\right)'




```python
Math(sym.latex(factor))
```




$$- \frac{\mu}{s_{1}^{2} s_{2}^{2}} \left(\hat{x} \left(I_{1} s_{2}^{2} y + I_{2} s_{1}^{2} y\right) + \hat{y} \left(- I_{1} s_{2}^{2} x_{1} - I_{2} s_{1}^{2} x_{2}\right)\right)$$




```python
xhat2 = -(I1*s2**2*y + I2*s1**2*y)/(s1**2*s2**2)
factor = sym.factor(xhat2, (s1, s2))
factor
```




    -y*(I1*s2**2 + I2*s1**2)/(s1**2*s2**2)




```python
sym.latex(factor)
```




    '- \\frac{y}{s_{1}^{2} s_{2}^{2}} \\left(I_{1} s_{2}^{2} + I_{2} s_{1}^{2}\\right)'




```python
Math(sym.latex(factor))
```




$$- \frac{y}{s_{1}^{2} s_{2}^{2}} \left(I_{1} s_{2}^{2} + I_{2} s_{1}^{2}\right)$$




```python
yhat2 = (I1*s2**2*x1 + I2*s1**2*x2)/(s1**2*s2**2)
factor = sym.factor(yhat2, (s1, s2))
factor
```




    (I1*s2**2*x1 + I2*s1**2*x2)/(s1**2*s2**2)




```python
sym.latex(factor)
```




    '\\frac{1}{s_{1}^{2} s_{2}^{2}} \\left(I_{1} s_{2}^{2} x_{1} + I_{2} s_{1}^{2} x_{2}\\right)'




```python
Math(sym.latex(factor))
```




$$\frac{1}{s_{1}^{2} s_{2}^{2}} \left(I_{1} s_{2}^{2} x_{1} + I_{2} s_{1}^{2} x_{2}\right)$$



From this, we get these:

\begin{align}
- \frac{I_1 s_2^2 + I_2 s_1^2}{s_1^2 s_2^2} y \hat{x}
\end{align}

\begin{align}
\frac{I_1 s_2^2 x_1 + I_2 s_1^2 x_2}{s_1^2 s_2^2} \hat{y}
\end{align}

and thus our equation for the magnetic field due to two wires with different currents:

\begin{align}
\vec{\mathbf{B}} = {\mu} \Bigg( - \bigg( \frac{I_1 s_2^2 + I_2 s_1^2}{s_1^2 s_2^2} y \bigg) \hat{x}
                                + \bigg( \frac{I_1 s_2^2 x_1 + I_2 s_1^2 x_2}{s_1^2 s_2^2} \bigg) \hat{y} \Bigg)
\end{align}

\begin{align}
= \frac{\mu}{s_1^2 s_2^2} \Big( - y \big( I_1 s_2^2 + I_2 s_1^2 \big) \hat{x}
                                + \big( I_1 s_2^2 (x - d) + I_2 s_1^2 (x + d) \big) \hat{y} \Big)
\end{align}
