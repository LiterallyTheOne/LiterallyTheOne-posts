---
date: '2025-07-29T08:00:00+03:30'
draft: false
title: 'Working with Time of Flight data'
description: "A post about Working with Time of Flight data"
tags: ["Polars", "Sensor", "Kaggle", "CMI", "Deep-learning"]
image: "time-of-flight-cmi.webp"
---

# Working with Time of Flight data

## Introduction

Yesterday, I mostly focused on preparing `Time of flight` sensor data
to give it to a deep model.
The data that I'm working on belongs to a competition on `Kaggle`, and
here is the link to it:
[CMI - Detect Behavior with Sensor Data](https://www.kaggle.com/competitions/cmi-detect-behavior-with-sensor-data)
They have used 5 `Time of flight` sensors.
The output of each sensor is an 8x8 matrix,
but they have stored it as a 64 array.
So, I was trying to write a `Preprocess` module to make the data into an 8x8 matrix.
Then, write a `Transform` to make all 5 sensors into a tensor like below:

```text
[ Sequence_length, 5, 8, 8]
```

The reason that I wanted to make the `preprocess` module for turning data into 8x8 matrices was
to being able to do other `image` related `preprocesses` on them.
If I have written it in the `Transform` module, I might have lost some important `preprocesses`.
But I should see.

## Preprocess module

To change the `Time of Flight` data into to 2d format, I have written the module below:

```python
import polars as pl
import polars.selectors as cs


class ChangeTOFTo2D:

    def __init__(
            self,
    ):
        pass

    @classmethod
    def from_config(
            cls,
            cfg: DictConfig,
    ) -> "ChangeTOFTo2D":
        return cls()

    def __call__(self, df: pl.DataFrame) -> pl.DataFrame:
        new_df = df.clone()
        df_tof = new_df.select(cs.starts_with("tof"))
        for i in range(1, 6):
            s_i = df_tof.select(cs.starts_with(f"tof_{i}_v"))
            s_i_n = s_i.to_numpy()
            s_i_n = s_i_n.reshape((-1, 8, 8))
            new_df = new_df.with_columns(pl.Series(f"tof_{i}_2d", s_i_n))
            new_df = new_df.drop(cs.starts_with(f"tof_{i}_v"))
        return new_df

```

As you can see, at first, I select all the `Time of Flight data`.
Then I select each sensor's data in a for loop, reshape them, and add them to a new `Dataframe`.

## Transform module

To transform a `Time of Flight` data to a tensor, I have written the module below:

```python

class TOFTransform:
    def __init__(
            self,
            features_to_use: list[str],
            max_sequence_count,
    ):
        self.features_to_use = features_to_use
        self.max_sequence_count = max_sequence_count

    @classmethod
    def from_config(cls, cfg: DictConfig) -> "TOFTransform":
        if "features_to_use" not in cfg:
            raise ValueError("features_to_use is required")

        if "max_sequence_count" not in cfg:
            raise ValueError("max_sequence_count is required")

        return cls(
            cfg.features_to_use,
            cfg.max_sequence_count,
        )

    def __call__(
            self,
            sequence: pl.DataFrame,
    ) -> tuple[torch.Tensor, torch.Tensor]:

        num_features = len(self.features_to_use)
        sequence_np = sequence.select(self.features_to_use).to_numpy()

        all_zeros = np.zeros((self.max_sequence_count, num_features, 8, 8))
        mask = np.zeros(self.max_sequence_count, dtype=bool)

        result_list = []
        for i in range(sequence_np.shape[0]):
            result_row = []
            for j in range(sequence_np.shape[1]):
                result_row.append(sequence_np[i][j])
            result_list.append(result_row)

        result_np = np.array(result_list)

        all_zeros[: result_np.shape[0]] = result_np
        mask[: result_np.shape[0]] = True

        data = torch.from_numpy(all_zeros).float()
        attention_mask = torch.from_numpy(mask).bool()

        return data, attention_mask

```

As you can see, at first, I create a sequence full of zeros.
The reason for doing that is to zero-pad the sequences to have all the sequences in the same shape.
When I convert the sequence to numpy, the result of that would be: `[sequence_length, 5]`.
The 8x8 matrices are being seen as `objects`.
But my desired output is: `[sequence_length, 5, 8, 8]`.
So, to achieve that, I defined `result_list`, and with two for loops, I changed the objects
to `8x8` arrays.

## Final thoughts

`Time of Flight` sensor's data can be seen as images.
I'm trying to assess their importance in the
[CMI - Detect Behavior with Sensor Data](https://www.kaggle.com/competitions/cmi-detect-behavior-with-sensor-data)
competition on `Kaggle`.
In my opinion, they can contain more important features than `IMU`, but I'm not sure until I give it a try.

