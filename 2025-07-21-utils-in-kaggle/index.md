---
date: '2025-07-21T08:51:00+03:30'
draft: false
title: 'Using util files in a Kaggle competition'
description: "A post about how to usi util files in a Kaggle competition"
tags: ["Kaggle", "Deep-Learning", "Python"]
image: "utils-in-kaggle-dataset.webp"
---

# Utils package in Kaggle Competition

## Introduction

I am currently competing in a `Kaggle competition`.
This competition is about detecting repetitive behaviours, mostly
focused on children.
They are connecting a device that has an **IMU**, **Thermopile sensor**, and
**Time of flight sensor**.
There are 18 classes, 10 of which are BFRB (Body Focus repetitive behaviours), and
the rest of them are non-BFRB.
Our job is to classify the given sequences to these 18 classes.
The competition is called **CMI - Detect Behavior with Sensor Data**, and
here is the link to the competition:
[CMI - Detect Behavior with Sensor Data](https://www.kaggle.com/competitions/cmi-detect-behavior-with-sensor-data)

Every time I want to submit my code to this competition, I have to
copy all my utils files to the notebook.
Not going to lie, it is really frustrating.
I saw this `Kaggle competition` doesn't accept `zip` files, but you
can install packages.
So, I came up with the idea of making a package of my util files, and
upload them to `Pypi` to be able to install them and avoid copying them
all the time.
Then, I asked **ChatGPT** about it, and it told me if I want to make the
package private, I can make a `Kaggle Dataset`, upload my `whl` file
into it and use it in the competition.
So, I decided to do that.

## Make a whl file

So, I added a setup file like below:

```python
from setuptools import setup, find_packages


# to build: python setup.py sdist bdist_wheel

def parse_requirements(filename):
    with open(filename, "r") as f:
        lines = f.readlines()
    return [line.strip() for line in lines if line.strip() and not line.startswith("#")]


setup(
    name="cmi_sensor_kaggle",
    version="0.1.1",
    description="Utility package for the CMI sensor data Kaggle competition",
    long_description=open("README.md").read(),
    long_description_content_type="text/markdown",
    author="LiterallyTheOne",
    author_email="Ramin.Zarebidoky@gmail.com",
    url="https://github.com/LiterallyTheOne/cmi_sensor_kaggle",  # optional
    packages=find_packages(),
    install_requires=parse_requirements("requirements_new.txt"),
    classifiers=[
        "Programming Language :: Python :: 3",
        "Operating System :: OS Independent",
    ],
    python_requires=">=3.8",
)
```

This file parses the `README` file and also adds all the requirements
that is needed for the project to install_requires.

## Upload to Kaggle dataset and use it

Then, I made a `Kaggle dataset` and uploaded my `whl` file into it.
To test it, I have used `Google Colab` and I was able to download
and install my util files with the code below:

```python
import os

if os.getenv("KAGGLE_IS_COMPETITION_RERUN"):
    MACHINE = "KAGGLE_COMPETITION"
elif "KAGGLE_KERNEL_INTEGRATIONS" in os.environ:
    MACHINE = "KAGGLE"
elif os.getenv("COLAB_RELEASE_TAG"):
    MACHINE = "COLAB"
else:
    MACHINE = "PC"

print(f"MACHINE: {MACHINE}")

if MACHINE == "COLAB":
    import kagglehub
    from pathlib import Path

    path = Path(kagglehub.dataset_download("literallytheone/cmi-sensor-kaggle-utils"))

    for p in path.glob("*.whl"):
        %pip install {p}
```

At first, I check what `MACHINE` I am in.
If `COLAB_RELEASE_TAG` is in the `Environment variables`, and the
other `Kaggle` related `variables` are not in the `Environment variables`,
I am in `Google Colab`.
Then, if I'm in `Google Colab`, I download the dataset, then find all
the `whl` files and install all of them using `%pip install {p}`.
After that, I am able to import my util files.
Today, I am going to test it on `Kaggle` to see what challenges
I'm going to face.

## Final Thoughts

Making util files is a good practice for the `deep learning` project
and it is a preferred way for me.
Copying all my files to the notebook every time is so frustrating.
To avoid that, I made a package of my util files and uploaded them
to a private `Kaggle Dataset`.
In my opinion, it is a more convenient way to handle this problem.
