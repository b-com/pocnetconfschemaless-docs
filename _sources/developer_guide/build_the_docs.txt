Generate the documentation
==========================

pocnetconfschemaless documentation is written in reStructuredText text files
using Sphinx extensions. The HTML version of the documentation can be generated
with Sphinx.

Install Sphinx
--------------

::

    $ pip install Sphinx

Generate the HTML documentation
-------------------------------

::

    $ cd docs
    $ make html

The HTML doc will be generated in ``_build/html``. Its entry point will be
``_build/html/index.html``.

.. note::

   If you're a b-com developer, you can copy the HTML doc to the
   ``pocnetconfschemaless`` folder of the ROADS project on defacto web site
   with::

      $ scp -r _build/html/* forge.b-com.com:/home/groups/ROADS/htdocs/pocnetconfschemaless

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
