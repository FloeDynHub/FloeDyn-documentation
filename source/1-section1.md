# Formatting tips

## latex integration:

Note that you can already write some nice latex:

$$
   \begin{eqnarray}
      y    & = & ax^2 + bx + c \\
      f(x) & = & x^2 + 2xy + y^2
   \end{eqnarray}
$$

See [https://myst-parser.readthedocs.io/en/latest/syntax/optional.html](https://myst-parser.readthedocs.io/en/latest/syntax/optional.html) for more info about syntax possibilities.

## References:
To call the reference, use `[1]_` in the markdown file. And make reference to the full citation somewhere else:

```
I call this famous reference here [1]_

---
Here is the list of full citations:
.. [1] P. Rampal, S. Bouillon, E. Olason, and M. Morlighem. neXtSIM: a new Lagrangian sea ice model.  ́ The Cryosphere, 10:1055–1073, 2016.
```
How it renders in markdown:

I call this famous reference here [1]_

---
Here is the list of full citations:

.. [1] P. Rampal, S. Bouillon, E. Olason, and M. Morlighem. neXtSIM: a new Lagrangian sea ice model.  ́ The Cryosphere, 10:1055–1073, 2016.
