---

layout: page
title: Resizing an Instance (Non-live)
featured: False
weight: 12
tags: [getting started, instances, resize, migration]
dateAdded: May 23, 2016
author: Jesse Keating, Sina Sinadeghi
editor: Leslie Lundquist

---

### How Can I Resize my Instances?

**Feature:** Instance Resize (up)

**Description:** Starting with the 1.3 Release of Blue Box Cloud, we have supported instance resizing (up). The resize operation is not live: it _does_ stop and re-start the instance. A resize can happen on the same host, which is helpful in small environments.

Instance resizing lets an instance change its flavor to a larger flavor. An instance can be resized to a larger flavor **only**. This change incurs a small amount of interruption while the instance is stopped, migrated to a larger-sized flavor, and started again. This resize may also relocate the instance to a new hypervisor.  

Resizing large instances can be tricky, because it can cause failure if there isn’t enough disk space for the new disk image. 
**Note:** There may be a short delay until networking resumes once an instance has been resized or migrated and is in `VERIFY_RESIZE` state.

For more information, please see http://docs.openstack.org/user-guide/enduser/cli_change_the_size_of_your_server.html for documentation on how to perform a resize.

### Troubleshooting

If your resize operation takes a very long time for large instances (for example, some logs have shown an operation starting at 09:30 and the authentication error appearing at 14:41), the authentication token used by Nova to complete this operation could actually expire during this period.

It's very likely that if you attempt to initiate a new resize the same error will occur again. In that case, we suggest a different approach. If your cluster has block storage enabled, we suggest the following steps:

1.  Shut down the target VM.
2.  Take a snapshot of the target VM with `nova image-create`.
3.  Make a volume from that snapshot using `cinder create --image-id <SNAPSHOT UUID>`.
4.  Delete the snapshot with OpenStack `image delete` or `glance image-delete`.
5.  Boot a new VM from the volume with `nova boot --boot-volume`.

If you want to retain the same IP addresses you can:
 1. Get the port UUID of the volume booted VM using:
    
    `neutron port-list | grep <IP of new VM>` (the first column is the port UUID).
    
 2. Remove the port from the old VM using `nova interface-detach`

 3.  Update the port on the new VM to use the old VM IP with:
    `neutron port-update --fixed-ip ip_address=<old VM IP> <port UUID>`

Once you're happy with this switch you can clean up the old VM with `nova delete`.

This process is more convoluted than you probably prefer. However, switching to block storage as your guest backend comes with multiple benefits:

 * Flavor resize will happen much quicker because the disk resize is skipped.
 * The migration to a new hypervisor is very fast using block storage (the new hypervisor simply connects to the block storage cluster).
 * You can extend the disk of your VM on the fly with `cinder extend` command.
