# ec513_final_project

# Building Black-Parrot-Sim

In your home directory, clone the repository. 

```
cd [ec513 user directory]
git clone -b ec513 git@github.com:JefferyLim/black-parrot-sim.git
```

You will need to build a docker or singularity image. 

## Building Image
I've created a Singularity definition file which can be built on SCC for use on the SCC. In order to build, follow the instructions from an scc1 node.
```
ssh scc-i01
mkdir $TMPDIR/$USER
SING_DIR=$TMPDIR/$USER
cd $SING_DIR

git clone -b ec513 git@github.com:JefferyLim/black-parrot-sim.git
cd black-parrot-sim

make singularity-image
```

This will produce `black-parrot-sim.sif` file, which is the singularity image file. Copy this file to your own directory. You will need this when you begin building black-parrot-sim and running simulations.

## Prepping 

If you are on a Desktop node, you can run `make singularity-run` and then follow the [instructions](https://github.com/JefferyLim/black-parrot-sim/tree/ec513#tire-kick) in the black-parrot-sim repo. Keep in mind, you cannot do this on the scc1 login node, and it must be a desktop node or job.

Instead, you could submit a batch job to SCC to have it build. There are scripts already in the scc directory to do this.

`qsub scc/blackparrotbuild.sh`

# Running 

I haven't quite figured out the SDK, but there are example programs here: https://github.com/black-parrot-sdk/bp-demos/tree/ffcb9d9d8365d686f9b10e7491187654c9e0f4f7


# Software 

https://github.com/black-parrot-sdk/black-parrot-sdk#building-the-sdk
