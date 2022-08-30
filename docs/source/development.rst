Development
============

.. _development:

How to train on a new task
--------------------------
For a new task, the most important things are the dataset and the model to be used.

To prepare your dataset
"""""""""""""""""""""""

Basically, you can create a **DataModule** in [src/datamodules/](src/datamodules/) to prepare your dataloader.
For example, we have [cora_datamodule.py](src/datamodules/cora_datamodule.py) for Cora dataset.

A **DataModule** standardizes the training, val, test splits, data preparation and transforms.
A datamodule looks like this:

.. code-block:: python

    from pytorch_lightning import LightningDataModule

    class MyDataModule(LightningDataModule):
        def __init__(self):
            super().__init__()
        def prepare_data(self):
            # download, split, etc...
            # only called on 1 GPU/TPU in distributed
        def setup(self, stage):
            # make assignments here (val/train/test split)
            # called on every process in DDP
        def train_dataloader(self):
            train_split = Dataset(...)
            return DataLoader(train_split)
        def val_dataloader(self):
            val_split = Dataset(...)
            return DataLoader(val_split)
        def test_dataloader(self):
            test_split = Dataset(...)
            return DataLoader(test_split)
        #def teardown(self):
            # clean up after fit or test
            # called on every process in DDP

They are actually hook functions, so you can simply overwrite them as you like.

In ``datamodules/components``, you can save some fixed properties such as the label set.

There should be some customed functions for preprocessing which can be shared in several tasks. For example, the procedures for tokenization and padding of different sequence labeling tasks remain consistent. It will be good if you define them as an utility in [src/utils](src/utils), which may facilitates others' work.

Then, create a ``.yaml`` in ``configs/datamodule`` to instantiate your datamodule.
A data config file looks like this:

.. code-block:: yaml

    # The target class of the following configs
    _target_: src.datamodules.my_datamodule.MyDataModule

    # Pass constructor parameters to the target class
    data_repo: "myvision/cora-dataset-final"
    train_batch_size: 8
    num_workers: 0
    pin_memory: False
    data_cache_dir:  ${paths.data_dir}/new_task/


To add a model
""""""""""""""
All the components of a model should be included in ``src/models/components``, including the model structure or a customed tokenizer and so on.

Next, define the logic of training, validation and test for your model in a **LightningModule**.
Same as a LightningDataModule, a **LightningModule** provides some hook functions to simplify the procedure.
Usually it looks like:

.. code-block:: python

    import pytorch_lightning as pl
    import torch.nn as nn
    import torch.nn.functional as F


    class LitModel(pl.LightningModule):
        def __init__(self):
            super().__init__()
            self.l1 = nn.Linear(28 * 28, 10)
            # Define computations here
            # You can easily use multiple components in `models/components`

        def forward(self, x):
            # Use for inference only (separate from training_step)
            return torch.relu(self.l1(x.view(x.size(0), -1)))

        def training_step(self, batch, batch_idx):
            # the complete training loop
            x, y = batch
            y_hat = self(x)
            loss = F.cross_entropy(y_hat, y)
            return loss

        def validation_step(self, batch: Any, batch_idx: int):
            # the complete validation loop
            return loss

        def test_step(self, batch: Any, batch_idx: int):
            # the complete test loop
            return loss

        def configure_optimizers(self):
            # define optimizers and LR schedulers
            return torch.optim.Adam(self.parameters(), lr=0.02)

The **LightningModule** has many convenience methods, and here are the core ones.
Check https://pytorch-lightning.readthedocs.io/en/stable/common/lightning_module.html for further information.

Also, create a config file in ``configs/model``:

.. code-block:: yaml

    # The target Class
    _target_: src.models.cora_module.LitModule
    lr: 2e-5

    # Parameters can be nested
    # When instantiating the LitModule, the following model will be automatically constructed.
    model:
      _target_: src.models.components.bert_token_classifier.BertTokenClassifier
      model_checkpoint: "allenai/scibert_scivocab_uncased"
      output_size: 13
      cache_dir: ${paths.root_dir}/.cache/
      save_name: ${model_name}
      model_dir: ${paths.model_dir}

To create a Trainer and train
"""""""""""""""""""""""""""""
.. note::

    Actually there have been a perfect ``train_pipeline.py`` in our project, so there's no need to write a train pipeline yourself. To prepare the **LightningDataModule** and **LightningModule** is all you need to do.
    But here's an introduction to this procedure in case of any unknown problem.

The last step before starting training is to prepare a trainer config:

.. code-block:: yaml

    _target_: pytorch_lightning.Trainer

    accelerator: 'gpu'
    devices: 1
    min_epochs: 1
    max_epochs: 5

    # ckpt path
    resume_from_checkpoint: null

And then you can create a Pytorch lightning Trainer to manage the whole training process:

.. code-block:: python

    import hydra
    from omegaconf import DictConfig
    from pytorch_lightning import (
        LightningDataModule,
        LightningModule,
        Trainer,
    )

    # To introduce hydra config files
    @hydra.main(version_base="1.2", config_path="configs/", config_name="train.yaml")
    def train(config: DictConfig):
        # Init datamodule
        datamodule: LightningDataModule = hydra.utils.instantiate(config.datamodule)

        # Init lightning model
        model: LightningModule = hydra.utils.instantiate(config.model)

        # Init Trainer
        trainer: Trainer = hydra.utils.instantiate(
            config.trainer, callbacks=callbacks, logger=logger, _convert_="partial"
        )

        # To train the model
        trainer.fit(model=model, datamodule=datamodule)


Finally, you can choose your config files and train your model with the command line:
.. code-block:: bash

    python train.py trainer=gpu datamodule=dataconfig model=modelconfig

How to build a pipeline for a new task
--------------------------------------
As SciAssist aims to serve users, you need to write a pipeline easy to use.
The pipelines are stored in ``src/pipelines``.

For convenience, we don't use hydra in a pipeline.
So simply create a ``xx.py`` file, in which you load a model and define functions which can be directly used:

.. code-block:: python

    model = BertTokenClassifier(
        model_checkpoint="allenai/scibert_scivocab_uncased",
        output_size=13,
        cache_dir=BASE_CACHE_DIR
    )

    model.load_state_dict(torch.load("models/default/scibert-uncased.pt"))
    model.eval()

    def predict(...):
        return results

And in this example we hope it can be imported with:

.. code-block:: python
    from src.pipelines.xx import predict
    res = predict(...)

Other points
------------
Default directories
"""""""""""""""""""
For convenient management, we set some default value as follows.

* src/: all source codes

* configs/: hydra config files

* bin/: third-party toolkits
* data/: datasets
* models/: models or checkpoints we trained
* .cache/: cached files such as models loaded from huggingface
* logs/: experiment logs
* scripts/: quickstart

Some files such as experimental logs and checkpoints
don't need to be commited to the repo.

(Other standards and regulations are to be added here)
""""""""""""""""""""""""""""""""""""""""""""""""""""""