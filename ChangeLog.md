# ChangeLog

## 0.2.0

  In this release, the code related to `memory region` has been reorganized.
  This change makes the abstraction more explicit and easy to maintain later.

  Some new APIs for RDMA operations with immediate data was added. And we also
  fixed some bugs, which made the lib more stable.

### New features

* Implement memory region slice. `LocalMrSlice` and `RemoteMrSlice` enable us to operate on
  part of memory region. And we can only get one mutable slice or many unmutable slices at the
  same time, just like any other type in rust.
* Add imm data APIs.
  * send_with_imm
  * receive_with_imm
  * write_with_imm
  * receive_write_imm

### Optimizations and refactors

* Redesign memory region. Use traits to describe three kinds of memory region abstraction.
  `RawMemoryRegion` records all information about the memory region Registered locally.
  `LocalMr` records information for local use. `RemoteMr` records information for remote use.
* Discard preapplication of memroy region strategy. The old preapplication strategy is not
  flexible enough, so that is currently abandoned. And we will transform jemalloc to manage
  memory region and reuse it's preapplication strategy in the next release.
* Change from dynamic generic to static generic.
* Implement multi-task receiver. Just one task work with one `ibv_recv_wr` can not
  handle high-concurrency SEND requests. So more receiver tasks with more `ibv_recv_wr` were
  spawned to handle highly concurrent requests in this release.
* Redesign memory region transfer APIs.
  * Refine the interface capability
    * Change from `send_mr` to `send_local_mr` and `send_remote_mr`.
    * Change from `receive_mr` to `receive_local_mr` and `receive_remote_mr`.
  * Take the ownership of the sent memory region. Prevent both ends from operating on the
    same memory region at the same time.

### Bug fixes

* Fix retry bug. When `rnr_retry`==7, the sender will keep retrying until the system freezes.
  We can't get any effective information to debug it because this part of the work is performed
  by the kernel module and there is no error message. But that will deplete CPU resources and the
  only thing we can observe is the system freezes. Solution is to make `rnr_retry` < 7.
* Add timeout mechanism for remote requet operations to prevent Infinite wait.
* Make `Agent` hold `AgentThread` to prevent the senders' release.

## 0.1.0

### What is it and why make it?

[**`Async-rdma`**](https://github.com/datenlord/async-rdma) is a framework for
writing RDMA applications with high-level abstraction and asynchronous APIs.

Remote Direct Memory Access(RDMA) is direct access of memory from memory of one machine to the
memory of another. It helps in boosting the performance of applications that need low latency
and high throughput as it supports **kernel bypass** and **zero copy** while **not involving CPU**.

However, writing RDMA applications with low-level c library is laborious and error-prone. We want
easy-to-use APIs that hide the complexity of the underlying RDMA operations, so we developed
`async-rdma`. With the help of `async-rdma`, most RDMA operations can be completed by writing
only **one line** of code.

### What does it provide?

* Tools for establishing connections with rdma endpoints.

* High-level async APIs for data transmission between endpoints.

* High-level APIs for rdma memory region management.

* A framework working behind APIs for memory region management and executing rdma requests asynchronously.

### Note

We develop `async-rdma` with the aim of becoming production-grade. But now it is too young, still in the
experimental stage. We are adding more features and improving stability. We welcome everyone to try, ask
questions and make suggestions.

### Links

* Github: <https://github.com/datenlord/async-rdma>

* Crate: <https://crates.io/crates/async-rdma>

* Docs: <https://docs.rs/async-rdma/0.1.0/async_rdma>

* RDMA introduction video: <https://www.youtube.com/watch?v=lu78_C-9jvA>

* RDMA introduction doc: <http://www.reports.ias.ac.in/report/12829/understanding-the-concepts-and-mechanisms-of-rdma>
