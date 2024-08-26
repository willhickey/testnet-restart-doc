This testnet restart is NOT urgent. Follow these instructions when you have time, but don’t skip sleep or disrupt other plans for this.

## Summary
Testnet has stalled. Agave v2.0.7 and Frankendancer v0.112.20007 contain bug fixes that we expect will prevent these stalls from recurring.

|Attribute|Value|
|---------|-----|
|Validator version|Agave: v2.0.7. Frankendancer: v0.112.20007|
|Snapshot slot|289_624_982|
|Restart slot|289_624_982|
|Shred version|4084|
|Expected bank hash|EXknCC4rNBR5SyBVrUgUB3FaoGbujPMoraEjG7C49Bdk|


## Step 1. Stop validator process if you haven’t already

## Step 2. Create snapshot
    agave-ledger-tool --ledger <ledger-path> create-snapshot \
    --snapshot-archive-path  <snapshot-path> \
    --hard-fork 289624982 \
    --  289624982 <snapshot-path>

The output should include this at (or near) the end:

    Successfully created snapshot for slot 289624982, hash EXknCC4rNBR5SyBVrUgUB3FaoGbujPMoraEjG7C49Bdk: /home/sol/ledger-restart-2024-0826/snapshot-289624982-9CGdqTDdZ3DJqQtyohr9qX6NQbsDxfFrkm8FoWjMTrUS.tar.zst
Shred version: 4084

Once you have created a snapshot move all the other snapshots to a backup directory, so your snapshot directory contains only
    snapshot-289624982-9CGdqTDdZ3DJqQtyohr9qX6NQbsDxfFrkm8FoWjMTrUS.tar.zst

If you fail to create a snapshot see the appendix for possible fixes.

## Step 3: Install patched version of the validator software
Make sure to upgrade your software to a patched version:

Agave: `agave-install init v2.0.7`

Frankendancer: Update to v0.112.20007

## Step 4: Update startup config and start your validator
### Agave
Add these arguments to your validator startup script:

    --wait-for-supermajority 289624982 \
    --expected-shred-version 4084 \
    --expected-bank-hash EXknCC4rNBR5SyBVrUgUB3FaoGbujPMoraEjG7C49Bdk \


### Frankendancer
Include these lines in the `[consensus]` section of your toml file:

    [consensus]
        # ...
        wait_for_supermajority_at_slot = 289624982
        expected_bank_hash = "EXknCC4rNBR5SyBVrUgUB3FaoGbujPMoraEjG7C49Bdk"
        expected_shred_version = 4084

After making the appropriate changes start your validator as you normally would. As it starts, the validator will load the snapshot for slot `289624982` and wait for 80% of the stake to come online before producing/validating new blocks.

To confirm your restarted validator is correctly waiting for 80% stake, look for this periodic log message to confirm it is waiting:

    INFO  solana_core::validator] Waiting for 80% of activated stake at slot 289624982 to be in gossip...

And if you have RPC enabled, ask it for the current slot:

    solana --url http://127.0.0.1:8899 slot

Any number other than `289624982` means you did not complete the steps correctly.

Once started you should see log entries for “active stake” visible in gossip and “waiting for 80% of stake” to be visible. You can track these to see how the stake progresses.


***

## Appendix (use this only if step 2 failed)

If you get an error like this:

    Error: Slot 289624982 is not available

Or this:

    Unable to process blockstore from starting slot <slot> to 289624982; the ending slot is less than the starting slot. The starting slot will be the latest snapshot slot, or genesis if the --no-snapshot flag is specified or if no snapshots are found.

Your snapshots directory contains a snapshot that is for a slot `>289624982`. If you also have a snapshot for slot `<=289624982` then move snapshots for slots `>289624982` to a backup directory and run the `agave-ledger-tool` command again. If you do not have a snapshot for slot `<=289624982` then you will need to download a snapshot

If you successfully created a snapshot, resume the instructions above starting at Step 3. If you are unable to create a snapshot, follow the instructions below on downloading a snapshot.

### Step 1: Download a snapshot from a known validator

If you are unable to generate a snapshot locally for slot `289624982` you will need to download one from a known validator. Add these lines to your startup script.

    --known-validator 5D1fNXzvv5NjV1ysLjirC4WY92RNsVH18vjmcszZd8on \
    --expected-shred-version 4084 \

Remove the flag `--no-snapshot-fetch` in your startup script if it is present.

### Step 2: After download, restart

Verify that you have a new snapshot in your snapshot directory.  If the snapshot is done downloading, stop your validator process.

Add the flag `--no-snapshot-fetch` to your startup script

Start your validator again using the instructions from step 4 above.
