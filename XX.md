# NUT-XX: External Signer Requests for P2PK

`optional`

`depends on: NUT-10, NUT-11`

---

This NUT defines a transport-neutral message format that lets a wallet ask an external signer (a hardware device, an air-gapped phone, a separate process) for the signatures that [NUT-11][11] requires, without the wallet holding the signing key. It also defines the rules a signer MUST follow before it signs.

Mints are not affected. This NUT changes nothing in the mint API.

## Motivation

[NUT-11][11] locks `Proofs` to public keys. If the wallet also holds the private key, a compromised wallet can spend everything locked to it. With an external signer the key stays on a separate device and a person can review what is being signed. Combined with [NUT-11][11] multisig (`pubkeys` and `n_sigs`), a token can require both a wallet key and a device key.

There is currently no standard way for a wallet to request a signature, so each wallet and each signer needs custom code for the other.

## Roles

- `Alice` is the wallet. She builds the transaction and the request, and assembles `Proof.witness`.
- The `Signer` holds a private key and produces signatures. A person operates the signer.

This NUT does not define a transport. A request and a response MAY travel over any channel (USB, QR codes, NFC, files, a relay). When a message is written as text, it SHOULD be UTF-8 JSON.

## Sign request

`Alice` sends the `Signer` a `SignRequest`:

```json
{
  "v": 1,
  "mint": <str>,
  "unit": <str>,
  "op": <str_enum["swap", "melt"]>,
  "inputs": <Array[Proof]>,
  "outputs": <Array[BlindedMessage]>,
  "quote": <str|null>,
  "pubkey": <hex_str|null>
}
```

- `v` is the format version. This NUT defines version `1`.
- `inputs` are the `Proofs` being spent (see [NUT-03][03] and [NUT-05][05]). `Proof.id` MUST be the full keyset ID (see [NUT-02][02]).
- `outputs` are the `BlindedMessages` of the transaction. For `melt` they are the blank outputs of [NUT-08][08]. `outputs` MUST be present if any input has the signature flag `SIG_ALL`, and is otherwise OPTIONAL.
- `quote` is the quote ID. It MUST be present if `op` is `melt` and any input has `SIG_ALL`.
- `pubkey` is OPTIONAL and names the key `Alice` expects to sign.

The `Signer` MUST NOT base a security decision on `mint`, `unit`, `op` or `pubkey`. They are informational. The security-relevant fields are `secret`, `C`, `amount`, `B_` and `quote`.

## Sign response

If the `Signer` signs, it responds with a `SignResponse`:

```json
{
  "v": 1,
  "pubkey": <hex_str>,
  "signatures": [
    {
      "input": <int>,
      "sig": <hex_str>
    },
    ...
  ]
}
```

- `pubkey` is the x-only (32 byte) public key that produced the signatures.
- `input` is the index into `inputs`. For `SIG_ALL` there is a single entry with `input` set to `0`.
- `sig` is a hex-encoded 64 byte BIP-340 Schnorr signature, as specified in [NUT-11][11].

If the `Signer` refuses, it responds with a `SignError`:

```json
{
  "v": 1,
  "error": {
    "code": <str>,
    "message": <str>
  }
}
```

The error codes below are local to this exchange. They are not mint error codes and are not part of `error_codes.md`.

| code          | meaning                                                    |
| ------------- | ---------------------------------------------------------- |
| `cancelled`   | the person declined                                        |
| `unsupported` | the `Signer` cannot handle this secret                     |
| `not_signer`  | the `Signer`'s key cannot sign for an input                |
| `malformed`   | the request or a secret breaks a [NUT-11][11] rule         |
| `too_large`   | the `Signer` cannot display everything it is asked to sign |

A `Signer` returns only its own signatures. `Alice` assembles `Proof.witness` as a serialized `P2PKWitness` (see [NUT-11][11]) from the signatures of all required signers. If several signers are required, `Alice` sends one request to each.

## Message to sign

The `Signer` MUST compute the message itself from the request data. It MUST NOT accept a precomputed message from `Alice`.

- `SIG_INPUTS` (the default): for each input the message is the `SHA256` hash of `secret`, where `secret` is the **unescaped** string in `Proof.secret`.
- `SIG_ALL`: the message is the `SHA256` hash of the aggregate message defined in [NUT-11][11] for `swap` or `melt`.

## Signer requirements

A `Signer`:

1. MUST parse each input's `secret` as a [NUT-10][10] secret.
2. MUST refuse with `malformed` if a secret is malformed according to [NUT-11][11]. This includes a spending condition tag that appears more than once or has no value, an unknown `sigflag`, an `n_sigs` or `n_sigs_refund` that is not a positive integer or exceeds the number of keys in its pathway, and a key that appears more than once in a pathway.
3. MUST only sign secrets of kind `P2PK`, and only if its own key appears in `data`, `pubkeys` or `refund`. Keys are compared by their x-coordinate as defined in [NUT-11][11]. If the `Signer` cannot sign for an input, it MUST refuse the whole request.
4. MUST, if any input has `SIG_ALL`, verify that all inputs have the same kind, `data` and `tags`, and otherwise refuse with `malformed`.
5. MUST display the entire secret of every input it signs, including all tags, before signing. If it cannot, it MUST refuse with `too_large` and MUST NOT display a truncated secret.
6. MUST, for `SIG_ALL`, display the number of outputs and the amount of each output, and SHOULD display the `quote` for a `melt`.
7. MUST require an explicit action by the person for every signature. Automatic approval is not permitted.
8. SHOULD warn the person if `locktime` is present and no `refund` tag is present, because once the lock has expired the `Proof` is spendable by anyone.
9. SHOULD use fresh auxiliary randomness for each signature.
10. MUST NOT reveal or export private key material in any response.

## Security considerations

- **Outputs are not covered by `SIG_INPUTS`.** A signature approves spending a `Proof`, not where the value goes. A compromised `Alice` can swap to outputs she controls.
- **`SIG_ALL` covers outputs, but they are blinded.** The `Signer` cannot tell whose outputs they are. It can only display their number and amounts.
- **Display is the control.** On a device with a small screen, requirement 5 is the main protection. A `Signer` that signs what it did not display is no better than a software key.
- **Replay.** A `SIG_INPUTS` signature is valid for any spend of that `Proof`. A mint accepts a `Proof` only once, so the practical risk is a malicious `Alice` choosing the transaction, which is the first point.
- **The `Signer` sees secrets and amounts.** This is expected, because the `Signer` belongs to the owner of the `Proofs`.

## Test vectors

Test vectors are in [tests/XX-tests.md](tests/XX-tests.md).

[02]: 02.md
[03]: 03.md
[05]: 05.md
[08]: 08.md
[10]: 10.md
[11]: 11.md
