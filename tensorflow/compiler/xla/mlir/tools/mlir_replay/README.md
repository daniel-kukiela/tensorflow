# MLIR Replay tool

This tool is mainly intended for helping debug miscompiles. It takes as inputs
an HLO snapshot proto with input tensors and a compiler trace proto with the
state of the IR after each pass.

This tool is built on top of
[mlir-interpreter](https://github.com/tensorflow/mlir-hlo/tree/master/tools/mlir_interpreter/).

Example usage:

```
# Run a JAX test with debug flags enabled:
$ bazel test :some_jax_test --compilation_mode=opt \
  --test_env=XLA_FLAGS="--xla_cpu_use_xla_runtime --xla_dump_to=/tmp/test-dump --xla_dump_hlo_snapshots" \
  --test_filter=SomeSpecific.TestCase \
  --test_sharding_strategy=disabled

# Run the IR after each pass. Note that JAX typically compiles many modules, so
# you may have check more than one.
# There is one .mlir-trace.pb file per module (containing the intermediate IR)
# and one .snapshot.pb file per execution (containing the inputs and outputs).
$ ./mlir_replay \
  --mlir_compilation_trace=/tmp/test-dump/module_1234.jit_something.mlir-trace.pb \
  --hlo_snapshot=/tmp/test-dump/module_1234.jit_something.snapshot.56.pb \
  --print_changes_only
Running IR after APass
Results: [1, 2, 3]

Running IR after BPass
Running IR after CPass
Running IR after BrokenPass
Results: [1, 1, 1]

Running IR after DPass
```

If you don't have an HLO snapshot with actual inputs, random values will be
used.
