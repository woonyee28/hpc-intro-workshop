# NASA Parallel Benchmark (NPB) on ASPIRE2A

## 1. Login ASPIRE2A
a. Open your Command Prompt (Windows), Terminal (Mac), or MobaXterm
```
ssh <nscc_username>@aspire2antu.nscc.sg
```

b. See all available modules
```
module avail
```

## 2. Configure NPB

a. clone the project files
```
wget https://www.nas.nasa.gov/assets/npb/NPB3.4.3.tar.gz
tar -zxvf NPB3.4.3.tar.gz
cd NPB3.4.3/NPB3.4-OMP/
```

b. create the `make.def` file to configure the build for NVIDIA HPC compilers:
```
cat > config/make.def <<'EOF'
FC = nvfortran
FLINK = $(FC)
F_LIB =
F_INC =
FFLAGS = -O3 -mp
FLINKFLAGS = $(FFLAGS)
CC = nvc
CLINK = $(CC)
C_LIB = -lm
C_INC =
CFLAGS = -O3 -mp
CLINKFLAGS = $(CFLAGS)
UCC = gcc
BINDIR = ../bin
RAND = randi8
WTIME  = wtime.c
EOF
```

c. create the `suite.def` file to build benchmark with the `S` and `W` problem sizes:
```
cat > config/suite.def <<'EOF'
bt  S
bt  W
bt  A
bt  B
EOF
```

d. compile the benchmarks
```
module purge
module load nvhpc
make -j suite
```

## 3. Interactive session in Compute Node
a. request for an interactive job
```
qsub -I -l select=1:ncpus=16 -l walltime=1:00:00
```

b. move to the bin directory 
```
cd hpc-intro-workshop/NPB3.4.3/NPB3.4-OMP
```

c. run the benchmark
```
module purge
module load nvhpc

OMP_NUM_THREADS=16 OMP_PROC_BIND=close numactl -m0 ./bin/bt.S.x 
OMP_NUM_THREADS=16 OMP_PROC_BIND=close numactl -m0 ./bin/bt.W.x 
OMP_NUM_THREADS=16 OMP_PROC_BIND=close numactl -m0 ./bin/bt.A.x 
OMP_NUM_THREADS=16 OMP_PROC_BIND=close numactl -m0 ./bin/bt.B.x 
```