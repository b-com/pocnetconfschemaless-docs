Generate the documentation
==========================

pocnetconfschemaless documentation is written in reStructuredText text files
using Sphinx extensions. The HTML version of the documentation can be generated
with Sphinx.

Install Sphinx
--------------

::

    $ pip install Sphinx

Generate and publish the HTML documentation
-------------------------------------------

Generate the HTML documentation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

    $ cd docs
    $ make html

The HTML doc will be generated in ``_build/html``. Its entry point will be
``_build/html/index.html``.

Publish the HTML documentation on GitHub
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A copy of the generated HTML documentation is kept in the
`pocnetconfschemaless-docs GitHub project`_. After a documentation update and
generation, follow these steps to publish the updated documentation on GitHub::

   $ cd ~/code/roads
   $ git clone https://github.com/b-com/pocnetconfschemaless-docs.git
   $ cd pocnetconfschemaless-docs
   $ cp -a ~/code/roads/pocnetconfschemaless/docs/_build/html/* .
   $ git add *
   $ git commit
   $ git push origin master

.. _pocnetconfschemaless-docs GitHub project: https://github.com/b-com/pocnetconfschemaless-docs

The updated documentation will be available at:
https://b-com.github.io/pocnetconfschemaless-docs/

Generate the PDF documentation
------------------------------

.. warning::

   In its current form, the PDF-generated documentation is not good looking
   (incomplete table of contents, truncated code examples, ...)

If you want to generate pocnetconfschemaless doc as a PDF document, you'll
need to install extra dependencies. Under Ubuntu 16.04::

    $ sudo apt install texlive texlive-latex-extra

.. note::

   ``texlive-latex-extra`` provides the ``titlesec.sty`` file which is needed
   by Sphinx.

You'll build the PDF file with::

    $ make latexpdf

You'll find the file in ``_build/latex/pocnetconfschemaless.pdf``.
