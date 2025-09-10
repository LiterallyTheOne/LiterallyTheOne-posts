---
date: '2025-07-26T21:13:00+03:30'
draft: false
title: 'Data augmentation is so important'
description: "A post about the importance of data augmentation"
tags: ["Pytorch", "Deep-learning", "Data-augmentation", "Kaggle", "CMI"]
image: "data-augmentation.webp"
---

# Data augmentation

## Introduction

Yesterday, I was working on [cmi project](https://www.kaggle.com/competitions/cmi-detect-behavior-with-sensor-data)
and my score has increased by `2%` when I added two new `data augmentations`.
`Data augmentation` is a technique in `machine learning` that artificially expands
our training dataset by applying different `transformations`.
In `torchvision`, there are multiple `transfromation` have been implemented.
You can access these `transformations` by importing `v2` (version 2) from
`torchvision.transforms` like this:
`from torchvision.transforms import v2`.
`Data augmentation` is extremely helpful when we want to generalize
our training data, so the model doesn't `overfit` on training data.
Also, it helps the model not to get biased with some specific features
that are only available in training data.

## My use-case and Gaussian noise

In this project, we have to classify the behaviour of an individual with
a sequence.
I was specifically working on the `IMU` (Inertial Measurement Unit) part of
the data, to increase my score on that as much as I can.
The augmentation that I had previously was to add a small `Gaussian noise`.
The code below shows my approach:

```python

class AddGaussianNoise:
    def __init__(
            self,
            features_to_use: list[str],
            probability: float = 0.5,
            noise_std: float = 0.01,
    ):
        self.features_to_use = features_to_use
        self.probability = probability
        self.noise_std = noise_std

    @classmethod
    def from_config(cls, cfg: DictConfig) -> "AddGaussianNoise":
        if "features_to_use" not in cfg:
            raise ValueError("features_to_use is required")

        if "gaussian_noise" in cfg and "probability" in cfg.gaussian_noise:
            probability = cfg.gaussian_noise.probability
        else:
            probability = 0.5
            print(f"Using default probability: {probability}")

        if "gaussian_noise" in cfg and "noise_std" in cfg.gaussian_noise:
            noise_std = cfg.gaussian_noise.noise_std
        else:
            noise_std = 0.01
            print(f"Using default gaussian noise_std: {noise_std}")

        return cls(
            cfg.features_to_use,
            probability=probability,
            noise_std=noise_std,
        )

    def __call__(
            self,
            sequence: pl.DataFrame,
    ):

        result = sequence

        if np.random.rand() < self.probability:
            for feature in self.features_to_use:
                result = result.with_columns(
                    pl.col(feature)
                    + np.random.normal(
                        loc=0.0, scale=self.noise_std, size=result[feature].shape
                    ),
                )

        return result

```

The code above adds `Gaussian noise` with the scale of `noise_std`
with the given `probibility`.
Also, I have a standard to be able to load each class that I write with
config.
So, in this class, I have a `class method` called `from_config` which
helps me to create a new instance of this object simply, like the code below:

```python
agn = AddGaussianNoise.from_config(cfg)
```

I really like this approach, and it made my code way cleaner and more readable.

## Shrink Sequence

The other augmentation that I add is to shrink the sequence.
Shrinking a sequence means increasing the speed of actions.
So it teaches the model not to care too much about the speed of the actions.
I did that with the code below:

```python

class ShrinkOneSequence:

    def __init__(
            self,
            probability: float = 0.2,
    ):
        super().__init__()
        self.probability = probability

    @classmethod
    def from_config(
            cls,
            cfg: DictConfig,
    ) -> "ShrinkOneSequence":

        if "shrink_one_sequence" in cfg and "probability" in cfg.shrink_one_sequence:
            probability = cfg.shrink_one_sequence.probability
        else:
            probability = 0.2
            print(f"Using default probability: {probability}")

        return cls(
            probability=probability,
        )

    def __call__(self, sequence: pl.DataFrame) -> pl.DataFrame:

        if np.random.rand() < self.probability:
            max_len = sequence.shape[0]
            shrink_random = (np.random.randint(70, 90)) / 100
            max_len *= shrink_random
            max_len = round(max_len)
            take_sample = np.linspace(
                0,
                sequence.shape[0],
                max_len,
                endpoint=False,
                dtype=int,
            )

            sequence = sequence[take_sample]

        return sequence


```

The code above shrinks the given sequence with the given probability.
The result size of the sequence would be in the range of [70%, 90%].
These numbers still need to be calculated carefully, but right now, this
technique has been pretty helpful.

## Crop the start of the sequence

The other `Augmentation` that I use is to crop the start of the sequence.
I know the main action that happens is at the end of the sequence.
So, by cropping the start of the sequence, it seems that I'm giving
the model new data also helps it to focus more on the end of the sequence.
To do that, I wrote the code below:

```python

class CropStartSequence:

    def __init__(
            self,
            probability: float = 0.2,
    ):
        super().__init__()
        self.probability = probability

    @classmethod
    def from_config(
            cls,
            cfg: DictConfig,
    ) -> "CropStartSequence":

        if "crop_start_sequence" in cfg and "probability" in cfg.crop_start_sequence:
            probability = cfg.crop_start_sequence.probability
        else:
            probability = 0.2
            print(f"Using default probability: {probability}")

        return cls(
            probability=probability,
        )

    def __call__(self, sequence: pl.DataFrame) -> pl.DataFrame:

        if np.random.rand() < self.probability:
            max_len = sequence.shape[0]
            crop_percentage = (np.random.randint(10, 50)) / 100
            crop_start = round(max_len * crop_percentage)
            sequence = sequence[crop_start:]

        return sequence

```

The code above crops the start of the sequence with the given probability.
The crop size can be between [10%, 50%].
These crop sizes should be calculated with careful measures, but right now
it is working really well.

## Final thoughts

In deep learning, `Data` is literally everything.
The better the way that we present our data to the model, the better
our results will be.
In this project, we don't have that much data.
So, by using correct `augmentation`, I am trying to generalize my model.
One of my next plans is to make a data generator to create similar data
to the data that we have, or find some related data.
