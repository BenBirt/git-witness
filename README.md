# git-witness

An independent [witness](https://github.com/transparency-dev/witness) for [git-ratchet](https://github.com/project-oak/git-ratchet) transparency logs.

## Overview

This repository operates as a **transparency log witness** — it cosigns checkpoints from other repositories that use [git-ratchet](https://github.com/project-oak/git-ratchet) to maintain tamper-evident records of their Git refs (branches and tags).

Witnesses play a critical role in preventing **split-view attacks**, where a log operator might present different versions of history to different users. By independently verifying and cosigning checkpoints, this witness provides assurance that the observed log state is globally consistent.

## How it works

1. A monitored repository (e.g. `project-oak/git-ratchet`) publishes a new checkpoint by opening an issue on this repository with the title prefix `checkpoint:`.
2. A [GitHub Actions workflow](.github/workflows/cosign.yml) is triggered, which uses the [`project-oak/git-ratchet/actions/cosign`](https://github.com/project-oak/git-ratchet/tree/main/actions/cosign) action to verify and cosign the checkpoint.
3. The cosigned checkpoint is committed to the [`checkpoints/`](checkpoints/) directory, recording the witnessed state of the monitored repository's ref.

## Repository structure

```
├── .github/workflows/
│   └── cosign.yml            # GitHub Actions workflow for cosigning checkpoints
└── checkpoints/
    └── github.com/
        └── <org>/<repo>/
            ├── vkeys.txt     # Verification keys for the monitored repository
            └── refs/
                ├── heads/    # Cosigned branch checkpoints
                └── tags/     # Cosigned tag checkpoints
```

## Verification

The public verification key for this witness is:

```
github.com/BenBirt/git-witness+ca289127+BNn4Sjn2yt3uHK8N7nFgWtZccQoei3GAxWJjZN9Uc3tv
```

Each checkpoint file in `checkpoints/` contains:
- The repository and ref being witnessed
- The commit hash at the time of witnessing
- Signatures from both the origin repository and this witness

## License

This project is licensed under the [MIT License](LICENSE).
