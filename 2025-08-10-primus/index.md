---
date: '2025-08-10T16:47:00+03:30'
draft: false
title: 'Try PRIMUS, a pretrained model for IMU'
description: "A post about trying PRIMUS"
tags: ["PyTorch", "Deep-learning", "Kaggle", "CMI"]
image: "try_primus.webp"
---

# Try PRIMUS, a pretrained model for IMU

Yesterday, I was thinking about how I can make my results better.
I thought there might be some models that have already been trained
on `IMU` data.
So, I only fine-tune them and use them to see how much my result changes.
After some research, I decided to start with a model called
[PRIMUS](https://github.com/nokia-bell-labs/pretrained-imu-encoders).
This model is trained on `IMU` data, and I thought it might do the job for me.

## First step

At first, I cloned their repository from GitHub
([PRIMUS](https://github.com/nokia-bell-labs/pretrained-imu-encoders))
and downloaded their pretrained model.
Then, I tried to run their model with dummy data.
So, I came up with the code below:

```python
# -------------------[ Make an instance of their model ]-------------------
encoder = MW2StackRNNPoolingMultihead(size_embeddings=512).to("cpu")

# -------------------[ Load the pretrained model ]-------------------

path_load_pretrained_imu_encoder = "best_model.ckpt"

dic = torch.load(
    path_load_pretrained_imu_encoder,
    map_location="cpu",
    weights_only=True,
)
dic = {k[12:]: v for k, v in dic["state_dict"].items() if "imu_encoder" in k}
encoder.load_state_dict(dic)
encoder.eval()

print(encoder)

# -------------------[ Test the model with a dummy data ]-------------------

imu_data = torch.randn(4, 6, 200).to("cpu")
features = encoder(imu_data)
print(features)  
```

At first, I make an instance of their model and cast it to `cpu`
(`mps` doesn't work on this model. Also, they have hardcoded so many `.cuda()`
which I had to change them to `.to("cpu")`).
Then, with the code they have provided in
`downstream_evaluation/classification_downstream_egoexo4d.py`.
I managed to load their model from their pretrained weights.
After that, I fed the model with dummy data.
The result shape would be `[4, 512]`(`[batch_size, size_embeddings]`).

## Model Input

As you can see, the model input has 6 channels.
After some research, I found out they use `acceleration` and `gyro` to
train their model.
But my data is `acceleration` and `quaternion`.
So I had to change the `quaternion` ([`rot_w`, `rot_x`, `rot_y`, `rot_z`]) to
([`gyro_x`, `gyro_y`, `gyro_z`]).
Because I didn't know anything about what `gyro` actually is, I told
`ChatGPT` to write a `PreProcess Module` in my style, and it came up
with this:

```python
class AddGyroFromQuat(PrepareData):
    def __init__(
            self,
            quat_cols: list[str] = None,
            seq_id_col: str = "sequence_id",
            seq_counter_col: str = "sequence_counter",
            sample_rate_hz: float = 50.0,
    ):
        super().__init__()
        if quat_cols is None:
            quat_cols = ["rot_w", "rot_x", "rot_y", "rot_z"]
        self.quat_cols = quat_cols
        self.seq_id_col = seq_id_col
        self.seq_counter_col = seq_counter_col
        self.sample_rate_hz = sample_rate_hz

    @classmethod
    def from_config(
            cls,
            cfg: DictConfig,
    ) -> "AddGyroFromQuat":
        return cls(
            quat_cols=cfg.get("quat_cols", ["rot_x", "rot_y", "rot_z", "rot_w"]),
            seq_id_col=cfg.get("seq_id_col", "sequence_id"),
            seq_counter_col=cfg.get("seq_counter_col", "sequence_counter"),
            sample_rate_hz=cfg.get("sample_rate_hz", 50.0),
        )

    def __call__(self, df: pl.DataFrame) -> pl.DataFrame:
        df = df.sort([self.seq_id_col, self.seq_counter_col])

        gyro_x_all = []
        gyro_y_all = []
        gyro_z_all = []

        for _, group in df.group_by(self.seq_id_col):
            quats = group.select(self.quat_cols).to_numpy()
            # scipy expects quaternions in [x, y, z, w]
            rots = R.from_quat(quats)
            dt = 1.0 / self.sample_rate_hz

            gx, gy, gz = [], [], []
            for i in range(len(rots) - 1):
                relative_rot = rots[i].inv() * rots[i + 1]
                rotvec = relative_rot.as_rotvec()  # radian rotation vector
                omega = rotvec / dt  # rad/s angular velocity
                gx.append(omega[0])
                gy.append(omega[1])
                gz.append(omega[2])

            # Repeat last value so lengths match
            gx.append(gx[-1] if gx else 0.0)
            gy.append(gy[-1] if gy else 0.0)
            gz.append(gz[-1] if gz else 0.0)

            gyro_x_all.extend(gx)
            gyro_y_all.extend(gy)
            gyro_z_all.extend(gz)

        df = df.with_columns(
            [
                pl.Series("gyro_x", gyro_x_all),
                pl.Series("gyro_y", gyro_y_all),
                pl.Series("gyro_z", gyro_z_all),
            ]
        )

        return df
```

I have changed it a little bit.
Not going to lie, I'm not sure if it's working correctly or not,
but I know that it makes the columns that I want.

## Change the model to my project style

The next step was to change their model to the style that I use in my project.
At first, I made a class in my style and named it `NormalPrimus`.
Here is the code for it:

```python
class NormalPrimus(nn.Module):

    def __init__(
            self,
            input_dim: int,
            num_classes: int,
            d_model: int,
            n_heads: int,
            num_layers: int,
            size_embeddings: int = 512,
            name: str = "model",
    ):
        super(NormalPrimus, self).__init__()
        self.input_dim = input_dim
        self.num_classes = num_classes
        self.d_model = d_model
        self.n_heads = n_heads
        self.num_layers = num_layers
        self.size_embeddings = size_embeddings
        self.name = name

        self.primus_model = MW2StackRNNPoolingMultiheadTorch(
            size_embeddings=self.size_embeddings
        )

        self.classifier = nn.Sequential(
            nn.Linear(self.size_embeddings, self.size_embeddings * 2),
            nn.ReLU(),
            nn.Dropout(0.3),
            nn.Linear(self.size_embeddings * 2, self.num_classes),
        )

    @classmethod
    def from_config(cls, cfg: DictConfig) -> "NormalPrimus":
        """Create an instance of the model from config

        Args:
            cfg (DictConfig): config Object

        Raises:
            ValueError: input_dim is missing
            ValueError: num_classes is missing
            ValueError: d_model is missing
            ValueError: n_head is missing
            ValueError: num_layers is missing

        Returns:
            RModel: Instance of RModel from config
        """
        if "model" not in cfg:
            raise ValueError("cfg.model is required")

        model_cfg = cfg.model

        if "input_dim" not in model_cfg:
            raise ValueError("cfg.input_dim is required")

        if "num_classes" not in model_cfg:
            raise ValueError("cfg.num_classes is required")

        if "d_model" not in model_cfg:
            raise ValueError("cfg.d_model is required")

        if "n_head" not in model_cfg:
            raise ValueError("cfg.n_head is required")

        if "num_layers" not in model_cfg:
            raise ValueError("cfg.num_layers is required")

        if "size_embeddings" not in model_cfg:
            model_cfg.size_embeddings = 512
            print(f"Using default size_embeddings: {model_cfg.size_embeddings}")

        if "name" not in cfg:
            name = f"{cls.__name__}"
            print(f"cfg.name wasn't defined, using default name: {name}")
        else:
            name = cfg.name

        return cls(
            input_dim=model_cfg.input_dim,
            num_classes=model_cfg.num_classes,
            d_model=model_cfg.d_model,
            n_heads=model_cfg.n_head,
            num_layers=model_cfg.num_layers,
            size_embeddings=model_cfg.size_embeddings,
            name=name,
        )

    def forward(
            self,
            x: [torch.Tensor, torch.Tensor],
    ) -> torch.Tensor:
        """Processes the data

        Args:
            x (torch.Tensor): data

        Returns:
            torch.Tensor: result of the processing
        """
        x, mask = x
        x = self.primus_model(x)["emb"]
        logits = self.classifier(x)

        return logits
```

As you can see, I used the previous templates and didn't clean
the unused arguments, but I'm going to clean them in the near future.
So, at first, I feed the data to the `primus_model` and then to the `classifier`.

After my model was ready, I should change the `checkpoint` that they have
provided, which is in `.cpkt`, to my style, which is `.pt`.
To do that, I came up with the code below:

```python

@hydra.main(version_base=None, config_name="config")
def main(cfg: DictConfig) -> None:
    path_load_pretrained_imu_encoder = (
        "/Users/ramin/ramin_programs/Files/models/primus/best_model.ckpt"
    )

    result_path = "/Users/ramin/ramin_programs/Files/models/primus/normal_primus.pt"

    model = NormalPrimus.from_config(cfg)

    dic = torch.load(
        path_load_pretrained_imu_encoder,
        map_location="cpu",
        weights_only=True,
    )
    dic = {k[12:]: v for k, v in dic["state_dict"].items() if "imu_encoder" in k}
    model.primus_model.load_state_dict(dic)

    torch.save(model.state_dict(), result_path)

    print(model)


if __name__ == "__main__":
    main()
```

I create a model in my format.
Then, I load their weights to `primus_model`.
And finally, I save the `weights` in `.pt` format to make the loading
faster, also remove unnecessary data in `.cpkt`.
Their checkpoint size was about `621 MB`, after I saved it this way,
my `.pt` is `7.7 MB`.

Now I am ready to use `transfer learning` on my model.
I just froze all the `primus_model` weights and started training.
To be honest results weren't good at all.
For the next step, I want to first `fine-tune` and then
train on all weights to see what happens.

## Final thoughts

Using a pretrained model is a good idea.
Because the model has been trained to extract some important features,
and we can use it for our new problem.
I tried
[PRIMUS](https://github.com/nokia-bell-labs/pretrained-imu-encoders)
which has gained so many great results.
Unfortunately, `transfer learning` didn't work in my case.
For the next step, I am going to try `fine-tuning` and training
on all weights to see if any improvement happens or not.
