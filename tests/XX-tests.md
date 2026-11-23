# NUT-XX Test Vectors

### SIG_INPUTS, single key

A `Proof` locked to one key with no tags. The signature is a BIP-340 signature by the signer key over the message. BIP-340 signatures use fresh randomness, so other valid signatures exist.

```shell
signer_pubkey_xonly:
ad30cd8d48e1c22486c5272a451b3ddf8d0f1d49b2ac030e7556b441e43fec47

secret (171 characters, unescaped):
["P2PK",{"nonce":"e18a30ce60e8961b1a9bfb8e54b1d7a81d574d7361b09bbe2d02afeec4c78f7e","data":"02ad30cd8d48e1c22486c5272a451b3ddf8d0f1d49b2ac030e7556b441e43fec47","tags":[]}]

message (SHA256 of secret):
b1baa06ec250d8450e1bc0a673afb16b31354756960b95594210861851604309

signature:
6bfa8c005e955757bdd5af25ac2361bed8e20d201e3c9cdd5bbc39ae2ce12ec598d08652209ad83ad975db4cdf6e8924b80ea2067df98f126ee282590694e2cb
```

The signature was produced by a device and accepted by `mint.minibits.cash` in a `swap`.

#### SignRequest

```json
{
  "v": 1,
  "mint": "https://mint.minibits.cash/Bitcoin",
  "unit": "sat",
  "op": "swap",
  "inputs": [
    {
      "amount": 8,
      "id": "01fc0ec0e59cd6fa01b7a88f8cd77fce81fd1e64bca67d752e984992b7a3c3a821",
      "secret": "[\"P2PK\",{\"nonce\":\"e18a30ce60e8961b1a9bfb8e54b1d7a81d574d7361b09bbe2d02afeec4c78f7e\",\"data\":\"02ad30cd8d48e1c22486c5272a451b3ddf8d0f1d49b2ac030e7556b441e43fec47\",\"tags\":[]}]",
      "C": "034d49e3a4f0bbd2aefa4008daa49ea63809c14c601b47128a10428e9748586f9e"
    }
  ],
  "outputs": [
    {
      "amount": 8,
      "id": "01fc0ec0e59cd6fa01b7a88f8cd77fce81fd1e64bca67d752e984992b7a3c3a821",
      "B_": "02fc2425b9da63ed92c5bc98e8d800ccffd7a2818fe32045aa00b97d66f115e2ba"
    }
  ],
  "pubkey": "02ad30cd8d48e1c22486c5272a451b3ddf8d0f1d49b2ac030e7556b441e43fec47"
}
```

#### SignResponse

```json
{
  "v": 1,
  "pubkey": "ad30cd8d48e1c22486c5272a451b3ddf8d0f1d49b2ac030e7556b441e43fec47",
  "signatures": [
    {
      "input": 0,
      "sig": "6bfa8c005e955757bdd5af25ac2361bed8e20d201e3c9cdd5bbc39ae2ce12ec598d08652209ad83ad975db4cdf6e8924b80ea2067df98f126ee282590694e2cb"
    }
  ]
}
```

The `Proof.witness` that `Alice` assembles from this response is:

```json
"{\"signatures\":[\"6bfa8c005e955757bdd5af25ac2361bed8e20d201e3c9cdd5bbc39ae2ce12ec598d08652209ad83ad975db4cdf6e8924b80ea2067df98f126ee282590694e2cb\"]}"
```
