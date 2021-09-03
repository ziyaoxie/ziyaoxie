# libaio

User space library implementation and use system calls internally.

## Install

```shell
$ sudo apt search libaio
Sorting... Done
Full Text Search... Done
libaio-dev/focal,now 0.3.112-5 amd64 [installed]
  Linux kernel AIO access library - development files

libaio1/focal,now 0.3.112-5 amd64 [installed,automatic]
  Linux kernel AIO access library - shared library

$ sudo apt install libaio
libaio1     libaio-dev
$ sudo apt install libaio-dev
```

## Syscall Wrappers

```c
/* /usr/include/libaio.h */
/* Actual syscalls */
extern int io_setup(int maxevents, io_context_t *ctxp);
extern int io_destroy(io_context_t ctx);
extern int io_submit(io_context_t ctx, long nr, struct iocb *ios[]);
extern int io_cancel(io_context_t ctx, struct iocb *iocb, struct io_event *evt);
extern int io_getevents(io_context_t ctx_id, long min_nr, long nr, struct io_event *events, struct timespec *timeout);
```

## Helper Functions

```c
static inline void io_prep_pread(struct iocb *iocb, int fd, void *buf, size_t count, long long offset)
static inline void io_prep_pwrite(struct iocb *iocb, int fd, void *buf, size_t count, long long offset)
static inline void io_prep_preadv(struct iocb *iocb, int fd, const struct iovec *iov, int iovcnt, long long offset)
static inline void io_prep_pwritev(struct iocb *iocb, int fd, const struct iovec *iov, int iovcnt, long long offset)

static inline void io_prep_poll(struct iocb *iocb, int fd, int events)
static inline void io_prep_fsync(struct iocb *iocb, int fd)
static inline void io_prep_fdsync(struct iocb *iocb, int fd)

static inline int io_poll(io_context_t ctx, struct iocb *iocb, io_callback_t cb, int fd, int events)
static inline int io_fsync(io_context_t ctx, struct iocb *iocb, io_callback_t cb, int fd)
static inline int io_fdsync(io_context_t ctx, struct iocb *iocb, io_callback_t cb, int fd)

static inline void io_set_eventfd(struct iocb *iocb, int eventfd);
```

## struct iocb

```c
struct io_iocb_poll {
        PADDED(int events, __pad1);
};      /* result code is the set of result flags or -'ve errno */

struct io_iocb_sockaddr {
        struct sockaddr *addr;
        int             len;
};      /* result code is the length of the sockaddr, or -'ve errno */

struct io_iocb_common {
        PADDEDptr(void  *buf, __pad1);
        PADDEDul(nbytes, __pad2);
        long long       offset;
        long long       __pad3;
        unsigned        flags;
        unsigned        resfd;
};      /* result code is the amount read or -'ve errno */

struct io_iocb_vector {
        const struct iovec      *vec;
        int                     nr;
        long long               offset;
};      /* result code is the amount read or -'ve errno */

struct iocb {
        PADDEDptr(void *data, __pad1);  /* Return in the io completion event */
        PADDED(unsigned key, __pad2);   /* For use in identifying io requests */

        short           aio_lio_opcode;
        short           aio_reqprio;
        int             aio_fildes;

        union {
                struct io_iocb_common           c;
                struct io_iocb_vector           v;
                struct io_iocb_poll             poll;
                struct io_iocb_sockaddr saddr;
        } u;
};

struct io_event {
        PADDEDptr(void *data, __pad1);
        PADDEDptr(struct iocb *obj,  __pad2);
        PADDEDul(res,  __pad3);
        PADDEDul(res2, __pad4);
};
```

## Example

```c
// compile: cc libaio.c -o libaio -laio

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <err.h>
#include <errno.h>

#include <unistd.h>
#include <fcntl.h>
#include <libaio.h>

int main() {
	io_context_t ctx;
	struct iocb iocb;
	struct iocb * iocbs[1];
	struct io_event events[1];
	struct timespec timeout;
	int fd;

	fd = open("/tmp/test", O_WRONLY | O_CREAT);
	if (fd < 0) err(1, "open");

	memset(&ctx, 0, sizeof(ctx));
	if (io_setup(10, &ctx) != 0) err(1, "io_setup");

	const char *msg = "hello";
	io_prep_pwrite(&iocb, fd, (void *)msg, strlen(msg), 0);
	iocb.data = (void *)msg;

	iocbs[0] = &iocb;

	if (io_submit(ctx, 1, iocbs) != 1) {
		io_destroy(ctx);
		err(1, "io_submit");
	}

	while (1) {
		timeout.tv_sec = 0;
		timeout.tv_nsec = 500000000;
		if (io_getevents(ctx, 0, 1, events, &timeout) == 1) {
			close(fd);
			break;
		}
		printf("not done yet\n");
		sleep(1);
	}
	io_destroy(ctx);

	return 0;
}
```

