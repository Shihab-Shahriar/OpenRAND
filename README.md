# OpenRAND: Reproducible Random Number For Parallel Computations
[![Build Status](https://github.com/shihab-shahriar/rnd/actions/workflows/run-tests.yml/badge.svg)](https://github.com/shihab-shahriar/rnd/actions)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

OpenRAND is a C++ library designed to foster reproducible scientific research by providing a robust and replicable random number generation solution. It is a simple header only library that is performance portable, statistically robust, and easy to integrate into any HPC computing project.

## Features
**Cross-Platform Support**: OpenRAND is designed to work seamlessly across various platforms, including CPUs and GPUs. Its header-only library design allows it to be easily integrated into your project.

**User-Friendly API**: OpenRAND provides a user-friendly API, making it straightforward to generate random numbers in your applications.

**Statistical Robustness**: Built-in tests ensure that random number streams generated by OpenRAND are statistically robust, with no discernible patterns.

**Performance**: It is as fast and often faster than native libraries like libstdc++ or Nvidia's Curand, with minimal memory overhead.


## Installation
OpenRAND is header only, so there is no need to install it. Simply copy the header files in `include/` directory into your project and you're good to go!

You can also install OpenRAND using CMake. To integrate OpenRAND into your CMake project, add the following lines to your CMakeLists.txt file:

```
include(FetchContent)
FetchContent_Declare(
  crng
  GIT_REPOSITORY https://github.com/msu-sparta/OpenRAND.git
  GIT_TAG        main
)

FetchContent_MakeAvailable(crng)
```


## Usage
Here's a simple example of how to generate random numbers using OpenRAND:

```
#include <phillox.h>

int main() {
    using RNG = Phillox; // Tyche
    
    // Initialize RNG with seed and counter
    RNG rng(1, 0);

    // Draw uniform random numbers of many types
    int a = rng.rand<int>();            // range [0,2^32) 
    auto b = rng.rand<long long int>(); // range [0,2^64)
    double c = rng.rand<double>();      // range [0,1)
    float f = rng.rand<float>();        // range [0,1)

    // use std distribution functions with RNG
    std::lognormal_distribution<double> dist(0.0, 1.0);
    double x = dist(rng);
    ...
}
```

The seed should correspond to a work-unit of the program. For example, it could be the unique global id of a particle in a monte carlo simulation, or the (1D) pixel index in a ray tracing renderer. The counter should be incremented every time a new random number is drawn for a particular seed. This is helpful, for example, when a particle undergoes multiple kernel launches in it's lifespan (with a new random stream required in each).

Below is a simplified monte carlo paticle simulation example that runs for 10000 time steps. Conveniently, we can simply use the iteration number as the `counter` for all particles. For `seed`, we assume each particle below has a unique global id atribute called `pid`. 

```
__global__ void apply_forces(Particle *particles, int counter){
    int i = blockIdx.x * blockDim.x + threadIdx.x;
    Particle p = particles[i];
    ...

    // Apply random force
    RNG local_rand_state(p.pid, counter);
    
    p.vx += (local_rand_state.rand<double>()  * 2.0 - 1.0);
    ...
}


int main(){
    ...

    // Simulation loop
    int iter = 0;
    while (iter++ < 10000) {
        apply_forces<<<nblocks, nthreads>>>(particles, iter);
        ...
    }
}
```

## Documentation
For a quick intorduction to OpenRAND: please refer to [this guide](https://msu-sparta.github.io/OpenRAND/md_quickstart.html).

For more detailed information, refer to the doxygen doc [here](https://msu-sparta.github.io/OpenRAND/), 

## Contributing
We welcome all sorts of contributions from the community- code, bug reports, documentation improvements, and any general feedback is always appreciated. 

If you want to contribute code, please note that we follow [Github Workflow](https://docs.github.com/en/get-started/quickstart/github-flow) for development. In short, fork this repo; create a new branch off of `main` in your fork; make changes, commit, push your changes and then open a Pull Request. 

If you'd like to make a major change or introduce a new feature, please open an issue first to discuss it with us.

## License
This project is licensed under the MIT License - see the LICENSE file for details.

