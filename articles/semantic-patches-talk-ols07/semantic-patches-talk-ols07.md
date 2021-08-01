# Semantic Patches

## Introduction

We propose a **scripting language** which offers a WYSIWYG, What You See Is What You Get, approach to **program transformation**. In the spirit of Linux development practice, this language is based on the patch syntax. As opposed to traditional patches, our patches are not line-oriented but **semantics-oriented**, and hence we give them the name *semantic patches*.

This paper gives a tutorial on our semantic patch language, SmPL, and its associated transformation tool, spatch. We first give an idea of the kind of program transformations we target, collateral evolutions, and then present SmPL using an example based on Linux driver code. Finally, we describe the current status of our project and propose some future work.

## Evolutions and Collateral Evolutions

The evolutions we consider are those that affect a library API. Elements of a library API that can be affected include **functions, global variables and constants, types, and macros**.

Many kinds of changes in the API can result from an evolution that affect one of these elements. For example, functions or macros can change name or gain or lose arguments. Structure types can be reorganized and accesses to them can be encapsulated in getter and setter functions. The protocol for using a sequence of functions, such as up and down can change, as can the protocol for when error checking is needed and what kind of error values should be returned.

Each of these changes requires corresponding collateral evolutions, in all drivers using the API. Thus, these changes have to be mapped onto the structure of each affected driver file.

## Semantic Patch Tutorial

### The “proc_info” evolution

As an example, we consider an evolution and associated collateral evolutions affecting the SCSI API functions `scsi_host_hn_get` and `scsi_host_put`. These functions access and release, respectively, a structure of type `Scsi_Host`, and additionally increment and decrement, respectively, a reference count. In Linux 2.5.71, it was decided that, due to the criticality of the reference count, driver code could not be trusted to use these functions correctly and they were removed from the SCSI API.

<img src="img/simplified_excerpt_of_the_patch_file.png" style="zoom:67%;" />

This evolution had collateral effects on the “proc_info” callback functions defined by SCSI drivers, which call these API functions. The code above shows a slightly simplified excerpt of the traditional patch file. Similar collateral evolutions were performed in Linux 2.5.71 in 18 other SCSI driver files inside the kernel source tree.

### A semantic patch, step by step

We now describe the semantic patch that will perform the previous collateral evolutions, on any of the 19 relevant files inside the kernel source tree, and on any relevant drivers outside the kernel source tree. We first describe step-by-step various excerpts of this semantic patch, and then present its complete definition later.

#### Modifiers

The first excerpt adds and removes the affected parameters of the proc_info callback function:

```C
 proc_info_func (
+     struct Scsi_Host *hostptr,
      char *buffer, char **start, off_t offset,
-     int hostno,
      int inout) { ... }
```

