# Miscellaneous tips

## How to clear the cache of a Buildkite worker

### Requirements

- An active CSAIL Kerberos account with permission to SSH into the Julia CI machines
- `sudo` access on the appropriate CI machine

### Instructions

Step 1: Identify the worker. For example, suppose that the worker is `amdci5.2`.

Step 2: SSH into the CSAIL login node:

```
ssh YOUR_USERNAME@login.csail.mit.edu
```

Step 3: Based on the worker, SSH into the appropriate CI machine. In this example, we need to SSH into `amdci5`:

```
ssh YOUR_USERNAME@amdci5.julia.csail.mit.edu
```

Step 4: Switch to the appropriate user. You will need to enter your `sudo` password.

```
sudo su sabae
```

Step 5: Go to the `scratchspaces` directory and look for the scratchspace for the worker in question:

```
cd /home/sabae/.julia/scratchspaces
ls *
ls */buildkite-agent-cache
```

Step 6: Identify the correct scratchspace for the worker. For example, it might be something like this. The specific value that you see will differ from this example.
```
ls /home/sabae/.julia/scratchspaces/a66863c6-20e8-4ff4-8a62-49f30b1f605e/buildkite-agent-cache/amdci5.2
```

Step 7: Delete the scratchspace:
```
rm -rf /home/sabae/.julia/scratchspaces/a66863c6-20e8-4ff4-8a62-49f30b1f605e/buildkite-agent-cache/amdci5.2
```

Step 8: Re-create an empty cache folder:
```
mkdir -p /home/sabae/.julia/scratchspaces/a66863c6-20e8-4ff4-8a62-49f30b1f605e/buildkite-agent-cache/amdci5.2
```
