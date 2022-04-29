# QEMU Exclusive Work

- do something when no vCPU is executing (aka. outside cpu-exec)
- synchronization for MTTCG

- example: `tb_flush` 

## TB flush

```c
void tb_flush(CPUState *cpu)
{
    if (tcg_enabled()) {
        unsigned tb_flush_count = qatomic_mb_read(&tb_ctx.tb_flush_count);

        if (cpu_in_exclusive_context(cpu)) {
            do_tb_flush(cpu, RUN_ON_CPU_HOST_INT(tb_flush_count));
        } else {
            async_safe_run_on_cpu(cpu, do_tb_flush,
                                  RUN_ON_CPU_HOST_INT(tb_flush_count));
        }
    }
}
```

- #3: flush tb is designed for TCG only
- #6: `tb_flush` could be done in exclusive context ONLY
- #9: put into the work list of cpu and KICK it(self)

```c
tb_flush()
-> async_save_run_on_cpu()
   -> queue_work_on_cpu()
      -> qemu_cpu_kick()
         -> mttcg_kick_vcpu_thread()
            -> cpu_exit()
               -> qatomic_set(&FLAG, -1) // FLAG will be checked inside each TB

void qemu_cpu_kick(CPUState *cpu)
{
    qemu_cond_broadcast(cpu->halt_cond);
    if (cpus_accel->kick_vcpu_thread) {
        cpus_accel->kick_vcpu_thread(cpu);
    } else { /* default */
        cpus_kick_thread(cpu);
    }
}

void mttcg_kick_vcpu_thread(CPUState *cpu)
{
    cpu_exit(cpu);
}

void cpu_exit(CPUState *cpu)
{
    qatomic_set(&cpu->exit_request, 1);
    /* Ensure cpu_exec will see the exit request after TCG has exited.  */
    smp_wmb();
    qatomic_set(&cpu->icount_decr_ptr->u16.high, -1);    
}
```

### execution trace

```c
cpu_exec()
-> cpu_exec_enter(cpu);
-> sigsetjmp()
-> while (!cpu_handle_exception())
|  -> while (!cpu_handle_interrupt())
|  |  -> tb = tb_find()
|  |  |  -> tb = tb_lookup()
|  |  |  -> if (tb == NULL)
|  |  |  |  -> tb = tb_gen_code()
|  |  |  |  |  -> tb = tcg_tb_alloc()
|  |  |  |  |  -> if (!tb) // no more space to alloc TB
|  |  |  |  |  |  -> tb_flush(cpu)
|  |  |  |  |  |  -> cpu->exception_index = EXCP_INTERRUPT
|  |  |  |  |  |  -> cpu_loop_exit(cpu) // siglongjmp()
|  |  |  |  -> set into jmp cache
|  |  |  -> tb_add_jump()
|  |  -> cpu_loop_exec_tb()
-> cpu_exec_exit()
```

### execution graph

```c
/* CPU exec */

( cpu_exec_start )  +-----> ( sigsetjmp ) <------------------------+
      1 |         2 |           3 | 8                            7 |
   ( cpu_exec ) ----+   9         |                                |
        | 10 <-------------( handle_excp )                         |
(  cpu_exec_end  )              4 |        5                       |
                             ( tb_flush ) --> ( cpu->excp_index )  |
                                                     6 |           |
                                              ( cpu_loop_exit  ) --+
```

```c
/* mttcg thread */

( tcg_cpus_exec ) // start + exec + end
    ↓ loop ↑
( wait_io_event ) -------> ( start_exclusive )
                                    ↓
                              ( wi->func )
                                    ↓
                            ( end_exclusive ) 
```

## start or end exclusive

```c
/* Wait for pending exclusive operations to complete.  The CPU list lock
   must be held.  */
static inline void exclusive_idle(void)
{
    while (pending_cpus) {
        qemu_cond_wait(&exclusive_resume, &qemu_cpu_list_lock);
    } 
}
```

- `exclusive_idle` wait for a vCPU finishing its exclusive work

### start exclusive

```c
void start_exclusive(void)
{
    CPUState *other_cpu;
    int running_cpus;

    qemu_mutex_lock(&qemu_cpu_list_lock);
    exclusive_idle();

    /* Make all other cpus stop executing.  */
    qatomic_set(&pending_cpus, 1);

    /* Write pending_cpus before reading other_cpu->running.  */
    smp_mb();
    running_cpus = 0;
    CPU_FOREACH(other_cpu) {
        if (qatomic_read(&other_cpu->running)) {
            other_cpu->has_waiter = true;
            running_cpus++;
            qemu_cpu_kick(other_cpu);
        }
    }

    qatomic_set(&pending_cpus, running_cpus + 1);
    while (pending_cpus > 1) {
        qemu_cond_wait(&exclusive_cond, &qemu_cpu_list_lock);
    }

    /* Can release mutex, no one will enter another exclusive
     * section until end_exclusive resets pending_cpus to 0.
     */
    qemu_mutex_unlock(&qemu_cpu_list_lock);

    current_cpu->in_exclusive_context = true;
}
```

- #6: LOCK `qemu_cpu_list_lock` 
- #7: `exclusive_idle()` wait for other exclusive works to be finished on other vCPUs
- #10: set to stop other vCPUs executing [more details](#cpu-exec)
  - In short: `pending_cpus` will be checked before and after`cpu-exec` 
- #13: make sure other vCPUs will see the `pending_cpus` before this vCPU read other vCPU's running state
- #14: counter
- #16: read other vCPU's running state
- #17-19: we know this vCPU is running inside cpu-exec
  - #17: tell it we are wating for you to stop
  - #18: increase the counter
  - #19: kick it to stop executing
- #23: set the `pending_cpus` to the number of vCPUs we are waiting for
- #24-25: keep waiting
  - #24: check the number of `pending_cpus` repeatly
  - #25: wait for other vCPUs to stop
    - the `exclusive_cond` will be signaled after `cpu-exec` [more details](#cpu-exec)
- #28: at here, no other vCPUs are executing, and the `pending_cpus` is 1
- #31: UNLOCK `qemu_cpu_list_lock`
- #33: mark current cpu in exclusive context

### end exclusive

```c
/* Finish an exclusive operation.  */
void end_exclusive(void)
{
    current_cpu->in_exclusive_context = false;

    qemu_mutex_lock(&qemu_cpu_list_lock);
    qatomic_set(&pending_cpus, 0);
    qemu_cond_broadcast(&exclusive_resume);
    qemu_mutex_unlock(&qemu_cpu_list_lock);
}
```

- #4: reset the state of exclusive context
- #6: LOCK `qemu_cpu_list_lock`
- #7: set the `pending_cpus` to 0, which means no vCPU is waiting to enter exclusive context
- #8: signal the `exclusive_resume` , let the vCPUs that are waiting for this vCPU finishing the exclusive works to continue to execute
- #9: UNLOCK `qemu_cpu_list_lock` 

## cpu-exec

### cpu exec start

```c
void cpu_exec_start(CPUState *cpu)
{
    qatomic_set(&cpu->running, true);
    
    /* Write cpu->running before reading pending_cpus.  */
    smp_mb();

    if (unlikely(qatomic_read(&pending_cpus))) {
        QEMU_LOCK_GUARD(&qemu_cpu_list_lock);
        if (!cpu->has_waiter) {
            /* Not counted in pending_cpus, let the exclusive item
             * run.  Since we have the lock, just set cpu->running to true
             * while holding it; no need to check pending_cpus again.
             */
            qatomic_set(&cpu->running, false);
            exclusive_idle();
            /* Now pending_cpus is zero.  */
            qatomic_set(&cpu->running, true);
        } else {
            /* Counted in pending_cpus, go ahead and release the
             * waiter at cpu_exec_end.
             */
        }
	}
}
```

- #3: mark current vCPU is running
- #6: memory barrier, make sure others vCPUs will see the `cpu->running` is TRUE
  - before this vCPU read the `pending_cpus` 
- #8: check **pending_cpus**
  - `pending_cpus == 0` means no other vCPU wants to enter exclusive context
  - `pending_cpus > 0` means there is one vCPU wants to enter exclusive context
- #9: LOCK `qemu_cpu_list_lock` and release it before this function returnes automatically
- #10: check `cpu->has_waiter` 
  - if `cpu->has_waiter` is FALSE, it means this vCPU is not counted in the `start_exclusive()` of other vCPU
  - WHY? because the `start_exclusive()` is protected by the LOCK `qemu_cpu_list_lock` 
  - at here, if this vCPU is counted, the `cpu->has_waiter` must be TRUE
  - but `pending_cpus` is larger than 0, which means there must be one vCPU waiting
  - since this vCPU is not counted, it should stop executing
- #15: stop executing
- #16:  `exclusive_idle()` wait for that vCPU finishes its exclusive work
- #17: resume executing
- #20: the `cpu->has_waiter` is TRUE, this vCPU could continue to execute until finish

### cpu exec end

```c
void cpu_exec_end(CPUState *cpu)
{
    qatomic_set(&cpu->running, false);

    /* Write cpu->running before reading pending_cpus.  */
    smp_mb();

    if (unlikely(qatomic_read(&pending_cpus))) {
        QEMU_LOCK_GUARD(&qemu_cpu_list_lock);
        if (cpu->has_waiter) {
            cpu->has_waiter = false;
            qatomic_set(&pending_cpus, pending_cpus - 1);
            if (pending_cpus == 1) {
                qemu_cond_signal(&exclusive_cond);
            }
        }
    }
}
```

- #3: mark current vCPU is not running
- #6: memory barrier, make sure others vCPUs will see the `cpu->running` is FALSE
  - before this vCPU read the `pending_cpus` '
- #8: check **pending_cpus** 
  - `pending_cpus == 0` means no other vCPU wants to enter exclusive context
  - `pending_cpus > 0` means there is one vCPU wants to enter exclusive context
- #9: LOCK `qemu_cpu_list_lock` and release it before this function returnes automatically
- #10: check `cpu->has_waiter` 
  - if `cpu->has_waiter` is TRUE, it means this vCPU is counted by other vCPU
- #11: reset to FALSE
- #12: decrease the `pending_cpus` by 1, tell that vCPU that this vCPU is stopped
- #13: if this vCPU is the last vCPU that is being waited by another vCPU
- #14: signal the `exclusive_cond` to tell that vCPU that no other vCPUs is executing now

## locks, conditions and certain variables

### qemu cpu list lock

- in system mode, used **mainly** for the following functions' serialization
  -  `start_exclusive()` 
  -  `end_exclusive()` 
  -  `cpu_exec_start()` 
  -  `cpu_exec_end()` 
- also used in initilization
  -  `cpu_exec_realizefn() -> cpu_list_add() ` 
  -  `cpu_exec_unrealizefn() -> cpu_list_remove()` 

### pending cpus

- number of vCPUs that need to stop executing
- set in `start_exclusive()` ONLY and clear in `start_exclusive()` ONLY
  - which means it is protected by the LOCK `qemu_cpu_list_lock` 
- when `pending_cpus` is large than one (eg. 3)
  - one vCPU wants to enter exclusive context
  - and it is waiting for other vCPUs stop executing (eg. wait for other 2 vCPUs)
- when `pending_cpus` is equal to one
  - one vCPU wants to enter exclusive context and it could be
    - counting number of vCPU to wait
    - executing the exclusive work

### usage of pthead cond

- example from stackoverflow

```c
thread 1:
    pthread_mutex_lock(&mutex);
    while (!condition)
        pthread_cond_wait(&cond, &mutex);
    /* do something that requires holding the mutex and condition is true */
    pthread_mutex_unlock(&mutex);

thread2:
    pthread_mutex_lock(&mutex);
    /* do something that might make condition true */
    pthread_cond_signal(&cond);
    pthread_mutex_unlock(&mutex);
```

## case study

### the simplest one

- **vCPU0** calls `tb_flush()` , **vCPU1** is executing blocks
- **vCPU0** add this work into cpu work list, and kick itself
- **vCPU0** calls `start_exclusive()` , see **vCPU1** is executing, set `pending_cpus = 2` 
- **vCPU0** set **vCPU1** `has_waiter` to TRUE and kick the **vCPU1**
- **vCPU0** waits for **vCPU1** through `qemu_cond_wait()` on `exclusive_cond` and release the *LOCK* `qemu_cpu_list_lock` 
- **vCPU1** is kicked out, and calls `cpu_exec_end()` 
- **vCPU1** see the `pending_cpus` is large than zero, so it knows other vCPU is waiting
- **vCPU1** get the *LOCK* `qemu_cpu_list_lock` 
- **vCPU1** decrese the `pending_cpus` by 1, and it finds out that the `pending_cpus` is equal to 1, then it will signal the `exclusive_cond` 
- **vCPU1** return from `cpu_exec_end()` and release `qemu_cpu_list_lock` automatically
- **vCPU0** resume from pthread cond waiting, get the *LOCK* `qemu_cpu_list_lock` again
- **vCPU1** calls `cpu_exec_start()` to go on executing, but `pending_cpus` is 1, so it will keep waiting on the condition `exclusive_resume()` 
- **vCPU0** does the exclusive work, finally reset the `pending_cpus` to zero and broadcast the condition `exclusive_resume` in `end_exclusive()` 
- **vCPU0** goes on executing
- **vCPU1** receives the `exclusive_resume` and goes on executing

### not counted vCPU1

- **vCPU0** calls `tb_flush()` , add this work into cpu work list, and kick itself
- **vCPU0** calls `start_exclusive()` , set `pending_cpus = 1`  and start count other vCPUs
- **vCPU0** see **vCPU1** is not running, release the *LOCK* `qemu_cpu_list_lock` 
- **vCPU0** goes on executing the exclusive work
- **vCPU1** just calls `cpu_exec_start()` and set its running state to TRUE
- **vCPU1** find the `pending_cpus` is equal to 1 in `cpu_exec_start()` 
- **vCPU1** get the *LOCK* `qemu_cpu_list_lock` 
- **vCPU1** see its `has_waiter` is FALSE, and it knows that it is not counted by the **vCPU0** 
- **vCPU1** set its running state to FALSE and call `exclusive_idle()` to wait for **vCPU0** 
- **vCPU0** finishse the exclusive work, reset the `pending_cpus` to zero and broadcaset
- **vCPU0** goes on executing
- **vCPU1** receives the signal, it set its running state to TRUE and goes on executing

### vCPU1 wants to enter exclusive context

when **vCPU1** starts the `start_exclusive()` it will execute the `exclusive_idle()` to wait for **vCPU0** finished its exclusive work

















































