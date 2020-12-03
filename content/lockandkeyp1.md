---
title: "Lock And Key - Part 1"
date: 2020-12-02T20:39:22+05:30
draft: false
---

## `1. A primer on processes and locks`

Every application, every program that runs on a computer runs as a process. The shell you use to interact with the os, the music player on your machine, your browser- basically, every set of instructions is executed as a process. A process is an instance of sourcecode that needs to be executed. Whenever a program is to be run, the kernel allocates certain resources like memory, address space etc. Data may be accessed by more than one process at the same time. A common situation is when there are multiple instances of an application running- such as several open tabs in a browser. You can verify this using [htop](). A process comprises data and memory resources that are associated with the program. When several processes try to concurrently access this common set of resources, proper synchronisation is necessary to implement efficient resource-sharing among the processes. This is where locking comes in.

Locks are implemented on shared data resources to ensure that processes are in sync when accessing common data. Only one process should have permission to modify or write to the data at a time. A second process can make changes to the same data only when the previous process is done with it. To facilitate this, processes acquire locks on the data for the duration of their execution and then release these locks from memory. It is necessary that every lock that is acquired must be released as well to prevent processes from hoarding memory and delaying other tasks.

Often, locks are acquired within a function only if certain conditions are met. However, this function can have control flow paths through the entire program. It must be ensured that all potential paths have an unlock as well. Given the huge extent of kernel code, it is easy to skip adding an unlock function to each path the funtion might take. We can leverage Coccinelle semantic patches to detect and examine such situations. 

## `2. Using Coccinelle`

The philosophy behind Coccinelle is to frame a pattern that satisfies the usecase we have in mind. This pattern is written in form of transformation rules. We want to find a scenario where a lock may be potentially acquired and released under different conditions leading to resource hogging. Let's start!

```
@prelocked@
expression E;
position p;
@@
mutex_lock(E@p)
```

Our first rule `prelocked` tries to check whether a lock is acquired. It keeps track of the resource on which lock is acquired using the variable `E`. The `@p` storesthe position of the lock declaration. This helps to identify the particular lock in case there are more than one locks. Next we define a second rule `balanced` :

```
@balanced@
position prelocked.p;
position pif;
expression e,prelocked.E;
statement S1,S2;
@@

if (e) {
 ... when any
mutex_lock(E@p)
... when != E
    when any
 else S1
.. when != E
   when any
f@pif (e) {
... when != E
    when any
mutex_unlock(E);
... when any
 else S2
```
Usually, to find a non-ideal condition- we need to define the ideal first. Drawing on that analogy we define a rule where locking has been properly implemented- i.e. the acquired lock is released under the same conditions as well. Later, we will take this balanced rule and negate it to detect situations where locking has been irregularly implemented.

We want to check if the lock is released under the same condition as under which it was acquired. The condition here is represented by the expression variable `e`. First, we check for the `mutex_lock` we found in the prelocked rule. In the next if block we check for `mutex_unlock` for the same condition `e`. In both these cases, we make sure the resource E is not called elsewhere in these blocks, so as to avoid false positives. We mark the if block where `mutex_unlock()` is called using the position variable `pif`.

Let's move on to the next rule where we define the pattern we want.

```
@@
expression E,x,f;
position prelocked.p;
position any balanced.pif;
statement S1,S2,S3,S4;
@@

*mutex_lock(E@p);
... when != mutex_unlock(E);
    when != if@pif (...) S1 else S2
    when any
x = f(...);
*if (<+...x...+>)
{
  ... when != mutex_unlock(E);
      when != if@pif (...) S3 else S4
* return ...;
}
... when any
*mutex_unlock(E);
... when != mutex_lock(E);
```
Here we inherit the `p` and the `pif` position metavariables to specify the respective `mutex_lock` and `mutex_unlock` we marked in the `prelocked` and `balanced` rules respetively. In

```
*mutex_lock(E@p);
... when != mutex_unlock(E);
    when != if@pif (...) S1 else S2
    when any

```
we first identify the mutex lock, then negate the if condition from the balanced rule, making sure `mutex_unlock` has not been called. Then,

```
x = f(...);
*if (<+...x...+>)
{
  ... when != mutex_unlock(E);
      when != if@pif (...) S3 else S4
* return ...;
}

```
we check to ensure that the mutex_unlock() has not been called within an arbitrary function denoted by `x=f(...)`. We also check that the balanced rule is not satisfied here. 

Finally, we check if `mutex_unlock` has been called in the program elsewhere and no other lock has been acquired:

```
... when any
*mutex_unlock(E);
... when != mutex_lock(E);
```

In all, we try to find a pattern where:
1. mutex_lock` is present but the balanced rule is not satisfied.
2. In the case where a function is declared within an if block, it returns without satisfying the balanced rule or calling `mutex_unlock`. 
3. `mutex_unlock` is called outside an if block.

We save this script in a _.cocci_ file and then run it to find code segments of interest.


## `3. Sematic patch results`
On running he script, I found several matching patches that did not have matching unlocks. Next, we need to verify these results, identify common false positive scenarios, nd improve the script. Some common scenarios were:
1. Sometimes, the unlock was called in a different function which was often specified under an if condition.
   ```
   diff -u -p ./drivers/vfio/vfio.c /tmp/nothing/vfio/vfio.c
       --- ./drivers/vfio/vfio.c
       +++ /tmp/nothing/vfio/vfio.c
       @@ -360,7 +360,6 @@ static struct vfio_group *vfio_create_gr
           return ERR_PTR(ret);
         }
   
       -mutex_lock(&vfio.group_lock);
         / Did we race creating this group? */
         lst_for_each_entry(tmp, &vfio.group_list, vfio_next) {
       @@ 372,19 +371,15 @@ static struct vfio_group *vfio_create_gr
         
         mnor = vfio_alloc_group_minor(group);
       -if (minor < 0) {
       vfio_group_unlock_and_free(group);
       -	return ERR_PTR(minor);
         
         dv = device_create(vfio.class, NULL,
             MKDEV(MAJOR(vfio.group_devt), minor),
                 group, "%s%d", group->noiommu ? "noiommu-" : "",
                 iommu_group_id(iommu_group));
       -	if (IS_ERR(dev)) {
           vfio_free_group_minor(minor);
           vfio_group_unlock_and_free(group);
       -		return ERR_CAST(dev);
         }
   ```
   In the `if` cases, there is no `mutex_unlock()` present. However, the `vfio_group_unlock_and_free()` function called in these `if` blocks calls the `mutex_unlock()` function. 

2. In another case, the conditional statements were directed towards a `goto` statement which called `mutex_unlock().`
           
    ```
    /* Check if bank switch was successful */
 	   dret = sdw_ml_sync_bank_switch(bus);
	   - if (ret < 0) {
           dev_err(bus->dev,
           "multi link bank switch failed: %d\n", ret);
           goto error;
         }
    ```
    Here, the `error` block contained a `mutex_unlock()`.
3. In yet other cases, certain conditions only returned an error and exited/ended. It was difficult to ascertain whether the intended behaviour was to exit the function with the lock still held.

    ```
    mutex_lock(&blka->mutex);
		locked = true;
		for (i = 0; i < rhte_src->lxt_cnt; i++) {
			aun = (lxt[i].rlba_base >> MC_CHUNK_SHIFT);
			if (ba_clone(&blka->ba_lun, aun) == -1ULL) {
				rc = -EIO;
				goto err;
			}
		}
	}
    
    rc = cxlflash_afu_sync(afu, ctxid, rhndl, AFU_LW_SYNC);
	if (unlikely(rc)) {
		rc = -EAGAIN;
		goto err2;
    }
    
    out:
    	if (locked)
    		mutex_unlock(&blka->mutex);
    	dev_dbg(dev, "%s: returning rc=%d\n", __func__, rc);
    	return rc;
    err2:
    	/* Reset the RHTE */
    	rhte->lxt_cnt = 0;
    	dma_wmb();
    	rhte->lxt_start = NULL;
    	dma_wmb();
    err:
    	/* free the clones already made */
    	for (j = 0; j < i; j++) {
    		aun = (lxt[j].rlba_base >> MC_CHUNK_SHIFT);
    		ba_free(&blka->ba_lun, aun);
    	}
    	kfree(lxt);
    	goto out;
    }
    ```
    If control ends up in `err2`, then the fate of the lock becomes unclear. `dma_wmb` has not been defined in `vlun.c`

4. In a specific case, sometimes the code checked whether the lock was held or not using a function such as `mutex_is_locked()` which returns a boolean.
    ```
     --- ./drivers/soundwire/stream.c       
     @@ -851,11 +845,9 @@ msg_unlock:
     list_for_each_entry(m_rt, &stream->master_list, stream_node) {
     	bus = m_rt->bus;
     	if (mutex_is_locked(&bus->msg_lock))
     		mutex_unlock(&bus->msg_lock);
     }
     
     
    return ret;
     
    ```

 Cases 2 and 4 have reliable patterns that can be matched by checking for `mutex_unlock()` within `goto` and `mutex_is_locked()`.
 In the next post, we will modify this script to eliminate these false positives.

