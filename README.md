# SQL Server Management Studio(SSMS) saved password dumper

**by using [DPAPI](https://msdn.microsoft.com/en-us/library/ms995355.aspx),program MUST be run on USER CONTEXT.**

(use *impersonate* to do it.)

### Bulid

    #for .net 2.0
    %systemroot%\microsoft.net\framework\v2.0.50727\csc.exe SSMSPwd.cs
    #for .net 4.0
    %systemroot%\microsoft.net\framework\v4.0.30319\csc.exe /out:SSMSPwd40.exe SSMSPwd.cs

### Usage

	SSMSPwd [-f file] [-p path] [-all]
       -f: decrypt from specified file
       -p: path of SSMS installation
       -a: dump all saved info(only dump password information default)

### Remarks

SSMS save password to a binary file using [BinaryFormatter](https://msdn.microsoft.com/en-us/library/system.runtime.serialization.formatters.binary.binaryformatter.aspx),and the type was defined on **private assembly** in the installation directory.

This file saved in `%appdata%\Microsoft\Microsoft SQL Server\(VERSION)\Tools\Shell` on SSMS2005 or SSMS2008, `%appdata%\Microsoft\Microsoft SQL Server\(VERSION)\Tools\ShellSEM` on **Express Version**,`%appdata%\Microsoft\SQL Server Management Studio` on others.its named `mru.dat` on SSMS2005,`SqlStudio.bin` on SSMS2008 to last release.

SSMS2005 saved a `IDirectory` named `stringTable` in file,the key like this:

	8c91a03d-f9b4-46c0-a305-b5dcc79ff907@192.168.223.250\SQLEXPRESS@1@sa@Password
	8c91a03d-f9b4-46c0-a305-b5dcc79ff907@192.168.223.250\SQLEXPRESS@1@sa@ET

Split by `@`,we can get `instance`,`user`.

If the key ends with `Password`,the value will be encrypted password using [DPAPI](https://msdn.microsoft.com/en-us/library/ms995355.aspx) and `Base64Encode`.

Other versions,binary file saved a big tree like:

    SqlStudio
    └─SSMS
      └─ConnectionOptions
          ├─ServerTypes-1
          │  ├─Servers-1
          │  │  │  Instance
          │  │  │
          │  │  ├─Connections-1
          │  │  │      Password
          │  │  │      UserName
          │  │  │
          │  │  └─Connections-2
          │  │          Password
          │  │          UserName
          │  │
          │  └─Servers-2
          │      │  Instance
          │      │
          │      └─Connections-1
          │              Password
          │              UserName
          │
          └─ServerTypes-2
              ├─Servers-1
              │  │  Instance
              │  │
              │  ├─Connections-1
              │  │      Password
              │  │      UserName
              │  │
              │  └─Connections-2
              │          Password
              │          UserName
              │
              └─Servers-2
                  │  Instance
                  │
                  └─Connections-1
                          Password
                          UserName

We can get `IDirectory` and `IEnumerable` on nodes,the pseudocode was:

    foreach ServerType in SqlStudio['SSMS']['ConnectionOptions']['ServerTypes']
    {
      foreach Server in ServerType['Servers']
      {
        print Server.Instance
        foreach Connection in Server['Connections']
        {
          print Connection.UserName,Connection.Password
        }
      }
    }
The `Password` use [DPAPI](https://msdn.microsoft.com/en-us/library/ms995355.aspx) and `Base64Encode` too.Using [ProtectedData::Uprotect](https://msdn.microsoft.com/en-us/library/xh68ketz(v=vs.110).aspx) to decrypt it.

DONOT forget,**run program on USER CONTEXT**.