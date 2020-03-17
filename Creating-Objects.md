# Creating Objects

The TPM allows one to create objects. These objects can be keys or small amounts of free form data. This tutorial shows you the first steps to using the TPM: creating keys and objects.

## Hierarchies

Before we create an object, it's important to understand the concept of hierarchies. The TPM 2.0 has 4 hierarchies, they are:
1. platform - used by firmware and the OS.
2. owner - used by the owner (this is the one we will be using).
3. endorsement - used for privacy sensitive keys, these keys are known to come from a valid TPM and platform.
4. null - reset on every boot, good for transient session keys.

Each hierarchy is essentially a static seed value, with the exception of the null hierarchy, which is a new seed on every boot.

Every hierarchy has its own authorization value.

## Primary Objects

Under a hierarchy, one can create N primary objects. These objects do not persist across reboots by default. However, a primary object created with the same inputs under a given hierarchy will produce the same exact key, with the exception of the null hierarchy (the seed changes each boot). Primary objects have special attributes and thus are not intended for general purpose application. To add a primary object to a hierarchy requires the hierarchy authorization value.

## Objects

Under a primary object, one can create N objects. These objects can be created to form key hierarchies. Thus, an object could have P parents, where P > 1 terminating at the primary object.

# Getting Started

We assume you have a working installation of tss, tools, a tpm simulator and abrmd. If you don't, see: [Getting Started](https://github.com/tpm2-software/tpm2-tools/wiki/Getting-Started).

The below commands were tested on the 4.1.1 release of the tpm2-tools.

## Creating a Primary Key

Let's create a primary key under the owner hierarchy:
```sh
tpm2_createprimary -c primary.ctx
```
That's all there is to it. The tool, `tpm2_createprimary` defaults to creating the object under the owner hierarchy. One might ask, "Why didn't we have to "authorize" to the owner hierarchy?" The tool itself defaults to an empty password, which is the default for the owner hierarchy when using the TPM simulator.

## Creating an Object

Now that we created a primary key, lets create two children keys under the primary key, where one of the keys is suitable for becoming a parent key.

### Step 1 - Creating a key
Create the first child, which will not be suitable for having children:
```sh
tpm2_create -C primary.ctx -Grsa2048 -u key.pub -r key.priv
```
That tool outputs data to stdout in a YAML format. Study the output, even though all the fields and data may not make sense or be pertinent at this point in time, it may be useful in future TPM endeavors.

### Step 2 - Creating a parent key
Create the second child, which will be suitable for having children keys, ie a parent key. This will require us to explicitly select the object attributes to achieve this goal. Object attributes are flags that control the behavior of objects. A parent object needs to have the restricted attribute set. Restricted keys are limited to what and how they can be used in operations.

```sh
tpm2_create -C primary.ctx -Grsa2048 -u key2.pub -r key2.priv -a "restricted|decrypt|fixedtpm|fixedparent|sensitivedataorigin|userwithauth"
```

### Step 3 - Load a key
In order to create a child under the key created in step 2, we need to load that key; an object cannot be used without being loaded in the TPM. We will use the load command to achieve this, like so:
```sh
tpm2_load -C primary.ctx -u key2.pub -r key2.priv -c key2.ctx
```
Notice the output, a handle. If you were writing an application that didn't exit or using the TPM without a resource manager, that handle would be valid for use subsequently. However, we're using a resource manager (RM), and the RM flushes transient objects when the tool exits and tpm2_load loads objects into non-persistent transient memory. The tools support using context blobs where possible to overcome this condition. In this case, the tpm2_load command takes a -c option to output the saved result of a load to disk. We can use that later.

### Step 4 - Create a leaf key under a parent key
Now that we have the parent key loaded in Step 3, we can create the last key. For this key, well make an AES key.
```sh
tpm2_create -C key2.ctx -Gaes -u key3.pub -r key3.priv
```
### Step 5 - Create an object under a parent key
You don't have to create keys, you can create free-form objects for "sealing" small amounts of data to the TPM.
For instance, I'll seal a string from stdin to the TPM:

```sh
echo "my sealed data" | tpm2_create -C key2.ctx -i- -u key4.pub -r key4.priv
```
This data can be recovered with tpm2_unseal.
```sh
tpm2_load -C key2.ctx -u key4.pub -r key4.priv -c key4.ctx
tpm2_unseal -c key4.ctx
my sealed data
```
# Conclusion

This tutorial shows you how to create various objects, load objects into the TPM, and as a bonus how to seal and unseal data. To keep things simple, authorization values were omitted. In real cases, you want to use passwords at a minimum. Note that password based authorizations are sent to the TPM in the clear, thus anyone able to snoop on this data will have authorization to the object. Other, more complex authorization schemes will be covered in subsequent tutorials.
