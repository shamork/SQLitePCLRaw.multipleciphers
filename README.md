# SQLitePCLRaw.multipleciphers
Microsoft.Data.SQLite.Core provider for sqlite mulitple ciphers

# steps

doc https://docs.microsoft.com/en-us/dotnet/standard/data/sqlite/custom-versions?tabs=netcore-cli

1. Add nuget packages

```
dotnet add package Microsoft.Data.Sqlite.Core
dotnet add package SQLitePCLRaw.core
dotnet add SQLitePCLRaw.provider.dynamic
```

2. Copy & paste code from doc

```
void Main()
{
  SQLite3Provider_dynamic_cdecl
	.Setup("sqlite3", new NativeLibraryAdapter(@"sqlite3mc"+(Environment.Is64BitProcess?"_x64.dll":".dll")));
	SQLitePCL.raw.SetProvider(new SQLite3Provider_dynamic_cdecl());
	using var connection = new SqliteConnection(@"Data Source=sqlmc.db");
	Console.WriteLine($"System SQLite version: {connection.ServerVersion}");
  
	const string setKey = "PRAGMA key = '';";//new db without pass
	connection.Execute(setKey);  
  
	connection.Execute("PRAGMA cipher = 'aes256cbc';");//set cipher
	const string resetKey = "PRAGMA rekey = '12345678';";//reset key
	connection.Execute(resetKey);
	connection.Close();
  //now the db is ecrypted
}
class NativeLibraryAdapter : IGetFunctionPointer
{
    readonly IntPtr _library;

    public NativeLibraryAdapter(string name)
        => _library = NativeLibrary.Load(name);

    public IntPtr GetFunctionPointer(string name)
        => NativeLibrary.TryGetExport(_library, name, out var address)
            ? address
            : IntPtr.Zero;
}
```

3. Add sqlitemc.dll/sqlitemc_x64.dll to project
Down load zip and get the dll from https://github.com/utelle/SQLite3MultipleCiphers/releases
You can use dll from then following zips as your wish

```
sqlite3mc-x.y.z-sqlite-x.y.z-icu-win32.zip
sqlite3mc-x.y.z-sqlite-x.y.z-icu-win64.zip
sqlite3mc-x.y.z-sqlite-x.y.z-win32.zip
sqlite3mc-x.y.z-sqlite-x.y.z-win64.zip
```

4. Set sqlitemc.dll/sqlitemc_x64.dll ```Copy To Output Folder``` property: ```copy if newer``` or ```copy allways```

5. Build the project and run. Enjoy!

