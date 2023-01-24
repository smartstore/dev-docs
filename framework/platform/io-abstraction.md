# 🐣 IO abstraction

## Overview

The `FileProvider` interface in .NET Core already provides file system abstraction, but it is only intended for reading files and has a very simple signature. For this reason, Smartstore provides the [IFileSystem](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/IO/IFileSystem.cs) interface, along with [IFile](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/IO/IFile.cs) and [IDirectory](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/IO/IDirectory.cs). They offer richer signatures and are oriented towards the classes in `System.IO`, with which you should be more familiar.

`IFileSystem` implements `IFileProvider` and both `IFile` and `IDirectory` implement `IFileInfo`. Because of this, they act as an extension of `IFileProvider`, so all Smartstore IO interfaces have async methods that `IFileProvider` doesn’t provide. This is necessary, because `IFileSystem` can also represent cloud storage providers (such as Azure BLOB storage) and virtualize database-driven file storage.

The virtual file system uses the forward slash (`/`) as path delimiter and has no concept of volumes or drives. All paths are specified and returned as relative paths to the root of the virtual file system. Absolute paths that use a leading slash (`/`), dot (`.`) or parent traversal (`../`) are not supported.

The default implementation of `IFileProvider` is [LocalFileSystem](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/IO/LocalFileSystem/LocalFileSystem.cs). It looks for files using the local disk file system.

* The async methods just delegate to the sync methods.

_BASIC EXAMPLES_

* Enumerating directory contents
* Reading a file's text content
* Writing a file
* Checking for unique file names

## Adapters & decorators

### ExpandedFileSystem

The [ExpandedFileSystem](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/IO/ExpandedFileSystem.cs) is a decorator that takes a path prefix and an inner `IFileSystem` instance. Its root path is expanded with the specified prefix.

* _SAMPLE_

### CompositeFileSystem

The [CompositeFileSystem](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/IO/CompositeFileSystem.cs) is an adapter that looks up files using a collection of `IFileSystem` instances. Because it implements the `IFileSystem` interface, it can be used in the same way.

* _SAMPLE_

## Special utilities

### DirectoryHasher

The [DirectoryHasher](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/IO/DirectoryHasher.cs) creates a hash of the contents of a directory. It can load and persist the hash code for comparison with any previous state, that are saved in **App\_Data/Tenants/Default/Hash**. By default, it scans flat, but you can choose to scan deep as well.

* _SAMPLE_

