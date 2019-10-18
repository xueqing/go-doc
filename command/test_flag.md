# 测试标识

`go test` 命令使用只适用于 `go test` 的标识以及适用于生成二进制测试的标识。

一些标识控制概要并且写适用于 `go tool pprof` 的执行概要；运行 `go tool pprof -h` 查看更多信息。pprof 的 --alloc_space、--alloc_objects 和 --show_bytes 选项控制如何显示这些信息。

下面的标识被 `go test` 命令识别，并且控制测试的执行：

```txt
-bench regexp
    Run only those benchmarks matching a regular expression.
    By default, no benchmarks are run.
    To run all benchmarks, use '-bench .' or '-bench=.'.
    The regular expression is split by unbracketed slash (/)
    characters into a sequence of regular expressions, and each
    part of a benchmark's identifier must match the corresponding
    element in the sequence, if any. Possible parents of matches
    are run with b.N=1 to identify sub-benchmarks. For example,
    given -bench=X/Y, top-level benchmarks matching X are run
    with b.N=1 to find any sub-benchmarks matching Y, which are
    then run in full.

-benchtime t
    Run enough iterations of each benchmark to take t, specified
    as a time.Duration (for example, -benchtime 1h30s).
    The default is 1 second (1s).
    The special syntax Nx means to run the benchmark N times
    (for example, -benchtime 100x).

-count n
    Run each test and benchmark n times (default 1).
    If -cpu is set, run n times for each GOMAXPROCS value.
    Examples are always run once.

-cover
    Enable coverage analysis.
    Note that because coverage works by annotating the source
    code before compilation, compilation and test failures with
    coverage enabled may report line numbers that don't correspond
    to the original sources.

-covermode set,count,atomic
    Set the mode for coverage analysis for the package[s]
    being tested. The default is "set" unless -race is enabled,
    in which case it is "atomic".
    The values:
  set: bool: does this statement run?
  count: int: how many times does this statement run?
  atomic: int: count, but correct in multithreaded tests;
    significantly more expensive.
    Sets -cover.

-coverpkg pattern1,pattern2,pattern3
    Apply coverage analysis in each test to packages matching the patterns.
    The default is for each test to analyze only the package being tested.
    See 'go help packages' for a description of package patterns.
    Sets -cover.

-cpu 1,2,4
    Specify a list of GOMAXPROCS values for which the tests or
    benchmarks should be executed. The default is the current value
    of GOMAXPROCS.

-failfast
    Do not start new tests after the first test failure.

-list regexp
    List tests, benchmarks, or examples matching the regular expression.
    No tests, benchmarks or examples will be run. This will only
    list top-level tests. No subtest or subbenchmarks will be shown.

-parallel n
    Allow parallel execution of test functions that call t.Parallel.
    The value of this flag is the maximum number of tests to run
    simultaneously; by default, it is set to the value of GOMAXPROCS.
    Note that -parallel only applies within a single test binary.
    The 'go test' command may run tests for different packages
    in parallel as well, according to the setting of the -p flag
    (see 'go help build').

-run regexp
    Run only those tests and examples matching the regular expression.
    For tests, the regular expression is split by unbracketed slash (/)
    characters into a sequence of regular expressions, and each part
    of a test's identifier must match the corresponding element in
    the sequence, if any. Note that possible parents of matches are
    run too, so that -run=X/Y matches and runs and reports the result
    of all tests matching X, even those without sub-tests matching Y,
    because it must run them to look for those sub-tests.

-short
    Tell long-running tests to shorten their run time.
    It is off by default but set during all.bash so that installing
    the Go tree can run a sanity check but not spend time running
    exhaustive tests.

-timeout d
    If a test binary runs longer than duration d, panic.
    If d is 0, the timeout is disabled.
    The default is 10 minutes (10m).

-v
    Verbose output: log all tests as they are run. Also print all
    text from Log and Logf calls even if the test succeeds.

-vet list
    Configure the invocation of "go vet" during "go test"
    to use the comma-separated list of vet checks.
    If list is empty, "go test" runs "go vet" with a curated list of
    checks believed to be always worth addressing.
    If list is "off", "go test" does not run "go vet" at all.
```

下面的标识被 `go test` 命令识别，且可用于概述执行期间的测试：

```txt
-benchmem
    Print memory allocation statistics for benchmarks.

-blockprofile block.out
    Write a goroutine blocking profile to the specified file
    when all tests are complete.
    Writes test binary as -c would.

-blockprofilerate n
    Control the detail provided in goroutine blocking profiles by
    calling runtime.SetBlockProfileRate with n.
    See 'go doc runtime.SetBlockProfileRate'.
    The profiler aims to sample, on average, one blocking event every
    n nanoseconds the program spends blocked. By default,
    if -test.blockprofile is set without this flag, all blocking events
    are recorded, equivalent to -test.blockprofilerate=1.

-coverprofile cover.out
    Write a coverage profile to the file after all tests have passed.
    Sets -cover.

-cpuprofile cpu.out
    Write a CPU profile to the specified file before exiting.
    Writes test binary as -c would.

-memprofile mem.out
    Write an allocation profile to the file after all tests have passed.
    Writes test binary as -c would.

-memprofilerate n
    Enable more precise (and expensive) memory allocation profiles by
    setting runtime.MemProfileRate. See 'go doc runtime.MemProfileRate'.
    To profile all memory allocations, use -test.memprofilerate=1.

-mutexprofile mutex.out
    Write a mutex contention profile to the specified file
    when all tests are complete.
    Writes test binary as -c would.

-mutexprofilefraction n
    Sample 1 in n stack traces of goroutines holding a
    contended mutex.

-outputdir directory
    Place output files from profiling in the specified directory,
    by default the directory in which "go test" is running.

-trace trace.out
    Write an execution trace to the specified file before exiting.
```

所有这些标识也有一个可选的 “test.” 前缀被识别(如 -test.v)。但是当直接调用生成的二进制测试时(`go test -c` 生成)，这个前缀是强制的。

`go test` 命令在调用二进制测试之前，适当地重写或移除在可选的包列表之前或之后识别的标识。

比如，命令 `go test -v -myflag testdata -cpuprofile=prof.out -x` 将会编译二进制测试然后运行 `pkg.test -test.v -myflag testdata -test.cpuprofile=prof.out`。(-x 标识被移除，因为它只适用于go 命令的执行，而不是`go test`)

生成概述(除了用于覆盖)的测试标识也会将二进制测试留在 pkg.test 以便用于分析概述。

当 `go test` 运行一个二进制测试时，它从对应包源码目录内部运行。视测试而定，可能需要在直接调用一个生成的二进制测试时也这样做。

命令行的包列表，如果有的话，必须出现在所有 `go test` 命令不知道的标识之前。继续上面的例子，包列表需要出现在 -myflag 之前，但是可以出现在 -v 两侧。

当 `go test` 在列表模式运行时，`go test` 缓存成功的包测试结果以避免不必要的重复运行测试。要禁用测试缓存，使用除了可缓存的标识以外的任意的测试标识或参数。惯用的显式禁用测试缓存的方法是使用 -count=1。

要保持二进制测试的一个参数不被翻译成一个已知的标识或者包名，使用 -args (查看 `got help test`) 换地命令行的剩余部分给二进制测试，该部分不会被解释或修改。

例如，命令 `go test -v -args -x -v` 会编译二进制测试然后运行 `pkg.test -test.v -x -v`。类似的，`go test -args math` 会编译二进制测试然后运行 `pkg.test math`。

在第一个例子中，-x 和第二个 -v 被传递给二进制测试且未被修改，且对 go 命令本身没有影响。在第二个例子中，参数 math 被传递给二进制测试，而不是解释成包列表。
