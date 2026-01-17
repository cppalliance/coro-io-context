# Capy + Corosio Path to Review

This document compares the combined features of Boost.Capy and Boost.Corosio against Boost.Asio to identify gaps for feature parity.

## Design Philosophy & Scope

**Capy and Corosio are coroutine-first libraries.** The following Asio features are explicitly out of scope:

- **Callbacks / completion handlers** - Use coroutines instead
- **Futures (`use_future`)** - Use `task<T>` and `co_await` instead
- **Stackless macro coroutines (`BOOST_ASIO_CORO_YIELD`)** - Use real C++20 coroutines
- **Generalized completion tokens** - Coroutines provide a single, unified async model
- **Associated characteristics** - Unnecessary complexity; allocators/executors handled differently
- **`system_executor`** - Causes confusion about where callbacks run; not needed with coroutines

This focused design eliminates the complexity of supporting multiple async paradigms while providing a cleaner, more predictable programming model.

## Feature Comparison Matrix

### Legend
- :white_check_mark: Implemented
- :x: **Gap** - Needed for review (maybe partial)
- :construction: **Planned** - Post-review implementation
- :hourglass: **Deferred** - Implement when requested
- '-': Out of scope

---

## 1. Asynchronous Model

| Feature | Asio | Capy+Corosio | Notes |
|---------|------|--------------|-------|
| Proactor pattern | :white_check_mark: | :white_check_mark: | |
| Completion handlers (callbacks) | :white_check_mark: | &nbsp;&nbsp;- | Out of scope: use coroutines |
| Futures (`use_future`) | :white_check_mark: | &nbsp;&nbsp;- | Out of scope: use coroutines |
| C++20 coroutines (`use_awaitable`) | :white_check_mark: | :white_check_mark: | |
| Deferred operations (`deferred`) | :white_check_mark: | :white_check_mark: | `auto t = coro();` |
| Detached operations (`detached`) | :white_check_mark: | :white_check_mark: | `capy::async_run` |
| Cancellation (per-operation) | :white_check_mark: | :white_check_mark: | `std::stop_token` |
| Cancellation slots | :white_check_mark: | :white_check_mark: | `co_await capy::get_stop_token()` |
| Parallel operations (`parallel_group`) | :white_check_mark: | :white_check_mark: | `capy::when_all`, et. al. |
| Associated characteristics | :white_check_mark: | &nbsp;&nbsp;- | Out of scope: unnecessary complexity |

**Note on deferred operations:** In coroutine-based code, deferred execution is natural:
```cpp
auto rv = co_await do_session(sock);  // immediate execution
auto t = do_session(sock);            // deferred - task not started until awaited
```

---

## 2. Execution Context & Executors

| Feature | Asio | Capy+Corosio | Notes |
|---------|------|--------------|-------|
| `io_context` | :white_check_mark: | :white_check_mark: | |
| `thread_pool` | :white_check_mark: | :white_check_mark: | |
| `system_executor` | :white_check_mark: | &nbsp;&nbsp;- | Out of scope: causes callback confusion |
| `any_io_executor` | :white_check_mark: | :white_check_mark: | `capy::any_dispatcher` (zero-alloc) |
| `strand` | :white_check_mark: | :white_check_mark: | `capy::stran` |
| `executor_work_guard` | :white_check_mark: | :white_check_mark: | `capy::executor_work_guard` |
| `execution_context` base | :white_check_mark: | :white_check_mark: | `capy::execution_context` |
| `execution_context::service` | :white_check_mark: | :white_check_mark: | `capy::execution_context::service` |
| `post()` / `dispatch()` / `defer()` | :white_check_mark: | :white_check_mark: | `concept capy::executor` |
| `run()` / `run_one()` | :white_check_mark: | :white_check_mark: | `corosio::io_context` |
| `run_for()` / `run_until()` | :white_check_mark: | :white_check_mark: | `corosio::io_context` |
| `poll()` / `poll_one()` | :white_check_mark: | :white_check_mark: | `corosio::io_context` |
| `stop()` / `restart()` | :white_check_mark: | :white_check_mark: | `corosio::io_context` |
| Concurrency hint | :white_check_mark: | :white_check_mark: | `corosio::io_context` |

---

## 3. Networking - TCP

| Feature | Asio | Capy+Corosio | Notes |
|---------|------|--------------|-------|
| TCP socket | :white_check_mark: | :white_check_mark: | `socket` |
| TCP acceptor | :white_check_mark: | :white_check_mark: | `acceptor` |
| `async_connect` | :white_check_mark: | :white_check_mark: | `socket::connect` |
| `async_accept` | :white_check_mark: | :white_check_mark: | `acceptor::accept` |
| `async_read_some` | :white_check_mark: | :white_check_mark: | `io_stream::read_some`|
| `async_write_some` | :white_check_mark: | :white_check_mark: | `io_stream::write_some` |
| `async_read` (composed) | :white_check_mark: | :white_check_mark: | `capy::read` |
| `async_read_until` | :white_check_mark: | :x: | `capy::read_until` |
| `async_write` (composed) | :white_check_mark: | :white_check_mark: | `capy::write` |
| Socket options | :white_check_mark: | :construction: | **Planned** |
| `no_delay` (Nagle) | :white_check_mark: | :construction: | **Planned** |
| `keep_alive` | :white_check_mark: | :construction: | **Planned** |
| `reuse_address` | :white_check_mark: | :construction: | **Planned** |
| `linger` | :white_check_mark: | :construction: | **Planned** |
| `receive_buffer_size` | :white_check_mark: | :construction: | **Planned** |
| `send_buffer_size` | :white_check_mark: | :construction: | **Planned** |
| `shutdown()` | :white_check_mark: | :x: | |
| `local_endpoint()` | :white_check_mark: | :x: | Design for noexcept |
| `remote_endpoint()` | :white_check_mark: | :x: | Design for noexcept |
| `native_handle()` | :white_check_mark: | :x: | |
| Non-blocking mode | :white_check_mark: | :construction: | Needs research |
| IPv4 | :white_check_mark: | :white_check_mark: | |
| IPv6 | :white_check_mark: | :x: | |

**Implementation note:** `local_endpoint()` and `remote_endpoint()` could cache values when socket becomes connected, allowing noexcept access.

---

## 4. Networking - UDP

| Feature | Asio | Corosio | Notes |
|---------|------|--------------|-------|
| UDP socket | :white_check_mark: | :construction: | **Planned** |
| `async_send_to` | :white_check_mark: | :construction: | **Planned** |
| `async_receive_from` | :white_check_mark: | :construction: | **Planned** |
| `async_send` (connected) | :white_check_mark: | :construction: | **Planned** |
| `async_receive` (connected) | :white_check_mark: | :construction: | **Planned** |
| Multicast join/leave | :white_check_mark: | :construction: | **Planned** |
| Broadcast | :white_check_mark: | :construction: | **Planned** |

---

## 5. Networking - Other Protocols

| Feature | Asio | Corosio | Notes |
|---------|------|--------------|-------|
| ICMP socket | :white_check_mark: | :construction: | **Planned** |
| Raw sockets | :white_check_mark: | :construction: | **Planned** |
| Generic sockets | :white_check_mark: | :construction: | **Planned** |
| Unix domain sockets (stream) | :white_check_mark: | :construction: | **Planned** |
| Unix domain sockets (datagram) | :white_check_mark: | :construction: | **Planned** |
| Local endpoints | :white_check_mark: | :construction: | **Planned** |

---

## 6. DNS Resolution

| Feature | Asio | Corosio | Notes |
|---------|------|--------------|-------|
| `resolver` | :white_check_mark: | :white_check_mark: | `corosio::resolver` |
| `async_resolve` | :white_check_mark: | :white_check_mark: | |
| Resolve flags | :white_check_mark: | :white_check_mark: | |
| `resolver_results` iteration | :white_check_mark: | :white_check_mark: | |
| Reverse lookup | :white_check_mark: | :x: | |

---

## 7. SSL/TLS

| Feature | Asio | Corosio | Notes |
|---------|------|--------------|-------|
| SSL context | :white_check_mark: | :x: | |
| SSL stream | :white_check_mark: | :x: | |
| `async_handshake` | :white_check_mark: | :white_check_mark: | |
| `async_shutdown` | :white_check_mark: | :x: | |
| Client/server handshake types | :white_check_mark: | :white_check_mark: | |
| Certificate verification | :white_check_mark: | :construction: | **Planned** |
| SNI (Server Name Indication) | :white_check_mark: | :construction: | **Planned** |
| ALPN | :white_check_mark: | :construction: | **Planned** |
| OpenSSL backend | :white_check_mark: | :x: | |
| WolfSSL backend | :x: | :white_check_mark: | `corosio::wolfssl_stream` |

---

## 8. Timers

| Feature | Asio | Capy+Corosio | Notes |
|---------|------|--------------|-------|
| `steady_timer` | :white_check_mark: | :x: | |
| `system_timer` | :white_check_mark: | :x: | |
| `high_resolution_timer` | :white_check_mark: | :x: | |
| `async_wait` | :white_check_mark: | :x: | |
| Timer cancellation | :white_check_mark: | :x: | |
| `expires_at()` / `expires_after()` | :white_check_mark: | :x: | |
| Custom clock support | :white_check_mark: | :x: | |

---

## 9. Signal Handling

| Feature | Asio | Capy+Corosio | Notes |
|---------|------|--------------|-------|
| `signal_set` | :white_check_mark: | :x: | |
| `async_wait` for signals | :white_check_mark: | :x: | |
| Add/remove signals | :white_check_mark: | :x: | |
| Cross-platform (including Windows) | :white_check_mark: | :x: | |

---

## 10. Serial Ports

| Feature | Asio | Capy+Corosio | Notes |
|---------|------|--------------|-------|
| `serial_port` | :white_check_mark: | :hourglass: | **Deferred** |
| Baud rate | :white_check_mark: | :hourglass: | **Deferred** |
| Character size | :white_check_mark: | :hourglass: | **Deferred** |
| Flow control | :white_check_mark: | :hourglass: | **Deferred** |
| Parity | :white_check_mark: | :hourglass: | **Deferred** |
| Stop bits | :white_check_mark: | :hourglass: | **Deferred** |

---

## 11. File I/O

| Feature | Asio | Capy+Corosio | Notes |
|---------|------|--------------|-------|
| Sync file operations | :white_check_mark: | :x: | |
| `stream_file` (sequential) | :white_check_mark: | :construction: | **Planned** |
| `random_access_file` | :white_check_mark: | :construction: | **Planned** |
| `async_read_some_at` | :white_check_mark: | :construction: | **Planned** |
| `async_write_some_at` | :white_check_mark: | :construction: | **Planned** |

---

## 12. Pipes

| Feature | Asio | Capy+Corosio | Notes |
|---------|------|--------------|-------|
| `readable_pipe` | :white_check_mark: | :hourglass: | **Deferred** |
| `writable_pipe` | :white_check_mark: | :hourglass: | **Deferred** |
| `connect_pipe()` | :white_check_mark: | :hourglass: | **Deferred** |

---

## 13. POSIX-Specific

| Feature | Asio | Capy+Corosio | Notes |
|---------|------|--------------|-------|
| `posix::stream_descriptor` | :white_check_mark: | :hourglass: | **Deferred** |
| `posix::basic_descriptor` | :white_check_mark: | :hourglass: | **Deferred** |
| Fork support | :white_check_mark: | :hourglass: | **Deferred** |

---

## 14. Windows-Specific

| Feature | Asio | Capy+Corosio | Notes |
|---------|------|--------------|-------|
| `windows::stream_handle` | :white_check_mark: | :hourglass: | **Deferred** |
| `windows::random_access_handle` | :white_check_mark: | :hourglass: | **Deferred** |
| `windows::object_handle` | :white_check_mark: | :hourglass: | **Deferred** |
| `windows::overlapped_ptr` | :white_check_mark: | :hourglass: | **Deferred** |

**Note:** Windows-specific handles are deferred. These are non-portable and will only be considered if explicitly requested.

---

## 15. Buffers

| Feature | Asio | Capy+Corosio | Notes |
|---------|------|--------------|-------|
| `const_buffer` | :white_check_mark: | :white_check_mark: | `capy::const_buffer` |
| `mutable_buffer` | :white_check_mark: | :white_check_mark: | `capy::mutable_buffer` |
| Buffer sequences | :white_check_mark: | :white_check_mark: | Capy concepts|
| `dynamic_buffer` | :white_check_mark: | :white_check_mark: | `capy::flat_buffer`, et. al. |
| `buffer()` factory | :white_check_mark: | :x: | |
| `buffer_copy()` | :white_check_mark: | :white_check_mark: | `capy::buffer_copy` |
| `buffer_size()` | :white_check_mark: | :white_check_mark: | `capy::buffer_size` |
| Buffer algorithms | :x: | :white_check_mark: | `capy::prefix`, et. al. |
| `streambuf` | :white_check_mark: | :hourglass: | **Deferred** |
| `buffered_stream` | :white_check_mark: | :hourglass: | **Deferred** |
| `buffered_read_stream` | :white_check_mark: | :hourglass: | **Deferred** |
| `buffered_write_stream` | :white_check_mark: | :hourglass: | **Deferred** |

---

## 16. Iostreams

| Feature | Asio | Capy+Corosio | Notes |
|---------|------|--------------|-------|
| `ip::tcp::iostream` | :white_check_mark: | :hourglass: | **Deferred** |
| Socket streambuf | :white_check_mark: | :hourglass: | **Deferred** |

---

## 17. Coroutine Support

| Feature | Asio | Capy+Corosio | Notes |
|---------|------|--------------|-------|
| `awaitable<T>` | :white_check_mark: | :white_check_mark: | `capy::task` |
| `co_spawn` | :white_check_mark: | :white_check_mark: | `capy::async_run` |
| `this_coro::executor` | :white_check_mark: | :x: | |
| `this_coro::cancellation_state` | :white_check_mark: | :white_check_mark: | `co_await capy::get_stop_token()` |
| `this_coro::reset_cancellation_state` | :white_check_mark: | &nbsp;&nbsp;- | Not applicable |
| Stackless macro coroutines | :white_check_mark: | &nbsp;&nbsp;- | Uses callbacks |
| Stackful coroutines | :white_check_mark: | :hourglass: | **Deferred** |

---

## 18. Utilities

| Feature | Asio | Capy+Corosio | Notes |
|---------|------|--------------|-------|
| `async_compose` | :white_check_mark: | :white_check_mark: | * |
| `async_initiate` | :white_check_mark: | :white_check_mark: | * |
| `bind_executor` | :white_check_mark: | :white_check_mark: | * |
| `bind_allocator` | :white_check_mark: | :white_check_mark: | * |
| `bind_cancellation_slot` | :white_check_mark: | :white_check_mark: | stop token everywhere |
| `redirect_error` | :white_check_mark: | &nbsp;&nbsp;- | `corosio::io_result` |
| `as_tuple` | :white_check_mark: | &nbsp;&nbsp;- | `corosio::io_result` |

** \* ** Many Asio utilities exist to bridge different async models. With a coroutine-only design, these capabilities emerge naturally without explicit APIs.

---

## Summary: Priority Gaps for Review

### Gaps (Must Have for Review) :x:

| Feature | Notes |
|---------|-------|
| **Timers** (`steady_timer`) | Core functionality |
| **Signal handling** (`signal_set`) | Graceful shutdown |
| **`async_read_until`** | Line-based protocols |
| **`shutdown()`** | Clean connection termination |
| **`local_endpoint()` / `remote_endpoint()`** | Cache on connect for noexcept |
| **`native_handle()`** | Interop with other libraries |
| **Reverse DNS lookup** | DNS completeness |
| **OpenSSL backend** | Immediate target |
| **Sync file operations** | Basic file I/O |

### Planned (Post-Review) :construction:

| Feature | Notes |
|---------|-------|
| **Socket options** | `set_option`/`get_option` and common options |
| **Non-blocking mode** | Needs research for coroutine signaling |
| **UDP sockets** | Full UDP support |
| **Other protocols** | ICMP, raw sockets, Unix domain sockets |
| **SSL certificate verification** | Security |
| **SSL SNI/ALPN** | Virtual hosting, HTTP/2 |
| **SSL `async_shutdown`** | Clean TLS termination |
| **Async file I/O** | `stream_file`, `random_access_file` |
| **IPv6 full support** | Complete implementation |

### Deferred (On Request) :hourglass:

| Feature | Notes |
|---------|-------|
| **Serial ports** | Implement when requested |
| **Pipes** | Low priority |
| **POSIX stream descriptors** | On-demand |
| **Windows-specific handles** | Non-portable |
| **Buffered streams** | Rarely used |
| **`streambuf`** | Compatibility |
| **Stackful coroutines** | Not ruled out for future |

---

## Recommended Implementation Order

### Phase 1: Gaps (For Review)
1. `steady_timer` with `async_wait`, cancellation, `expires_after()`
2. `signal_set` for graceful shutdown
3. Socket query methods (`local_endpoint`, `remote_endpoint`, `native_handle`)
4. `shutdown()` support
5. `async_read_until` for line-based protocols
6. Reverse DNS lookup
7. OpenSSL backend (alongside WolfSSL)
8. Sync file operations

### Phase 2: Planned (Post-Review)
1. Socket options API (`set_option`, `get_option`, common options)
2. UDP socket implementation
3. Unix domain sockets
4. SSL completeness (cert verification, SNI, ALPN, `async_shutdown`)
5. Async file I/O
6. Other protocols (ICMP, raw sockets)

### Phase 3: Deferred (On Request)
- Implement features as users request them

---

## Unique Features (Capy+Corosio advantages over Asio)

| Feature | Description |
|---------|-------------|
| **Native C++20 coroutines** | First-class support, not bolted on |
| **Single async model** | No complexity from multiple completion token types |
| **`when_all` combinator** | Built-in parallel operation coordination |
| **Affine awaitable protocol** | Zero-overhead scheduler affinity |
| **Frame allocator system** | Custom coroutine frame allocation with recycling |
| **`async_mutex`** | Zero-allocation async mutex |
| **`io_result<T>`** | Structured binding friendly result type |
| **Stop token integration** | Native `std::stop_token` support |
| **Type-erased buffers** | `any_bufref` for virtual interfaces |
| **Modern error handling** | Both error codes and exceptions via `.value()` |
| **WolfSSL support** | Alternative TLS backend |
| **Natural deferred execution** | `auto t = coro();` defers without special tokens |

---

## References

- [Boost.Asio Documentation](https://www.boost.org/doc/libs/1_85_0/doc/html/boost_asio.html)
- [Boost.Asio Overview](https://www.boost.org/doc/libs/1_85_0/doc/html/boost_asio/overview.html)