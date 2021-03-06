
# SIL Bug Reducer

This directory contains a script called bug_reducer. It is a reimplementation of
llvm's bugpoint using a python script + high level IR utilities. This will
enable bugpoint to easily be ported to newer IRs.

An example invocation would be:

./swift/utils/bug-reducer/bug_reducer.py \
    opt \
    --sdk=$(xcrun --sdk macosx --toolchain default --show-sdk-path) \
    --module-name=${MODULE_NAME} \
    --work-dir=${PWD}/bug_reducer \
    --module-cache=${PWD}/bug_reducer/module-cache \
    ${SWIFT_BUILD_DIR} \
    ${SIB_FILE}

This is still very alpha and needs a lot of work including tests, so please do
not expect it to work at all until tests are available. Once testing is
available, please only use this if you are willing to tolerate bugs (and
hopefully help fix them/add tests).

# Tasks

1. A lot of this code was inspired by llvm's bisect tool. This includes a bit of
   the code style, we should clean this up and pythonify these parts of the
   tool.
2. Once this is done, we should consider implementing function reducing
   functionality. We already have sil-func-extractor which can reduce functions
   in modules just how a bugpoint tool would like. We should wire up code to use
   that tool to reduce functions. After this point, our tool should be good
   enough for reducing test cases. Later we can implement block/instruction
   extraction, but this is a first step.
3. Then we need to implement miscompile detection support. This implies
   implementing support for codegening code from this tool using swiftc. This
   implies splitting modules into optimized and unoptimized parts. Luckily,
   sil-func-extractor can perform this task modulo one task*.

* Specifically, we need to be able to create internal shims that call shared
  functions so that if a shared function is partitioned on the opposite side of
  any of its call sites, we can avoid having to create a shared function
  declaration in the other partition. We avoid this since creating shared
  function declarations is illegal in SIL.
