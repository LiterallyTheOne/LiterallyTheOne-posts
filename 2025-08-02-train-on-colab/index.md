---
date: '2025-08-02T16:33:00+03:30'
draft: false
title: 'Train on google colab'
description: "A post about my approach to train my model on google colab"
tags: ["Python", "Google-Colab", "Kaggle", "Deep-learning"]
image: "train-on-colab.webp"
---

# Train on Google Colab

## Introduction

Today, I was training my model for the
[CMI - Detect Behavior with Sensor Data](https://www.kaggle.com/competitions/cmi-detect-behavior-with-sensor-data)
competition, and I saw that my MacBook was becoming so hot.
Also, I had other things to do with it simultaneously.
I thought `Google Colab` gives us approximately 4 hours of GPU.
Also, my models aren't that deep and don't require so many resources and time.
So, I decided to train my model on `Google Colab` as much as I can, and if
my free limit is over, I switch to `Kaggle` or my MacBook.

## Setup a notebook

The first thing that I did was to make a notebook in my GitHub repository.
I have already given access to `Google Colab` in order to see my private
GitHub repositories.
After that, I loaded the notebook into `Google Colab`.
Now, it is time to clone the whole GitHub repository
To do that, I first created a `token` in `developors` section in
`Github` settings.
Then, I made a `secret` in `colab` with the name of: `GITHUB_TOKEN`.
After that, I used the code below to `clone` the `repository` and
go to the main directory of it, also install the requirements that I
need in `Google Colab`.

```md
from google.colab import userdata

!git clone https://{userdata.get('GITHUB_TOKEN')}@github.com/LiterallyTheOne/cmi_sensor_kaggle

%cd cmi_sensor_kaggle

%pip install -r train_on_colab/requirements.txt -q
```

There are two packages that should be installed prior to the other packages
that are already installed in `Google Colab`.
These two packages are: `pytorch-ignite` and `hydra-core`
Which I put them in the `train_on_colab/requirements.txt`

Then, to use `tensorboard`, I used the code below (I save `tensorboard` logs
in `tb-logger`):

```md
%load_ext tensorboard
%tensorboard --logdir=tb-logger
```

Then, I should run my training procedure.
I already have them, the only thing that I should do is to set them up
in a way that can be run on a notebook.
At first, from the script that I already have, I import `main`, `register_conifg`
like below:

```python
from scripts.run_tof_all_normal_ignite import main, register_config
```

In my script, I used `decorator` in order to work with `Hydra`.
But it is not supported in a `notebook`.
So, I should make an instance of my `hydra` config as they suggested
in their `website`:

```python
from hydra import compose, initialize

register_config()

with initialize(version_base=None):
    cfg = compose(config_name="config")
```

Now, I can run my `main` function like below:

```python
main(cfg)
```

Then, after everything is finished, I zip `checkpoints` and `tensorboard logs`
to be able to download them on my laptop with the code below:

```md
!zip -r combined.zip checkpoints tb-logger
```

## Final Thoughts

`Google Colab` is a really great and powerful tool to train and test
our models.
It has so many great options, and the most useful deep learning packages
are already installed on it.
I am really grateful that `Google` gave us this masterpiece for free.
Right now, my plan is to train my models on `Google Colab`.
Then, download them and submit them to `Kaggle`.