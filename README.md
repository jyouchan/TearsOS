= TearsOS =

Final project for the CS439H Fall 2016 class at the University of Texas at Austin.

A large-scale implementation of a full operating system, with various teams working on different
components in hopes of creating a relatively feature complete operating system kernel and associated
user programs.

= Team Assignments =

For team assignments, see the Google Spreadsheet linked below. Note that all team assignments should be done by
Monday, November 14th; everyone should sign up for 1 major project, but may sign up for additional
optional projects they won't be strictly graded against.

Spreadsheet: https://docs.google.com/spreadsheets/d/1VS3HQCBj_3ILjFM-nHK4Su4BSxlQ67Z0NJ4yrR1Yy-s/edit#gid=0 

= Setup =

```bash
git clone https://github.com/souvik1997/TearsOS.git
```

This will clone the directory locally to your computer; to update with any upstream changes
in the directory which have happened in the past, use

```bash
git pull --rebase
```

Commiting changes is your typical

```bash
git add --all
git commit -m "<descriptive message here>"
git push
```

= Compilation =

This is (unfortunately) a makefile based system, so compilation can be done through

```bash
make
```

Running with specific tests is simply

```bash
make <test>.result
```

Running all tests is simply

```bash
make test
```
