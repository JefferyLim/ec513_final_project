# ec513_final_project

# Building Black-Parrot-Sim

In your home directory, clone the repository. 

```
cd [ec513 user directory]
git clone -b ec513 git@github.com:JefferyLim/black-parrot-sim.git
```
If you don't have SSH keys installed:
```
cd [ec513 user directory]
git clone -b ec513 https://github.com/JefferyLim/black-parrot-sim
```

You will need to build a docker or singularity image. 

## Building Image
I've created a Singularity definition file which can be built on SCC for use on the SCC. In order to build, follow the instructions from an scc1 node.
`ssh scc-i01`

```
mkdir $TMPDIR/$USER
SING_DIR=$TMPDIR/$USER
cd $SING_DIR

git clone -b ec513 https://github.com/JefferyLim/black-parrot-sim
cd black-parrot-sim

make singularity-image
```


This will produce `black-parrot-sim.sif` file in the `docker` directory, which is the singularity image file. Copy this file to the repo in your ec513 directory. `cp [Location of your black-parrot-sim repo]/docker` You will need this when you begin building black-parrot-sim and running simulations.

## Prepping 

If you are on a Desktop node, you can run `make singularity-run` and then follow the [instructions](https://github.com/JefferyLim/black-parrot-sim/tree/ec513#tire-kick) in the black-parrot-sim repo. Keep in mind, you cannot do this on the scc1 login node, and it must be a desktop node or job.

Instead, you could submit a batch job to SCC to have it build. There are scripts already in the scc directory to do this.

`qsub scc/blackparrotbuild.sh`

# Running 

I haven't quite figured out the SDK, but there are example programs here: https://github.com/black-parrot-sdk/bp-tests


# Software 

https://github.com/black-parrot-sdk/black-parrot-sdk#building-the-sdk

# Important Discoveries

Traces are done now by TAG=[TRACE NAME]

# Maybe Important Links

https://github.com/black-parrot/black-parrot/blob/master/docs/eval_guide.md

https://github.com/black-parrot/black-parrot/blob/master/docs/testbench_guide.md

# Tracers

Some notes on tracers:
There is an trace called `i_cache_trace_p`

You can find it as a parameter in the [testbench.sv](https://github.com/black-parrot/black-parrot/blob/45a28bf96e58f55687f8e09b2521ceada121ad95/bp_top/test/tb/bp_tethered/testbench.sv#L27)

You can find it used in this declaration of [bp_fe_nonsynth_icache_tracer](https://github.com/black-parrot/black-parrot/blob/45a28bf96e58f55687f8e09b2521ceada121ad95/bp_top/test/tb/bp_tethered/testbench.sv#L351)

I'm unfamiliar with bind, but I assume it somehow creates both the bp_fe_icache and bp_fe_nonsynth_icache_tracer at the same time.

The tracer is found [here](https://github.com/black-parrot/black-parrot/blob/45a28bf96e58f55687f8e09b2521ceada121ad95/bp_fe/test/common/bp_fe_nonsynth_icache_tracer.sv#L5). It's simply just writes the input/outputs to a file using ` $fwrite`. It seems like if we want to "peek" inside the module, we will have to add new output ports to read internal parameters in the module

# Privilege Modes

We found the [CSR](https://github.com/JefferyLim/black-parrot/blob/45a28bf96e58f55687f8e09b2521ceada121ad95/bp_be/src/v/bp_be_calculator/bp_be_csr.sv#L74) that tells us if we're in privilege or non-privilege mode. There's 3 modes (Machine, Supervisor, User). It seems like the instruction "mret" or "sret" gets us into user mode, but that's as far as we've seen.  The location of the csr is along the execution pipelines, under the system pipeline.

I found some code that looks like it does things with privilege modes: https://github.com/black-parrot-sdk/bp-tests/blob/master/src/misaligned_instructions_virtual_memory.c

