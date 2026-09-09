# Configuration (`config.yaml`)
OCTRON keeps a small set of user-tunable settings in a `config.yaml` file — things like where to cache downloaded models and the default train/val/test split. The same file is read by both the GUI and the [command line interface](cli.md), so a setting you change once applies everywhere.

You rarely need to edit this file by hand: the [`octron config`](#viewing-and-editing-settings) commands read and write it for you (with validation), and the GUI writes to it when you change the relevant options.

## Where it lives
The file lives in your platform's per-user configuration directory:

| Platform | Location |
| --- | --- |
| macOS | `~/Library/Application Support/octron/config.yaml` |
| Linux | `~/.config/octron/config.yaml` (honors `$XDG_CONFIG_HOME`) |
| Windows | `%LOCALAPPDATA%\octron\config.yaml` |

Run `octron config path` to print the exact location on your machine (and whether it exists yet). Set the `OCTRON_CONFIG_PATH` environment variable to point OCTRON at a different file (handy for tests or a per-project config).

The file is created automatically the first time a setting is saved; until then, OCTRON simply uses the built-in defaults.

## How values are resolved
A setting's effective value is resolved with the following precedence (later wins):

1. the **built-in default** (shipped with OCTRON),
2. the value stored in **`config.yaml`**,
3. a **per-run command-line flag** (e.g. `octron split --seed 7`), where one exists.

A malformed file never breaks OCTRON: unknown keys are ignored and invalid values fall back to their default (both with a warning).

## Viewing and editing settings
The `octron config` sub-commands are the recommended way to inspect and change settings:

| Command | What it does |
| --- | --- |
| `octron config list` | Show every setting with its current value, default and source (default vs `config.yaml`). |
| `octron config get KEY` | Print a single value to stdout — only the value, so it is safe in scripts. |
| `octron config set KEY VALUE` | Validate `VALUE` and save it to `config.yaml` (created if needed). |
| `octron config path` | Print the `config.yaml` location and whether it exists yet. |
| `octron config edit` | Open `config.yaml` in your `$EDITOR`. |

Examples:
```
# See everything at a glance
octron config list

# Cache downloaded models on shared storage
octron config set model_cache_dir /nas/octron_models

# Change the default training fraction
octron config set split_train_fraction 0.8

# Read a value in a script
SEED=$(octron config get split_seed)

# Clear a directory setting (revert to the default behavior)
octron config set prediction_cache_dir ""
```

!!! tip "`octron config list` is the source of truth"
    Available settings can change between OCTRON versions. `octron config list` always reflects exactly what your installed version supports, including each setting's description.

## Settings reference
| Key | Default | Description |
| --- | --- | --- |
| `prediction_cache_dir` | *(unset)* | Local directory used to stage [prediction](analysing.md) output before moving it to the final destination (e.g. fast NVMe scratch). Empty = write directly to the destination (caching off). Overridden per run by `octron predict --local-cache-dir`. |
| `model_cache_dir` | *(unset)* | Directory for downloaded model weights and SAM checkpoints. Empty = the per-user cache directory. Point it at shared/NAS storage to reuse downloads across machines or installs. |
| `split_train_fraction` | `0.7` | Fraction of annotated frames used for the training split. Overridden per run by `octron split`/`train --train`. |
| `split_val_fraction` | `0.15` | Fraction used for validation; the remainder becomes the test split. Overridden by `--val`. |
| `split_seed` | `88` | Random seed for the reproducible [train/val/test split](training.md#how-the-split-works). Overridden by `--seed`. |
| `split_buffer` | `1` | Frames dropped at each split block boundary to add a temporal gap between train and val/test (`0` disables). Overridden by `--buffer`. |

The split settings are explained in more detail under [Training › How the split works](training.md#how-the-split-works).
