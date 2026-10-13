---
date: '2026-10-13T15:56:00+03:30'
draft: false
title: "Got10k dataloader using PyTorch for siamese like training"
description: "A post about how I currently load Got10k to load my dataset to train a Visual Object Tracking in a Siamese-like style"
tags: [ "VOT", "GOT10k", "PyTorch" ]
image: "got10k-dataloader.webp"
---

# Got10k dataloader using PyTorch for siamese like training

## Introduction

I'm currently working on my PhD proposal.
My idea is to add context to Visual Object tracking in order to increase their robustness.
As the first step, I am designing a convenient way to load Got10k using PyTorch dataset.

## Current implementation

Here is my current implementation of it:

```python
from pathlib import Path

import polars as pl
import torch
from PIL import Image
from torch.utils.data import Dataset
from torchvision.transforms import Compose, Resize, ToTensor


class GOT10kDataset(Dataset):
    def __init__(
            self,
            root_path: Path,
            frame_type: str = "jpg",
            image_width: int = 448,
            image_height: int = 448,
            how_many_sequences: int = 0,
            normalize_bbox: bool = True,
            template_cropping: bool = True,
            transform=None,
            target_transform=None,
    ):
        self.root_path = root_path
        self.frame_type = frame_type
        self.image_width = image_width
        self.image_height = image_height
        self.how_many_sequences = how_many_sequences
        self.normalize_bbox = normalize_bbox
        self.template_cropping = template_cropping
        self.transform = transform
        self.target_transform = target_transform

        self.resize_transform = Compose(
            [
                Resize((image_height, image_width)),
                ToTensor(),
            ]
        )

        self.data = []

        i = 0

        # Can't use enumerate, because we also iterate through files
        # not all the iteration units are directories
        for folder in self.root_path.iterdir():
            if folder.is_dir():
                if (self.how_many_sequences != 0) and (i >= self.how_many_sequences):
                    break

                grand_truth_path = folder / "groundtruth.txt"
                grand_truth = pl.read_csv(grand_truth_path, has_header=False)

                frame_paths = list(folder.glob(f"*.{frame_type}"))
                frame_paths.sort()

                template_path = frame_paths[0]
                template_bbox = grand_truth.row(0)

                search_paths = frame_paths[1:]
                search_bboxes = grand_truth.rows()[1:]

                for search_path, search_bbox in zip(search_paths, search_bboxes):
                    self.data.append(
                        {
                            "sequence_path": folder,
                            "sequence_id": folder.name,
                            "template_path": template_path,
                            "template_bbox": template_bbox,
                            "search_path": search_path,
                            "search_bbox": search_bbox,
                        }
                    )

                i += 1

    def __len__(self):
        return len(self.data)

    def _load_image(
            self,
            path: Path,
            template_bbox: tuple = None,
    ) -> tuple[torch.Tensor, int, int]:
        with Image.open(path) as img:
            image = img.convert("RGB")
            width, height = image.size

        if (template_bbox is not None) and self.template_cropping:
            x, y, w, h = template_bbox
            image = image.crop((x, y, x + w, y + h))

        if self.transform:
            tr_image = self.transform(image)
        else:
            tr_image = self.resize_transform(image)
        return tr_image, width, height

    def _convert_bbox(self, bbox, original_width, original_height):

        x, y, w, h = bbox

        new_x = (x * self.image_width) / original_width
        new_y = (y * self.image_height) / original_height
        new_w = (w * self.image_width) / original_width
        new_h = (h * self.image_height) / original_height

        if self.normalize_bbox:
            new_x = new_x / self.image_width
            new_y = new_y / self.image_height
            new_w = new_w / self.image_width
            new_h = new_h / self.image_height

        return torch.Tensor(
            (
                new_x,
                new_y,
                new_w,
                new_h,
            )
        )

    def __getitem__(
            self,
            index: int,
    ):
        sample = self.data[index]

        template_frame, width, height = self._load_image(
            sample["template_path"], sample["template_bbox"]
        )
        template_bbox = self._convert_bbox(sample["template_bbox"], width, height)
        search_frame, width, height = self._load_image(sample["search_path"])
        search_bbox = self._convert_bbox(sample["search_bbox"], width, height)

        if self.target_transform:
            template_bbox = self.target_transform(template_bbox)
            search_bbox = self.target_transform(search_bbox)

        return {
            "sequence_id": sample["sequence_id"],
            "template_frame": template_frame,
            "template_bbox": template_bbox,
            "search_frame": search_frame,
            "search_bbox": search_bbox,
        }

# -------------------[ Ramin Zarebidoky ]-------------------
# -------------------[ LiterallyTheOne.github.io ]-------------------

```

As you can see, my current version does a simple job with some extra arguments that comes handy.
The main approach is that I get the `root_path`.
Then, I iterate through all the directories on it.
Each directory contains a sequence of frames and a file called `grandtruth.txt`.
I load the `grandtruth.txt`, which is in csv format, using `polars`.
I reserve the first frame's path and the first bounding box as `template_path` and `template_bbox`.
After that, I consider all the other frames' paths and bounding boxes as `search_paths` and `search_bboxes`.
As you can see, I pair the `template` with one of the `search` frames and store them in `self.data`.

When someone iterates through my dataset, in each iteration it would get a dictionary.
This dictionary contains:

* `sequence_id`: name of the directory
* `template_frame`: template frame
* `template_bbox`: bounding box of the template
* `search_frame`: search frame
* `search_bbox`: bounding box of the search frame

I have designed the `_load_image` function to load the image and conver it to `Tensor`.
As you can see, I load the frame using `PIL.Image` and make sure it is `RGB`.
Then, I save the original width and height to then use it for resizing the bounding boxes,
which I'm going to explain later.
As you can see, I decided that cropping and preparing the data is the job of my `PyTorch` Dataset.
In my opinion, model should not handle these kind of things.
So, I added the *cropping* that my model expects in this function.
I should definitely make the *cropping* process a `callable` that can be changed
for each experiment, which I'm going to work on it later.
As you can see, right now I only crop the template to get the selected object.
There are some things that I should really add:

* I should not strictly crop around the object, I should leave some background info in the template
* I should also crop `search_frame` as well to not searching the **full frame** for the object

After cropping, in `_load_image`, I deal with the transforms.
My assumption is that user always expects to have `PIL.Image` as its first input of transformations.
So, in order to not combine the default `resize_transform` and the user's `transform` I only run one of them.
I expect that if there is a `Resize` transform, user specify the resize width and height at
`image_width`, `image_height` arguments.

I also have designed a function to convert the bounding box to the correct
aspect ratio after resizing.
It is pretty simple, I divide the current coordinates with the original width and height.
Then, multiply them with the current width and height.
Because, the output of *regression models* are mostly in a range of `[0,1]`,
I made an option to normalize the bbox if it's needed.

## Conclusion

Here is what I have programmed for the Got10k data loader using PyTorch.
For the first step, I plan to train my model in siamese fashion.
Then, I would train it on individual sequences and look at the problem of VOT as a sequence of frames problem,
not as template, search region comparison.
This seeing all the frames in the sequence is super important for me, because in a sequence context have more meanings.