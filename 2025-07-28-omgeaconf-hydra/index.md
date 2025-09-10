---
date: '2025-07-28T08:00:00+03:30'
draft: false
title: 'Using config in deep learning'
description: "A post about using config in deep learning"
tags: ["Hydra", "OmegaConf", "Deep-learning"]
image: "omegaconf-hydra.webp"
---

# Using config in deep learning

## Introduction

Yesterday, I trained so many variations of my model to
see which one gives me the better result.
To do that, I use `hydra` and `Omegaconf`.
`OmegaConf` is a strong tool to manage your configs.
One of the best things about `OmegaConf` is that it supports
`dataclass`.
Also, `hydra` is built on top of `OmegaConf` to manage the configs
for more complex systems.
With `hydra`, you can easily override the config by command line.

## Base config

I personally prefer to use `dataclass` in my configs.
It is far easier to manage, and you can use Python code on it,
which makes defining parameters far easier.
To manage my configs better, I have made a `base_config` with the
content below:

```python
# base_config.py
from dataclasses import dataclass, field

from pathlib import Path

features_to_use = [
    "acc_x",
    "acc_y",
    "acc_z",
    "rot_w",
    "rot_x",
    "rot_y",
    "rot_z",
]

label_category_to_int = {
    "Above ear - pull hair": 0,
    "Cheek - pinch skin": 1,
    "Drink from bottle/cup": 2,
    "Eyebrow - pull hair": 3,
    "Eyelash - pull hair": 4,
    "Feel around in tray and pull out an object": 5,
    "Forehead - pull hairline": 6,
    "Forehead - scratch": 7,
    "Glasses on/off": 8,
    "Neck - pinch skin": 9,
    "Neck - scratch": 10,
    "Pinch knee/leg skin": 11,
    "Pull air toward your face": 12,
    "Scratch knee/leg skin": 13,
    "Text on phone": 14,
    "Wave hello": 15,
    "Write name in air": 16,
    "Write name on leg": 17,
}


@dataclass
class ModelConfig:
    num_classes: int = 18
    input_channel: int = len(features_to_use)
    d_length: int = 103


@dataclass
class GaussianNoiseConfig:
    probability: float = 0.5
    noise_std: float = 0.01


@dataclass
class ShrinkOneSequenceConfig:
    probability: float = 0.2
    low: int = 70
    high: int = 90


@dataclass
class CropStartSequenceConfig:
    probability: float = 0.2
    low: int = 10
    high: int = 50


@dataclass
class StretchOneSequenceConfig:
    probability: float = 0.2
    low: int = 20
    high: int = 50


@dataclass
class TimeWarpConfig:
    probability: float = 0.2
    n_knots: int = 4


@dataclass
class BaseConfig:
    name: str = f"{Path(__file__).stem}"
    max_sequence_count: int = 103
    num_classes: int = 18
    features_to_use: list[str] = field(default_factory=lambda: features_to_use)
    features_to_check: list[str] = field(
        default_factory=lambda: [
            "acc_x",
            "acc_y",
            "acc_z",
            "rot_w",
            "rot_x",
            "rot_y",
            "rot_z",
        ]
    )
    target_label: str = "gesture"
    id_features: list[str] = field(
        default_factory=lambda: ["sequence_id", "sequence_counter"]
    )

    label_category_to_int: dict[str, int] = field(
        default_factory=lambda: label_category_to_int
    )

    model: ModelConfig = field(default_factory=ModelConfig)
    gaussian_noise: GaussianNoiseConfig = field(default_factory=GaussianNoiseConfig)
    shrink_one_sequence: ShrinkOneSequenceConfig = field(
        default_factory=ShrinkOneSequenceConfig
    )
    crop_start_sequence: CropStartSequenceConfig = field(
        default_factory=CropStartSequenceConfig
    )

    stretch_one_sequence: StretchOneSequenceConfig = field(
        default_factory=StretchOneSequenceConfig
    )

    time_warp: TimeWarpConfig = field(default_factory=TimeWarpConfig)

    valid_split: float = 0.2
    max_epochs: int = 200
    patience: int = 6
    min_delta: float = 0.0
    use_tensorboard: bool = True
    fill_value: float = 0.0


def register_config():
    from hydra.core.config_store import ConfigStore

    cs = ConfigStore.instance()
    cs.store(name="config", node=BaseConfig)

```

The config below consists of the important parameters for training my model.
Also, it has parameters for controlling augmentations (`GaussianNoiseConfig`,
`ShrinkOneSequenceConfig`, `CropStartSequenceConfig`, `CropStartSequenceConfig`,
`StretchOneSequenceConfig`, `TimeWarpConfig`).  
To manage it with `hydra`, I have used a `register_config()` function.
The reason that I used the import of `hydra` inside the `register_config()`
was because I use `hydra` only for training my model and collecting logs
on my computer.
To run the final result on `Google Colab` or `Kaggle`, I don't need to use
`hydra`.
`Omegaconf` is pre-installed in both of them, so there is no need to
install `hydra` on them.

## Make variations

For example, if my model is something different than the model
I defined in `base_config`, I can simply make a `subclass` of the
base model and change it in a way that I want.
For example:

```python
# a1_tcnt_1_config.py
from cmi_sensor_kaggle.configs.base_config import (
    BaseConfig,
    ModelConfig,
    GaussianNoiseConfig,
    ShrinkOneSequenceConfig,
    CropStartSequenceConfig,
)

from dataclasses import dataclass, field

from pathlib import Path

features_to_use = [
    "acc_x",
    "acc_y",
    "acc_z",
    "rot_w",
    "rot_x",
    "rot_y",
    "rot_z",
    "acc_gx",
    "acc_gy",
    "acc_gz_no_gravity",
    "acc_magnitude",
    "alpha",
    "beta",
    "gamma",
]

weight = [
    0.70977011,
    0.71088435,
    2.8126294,
    0.70977011,
    0.70755208,
    2.8126294,
    0.70755208,
    0.70755208,
    2.8126294,
    0.70755208,
    0.70755208,
    2.8126294,
    0.94933613,
    2.8126294,
    0.70755208,
    0.94735007,
    0.94933613,
    2.8126294,
]


@dataclass
class AttentionModelConfig(ModelConfig):
    d_model: int = 16
    n_head: int = 4
    num_layers: int = 2
    num_classes: int = 18
    input_dim: int = len(features_to_use)


@dataclass
class TransformTcnGlobalFeaturesConfig(BaseConfig):
    name: str = f"{Path(__file__).stem}"
    features_to_use: list[str] = field(default_factory=lambda: features_to_use)
    model: AttentionModelConfig = field(default_factory=AttentionModelConfig)
    weight: list[float] = field(default_factory=lambda: weight)

    patience: int = 10


def register_config():
    from hydra.core.config_store import ConfigStore

    cs = ConfigStore.instance()
    cs.store(name="config", node=TransformTcnGlobalFeaturesConfig)

```

In the code above, I have imported `base_config`.
Then made some subclasses out of them to apply my changes.
As you can see, in the `register_config()` I use the new class
as my node.

Then, if I want to make another variation of the above config, I
can do something like below:

```python
# a1_tcnt_d_model_128.py
from .a1_tcnt_1_config import TransformTcnGlobalFeaturesConfig, AttentionModelConfig

from dataclasses import dataclass, field


@dataclass
class AttentionModelConfig2(AttentionModelConfig):
    d_model: int = 128


@dataclass
class TransformTcnGlobalFeaturesConfig2(TransformTcnGlobalFeaturesConfig):
    name: str = "a1_tcnt_d_model_128"
    model: AttentionModelConfig = field(default_factory=AttentionModelConfig2)


def register_config():
    from hydra.core.config_store import ConfigStore

    cs = ConfigStore.instance()
    cs.store(name="config", node=TransformTcnGlobalFeaturesConfig2)
```

In the code above, I just changed the `d_model` to `128`.
As you can see, managing configs right now is super simple.

## Use the configs

When I want to use the configs, I take two approaches.
First, when I want to use it for training on my computer,
I use it like below:

```python

from cmi_sensor_kaggle.configs.tcnt_weight_global.a1_tcnt_d_model_128 import (
    register_config,
)

register_config()


@hydra.main(
    version_base=None,
    config_name="config",
)
def main(cfg: DictConfig) -> None:
    # -------------------[ Load config ]-------------------
    print(OmegaConf.to_yaml(cfg))

    # rest of the code
    # ...


if __name__ == "__main__":
    main()

```

Second, when I want to run it on a `notebook` on `Google Colab` or `Kaggle`,
I use it like this:

```python
from cmi_sensor_kaggle.configs.tcnt_weight_global.a1_tcnt_d_model_128 import TransformTcnGlobalFeaturesConfig2
from omegaconf import OmegaConf

# %%
cfg = OmegaConf.structured(TransformTcnGlobalFeaturesConfig2)
print(OmegaConf.to_yaml(cfg))
```

## Final thoughts

`OmegaConf` and `hydra` are super great tools to manage config files.
They give you the ability to define a hierarchy of your configs, which
makes the code more efficient.
I have really enjoyed using them, and they made managing configs super
easier for me.
