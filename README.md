Apollo+LLVM Notes
===

## Tips

- ***!!! Do not use `dev_start.sh` to start dev docker after rebooting the machine***

    For the first time of starting Apollo dev docker, you can use `bash docker/scripts/dev_start.sh`. However, after rebooting, if you still use `dev_start.sh`, you will lose all previous modifications within the docker, because the dev docker container will be deleted. Using the following commands can avoid such issue.

    **Apollo 3.0**

    ```bash
    docker run -it -d --rm --name apollo_localization_volume apolloauto/apollo:localization_volume-x86_64-latest
    docker run -it -d --rm --name apollo_yolo3d_volume apolloauto/apollo:yolo3d_volume-x86_64-latest
    docker run -it -d --rm --name apollo_map_volume-sunnyvale_big_loop apolloauto/apollo:map_volume-sunnyvale_big_loop-latest
    docker run -it -d --rm --name apollo_map_volume-sunnyvale_loop apolloauto/apollo:map_volume-sunnyvale_loop-latest

    # Make sure that you can see `apollo_dev` in `docker ps -a` as `EXITED`
    docker start apollo_dev

    bash docker/scripts/dev_into.sh
    ```

- May need to delete inconsistent `gtest` header file
    ```bash
    sudo mv /usr/include/gtest /usr/include/gtest_bak
    ```

## Compile Apollo using LLVM

(All following commands are assumed to be executed in Apollo dev docker)

1. Install LLVM

    **LLVM 8 (Recommended)**

    ```bash
    # LLVM 8
    sudo apt install llvm-8 llvm-8-dev clang-8 libclang-8-dev

    # Create soft links
    sudo ln -sf /usr/bin/llvm-config-8 /usr/bin/llvm-config
    sudo ln -sf /usr/bin/llvm-link-8 /usr/bin/llvm-link
    ```

2. Install `whole-program-llvm`

    ```bash
    sudo pip install wllvm
    ```

3. Copy `wllvm` directory to `/apollo/tools`

    ```bash
    cp -r wllvm /apollo/tools
    ```

4. Compile Apollo

    ```bash
    cd /apollo
    mkdir wllvm_bc  # This directory will contain all seperate bitcode files
    bash apollo.sh build --copt=-mavx2 --cxxopt=-mavx2 --copt=-mno-sse3 --crosstool_top=tools/wllvm:toolchain
    ```

    Those `copt` and `cxxopt` can be removed, if your machine supports the corresponding instruction sets.

5. Or compile single module (e.g., `planning` module)

    ```bash
    ## Apollo 3.0
    bazel build --define ARCH=x86_64 --define CAN_CARD=fake_can --cxxopt=-DUSE_ESD_CAN=false --copt=-mavx2 --copt=-mno-sse3 --cxxopt=-DCPU_ONLY --crosstool_top=tools/wllvm:toolchain //modules/planning:planning --compilation_mode=opt
    ```

6. Extract bitcode file (e.g., `planning` module)

    **Apollo 3.0**

    ```bash
    cd /apollo/bazel-bin/modules/planning
    extract-bc planning

    # Check output
    file planning.bc
    llvm-dis planning.bc
    ```
