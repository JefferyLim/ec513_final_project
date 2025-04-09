# Setting up the Repo

In your black-parrot-sim repo:

```
cd black-parrot/
git pull
git checkout [arch_trace/uarch_trace/cache_trace]
```

Make changes to:

`bp_top/test/common/bp_nonsynth_[arch/uarch/cache]_tracer.sv`
`bp_top/test/tb/bp_tethered/testbench.sv`

# Writing New Code

I've been making changes to the `black-parrot-sdk/bp-tests/src` tests. We haven't setup a new repo to this, so for now, just keep your files local.

You have to rebuild the `bp-tests` by running the command (in the black-parrot-sdk directory).

`make rebuild.bp-tests`

or from the root repo directory:

`make -C black-parrot-sdk rebuild.bp-tests`

## Looking at the Assembly

You can do an objdump of the `prog.riscv` file inside the verilator results file. The objdump binary is located in:

`black-parrot-sim/black-parrot-sdk/install/bin/riscv64-unknown-elf-dramfs-objdump -S [binary]`

The singularity image should add it to the path, but if it can't find `riscv64-unknown-elf-dramfs-objdump`, you can manually add the path: `export PATH=$PATH:black-parrot-sdk/install/bin/` 

# Running Simulations

You don't have to add the [UARCH/ARCH/CACHE]_TRACE_P=1 flags because they are set to 1 by default.

`make -C black-parrot/bp_top/syn build_dump.verilator sim_dump.verilator`

# Commiting Code
 
We only have a local copy of black-parrot and black-parrot-sim. You should be able to git add the changes with:

```
git add -u
git commit
git push
```
