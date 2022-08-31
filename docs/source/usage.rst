Usage
=====

.. _usage:

How to Parse Reference Strings
------------------------------

Parse strings from PDFs
"""""""""""""""""""""""

To parse reference strings from **a PDF file**, try:

.. code-block:: python

    from src.pipelines.bert_parscit import predict_for_pdf

    results, tokens, tags = predict_for_pdf(filename, output_dir, temp_dir)

This will generate a text file of reference strings in the specified ``output_dir``.
And the JSON format of the origin PDF will be saved in the specified ``temp_dir``.
The default ``output_dir`` is ``result/`` from your path and the default ``temp_dir`` is ``temp/`` from your path.

The output ``results`` is a list of tagged strings, which seems like:

::

    ['<author>Waleed</author> <author>Ammar,</author> <author>Matthew</author> <author>E.</author> <author>Peters,</author> <author>Chandra</author> <author>Bhagavat-</author> <author>ula,</author> <author>and</author> <author>Russell</author> <author>Power.</author> <date>2017.</date> <title>The</title> <title>ai2</title> <title>system</title> <title>at</title> <title>semeval-2017</title> <title>task</title> <title>10</title> <title>(scienceie):</title> <title>semi-supervised</title> <title>end-to-end</title> <title>entity</title> <title>and</title> <title>relation</title> <title>extraction.</title> <booktitle>In</booktitle> <booktitle>ACL</booktitle> <booktitle>workshop</booktitle> <booktitle>(SemEval).</booktitle>']


``tokens`` is a list of origin tokens of the input file and ``tags`` is the list of predicted tags corresponding to the input:

tokens::

    [['Waleed', 'Ammar,', 'Matthew', 'E.', 'Peters,', 'Chandra', 'Bhagavat-', 'ula,', 'and', 'Russell', 'Power.', '2017.', 'The', 'ai2', 'system', 'at', 'semeval-2017', 'task', '10', '(scienceie):', 'semi-supervised', 'end-to-end', 'entity', 'and', 'relation', 'extraction.', 'In', 'ACL', 'workshop', '(SemEval).']]

tags::

[['author', 'author', 'author', 'author', 'author', 'author', 'author', 'author', 'author', 'author', 'author', 'date', 'title', 'title', 'title', 'title', 'title', 'title', 'title', 'title', 'title', 'title', 'title', 'title', 'title', 'title', 'booktitle', 'booktitle', 'booktitle', 'booktitle']]

Parse strings from TEXT
"""""""""""""""""""""""
You can also process **a single string** or parse strings from **a TEXT file**:

.. code-block:: python
    from src.pipelines.bert_parscit import predict_for_string, predict_for_text

    str_results, str_tokens, str_tags = predict_for_string(
        "Waleed Ammar, Matthew E. Peters, Chandra Bhagavat- ula, and Russell Power. 2017. The ai2 system at semeval-2017 task 10 (scienceie): semi-supervised end-to-end entity and relation extraction. In ACL workshop (SemEval).")
    text_results, text_tokens, text_tags = predict_for_text(filename)


Extract strings from a PDF
""""""""""""""""""""""""""
You can extract strings you need with the script.
For example, to get reference strings, try:

.. code-block:: console

    python pdf2text.py --input_file file_path --reference --output_dir output/ --temp_dir temp/


