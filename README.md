# ROC-profiler

ROC profiler library. Profiling with perf-counters and derived metrics. Library supports GFX8/GFX9.

HW specific low-level performance analysis interface for profiling of GPU compute applications. The profiling includes HW performance counters with complex performance metrics and HW traces

The library source tree:
 - bin
   - rpl_run.sh - Profiling tool run script
 - doc - Documentation
 - inc/rocprofiler.h - Library public API
 - src  - Library sources
   - core - Library API sources
   - util - Library utils sources
   - xml - XML parser
 - test - Library test suite
   - tool - Profiling tool
     - tool.cpp - tool sources
     - metrics.xml - metrics config file
   - ctrl - Test controll
   - util - Test utils
   - simple_convolution - Simple convolution test kernel

## Build environment:
```
  export CMAKE_PREFIX_PATH=<path to hsa-runtime includes>:<path to hsa-runtime library>
  export CMAKE_BUILD_TYPE=<debug|release> # release by default
  export CMAKE_DEBUG_TRACE=1 # to enable debug tracing
```

## To build with the current installed ROCM:
```
 - To build and install to /opt/rocm/rocprofiler
  cd .../rocprofiler
  mkdir build
  cd build
  export CMAKE_PREFIX_PATH=/opt/rocm/lib:/opt/rocm/include/hsa
  cmake -DCMAKE_INSTALL_PREFIX=/opt/rocm ..
  make
  make install
  
  For ROCM under 1.9 need:
  export CMAKE_CURR_API=1
```

## To run the test:
```
  cd .../rocprofiler/build
  export LD_LIBRARY_PATH=.:<other paths> # paths to ROC profiler and oher libraries
  export HSA_TOOLS_LIB=librocprofiler64.so # ROC profiler library loaded by HSA runtime
  export ROCP_TOOL_LIB=test/libtool.so # tool library loaded by ROC profiler
  export ROCP_METRICS=metrics.xml # ROC profiler metrics config file
  export ROCP_INPUT=input.xml # input file for the tool library
  export ROCP_OUTPUT_DIR=./ # output directory for the tool library, for metrics results file 'results.txt'
  <your test>
```

## Internal 'simple_convolution' test run script:
```
  cd .../rocprofiler/build
  run.sh
```

## To enable error messages logging to '/tmp/rocprofiler_log.txt':
```
  export ROCPROFILER_LOG=1
```

## To enable verbose tracing:
```
  export ROCPROFILER_TRACE=1
```

## Profiling utility usage:
```
  rpl_run.sh [-h] [--list-basic] [--list-derived] [-i <input .txt/.xml file>] [-o <output CSV file>] <app command line>

Options:
  -h - this help
  --verbose - verbose mode, dumping all base counters used in the input metrics
  --list-basic - to print the list of basic HW counters
  --list-derived - to print the list of derived metrics with formulas

  -i <.txt|.xml file> - input file
      Input file .txt format, automatically rerun application for every pmc/sqtt line:

        # Perf counters group 1
        pmc : Wavefronts VALUInsts SALUInsts SFetchInsts FlatVMemInsts LDSInsts FlatLDSInsts GDSInsts VALUUtilization FetchSize
        # Perf counters group 2
        pmc : WriteSize L2CacheHit
        # Filter by dispatches range, GPU index and kernel names
        # supported range formats: "3:9", "3:", "3"
        range: 1 : 4
        gpu: 0 1 2 3
        kernel: simple Pass1 simpleConvolutionPass2

      Input file .xml format, for single profiling run:

        # Metrics list definition, also the form "<block-name>:<event-id>" can be used
        # All defined metrics can be found in the 'metrics.xml'
        # There are basic metrics for raw HW counters and high-level metrics for derived counters
        <metric name=SQ:4,SQ_WAVES,VFetchInsts
        ></metric>

        # Filter by dispatches range, GPU index and kernel names
        <metric
          # range formats: "3:9", "3:", "3"
          range=""
          # list of gpu indexes "0,1,2,3"
          gpu_index=""
          # list of matched sub-strings "Simple1,Conv1,SimpleConvolution"
          kernel=""
        ></metric>

  -o <output file> - output CSV file [<input file base>.csv]
  -d <data directory> - directory where profiler store profiling data including thread treaces [/tmp]
      The data directory is renoving autonatically if the directory is matching the temporary one, which is the default.
  -t <temporary directory> - to change the temporary directory [/tmp]
      By changing the temporary directory you can prevent removing the profiling data from /tmp or enable removing from not '/tmp' directory.

  --basenames <on|off> - to turn on/off truncating of the kernel full function names till the base ones [off]
  --timestamp <on|off> - to turn on/off the kernel disoatches timestamps, dispatch/begin/end/complete [off]
  --ctx-limit <max number> - maximum number of outstanding contexts [0 - unlimited]
  --heartbeat <rate sec> - to print progress heartbeats [0 - disabled]
  --sqtt-size <byte size> - to set SQTT buffer size, aggregate for all SE [0x2000000]
      Can be set in KB (1024B) or MB (1048576) units, examples 20K or 20M respectively.
  --sqtt-local <on|off> - to allocate SQTT buffer in local GPU memory [on]

Configuration file:
  You can set your parameters defaults preferences in the configuration file 'rpl_rc.xml'. The search path sequence: .:/home/evgeny:<package path>
  First the configuration file is looking in the current directory, then in your home, and then in the package directory.
  Configurable options: 'basenames', 'timestamp', 'ctx-limit', 'heartbeat', 'sqtt-size', 'sqtt-local'.
  An example of 'rpl_rc.xml':
    <defaults
      basenames=off
      timestamp=off
      ctx-limit=0
      heartbeat=0
      sqtt-size=0x20M
      sqtt-local=on
    ></defaults>
```
