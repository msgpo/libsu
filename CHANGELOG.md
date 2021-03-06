# 2.0.2

## Fixes
- Return proper list in `Shell.Result.getErr()`

# 2.0.1

## Fixes / Improvements
- Calling `submit` in `PendingJob` used to end up calling the overridden `exec` instead of `exec` in `JobImpl`. Proxy through a private method in `JobImpl` to prevent changed behavior of `JobImpl` subclasses.
- Even though the parameters of `Shell.Job.add(...)` is labeled `@NonNull`, it is still possible that developers still pass in `null` and cause NPE. Check `null` before proceed any further.
- Fix a bug that constructed shell instances weren't cached in the container when created using fallback methods (e.g. no root -> fallback to non-root shell). This would cause infinite loop when no root is available.
- Prevent `IllegalStateException` if the user provide a filter accepting `.` or `..` in `SuFile.list()` family methods.

# 2.0.0

Massive API improvements! Nearly all APIs in 1.x.x versions are deprecated. A compatibility layer is created to make the migration easier for existing developers: your existing code will work with `libsu` 2.0.0 without any modifications.

However, the compatibility layer **will** be removed in a future update, please do update your code to utilize the new APIs since it is much cleaner and more flexible!

### Incompatible API Changes
- Remove `readFully(InputStream, byte[])` and `readFully(InputStream, byte[], int, int)` from `ShellUtils`.

### Fixes / Improvements / Behavior Changes
- Global shell is now set before `Shell.Initializer` runs: this means you can now use high level APIs and root I/O classes in your initializer
- No more `null` lists will be returned from `libsu`: all methods will return empty lists if no output is available
- `SuFile.list()` family methods shall return hidden files now (filenames starting with `'.'`) (also fix #15)
- `SuFile` will use the tool `stat` in more methods for consistent results (fix #11)
- No longer uses raw threads and `AsyncTask.THREAD_POOL_EXECUTOR` for running code in background threads, switch to an internal `ExecutorService`
- `Shell.GetShellCallback.onShell(Shell)` will run on the main thread
- Add Proguard rules to strip out logging code when minify and optimization (`proguard-android-optimize.txt`) is enabled
- Lower `minSdkVersion` to 9

### API 2.0 Migration
Note: The following list is **ONLY** to point out a brief direction for migration, check the examples and documentation to know the details!

- All configuration methods are moved to the nested class `Shell.Config`
- Everything in `Shell.Sync` and `Shell.Async` is deprecated. Check the new APIs: `Shell.su(...)` and `Shell.sh(...)`
- New methods:
  - `Shell.getCachedShell()`: A static method to obtain the cached global root shell from the container, or return `null` of no active shell exists
  - `Shell.isRoot()`: Check root access of a root instance (Note: static method `Shell.rootAccess()` is used to check the global shell)
  - `Shell.Config.newContainer()`: construct a pre-configured shell container, used to simplify registering a container
- New classes:
  - `Shell.Job`: represent a job that outputs to a single result object. You can chain additional operations, assign output destination, execute or submit the job.
  - `Shell.Result`: stores the result of a `Shell.Job`. Includes STDOUT/STDERR, and something not existing in 1.x.x: return code
  - `Shell.ResultCallback`: a callback interface to get the result when you submit a `Shell.Job` to background threads
- For `Shell.Initializer`: deprecate `onShellInit(...)` and `onRootShellInit(....)`; use `onInit(Context, Shell)` instead: manually detect the shell status and handle Exceptions
- Move `Shell.ContainerApp` out of `Shell` to a separate class `ContainerApp`: easier to directly assign in `AndroidManifest.xml`
