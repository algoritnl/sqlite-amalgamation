[User Guide](User-Guide.md) | [Configuration Options](Configuration-Options.md) | [Step-by-Step Guide](Step-by-Step-Guide.md)
--------------------------- | ------------------------------------------------- | -------------------------------------------

# Configuration Options

## Table of Contents

- [General Options](#general-options)
- [Library Options](#library-options)
  - [Recommended Options](#recommended-options)
  - [Other Options](#other-options)
- [Interactive Shell Options](#interactive-shell-options)

## General Options

- `CMAKE_BUILD_TYPE:STRING`:
  value in (**Debug**|**Release**|**MinSizeRel**|**RelWithDebInfo**), default = none.  
  If the library is built for configuration **Debug** then it gets a postfix
  **-dbg**.

- `BUILD_SHARED_LIBS:BOOL`:
  value is a **boolean**, default = **OFF**.  
  If this option is set to **ON** then shared libraries will be built instead
  of static library.

## Library Options

### Recommended Options

- `SQLITE_THREADSAFE:STRING`=**0**:
  value in [**0**-**2**], default = **1**.  
  Supported thread models: **0** = single-threaded; **1** = serialized;
  **2** = multi-threaded.  
  The recommendation is only applicable if SQLite is never used by more than a
  single thread at a time, even if each thread has its own database connection.  
  _Note: The library name gets an infix **-st** if the single-threaded model
  is selected and **-mt** for the multi-threaded model._

- `SQLITE_DEFAULT_MEMSTATUS:STRING`=**0**:
  value in [**0**-**1**], default = **1**.  
  This option is used to determine whether or not the features enabled and
  disabled using the **SQL_CONFIG_MEMSTATUS** argument to **sqlite3_config()**
  are available by default.

- `SQLITE_DEFAULT_SYNCHRONOUS:STRING`=**2**:
  value in [**0**-**3**], default = **2**.  
  This option determines the default value of the **PRAGMA synchronous**
  setting. The default and recommended setting is **2** (**FULL**).

- `SQLITE_DEFAULT_WAL_SYNCHRONOUS:STRING`=**1**:
  value in (**-1**|[**0**-**3**]), default = **-1**.  
  This option determines the default value of the **PRAGMA synchronous** setting
  for database files that open in **WAL mode**. The default setting is **-1**
  (follow `SQLITE_DEFAULT_SYNCHRONOUS`). The recommended value is **1**
  (**NORMAL**). For more information, see
  [**PRAGMA** *schema*.**synchronous**](https://sqlite.org/pragma.html#pragma_synchronous)
  on the SQLite website.

- `SQLITE_DQS:STRING`=**0**:
  value in [**0**-**3**], default = **3**  
  Controls the **Double-Quoted String** literals misfeature in **DDL** (Data
  Definition Language) and **DML** (Data Manipulation Language):
  - 0: Disable **DQS** in **DDL**, disable **DQS** in **DML** (recommended).
  - 1: Disable **DQS** in **DDL**, enable **DQS** in **DML**.
  - 2: Enable **DQS** in **DDL**, disable **DQS** in **DML**.
  - 3: Enable **DQS** in **DDL**, enable **DQS** in **DML** (default).

- `SQLITE_LIKE_DOESNT_MATCH_BLOBS:BOOL`=**ON**:
  value is a **boolean**, default = **OFF**.  
  When this option is set, it means that the **LIKE** and **GLOB** operators
  always **FALSE** if either operand is a BLOB. That simplifies the
  implementation of the LIKE optimization and allows queries that use the
  LIKE optimization to run faster.

- `SQLITE_MAX_EXPR_DEPTH:STRING`=**0**:
  value is an **integer**, default = **-1**.  
  This option determines the maximum expression tree depth. If the value is
  **0**, then no limit is enforced. If the option is set to **-1** then it is
  not passed on to the compiler and the implementation uses its own default
  value, which is **1000** currently. The recommended value is **0** (no limit).

- `SQLITE_ENABLE_COLUMN_METADATA:BOOL`=**OFF**:
  value is a **boolean**, default = **OFF**.  
  This enables some extra APIs that are required by some common systems,
  including Ruby-on-Rails.  
  _This option is mutually exclusive with `SQLITE_OMIT_DECLTYPE`._

- `SQLITE_OMIT_DECLTYPE:BOOL`=**ON**:
  value is a **boolean**, default = **OFF**.  
  By omitting the (seldom-needed) ability to return the declared type of
  columns from the result set of query, prepared statements can be made to
  consume less memory.  
  _This option is mutually exclusive with `SQLITE_ENABLE_COLUMN_METADATA`._

- `SQLITE_OMIT_DEPRECATED:BOOL`=**ON**:
  value is a **boolean**, default = **OFF**.  
  Omitting deprecated interfaces and features will not help SQLite to run any
  faster. It willreduce the library footprint, however. And it is the right
  thing to do.

- `SQLITE_USE_ALLOCA:BOOL`=**ON**:
  value is a **boolean**, default = **OFF**.  
  If this option is enabled, then the **alloca()** memory allocator will be used
  in a few situations where it is appropriate. This results in a slightly
  smaller and faster binary. It only works, of course, on systems that support
  **alloca()**.

### Other Options

- `SQLITE_ENABLE_DBSTAT_VTAB:BOOL`=**OFF**:
  value is a **boolean**.  
  This option enables the dbstat virtual table.

- `SQLITE_ENABLE_EXPLAIN_COMMENTS:BOOL`=**OFF**:
  value is a **boolean**.  
  This option adds extra logic to SQLite that inserts comment text into the
  output of **EXPLAIN**.

- `SQLITE_ENABLE_FTS3:BOOL`=**OFF**:
  value is a **boolean**.  
  Enable version 3 of the full-text search engine.

- `SQLITE_ENABLE_FTS3_PARENTHESIS:BOOL`=**OFF**:
  value is a **boolean**.  
  This option modifies the query pattern parser in FTS3 such that it supports
  operators **AND** and **NOT** (in addition to the usual **OR** and **NEAR**)
  and also allows query expressions to contain nested parentheses.

- `SQLITE_ENABLE_FTS4:BOOL`=**OFF**:
  value is a **boolean**.  
  Enable versions 3 and 4 of the full-text search engine.

- `SQLITE_ENABLE_FTS5:BOOL`=**OFF**:
  value is a **boolean**.  
  Enable version 5 of the full-text search engine.

- `SQLITE_ENABLE_GEOPOLY:BOOL`=**OFF**:
  value is a **boolean**.  
  When this option is defined, the Geopoly extension is included in the build.

- `SQLITE_ENABLE_ICU:BOOL`=**OFF**:
  value is a **boolean**.  
  This option causes the International Components for Unicode or "ICU" extension
  to SQLite to be added to the build.

- `SQLITE_ENABLE_JSON1:BOOL`=**OFF**:
  value is a **boolean**.  
  When this option is defined the JSON SQL functions are added to the build
  automatically.

- `SQLITE_ENABLE_RBU:BOOL`=**OFF**:
  value is a **boolean**.  
  Enable the code that implements the RBU extension.

- `SQLITE_ENABLE_RTREE:BOOL`=**OFF**:
  value is a **boolean**.  
  This option causes SQLite to include support for the R*Tree index extension.

- `SQLITE_ENABLE_STMTVTAB:BOOL`=**OFF**:
  value is a **boolean**.  
  This compile-time option enables the SQLITE_STMT virtual table logic.

- `SQLITE_OMIT_LOAD_EXTENSION:BOOL`=**OFF**:
  value is a **boolean**, recommended = **OFF**.  
  This option omits the entire extension loading mechanism from SQLite,
  including **sqlite3_enable_load_extension()** and
  **sqlite3_load_extension()** interfaces.

- `SQLITE_OMIT_PROGRESS_CALLBACK:BOOL`=**OFF**:
  value is a **boolean**.  
  This option may be defined to omit the capability to issue "progress"
  callbacks during long-running SQL statements. The
  **sqlite3_progress_handler()** API function is not present in the library.

- `SQLITE_OMIT_SHARED_CACHE:BOOL`=**OFF**:
  value is a **boolean**.  
  This option builds SQLite without support for shared-cache mode.
  The **sqlite3_enable_shared_cache()** is omitted along with a fair amount of
  logic within the B-Tree subsystem associated with shared cache management.

## Interactive Shell Options

- `SQLITE_BUILD_SHELL:BOOL`=**ON**:
  value is a **boolean**, recommended = **ON**.  
  This option controls whether or not to build the interactive shell.

- `SQLITE_HAVE_ZLIB:BOOL`=**OFF**:
  value is a **boolean**, recommended = **ON**.  
  This option is necessary for the compression and decompression functions that
  are part of SQL Archive support in the command-line shell.
