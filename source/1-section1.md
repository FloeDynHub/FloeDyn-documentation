# Formatting tips

## latex integration:
<a name="subseclatex"></a>

Note that you can already write some nice latex:

$$
   \begin{eqnarray}
      y    & = & ax^2 + bx + c \\
      f(x) & = & x^2 + 2xy + y^2
   \end{eqnarray}
$$

See [https://myst-parser.readthedocs.io/en/latest/syntax/optional.html](https://myst-parser.readthedocs.io/en/latest/syntax/optional.html) for more info about syntax possibilities.

## References:
How to do in markdown? You can add "anchors" in the text like this: `<a name="nameforyouranchor"></a>` and then you can make reference to it with `[text of the link](#nameforyouranchor)`. Here is an example:

```
## latex integration:
<a name="subseclatex"></a>

(...)

* Here i call a famous paper [[1]](#ref1).
* Here is want to refer to the above [section about latex integration](#subseclatex).

---
Here is the list of the full citations:

<a name="ref1">[1]</a> P. Rampal, S. Bouillon, E. Olason, and M. Morlighem. neXtSIM: a new Lagrangian sea ice model.  ́ The Cryosphere, 10:1055–1073, 2016.
```

How it renders in html:

* Here i call a famous paper [[1]](#ref1).
* Here is want to refer to the above [section about latex integration](#subseclatex).

---
Here is the list of full citations:

<a name="ref1">[1]</a> P. Rampal, S. Bouillon, E. Olason, and M. Morlighem. neXtSIM: a new Lagrangian sea ice model.  ́ The Cryosphere, 10:1055–1073, 2016.
