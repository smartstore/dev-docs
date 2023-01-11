# 🥚 IO abstraction

* .NET Core already provides file system abstraction through the `IFileProvider` interface
* However, it has a very simple signature and is only intended for reading files
* Therefore Smartstore provides the [IFileSystem](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/IO/IFileSystem.cs) interface, along with [IFile](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/IO/IFile.cs) and [IDirectory](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/IO/IDirectory.cs).
  * Their signatures are comprehensive and more oriented towards the classes in `System.IO`, which you should be more familiar with
  * `IFileSystem` also implements `IFileProvider`, and `IFile` and `IDirectory` both implement `IFileInfo`.
    * So think of it as a kind of extension to `IFileProvider`
    * All Smartstore IO interfaces have async methods (`IFileProvider` does not)
    * Because `IFileSystem` can also represent cloud storage providers (like Azure BLOB storage) or virtualize database-driven file storage
* The virtual file system uses forward slash (`/`) as the path delimiter, and has no concept of volumes or drives
* All paths are specified and returned as relative to the root of the virtual file system.
* Absolute paths using a leading slash or leading period, and parent traversal using `../`, are not supported
* [LocalFileSystem](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/IO/LocalFileSystem/LocalFileSystem.cs) is the default implementation. Looks up files using the local disk file system.
  * The async methods just delegate to the sync methods
* _BASIC EXAMPLES_
  * Enumerating directory contents
  * Reading a file's text content
  * Writing a file
  * Checking for unique file names

## Adapters & decorators

### ExpandedFileSystem

* A decorator that takes a path prefix and an inner `IFileSystem` instance whose root path should be expanded with a prefix.
* _SAMPLE_

### CompositeFileSystem

* Looks up files using a collection of `IFileSystem` instances
* _SAMPLE_

## Special utilities

### DirectoryHasher

* Creates a hash for the contents of a directory, either deep or flat
* Can load and persist the hash code for comparisons with any previous state
  * Directory is: _App\_Data/Tenants/Default/Hash_
* _SAMPLE_

