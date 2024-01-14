---
layout: post
title: "Genetically Optimized Compiler Flags"
description: "Optimizing GCC compiler flag selection using genetic algorithm"
meta: "gcc,optimize,compiler flags,genetic algorithm"
math: true
---


### What is a compiler?
A *compiler* is responsible for translating code written in a high level programming language, like C, 
to machine level language which are a bunch of 1's and 0's the hardware can understand. During this translation a compiler performs 
various types of optimization to transform the code without changing its behaviour and produce an executable 
file (containing machine level code) which is efficient. This efficiency can be related to execution time, 
file size, power consumption, etc. 

Consider this example,
```c
int main() {
    int a, b = 10, c = 20;
    a = b + c;
    printf("a = %d\n", a);
    return 0;
}
```

During compilation, the compiler will observe that the variables `b` and `c` are not being used after the expression `a = b + c`.
it'll try to optimize the code by not allocating any additional memory for the variables `b` and `c`. 

In essence the code will become,

```c
int main() {
    int a;
    a = 10 + 20;
    printf("a = %d\n", a);
    return 0;
}
```

### GCC Compiler flags

GCC Compiler flags offer control over which optimizations should be enabled/disabled during the compilation of a program. 
It offers ~60 flags related to different types of optimization, a list of these flags can be found 
[here](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html). It also offers certain preset optimizations like - O1, O2, O3. 
These flags can affect the execution time, binary file size, power consumption, etc. 

These preset optimizations do not always provide the most optimal executables and thus we require a more fine grain control over
which optimizations are enabled. The problem is to find these set of optimal compiler flags, with around 60 compiler flags offered by
GCC we are looking at **2<sup>60</sup> combination of flags**. 


### Using Genetic Algorithm
A large search space of about 2<sup>60</sup> combination of flags makes it impossible to try all possibilities, 
enter *evolutionary algorithm*. 

An evolutionary algorithm starts with a random set of population and over generations of selection, cross-over and mutation tries to 
converge to a *global optimal solution*. Each member of this population has a DNA which is a binary string of 58 
characters corresponding to the compiler flags.

Pseudo code
```python
init_population()
calculate_fitness()
while generation < MAX_GENERATIONS:
    perform_selection()
    perform_mutation()
    calculate_fitness()
    Selection involves,
```

* *Selection* involves

    - *Elitism*, maintaining the top *10%* of the population of current generation in the next generation

    - *Crossover*, selecting two parents and producing a child using one point cross over with *60%* probability. 

* *Mutation* performs a bit-flip at a random position in the DNA of a member with *1%* probability.

#### Fitness
> *Survival of the fittest!*

The fitness of each member in the population determines which members cross-over and/or move forward in the next generation. 
For our use case the fitness is calculated based on the execution time.

$$ fitness = \frac{1}{execution\_time} $$ 

In python this is implemented using the subprocess module,

```python
# take average execution time
stmt = 'subprocess.run({}, stderr=subprocess.STDOUT,\
        stdout=subprocess.DEVNULL, check=True)'.format(cmd_list)
        return timeit.timeit(stmt=stmt,
                             setup='import subprocess',
                             number=iterations) / iterations
```

#### Selection
The selection process involves picking two members to possibly crossover and produce a child which will move to the next generation.
This process is carried out using [roulette wheel selection](http://www.edc.ncl.ac.uk/highlight/rhjanuary2007g02.php) algorithm.

#### Crossover
Crossover is performed by taking first half of parent 1's DNA and second half of parent 2's DNA to produce a new member. 
It occurs for only *60%* of the population.  
```
 new_DNA = parent1_DNA[:mid] + parent2_DNA[mid:]
```

#### Mutation
Mutation will provide the necessary diversity that is required for evolution in any population. A one-bit flip mutation is used in our 
case wherein a single flag will be set or reset for *1%* of the population.

### Conclusion
The algorithm is implemented using *Python*, there is also a web application written in *Angular* to provide a simulation of the 
algorithm running over the MiBench programs. The source code can be found on 
[CaptainDaVinci/Compiler](https://github.com/CaptainDaVinci/Compiler) and the web app can be viewed at 
[captaindavinci.github.io/Compiler](https://captaindavinci.github.io/Compiler/home)

