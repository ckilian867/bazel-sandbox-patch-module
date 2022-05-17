# bazel-sandbox-patch-module

`Problem.java` is created with package `sun.misc`, which is a package that exists in `jdk.unsupported`. The goal is to patch `jdk.unsupported` to include `Problem.java`
In this example, we attempt to do that by including `"--patch-module=jdk.unsupported=example"` in the `javacopts`.

When running a build with `--patch-module` in the `javacopts`, the build succeeds when the Javac strategy is local but fails when sandboxed.

Strategy = local:
```
$ bazel build --strategy=Javac=local //example:problem --verbose_failures --sandbox_debug
INFO: Invocation ID: ede24eed-5feb-4631-b343-5d19115ee591
INFO: Analyzed target //example:problem (0 packages loaded, 0 targets configured).
INFO: Found 1 target...
Target //example:problem up-to-date:
  bazel-bin/example/libproblem.jar
INFO: Elapsed time: 2.166s, Critical Path: 1.92s
INFO: 2 processes: 1 internal, 1 local.
INFO: Build completed successfully, 2 total actions
```

Strategy = sandboxed:
```
$ bazel build --strategy=Javac=sandboxed //example:problem --verbose_failures --sandbox_debug
INFO: Invocation ID: 9e741bcf-6986-47fe-b2d2-0feabea28334
INFO: Analyzed target //example:problem (46 packages loaded, 747 targets configured).
INFO: Found 1 target...
ERROR: /home/user/bazel-sandbox-patch-module/example/BUILD.bazel:1:13: Building example/libproblem.jar (1 source file) failed: (Exit 1): process-wrapper failed: error executing command
  (cd /home/user/.cache/bazel/_bazel_cjk/1c9ce485b9ebe93d6b96a1f759641852/sandbox/processwrapper-sandbox/16/execroot/__main__ && \
  exec env - \
    LC_CTYPE=en_US.UTF-8 \
    PATH=/opt/nodejs/.nvm/versions/node/v16.13.2/bin:/opt/jvm/jdk-11/bin:/opt/python/3.9.2/bin:/opt/lda/pex_resources/scripts/binaries:/home/user/bin:/opt/git/bin:/home/user/go-repos:/opt/go/path/bin:/opt/go/root/bin:/usr/lib/jvm/adoptopenjdk-8-hotspot-amd64/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/opt/nodejs/node_modules/bin:/opt/nodejs/node_modules/bin \
    TMPDIR=/tmp \
  /home/user/.cache/bazel/_bazel_cjk/install/79570a41fb8272e4808f43403af3b38c/process-wrapper '--timeout=0' '--kill_delay=15' external/remotejdk11_linux/bin/java -XX:-CompactStrings '--add-exports=jdk.compiler/com.sun.tools.javac.api=ALL-UNNAMED' '--add-exports=jdk.compiler/com.sun.tools.javac.main=ALL-UNNAMED' '--add-exports=jdk.compiler/com.sun.tools.javac.model=ALL-UNNAMED' '--add-exports=jdk.compiler/com.sun.tools.javac.parser=ALL-UNNAMED' '--add-exports=jdk.compiler/com.sun.tools.javac.processing=ALL-UNNAMED' '--add-exports=jdk.compiler/com.sun.tools.javac.resources=ALL-UNNAMED' '--add-exports=jdk.compiler/com.sun.tools.javac.tree=ALL-UNNAMED' '--add-exports=jdk.compiler/com.sun.tools.javac.util=ALL-UNNAMED' '--add-opens=jdk.compiler/com.sun.tools.javac.code=ALL-UNNAMED' '--add-opens=jdk.compiler/com.sun.tools.javac.comp=ALL-UNNAMED' '--add-opens=jdk.compiler/com.sun.tools.javac.file=ALL-UNNAMED' '--add-opens=java.base/java.nio=ALL-UNNAMED' '--add-opens=java.base/java.lang=ALL-UNNAMED' '--patch-module=java.compiler=external/remote_java_tools/java_tools/java_compiler.jar' '--patch-module=jdk.compiler=external/remote_java_tools/java_tools/jdk_compiler.jar' -jar external/remote_java_tools/java_tools/JavaBuilder_deploy.jar @bazel-out/k8-fastbuild/bin/example/libproblem.jar-0.params @bazel-out/k8-fastbuild/bin/example/libproblem.jar-1.params)
example/Problem.java:1: error: package exists in another module: jdk.unsupported
package sun.misc;
^
Target //example:problem failed to build
INFO: Elapsed time: 5.737s, Critical Path: 4.99s
INFO: 4 processes: 2 internal, 2 processwrapper-sandbox.
FAILED: Build did NOT complete successfully
```

When entering the sandbox and running `ls`, the `example` directory is present and contains the file I want to patch in:

```
$ cd /home/user/.cache/bazel/_bazel_cjk/1c9ce485b9ebe93d6b96a1f759641852/sandbox/processwrapper-sandbox/16/execroot/__main__ && ls example
Problem.java
$
```
