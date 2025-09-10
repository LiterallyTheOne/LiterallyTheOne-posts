---
date: '2025-07-13T21:04:23+03:30'
draft: false
title: 'Export Command'
tags: ['Linux']
image: "export.png"
---

# Export Command

Why should we mostly use `export` for setting environmental variables?

Every process has its own specific environmental variables and doesn't
share it with the other processes.
Imagine that we have process `a`.
In process `a` we set an environment variable called `EXAMPLE_VARIABLE=10`.
Now, process `a` has access to this variable, so if we print it out like this:

```shell
echo $EXAMPLE_VARIABLE
```

the output would be, 10.
But the subprocesses of this process don't have access to this variable.
for example, if I have `Python` file, named `export_example.py`, like below:

```python
# export_example.py

import os

print(os.getenv("EXAMPLE_VARIABLE", "Not found"))

```

Which, if `EXAMPLE_VARIABLE` is on the environmental variables, it just
prints it and if not, it says it doesn't exist.
Now if we run this python file with `python export_command.py`,
the output would be.

```text
There is no EXAMPLE_VARIABLE in environment variables
```

But, if we set our environmental variable like below:

```shell
export EXAMPLE_VARIABLE=10
```

Then run our python file, the output would be:

```text
10
```

Also, the output of `echo $EXAMPLE_VARIABLE` would be `10` to.

