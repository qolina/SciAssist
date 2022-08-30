Installation
============

.. _installation:

Prepare the environment
-----------------------
To use SciAssist, first clone the project and create a conda environment for it:

.. code-block:: bash

    # clone project
    git clone https://github.com/WING-NUS/SciAssist
    cd SciAssist

    # [OPTIONAL] create conda environment
    conda create -n myenv python=3.8
    conda activate myenv

    # install pytorch according to instructions
    # https://pytorch.org/get-started/

    # install requirements
    pip install -r requirements.txt

Set up PDF parsing engine s2orc-doc2json
----------------------------------------
The current ``doc2json`` tool uses Grobid to first process each PDF into XML, then extracts paper components from the XML.
If you fail to install Doc2Json or Grobid with ``bin/doc2json/scripts/run.sh`` ,
try to execute the following command:

.. code-block:: bash

    cd bin/doc2json
    python setup.py develop
    cd ../..

This will setup Doc2Json.

Install Grobid
--------------

You will need to have Java installed on your machine. Then, you can install your own version of Grobid and get it running, or you can run the following script:

.. code-block:: console

    bash bin/doc2json/scripts/setup_grobid.sh


This will setup Grobid, currently hard-coded as version 0.6.1. Then run:

.. code-block:: console

    bash bin/doc2json/scripts/run_grobid.sh

to start the Grobid server. Don't worry if it gets stuck at 87%; this is normal and means Grobid is ready to process PDFs.



