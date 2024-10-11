# Collaborative proof of computing addition of different private inputs

Trying out co-noir after following the [Taceo zkSummit workshop](https://github.com/TaceoLabs/zksummit12_workshop/tree/main). 

Setting: 3 inputs x,y and z, we want proof that they compute addition of the inputs. 

Three parties will collaborate in creating the proof. 

Note that number of collaborating parties is fixed (3), but the number of inputs is not. (They just both happen to be 3 in this example)

## Run example 

### Install co-noir

You need to have cargo & nargo installed. This step can be followed to install co-noir. 

```bash
git clone https://github.com/TaceoLabs/collaborative-circom.git --branch co-noir-v0.2.0 && cargo install --path collaborative-circom/co-noir/co-noir/
```

### Get started

Initial setup: create necessary directories.
```bash
mkdir -p out/secret_shared_inputs out/merged_inputs out/witness out/proofs
```

Compile Noir program. 
```bash
nargo compile
```

### Secret Sharing the Input

Secret share the 3 inputs to all 3 proof-collaborators.

```bash
co-noir split-input --input Alice.toml --circuit target/simple_add.json --protocol REP3 --out-dir out/secret_shared_inputs/
co-noir split-input --input Bob.toml --circuit target/simple_add.json --protocol REP3 --out-dir out/secret_shared_inputs/
co-noir split-input --input Davina.toml --circuit target/simple_add.json --protocol REP3 --out-dir out/secret_shared_inputs/
```

### Merging the Input

Merge the inputs:

```bash
co-noir merge-input-shares --inputs out/secret_shared_inputs/Alice.toml.0.shared --inputs out/secret_shared_inputs/Bob.toml.0.shared --inputs out/secret_shared_inputs/Davina.toml.0.shared --protocol REP3 --out out/merged_inputs/input.toml.0.shared 
co-noir merge-input-shares --inputs out/secret_shared_inputs/Alice.toml.1.shared --inputs out/secret_shared_inputs/Bob.toml.1.shared --inputs out/secret_shared_inputs/Davina.toml.1.shared --protocol REP3 --out out/merged_inputs/input.toml.1.shared 
co-noir merge-input-shares --inputs out/secret_shared_inputs/Alice.toml.2.shared --inputs out/secret_shared_inputs/Bob.toml.2.shared --inputs out/secret_shared_inputs/Davina.toml.2.shared --protocol REP3 --out out/merged_inputs/input.toml.2.shared 
```

### Generating the Extended Witness

The extended witness includes not just the private input (witness) but also other additional data necessary for the proof system to work, such as intermediate computation results, auxiliary variables, and any other inputs generated or used throughout the circuit (computation process).

Compute the extended witness in 3 terminals.

Terminal 0:

```bash
# start party 0
co-noir generate-witness --input out/merged_inputs/input.toml.0.shared --circuit target/simple_add.json --protocol REP3 --config configs/party0.toml --out out/witness/witness.wtns.0.shared 
```

Terminal 1:
```bash
# start party 1
co-noir generate-witness --input out/merged_inputs/input.toml.1.shared --circuit target/simple_add.json --protocol REP3 --config configs/party1.toml --out out/witness/witness.wtns.1.shared 
```

Terminal 2:
```bash
# start party 2
co-noir generate-witness --input out/merged_inputs/input.toml.2.shared --circuit target/simple_add.json --protocol REP3 --config configs/party2.toml --out out/witness/witness.wtns.2.shared 
```

### Generating the 3 parts of the Proof

Terminal 0:
```bash
# start party 0
co-noir generate-proof --witness out/witness/witness.wtns.0.shared --crs bn254_g1.dat --protocol REP3 --config configs/party0.toml --out out/proofs/proof.0.dat --public-input out/proofs/public_input.0.json --circuit target/simple_add.json 
```

Terminal 1:
```bash
# start party 1
co-noir generate-proof --witness out/witness/witness.wtns.1.shared --crs bn254_g1.dat --protocol REP3 --config configs/party1.toml --out out/proofs/proof.1.dat --public-input out/proofs/public_input.1.json --circuit target/simple_add.json
```

Terminal 2:
```bash
# start party 2
co-noir generate-proof --witness out/witness/witness.wtns.2.shared --crs bn254_g1.dat --protocol REP3 --config configs/party2.toml --out out/proofs/proof.2.dat --public-input out/proofs/public_input.2.json --circuit target/simple_add.json
```

// TODO error:
```
Expected public value
```

### Extract the Verification Key

In contrast to `circom` where we can extract the verification key from the `zkey` with snarkJS, we need another step with `co-noir`. Generate the verification key with this command:

```bash
# Create verification key
co-noir create-vk --circuit target/simple_add.json --crs bn254_g1.dat --vk verification_key.dat
```

### Verifying the Proof with co-circom

Finally verify the proof:

```bash
# verify proof
co-noir verify --proof out/proofs/proof.0.dat --vk verification_key.dat --crs bn254_g2.dat
```