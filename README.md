# git-witness

An independent [witness](https://github.com/transparency-dev/witness) for [git-ratchet](https://github.com/project-oak/git-ratchet) transparency logs.

## Overview

This repository operates as a **transparency log witness** — it cosigns checkpoints from other repositories that use [git-ratchet](https://github.com/project-oak/git-ratchet) to maintain tamper-evident records of their Git refs (branches and tags).

Witnesses play a critical role in preventing **split-view attacks**, where a log operator might present different versions of history to different users. By independently verifying and cosigning checkpoints, this witness provides assurance that the observed log state is globally consistent.

It speaks the C2SP [tlog-witness](https://c2sp.org/tlog-witness) protocol. What is unusual is only the wire: an issue carries the request and a comment carries the response, in place of a POST and its reply. A witness reached this way answers exactly as an HTTP one would, because it runs the same [transparency-dev/witness](https://github.com/transparency-dev/witness) handler behind that transport.

## How it works

1. A monitored repository publishes a new checkpoint by opening an issue here, titled `checkpoint: <origin>`, with the `add-checkpoint` request in a fenced `http` block.
2. The [workflow](.github/workflows/cosign.yml) runs [`actions/cosign`](https://github.com/project-oak/git-ratchet/tree/main/actions/cosign), which checks the origin is registered, answers the request, and commits the resulting state.
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
github.com/BenBirt/git-witness+ccaa237e+BtZXGyOO21nbPodNujc4anya6XaEIua5ZIikSfAUcf/foTmT1E0E7faSjlYDmlKUDXl8YTVhVAQvsn+C56zcoZuJoqQK8x82tdl0Eg0etqhqfY8g/mUpr5SojXju2JmbeX/rn/W8XB4KzU2VC+sWkIbJhUH6UgTOb9Scx/OPvNkamENo18xk/CdrG0pX7pbSNl1zkLBVq9arKgJhKSj7KTCGZZRLWdZD/RcVrCnQRAMxdss5HIWPkL+OL6cItrfyXqONXFLh8iiS1ZX8xsRmWxy+PUyCb9mlLOkxq4qOG+xvLc2IcRc8MeEHlA2RCtIzZKNUy0yJ7xLVycY0HQPDtTZLqWKFv4dB/Ix4T0a5q3yUJnfQ0mOvH5jiTyR+9G5kR8NLX6KWvpcynLudAv+i+ogrXT988nT2CwXJqifEa+eX6gxprQA0ahN5O3IicIbMtGH8ES/6wkkyqMFOWph80hsmm2TWMA1iO1vv9BRtBMgONIoe5+fQz1SIMASZIcXIlSdWvd6BJApoCx/C4n1+jaZX6Kp71fWd4Pp3n8vUaU5ER8Z2o6BQ8Ti80OCFVE64Ln9DlJG1H8WhKN/WIyT5SqETOuDGvz2aQLQjZmsVu80tLsjdd0NahnZxPWq54gzrZsx+Jy5plKMsyxafzxtqlynBtPJryHfYvPRnhGCEnxU4adeQLRASApXVCZUdLLHPZTmeyOiPRiKJfgEeNg8sHYImJBCEs/wzbPxESbefP2SvZl9lufAg8zHelPDzmTuNOqbCqhQWWll6FIbs/ytgyHdKIvlDrb8hiXpu3BIFi9/vhH9MWPVF13bW1i+FXg9ttWh6KE4oTMnhruHXHvUiS9PnsMxTH/mHtwtTFXpD7Ra1xCu92cQBV0rLuXw+dDgnQbNdctvOY/3Z3eUwB3OZBSEFy648Rin2FnST6b1DQjORSBrRZDrCWsq81LkODJzwlDY/Lzj9thKEPkhLfayzrBq6kXfhpalGJ640Lq74zPBpmVmZgtN4T3eSUWUqsccDVLSOuD+VNv8kv9k7Py0e73iwaueXKHMI1/1kbsCqJMbfE0BED5cZdfIyGDefYFxtGTKzIgtLP3W7GTLfrRVeAK6gYdl8OWpdpHqAEm6PF1sW4GMBeTGG+Jh2HdRMHSLrfz02OE2B6K83V8oXapvmUpSeadIfRc7SLRod6hDxI5+J/QdpnWRlbj5nHRoc1Lfh3rnJyVz74tiB9BEjpweVoKuAYwHXYiQcf9rMyUAnmSxRiZTgaurg0NNtWupkJ9mI8AS4YPwXc4q+bz/x3xAO2zmKJnlpJ86ttTW4KvCuoUcmcbQgetDX96g/bTAN/XysIs7LrdTL8rmnc3qGX232RUp2DR51Wge7XFdQjj7LSWIarPsDu1w15bpzqtpMlQzLIZanyyRbkxxr/N8ngrm5H9wtPfs49vtCeiE4qEa/FC8avNFiLlrY4ZPjKvVMVYnvGQ3y5GbMj5C7KGoa/2Op5wkukiGc97Hgt8bz5gLoiy7Rv4TWAmOie0+DP4jttiFQnB9m8s8jcPZVphUd+18Ku6grhcQ3VE6EfE9wjQYJaRZsMqBnUdF+bGFHSTHwgZkh33oV+X02yRE9qQlNQ6KsGBP6TxoFYq98yI+NBNa/PIQpR15ktoNUUU2IfNK9UbM93hrhYdxqs0flcot8DPXhxBZKpBUVn6pgDFLTpWsm7l1JKbvDV4Fpb1qG2HU7tLIGB4CZb0XmMurVUFFVoM5kAjs=
```

Each file in `checkpoints/` is a cosigned [tlog-checkpoint](https://c2sp.org/tlog-checkpoint): the log's origin, its tree size, its root hash, and signatures from both the log and this witness.

## License

This project is licensed under the [MIT License](LICENSE).
