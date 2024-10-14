# Introduction

Welcome to the Meridian Certora workshop! In the next hour, we will introduce you to a new tool by Certora, called "Sunbeam" that enables formal verification of Soroban smart contracts. The goal of this workshop is to show you how to use Sunbeam on a simple example. Work on Sunbeam is ongoing, so feel free to reach out to us with questions and feature requests. We will do our best to help!

# Installing Sunbeam

1. First, we will need to install the Sunbeam prover. For that, please visit [Certora.com](https://www.certora.com/) and sign up for a free account [here](https://www.certora.com/signup).

2. You will receive an email with a temporary password and a `Certora Key`. Use the password to login to Certora following the link in the email.

3. Next, install Python3.8.16 or newer on your machine. If you already have Python3 installed, you can check the version: `python3 --version`. If you need to upgrade, follow the instructions [here](https://wiki.python.org/moin/BeginnersGuide/Download).

4. Next, install Java. Check your Java version: `java -version`. If the version is < 11, download and install Java version 11 or later from [Oracle](https://www.oracle.com/java/technologies/downloads/).

5. Then, install the Certora Prover: `pip3 install certora-cli-beta`.

6. Recall that you received a `Certora Key` in your email (Step 2). Use the key to set a temporary environment variable like so `export CERTORAKEY=<personal_access_key>`. Alternative, store the key in your profile see [here](https://docs.certora.com/en/latest/docs/user-guide/install.html#step-3-set-the-personal-access-key-as-an-environment-variable).


# Rust and Stellar CLI Setup

1. We recommend installing Rust as on the [official website](https://www.rust-lang.org/tools/install): `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`

2. Next, install the WASM target like so: `rustup target add wasm32-unknown-unknown`

3. We recommend setting up the Stellar CLI as shown [here](https://soroban.stellar.org/docs/getting-started/setup#install-the-soroban-cli).

4. We also recommend installing the [wabt](https://github.com/WebAssembly/wabt) toolkit. `wasm2wat` is a useful tool for converting the WASM bytecode to a human readable format.

5. Finally, install `rustfilt` like so: `cargo install rustfilt`.

With that, you should be all set for using Certora Sunbeam. Congratulations!

# Exercises

In this section, we will first go over an example on how to run Certora Sunbeam and then do some exercises on formal verification of a simple Soroban smart contract.

First, let's understand the what is in this Rust project directory. 

- `src/lib.rs` has a Soroban smart contract with some functionality of a Token.

- `confs/` has several configuration files to help you run Certora Sunbeam.

- `src/certora/` is where we will write the formal specs for this contract. It also has a directory of `mutants` to evaluate the specs.

- `solutions/` has solutions to some of the exercises we will do in this workshop. You can consult it if you want to know the answers.


#### Exercise 0: Warm up

Let's make sure you are able to run Certora Sunbeam. Run the following:

```
cd meridian2024-workshop
certoraRun confs/setup.conf
```
You should see an output like so:

```
INFO: Executing build process...
Executing:  cargo build --target=wasm32-unknown-unknown --release  --features cvt

Connecting to server...

Job submitted to server

Manage your jobs at https://prover.certora.com
Follow your job and see verification results at <LINK>
```
Click on that link to see the report page generated by Certora Sunbeam.
It will show you that a basic sanity check has passed. You can see the `Source Files` on the report page. There is also a `Call Trace` that shows you what sequence of function calls led to this outcome. 

`src/certora/spec.rs` will show you the `sanity` rule we just ran. This rule simply calls `Token::balance()` and checks that the control reaches the `satisfy` statement that follows. You can read more about `satisfy` [here](https://docs.certora.com/en/latest/docs/cvl/statements.html#satisfy).


If you are not able to run certoraRun, see the Troubleshooting section at the end of this document.

#### Exercise 1. A property to check that the initial balance of an account.

What should be the balance of a new address? Write a property to check that this is indeed the balance of a new address.
You can write your property (or rule) in `src/certora/spec.rs` inside the function `init_balance`. We have already provided the right signature for this rule.

Once you have written the rule, you can run Certora Sunbeam to check it by running:

```
certoraRun confs/exercise1.conf
```

<details>
  <summary>Hint</summary>
  You'll need to use `require!(<CONDITION>, "expect address to exist");` to ensure the `address` actual exists.  
</details>

You can see the solution in `solutions/solution_specs.rs`.


#### Exercise 2. Effect of transfer on the balances of various addresses

What should be the effect of a `transfer` of `amount` between two addresses, `to` and `from`?  Write a rule to capture the correct behavior. Whose balance should change by what amount?

Note that `transfer` should not affect any address other than the one being transferred to and from. Write another rule to encode the effect of `transfer` on some `other` address. 

You can write these two property in `transfer_is_correct` and `transfer_no_effect_on_other` in `src/certora/spec.rs`.

Once you have written the rule, you can run Certora Sunbeam to check it by running:

```
certoraRun confs/exercise2.conf
```

You can see the solution in `solutions/solution_specs.rs`.

#### Exercise 3. `transfer` under insufficient funds

If `from` does not have sufficient balance, `transfer` of funds should not succeed. Write a rule to capture this behavior in the function `transfer_fails_if_low_balance`

Once you have written the rule, you can run Certora Sunbeam to check it by running:

```
certoraRun confs/exercise3.conf
```

You can see the solution in `solutions/solution_specs.rs`.

#### Exercise 4. Specs for `mint` and `burn`

Now that we have seen rules for `transfer`, think of some properties for `mint` and `burn` and write them in the `src/certora/spec.rs` file. To run them,
create your own `conf` files under `confs` by looking at the existing conf files. You will only need to change the names of the rules passed into the `"rule"` field. The rest should be the same.

You can see several rules we have written for these functions in `solutions/solution_specs.rs`.

#### Exercise 5. Assessing your specs through mutation testing

How do you know if your rules are good enough to catch potential bugs? One technique is called "mutation testing" where small faults are injected in to the program and it is checked against the same rule. Verification should fail if the rule is good at catching the fault. If verification passes, that means your rule has gaps that must be addressed.

We have provided 3 hand-written mutants in `src/certora/mutants`. Copy them one at a time to `src` and rename them to `lib.rs` to replace the original `src/lib.rs`. Then run the rules you wrote above for `transfer` on these mutants. Are they caught?

Can you detect what the mutation was, for each mutant? You can see the solution in `solutions/bugs-in-mutants.md`.


Note that there are other ways to assess the quality of your rule. You can mutate the rule to see if it is vacuous, you can check if the rule is a tautology, and you can use UNSAT cores to understand what parts of the code were covered by the rule.


# Troubleshooting

If you are unable to run `certoraRun`, we recommend trying it from within a `venv`.

1. First, create a `venv` and make sure you are inside the `venv` by running the following:

```
cd meridian2024-workshop
python3 -m venv .venv
source .venv/bin/activate
```

2. Then, install all required packages like so:
```
pip3 install -r requirements.txt
``` 

3. Finally, try running certoraRun again:
```
certoraRun confs/setup.conf
```
