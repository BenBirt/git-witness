# git-witness

An independent [witness](https://github.com/transparency-dev/witness) for [git-ratchet](https://github.com/project-oak/git-ratchet) transparency logs.

## Overview

This repository operates as a **transparency log witness** — it cosigns checkpoints from other repositories that use [git-ratchet](https://github.com/project-oak/git-ratchet) to maintain tamper-evident records of their Git refs (branches and tags).

Witnesses play a critical role in preventing **split-view attacks**, where a log operator might present different versions of history to different users. By independently verifying and cosigning checkpoints, this witness provides assurance that the observed log state is globally consistent.

It speaks the C2SP [tlog-witness](https://c2sp.org/tlog-witness) protocol. What is unusual is only the wire: an issue carries the request and a comment carries the response, in place of a POST and its reply. A witness reached this way answers exactly as an HTTP one would, because it runs the same [transparency-dev/witness](https://github.com/transparency-dev/witness) handler behind that transport.

## How it works

1. A monitored repository publishes a new checkpoint by opening an issue here, titled `checkpoint: <origin>`, with the `add-checkpoint` request in a fenced `http` block.
2. The [workflow](.github/workflows/cosign.yml) runs [`actions/cosign`](https://github.com/BenBirt/git-ratchet/tree/main/actions/cosign), which checks the origin is registered, answers the request, and commits the resulting state.
3. The response is posted as a comment — whatever its status, since a refusal is an answer the origin needs to read — and the issue is closed.

Both messages are HTTP messages serialised as `message/http` ([RFC 9112](https://www.rfc-editor.org/rfc/rfc9112.html)), so the exchange in each issue reads as the HTTP request and response it stands for.

## Repository structure

```
├── .github/workflows/
│   └── cosign.yml     # Cosigns checkpoints submitted as issues
├── origins/
│   └── <origin>       # Verifier keys for a registered origin, one per line
└── checkpoints/
    └── <origin>       # The last checkpoint cosigned for that origin's log
```

`<origin>` is the log's origin identifier, so a witness for a log named
`github.com/example/repo` has `origins/github.com/example/repo` and
`checkpoints/github.com/example/repo`. The intermediate directories come from
the origin's own name.

There is one log per monitored repository, so there is one state file per
origin: the ratchet is over the log's tree size, not over any one ref.

## Registering an origin

An origin is registered by adding its verifier keys, one per line, at
`origins/<origin>`. Lines beginning with `#` are comments. A request from an
unregistered origin is answered `404 Not Found` and no state is written.

## Verification

The public verification key for this witness is:

```
github.com/BenBirt/git-witness+ca289127+BNn4Sjn2yt3uHK8N7nFgWtZccQoei3GAxWJjZN9Uc3tv
```

Each file in `checkpoints/` is a cosigned [tlog-checkpoint](https://c2sp.org/tlog-checkpoint): the log's origin, its tree size, its root hash, and signatures from both the log and this witness.

## License

This project is licensed under the [MIT License](LICENSE).
