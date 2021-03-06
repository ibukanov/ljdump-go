# ljdumpgo
LiveJournal backup utility in Go

This is a port of the [ljdump.py](https://github.com/ghewgill/ljdump) utility to Go to backup LiveJournal user and community posts and comments.

The software should be considered alpha-version quality and is used with extreme caution. In particular, a bug in the utility may cause excessive number of requests sent to LiveJournal and blocking of the IP address. A bug can also overwrite or damage already archived files so if you have a big archive downloaded with ljdump.py, backup it before running the utility.

## Invocation
Create a directory where to store the archive and create a config file named `ljdump.config` there as described in the [sample file](ljdump.config.sample). Then run compiled ljdumpgo binary from that directory.

To specify the password separately from the config file create a file with the password on its first line and then either add `<passwordFile>` to `ljdump.config` or set the environment variable `LJDUMP_PASSWORD_FILE` with the location of the password file.

Alternatively, configuration can be given as command line options. Those take precedence over configuration from `ljdump.config`.
```
...$ ljdumpgo -h
Usage: ljdumpgo [OPTION]...

Option summary:
  -h    shorthand for -help 
  -help
        print usage on stdout and exit
  -j journal
        shorthand for -journal journal
  -journal journal
        add journal to the list of journals to archive. If none are given, use LJ username
  -p path
        shorthand for -password-file path
  -password-file path
        path to file with LJ user password, use '-' to read from stdin (password will be echoed)
  -s server
        shorthand for -server server (default "http://livejournal.com")
  -server server
        LJ server (default "http://livejournal.com")
  -u username
        shorthand for -username username
  -username username
        LJ username
```

Archive of each journal is stored in the accordingly named subdirectory of the main directory. In addition userpics and their keywords are stored in the subdirectory `account.data`.

## Compilation
Ensure that the directory with ljdumpgo sources is on the GOPATH and then run from there:
```
go build -v -o ljdumpgo *.go
```

To produce a fully static binary on Linux build with `CGO_ENABLED=0`:
```
CGO_ENABLED=0 go build -v -o ljdumpgo *.go
```

All dependent packages that the utility uses are stored under the vendor folder so no network connection is necessary to compile it. When using a Go compiler older than 1.6, add the vendor directory to GOPATH.

## Compatibility with ljdump.py
ljdumpgo reads most configuration and database files created by ljdump.py and can be used to continue archiving the data previously downloaded by that utility. The only exception is `userpics.xml` file and corresponding image files. As ljdump.py downloads those files each time it runs, ljdumpgo replaces that with separated `account.data` directory that stores the picture files and meta-information about them allowing to skip downloads if the files have not changed.

ljdumpgo does not support writing meta-information about the state of comments using Python-specific pickle files. Instead it uses diff-friendly plain text files that can easily be edited by hands. Thus running the Python utility after ljdumpgo downloaded some posts or comments will re-download those again.
