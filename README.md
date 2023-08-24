# Compilation Avoidance Performance Experiments

This project contains the experimental project used to generate results in the [Gradle's Approach to Faster Compilation](https://blog.gradle.org/gradles-approach-to-faster-compilation) blog post.
It is a synthetic build generated by modifying Gradle's own performance tests that compares Gradle's Java compilation avoidance approach with the Header Jars approach used by Bazel.
It contains 1000 subprojects, each with 10 source files, in order to be representative of a typical Bazel project's structure, yet still easily manageable in an IDE.

There are implementation dependencies between the subprojects, for example, the class `Production0` defined in `project0` is used in every other subproject with a project number evenly divisible by 4.  

For example the `project4` project contains the class `Production41`, with the code:

```java
    private Production0 property0 = new Production0();

    public Production0 getProperty0() {
        return property0;
    }

    public void setProperty0(Production0 value) {
        property0 = value;
    }

    private String property1 = property0.getProperty1();
```

Some classes in `project4` use the `Production0` class directly, while others use it indirectly via (for example) the `Production41` class.

This structure should require at least some recompilation in a quarter of the projects when the modified class's ABI is changed.

## Viewing the Results
Results are provided in the `/results` directory.
These results were obtained using Gradle 8.0, Gradle 8.1, Gradle 8.3, and Bazel 6.3.2. on an Apple M2 Max MacBook Pro with 64 GB running macOS 13.4.1 and the Temurin 17.0.7 JDK.

Comparisons of the results used to build the graphs in the blog posts are provided in the `/comparisons` directory.

## Running the Benchmarks
These instructions explain how to reproduce the results used in this blog post.

> Due to environmental variations your exact timings may differ. 
> The overall analysis should still apply.

1. Download this repository.
1. Install Gradle Profiler using the instructions [here](https://github.com/gradle/gradle-profiler).
1. Install Bazel using the instructions [here](https://bazel.build/install).
1. Install Python3 using the instructions [here](https://www.python.org/downloads/) (the script to extract and compare the results requires Python).
1. Execute the `run.performance.experiments.sh` script from the project root directory.
1. Results will be written to the `/results` directory and comparisons to the `/comparisons` directory (existing data will be deleted).
The html graphs in can be viewed directly, or the CSV files can be used to generate your own graphs and comparisons.
1. (Optionally) To assemble a CSV file comparing multiple results in a format suitable for easy creation of charts such as those included in the blog post, provide an output file and a list of result CSV files to the `fix-bench-csvs.py` python script.
   - For example:`python3 fix-bench-csvs.py comparisons/bazel-and-gradle-8.1.csv results/abiChange/bazel/benchmark.csv results/abiChange/gradle-8.1/benchmark.csv results/nonAbiChange/bazel/benchmark.csv results/nonAbiChange/gradle-8.1/benchmark.csv
     ` run from the project root directory will create a file named `bazel-and-gradle-8.1.csv` in the `comparisons` directory comparing Bazel with Gradle 8.1.

## Other Variations

- By changing the argument after `--gradle-version` in `run.performance.experiments.sh` you can compare the performance of different versions of Gradle.  
  - See the [Gradle Profiler project](https://github.com/gradle/gradle-profiler) for explanation of additional arguments. 
- Comment or uncomment lines in the `gradle.properties` file to run variations of the experiment using or disabling different Gradle features.
