## Proof-of-concept _untested_ Gramine support

> :warning: **Note.** Integration with Gramine (v1.3.1) is currently _untested_
> and only provided as an example/starter for people wishing to experiment with
> SGX-Step on Gramine (e.g., see issue #47). Particularly, the patches below
> were validated to successfully compile but were never actually ran(!)
> Furthermore, single-stepping itself is not currently provided for Gramine,
> but should be straightforwardly feasible based on the existing code for the
> Intel SDK. As always, issues/PRs are welcome if you want to contribute
> improvements for a work-in-progress Gramine port.

### Building the patched Gramine

0. First, make sure to build `libsgxstep.a`:

```bash
$ cd ../../libsgxstep
$ make clean all
```

1. Apply the patches in the untrusted Gramine runtime `host_entry.S` to
be able to link to `libsgxstep.a`:

```bash
$ ./patch_entry.sh
```

2. Build the patched Gramine runtime and validate that the patches were
properly applied in the modified Gramine loader:

```bash
$ cd gramine
$ meson setup build/ --buildtype=release -Dsgx=enabled
$ meson configure build/ -Dsgx=enabled
$ ninja -C build
$ objdump -d build/pal/src/host/linux-sgx/loader | grep sgx_set_aep
000000000000924c <sgx_set_aep>:
```

3. Now, you can implement the required attack code in Gramine's untrusted
runtime using `libsgxstep` functionality as usual. For example, the following
patch in `host_ecalls.c` demonstrates some basic usages:

```bash
$ ./patch_ecall.sh
$ cd gramine
$ ninja -C build
$ objdump -d build/pal/src/host/linux-sgx/loader | grep sgx_step
   18631:	48 8d 05 2c 1c 00 00 	lea    0x1c2c(%rip),%rax        # 1a264 <sgx_step_aep_trampoline>
```

4. Install the patched Gramine.

```bash
$ sudo ninja -C build/ install
```
### Running a sample application with the patched Gramine

Proceed as follows:

```bash
$ export PYTHONPATH=/usr/local  # Fix https://github.com/gramineproject/gramine/issues/492
$ gramine-sgx-gen-private-key
$ cd CI-Examples/helloworld
$ make SGX=1
$ gramine-sgx helloworld
```
