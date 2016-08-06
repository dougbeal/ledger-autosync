ledger-autosync
===============

ledger-autosync is a program to pull down transactions from your bank and create
[ledger](http://ledger-cli.org/) transactions for them. It is designed to only
create transactions that are not already present in your ledger files (that is,
deduplicate transactions). This should make it comparable to some of the
automated synchronization features available in products like GnuCash, Mint,
etc. In fact, ledger-autosync performs OFX import and synchronization better
than all the alternatives I have seen.

Features
--------

-   supports [ledger](http://ledger-cli.org/) 3 and
    [hledger](http://hledger.org/)
-   like ledger, ledger-autosync will never modify your files directly
-   interactive banking setup via
    [ofxclient](https://github.com/captin411/ofxclient)
-   multiple banks and accounts
-   support for non-US currencies
-   support for 401k and investment accounts
    -   tracks investments by share, not dollar value
    -   support for complex transaction types, including transfers, buys, sells,
        etc.
-   import of downloaded OFX files, for banks not supporting automatic
    download
-   import of downloaded CSV files from Paypal, Amazon and Mint

Platforms
---------

ledger-autosync is developed on Linux with ledger 3; it has been tested
on Windows (although it will run slower) and should run on OS X. It
requires ledger 3 or hledger, but it should run faster with ledger,
because it will not need to start a command to check every transaction.

Quickstart
----------

If you are on Debian or Ubuntu, an (older) version of ledger-autosync should be
available for installation. Try:

    $ sudo apt-get install ledger-autosync

If you use pip, you can install the latest released version:

    $ pip install ledger-autosync

You can also install from source, if you have downloaded the source:

    $ python setup.py install

Run ofxclient to set up banking:

    $ ofxclient

When you have added your institution, quit ofxclient.

(At least one user has reported being signed up for a pay service by
setting up OFX direct connect. Although this seems unusual, please be
aware of this.)

Edit the generated `~/ofxclient.ini` file. Change the `description`
field of your accounts to the name used in ledger. Optionally, move the
`~/ofxclient.ini` file to your `~/.config` directory.

Run:

    ledger-autosync

This will download a maximum of 90 days previous activity from your
accounts. The output will be in ledger format and printed to stdout. Add
this output to your ledger file. When that is done, you can call:

    ledger-autosync

again, and it should print nothing to stdout, because you already have
those transactions in your ledger.

Syncing a file
--------------

Some banks allow users to download OFX files, but do not support
fetching via the OFX protocol. If you have an OFX file, you can convert
to ledger:

    ledger-autosync /path/to/file.ofx

This will print unknown transactions in the file to stdout in the same
way as ordinary sync. If the transaction is already in your ledger, it
will be ignored.

How it works
------------

ledger-autosync stores a unique identifier, (for OFX files, this is a unique ID
provided by your institution for each transaction), as metadata in each
transaction. When syncing with your bank, it will check if the transaction
exists by running the ledger or hledger command. If the transaction exists, it
does nothing. If it does not exist, the transaction is printed to stdout.

Syncing a CSV file
------------------

If you have a CSV file, you may also be able to import it using a recent
(installed via source) version of ledger-autosync. ledger-autosync can currently
process CSV files as provided by Paypal, Amazon, or Mint. You can process the
CSV file as follows:

    ledger-autosync /path/to/file.csv -a Assets:Paypal

With Amazon and Paypal CSV files, each row includes a unique identifier, so
ledger-autosync will be able to deduplicate against any previously imported
entries in your ledger files.

With Mint, a unique identifier based on the data in the row is generated and
stored. If future downloads contain identical rows, they will be deduplicated.
This method is probably not as robust as a method based on unique ids, but Mint
does not provide a unique id, and it should be better than nothing. It is likely
to generate false negatives: transactions that seem new, but are in fact old. It
will not generate false negatives: transactions that are not generated because
they seem old.

If you are a developer, you should fine it easy enough to add a new CSV format
to ledger-autosync. See, for example, the `MintConverter` class in the
`ledgerautosync/converter.py` file in this repository.

Assertions
----------

If you supply the `--assertions` flag, ledger-autosync will also print
out valid ledger assertions based on your bank balances at the time of
the sync. These otherwise empty transactions tell ledger that your
balance *should* be something at a given time, and if not, ledger will
fail with an error.

resync
------

By default, ledger-autosync will process transactions backwards, and
stop when it sees a transaction that is already in ledger. To force it
to process all transactions up to the `--max` days back in time
(default: 90), use the `--resync` option. This can be useful when
increasing the `--max` option. For instance, if you previously
synchronized 90 days and now want to get 180 days of transactions,
ledger-autosync would stop before going back to 180 days without the
`--resync` option.

Testing
-------

ledger-autosync uses nose for tests. To test, run nosetests in the
project directory. This will test the ledger, hledger and ledger-python
interfaces. To test a single interface, use nosetests -a
hledger. To test the generic code, use nosetests -a generic. To test
both, use nosetests -a generic -a hledger. For some reason
nosetests -a '!hledger' will not work.
