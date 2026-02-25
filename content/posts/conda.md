---
title: "Ephemeral conda enviroments"
date: 2026-02-20
summary: "I bet you’re also tired of cloning repos online that use conda as package manager"
tags: ["developement", "blog", "conda"]
---

I bet you’re also tired of cloning repos online that use conda as package manager. Nowadays, the community is shifting towards uv.

Sometimes you just want to try a piece of software and installing conda feels like overkill.

The goal of this page is to teach you to create conda environments on demand, that can be deleted without residue. No shell mutation. No base auto-activation. No fixed installation of conda needed. You download conda, create the environment you need, after use, when you turn off your machine, it will disappear.
The Idea

I break down the idea in steps:

1. First we will download conda and put it into /tmp (remember we don’t want it to persist)
2. We will install it as usual without always pressing a to accept the ToS…
3. We will load conda functions in our current shell
4. We will create our environment
5. We will activate our environment

# The Code

Download the repo which you are interested in, make sure there is an environment.yaml. cd inside it and run this one-liner in your shell.

```bash
wget -qO /tmp/miniconda.sh https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh && \
bash /tmp/miniconda.sh -b -p /tmp/miniconda && \
source /tmp/miniconda/etc/profile.d/conda.sh && \
conda env create -f environment.yaml && \
conda activate "$(sed -n 's/^name:[[:space:]]*//p' environment.yaml)"
```

# The Explanation

1. `wget ...` downloads the Miniconda installer and saves it to /tmp.

2. `bash ... -b -p /tmp/miniconda.sh` installs Miniconda in batch mode to a fixed prefix (read the Note*)

    - `-b` disables all interactive prompts and license confirmation.
    - `-p` forces installation into the specified directory and prevents shell modification.

3. `source .../conda.sh` loads conda functions into the current shell without activating base.

4. `conda env create -f environment.yaml` materializes the environment declared in YAML.

5. `conda activate ...` activates the environment explicitly by looking procedurally inside your environment.yaml through sed (which looks for the correct env name).

At no point is the system shell altered. conda remains inert unless sourced. Now you’re ready to use your evironment as you want and whenever you turn off your machine Miniconda will disappear.
Note* Note

{{< alert >}}
if during installation the script prompts you for accepting the ToS you can put yes a | before the installation command
{{< /alert >}}

