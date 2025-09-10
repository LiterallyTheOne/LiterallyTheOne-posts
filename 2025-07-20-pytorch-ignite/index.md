---
date: '2025-07-20T09:20:00+03:30'
draft: false
title: 'Trying PyTorch Ignite'
description: "A post about trying PyTorch Ignite"
tags: ["Pytorch-Ignite", "PyTorch", "Python", "Deep-Learning"]
image: "Trying-PyTorch-Ignite.webp"
---

# Trying Pytorch Ignite

## Introduction

The day before yesterday, I ran into a problem.
The problem was, my training procedure on `Google Colab` was pretty slow
and barely any `GPU` ram was occupied.
But in my MacBook Pro, it was pretty fast, and it was working perfectly with
`MPS`.
At that time, I was using `Pytorch Lightning`, which has made my life
pretty easy.
In `Pytorch Lightning`, we have to create instances of `LightningModule`
for training the model and for loading data, we can optionally
use `LightningDataModule`.
I was using both, and it was great.
But when I ran into the problem in `Google Colab`, it seemed like
the debugging is a little bit hard for me.
I prefer my code to be completely modular and have the ability to
be separated into different parts with ease.
So yesterday, I gave `Pytorch ignite` a shot.
I have started with the [quick start](https://pytorch-ignite.ai/tutorials/beginner/01-getting-started/),
and it was super easy
to change my training procedure from `Pytorch Lightning` to `Pytorch Ignite`.
To be honest, I have a really great feeling about `Pytorch ignite`.
I strongly recommend that you give it a shot.
Here is the link to the quick start again:
[quick start](https://pytorch-ignite.ai/tutorials/beginner/01-getting-started/)

## My setup

At first, I have defined my `trainer` and `evaluators` like below:

```python
# Define trainer
trainer = create_supervised_trainer(
    model=model,
    optimizer=optimizer,
    loss_fn=criterion,
    device=device,
)

# Define evaluators
val_metrics = {
    "accuracy": Accuracy(),
    "loss": Loss(criterion),
}

train_evaluator = create_supervised_evaluator(
    model=model,
    metrics=val_metrics,
    device=device,
)
val_evaluator = create_supervised_evaluator(
    model=model,
    metrics=val_metrics,
    device=device,
)
```

Then, because I like to see the progress,
I attached `ProgressBar` to each of them.

```python
# Add progress bar to trainer
pbar = ProgressBar()
pbar.attach(trainer)

# Add progress bar to evaluators
pbar_1 = ProgressBar()
pbar_1.attach(train_evaluator)

pbar_2 = ProgressBar()
pbar_2.attach(val_evaluator)
```

To run the evaluators after each epoch ends, I have used the events
below and add them to the `trainer` handlers.

```python
# Add logging of training results to trainer
@trainer.on(Events.EPOCH_COMPLETED)
def log_training_results(engine):
    train_evaluator.run(train_loader)
    metrics = train_evaluator.state.metrics
    print(
        f"Training Results - Epoch[{engine.state.epoch}] Avg accuracy: {metrics['accuracy']:.2f} Avg loss: {metrics['loss']:.2f}"
    )


# Add logging of validating results to trainer
@trainer.on(Events.EPOCH_COMPLETED)
def log_validation_results(engine):
    val_evaluator.run(val_loader)
    metrics = val_evaluator.state.metrics
    print(
        f"Validation Results - Epoch[{engine.state.epoch}] Avg accuracy: {metrics['accuracy']:.2f} Avg loss: {metrics['loss']:.2f}"
    )
```

For the score function, I added the code below, which uses `accuracy`:

```python
def score_function(engine):
    return engine.state.metrics["accuracy"]
```

To always have the best model saved, I added a `ModelCheckPoint` to
`val_evaluator`.

```python
# Add model checkpointing to val evaluator
model_checkpoint = ModelCheckpoint(
    f"checkpoints/{cfg.model.name}_{run_counter}",
    n_saved=1,
    filename_prefix="best",
    score_function=score_function,
    score_name="accuracy",
    global_step_transform=global_step_from_engine(trainer),
)

val_evaluator.add_event_handler(
    Events.COMPLETED,
    model_checkpoint,
    {"model": model},
)
```

Then, for early stopping, I added the code below.

```python
# Add Early stopping
early_stopping = EarlyStopping(
    patience=cfg.patience,
    score_function=score_function,
    trainer=trainer,
    min_delta=cfg.min_delta,
)

val_evaluator.add_event_handler(
    Events.COMPLETED,
    early_stopping,
)
```

And, to have tensorboard logging, I added the code below:

```python
# Add tensorboard
tb_logger = TensorboardLogger(log_dir=f"tb-logger/{cfg.model.name}_{run_counter}")

tb_logger.attach_output_handler(
    trainer,
    event_name=Events.ITERATION_COMPLETED(every=100),
    tag="training",
    output_transform=lambda loss: {"batch_loss": loss},
)

for tag, evaluator in [
    ("training", train_evaluator),
    ("validation", val_evaluator),
]:
    tb_logger.attach_output_handler(
        evaluator,
        event_name=Events.EPOCH_COMPLETED,
        tag=tag,
        metric_names="all",
        global_step_transform=global_step_from_engine(trainer),
    )

```

And the only other thing that I should do, is to `run` my `trainer engine`
by the code below.

```python
trainer.run(train_loader, max_epochs=cfg.max_epochs)
```

And it works, perfectly.

`cfg` in code is the config that I am using.
I use `hydra` to control my configs.

## Final thoughts

`PyTorch Ignite` is an extremely great package to
train our models.
It is modular which I pretty much like and is closer to the
way that I think.
I strongly recommend that you check it out.
Here is the link to the official site:
[https://docs.pytorch.org/ignite/index.html](https://docs.pytorch.org/ignite/index.html)
