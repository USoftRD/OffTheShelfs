# OTS_LOGGER 2.0 — notes for reviewers

OTS_LOGGER 2.0 replaces the 1.x logger (one hard-coded file, no levels, throws into the transaction when
the file cannot be written). It is built and tested on USoft 11.2.1 (SQL Server 2022, .NET 10).

## What changed

| | 1.x | 2.0 |
|---|---|---|
| Level | none (everything written) | OFF, ERROR, WARNING, INFORMATION, DEBUG; at OFF nothing is written |
| Default file | hard-coded `C:\temp\mylog.log` | `OTS_LOGGER.log` in the USoft log folder, `RulesEngine.GetProperty('USoftLogDir')` (e.g. `C:\USoft\USD11-Logs\USoft_logs\`) |
| Where the level/path is set | only through the `...PATH` methods | table `OTS_LOGGER_SETTING` (wins) and/or a configuration file, re-read every 5 s; the configuration file is created on first use |
| Line | `2024-11-22 12:08:51 [Information] - msg` | `2026-10-03 12:00:41.440 [DEBUG] APP user=U session=<pid>.<engine> tx=<pid>.<n> - msg` |
| Failure to write | exception, the statement/transaction fails | never throws; reported in `%ProgramData%\USoft\OTS_LOGGER\OTS_LOGGER_fallback.log` (max once a minute per cause) |
| Several processes | in-process `Mutex()` only (unnamed) | named mutex `Global\OTS_LOGGER_<hash of path>` (Everyone may use it) + append-mode single writes |
| File size | grows forever | rotates at `max_size_kb` (10240), keeps `max_files` (5): `OTS_LOGGER.1.log` ... |
| Methods | WRITEERROR/WARNING/INFORMATION(+PATH) | same names and signatures, plus WRITEDEBUG(+PATH), WRITE(level, msg), ISENABLED(level), GETCONFIGURATION(), RELOAD() |
| Component | stateless | stateful, Lifetime = Transaction (one instance per transaction gives the `tx` id); no transaction participation |
| Base class | none | `RulesEngine` (RDMIRulesEngine.dll, ASSEMBLYREFS set) to read `OTS_LOGGER_SETTING` in the calling engine |

Without any setting it logs at level INFORMATION, like 1.x, but to `OTS_LOGGER.log` in the USoft log folder
instead of `C:\temp\mylog.log`. **Breaking change:** whoever reads `C:\temp\mylog.log` must look in the new file,
or set `path=C:\temp\mylog.log` in the configuration file or the table. The calls keep working unchanged (the
line format is new).

- **USoft log folder:** `RulesEngine.GetProperty('USoftLogDir')`, asked once per process. It is the
  installation's `LogPath` (registry `HKLM\SOFTWARE\USoft\USoft112`) plus `USoft_logs\`, and it gives the same
  folder in a client, in runbatch and in the Rules Service. When it gives no folder, or no Rules Engine is
  available, the default is `%ProgramData%\USoft\OTS_LOGGER\OTS_LOGGER.log`, and the fallback file says why.
- **Configuration file on first use:** when `%ProgramData%\USoft\OTS_LOGGER\` holds no `*.config` and
  `%OTS_LOGGER_CONFIG%` names no file, the first call writes `OTS_LOGGER.config` there with every key and its
  current value, the log path written out in full. It is never overwritten. When the Rules Service (LocalSystem)
  creates it, only administrators can edit it (Users: read), which suits a machine-wide setting.
- **`{USOFTLOGDIR}`** in a configured path is replaced by that folder, e.g. `path={USOFTLOGDIR}{APP}.log`.

The package now also contains table `OTS_LOGGER_SETTING` (domains `OTS_LOGGER_SETTING_NAME` with allowed values,
`OTS_LOGGER_SETTING_VALUE`, constraint `OTS_LOGGER_SETTING_LEVEL`). After the import: check, Create Tables,
restart the Rules Service (a running Rules Service answers `Component "OTS_LOGGER" does not exist` for a new
component until it restarts).

## Design decisions

- **Database setting wins over the file** (user decision): a row in `OTS_LOGGER_SETTING` can be changed per
  environment without file access; the file serves environments without the table or before it is created.
  Keys are merged key by key: file first, then database.
- **File locations**: `%OTS_LOGGER_CONFIG%` (a file path), else `%ProgramData%\USoft\OTS_LOGGER\<APP>.config`,
  else `%ProgramData%\USoft\OTS_LOGGER\OTS_LOGGER.config`. Format `key=value`, `#` comments. Keys: `level`,
  `path` (`{APP}`, `{USOFTLOGDIR}` and `%VAR%` replaced), `max_size_kb` (0 = never rotate), `max_files`, `refresh_seconds`,
  `fallback_path`, `database` (`off` = do not read the table), `settings_table` (another table with the same two columns).
- **Reading the table from C#** goes through the calling Rules Engine (`RulesEngine.Query`). Two 11.2.1
  behaviours forced extra code:
  - a nested query that fails with an **RDBMS error** (table defined but not created) makes the *calling*
    statement return no rows, even though no exception reaches C# and even inside
    `RulesEngine.StartCatchingErrors`; so on SQL Server the logger first asks `OBJECT_ID(...)` (cannot fail)
    and skips a missing table;
  - a nested query with a **parse error** (table not in the model) shows an error in the client unless it runs
    inside `StartCatchingErrors('Yes')`/`StopCatchingErrors()`; with that it is silent and reported in the fallback file.
  After a failure the table is not read again for 60 s.
- `setUSoftOwnerUserRole(owner, user, role)` (private) is called by the 11.2.1 .NET proxy before each call and
  gives the application name and user; `getEngine()` gives the engine id.
- `string.GetHashCode()` differs per process in .NET, so the mutex name uses FNV-1a of the lower-case full path.

## Tests (USoft 11.2.1, projects CLDTEST_LOG / CLDTEST_LOG2)

- Fresh install (import, check, Create Tables, restart, smoke test) in 29 s; upgrade of an installed 1.x in place.
- A corrective constraint, a second corrective constraint and a SQL task in a job changing the same field in one
  transaction: the log shows every intermediate value (5000 -> 1000 -> 20 -> 30) with one `tx` id.
- Level switching in a running Rules Service (LocalSystem) over HTTP, no restart: DB DEBUG beats file OFF;
  DB row removed -> file OFF -> 0 lines; file WARNING -> INFORMATION not written, WARNING written; no file ->
  default INFORMATION; DB DEBUG again.
- Write failures: path on a missing drive, a log file locked exclusively by another process, settings table
  missing in the database or in the model: every transaction committed with the right data; fallback report written.
  A missing folder is created.
- Parallel writers: 4 clients x 500 lines + 2 runbatch jobs x 500 + Rules Service 2 x 100 (3200 lines, 7
  processes, two Windows accounts) and a .NET harness with 6 processes x 4 threads x 2000 lines (48 000 lines,
  38 722 process switches, also with rotation at 500 KB): every line whole, order per process kept.
- Cost: a call at level OFF costs ~7 microseconds in the client (5000 calls 50 ms vs 10-24 ms without the
  call); in a 2000-row update with three logging constraints the difference was within the noise of the
  test machine (run-to-run spread 30-50 %). In-process (harness) an OFF call costs 64 ns. A DEBUG line
  costs ~0.4 ms (open, append, close under the mutex).
- Default path and configuration file (project CLDTEST_LOGDIR): `USoftLogDir` is `C:\USoft\USD11-Logs\USoft_logs\`
  in usd.exe and in the Rules Service (LocalSystem, over HTTP), and both wrote to `OTS_LOGGER.log` there. The
  Rules Service's first call created `OTS_LOGGER.config` with the full path; a client then read it
  (`source=file`). `level=DEBUG`, edited in the file, was used and the file was not rewritten.
  `PATH = {USoftLogDir}{APP}_token.log` in the table gave `...\USoft_logs\CLDTEST_LOGDIR_token.log`.
- A build that asks for a property that does not exist: the calling statement still ran, the line went to
  `%ProgramData%\USoft\OTS_LOGGER\OTS_LOGGER.log`, and the fallback file says why.

## Not tested

Oracle (the `OBJECT_ID` pre-check is SQL Server only; a defined-but-not-created table on Oracle would make
the first logging statement per minute return no rows), disk full (simulated only by missing drive and
locked file), a non-administrator account opening the mutex created by the Rules Service (the mutex grants
Everyone; tested with an elevated user and LocalSystem), `USoftLogDir` on an installation without `LogPath`,
and two processes creating the configuration file at the same moment (the file is opened create-new, so one wins
and the other keeps the built-in defaults until its next refresh).

## How the files were made

`OTS_LOGGER.xml` was generated from the C# source (methods and parameters by the Definer's own
`DOTNETCOMPILER.GENERATEMETHODS_LANG_APPDOMAIN`), and the `src` files were written by a USoft 11.2.1 Definer
commit after importing it (`XML.Import` with `IgnoreGUK = no`, so the G_U_Ks of the file are kept; the
component keeps the 1.x G_U_K). CREATED_BY was set to `USD_COMPONENTS1100` like the other `src` files.
The default-path change was copied into `src/COMPONENTS/OTS_LOGGER.XML` and `src/DOMAINS/OTS_LOGGER_SETTING_NAME.XML`
by hand: the same source as `OTS_LOGGER.xml`.

## Behaviour compared with 1.x on three points raised in review

| Point | 1.x | 2.0 (tested) |
|---|---|---|
| The file cannot be written | throws: the manipulation is refused | the transaction goes on unchanged; fallback report |
| Logging constraint without `OLD()` (non-transitional) | logs once, at commit, the final value | same (Rules Engine behaviour, not the logger's): use `OLD(col)` in the logging constraint to get a line at every store |
| Rolled-back transaction | its lines stay in the file | same: the file is not transactional; the `tx` id shows which lines belong together, a commit-time constraint of a rolled-back transaction writes nothing |

A log call whose argument fails (on SQL Server `'text' || NUMBER`, sent as `+`) still fails the statement:
the database evaluates the argument before the logger is called. Use `NUMBERTOCHAR()` and `NVL()`.
