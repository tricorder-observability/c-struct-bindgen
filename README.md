# struct-bindgen

Generate C struct bindings between Wasm modules and host env/eBPF programs

## Usage - From pre-compiled bpf object with BTF info

```bash
struct-bindgen examples/source.bpf.o
```

You will get a `source-struct-binding.h` file, for correct access to the C struct memory in the bpf programs or host env:

```c
// Generated by ecc -
#ifndef __STRUCT_UNMARSHAL_EVENT_H__
#define __STRUCT_UNMARSHAL_EVENT_H__

#include <string.h>

static void unmarshal_event__from_binary(struct event *dst, const void *src) {
    assert(dst && src);
    dst->pid = *(unsigned int*)(src + 0);
    dst->tpid = *(unsigned int*)(src + 4);
    dst->sig = *(int*)(src + 8);
    dst->ret = *(int*)(src + 12);
    memcpy(dst->comm, src + 16, 16);
}

static void marshal_struct_event__to_binary(void *dst, const struct event *src) {
    assert(dst && src);
    *(unsigned int*)(dst + 0) = src->pid;
    *(unsigned int*)(dst + 4) = src->tpid;
    *(int*)(dst + 8) = src->sig;
    *(int*)(dst + 12) = src->ret;
    memcpy(dst + 16, src->comm, 16);
}

#endif /* __STRUCT_UNMARSHAL_EVENT_H__ */
```

It can be used in the wasm module communication with the bpf program.

## Usage - Compile BTF info from source with ecc

See `examples/` for [examples](examples).

1. Create a C struct header and a include file, for example:

    examples/test-event.h:

    ```c
    #ifndef __SIGSNOOP_H
    #define __SIGSNOOP_H

    #define TASK_COMM_LEN 16

    struct event {
        unsigned int pid;
        double x;
        int sig;
        float y;
        char comm[TASK_COMM_LEN];
    };

    struct event2 {
        struct event e2;
        double g;
    };

    #endif /* __SIGSNOOP_H */
    ```

    Note: the struct name can be any name, and multiple structs in a header are also supported. The type of struct fields can be any C type, and the struct can be nested.

    source.c:

    ```c
    #include "test-event.h"

    // ...
    ```

    Compile with ecc (see <https://github.com/eunomia-bpf/eunomia-bpf>):

    ```bash
    ecc examples/source.c examples/test-event.h
    ```

    You will get a `source.bpf.o` file.

2. Generate bindings:

    ```bash
    struct-bindgen examples/source.bpf.o
    ```

    You will get a `source-struct-binding.h` file.

## TODO

- [ ] Support for union in structs fields
- [ ] Support for composite types in array fields
- [ ] handle byte order in host env
- [ ] Support for print out info in JSON format
