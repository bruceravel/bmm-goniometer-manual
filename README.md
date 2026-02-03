# bmm-goniometer-manual
User manual for goniometer operations using Bluesky


## Dependencies

To compile this manual, you will need to install the following using
pip or some other method:

1. [sphinx-math-dollar](https://github.com/sympy/sphinx-math-dollar/)
2. [sphinx-subfigure](https://github.com/sphinx-extensions2/sphinx-subfigure)
3. [spinx-book-theme](https://github.com/executablebooks/sphinx-book-theme)

## Hints

To compile the manual on a machine with an externally managed python
environment, you may need to use a python virtual environment.  

First cd to the docs/ folder and do

    python3 -m venv ./my-venv
	
Then install the dependencies locally by doing:

    ./my-venv/bin/pip install sphinx_math_dollar
    ./my-venv/bin/pip install sphinx_subfigure
    ./my-venv/bin/pip install sphinx_book_theme

After installing the dependencies into the virtual environment, try
something like this:

     make SPHINXBUILD=./my-venv/bin/sphinx-build html

## A note about copyright

This document and the BlueSky data collection profile it covers was
developed primarily by a NIST employee. Pursuant to title 17 United
States Code Section 105, works of NIST employees are not subject to
copyright protection in the United States. Thus this repository may
not be licensed under the same terms as Bluesky itself or its
documentation.
