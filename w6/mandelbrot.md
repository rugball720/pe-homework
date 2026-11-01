---
jupytext:
  cell_metadata_json: true
  encoding: '# -*- coding: utf-8 -*-'
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.19.5
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
---

# l'ensemble de Mandelbrot

il s'agit de calculer l'image de la convergence de mandelbrot:

```{image} media/mandelbrot.svg
:width: 400px
:align: right
```

```{code-cell} ipython3
import numpy as np
import matplotlib.pyplot as plt
%matplotlib ipympl
```

## comment ça marche ?

+++ {"cell_style": "center"}

* dans l'espace complexe, on définit pour chaque $c\in\mathbb{C}$ la suite
   * $z_0 = c$
   * $z_{n+1} = z_n^2 + c$
* on démontre que 
  * lorsque $|z_n|>2$, la suite diverge

+++ {"cell_style": "center"}

il s'agit pour nous de 

* partir d'un pavé rectangulaire; par exemple sur la figure, on a pris l'habituel  
  $re \in [-2, 0.8]$ et  $im \in [-1.4, 1.4]$

* découper ce pavé en un maillage de $h \times w$ points  (sur la figure, 1000 x 1000)
* on se fixe un nombre maximal `max` d'itérations (disons 20)  
  et pour chaque point du maillage, on va calculer si la suite diverge avant `max` itérations

* c'est-à-dire plus spécifiquement on calcule un tableau `diverge` de la taille du maillage
  * pour chaque point `z`, on calcule les `max` premiers termes de la suite
  * et à la première itération `n` où la suite diverge (son module est supérieur à 2)  
    alors on affecte `diverge[z] = n`

* on n'a plus qu'à afficher ensuite l'image obtenue `diverge` avec `plt.imshow`

+++

````{admonition} indices

pour fabriquer la grille des points de départ, on pourra regarder `np.linspace` et `np.meshgrid`
````

```{code-cell} ipython3
:tags: [level_basic]

def mandelbrot(h, w):
    x = np.linspace(-2, 0.8, w)
    y = np.linspace(-1.4, 1.4, h)
    X, Y = np.meshgrid(x, y)
    c = X + 1j * Y
    z = np.copy(c)
    diverge = np.zeros((h, w))
    
    for n in range(20):
        mask = np.abs(z) <= 2
        z[mask] = z[mask]**2 + c[mask]
        diverge[mask] = n
        
    plt.imshow(diverge)
    plt.show()
```

```{code-cell} ipython3
:tags: [level_intermediate]

# et pour la tester, pour produire la mème figure que ci-dessus

mandelbrot(1000, 1000)
```

## v2

* on peut passer en paramètre à la fonction
  * le domaine en x et en y
  * le nombre maximum d'itérations
* on veut pouvoir produire une image (pour l'insérer dans l'énoncé par exemple)
  * quels formats sont disponibles ?
  * sauvez votre image dans un format vectoriel
  * affichez cette depuis votre notebook

```{code-cell} ipython3
:tags: [level_basic]

def mandelbrot2(xmin, xmax, ymin, ymax, h, w, max_iter, filename):
    x = np.linspace(xmin, xmax, w)
    y = np.linspace(ymin, ymax, h)
    X, Y = np.meshgrid(x, y)
    c = X + 1j * Y
    z = np.copy(c)
    diverge = np.zeros((h, w))
    
    for n in range(max_iter):
        mask = np.abs(z) <= 2
        z[mask] = z[mask]**2 + c[mask]
        diverge[mask] = n
        
    plt.figure()
    plt.imshow(diverge, extent=[xmin, xmax, ymin, ymax])
    plt.savefig(filename)
    plt.close()
```

et je vous demande de mettre ici quelques tests de votre deuxième fonction

```{code-cell} ipython3
:tags: [level_basic]

mandelbrot2(-2, 0.8, -1.4, 1.4, 500, 500, 30, 'test1.svg')

from IPython.display import SVG
SVG('test1.svg')
```

```{code-cell} ipython3
:tags: [level_basic]

mandelbrot2(-1, 0, -0.5, 0.5, 400, 400, 50, 'test2.svg')

SVG('test2.svg')
```

```{code-cell} ipython3
:tags: [level_basic]

mandelbrot2(-0.8, -0.7, 0.05, 0.15, 600, 600, 100, 'test3.svg')

SVG('test3.svg')
```

----
