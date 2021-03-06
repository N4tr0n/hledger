* toc



## hledger

This doc is for version **1.2**. []{.docversions}

### NAME

hledger - a command-line accounting tool

### SYNOPSIS

`hledger [-f FILE] COMMAND [OPTIONS] [ARGS]`\
`hledger [-f FILE] ADDONCMD -- [OPTIONS] [ARGS]`\
`hledger`

### DESCRIPTION

hledger is a cross-platform program for tracking money, time, or any
other commodity, using double-entry accounting and a simple, editable
file format. hledger is inspired by and largely compatible with
ledger(1).\
Tested on unix, mac, windows, hledger aims to be a reliable, practical
tool for daily use.

This is hledger’s command-line interface (there are also curses and web
interfaces). Its basic function is to read a plain text file describing
financial transactions (in accounting terms, a general journal) and
print useful reports on standard output, or export them as CSV. hledger
can also read some other file formats such as CSV files, translating
them to journal format. Additionally, hledger lists other hledger-\*
executables found in the user’s \$PATH and can invoke them as
subcommands.

hledger reads data from one or more files in hledger journal, timeclock,
timedot, or CSV format specified with `-f`, or `$LEDGER_FILE`, or
`$HOME/.hledger.journal` (on windows, perhaps
`C:/Users/USER/.hledger.journal`). If using `$LEDGER_FILE`, note this
must be a real environment variable, not a shell variable. You can
specify standard input with `-f-`.

Transactions are dated movements of money between two (or more) named
accounts, and are recorded with journal entries like this:

``` {.journal}
2015/10/16 bought food
 expenses:food          $10
 assets:cash
```

For more about this format, see hledger\_journal(5).

Most users use a text editor to edit the journal, usually with an editor
mode such as ledger-mode for added convenience. hledger’s interactive
add command is another way to record new transactions. hledger never
changes existing transactions.

To get started, you can either save some entries like the above in
`~/.hledger.journal`, or run `hledger add` and follow the prompts. Then
try some commands like `hledger print` or `hledger balance`. Run
`hledger` with no arguments for a list of commands.

### EXAMPLES

Two simple transactions in hledger journal format:

``` {.journal}
2015/9/30 gift received
  assets:cash   $20
  income:gifts

2015/10/16 farmers market
  expenses:food    $10
  assets:cash
```

Some basic reports:

``` {.shell}
$ hledger print
2015/09/30 gift received
    assets:cash            $20
    income:gifts          $-20

2015/10/16 farmers market
    expenses:food           $10
    assets:cash            $-10
```

``` {.shell}
$ hledger accounts --tree
assets
  cash
expenses
  food
income
  gifts
```

``` {.shell}
$ hledger balance
                 $10  assets:cash
                 $10  expenses:food
                $-20  income:gifts
--------------------
                   0
```

``` {.shell}
$ hledger register cash
2015/09/30 gift received   assets:cash               $20           $20
2015/10/16 farmers market  assets:cash              $-10           $10
```

More commands:

``` {.shell}
$ hledger                                 # show available commands
$ hledger add                             # add more transactions to the journal file
$ hledger balance                         # all accounts with aggregated balances
$ hledger balance --help                  # show detailed help for balance command
$ hledger balance --depth 1               # only top-level accounts
$ hledger register                        # show account postings, with running total
$ hledger reg income                      # show postings to/from income accounts
$ hledger reg 'assets:some bank:checking' # show postings to/from this checking account
$ hledger print desc:shop                 # show transactions with shop in the description
$ hledger activity -W                     # show transaction counts per week as a bar chart
```

### OPTIONS

#### General options

To see general usage help, including general options which are supported
by most hledger commands, run `hledger -h`. (Note -h and --help are
different, like git.)

General help options:

`-h`
:   show general usage (or after COMMAND, command usage)

`--help`
:   show this program's manual as plain text (or after an add-on
    COMMAND, the add-on's manual)

`--man`
:   show this program's manual with man

`--info`
:   show this program's manual with info

`--version`
:   show version

`--debug[=N]`
:   show debug output (levels 1-9, default: 1)

General input options:

`-f FILE --file=FILE`
:   use a different input file. For stdin, use - (default:
    `$LEDGER_FILE` or `$HOME/.hledger.journal`)

`--rules-file=RULESFILE`
:   Conversion rules file to use when reading CSV (default: FILE.rules)

`--alias=OLD=NEW`
:   rename accounts named OLD to NEW

`--anon`
:   anonymize accounts and payees

`--pivot TAGNAME`
:   use some other field/tag for account names

`-I --ignore-assertions`
:   ignore any failing balance assertions

General reporting options:

`-b --begin=DATE`
:   include postings/txns on or after this date

`-e --end=DATE`
:   include postings/txns before this date

`-D --daily`
:   multiperiod/multicolumn report by day

`-W --weekly`
:   multiperiod/multicolumn report by week

`-M --monthly`
:   multiperiod/multicolumn report by month

`-Q --quarterly`
:   multiperiod/multicolumn report by quarter

`-Y --yearly`
:   multiperiod/multicolumn report by year

`-p --period=PERIODEXP`
:   set start date, end date, and/or reporting interval all at once
    (overrides the flags above)

`--date2`
:   show, and match with -b/-e/-p/date:, secondary dates instead

`-C --cleared`
:   include only cleared postings/txns

`--pending`
:   include only pending postings/txns

`-U --uncleared`
:   include only uncleared (and pending) postings/txns

`-R --real`
:   include only non-virtual postings

`--depth=N`
:   hide accounts/postings deeper than N

`-E --empty`
:   show items with zero amount, normally hidden

`-B --cost`
:   convert amounts to their cost at transaction time (using the
    [transaction price](journal.html#transaction-prices), if any)

`-V --value`
:   convert amounts to their market value on the report end date (using
    the most recent applicable [market
    price](journal.html#market-prices), if any)

Note when multiple similar reporting options are provided, the last one
takes precedence. Eg `-p feb -p mar` is equivalent to `-p mar`.

Some of these can also be written as [queries](#queries).

#### Command options

To see options for a particular command, including command-specific
options, run: `hledger COMMAND -h`.

Command-specific options must be written after the command name, eg:
`hledger print -x`.

Additionally, if the command is an [addon](#commands), you may need to
put its options after a double-hyphen, eg: `hledger ui -- --watch`. Or,
you can run the addon executable directly: `hledger-ui --watch`.

#### Command arguments

Most hledger commands accept arguments after the command name, which are
often a [query](#queries), filtering the data in some way.

#### Special characters

Option and argument values which contain problematic characters should
be escaped with double quotes, backslashes, or (best) single quotes.
Problematic characters means spaces, and also characters which are
significant to your command shell, such as less-than/greater-than. Eg:
`hledger register -p 'last year' "accounts receivable (receivable|payable)" amt:\>100`.

Characters which are significant both to the shell and in [regular
expressions](#regular-expressions) sometimes need to be double-escaped.
These include parentheses, the pipe symbol and the dollar sign. Eg, to
match the dollar symbol, bash users should do:
`hledger balance cur:'\$'` or `hledger balance cur:\\$`.

There's more.. options and arguments get de-escaped when hledger is
passing them to an addon executable. In this case you might need
*triple*-escaping. Eg: `hledger ui cur:'\\$'` or `hledger ui cur:\\\\$`.

If in doubt, keep things simple:

-   run add-on executables directly
-   write options after the command
-   enclose problematic args in single quotes
-   if needed, also add a backslash to escape regexp metacharacters

If you're really stumped, add `--debug=2` to troubleshoot.

#### Input files

hledger reads transactions from a data file (and the add command writes
to it). By default this file is `$HOME/.hledger.journal` (or on Windows,
something like `C:/Users/USER/.hledger.journal`). You can override this
with the `$LEDGER_FILE` environment variable:

``` {.bash}
$ setenv LEDGER_FILE ~/finance/2016.journal
$ hledger stats
```

or with the `-f/--file` option:

``` {.bash}
$ hledger -f /some/file stats
```

The file name `-` (hyphen) means standard input:

``` {.bash}
$ cat some.journal | hledger -f-
```

Usually the data file is in hledger's journal format, but it can also be
one of several other formats, listed below. hledger detects the format
automatically based on the file extension, or if that is not recognised,
by trying each built-in "reader" in turn:

  Reader:       Reads:                                                Used for file extensions:
  ------------- ----------------------------------------------------- --------------------------------------
  `journal`     hledger's journal format, also some Ledger journals   `.journal` `.j` `.hledger` `.ledger`
  `timeclock`   timeclock files (precise time logging)                `.timeclock`
  `timedot`     timedot files (approximate time logging)              `.timedot`
  `csv`         comma-separated values (data interchange)             `.csv`

If needed (eg to ensure correct error messages when a file has the
"wrong" extension), you can force a specific reader/format by prepending
it to the file path with a colon. Examples:

``` {.bash}
$ hledger -f csv:/some/csv-file.dat stats
$ echo 'i 2009/13/1 08:00:00' | hledger print -ftimeclock:-
```

You can also specify multiple `-f` options, to read multiple files as
one big journal. There are some limitations with this:

-   directives in one file will not affect the other files
-   [balance assertions](/journal.html#balance-assertions) will not see
    any account balances from previous files

If you need those, either use the [include
directive](/journal.html#including-other-files), or concatenate the
files, eg: `cat a.journal b.journal | hledger -f- CMD`.

#### Smart dates

hledger's user interfaces accept a flexible "smart date" syntax (unlike
dates in the journal file). Smart dates allow some english words, can be
relative to today's date, and can have less-significant date parts
omitted (defaulting to 1).

Examples:

  -------------------------------------------------- -------------------------------------------------------
  `2009/1/1`, `2009/01/01`, `2009-1-1`, `2009.1.1`   simple dates, several separators allowed
  `2009/1`, `2009`                                   same as above - a missing day or month defaults to 1
  `1/1`, `january`, `jan`, `this year`               relative dates, meaning january 1 of the current year
  `next year`                                        january 1 of next year
  `this month`                                       the 1st of the current month
  `this week`                                        the most recent monday
  `last week`                                        the monday of the week before this one
  `lastweek`                                         spaces are optional
  `today`, `yesterday`, `tomorrow`                   
  -------------------------------------------------- -------------------------------------------------------

#### Report start & end date

Most hledger reports show the full span of time represented by the
journal data, by default. So, the effective report start and end dates
will be the earliest and latest transaction or posting dates found in
the journal.

Often you will want to see a shorter time span, such as the current
month. You can specify a start and/or end date using
[`-b/--begin`](#reporting-options), [`-e/--end`](#reporting-options),
[`-p/--period`](#period-expressions) or a [`date:` query](#queries)
(described below). All of these accept the [smart date](#smart-dates)
syntax. One important thing to be aware of when specifying end dates: as
in Ledger, end dates are exclusive, so you need to write the date
*after* the last day you want to include.

Examples:

  ------------------- ---------------------------------------------------------------------------------------------
  `-b 2016/3/17`      begin on St. Patrick's day 2016
  `-e 12/1`           end at the start of december 1st of the current year (11/30 will be the last date included)
  `-b thismonth`      all transactions on or after the 1st of the current month
  `-p thismonth`      all transactions in the current month
  `date:2016/3/17-`   the above written as queries instead
  `date:-12/1`        
  `date:thismonth-`   
  `date:thismonth`    
  ------------------- ---------------------------------------------------------------------------------------------

#### Report intervals

A report interval can be specified so that commands like
[register](#register), [balance](#balance) and [activity](#activity)
will divide their reports into multiple subperiods. The basic intervals
can be selected with one of `-D/--daily`, `-W/--weekly`, `-M/--monthly`,
`-Q/--quarterly`, or `-Y/--yearly`. More complex intervals may be
specified with a [period expression](#period-expressions). Report
intervals can not be specified with a [query](#queries), currently.

#### Period expressions

The `-p/--period` option accepts period expressions, a shorthand way of
expressing a start date, end date, and/or report interval all at once.

Here's a basic period expression specifying the first quarter of 2009.
Note, hledger always treats start dates as inclusive and end dates as
exclusive:

`-p "from 2009/1/1 to 2009/4/1"`

Keywords like "from" and "to" are optional, and so are the spaces, as
long as you don't run two dates together. "to" can also be written as
"-". These are equivalent to the above:

  --------------------------
  `-p "2009/1/1 2009/4/1"`
  `-p2009/1/1to2009/4/1`
  `-p2009/1/1-2009/4/1`
  --------------------------

Dates are [smart dates](#smart-dates), so if the current year is 2009,
the above can also be written as:

  -------------------------
  `-p "1/1 4/1"`
  `-p "january-apr"`
  `-p "this year to 4/1"`
  -------------------------

If you specify only one date, the missing start or end date will be the
earliest or latest transaction in your journal:

  ---------------------- -----------------------------------
  `-p "from 2009/1/1"`   everything after january 1, 2009
  `-p "from 2009/1"`     the same
  `-p "from 2009"`       the same
  `-p "to 2009"`         everything before january 1, 2009
  ---------------------- -----------------------------------

A single date with no "from" or "to" defines both the start and end date
like so:

  ----------------- --------------------------------------------------------
  `-p "2009"`       the year 2009; equivalent to "2009/1/1 to 2010/1/1"
  `-p "2009/1"`     the month of jan; equivalent to "2009/1/1 to 2009/2/1"
  `-p "2009/1/1"`   just that day; equivalent to "2009/1/1 to 2009/1/2"
  ----------------- --------------------------------------------------------

The argument of `-p` can also begin with, or be, a [report
interval](#report-intervals) expression. The basic report intervals are
`daily`, `weekly`, `monthly`, `quarterly`, or `yearly`, which have the
same effect as the `-D`,`-W`,`-M`,`-Q`, or `-Y` flags. Between report
interval and start/end dates (if any), the word `in` is optional.
Examples:

  -----------------------------------------
  `-p "weekly from 2009/1/1 to 2009/4/1"`
  `-p "monthly in 2008"`
  `-p "quarterly"`
  -----------------------------------------

The following more complex report intervals are also supported:
`biweekly`, `bimonthly`, `every N days|weeks|months|quarters|years`,
`every Nth day [of month]`, `every Nth day of week`.

Examples:

  ------------------------------
  `-p "bimonthly from 2008"`
  `-p "every 2 weeks"`
  `-p "every 5 days from 1/3"`
  ------------------------------

Show historical balances at end of 15th each month (N is exclusive end
date):

`hledger balance -H -p "every 16th day"`

Group postings from start of wednesday to end of next tuesday (N is
start date and exclusive end date):

`hledger register checking -p "every 3rd day of week"`

#### Depth limiting

With the `--depth N` option, commands like [account](#account),
[balance](#balance) and [register](#register) will show only the
uppermost accounts in the account tree, down to level N. Use this when
you want a summary with less detail.

#### Pivoting

Normally hledger sums amounts, and organizes them in a hierarchy, based
on account name. The `--pivot TAGNAME` option causes it to sum and
organize hierarchy based on some other field instead.

TAGNAME is the full, case-insensitive name of a
[tag](/journal.html#tags) you have defined, or one of the built-in
implicit tags (like `code` or `payee`). As with account names, when tag
values have `multiple:colon-separated:parts` hledger will build
hierarchy, displayed in tree-mode reports, summarisable with a depth
limit, and so on.

`--pivot` is a general option affecting all reports; you can think of
hledger transforming the journal before any other processing, replacing
every posting's account name with the value of the specified tag on that
posting, inheriting it from the transaction or using a blank value if
it's not present.

An example:

``` {.journal}
2016/02/16 Member Fee Payment
    assets:bank account                    2 EUR
    income:member fees                    -2 EUR  ; member: John Doe
```

Normal balance report showing account names:

``` {.shell}
$ hledger balance
               2 EUR  assets:bank account
              -2 EUR  income:member fees
--------------------
                   0
```

Pivoted balance report, using member: tag values instead:

``` {.shell}
$ hledger balance --pivot member
               2 EUR
              -2 EUR  John Doe
--------------------
                   0
```

One way to show only amounts with a member: value (using a
[query](#queries), described below):

``` {.shell}
$ hledger balance --pivot member tag:member=.
              -2 EUR  John Doe
--------------------
              -2 EUR
```

Another way (the acct: query matches against the pivoted "account
name"):

    $ hledger balance --pivot member acct:.
                  -2 EUR  John Doe
    --------------------
                  -2 EUR

#### Regular expressions

hledger uses [regular expressions](http://www.regular-expressions.info)
in a number of places:

-   [query terms](#queries), on the command line and in the hledger-web
    search form: `REGEX`, `desc:REGEX`, `cur:REGEX`, `tag:...=REGEX`
-   [CSV rules](#csv-rules) conditional blocks: `if REGEX ...`
-   [account alias](#account-aliases) directives and options:
    `alias /REGEX/ = REPLACEMENT`, `--alias /REGEX/=REPLACEMENT`

hledger's regular expressions come from the
[regex-tdfa](http://hackage.haskell.org/package/regex-tdfa/docs/Text-Regex-TDFA.html)
library. In general they:

-   are case insensitive
-   are infix matching (do not need to match the entire thing being
    matched)
-   are [POSIX extended regular
    expressions](http://www.regular-expressions.info/posix.html#ere)
-   also support [GNU word
    boundaries](http://www.regular-expressions.info/wordboundaries.html)
    (\\&lt;, \\&gt;, \\b, \\B)
-   and parenthesised [capturing
    groups](http://www.regular-expressions.info/refcapture.html) and
    numeric backreferences in replacement strings
-   do not support [mode
    modifiers](http://www.regular-expressions.info/modifiers.html) like
    (?s)

Some things to note:

-   In the `alias` directive and `--alias` option, regular expressions
    must be enclosed in forward slashes (`/REGEX/`). Elsewhere in
    hledger, these are not required.

-   In queries, to match a regular expression metacharacter like `$` as
    a literal character, prepend a backslash. Eg to search for amounts
    with the dollar sign in hledger-web, write `cur:\$`.

-   On the command line, some metacharacters like `$` have a special
    meaning to the shell and so must be escaped at least once more. See
    [Special characters](#special-characters).

### QUERIES

One of hledger's strengths is being able to quickly report on precise
subsets of your data. Most commands accept an optional query expression,
written as arguments after the command name, to filter the data by date,
account name or other criteria. The syntax is similar to a web search:
one or more space-separated search terms, quotes to enclose whitespace,
optional prefixes to match specific fields. Multiple search terms are
combined as follows:

All commands except print: show transactions/postings/accounts which
match (or negatively match)

-   any of the description terms AND
-   any of the account terms AND
-   all the other terms.

The print command: show transactions which

-   match any of the description terms AND
-   have any postings matching any of the positive account terms AND
-   have no postings matching any of the negative account terms AND
-   match all the other terms.

The following kinds of search terms can be used:

**`REGEX`**
:   match account names by this regular expression

**`acct:REGEX`**
:   same as above

**`amt:N, amt:<N, amt:<=N, amt:>N, amt:>=N`**
:   match postings with a single-commodity amount that is equal to, less
    than, or greater than N. (Multi-commodity amounts are not tested,
    and will always match.) The comparison has two modes: if N is
    preceded by a + or - sign (or is 0), the two signed numbers are
    compared. Otherwise, the absolute magnitudes are compared, ignoring
    sign.

**`code:REGEX`**
:   match by transaction code (eg check number)

**`cur:REGEX`**
:   match postings or transactions including any amounts whose
    currency/commodity symbol is fully matched by REGEX. (For a partial
    match, use `.*REGEX.*`). Note, to match characters which are
    regex-significant, like the dollar sign (`$`), you need to prepend
    `\`. And when using the command line you need to add one more level
    of quoting to hide it from the shell, so eg do:
    `hledger print cur:'\$'` or `hledger print cur:\\$`.

**`desc:REGEX`**
:   match transaction descriptions

**`date:PERIODEXPR`**
:   match dates within the specified period. PERIODEXPR is a [period
    expression](#period-expressions) (with no report interval).
    Examples: `date:2016`, `date:thismonth`, `date:2000/2/1-2/15`,
    `date:lastweek-`. If the `--date2` command line flag is present,
    this matches [secondary dates](manual.html#secondary-dates) instead.

**`date2:PERIODEXPR`**
:   match secondary dates within the specified period.

**`depth:N`**
:   match (or display, depending on command) accounts at or above this
    depth

**`real:, real:0`**
:   match real or virtual postings respectively

**`status:*, status:!, status:`**
:   match cleared, pending, or uncleared/pending transactions
    respectively

**`tag:REGEX[=REGEX]`**
:   match by tag name, and optionally also by tag value. Note a tag:
    query is considered to match a transaction if it matches any of the
    postings. Also remember that postings inherit the tags of their
    parent transaction.

**`not:`**
:   before any of the above negates the match.

**`inacct:ACCTNAME`**
:   a special term used automatically when you click an account name in
    hledger-web, specifying the account register we are currently in
    (selects the transactions of that account and how to show them, can
    be filtered further with `acct` etc). Not supported elsewhere in
    hledger.

Some of these can also be expressed as command-line options (eg
`depth:2` is equivalent to `--depth 2`). Generally you can mix options
and query arguments, and the resulting query will be their intersection
(perhaps excluding the `-p/--period` option).

### COMMANDS

hledger provides a number of subcommands; `hledger` with no arguments
shows a list.

If you install additional `hledger-*` packages, or if you put programs
or scripts named `hledger-NAME` in your PATH, these will also be listed
as subcommands.

Run a subcommand by writing its name as first argument (eg
`hledger incomestatement`). You can also write any unambiguous prefix of
a command name (`hledger inc`), or one of the standard short aliases
displayed in the command list (`hledger is`).

<!--
---
comment:
for each command: name, synopsis, description, examples.
...
-->
#### accounts

Show account names.

`--tree`
:   show short account names, as a tree

`--flat`
:   show full account names, as a list (default)

`--drop=N`
:   in flat mode: omit N leading account name parts

This command lists all account names that are in use (ie, all the
accounts which have at least one transaction posting to them). With
query arguments, only matched account names are shown.

It shows a flat list by default. With `--tree`, it uses indentation to
show the account hierarchy.

In flat mode you can add `--drop N` to omit the first few account name
components.

Examples:

<div class="container-fluid">

<div class="row">

<div class="col-sm-4">

``` {.shell}
$ hledger accounts --tree
assets
  bank
    checking
    saving
  cash
expenses
  food
  supplies
income
  gifts
  salary
liabilities
  debts
```

</div>

<div class="col-sm-4">

``` {.shell}
$ hledger accounts --drop 1
bank:checking
bank:saving
cash
food
supplies
gifts
salary
debts
```

</div>

<div class="col-sm-4">

``` {.shell}
$ hledger accounts
assets:bank:checking
assets:bank:saving
assets:cash
expenses:food
expenses:supplies
income:gifts
income:salary
liabilities:debts
```

</div>

</div>

</div>

#### activity

Show an ascii barchart of posting counts per interval.

The activity command displays an ascii histogram showing transaction
counts by day, week, month or other reporting interval (by day is the
default). With query arguments, it counts only matched transactions.

``` {.shell}
$ hledger activity --quarterly
2008-01-01 **
2008-04-01 *******
2008-07-01 
2008-10-01 **
```

#### add

Prompt for transactions and add them to the journal.

`--no-new-accounts`
:   don't allow creating new accounts; helps prevent typos when entering
    account names

Many hledger users edit their journals directly with a text editor, or
generate them from CSV. For more interactive data entry, there is the
`add` command, which prompts interactively on the console for new
transactions, and appends them to the journal file (if there are
multiple `-f FILE` options, the first file is used.) Existing
transactions are not changed. This is the only hledger command that
writes to the journal file.

To use it, just run `hledger add` and follow the prompts. You can add as
many transactions as you like; when you are finished, enter `.` or press
control-d or control-c to exit.

Features:

-   add tries to provide useful defaults, using the most similar recent
    transaction (by description) as a template.
-   You can also set the initial defaults with command line arguments.
-   [Readline-style edit
    keys](http://tiswww.case.edu/php/chet/readline/rluserman.html#SEC3)
    can be used during data entry.
-   The tab key will auto-complete whenever possible - accounts,
    descriptions, dates (`yesterday`, `today`, `tomorrow`). If the input
    area is empty, it will insert the default value.
-   If the journal defines a [default commodity](#default-commodity), it
    will be added to any bare numbers entered.
-   A parenthesised transaction [code](#entries) may be entered
    following a date.
-   [Comments](#comments) and tags may be entered following a
    description or amount.
-   If you make a mistake, enter `<` at any prompt to restart the
    transaction.
-   Input prompts are displayed in a different colour when the terminal
    supports it.

Example (see the
[tutorial](step-by-step.html#record-a-transaction-with-hledger-add) for
a detailed explanation):

``` {.shell}
$ hledger add
Adding transactions to journal file /src/hledger/examples/sample.journal
Any command line arguments will be used as defaults.
Use tab key to complete, readline keys to edit, enter to accept defaults.
An optional (CODE) may follow transaction dates.
An optional ; COMMENT may follow descriptions or amounts.
If you make a mistake, enter < at any prompt to restart the transaction.
To end a transaction, enter . when prompted.
To quit, enter . at a date prompt or press control-d or control-c.
Date [2015/05/22]: 
Description: supermarket
Account 1: expenses:food
Amount  1: $10
Account 2: assets:checking
Amount  2 [$-10.0]: 
Account 3 (or . or enter to finish this transaction): .
2015/05/22 supermarket
    expenses:food             $10
    assets:checking        $-10.0

Save this transaction to the journal ? [y]: 
Saved.
Starting the next transaction (. or ctrl-D/ctrl-C to quit)
Date [2015/05/22]: <CTRL-D> $
```

#### balance

Show accounts and their balances. Alias: bal.

`--change`
:   show balance change in each period (default)

`--cumulative`
:   show balance change accumulated across periods (in multicolumn
    reports)

`-H --historical`
:   show historical ending balance in each period (includes postings
    before report start date)

`--tree`
:   show accounts as a tree; amounts include subaccounts (default in
    simple reports)

`--flat`
:   show accounts as a list; amounts exclude subaccounts except when
    account is depth-clipped (default in multicolumn reports)

`-A --average`
:   show a row average column (in multicolumn mode)

`-T --row-total`
:   show a row total column (in multicolumn mode)

`-N --no-total`
:   don't show the final total row

`--drop=N`
:   omit N leading account name parts (in flat mode)

`--no-elide`
:   don't squash boring parent accounts (in tree mode)

`--format=LINEFORMAT`
:   in single-column balance reports: use this custom line format

`-O FMT --output-format=FMT`
:   select the output format. Supported formats: txt, csv.

`-o FILE --output-file=FILE`
:   write output to FILE. A file extension matching one of the above
    formats selects that format.

`--pretty-tables`
:   Use unicode to display prettier tables.

The balance command displays accounts and balances. It is hledger's most
featureful and most useful command.

``` {.shell}
$ hledger balance
                 $-1  assets
                  $1    bank:saving
                 $-2    cash
                  $2  expenses
                  $1    food
                  $1    supplies
                 $-2  income
                 $-1    gifts
                 $-1    salary
                  $1  liabilities:debts
--------------------
                   0
```

More precisely, the balance command shows the *change* to each account's
balance caused by all (matched) postings. In the common case where you
do not filter by date and your journal sets the correct opening
balances, this is the same as the account's ending balance.

By default, accounts are displayed hierarchically, with subaccounts
indented below their parent. "Boring" accounts, which contain a single
interesting subaccount and no balance of their own, are elided into the
following line for more compact output. (Use `--no-elide` to prevent
this.)

Each account's balance is the "inclusive" balance - it includes the
balances of any subaccounts.

Accounts which have zero balance (and no non-zero subaccounts) are
omitted. Use `-E/--empty` to show them.

A final total is displayed by default; use `-N/--no-total` to suppress
it:

``` {.shell}
$ hledger balance -p 2008/6 expenses --no-total
                  $2  expenses
                  $1    food
                  $1    supplies
```

##### Flat mode

To see a flat list of full account names instead of the default
hierarchical display, use `--flat`. In this mode, accounts (unless
depth-clipped) show their "exclusive" balance, excluding any subaccount
balances. In this mode, you can also use `--drop N` to omit the first
few account name components.

``` {.shell}
$ hledger balance -p 2008/6 expenses -N --flat --drop 1
                  $1  food
                  $1  supplies
```

##### Depth limited balance reports

With `--depth N`, balance shows accounts only to the specified depth.
This is very useful to show a complex charts of accounts in less detail.
In flat mode, balances from accounts below the depth limit will be shown
as part of a parent account at the depth limit.

``` {.shell}
$ hledger balance -N --depth 1
                 $-1  assets
                  $2  expenses
                 $-2  income
                  $1  liabilities
```

<!-- $ for y in 2006 2007 2008 2009 2010; do echo; echo $y; hledger -f $y.journal balance ^expenses --depth 2; done -->
##### Multicolumn balance reports

With a [reporting interval](#reporting-interval), multiple balance
columns will be shown, one for each report period. There are three types
of multi-column balance report, showing different information:

1.  By default: each column shows the sum of postings in that period, ie
    the account's change of balance in that period. This is useful eg
    for a monthly income statement: <!--
        multi-column income statement: 

           $ hledger balance ^income ^expense -p 'monthly this year' --depth 3

        or cashflow statement:

           $ hledger balance ^assets ^liabilities 'not:(receivable|payable)' -p 'weekly this month'
        -->

    ``` {.shell}
    $ hledger balance --quarterly income expenses -E
    Balance changes in 2008:

                       ||  2008q1  2008q2  2008q3  2008q4 
    ===================++=================================
     expenses:food     ||       0      $1       0       0 
     expenses:supplies ||       0      $1       0       0 
     income:gifts      ||       0     $-1       0       0 
     income:salary     ||     $-1       0       0       0 
    -------------------++---------------------------------
                       ||     $-1      $1       0       0 
    ```

2.  With `--cumulative`: each column shows the ending balance for that
    period, accumulating the changes across periods, starting from 0 at
    the report start date:

    ``` {.shell}
    $ hledger balance --quarterly income expenses -E --cumulative
    Ending balances (cumulative) in 2008:

                       ||  2008/03/31  2008/06/30  2008/09/30  2008/12/31 
    ===================++=================================================
     expenses:food     ||           0          $1          $1          $1 
     expenses:supplies ||           0          $1          $1          $1 
     income:gifts      ||           0         $-1         $-1         $-1 
     income:salary     ||         $-1         $-1         $-1         $-1 
    -------------------++-------------------------------------------------
                       ||         $-1           0           0           0 
    ```

3.  With `--historical/-H`: each column shows the actual historical
    ending balance for that period, accumulating the changes across
    periods, starting from the actual balance at the report start date.
    This is useful eg for a multi-period balance sheet, and when you are
    showing only the data after a certain start date:

    ``` {.shell}
    $ hledger balance ^assets ^liabilities --quarterly --historical --begin 2008/4/1
    Ending balances (historical) in 2008/04/01-2008/12/31:

                          ||  2008/06/30  2008/09/30  2008/12/31 
    ======================++=====================================
     assets:bank:checking ||          $1          $1           0 
     assets:bank:saving   ||          $1          $1          $1 
     assets:cash          ||         $-2         $-2         $-2 
     liabilities:debts    ||           0           0          $1 
    ----------------------++-------------------------------------
                          ||           0           0           0 
    ```

Multi-column balance reports display accounts in flat mode by default;
to see the hierarchy, use `--tree`.

With a reporting interval (like `--quarterly` above), the report
start/end dates will be adjusted if necessary so that they encompass the
displayed report periods. This is so that the first and last periods
will be "full" and comparable to the others.

The `-E/--empty` flag does two things in multicolumn balance reports:
first, the report will show all columns within the specified report
period (without -E, leading and trailing columns with all zeroes are not
shown). Second, all accounts which existed at the report start date will
be considered, not just the ones with activity during the report period
(use -E to include low-activity accounts which would otherwise would be
omitted).

The `-T/--row-total` flag adds an additional column showing the total
for each row.

The `-A/--average` flag adds a column showing the average value in each
row.

Here's an example of all three:

``` {.shell}
$ hledger balance -Q income expenses --tree -ETA
Balance changes in 2008:

            ||  2008q1  2008q2  2008q3  2008q4    Total  Average 
============++===================================================
 expenses   ||       0      $2       0       0       $2       $1 
   food     ||       0      $1       0       0       $1        0 
   supplies ||       0      $1       0       0       $1        0 
 income     ||     $-1     $-1       0       0      $-2      $-1 
   gifts    ||       0     $-1       0       0      $-1        0 
   salary   ||     $-1       0       0       0      $-1        0 
------------++---------------------------------------------------
            ||     $-1      $1       0       0        0        0 

# Average is rounded to the dollar here since all journal amounts are
```

##### Market value

The `-V/--value` flag converts the reported amounts to their market
value on the report end date, using the most recent applicable market
prices, when known. Specifically, when there is a [market
price](journal.html#market-prices) (P directive) for the amount's
commodity, dated on or before the [report end
date](hledger.html#report-start-end-date) (see hledger -&gt; Report
start & end date), the amount will be converted to the price's
commodity. If multiple applicable prices are defined, the latest-dated
one is used (and if dates are equal, the one last parsed).

For example:

``` {.journal}
# one euro is worth this many dollars from nov 1
P 2016/11/01 € $1.10

# purchase some euros on nov 3
2016/11/3
    assets:euros        €100
    assets:checking

# the euro is worth fewer dollars by dec 21
P 2016/12/21 € $1.03
```

How many euros do I have ?

    $ hledger -f t.j bal euros
                    €100  assets:euros

What are they worth on nov 3 ? (no report end date specified, defaults
to the last date in the journal)

    $ hledger -f t.j bal euros -V
                 $110.00  assets:euros

What are they worth on dec 21 ?

    $ hledger -f t.j bal euros -V -e 2016/12/21
                 $103.00  assets:euros

Currently, hledger's -V only uses market prices recorded with P
directives, not [transaction prices](journal.html#transaction-prices)
(unlike Ledger).

Using -B and -V together is allowed.

##### Custom balance output

In simple (non-multi-column) balance reports, you can customise the
output with `--format FMT`:

``` {.shell}
$ hledger balance --format "%20(account) %12(total)"
              assets          $-1
         bank:saving           $1
                cash          $-2
            expenses           $2
                food           $1
            supplies           $1
              income          $-2
               gifts          $-1
              salary          $-1
   liabilities:debts           $1
---------------------------------
                                0
```

The FMT format string (plus a newline) specifies the formatting applied
to each account/balance pair. It may contain any suitable text, with
data fields interpolated like so:

`%[MIN][.MAX](FIELDNAME)`

-   MIN pads with spaces to at least this width (optional)
-   MAX truncates at this width (optional)
-   FIELDNAME must be enclosed in parentheses, and can be one of:

    -   `depth_spacer` - a number of spaces equal to the account's
        depth, or if MIN is specified, MIN \* depth spaces.
    -   `account` - the account's name
    -   `total` - the account's balance/posted total, right justified

Also, FMT can begin with an optional prefix to control how
multi-commodity amounts are rendered:

-   `%_` - render on multiple lines, bottom-aligned (the default)
-   `%^` - render on multiple lines, top-aligned
-   `%,` - render on one line, comma-separated

There are some quirks. Eg in one-line mode, `%(depth_spacer)` has no
effect, instead `%(account)` has indentation built in. <!-- XXX retest:
Consistent column widths are not well enforced, causing ragged edges unless you set suitable widths.
Beware of specifying a maximum width; it will clip account names and amounts that are too wide, with no visible indication.
--> Experimentation may be needed to get pleasing results.

Some example formats:

-   `%(total)` - the account's total
-   `%-20.20(account)` - the account's name, left justified, padded to
    20 characters and clipped at 20 characters
-   `%,%-50(account)  %25(total)` - account name padded to 50
    characters, total padded to 20 characters, with multiple commodities
    rendered on one line
-   `%20(total)  %2(depth_spacer)%-(account)` - the default format for
    the single-column balance report

##### Output destination

The balance, print, register and stats commands can write their output
to a destination other than the console. This is controlled by the
`-o/--output-file` option.

``` {.shell}
$ hledger balance -o -     # write to stdout (the default)
$ hledger balance -o FILE  # write to FILE
```

##### CSV output

The balance, print and register commands can write their output as CSV.
This is useful for exporting data to other applications, eg to make
charts in a spreadsheet. This is controlled by the `-O/--output-format`
option, or by specifying a `.csv` file extension with
`-o/--output-file`.

``` {.shell}
$ hledger balance -O csv       # write CSV to stdout
$ hledger balance -o FILE.csv  # write CSV to FILE.csv
```

#### balancesheet

Show a balance sheet. Alias: bs.

`--change`
:   show balance change in each period, instead of historical ending
    balances

`--cumulative`
:   show balance change accumulated across periods (in multicolumn
    reports), instead of historical ending balances

`-H --historical`
:   show historical ending balance in each period (includes postings
    before report start date) (default)

`--tree`
:   show accounts as a tree; amounts include subaccounts (default in
    simple reports)

`--flat`
:   show accounts as a list; amounts exclude subaccounts except when
    account is depth-clipped (default in multicolumn reports)

`-A --average`
:   show a row average column (in multicolumn mode)

`-T --row-total`
:   show a row total column (in multicolumn mode)

`-N --no-total`
:   don't show the final total row

`--drop=N`
:   omit N leading account name parts (in flat mode)

`--no-elide`
:   don't squash boring parent accounts (in tree mode)

`--format=LINEFORMAT`
:   in single-column balance reports: use this custom line format

This command displays a simple [balance
sheet](http://en.wikipedia.org/wiki/Balance_sheet). It currently assumes
that you have top-level accounts named `asset` and `liability` (plural
forms also allowed.)

``` {.shell}
$ hledger balancesheet
Balance Sheet

Assets:
                 $-1  assets
                  $1    bank:saving
                 $-2    cash
--------------------
                 $-1

Liabilities:
                  $1  liabilities:debts
--------------------
                  $1

Total:
--------------------
                   0
```

With a [reporting interval](#reporting-interval), multiple columns will
be shown, one for each report period. As with [multicolumn balance
reports](#multicolumn-balance-reports), you can alter the report mode
with `--change`/`--cumulative`/`--historical`. Normally balancesheet
shows historical ending balances, which is what you need for a balance
sheet; note this means it ignores report begin dates.

#### cashflow

Show a cashflow statement. Alias: cf.

`--change`
:   show balance change in each period (default)

`--cumulative`
:   show balance change accumulated across periods (in multicolumn
    reports), instead of changes during periods

`-H --historical`
:   show historical ending balance in each period (includes postings
    before report start date), instead of changes during each period

`--tree`
:   show accounts as a tree; amounts include subaccounts (default in
    simple reports)

`--flat`
:   show accounts as a list; amounts exclude subaccounts except when
    account is depth-clipped (default in multicolumn reports)

`-A --average`
:   show a row average column (in multicolumn mode)

`-T --row-total`
:   show a row total column (in multicolumn mode)

`-N --no-total`
:   don't show the final total row (in simple reports)

`--drop=N`
:   omit N leading account name parts (in flat mode)

`--no-elide`
:   don't squash boring parent accounts (in tree mode)

`--format=LINEFORMAT`
:   in single-column balance reports: use this custom line format

This command displays a simple [cashflow
statement](http://en.wikipedia.org/wiki/Cash_flow_statement) It shows
the change in all "cash" (ie, liquid assets) accounts for the period. It
currently assumes that cash accounts are under a top-level account named
`asset` and do not contain `receivable` or `A/R` (plural forms also
allowed.)

``` {.shell}
$ hledger cashflow
Cashflow Statement

Cash flows:
                 $-1  assets
                  $1    bank:saving
                 $-2    cash
--------------------
                 $-1

Total:
--------------------
                 $-1
```

With a [reporting interval](#reporting-interval), multiple columns will
be shown, one for each report period. Normally cashflow shows changes in
assets per period, though as with [multicolumn balance
reports](#multicolumn-balance-reports) you can alter the report mode
with `--change`/`--cumulative`/`--historical`.

#### help

Show any of the hledger manuals.

The `help` command displays any of the main [hledger man
pages](/docs.html). (Unlike `hledger --help`, which displays only the
hledger man page.) Run it with no arguments to list available topics
(their names are shortened for easier typing), and run
`hledger help TOPIC` to select one. The output is similar to a man page,
but fixed width. It may be long, so you may wish to pipe it into a
pager. See also [info](#info) and [man](#man).

``` {.shell}
$ hledger help
Choose a topic, eg: hledger help cli
cli, ui, web, api, journal, csv, timeclock, timedot
```

``` {.shell}
$ hledger help cli | less

hledger(1)                   hledger User Manuals                   hledger(1)



NAME
       hledger - a command-line accounting tool

SYNOPSIS
       hledger [-f FILE] COMMAND [OPTIONS] [CMDARGS]
       hledger [-f FILE] ADDONCMD -- [OPTIONS] [CMDARGS]
:
```

#### incomestatement

Show an income statement. Alias: is.

`--change`
:   show balance change in each period (default)

`--cumulative`
:   show balance change accumulated across periods (in multicolumn
    reports), instead of changes during periods

`-H --historical`
:   show historical ending balance in each period (includes postings
    before report start date), instead of changes during each period

`--tree`
:   show accounts as a tree; amounts include subaccounts (default in
    simple reports)

`--flat`
:   show accounts as a list; amounts exclude subaccounts except when
    account is depth-clipped (default in multicolumn reports)

`-A --average`
:   show a row average column (in multicolumn mode)

`-T --row-total`
:   show a row total column (in multicolumn mode)

`-N --no-total`
:   don't show the final total row

`--drop=N`
:   omit N leading account name parts (in flat mode)

`--no-elide`
:   don't squash boring parent accounts (in tree mode)

`--format=LINEFORMAT`
:   in single-column balance reports: use this custom line format

This command displays a simple [income
statement](http://en.wikipedia.org/wiki/Income_statement). It currently
assumes that you have top-level accounts named `income` (or `revenue`)
and `expense` (plural forms also allowed.)

``` {.shell}
$ hledger incomestatement
Income Statement

Revenues:
                 $-2  income
                 $-1    gifts
                 $-1    salary
--------------------
                 $-2

Expenses:
                  $2  expenses
                  $1    food
                  $1    supplies
--------------------
                  $2

Total:
--------------------
                   0
```

With a [reporting interval](#reporting-interval), multiple columns will
be shown, one for each report period. Normally incomestatement shows
revenues/expenses per period, though as with [multicolumn balance
reports](#multicolumn-balance-reports) you can alter the report mode
with `--change`/`--cumulative`/`--historical`.

#### info

Show any of the hledger manuals using info.

The `info` command displays any of the [hledger reference
manuals](/docs.html) using the
[info](https://en.wikipedia.org/wiki/Info_(Unix)) hypertextual
documentation viewer. This can be a very efficient way to browse large
manuals. It requires the "info" program to be available in your PATH.

As with [help](#help), run it with no arguments to list available topics
(manuals).

#### man

Show any of the hledger manuals using man.

The `man` command displays any of the [hledger reference
manuals](/docs.html) using
[man](https://en.wikipedia.org/wiki/Man_page), the standard
documentation viewer on unix systems. This will fit the text to your
terminal width, and probably invoke a pager automatically. It requires
the "man" program to be available in your PATH.

As with [help](#help), run it with no arguments to list available topics
(manuals).

#### print

Show transactions from the journal.

`-x     --explicit`
:   show all amounts explicitly

`-m STR --match=STR`
:   show the transaction whose description is most similar to STR, and
    is most recent

`-O FMT --output-format=FMT`
:   select the output format. Supported formats: txt, csv.

`-o FILE --output-file=FILE`
:   write output to FILE. A file extension matching one of the above
    formats selects that format.

``` {.shell}
$ hledger print
2008/01/01 income
    assets:bank:checking            $1
    income:salary                  $-1

2008/06/01 gift
    assets:bank:checking            $1
    income:gifts                   $-1

2008/06/02 save
    assets:bank:saving              $1
    assets:bank:checking           $-1

2008/06/03 * eat & shop
    expenses:food                $1
    expenses:supplies            $1
    assets:cash                 $-2

2008/12/31 * pay off
    liabilities:debts               $1
    assets:bank:checking           $-1
```

The print command displays full journal entries (transactions) from the
journal file, tidily formatted.

As of hledger 1.2, print's output is always a valid [hledger
journal](/journal.html). However it may not preserve all original
content, eg it does not print directives or inter-transaction comments.

Normally, transactions' implicit/explicit amount style is preserved:
when an amount is omitted in the journal, it will be omitted in the
output. You can use the `-x/--explicit` flag to make all amounts
explicit, which can be useful for troubleshooting or for making your
journal more readable and robust against data entry errors. Note, in
this mode postings with a multi-commodity amount (possible with an
implicit amount in a multi-commodity transaction) will be split into
multiple single-commodity postings, for valid journal output.

With -B/--cost, amounts with [transaction
prices](/journal.html#transaction-prices) are converted to cost (using
the transaction price).

The print command also supports [output
destination](#output-destination) and [CSV output](#csv-output). Here's
an example of print's CSV output:

``` {.shell}
$ hledger print -Ocsv
"txnidx","date","date2","status","code","description","comment","account","amount","commodity","credit","debit","posting-status","posting-comment"
"1","2008/01/01","","","","income","","assets:bank:checking","1","$","","1","",""
"1","2008/01/01","","","","income","","income:salary","-1","$","1","","",""
"2","2008/06/01","","","","gift","","assets:bank:checking","1","$","","1","",""
"2","2008/06/01","","","","gift","","income:gifts","-1","$","1","","",""
"3","2008/06/02","","","","save","","assets:bank:saving","1","$","","1","",""
"3","2008/06/02","","","","save","","assets:bank:checking","-1","$","1","","",""
"4","2008/06/03","","*","","eat & shop","","expenses:food","1","$","","1","",""
"4","2008/06/03","","*","","eat & shop","","expenses:supplies","1","$","","1","",""
"4","2008/06/03","","*","","eat & shop","","assets:cash","-2","$","2","","",""
"5","2008/12/31","","*","","pay off","","liabilities:debts","1","$","","1","",""
"5","2008/12/31","","*","","pay off","","assets:bank:checking","-1","$","1","","",""
```

-   There is one CSV record per posting, with the parent transaction's
    fields repeated.
-   The "txnidx" (transaction index) field shows which postings belong
    to the same transaction. (This number might change if transactions
    are reordered within the file, files are parsed/included in a
    different order, etc.)
-   The amount is separated into "commodity" (the symbol) and "amount"
    (numeric quantity) fields.
-   The numeric amount is repeated in either the "credit" or "debit"
    column, for convenience. (Those names are not accurate in the
    accounting sense; it just puts negative amounts under credit and
    zero or greater amounts under debit.)

#### register

Show postings and their running total. Alias: reg.

`--cumulative`
:   show running total from report start date (default)

`-H --historical`
:   show historical running total/balance (includes postings before
    report start date)

`-A --average`
:   show running average of posting amounts instead of total (implies
    --empty)

`-r --related`
:   show postings' siblings instead

`-w N --width=N`
:   set output width (default: terminal width or COLUMNS. -wN,M sets
    description width as well)

`-O FMT --output-format=FMT`
:   select the output format. Supported formats: txt, csv.

`-o FILE --output-file=FILE`
:   write output to FILE. A file extension matching one of the above
    formats selects that format.

The register command displays postings, one per line, and their running
total. This is typically used with a [query](#queries) selecting a
particular account, to see that account's activity:

``` {.shell}
$ hledger register checking
2008/01/01 income               assets:bank:checking            $1            $1
2008/06/01 gift                 assets:bank:checking            $1            $2
2008/06/02 save                 assets:bank:checking           $-1            $1
2008/12/31 pay off              assets:bank:checking           $-1             0
```

The `--historical`/`-H` flag adds the balance from any undisplayed prior
postings to the running total. This is useful when you want to see only
recent activity, with a historically accurate running balance:

``` {.shell}
$ hledger register checking -b 2008/6 --historical
2008/06/01 gift                 assets:bank:checking            $1            $2
2008/06/02 save                 assets:bank:checking           $-1            $1
2008/12/31 pay off              assets:bank:checking           $-1             0
```

The `--depth` option limits the amount of sub-account detail displayed.

The `--average`/`-A` flag shows the running average posting amount
instead of the running total (so, the final number displayed is the
average for the whole report period). This flag implies `--empty` (see
below). It is affected by `--historical`. It works best when showing
just one account and one commodity.

The `--related`/`-r` flag shows the *other* postings in the transactions
of the postings which would normally be shown.

With a [reporting interval](#reporting-interval), register shows summary
postings, one per interval, aggregating the postings to each account:

``` {.shell}
$ hledger register --monthly income
2008/01                 income:salary                          $-1           $-1
2008/06                 income:gifts                           $-1           $-2
```

Periods with no activity, and summary postings with a zero amount, are
not shown by default; use the `--empty`/`-E` flag to see them:

``` {.shell}
$ hledger register --monthly income -E
2008/01                 income:salary                          $-1           $-1
2008/02                                                          0           $-1
2008/03                                                          0           $-1
2008/04                                                          0           $-1
2008/05                                                          0           $-1
2008/06                 income:gifts                           $-1           $-2
2008/07                                                          0           $-2
2008/08                                                          0           $-2
2008/09                                                          0           $-2
2008/10                                                          0           $-2
2008/11                                                          0           $-2
2008/12                                                          0           $-2
```

Often, you'll want to see just one line per interval. The `--depth`
option helps with this, causing subaccounts to be aggregated:

``` {.shell}
$ hledger register --monthly assets --depth 1h
2008/01                 assets                                  $1            $1
2008/06                 assets                                 $-1             0
2008/12                 assets                                 $-1           $-1
```

Note when using report intervals, if you specify start/end dates these
will be adjusted outward if necessary to contain a whole number of
intervals. This ensures that the first and last intervals are full
length and comparable to the others in the report.

##### Custom register output

register uses the full terminal width by default, except on windows. You
can override this by setting the `COLUMNS` environment variable (not a
bash shell variable) or by using the `--width`/`-w` option.

The description and account columns normally share the space equally
(about half of (width - 40) each). You can adjust this by adding a
description width as part of --width's argument, comma-separated:
`--width W,D` . Here's a diagram:

    <--------------------------------- width (W) ---------------------------------->
    date (10)  description (D)       account (W-41-D)     amount (12)   balance (12)
    DDDDDDDDDD dddddddddddddddddddd  aaaaaaaaaaaaaaaaaaa  AAAAAAAAAAAA  AAAAAAAAAAAA

and some examples:

``` {.shell}
$ hledger reg                     # use terminal width (or 80 on windows)
$ hledger reg -w 100              # use width 100
$ COLUMNS=100 hledger reg         # set with one-time environment variable
$ export COLUMNS=100; hledger reg # set till session end (or window resize)
$ hledger reg -w 100,40           # set overall width 100, description width 40
$ hledger reg -w $COLUMNS,40      # use terminal width, and set description width
```

The register command also supports the `-o/--output-file` and
`-O/--output-format` options for controlling [output
destination](#output-destination) and [CSV output](#csv-output).

#### stats

Show some journal statistics.

`-o FILE --output-file=FILE`
:   write output to FILE. A file extension matching one of the above
    formats selects that format.

``` {.shell}
$ hledger stats
Main journal file        : /src/hledger/examples/sample.journal
Included journal files   : 
Transactions span        : 2008-01-01 to 2009-01-01 (366 days)
Last transaction         : 2008-12-31 (2333 days ago)
Transactions             : 5 (0.0 per day)
Transactions last 30 days: 0 (0.0 per day)
Transactions last 7 days : 0 (0.0 per day)
Payees/descriptions      : 5
Accounts                 : 8 (depth 3)
Commodities              : 1 ($)
```

The stats command displays summary information for the whole journal, or
a matched part of it. With a [reporting interval](#reporting-interval),
it shows a report for each report period.

The stats command also supports `-o/--output-file` for controlling
[output destination](#output-destination).

#### test

Run built-in unit tests.

``` {.shell}
$ hledger test
Cases: 74  Tried: 74  Errors: 0  Failures: 0
```

This command runs hledger's built-in unit tests and displays a quick
report. With a regular expression argument, it selects only tests with
matching names. It's mainly used in development, but it's also nice to
be able to check your hledger executable for smoke at any time.

### ADD-ON COMMANDS

hledger also searches for external add-on commands, and will include
these in the commands list. These are programs or scripts in your PATH
whose name starts with `hledger-` and ends with a recognised file
extension (currently: no extension, `bat`,`com`,`exe`,
`hs`,`lhs`,`pl`,`py`,`rb`,`rkt`,`sh`).

Add-ons can be invoked like any hledger command, but there are a few
things to be aware of. Eg if the `hledger-web` add-on is installed,

-   `hledger -h web` shows hledger's help, while `hledger web -h` shows
    hledger-web's help.

-   Flags specific to the add-on must have a preceding `--` to hide them
    from hledger. So `hledger web --serve --port 9000` will be rejected;
    you must use `hledger web -- --serve --port 9000`.

-   You can always run add-ons directly if preferred:
    `hledger-web --serve --port 9000`.

Add-ons are a relatively easy way to add local features or experiment
with new ideas. They can be written in any language, but haskell scripts
have a big advantage: they can use the same hledger (and haskell)
library functions that built-in commands do, for command-line options,
journal parsing, reporting, etc.

Here are some hledger add-ons available:

#### Official add-ons

These are maintained and released along with hledger.

##### api

[hledger-api](hledger-api.html) serves hledger data as a JSON web API.

##### ui

[hledger-ui](hledger-ui.html) provides an efficient curses-style
interface.

##### web

[hledger-web](hledger-web.html) provides a simple web interface.

#### Third party add-ons

These are maintained separately, and usually updated shortly after a
hledger release.

##### diff

[hledger-diff](http://hackage.haskell.org/package/hledger-diff) shows
differences in an account's transactions between one journal file and
another.

##### iadd

[hledger-iadd](http://hackage.haskell.org/package/hledger-iadd) is a
curses-style, more interactive replacement for the [add
command](/hledger.html#add).

##### interest

[hledger-interest](http://hackage.haskell.org/package/hledger-interest)
generates interest transactions for an account according to various
schemes.

##### irr

[hledger-irr](http://hackage.haskell.org/package/hledger-irr) calculates
the internal rate of return of an investment account.

#### Experimental add-ons

These are available in source form in the hledger repo's bin/ directory;
installing them is [pretty easy](/download.html#d). They may be less
mature and documented than built-in commands. Reading and tweaking these
is a good way to start making your own!

##### autosync

[hledger-autosync](https://github.com/simonmichael/hledger/blob/master/bin/hledger-autosync)
is a symbolic link for easily running
[ledger-autosync](https://pypi.python.org/pypi/ledger-autosync), if
installed. ledger-autosync does deduplicating conversion of OFX data and
some CSV formats, and can also download the data [if your bank offers
OFX Direct
Connect](http://wiki.gnucash.org/wiki/OFX_Direct_Connect_Bank_Settings).

##### budget

[hledger-budget.hs](https://github.com/simonmichael/hledger/blob/master/bin/hledger-budget.hs#L10)
adds more budget-tracking features to hledger.

##### chart

[hledger-chart.hs](https://github.com/simonmichael/hledger/blob/master/bin/hledger-chart.hs#L47)
is an old pie chart generator, in need of some love.

##### check

[hledger-check.hs](https://github.com/simonmichael/hledger/blob/master/bin/hledger-check.hs)
checks more powerful account balance assertions.

##### check-dates

[hledger-check-dates.hs](https://github.com/simonmichael/hledger/blob/master/bin/hledger-check-dates.hs#L15)
checks that journal entries are ordered by date.

##### check-dupes

[hledger-check-dupes.hs](https://github.com/simonmichael/hledger/blob/master/bin/hledger-check-dupes.hs#L21)
checks for account names sharing the same leaf name.

##### equity

[hledger-equity.hs](https://github.com/simonmichael/hledger/blob/master/bin/hledger-equity.hs#L17)
prints balance-resetting transactions, useful for bringing account
balances across file boundaries.

##### prices

[hledger-prices.hs](https://github.com/simonmichael/hledger/blob/master/bin/hledger-prices.hs)
prints all prices from the journal.

##### print-unique

[hledger-print-unique.hs](https://github.com/simonmichael/hledger/blob/master/bin/hledger-print-unique.hs#L15)
prints transactions which do not reuse an already-seen description.

##### register-match

[hledger-register-match.hs](https://github.com/simonmichael/hledger/blob/master/bin/hledger-register-match.hs#L23)
helps ledger-autosync detect already-seen transactions when importing.

##### rewrite

[hledger-rewrite.hs](https://github.com/simonmichael/hledger/blob/master/bin/hledger-rewrite.hs#L28)
Adds one or more custom postings to matched transactions.

### ENVIRONMENT

**COLUMNS** The screen width used by the register command. Default: the
full terminal width.

**LEDGER\_FILE** The journal file path when not specified with `-f`.
Default: `~/.hledger.journal` (on windows, perhaps
`C:/Users/USER/.hledger.journal`).

### FILES

Reads data from one or more files in hledger journal, timeclock,
timedot, or CSV format specified with `-f`, or `$LEDGER_FILE`, or
`$HOME/.hledger.journal` (on windows, perhaps
`C:/Users/USER/.hledger.journal`).

### BUGS

The need to precede addon command options with `--` when invoked from
hledger is awkward.

When input data contains non-ascii characters, a suitable system locale
must be configured (or there will be an unhelpful error). Eg on POSIX,
set LANG to something other than C.

In a Microsoft Windows CMD window, non-ascii characters and colours are
not supported.

In a Cygwin/MSYS/Mintty window, the tab key is not supported in hledger
add.

Not all of Ledger's journal file syntax is supported. See [file format
differences](faq#file-format-differences).

On large data files, hledger is slower and uses more memory than Ledger.

### TROUBLESHOOTING

Here are some issues you might encounter when you run hledger (and
remember you can also seek help from the [IRC
channel](http://irc.hledger.org), [mail list](http://list.hledger.org)
or [bug tracker](http://bugs.hledger.org)):

**Successfully installed, but "No command 'hledger' found"**\
stack and cabal install binaries into a special directory, which should
be added to your PATH environment variable. Eg on unix-like systems,
that is \~/.local/bin and \~/.cabal/bin respectively.

**I set a custom LEDGER\_FILE, but hledger is still using the default
file**\
`LEDGER_FILE` should be a real environment variable, not just a shell
variable. The command `env | grep LEDGER_FILE` should show it. You may
need to use `export`. Here's an
[explanation](http://stackoverflow.com/a/7411509).

**"Illegal byte sequence" or "Invalid or incomplete multibyte or wide
character" errors**\
In order to handle non-ascii letters and symbols (like £), hledger needs
an appropriate locale. This is usually configured system-wide; you can
also configure it temporarily. The locale may need to be one that
supports UTF-8, if you built hledger with GHC &lt; 7.2 (or possibly
always, I'm not sure yet).

Here's an example of setting the locale temporarily, on ubuntu
gnu/linux:

``` {.shell}
$ file my.journal
my.journal: UTF-8 Unicode text                 # <- the file is UTF8-encoded
$ locale -a
C
en_US.utf8                             # <- a UTF8-aware locale is available
POSIX
$ LANG=en_US.utf8 hledger -f my.journal print   # <- use it for this command
```

Here's one way to set it permanently, there are probably better ways:

``` {.shell}
$ echo "export LANG=en_US.UTF-8" >>~/.bash_profile
$ bash --login
```

If we preferred to use eg `fr_FR.utf8`, we might have to install that
first:

``` {.shell}
$ apt-get install language-pack-fr
$ locale -a
C
en_US.utf8
fr_BE.utf8
fr_CA.utf8
fr_CH.utf8
fr_FR.utf8
fr_LU.utf8
POSIX
$ LANG=fr_FR.utf8 hledger -f my.journal print
```

Note some platforms allow variant locale spellings, but not all (ubuntu
accepts `fr_FR.UTF8`, mac osx requires exactly `fr_FR.UTF-8`).


## hledger-ui

This doc is for version **1.2**. []{.docversions}

### NAME

hledger-ui - curses-style interface for the hledger accounting tool

### SYNOPSIS

`hledger-ui [OPTIONS] [QUERYARGS]`\
`hledger ui -- [OPTIONS] [QUERYARGS]`

### DESCRIPTION

hledger is a cross-platform program for tracking money, time, or any
other commodity, using double-entry accounting and a simple, editable
file format. hledger is inspired by and largely compatible with
ledger(1).

<style>
.highslide img {max-width:200px; border:0;}
.highslide-caption {color:white; background-color:black;}
</style>
<div style="float:right; max-width:200px; text-align:right;">

<a href="images/hledger-ui/hledger-ui-sample-acc2.png" class="highslide" onclick="return hs.expand(this)"><img src="images/hledger-ui/hledger-ui-sample-acc2.png" title="Accounts screen with query and depth limit" /></a>
<a href="images/hledger-ui/hledger-ui-sample-acc.png" class="highslide" onclick="return hs.expand(this)"><img src="images/hledger-ui/hledger-ui-sample-acc.png" title="Accounts screen" /></a>
<a href="images/hledger-ui/hledger-ui-sample-acc-greenterm.png" class="highslide" onclick="return hs.expand(this)"><img src="images/hledger-ui/hledger-ui-sample-acc-greenterm.png" title="Accounts screen with greenterm theme" /></a>
<a href="images/hledger-ui/hledger-ui-sample-txn.png" class="highslide" onclick="return hs.expand(this)"><img src="images/hledger-ui/hledger-ui-sample-txn.png" title="Transaction screen" /></a>
<a href="images/hledger-ui/hledger-ui-sample-reg.png" class="highslide" onclick="return hs.expand(this)"><img src="images/hledger-ui/hledger-ui-sample-reg.png" title="Register screen" /></a>
<!-- <br clear=all> -->
<a href="images/hledger-ui/hledger-ui-bcexample-acc.png" class="highslide" onclick="return hs.expand(this)"><img src="images/hledger-ui/hledger-ui-bcexample-acc.png" title="beancount example accounts" /></a>
<a href="images/hledger-ui/hledger-ui-bcexample-acc-etrade-cash.png" class="highslide" onclick="return hs.expand(this)"><img src="images/hledger-ui/hledger-ui-bcexample-acc-etrade-cash.png" title="beancount example's etrade cash subaccount" /></a>
<a href="images/hledger-ui/hledger-ui-bcexample-acc-etrade.png" class="highslide" onclick="return hs.expand(this)"><img src="images/hledger-ui/hledger-ui-bcexample-acc-etrade.png" title="beancount example's etrade investments, all commoditiess" /></a>

</div>

hledger-ui is hledger's curses-style interface, providing an efficient
full-window text UI for viewing accounts and transactions, and some
limited data entry capability. It is easier than hledger's command-line
interface, and sometimes quicker and more convenient than the web
interface.

Like hledger, it reads data from one or more files in hledger journal,
timeclock, timedot, or CSV format specified with `-f`, or
`$LEDGER_FILE`, or `$HOME/.hledger.journal` (on windows, perhaps
`C:/Users/USER/.hledger.journal`). For more about this see hledger(1),
hledger\_journal(5) etc.

### OPTIONS

Note: if invoking hledger-ui as a hledger subcommand, write `--` before
options as shown above.

Any QUERYARGS are interpreted as a hledger search query which filters
the data.

`--watch`
:   watch for data and date changes and reload automatically

`--theme=default|terminal|greenterm`
:   use this custom display theme

`--register=ACCTREGEX`
:   start in the (first) matched account's register screen

`--change`
:   show period balances (changes) at startup instead of historical
    balances

`--flat`
:   show full account names, unindented

hledger input options:

`-f FILE --file=FILE`
:   use a different input file. For stdin, use - (default:
    `$LEDGER_FILE` or `$HOME/.hledger.journal`)

`--rules-file=RULESFILE`
:   Conversion rules file to use when reading CSV (default: FILE.rules)

`--alias=OLD=NEW`
:   rename accounts named OLD to NEW

`--anon`
:   anonymize accounts and payees

`--pivot TAGNAME`
:   use some other field/tag for account names

`-I --ignore-assertions`
:   ignore any failing balance assertions

hledger reporting options:

`-b --begin=DATE`
:   include postings/txns on or after this date

`-e --end=DATE`
:   include postings/txns before this date

`-D --daily`
:   multiperiod/multicolumn report by day

`-W --weekly`
:   multiperiod/multicolumn report by week

`-M --monthly`
:   multiperiod/multicolumn report by month

`-Q --quarterly`
:   multiperiod/multicolumn report by quarter

`-Y --yearly`
:   multiperiod/multicolumn report by year

`-p --period=PERIODEXP`
:   set start date, end date, and/or reporting interval all at once
    (overrides the flags above)

`--date2`
:   show, and match with -b/-e/-p/date:, secondary dates instead

`-C --cleared`
:   include only cleared postings/txns

`--pending`
:   include only pending postings/txns

`-U --uncleared`
:   include only uncleared (and pending) postings/txns

`-R --real`
:   include only non-virtual postings

`--depth=N`
:   hide accounts/postings deeper than N

`-E --empty`
:   show items with zero amount, normally hidden

`-B --cost`
:   convert amounts to their cost at transaction time (using the
    [transaction price](journal.html#transaction-prices), if any)

`-V --value`
:   convert amounts to their market value on the report end date (using
    the most recent applicable [market
    price](journal.html#market-prices), if any)

hledger help options:

`-h`
:   show general usage (or after COMMAND, command usage)

`--help`
:   show this program's manual as plain text (or after an add-on
    COMMAND, the add-on's manual)

`--man`
:   show this program's manual with man

`--info`
:   show this program's manual with info

`--version`
:   show version

`--debug[=N]`
:   show debug output (levels 1-9, default: 1)

### KEYS

`?` shows a help dialog listing all keys. (Some of these also appear in
the quick help at the bottom of each screen.) Press `?` again (or
`ESCAPE`, or `LEFT`) to close it. The following keys work on most
screens:

The cursor keys navigate: `right` (or `enter`) goes deeper, `left`
returns to the previous screen,
`up`/`down`/`page up`/`page down`/`home`/`end` move up and down through
lists. Vi-style `h`/`j`/`k`/`l` movement keys are also supported. A tip:
movement speed is limited by your keyboard repeat rate, to move faster
you may want to adjust it. (If you're on a mac, the Karabiner app is one
way to do that.)

With shift pressed, the cursor keys adjust the report period, limiting
the transactions to be shown (by default, all are shown).
`shift-down/up` steps downward and upward through these standard report
period durations: year, quarter, month, week, day. Then,
`shift-left/right` moves to the previous/next period. `t` sets the
report period to today. With the `--watch` option, when viewing a
"current" period (the current day, week, month, quarter, or year), the
period will move automatically to track the current date. To set a
non-standard period, you can use `/` and a `date:` query.

`/` lets you set a general filter query limiting the data shown, using
the same [query terms](/hledger.html#queries) as in hledger and
hledger-web. While editing the query, you can use [CTRL-a/e/d/k, BS,
cursor
keys](http://hackage.haskell.org/package/brick-0.7/docs/Brick-Widgets-Edit.html#t:Editor);
press `ENTER` to set it, or `ESCAPE`to cancel. There are also keys for
quickly adjusting some common filters like account depth and
cleared/uncleared (see below). `BACKSPACE` or `DELETE` removes all
filters, showing all transactions.

`ESCAPE` removes all filters and jumps back to the top screen. Or, it
cancels a minibuffer edit or help dialog in progress.

`g` reloads from the data file(s) and updates the current screen and any
previous screens. (With large files, this could cause a noticeable
pause.)

`I` toggles balance assertion checking. Disabling balance assertions
temporarily can be useful for troubleshooting.

`a` runs command-line hledger's add command, and reloads the updated
file. This allows some basic data entry.

`E` runs \$HLEDGER\_UI\_EDITOR, or \$EDITOR, or a default
(`emacsclient -a "" -nw`) on the journal file. With some editors (emacs,
vi), the cursor will be positioned at the current transaction when
invoked from the register and transaction screens, and at the error
location (if possible) when invoked from the error screen.

`q` quits the application.

Additional screen-specific keys are described below.

### SCREENS

#### Accounts screen

This is normally the first screen displayed. It lists accounts and their
balances, like hledger's balance command. By default, it shows all
accounts and their latest ending balances (including the balances of
subaccounts). if you specify a query on the command line, it shows just
the matched accounts and the balances from matched transactions.

Account names are normally indented to show the hierarchy (tree mode).
To see less detail, set a depth limit by pressing a number key, `1` to
`9`. `0` shows even less detail, collapsing all accounts to a single
total. `-` and `+` (or `=`) decrease and increase the depth limit. To
remove the depth limit, set it higher than the maximum account depth, or
press `ESCAPE`.

`F` toggles flat mode, in which accounts are shown as a flat list, with
their full names. In this mode, account balances exclude subaccounts,
except for accounts at the depth limit (as with hledger's balance
command).

`H` toggles between showing historical balances or period balances.
Historical balances (the default) are ending balances at the end of the
report period, taking into account all transactions before that date
(filtered by the filter query if any), including transactions before the
start of the report period. In other words, historical balances are what
you would see on a bank statement for that account (unless disturbed by
a filter query). Period balances ignore transactions before the report
start date, so they show the change in balance during the report period.
They are more useful eg when viewing a time log.

`C` toggles cleared mode, in which [uncleared transactions and
postings](/journal.html#transactions) are not shown. `U` toggles
uncleared mode, in which only uncleared transactions/postings are shown.

`R` toggles real mode, in which [virtual
postings](/journal.html#virtual-postings) are ignored.

`Z` toggles nonzero mode, in which only accounts with nonzero balances
are shown (hledger-ui shows zero items by default, unlike command-line
hledger).

Press `right` or `enter` to view an account's transactions register.

#### Register screen

This screen shows the transactions affecting a particular account, like
a check register. Each line represents one transaction and shows:

-   the other account(s) involved, in abbreviated form. (If there are
    both real and virtual postings, it shows only the accounts affected
    by real postings.)

-   the overall change to the current account's balance; positive for an
    inflow to this account, negative for an outflow.

-   the running historical total or period total for the current
    account, after the transaction. This can be toggled with `H`.
    Similar to the accounts screen, the historical total is affected by
    transactions (filtered by the filter query) before the report start
    date, while the period total is not. If the historical total is not
    disturbed by a filter query, it will be the running historical
    balance you would see on a bank register for the current account.

If the accounts screen was in tree mode, the register screen will
include transactions from both the current account and its subaccounts.
If the accounts screen was in flat mode, and a non-depth-clipped account
was selected, the register screen will exclude transactions from
subaccounts. In other words, the register always shows the transactions
responsible for the period balance shown on the accounts screen. As on
the accounts screen, this can be toggled with `F`.

`C` toggles cleared mode, in which [uncleared transactions and
postings](/journal.html#transactions) are not shown. `U` toggles
uncleared mode, in which only uncleared transactions/postings are shown.

`R` toggles real mode, in which [virtual
postings](/journal.html#virtual-postings) are ignored.

`Z` toggles nonzero mode, in which only transactions posting a nonzero
change are shown (hledger-ui shows zero items by default, unlike
command-line hledger).

Press `right` (or `enter`) to view the selected transaction in detail.

#### Transaction screen

This screen shows a single transaction, as a general journal entry,
similar to hledger's print command and journal format
(hledger\_journal(5)).

The transaction's date(s) and any cleared flag, transaction code,
description, comments, along with all of its account postings are shown.
Simple transactions have two postings, but there can be more (or in
certain cases, fewer).

`up` and `down` will step through all transactions listed in the
previous account register screen. In the title bar, the numbers in
parentheses show your position within that account register. They will
vary depending on which account register you came from (remember most
transactions appear in multiple account registers). The \#N number
preceding them is the transaction's position within the complete
unfiltered journal, which is a more stable id (at least until the next
reload).

#### Error screen

This screen will appear if there is a problem, such as a parse error,
when you press g to reload. Once you have fixed the problem, press g
again to reload and resume normal operation. (Or, you can press escape
to cancel the reload attempt.)

### ENVIRONMENT

**COLUMNS** The screen width to use. Default: the full terminal width.

**LEDGER\_FILE** The journal file path when not specified with `-f`.
Default: `~/.hledger.journal` (on windows, perhaps
`C:/Users/USER/.hledger.journal`).

### FILES

Reads data from one or more files in hledger journal, timeclock,
timedot, or CSV format specified with `-f`, or `$LEDGER_FILE`, or
`$HOME/.hledger.journal` (on windows, perhaps
`C:/Users/USER/.hledger.journal`).

### BUGS

The need to precede options with `--` when invoked from hledger is
awkward.

`-f-` doesn't work (hledger-ui can't read from stdin).

`-V` affects only the accounts screen.

When you press `g`, the current and all previous screens are
regenerated, which may cause a noticeable pause with large files. Also
there is no visual indication that this is in progress.

`--watch` is not yet fully robust. It works well for normal usage, but
many file changes in a short time (eg saving the file thousands of times
with an editor macro) can cause problems at least on OSX. Symptoms
include: unresponsive UI, periodic resetting of the cursor position,
momentary display of parse errors, high CPU usage eventually subsiding,
and possibly a small but persistent build-up of CPU usage until the
program is restarted.


## hledger-web

This doc is for version **1.2**. []{.docversions}

### NAME

hledger-web - web interface for the hledger accounting tool

### SYNOPSIS

`hledger-web [OPTIONS]`\
`hledger web -- [OPTIONS]`

### DESCRIPTION

hledger is a cross-platform program for tracking money, time, or any
other commodity, using double-entry accounting and a simple, editable
file format. hledger is inspired by and largely compatible with
ledger(1).

<style>
.highslide img {max-width:200px; border:thin grey solid; margin:0 0 1em 1em; }
.highslide-caption {color:white; background-color:black;}
</style>
<div style="float:right; max-width:200px; text-align:right;">

<a href="images/hledger-web/normal/register.png" class="highslide" onclick="return hs.expand(this)"><img src="images/hledger-web/normal/register.png" title="Account register view with accounts sidebar" /></a>
<a href="images/hledger-web/normal/journal.png" class="highslide" onclick="return hs.expand(this)"><img src="images/hledger-web/normal/journal.png" title="Journal view" /></a>
<a href="images/hledger-web/normal/help.png" class="highslide" onclick="return hs.expand(this)"><img src="images/hledger-web/normal/help.png" title="Help dialog" /></a>
<a href="images/hledger-web/normal/add.png" class="highslide" onclick="return hs.expand(this)"><img src="images/hledger-web/normal/add.png" title="Add form" /></a>

</div>

hledger-web is hledger's web interface. It starts a simple web
application for browsing and adding transactions, and optionally opens
it in a web browser window if possible. It provides a more user-friendly
UI than the hledger CLI or hledger-ui interface, showing more at once
(accounts, the current account register, balance charts) and allowing
history-aware data entry, interactive searching, and bookmarking.

hledger-web also lets you share a ledger with multiple users, or even
the public web. There is no access control, so if you need that you
should put it behind a suitable web proxy. As a small protection against
data loss when running an unprotected instance, it writes a numbered
backup of the main journal file (only ?) on every edit.

Like hledger, it reads data from one or more files in hledger journal,
timeclock, timedot, or CSV format specified with `-f`, or
`$LEDGER_FILE`, or `$HOME/.hledger.journal` (on windows, perhaps
`C:/Users/USER/.hledger.journal`). For more about this see hledger(1),
hledger\_journal(5) etc.

By default, hledger-web starts the web app in "transient mode" and also
opens it in your default web browser if possible. In this mode the web
app will keep running for as long as you have it open in a browser
window, and will exit after two minutes of inactivity (no requests and
no browser windows viewing it). With `--serve`, it just runs the web app
without exiting, and logs requests to the console.

By default the server listens on IP address 127.0.0.1, accessible only
to local requests. You can use `--host` to change this, eg
`--host 0.0.0.0` to listen on all configured addresses.

Similarly, use `--port` to set a TCP port other than 5000, eg if you are
running multiple hledger-web instances.

You can use `--base-url` to change the protocol, hostname, port and path
that appear in hyperlinks, useful eg for integrating hledger-web within
a larger website. The default is `http://HOST:PORT/` using the server's
configured host address and TCP port (or `http://HOST` if PORT is 80).

With `--file-url` you can set a different base url for static files, eg
for better caching or cookie-less serving on high performance websites.

Note there is no built-in access control (aside from listening on
127.0.0.1 by default). So you will need to hide hledger-web behind an
authenticating proxy (such as apache or nginx) if you want to restrict
who can see and add entries to your journal.

Command-line options and arguments may be used to set an initial filter
on the data. This is not shown in the web UI, but it will be applied in
addition to any search query entered there.

With journal and timeclock files (but not CSV files, currently) the web
app detects changes made by other means and will show the new data on
the next request. If a change makes the file unparseable, hledger-web
will show an error until the file has been fixed.

### OPTIONS

Note: if invoking hledger-web as a hledger subcommand, write `--` before
options as shown above.

`--serve`
:   serve and log requests, don't browse or auto-exit

`--host=IPADDR`
:   listen on this IP address (default: 127.0.0.1)

`--port=PORT`
:   listen on this TCP port (default: 5000)

`--base-url=URL`
:   set the base url (default: http://IPADDR:PORT). You would change
    this when sharing over the network, or integrating within a larger
    website.

`--file-url=URL`
:   set the static files url (default: BASEURL/static). hledger-web
    normally serves static files itself, but if you wanted to serve them
    from another server for efficiency, you would set the url with this.

hledger input options:

`-f FILE --file=FILE`
:   use a different input file. For stdin, use - (default:
    `$LEDGER_FILE` or `$HOME/.hledger.journal`)

`--rules-file=RULESFILE`
:   Conversion rules file to use when reading CSV (default: FILE.rules)

`--alias=OLD=NEW`
:   rename accounts named OLD to NEW

`--anon`
:   anonymize accounts and payees

`--pivot TAGNAME`
:   use some other field/tag for account names

`-I --ignore-assertions`
:   ignore any failing balance assertions

hledger reporting options:

`-b --begin=DATE`
:   include postings/txns on or after this date

`-e --end=DATE`
:   include postings/txns before this date

`-D --daily`
:   multiperiod/multicolumn report by day

`-W --weekly`
:   multiperiod/multicolumn report by week

`-M --monthly`
:   multiperiod/multicolumn report by month

`-Q --quarterly`
:   multiperiod/multicolumn report by quarter

`-Y --yearly`
:   multiperiod/multicolumn report by year

`-p --period=PERIODEXP`
:   set start date, end date, and/or reporting interval all at once
    (overrides the flags above)

`--date2`
:   show, and match with -b/-e/-p/date:, secondary dates instead

`-C --cleared`
:   include only cleared postings/txns

`--pending`
:   include only pending postings/txns

`-U --uncleared`
:   include only uncleared (and pending) postings/txns

`-R --real`
:   include only non-virtual postings

`--depth=N`
:   hide accounts/postings deeper than N

`-E --empty`
:   show items with zero amount, normally hidden

`-B --cost`
:   convert amounts to their cost at transaction time (using the
    [transaction price](journal.html#transaction-prices), if any)

`-V --value`
:   convert amounts to their market value on the report end date (using
    the most recent applicable [market
    price](journal.html#market-prices), if any)

hledger help options:

`-h`
:   show general usage (or after COMMAND, command usage)

`--help`
:   show this program's manual as plain text (or after an add-on
    COMMAND, the add-on's manual)

`--man`
:   show this program's manual with man

`--info`
:   show this program's manual with info

`--version`
:   show version

`--debug[=N]`
:   show debug output (levels 1-9, default: 1)

### ENVIRONMENT

**LEDGER\_FILE** The journal file path when not specified with `-f`.
Default: `~/.hledger.journal` (on windows, perhaps
`C:/Users/USER/.hledger.journal`).

### FILES

Reads data from one or more files in hledger journal, timeclock,
timedot, or CSV format specified with `-f`, or `$LEDGER_FILE`, or
`$HOME/.hledger.journal` (on windows, perhaps
`C:/Users/USER/.hledger.journal`).

### BUGS

The need to precede options with `--` when invoked from hledger is
awkward.

`-f-` doesn't work (hledger-web can't read from stdin).

Query arguments and some hledger options are ignored.

Does not work in text-mode browsers.

Does not work well on small screens.


## hledger-api

This doc is for version **1.2**. []{.docversions}

### NAME

hledger-api - web API server for the hledger accounting tool

### SYNOPSIS

`hledger-api [OPTIONS]`\
`hledger api -- [OPTIONS]`

### DESCRIPTION

hledger is a cross-platform program for tracking money, time, or any
other commodity, using double-entry accounting and a simple, editable
file format. hledger is inspired by and largely compatible with
ledger(1).

hledger-api is a simple web API server, intended to support client-side
web apps operating on hledger data. It comes with a series of simple
client-side app examples, which drive its evolution.

Like hledger, it reads data from one or more files in hledger journal,
timeclock, timedot, or CSV format specified with `-f`, or
`$LEDGER_FILE`, or `$HOME/.hledger.journal` (on windows, perhaps
`C:/Users/USER/.hledger.journal`). For more about this see hledger(1),
hledger\_journal(5) etc.

The server listens on IP address 127.0.0.1, accessible only to local
requests, by default. You can change this with `--host`, eg
`--host 0.0.0.0` to listen on all addresses. Note there is no other
access control, so you will need to hide hledger-api behind an
authenticating proxy if you want to restrict access. You can change the
TCP port (default: 8001) with `-p PORT`.

If invoked as `hledger-api --swagger`, instead of starting a server the
API docs will be printed in Swagger 2.0 format.

### OPTIONS

Note: if invoking hledger-api as a hledger subcommand, write `--` before
options as shown above.

`-f --file=FILE`
:   use a different input file. For stdin, use - (default:
    `$LEDGER_FILE` or `$HOME/.hledger.journal`)

`-d --static-dir=DIR`
:   serve files from a different directory (default: `.`)

`--host=IPADDR`
:   listen on this IP address (default: 127.0.0.1)

`-p --port=PORT`
:   listen on this TCP port (default: 8001)

`--swagger`
:   print API docs in Swagger 2.0 format, and exit

`--version`
:   show version

`-h`
:   show usage

`--help`
:   show manual as plain text

`--man`
:   show manual with man

`--info`
:   show manual with info

### ENVIRONMENT

**LEDGER\_FILE** The journal file path when not specified with `-f`.
Default: `~/.hledger.journal` (on windows, perhaps
`C:/Users/USER/.hledger.journal`).

### FILES

Reads data from one or more files in hledger journal, timeclock,
timedot, or CSV format specified with `-f`, or `$LEDGER_FILE`, or
`$HOME/.hledger.journal` (on windows, perhaps
`C:/Users/USER/.hledger.journal`).

### BUGS

The need to precede options with `--` when invoked from hledger is
awkward.


## journal format

This doc is for version **1.2**. []{.docversions}

### NAME

Journal - hledger's default file format, representing a General Journal

### DESCRIPTION

hledger's usual data source is a plain text file containing journal
entries in hledger journal format. This file represents a standard
accounting [general
journal](http://en.wikipedia.org/wiki/General_journal). I use file names
ending in `.journal`, but that's not required. The journal file contains
a number of transaction entries, each describing a transfer of money (or
any commodity) between two or more named accounts, in a simple format
readable by both hledger and humans.

hledger's journal format is a compatible subset,
[mostly](faq.html#file-format-differences), of [ledger's journal
format](http://ledger-cli.org/3.0/doc/ledger3.html#Journal-Format), so
hledger can work with [compatible](faq.html#file-format-differences)
ledger journal files as well. It's safe, and encouraged, to run both
hledger and ledger on the same journal file, eg to validate the results
you're getting.

You can use hledger without learning any more about this file; just use
the [add](#add) or [web](#web) commands to create and update it. Many
users, though, also edit the journal file directly with a text editor,
perhaps assisted by the helper modes for emacs or vim.

Here's an example:

``` {.journal}
; A sample journal file. This is a comment.

2008/01/01 income               ; <- transaction's first line starts in column 0, contains date and description
    assets:bank:checking  $1    ; <- posting lines start with whitespace, each contains an account name
    income:salary        $-1    ;    followed by at least two spaces and an amount

2008/06/01 gift
    assets:bank:checking  $1    ; <- at least two postings in a transaction
    income:gifts         $-1    ; <- their amounts must balance to 0

2008/06/02 save
    assets:bank:saving    $1
    assets:bank:checking        ; <- one amount may be omitted; here $-1 is inferred

2008/06/03 eat & shop           ; <- description can be anything
    expenses:food         $1
    expenses:supplies     $1    ; <- this transaction debits two expense accounts
    assets:cash                 ; <- $-2 inferred

2008/12/31 * pay off            ; <- an optional * or ! after the date means "cleared" (or anything you want)
    liabilities:debts     $1
    assets:bank:checking
```

### FILE FORMAT

<!-- Now let's explore the available journal file syntax in detail. -->
#### Transactions

Transactions are represented by journal entries. Each begins with a
[simple date](#simple-dates) in column 0, followed by three optional
fields with spaces between them:

-   a status flag, which can be empty or `!` or `*` (meaning
    "uncleared", "pending" and "cleared", or whatever you want)
-   a transaction code (eg a check number),
-   and/or a description

then some number of postings, of some amount to some account. Each
posting is on its own line, consisting of:

-   indentation of one or more spaces (or tabs)
-   optionally, a `!` or `*` status flag followed by a space
-   an account name, optionally containing single spaces
-   optionally, two or more spaces or tabs followed by an amount

Usually there are two or more postings, though one or none is also
possible. The posting amounts within a transaction must always balance,
ie add up to 0. Optionally one amount can be left blank, in which case
it will be inferred.

#### Dates

##### Simple dates

Within a journal file, transaction dates use Y/M/D (or Y-M-D or Y.M.D)
Leading zeros are optional. The year may be omitted, in which case it
will be inferred from the context - the current transaction, the default
year set with a [default year directive](#default-year), or the current
date when the command is run. Some examples: `2010/01/31`, `1/31`,
`2010-01-31`, `2010.1.31`.

##### Secondary dates

Real-life transactions sometimes involve more than one date - eg the
date you write a cheque, and the date it clears in your bank. When you
want to model this, eg for more accurate balances, you can specify
individual [posting dates](#posting-dates), which I recommend. Or, you
can use the secondary dates (aka auxiliary/effective dates) feature,
supported for compatibility with Ledger.

A secondary date can be written after the primary date, separated by an
equals sign. The primary date, on the left, is used by default; the
secondary date, on the right, is used when the `--date2` flag is
specified (`--aux-date` or `--effective` also work).

The meaning of secondary dates is up to you, but it's best to follow a
consistent rule. Eg write the bank's clearing date as primary, and when
needed, the date the transaction was initiated as secondary.

Here's an example. Note that a secondary date will use the year of the
primary date if unspecified.

``` {.journal}
2010/2/23=2/19 movie ticket
  expenses:cinema                   $10
  assets:checking
```

``` {.shell}
$ hledger register checking
2010/02/23 movie ticket         assets:checking                $-10         $-10
```

``` {.shell}
$ hledger register checking --date2
2010/02/19 movie ticket         assets:checking                $-10         $-10
```

Secondary dates require some effort; you must use them consistently in
your journal entries and remember whether to use or not use the
`--date2` flag for your reports. They are included in hledger for Ledger
compatibility, but posting dates are a more powerful and less confusing
alternative.

##### Posting dates

You can give individual postings a different date from their parent
transaction, by adding a [posting comment](#comments) containing a
[tag](#tags) (see below) like `date:DATE`. This is probably the best way
to control posting dates precisely. Eg in this example the expense
should appear in May reports, and the deduction from checking should be
reported on 6/1 for easy bank reconciliation:

``` {.journal}
2015/5/30
    expenses:food     $10   ; food purchased on saturday 5/30
    assets:checking         ; bank cleared it on monday, date:6/1
```

``` {.shell}
$ hledger -f t.j register food
2015/05/30                      expenses:food                  $10           $10
```

``` {.shell}
$ hledger -f t.j register checking
2015/06/01                      assets:checking               $-10          $-10
```

DATE should be a [simple date](#simple-dates); if the year is not
specified it will use the year of the transaction's date. You can set
the secondary date similarly, with `date2:DATE2`. The `date:` or
`date2:` tags must have a valid simple date value if they are present,
eg a `date:` tag with no value is not allowed.

Ledger's earlier, more compact bracketed date syntax is also supported:
`[DATE]`, `[DATE=DATE2]` or `[=DATE2]`. hledger will attempt to parse
any square-bracketed sequence of the `0123456789/-.=` characters in this
way. With this syntax, DATE infers its year from the transaction and
DATE2 infers its year from DATE.

#### Account names

Account names typically have several parts separated by a full colon,
from which hledger derives a hierarchical chart of accounts. They can be
anything you like, but in finance there are traditionally five top-level
accounts: `assets`, `liabilities`, `income`, `expenses`, and `equity`.

Account names may contain single spaces, eg:
`assets:accounts receivable`. Because of this, they must always be
followed by **two or more spaces** (or newline).

Account names can be [aliased](#account-aliases).

#### Amounts

After the account name, there is usually an amount. Important: between
account name and amount, there must be **two or more spaces**.

Amounts consist of a number and (usually) a currency symbol or commodity
name. Some examples:

`2.00001`\
`$1`\
`4000 AAPL`\
`3 "green apples"`\
`-$1,000,000.00`\
`INR 9,99,99,999.00`\
`EUR -2.000.000,00`

As you can see, the amount format is somewhat flexible:

-   amounts are a number (the "quantity") and optionally a currency
    symbol/commodity name (the "commodity").
-   the commodity is a symbol, word, or phrase, on the left or right,
    with or without a separating space. If the commodity contains
    numbers, spaces or non-word punctuation it must be enclosed in
    double quotes.
-   negative amounts with a commodity on the left can have the minus
    sign before or after it
-   digit groups (thousands, or any other grouping) can be separated by
    commas (in which case period is used for decimal point) or periods
    (in which case comma is used for decimal point)

You can use any of these variations when recording data, but when
hledger displays amounts, it will choose a consistent format for each
commodity. (Except for [price amounts](#prices), which are always
formatted as written). The display format is chosen as follows:

-   if there is a [commodity directive](#commodity-directive) specifying
    the format, that is used
-   otherwise the format is inferred from the first posting amount in
    that commodity in the journal, and the precision (number of decimal
    places) will be the maximum from all posting amounts in that
    commmodity
-   or if there are no such amounts in the journal, a default format is
    used (like `$1000.00`).

Price amounts and amounts in D directives usually don't affect amount
format inference, but in some situations they can do so indirectly. (Eg
when D's default commodity is applied to a commodity-less amount, or
when an amountless posting is balanced using a price's commodity, or
when -V is used.) If you find this causing problems, set the desired
format with a commodity directive.

#### Virtual Postings

When you parenthesise the account name in a posting, we call that a
*virtual posting*, which means:

-   it is ignored when checking that the transaction is balanced
-   it is excluded from reports when the `--real/-R` flag is used, or
    the `real:1` query.

You could use this, eg, to set an account's opening balance without
needing to use the `equity:opening balances` account:

``` {.journal}
1/1 special unbalanced posting to set initial balance
  (assets:checking)   $1000
```

When the account name is bracketed, we call it a *balanced virtual
posting*. This is like an ordinary virtual posting except the balanced
virtual postings in a transaction must balance to 0, like the real
postings (but separately from them). Balanced virtual postings are also
excluded by `--real/-R` or `real:1`.

``` {.journal}
1/1 buy food with cash, and update some budget-tracking subaccounts elsewhere
  expenses:food                   $10
  assets:cash                    $-10
  [assets:checking:available]     $10
  [assets:checking:budget:food]  $-10
```

Virtual postings have some legitimate uses, but those are few. You can
usually find an equivalent journal entry using real postings, which is
more correct and provides better error checking.

#### Balance Assertions

hledger supports [Ledger-style balance
assertions](http://ledger-cli.org/3.0/doc/ledger3.html#Balance-assertions)
in journal files. These look like `=EXPECTEDBALANCE` following a
posting's amount. Eg in this example we assert the expected dollar
balance in accounts a and b after each posting:

``` {.journal}
2013/1/1
  a   $1  =$1
  b       =$-1

2013/1/2
  a   $1  =$2
  b  $-1  =$-2
```

After reading a journal file, hledger will check all balance assertions
and report an error if any of them fail. Balance assertions can protect
you from, eg, inadvertently disrupting reconciled balances while
cleaning up old entries. You can disable them temporarily with the
`--ignore-assertions` flag, which can be useful for troubleshooting or
for reading Ledger files.

##### Assertions and ordering

hledger sorts an account's postings and assertions first by date and
then (for postings on the same day) by parse order. Note this is
different from Ledger, which sorts assertions only by parse order.
(Also, Ledger assertions do not see the accumulated effect of repeated
postings to the same account within a transaction.)

So, hledger balance assertions keep working if you reorder
differently-dated transactions within the journal. But if you reorder
same-dated transactions or postings, assertions might break and require
updating. This order dependence does bring an advantage: precise control
over the order of postings and assertions within a day, so you can
assert intra-day balances.

##### Assertions and included files

With [included files](#including-other-files), things are a little more
complicated. Including preserves the ordering of postings and
assertions. If you have multiple postings to an account on the same day,
split across different files, and you also want to assert the account's
balance on the same day, you'll have to put the assertion in the right
file.

##### Assertions and multiple -f options

Balance assertions don't work well across files specified with multiple
-f options. Use include or [concatenate the
files](/hledger.html#input-files) instead.

##### Assertions and commodities

The asserted balance must be a simple single-commodity amount, and in
fact the assertion checks only this commodity's balance within the
(possibly multi-commodity) account balance. We could call this a partial
balance assertion. This is compatible with Ledger, and makes it possible
to make assertions about accounts containing multiple commodities.

To assert each commodity's balance in such a multi-commodity account,
you can add multiple postings (with amount 0 if necessary). But note
that no matter how many assertions you add, you can't be sure the
account does not contain some unexpected commodity. (We'll add support
for this kind of total balance assertion if there's demand.)

##### Assertions and subaccounts

Balance assertions do not count the balance from subaccounts; they check
the posted account's exclusive balance. For example:

``` {.journal}
1/1
  checking:fund   1 = 1  ; post to this subaccount, its balance is now 1
  checking        1 = 1  ; post to the parent account, its exclusive balance is now 1
  equity
```

The balance report's flat mode shows these exclusive balances more
clearly:

``` {.shell}
$ hledger bal checking --flat
                   1  checking
                   1  checking:fund
--------------------
                   2
```

##### Assertions and virtual postings

Balance assertions are checked against all postings, both real and
[virtual](#virtual-postings). They are not affected by the `--real/-R`
flag or `real:` query.

#### Balance Assignments

[Ledger-style balance
assignments](http://ledger-cli.org/3.0/doc/ledger3.html#Balance-assignments)
are also supported. These are like [balance
assertions](#balance-assertions), but with no posting amount on the left
side of the equals sign; instead it is calculated automatically so as to
satisfy the assertion. This can be a convenience during data entry, eg
when setting opening balances:

``` {.journal}
; starting a new journal, set asset account balances 
2016/1/1 opening balances
  assets:checking            = $409.32
  assets:savings             = $735.24
  assets:cash                 = $42
  equity:opening balances
```

or when adjusting a balance to reality:

``` {.journal}
; no cash left; update balance, record any untracked spending as a generic expense
2016/1/15
  assets:cash    = $0
  expenses:misc
```

The calculated amount depends on the account's balance in the commodity
at that point (which depends on the previously-dated postings of the
commodity to that account since the last balance assertion or
assignment). Note that using balance assignments makes your journal a
little less explicit; to know the exact amount posted, you have to run
hledger or do the calculations yourself, instead of just reading it.

#### Prices

##### Transaction prices

Within a transaction posting, you can record an amount's price in
another commodity. This can be used to document the cost (for a
purchase), or selling price (for a sale), or the exchange rate that was
used, for this transaction. These transaction prices are fixed, and do
not change over time. <!--
This is different from Ledger, where transaction prices fluctuate by
default.  Ledger has a different syntax for specifying
[fixed prices](http://ledger-cli.org/3.0/doc/ledger3.html#Fixing-Lot-Prices):
`{=PRICE}`.  hledger parses that syntax, and (currently) ignores it.
-->
<!-- hledger treats this as an alternate spelling of `@ PRICE`, for greater compatibility with Ledger files. -->

Amounts with transaction prices can be displayed in the transaction
price's commodity, by using the
[`--cost/-B`](hledger.html#reporting-options) flag supported by most
hledger commands (mnemonic: "cost Basis").

There are several ways to record a transaction price:

1.  Write the unit price (aka exchange rate), as `@ UNITPRICE` after the
    amount:

    ``` {.journal}
    2009/1/1
      assets:foreign currency   €100 @ $1.35  ; one hundred euros at $1.35 each
      assets:cash
    ```

2.  Or write the total price, as `@@ TOTALPRICE` after the amount:

    ``` {.journal}
    2009/1/1
      assets:foreign currency   €100 @@ $135  ; one hundred euros at $135 for the lot
      assets:cash
    ```

3.  Or let hledger infer the price so as to balance the transaction. To
    permit this, you must fully specify all posting amounts, and their
    sum must have a non-zero amount in exactly two commodities:

    ``` {.journal}
    2009/1/1
      assets:foreign currency   €100          ; one hundred euros
      assets:cash              $-135          ; exchanged for $135
    ```

With any of the above examples we get:

``` {.shell}
$ hledger print -B
2009/01/01
    assets:foreign currency       $135.00
    assets:cash                  $-135.00
```

Example use for transaction prices: recording the effective conversion
rate of purchases made in a foreign currency.

##### Market prices

Market prices are not tied to a particular transaction; they represent
historical exchange rates between two commodities. (Ledger calls them
historical prices.) For example, the prices published by a [stock
exchange](https://en.wikipedia.org/wiki/Stock_exchange) or the [foreign
exchange market](https://en.wikipedia.org/wiki/Foreign_exchange_market).
Some commands ([balance](hledger.html#market-value), currently) can use
this information to show the market value of things at a given date.

To record market prices, use P directives in the main journal or in an
[included](#including-other-files) file. Their format is:

``` {.journal}
P DATE COMMODITYBEINGPRICED UNITPRICE
```

<!-- (A time and numeric time zone are allowed but ignored, like ledger.) -->
DATE is a [simple date](#simple-dates) as usual. COMMODITYBEINGPRICED is
the symbol of the commodity being priced. UNITPRICE is an ordinary
[amount](#amounts) (symbol and quantity) in a second commodity,
specifying the unit price or conversion rate for the first commodity in
terms of the second, on the given date.

For example, the following directives say that one euro was worth 1.35
US dollars during 2009, and \$1.40 from 2010 onward:

``` {.journal}
P 2009/1/1 € $1.35
P 2010/1/1 € $1.40
```

#### Comments

Lines in the journal beginning with a semicolon (`;`) or hash (`#`) or
asterisk (`*`) are comments, and will be ignored. (Asterisk comments
make it easy to treat your journal like an org-mode outline in emacs.)

Also, anything between [`comment` and `end comment`
directives](#multi-line-comments) is a (multi-line) comment. If there is
no `end comment`, the comment extends to the end of the file.

You can attach comments to a transaction by writing them after the
description and/or indented on the following lines (before the
postings). Similarly, you can attach comments to an individual posting
by writing them after the amount and/or indented on the following lines.

Some examples:

``` {.journal}
# a journal comment

; also a journal comment

comment
This is a multiline comment,
which continues until a line
where the "end comment" string
appears on its own.
end comment

2012/5/14 something  ; a transaction comment
    ; the transaction comment, continued
    posting1  1  ; a comment for posting 1
    posting2
    ; a comment for posting 2
    ; another comment line for posting 2
; a journal comment (because not indented)
```

#### Tags

Tags are a way to add extra labels or labelled data to postings and
transactions, which you can then [search](/journal.html#queries) or
[pivot](/hledger.html#pivoting) on.

A simple tag is a word (which may contain hyphens) followed by a full
colon, written inside a transaction or posting [comment](#comments)
line:

``` {.journal}
2017/1/16 bought groceries    ; sometag:
```

Tags can have a value, which is the text after the colon, up to the next
comma or end of line, with leading/trailing whitespace removed:

``` {.journal}
    expenses:food    $10   ; a-posting-tag: the tag value
```

Note this means hledger's tag values can not contain commas or newlines.
Ending at commas means you can write multiple short tags on one line,
comma separated:

``` {.journal}
    assets:checking       ; a comment containing tag1:, tag2: some value ...
```

Here,

-   "`a comment containing`" is just comment text, not a tag
-   "`tag1`" is a tag with no value
-   "`tag2`" is another tag, whose value is "`some value ...`"

Tags in a transaction comment affect the transaction and all of its
postings, while tags in a posting comment affect only that posting. For
example, the following transaction has three tags (`A`, `TAG2`,
`third-tag`) and the posting has four (those plus `posting-tag`):

``` {.journal}
1/1 a transaction  ; A:, TAG2:
    ; third-tag: a third transaction tag, <- with a value
    (a)  $1  ; posting-tag:
```

Tags are like Ledger's
[metadata](http://ledger-cli.org/3.0/doc/ledger3.html#Metadata) feature,
except hledger's tag values are simple strings.

##### Implicit tags

Some predefined "implicit" tags are also provided:

-   `code` - the transaction's code field
-   `description` - the transaction's description
-   `payee` - the part of description before `|`, or all of it
-   `note` - the part of description after `|`, or all of it

`payee` and `note` support descriptions written in a special
`PAYEE | NOTE` format, accessing the parts before and after the pipe
character respectively. For descriptions not containing a pipe character
they are the same as `description`.

#### Directives

##### Account aliases

You can define aliases which rewrite your account names (after reading
the journal, before generating reports). hledger's account aliases can
be useful for:

-   expanding shorthand account names to their full form, allowing
    easier data entry and a less verbose journal
-   adapting old journals to your current chart of accounts
-   experimenting with new account organisations, like a new hierarchy
    or combining two accounts into one
-   customising reports

See also [Cookbook: rewrite account names](account-aliases.html).

###### Basic aliases

To set an account alias, use the `alias` directive in your journal file.
This affects all subsequent journal entries in the current file or its
[included files](#including-other-files). The spaces around the = are
optional:

``` {.journal}
alias OLD = NEW
```

Or, you can use the `--alias 'OLD=NEW'` option on the command line. This
affects all entries. It's useful for trying out aliases interactively.

OLD and NEW are full account names. hledger will replace any occurrence
of the old account name with the new one. Subaccounts are also affected.
Eg:

``` {.journal}
alias checking = assets:bank:wells fargo:checking
# rewrites "checking" to "assets:bank:wells fargo:checking", or "checking:a" to "assets:bank:wells fargo:checking:a"
```

###### Regex aliases

There is also a more powerful variant that uses a regular expression,
indicated by the forward slashes. (This was the default behaviour in
hledger 0.24-0.25):

``` {.journal}
alias /REGEX/ = REPLACEMENT
```

or `--alias '/REGEX/=REPLACEMENT'`.

<!-- (Can also be written `'/REGEX/REPLACEMENT/'`). -->
REGEX is a case-insensitive regular expression. Anywhere it matches
inside an account name, the matched part will be replaced by
REPLACEMENT. If REGEX contains parenthesised match groups, these can be
referenced by the usual numeric backreferences in REPLACEMENT. Note,
currently regular expression aliases may cause noticeable slow-downs.
(And if you use Ledger on your hledger file, they will be ignored.) Eg:

``` {.journal}
alias /^(.+):bank:([^:]+)(.*)/ = \1:\2 \3
# rewrites "assets:bank:wells fargo:checking" to  "assets:wells fargo checking"
```

###### Multiple aliases

You can define as many aliases as you like using directives or
command-line options. Aliases are recursive - each alias sees the result
of applying previous ones. (This is different from Ledger, where aliases
are non-recursive by default). Aliases are applied in the following
order:

1.  alias directives, most recently seen first (recent directives take
    precedence over earlier ones; directives not yet seen are ignored)
2.  alias options, in the order they appear on the command line

###### end aliases

You can clear (forget) all currently defined aliases with the
`end aliases` directive:

``` {.journal}
end aliases
```

##### account directive

The `account` directive predefines account names, as in Ledger and
Beancount. This may be useful for your own documentation; hledger
doesn't make use of it yet.

``` {.journal}
; account ACCT
;   OPTIONAL COMMENTS/TAGS...

account assets:bank:checking
 a comment
 acct-no:12345

account expenses:food

; etc.
```

##### apply account directive

You can specify a parent account which will be prepended to all accounts
within a section of the journal. Use the `apply account` and
`end apply account` directives like so:

``` {.journal}
apply account home

2010/1/1
    food    $10
    cash

end apply account
```

which is equivalent to:

``` {.journal}
2010/01/01
    home:food           $10
    home:cash          $-10
```

If `end apply account` is omitted, the effect lasts to the end of the
file. Included files are also affected, eg:

``` {.journal}
apply account business
include biz.journal
end apply account
apply account personal
include personal.journal
```

Prior to hledger 1.0, legacy `account` and `end` spellings were also
supported.

##### Multi-line comments

A line containing just `comment` starts a multi-line comment, and a line
containing just `end comment` ends it. See [comments](#comments).

##### commodity directive

The `commodity` directive predefines commodities (currently this is just
informational), and also it may define the display format for amounts in
this commodity (overriding the automatically inferred format).

It may be written on a single line, like this:

``` {.journal}
; commodity EXAMPLEAMOUNT

; display AAAA amounts with the symbol on the right, space-separated,
; using period as decimal point, with four decimal places, and
; separating thousands with comma.
commodity 1,000.0000 AAAA
```

or on multiple lines, using the "format" subdirective. In this case the
commodity symbol appears twice and should be the same in both places:

``` {.journal}
; commodity SYMBOL
;   format EXAMPLEAMOUNT

; display indian rupees with currency name on the left,
; thousands, lakhs and crores comma-separated,
; period as decimal point, and two decimal places.
commodity INR
  format INR 9,99,99,999.00
```

##### Default commodity

The D directive sets a default commodity (and display format), to be
used for amounts without a commodity symbol (ie, plain numbers). (Note
this differs from Ledger's default commodity directive.) The commodity
and display format will be applied to all subsequent commodity-less
amounts, or until the next D directive.

``` {.journal}
# commodity-less amounts should be treated as dollars
# (and displayed with symbol on the left, thousands separators and two decimal places)
D $1,000.00

1/1
  a     5    # <- commodity-less amount, becomes $1
  b
```

##### Default year

You can set a default year to be used for subsequent dates which don't
specify a year. This is a line beginning with `Y` followed by the year.
Eg:

``` {.journal}
Y2009      ; set default year to 2009

12/15      ; equivalent to 2009/12/15
  expenses  1
  assets

Y2010      ; change default year to 2010

2009/1/30  ; specifies the year, not affected
  expenses  1
  assets

1/31       ; equivalent to 2010/1/31
  expenses  1
  assets
```

##### Including other files

You can pull in the content of additional journal files by writing an
include directive, like this:

``` {.journal}
include path/to/file.journal
```

If the path does not begin with a slash, it is relative to the current
file. Glob patterns (`*`) are not currently supported.

The `include` directive can only be used in journal files. It can
include journal, timeclock or timedot files, but not CSV files.

### EDITOR SUPPORT

Add-on modes exist for various text editors, to make working with
journal files easier. They add colour, navigation aids and helpful
commands. For hledger users who edit the journal file directly (the
majority), using one of these modes is quite recommended.

These were written with Ledger in mind, but also work with hledger
files:

  ----------------- --------------------------------------------------------------------------------
  Emacs             <http://www.ledger-cli.org/3.0/doc/ledger-mode.html>
  Vim               <https://github.com/ledger/ledger/wiki/Getting-started>
  Sublime Text      <https://github.com/ledger/ledger/wiki/Using-Sublime-Text>
  Textmate          <https://github.com/ledger/ledger/wiki/Using-TextMate-2>
  Text Wrangler     <https://github.com/ledger/ledger/wiki/Editing-Ledger-files-with-TextWrangler>
  ----------------- --------------------------------------------------------------------------------

<!-- Some related LedgerTips:
https://twitter.com/LedgerTips/status/504061626233159681
https://twitter.com/LedgerTips/status/502820400276193280
https://twitter.com/LedgerTips/status/502503912084361216
https://twitter.com/LedgerTips/status/501767602067472384
-->



## csv format

This doc is for version **1.2**. []{.docversions}

### NAME

CSV - how hledger reads CSV data, and the CSV rules file format

### DESCRIPTION

hledger can read
[CSV](http://en.wikipedia.org/wiki/Comma-separated_values) files,
converting each CSV record into a journal entry (transaction), if you
provide some conversion hints in a "rules file". This file should be
named like the CSV file with an additional `.rules` suffix (eg:
`mybank.csv.rules`); or, you can specify the file with
`--rules-file PATH`. hledger will create it if necessary, with some
default rules which you'll need to adjust. At minimum, the rules file
must specify the `date` and `amount` fields. For an example, see [How to
read CSV files](how-to-read-csv-files.html).

To learn about *exporting* CSV, see [CSV
output](hledger.html#csv-output).

### CSV RULES

The following six kinds of rule can appear in the rules file, in any
order. Blank lines and lines beginning with `#` or `;` are ignored.

#### skip

`skip`*`N`*

Skip this number of CSV records at the beginning. You'll need this
whenever your CSV data contains header lines. Eg: <!-- XXX -->
<!-- hledger tries to skip initial CSV header lines automatically. -->
<!-- If it guesses wrong, use this directive to skip exactly N lines. -->
<!-- This can also be used in a conditional block to ignore certain CSV records. -->

``` {.rules}
# ignore the first CSV line
skip 1
```

#### date-format

`date-format`*`DATEFMT`*

When your CSV date fields are not formatted like `YYYY/MM/DD` (or
`YYYY-MM-DD` or `YYYY.MM.DD`), you'll need to specify the format.
DATEFMT is a [strptime-like date parsing
pattern](http://hackage.haskell.org/packages/archive/time/latest/doc/html/Data-Time-Format.html#v:formatTime),
which must parse the date field values completely. Examples:

``` {.rules .display-table}
# for dates like "6/11/2013":
date-format %-d/%-m/%Y
```

``` {.rules .display-table}
# for dates like "11/06/2013":
date-format %m/%d/%Y
```

``` {.rules .display-table}
# for dates like "2013-Nov-06":
date-format %Y-%h-%d
```

``` {.rules .display-table}
# for dates like "11/6/2013 11:32 PM":
date-format %-m/%-d/%Y %l:%M %p
```

#### field list

`fields`*`FIELDNAME1`*, *`FIELDNAME2`*...

This (a) names the CSV fields, in order (names may not contain
whitespace; uninteresting names may be left blank), and (b) assigns them
to journal entry fields if you use any of these standard field names:
`date`, `date2`, `status`, `code`, `description`, `comment`, `account1`,
`account2`, `amount`, `amount-in`, `amount-out`, `currency`. Eg:

``` {.rules}
# use the 1st, 2nd and 4th CSV fields as the entry's date, description and amount,
# and give the 7th and 8th fields meaningful names for later reference:
#
# CSV field:
#      1     2            3 4       5 6 7          8
# entry field:
fields date, description, , amount, , , somefield, anotherfield
```

#### field assignment

*`ENTRYFIELDNAME`* *`FIELDVALUE`*

This sets a journal entry field (one of the standard names above) to the
given text value, which can include CSV field values interpolated by
name (`%CSVFIELDNAME`) or 1-based position (`%N`).
<!-- Whitespace before or after the value is ignored. --> Eg:

``` {.rules .display-table}
# set the amount to the 4th CSV field with "USD " prepended
amount USD %4
```

``` {.rules .display-table}
# combine three fields to make a comment (containing two tags)
comment note: %somefield - %anotherfield, date: %1
```

Field assignments can be used instead of or in addition to a field list.

#### conditional block

`if` *`PATTERN`*\
    *`FIELDASSIGNMENTS`*...

`if`\
*`PATTERN`*\
*`PATTERN`*...\
    *`FIELDASSIGNMENTS`*...

This applies one or more field assignments, only to those CSV records
matched by one of the PATTERNs. The patterns are case-insensitive
regular expressions which match anywhere within the whole CSV record
(it's not yet possible to match within a specific field). When there are
multiple patterns they can be written on separate lines, unindented. The
field assignments are on separate lines indented by at least one space.
Examples:

``` {.rules .display-table}
# if the CSV record contains "groceries", set account2 to "expenses:groceries"
if groceries
 account2 expenses:groceries
```

``` {.rules .display-table}
# if the CSV record contains any of these patterns, set account2 and comment as shown
if
monthly service fee
atm transaction fee
banking thru software
 account2 expenses:business:banking
 comment  XXX deductible ? check it
```

#### include

`include`*`RULESFILE`*

Include another rules file at this point. `RULESFILE` is either an
absolute file path or a path relative to the current file's directory.
Eg:

``` {.rules}
# rules reused with several CSV files
include common.rules
```

### TIPS

Each generated journal entry will have two postings, to `account1` and
`account2` respectively. Currently it's not possible to generate entries
with more than two postings.

If the CSV has debit/credit amounts in separate fields, assign to the
`amount-in` and `amount-out` pseudo fields instead of `amount`.

If the CSV has the currency in a separate field, assign that to the
`currency` pseudo field which will be automatically prepended to the
amount. (Or you can do the same thing with a field assignment.)

If an amount value is parenthesised, it will be de-parenthesised and
sign-flipped automatically.

The generated journal entries will be sorted by date. The original order
of same-day entries will be preserved, usually.
<!-- (by reversing the CSV entries if they seem to be in reverse date order). -->


## timeclock format

This doc is for version **1.2**. []{.docversions}

### NAME

Timeclock - the time logging format of timeclock.el, as read by hledger

### DESCRIPTION

hledger can read timeclock files. [As with
Ledger](http://ledger-cli.org/3.0/doc/ledger3.html#Time-Keeping), these
are (a subset of)
[timeclock.el](http://www.emacswiki.org/emacs/TimeClock)'s format,
containing clock-in and clock-out entries as in the example below. The
date is a [simple date](#simple-dates). The time format is
HH:MM\[:SS\]\[+-ZZZZ\]. Seconds and timezone are optional. The timezone,
if present, must be four digits and is ignored (currently the time is
always interpreted as a local time).

``` {.timeclock}
i 2015/03/30 09:00:00 some:account name  optional description after two spaces
o 2015/03/30 09:20:00
i 2015/03/31 22:21:45 another account
o 2015/04/01 02:00:34
```

hledger treats each clock-in/clock-out pair as a transaction posting
some number of hours to an account. Or if the session spans more than
one day, it is split into several transactions, one for each day. For
the above time log, `hledger print` generates these journal entries:

``` {.shell}
$ hledger -f t.timeclock print
2015/03/30 * optional description after two spaces
    (some:account name)         0.33h

2015/03/31 * 22:21-23:59
    (another account)         1.64h

2015/04/01 * 00:00-02:00
    (another account)         2.01h
```

Here is a
[sample.timeclock](https://raw.github.com/simonmichael/hledger/master/examples/sample.timeclock)
to download and some queries to try:

``` {.shell}
$ hledger -f sample.timeclock balance                               # current time balances
$ hledger -f sample.timeclock register -p 2009/3                    # sessions in march 2009
$ hledger -f sample.timeclock register -p weekly --depth 1 --empty  # time summary by week
```

To generate time logs, ie to clock in and clock out, you could:

-   use emacs and the built-in timeclock.el, or the extended
    [timeclock-x.el](http://www.emacswiki.org/emacs/timeclock-x.el) and
    perhaps the extras in
    [ledgerutils.el](http://hub.darcs.net/simon/ledgertools/ledgerutils.el)

-   at the command line, use these bash aliases:

    ``` {.shell}
    alias ti="echo i `date '+%Y-%m-%d %H:%M:%S'` \$* >>$TIMELOG"
    alias to="echo o `date '+%Y-%m-%d %H:%M:%S'` >>$TIMELOG"
    ```

-   or use the old `ti` and `to` scripts in the [ledger 2.x
    repository](https://github.com/ledger/ledger/tree/maint/scripts).
    These rely on a "timeclock" executable which I think is just the
    ledger 2 executable renamed.




## timedot format

This doc is for version **1.2**. []{.docversions}

### NAME

Timedot - hledger's human-friendly time logging format

### DESCRIPTION

Timedot is a plain text format for logging dated, categorised quantities
(eg time), supported by hledger. It is convenient for approximate and
retroactive time logging, eg when the real-time clock-in/out required
with a timeclock file is too precise or too interruptive. It can be
formatted like a bar chart, making clear at a glance where time was
spent.

Though called "timedot", the format does not specify the commodity being
logged, so could represent other dated, quantifiable things. Eg you
could record a single-entry journal of financial transactions, perhaps
slightly more conveniently than with hledger\_journal(5) format.

### FILE FORMAT

A timedot file contains a series of day entries. A day entry begins with
a date, and is followed by category/quantity pairs, one per line. Dates
are hledger-style [simple dates](#simple-dates) (see
hledger\_journal(5)). Categories are hledger-style account names,
optionally indented. There must be at least two spaces between the
category and the quantity. Quantities can be written in two ways:

1.  a series of dots (period characters). Each dot represents "a
    quarter" - eg, a quarter hour. Spaces can be used to group dots into
    hours, for easier counting.

2.  a number (integer or decimal), representing "units" - eg, hours. A
    good alternative when dots are cumbersome. (A number also can record
    negative quantities.)

Blank lines and lines beginning with \#, ; or \* are ignored. An
example:

``` {.timedot}
# on this day, 6h was spent on client work, 1.5h on haskell FOSS work, etc.
2016/2/1
inc:client1   .... .... .... .... .... ....
fos:haskell   .... .. 
biz:research  .

2016/2/2
inc:client1   .... ....
biz:research  .
```

Or with numbers:

``` {.timedot}
2016/2/3
inc:client1   4
fos:hledger   3
biz:research  1
```

Reporting:

``` {.shell}
$ hledger -f t.timedot print date:2016/2/2
2016/02/02 *
    (inc:client1)          2.00

2016/02/02 *
    (biz:research)          0.25
```

``` {.shell}
$ hledger -f t.timedot bal --daily --tree
Balance changes in 2016/02/01-2016/02/03:

            ||  2016/02/01d  2016/02/02d  2016/02/03d 
============++========================================
 biz        ||         0.25         0.25         1.00 
   research ||         0.25         0.25         1.00 
 fos        ||         1.50            0         3.00 
   haskell  ||         1.50            0            0 
   hledger  ||            0            0         3.00 
 inc        ||         6.00         2.00         4.00 
   client1  ||         6.00         2.00         4.00 
------------++----------------------------------------
            ||         7.75         2.25         8.00 
```

I prefer to use period for separating account components. We can make
this work with an [account alias](#account-aliases):

``` {.timedot}
2016/2/4
fos.hledger.timedot  4
fos.ledger           ..
```

``` {.shell}
$ hledger -f t.timedot --alias /\\./=: bal date:2016/2/4
                4.50  fos
                4.00    hledger:timedot
                0.50    ledger
--------------------
                4.50
```

Here is a
[sample.timedot](https://raw.github.com/simonmichael/hledger/master/examples/sample.timedot).
<!-- to download and some queries to try: -->

<!-- ```shell -->
<!-- $ hledger -f sample.timedot balance                               # current time balances -->
<!-- $ hledger -f sample.timedot register -p 2009/3                    # sessions in march 2009 -->
<!-- $ hledger -f sample.timedot register -p weekly --depth 1 --empty  # time summary by week -->
<!-- ``` -->

