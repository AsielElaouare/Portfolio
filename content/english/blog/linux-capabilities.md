---
title: "introduction to Linux Capabilities"
meta_title: ""
description: "What are capabilities and how to use them?"
date: 2024-01-29T05:00:00Z
image: "images/gallery/linux-capabilities.png"
categories: ["Linux"]
author: "Asiel Elaouare"
tags: ["Linux", "Security", "Commands"]
draft: false
---

{{< toc >}}



When it comes to securing a Linux system, many users are familiar with the traditional file permissions—those simple, yet effective, "read," "write," and "execute" settings. However, there's an often-overlooked feature called capabilities that takes the idea of permissions to a whole new level. In this blog post, we'll explore Linux capabilities and discover how they offer a more fine-grained control over privileges, surprising even seasoned users.

## Introduction to Capabilities:
Linux capabilities extend the traditional permissions model, allowing us to grant specific privileges to processes. This goes beyond the broad strokes of read, write, and execute, providing a more granular level of control. To understand this, let's start by checking the capabilities associated with a process with this command ```getcap``` and the path to the binary file ```/path/to/binary ``` .

The output of the ```getcap``` command will display the capabilities associated with the specified binary. The format is typically shown as a list of capabilities, each prefixed with "cap_" and followed by the capability name.

For example, if the binary has the ```cap_net_bind_service capability```, the output might look like this:
```bash
/path/to/binary = cap_net_bind_service+ep
```

This output informs you about the capabilities that have been explicitly assigned to the binary, allowing you to understand the specific privileges it possesses beyond traditional file permissions.

Here:
```/path/to/binary``` is the path to the binary that the user specifies.
```cap_net_bind_service``` is the capability.
```+ep``` indicates that the capability is enabled on both effective and permitted sets.

The +ep in the context of Linux capabilities indicates that the specified capabilities are added to both the effective and permitted sets. However, the execution of a binary is primarily determined by traditional file permissions, not capabilities.

If you have set the effective and permitted capabilities with +ep on a binary, it means that when the binary runs, it has those capabilities available for its use. However, the actual execution of the binary is still subject to traditional file permissions. If the file permissions allow execution for a specific user (root or non-root), that user can execute the binary.

So, in summary:

Effective and Permitted Capabilities (+ep): This affects what the binary can do in terms of capabilities during its execution.

File Permissions: This determines who can execute the binary. If the file permissions allow execution for a specific user, that user (whether root or non-root) can execute the binary.

### Lets see a second example about a possible output.

We fisrt ask for the capabilities this binary has:
 ```bash 
 $ getcap /usr/bin/apt-get
 ```
 
 The output of the getcap /usr/bin/apt-get command will typically be empty or indicate that there are no capabilities associated with the specified binary. Package management tools like apt-get usually do not require additional capabilities beyond what is provided by traditional file permissions. Here's an example of the possible output:
 
 ```bash 
 /usr/bin/apt-get =
 ```
 
 In this case, the absence of any capabilities is indicated by the equal sign (=) with no capabilities listed. This means that the apt-get binary relies solely on the standard file permissions for its execution and does not have any additional capabilities assigned. 
 
 Keep in mind that the output may vary depending on the specific configuration and security policies of your Linux system. If there are no capabilities associated with the binary, it's a good indication that the binary operates within the confines of traditional file permissions.
 
## Assigning Capabilities to apt-get
In certain scenarios, you might find the need to assign specific capabilities to system binaries to achieve fine-grained control over their privileges. Let's explore how you can set capabilities for the apt-get binary, allowing it to perform certain actions with elevated privileges.

### Step 1: Identify the Capability
Before assigning capabilities, determine which capability is required for the desired functionality. For instance, if apt-get needs to perform raw network operations, you might consider using CAP_NET_RAW but there are more capabilities. Here are some of them:

- CAP_DAC_READ_SEARCH: Bypass file read permission checks and directory read and execute permission checks.
- CAP_DAC_OVERRIDE: Bypass file read, write, and execute permission checks.
- CAP_SYS_ADMIN: Perform various system administration operations.
- CAP_SYS_RESOURCE: Override resource limits.
- CAP_SYS_TIME: Set system clock.
- CAP_SYS_NICE: Raise process nice value.
- CAP_SYS_PTRACE: Trace arbitrary processes using ptrace.
- CAP_SETUID: Make arbitrary manipulations of process UIDs.
- CAP_SETGID: Make arbitrary manipulations of process GIDs.
- CAP_CHOWN: Make arbitrary manipulations of file UIDs and GIDs.

### Step 2: Assign Capabilities
```bash
$ sudo setcap cap_net_raw+ep /usr/bin/apt-get
```
This command assigns the CAP_NET_RAW capability to apt-get. The +ep ensures that the capability is added to both the effective and permitted sets. This means that during the execution of apt-get, it will have the specified capability available.


### Step 3: Verify the Assignment
```bash
$ getcap /usr/bin/apt-get
```
Use this command to confirm that the capability has been successfully assigned to apt-get. The output should display the assigned capability, indicating that apt-get now possesses this capability.

### Step 4: Execute apt-get with Elevated Capabilities
Now, when you run apt-get, it will have the assigned capability available for use. Keep in mind that the execution permission for the binary must also be granted to the user running it.

```bash
$ sudo apt-get update
```
## Important Considerations:
- Security Implications: Modifying capabilities of system binaries should be approached with caution due to potential security implications. Only assign capabilities that are necessary for the intended functionality.

- File Permissions: Ensure that the file permissions of the binary allow execution by the desired user. You can adjust permissions using chmod if needed.

- By following these steps, you can tailor the capabilities of apt-get to meet specific requirements while maintaining a controlled and secure environment. Always document and thoroughly understand the implications of such changes to ensure the integrity of your system.

In conclusion, delving into the realm of Linux capabilities reveals a powerful tool for refining the security posture of your system. While traditional file permissions offer a foundational layer of control, capabilities provide a nuanced and flexible approach to privilege management.

Through the exploration of commands like getcap and setcap, we've uncovered the intricacies of capabilities, understanding how they extend the conventional permissions model. Assigning capabilities to binaries, such as apt-get, allows for tailored privileges, offering a balance between functionality and security.

It's crucial to approach capability modifications with caution, considering the potential security implications. Always identify the specific capabilities required for your applications and adhere to the principle of least privilege.

In the ever-evolving landscape of Linux security, capabilities stand as a dynamic tool, providing administrators and users alike with the means to finely tune access controls. As you navigate the intricate web of permissions and capabilities, an informed and strategic approach will empower you to create a robust and secure Linux environment.










 
 
 
 
 
 
 
 
 
 
 
 
 
 