---
layout: post
title:  "Jekyll + MathJax + CMU Serif = Machine Learning Blog"
date:   2019-03-31 21:54:56 +0200
categories: jekyll update
---
After fighting battles with WordPress on and off for over a year, I decided to look for new blog software that would give me the following:

* Attractive $$ \LaTeX $$ markup that integrates seamlessly into posts
* The ability to embed Computer Modern fonts
* Powerful support for code snippets

Essentially, I wanted a great-looking blog that looks like a mathematics paper with the occasional appearance of some lines of code. A dream blog, if you will, for a mathematician, theoretical physicist, or data scientist.

Enter [Jekyll][jekyll-gh], a worthy alternative to WordPress.

When Donald Knuth's wonderful Computer Modern font is used, we get a blog with the potential to show mathematics properly. When equipped with [MathJax][mathjax], Jekyll beautifully renders both inline mathematics like $$ I_p(x) = -\log p(x)$$ and display-style mathematics:

$$ H(p,q) = \sum_{x\in X} p(x)\, I_q(x). $$   

Code snippets also look great:

{% highlight python %}
class BCELoss(WeightedLoss):
  __constants__ = ['reduction', 'weight']
  def __init__(self, weight=None, size_average=None, reduce=None, reduction='mean'):
    super(BCELoss, self).__init__(weight, size_average, reduce, reduction)
  @weak_script_method
  def forward(self, input, target):
    return F.binary_cross_entropy(input, target, weight=self.weight, reduction=self.reduction)
{% endhighlight %}
As expected, the setup was non-trivial, and there are challenges to typesetting mathematics with Jekyll that may be unfamiliar to those who have experience with LaTeX. I'll review some of these issues here.

## Jekyll

Jekyll is a Ruby gem (i.e. a library written in the Ruby language). Provided that you already have Ruby, RubyGems (Ruby's standard package manager), GCC, and Make installed, then installing Jekyll is straightforward:
{% highlight shell %}
$ gem install bundler jekyll
$ jekyll new my-awesome-site
$ cd my-awesome-site
/my-awesome-site $ bundle exec jekyll serve
{% endhighlight %}
The first command uses RubyGems to install both Jekyll and [Bundler](https://bundler.io/), the latter of which is a package manager that, among other things, checks whether all gem dependencies are available for a given application. The second command creates a new local directory for a Jekyll website. After changing to that website's directory, we can serve the website locally with the last command. Local serving gives us the opportunity to develop and debug our website.

The next step is to create an `index.html` to verify the installation:
{% highlight html %}
<!doctype html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Home</title>
  </head>
  <body>
    <h1>Hello World!</h1>
  </body>
</html>
{% endhighlight %}
After we see our Hello World page, we can do one of two things. Either we can get a handle on general Jekyll website design by reading the amazing [step-by-step tutorial][tutorial], or we can `git clone` a pre-designed theme and then repurpose it for our needs. I chose the latter, picking Jekyll's [Minima][minima] theme. After cloning it to a separate directory, I created a new directory called "Math-Minima" and copied the Minima project into Math-Minima.

Minima has the following directory structure, which is typical for a Jekyll theme:

{% highlight shell %}
├── _includes
├── _layouts
├── _posts
├── _sass
│   └── minima
├── assets
│   └── css
└── script
{% endhighlight %}

These directories consist of the following:
* `_includes` : HTML and JavaScript for headers and footers; JavaScript for search engine optimization
* `_layouts` : HTML templates for various types of pages
* `_posts` : Markdown files for blog post content
* `_sass` : Stylesheets for headers, footers, fonts, links, etc.
* `assets` : A CSS directory in which the main stylesheet is built; other accoutrements for the website (images, video, etc.)
* `script` : Shell scripts for building the website

After a Jekyll theme is completed, we build our website in three steps. First, in the theme directory we input to Bundler a `.gemspec` file and get as output a `.gem` file containing all the visual design information for our site. This is done with a command that reads something like `gem build math-minima.gemspec`. Next, we move to the website directory and indicate our website's gem dependencies (including the version of the theme) in the website's `Gemfile`. We then use the command `bundle` to produce a new incarnation of our blog.

## MathJax

MathJax is an open-source JavaScript display engine for LaTeX and other mathematics markup languages. One can find MathJax on content delivery networks all over the web. In particular, many variants of MathJax can be found at [CDNJS](https://cdnjs.com/).

To use MathJax, we place the following code in `_includes/head.html`, immediately before `<\head>`:
{% highlight html %}
<script type="text/javascript" async
  src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/MathJax.js?config=TeX-MML-AM_CHTML" async>
</script> 
{% endhighlight %}
We then modify our theme's `_config.yml` by including the line `mathjax: true`.


## CMU Serif

To incorporate CMU Serif fonts, we work in the `assets` and `_sass` directories. In particular, we place a `fonts` folder, consisting of CMU Serif fonts, inside `assets`. This folder and the files in it will be referenced in a CSS file in the `_sass` directory.

{% highlight shell %}
├── _sass
│   ├── math-minima
│   │   ├── _base.scss
│   │   ├── _fonts.scss
│   │   ├── _layout.scss
│   │   └── _syntax-highlighting.scss
│   └── math-minima.scss
{% endhighlight %}

In `_sass`, we have `math-minima.scss` as our main stylesheet. This stylesheet imports *partial stylesheets* located in `_sass/math-minima`. The original Minima theme comes equipped with `_base.scss`, `_layout.scss`, and `_syntax-highlighting.scss`. The partial `_fonts.scss` is our new addition. It contains `@font-face` rules that upload CMU Serif to our end users.

To maximize compatibility among different browsers, it's good practice to source as many file types as possible of the same font:
{% highlight scss %}
@font-face {
    font-family: "Computer Modern";
    src: 
        url('../fonts/otf/cmunrm.otf') format("opentype"),
        url('../fonts/eot/cmunrm-webfont.eot'),
        url('../fonts/eot/cmunrm-webfont.eot?#iefix') format("embedded-opentype"),
        url('../fonts/svg/cmunrm-webfont.svg') format("svg"),
        url('../fonts/woff/cmunrm-webfont.woff') format("woff"),
        url('../fonts/ttf/cmunrm-webfont.ttf') format("truetype");
}
{% endhighlight %}


## TeXing with Markdown

[Markdown][markdown] is a markup language that is used for the rapid production of rich text documents, such as Jupyter notebooks and Jekyll blog posts. Typesetting mathematics with Markdown is relatively straightforward, though there are some quirks that merit attention.

Inline mathematics is produced with *two* dollar signs instead of one. For example, `$$\tilde G = \mathbb{R}^2\ltimes O(2)$$` gives us $$\tilde G = \mathbb{R}^2\ltimes O(2)$$. Alternatively, one could also use `\\(\tilde G = \mathbb{R}^2\ltimes O(2)\\)`. Note the *double* backslash. The first backslash is used to escape the second for the proper syntax.

Display mathematics may also be produced with two dollar signs, provided that a carriage return precedes the first pair of dollar signs and succeeds the second pair. Another option is to use double backslashes. For instance, \\[ H(p,q) = -\int_X p(x)\, \log q(x)\, d\mu(x)\\] is typeset via `\\[ H(p,q) = -\int_X p(x)\, \log q(x)\, d\mu(x)\\]`, which mimics LaTeX's shorthand for a display equation.

For aligned equations, escaping gets mysterious. To produce
\\[
\begin{split}
r &= \begin{pmatrix} R & 0 \\\\ 0 & 1 \\\\ \end{pmatrix} \\\\\\\\
t &= \begin{pmatrix} I & T \\\\ 0 & 1 \\\\ \end{pmatrix}
\end{split}
\\]
we write
```latex
\\[
\begin{split}
r &= \begin{pmatrix} R & 0 \\\\ 0 & 1 \\\\ \end{pmatrix} \\\\\\\\
t &= \begin{pmatrix} I & T \\\\ 0 & 1 \\\\ \end{pmatrix}
\end{split}
\\]
```
Aside from these minor deviations from standard LaTeX, I haven't run into any issues. These keyboard gymnastics seem like a small price to pay for the display quality. If you want to use Math-Minima, you can find it at [my GitHub page](https://github.com/admshumar/math-minima). If you want to learn more about modifying Minima, or about general design in Jekyll, check out [Jekyll's GitHub page for Minima](https://github.com/jekyll/minima).

[minima]: https://github.com/jekyll/minima
[tutorial]: https://jekyllrb.com/docs/step-by-step/02-liquid/
[mathjax]: https://www.mathjax.org/
[jekyll-gh]:   https://github.com/jekyll/jekyll
[markdown]: https://en.wikipedia.org/wiki/Markdown