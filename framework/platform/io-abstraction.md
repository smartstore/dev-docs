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
