+++
title = "How does codesandbox clone VMs quickly?"
date = 2024-01-18
+++

Being able to spin up dev environments in the cloud almost instantly enables users to:
- fork and run any piece of code you see online
- spin up a disposable environment to review someone's PR
- create different environments for different branches

Usecases like an online IDE where the environment does not need to keep running when the user has left, 
or where the VM can be stopped when the user is inactive to save cost; mean that the user experience improves if you
can reduce the delay between the user returning and the VM being ready to use.
<!-- more -->

Codesandbox shares how they achieved this in these two blog posts:
- [How we clone a running VM in 2 seconds](https://codesandbox.io/blog/how-we-clone-a-running-vm-in-2-seconds)
- [Cloning microVMs by sharing memory through userfaultfd](https://codesandbox.io/blog/cloning-microvms-using-userfaultfd)

In the first blog they describe how they reduced the time to clone a VM -
start another VM with the snapshot of the intial VM's state.
Cloning involves the following steps:
1. Pause VM
2. Save snapshot
3. Copy memory files
4. Start new VM from those files

Steps 2 and 3 take up the most time.

Firecracker does not load the entire snapshot file into memory, instead it uses `mmap`
which reserves a space in memory which points to the actual location of file on disk, 
the only parts loaded into memory are the ones that are actually read.

So when you write to memory the changes are not synced to the actual file on disk,
they are synced when you call `create_snapshot`, these saves then take too long to write.
However passing `MAP_SHARED` to mmap keeps syncing the changes to the backing file.
So only a little amount is left to be synced when we call `create_snapshot`

Saving a snapshot is now faster.

To reduce the time for copying these snapshots, we do not copy the memory files byte-for-byte, 
instead the new VM uses the same data as the old VM. 
Whenever the new VM needs to make a change to its data, 
it will copy the data from the old VM and apply the change to that data.

(Interestingly this is the default behaviour of the VM when it comes to writing a snapshot, 
i.e. it creates a copy of the memory instead of writing to the original file on disk.)