---
title: Sopel 7.0 Migration
covers_from: 6.x
covers_to: 7.0
---

# Sopel 7.0 Migration

Sopel 7 lays the groundwork for a lot of awesome stuff! We have a major update
to Sopel's command-line interface in progress (which will be finished in Sopel
8), and some API updates that might affect a few existing plugins.

But first, we really need to talk briefly about Python.


## A note about Python versions

If you still use Sopel under Python 2, you might notice that Sopel 7 emits
warnings about version compatibility. The sad reality is, as of January 1, 2020,
Python 2.7 no longer receives any updates. While this doesn't mean we will
simply drop support for running Sopel under Python 2 immediately, it _does_ mean
that we will no longer reject ideas that would require doing so.

For the life of Sopel 7.x, we still plan to maintain compatibility with Python
2.7 unless it becomes absolutely necessary to drop it. For example, if a severe
bug is found in one of Sopel's dependencies, and the fix is only released for
Python 3, we would consider dropping Python 2 support in the next minor version.

From Sopel 8 onward, it would be much easier to implement new features and
enhancements if we dropped support for very old Python releases. For example,
Sopel's support for reloading plugins during runtime can be made much more
robust using features added to the language in Python 3.4.

The crux of the matter is this: Sopel's range of supported Python releases
remained stagnant for far too long. While the Sopel project was effectively
unmaintained between late 2016 and early 2018, Python 3.3 reached end-of-life
(September 29, 2017). During the lengthy development period of Sopel 7, Python
3.4 reached end-of-life (March 18, 2019). Sopel 7 is likely to be released very
close to the EOL of Python 2.7 (January 1, 2020).

We can't keep testing support for these old versions forever. At some point,
Sopel's core developers will lose the ability to run them locally. (Python 3.3
is already difficult—though not impossible—to install on current popular Linux
systems like Ubuntu 18.04 LTS.) Travis CI, our continuous integration testing
provider for contributions from both maintainers and the community alike, won't
keep supporting the installation of EOL Python releases indefinitely. We can't
support what we can't test.

Furthermore, it's a waste of effort to fix bugs in new code that, for whatever
reason, only affect old, disused versions of Python. The ratio of time spent
to users impacted becomes too high as more systems get upgraded.

And finally, supporting older Python releases sometimes prevents us from
making highly desirable improvements (see above, re: reloading things).

Keeping all this in mind, the current plan is as follows. Note that it is
subject to change, as Sopel's development pace remains quite leisurely relative
to the overall Python ecosystem.

  - Sopel 7 maintains the same Python version compatibility range as 6.x
    - If our testing infrastructure becomes unable to run the older versions of
      Python, we might have to bend this one, but the goal is to avoid dropping
      any supported versions during the life of Sopel 7.x
  - Sopel 8 will drop support for EOL Python releases immediately at the start
    of its development cycle, and adjust version support based on the estimated
    release date target as that becomes clearer
    - This **definitely** means dropping 2.7, 3.3, and 3.4 (already EOL)
    - Python 3.5 support **might** be dropped, depending on timing; it's set
      to reach EOL around September 2020 (but no hard date yet)
    - Python 3.6 is [tentatively][PEP-494] EOL in December 2021, but Sopel 8
      should be finished long before then

[PEP-494]: https://www.python.org/dev/peps/pep-0494/


## Database support

Sopel 7 brings back support for databases other than SQLite—an often-requested
feature ever since Sopel went SQLite-only in version 5.

While we can't practically offer documentation on every possible setup, it
should be fairly easy to migrate an existing installation to any supported
database type with a bit of search-fu. Sopel's database support is built on
SQLAlchemy, which has [ample documentation][sqlalchemy-dialects] of its own
about getting up and running with various back-ends.

Please don't hesitate to offer feedback in [our IRC channel][sopel-channel] or
[a GitHub issue][gh-new-issue]. This is a huge feature, and with a small team
it's simply not possible for us to test everything—especially the less common
database types (paid ones like Oracle, especially).

[gh-new-issue]: https://github.com/sopel-irc/sopel/issues/new
[sopel-channel]: irc://irc.libera.chat/sopel
[sqlalchemy-dialects]: https://docs.sqlalchemy.org/en/13/dialects/index.html


## CLI restructuring

Version 7 deprecates many of the command-line arguments that Sopel has used for
most of its life, in favor of a much more extensible command structure.

Instead of having arguments like `--quit` or `--configure-modules`, Sopel's CLI
now works on a sub-command system (like Git and other popular tools). This will
allow adding more functions without having to share one global argument
namespace, and make the commands more "speakable".

The full new command structure will be documented in the [Command-line
arguments]({% link _usage/command-line-arguments.md %}) after release, but as a
general overview of the old structure vs. the new:

|             This            |             Becomes            |
| :-------------------------- | :----------------------------- |
| `sopel --quit`              | `sopel quit`                   |
| `sopel --kill`              | `sopel quit --kill`            |
| `sopel --configure-all`     | `sopel configure`              |
| `sopel --configure-modules` | `sopel configure --plugins`    |
| `sopel --list`              | `sopel-config list`            |
| `sopel -v`                  | `sopel -V` / `sopel --version` |

There's one argument deserving of special mention: `--migrate`/`-m`. It will
be removed in Sopel 7, because it has been a no-op since version 4.0.0.
Someone deleted the code to handle it without saying so anywhere, but Sopel
has continued to carry around this useless argument since 2014. No more!

New commands are not always going to be shorter than the old ones (see the
`--kill` example), but we're looking at the future picture. This is just the
start of Sopel's CLI evolution, and more features will come based on this new
command structure.

### Removal of old commands

For the life of Sopel 7, the existing ("legacy") arguments are still supported,
just with deprecation notices in the `--help` output.

In Sopel 8, the old arguments from Sopel 6 and lower will be removed.

Most deprecated arguments in Sopel 7 will output warnings to the terminal when
used. Hopefully we remembered them all during development—but if we missed any,
please do let us know via the [issue tracker][gh-new-issue] or IRC.

### New helper commands

Some helper tools debut with Sopel 7. So far, they are `sopel-config` &
`sopel-plugins`. Both are intended to make the job of configuring a Sopel
instance easier, without needing to edit the config file manually.

#### `sopel-config`

The `sopel-config` tool supports `init`ializing a new config file, `list`ing
existing config files, and `get`ting values from an existing config file.

More features will likely be added along the road to Sopel 8. Suggestions are
welcomed and encouraged!

#### `sopel-plugins`

With the `sopel-plugins` command, you can `list` available plugins; `enable`
or `disable` plugins in a specific config file; and `show` details about a
specific plugin such as where it is on disk and whether it's enabled.

Here, too, more features will likely be added along the road to Sopel 8.
Suggestions are welcomed and encouraged!


## Sopel 7 API changes

This is a good time to remind you that your plugins should specify a maximum
Sopel version in their requirements. Doing so reduces the likelihood that users
will be able to install your plugin alongside an incompatible version of Sopel,
and thus minimizes the risk of compatibility errors during startup or runtime.

We do everything possible to introduce breaking API changes *only* in major
versions, so it's typically safe for your plugin to require (for example)
`sopel>=6.0,<8.0`—at least the earliest major version you've tested, and less
than the next unreleased major version. Occasionally you might need to require
a specific minor version, if your code needs a new feature that was introduced
in between major releases, but major-version compatibility is typically all
you'll need to worry about.

Normally, when a feature needs to be removed from the API, we try to allow for
a deprecation period, during which the feature to be removed continues to work
but logs a warning if used. It is extremely rare for an API feature to be
removed in a minor release—and if we do so, that means there was no way to keep
that feature around in a working state until the next major Sopel version. The
[changelog][sopel-changelog] always notes deprecations and removals.

  [sopel-changelog]: {% link _changelogs/index.md %}

### `Identifier` case-mapping

In a twist of fate (and old spaghetti code), it turns out that Sopel has been
mapping `Identifier`s to lowercase incorrectly for a *long* time. Like, since
`Identifier` was called `Nick`, [over 7 years ago][commit-adding-nick-class].
Ever since version 3.2.0, the first version to introduce case-mapping for
nicknames, it's been non-compliant with the RFC.

Sopel 7 finally fixes that. Existing instances will opportunistically convert
values from the old (incorrect) representation to the new (correct) one as
needed, but that can't cover all plugins. With our apologies for the
inconvenience, you'll have to implement your own migration logic if your plugin
stores and/or manipulates `Identifier`s in any way outside the
`set/get_*_value()` functions provided by Sopel's DB API.

Everything you'll need from Sopel's API is still in the `Identifier` class. You
can use [`Identifier._lower()`][docs-identifier-lower] to get the corrected
representation of a nick or channel name. Call on
[`Identifier._lower_swapped()`][docs-identifier-lower-swapped] for the
previous, incorrect, representation. When working with channel names
specifically, the [`bot.db.get_channel_slug()`][docs-db-get-channel-slug]
method might be helpful.

Whether to write a one-off migration that converts everything at once, extra
logic that converts values on-the-fly as needed, or something else is *all* up
to you, based on your own plugin's considerations. Different plugin projects
will have different needs. For example, if you're the only user of your plugin,
you might simply write a short script to run once against your bot's database
and be done with it—but if your plugin is uploaded to PyPI, you might choose to
release the migration as part of your next version bump. The choice is yours;
we just provide the tools you need to get started.

  [commit-adding-nick-class]: https://github.com/sopel-irc/sopel/commit/f8ca0b9
  [docs-db-get-channel-slug]: /docs/db.html#sopel.db.SopelDB.get_channel_slug
  [docs-identifier-lower]: /docs/api.html#sopel.tools.Identifier._lower
  [docs-identifier-lower-swapped]: /docs/api.html#sopel.tools.Identifier._lower_swapped

### Accessing the database

While Sopel's [migration to SQLAlchemy](#database-support) doesn't affect
*most* of [the `bot.db` API][docs-db], some plugins that make use of the more
direct methods might need to be rewritten for Sopel 7. Non-exhaustively:

* [`bot.db.execute()`][docs-db-execute] *should* still return an object that
  behaves like a `Cursor`, but since it's actually a SQLAlchemy wrapper not
  everything is guaranteed to work exactly the same
* [`bot.db.get_uri()`][docs-db-get_uri] hasn't functionally changed, but it's
  important to remember that the returned URI *might* not point to SQLite
* [`bot.db.connect()`][docs-db-connect] is likewise functionally unchanged,
  except that it can now return a non-SQLite DBAPI connection object,
  potentially one with behavior different from the SQLite connections always
  returned in older versions

We recommend that authors of plugins which use a raw database connection from
`bot.db.connect()` aim to rewrite their code so it uses the ORM approach and
[`bot.db.session()`][docs-db-session] instead. In the interim, it never hurts
to update your plugin's documentation to warn users that non-SQLite databases
haven't been tested, or make sure the plugin is marked as compatible with Sopel
<7.0 only until it can be tested and/or updated.

  [docs-db]: /docs/db.html
  [docs-db-connect]: /docs/db.html#sopel.db.SopelDB.connect
  [docs-db-execute]: /docs/db.html#sopel.db.SopelDB.execute
  [docs-db-get_uri]: /docs/db.html#sopel.db.SopelDB.get_uri
  [docs-db-session]: /docs/db.html#sopel.db.SopelDB.session

### Managing URL callbacks

For quite a while, Sopel plugins wishing to override the `url.py` plugin's
automatic title-fetching for certain URLs have customarily done something along
these lines:

```python
# in the plugin's setup() function:
    if not bot.memory['url_callbacks']:
        bot.memory['url_callbacks'] = tools.SopelMemory()
    bot.memory['url_callbacks'][compiled_regex] = methodname
```

Similar manual manipulation of the object in memory was needed to unregister
handlers at plugin unload:

```python
# in the plugin's shutdown() function:
    try:
        del bot.memory['url_callbacks'][compiled_regex]
    except KeyError:
        pass
```

Going forward, a new set of API methods should be used instead:

  - `bot.register_url_callback(pattern, callback)`, to invoke `callback` when a
    URL in a message matches the `pattern`
  - `bot.unregister_url_callback(pattern, callback)`, to stop invoking the
    `callback` when a URL in a message matches the `pattern`
  - `bot.search_url_callbacks(url)`, to find callbacks matching the given `url`

`bot.memory['url_callbacks']` will remain unchanged for the life of Sopel 7.x.
We plan to make this data structure private in Sopel 8.0, so we can improve it
(e.g. allowing multiple callbacks for the same pattern). The new API methods
are already future-proofed against the changes we plan to make; that's why the
callback function is required both when registering *and* unregistering.

### Adding multiple command examples

Decorating a plugin callable like this was a great way to add documentation to
that command through Sopel's `help` plugin:

```python
from sopel import module

@module.example('.foo barbaz')
def foo_cmd(bot, trigger):
    bot.action('foos %s' % trigger.group(3))
```

However, *only one example* could ever appear in the `help` output. This was
[confusing](https://github.com/sopel-irc/sopel/issues/1200) to plugin authors.

Furthermore, if more than one example was defined:

```python
from sopel import module

@module.example('.foo barbaz')
@module.example('.foo spam eggs sausage bacon')
def foo_cmd(bot, trigger):
    bot.action('foos %s' % ', '.join(trigger.group(2).split())
```

It was not necessarily intuitive which example would be displayed if a user
did `.help foo`. The code says it would use `example[0]`. That's the first one
in the list, which makes sense. But take a guess which of the two above would
be used—`.foo barbaz` or `.foo spam eggs sausage bacon`?

Ready to see if you were right?

Sopel would use `.foo spam eggs sausage bacon` as the example, because due to
how decorators work, it ends up first in the internal list despite appearing
last in the source code. Not very intuitive for beginning plugin writers…

So, in Sopel 7, there is a `user_help` argument to `@module.example`. If at
least one of a callable's examples has this attribute set to `True`, all such
examples will be used when outputting help for that command:

```python
from sopel import module

@module.example('.foo barbaz', 'foos barbaz', user_help=True)
@module.example('.foo', "I can't foo that!")
@module.example('.foo egg sausage bacon', 'foos egg, sausage, bacon', user_help=True)
def foo_cmd(bot, trigger):
    if not trigger.group(2):
        return bot.say("I can't foo that!")
    bot.action('foos %s' % ', '.join(trigger.group(2).split())
```

Here, Sopel's help output will show both `.foo barbaz` and `.foo egg sausage
bacon` as examples. Plain `.foo` does not have `user_help=True` (it's purely
there for testing), and so it will not be shown.

Of course, backwards compatibility is important! That's why we used this
approach. Callables without any `user_help=True` examples will behave just
like they would have in Sopel 6 and older: The "first" example (the one
closest to the function's `def` line, last in the source line order) will
appear in `help`'s output.

Making `user_help=True` the default would make *a ton* of sense, definitely!
But if we did that, many (many) existing Sopel (and Willie) plugins would
potentially output "bad" help information—so we elected to keep the old
behavior by default in an effort to minimize any "breakage".

### Logging API rework

Plugins should use the new `sopel.tools.get_logger()` function to get a
logging object, starting with Sopel 7.0. It takes the same argument (the
plugin's name) as its predecessor, `sopel.logger.get_logger()`.

`sopel.logger.get_logger()` will have an extended deprecation cycle, to allow
ample time for the ecosystem to adapt without spamming too many log files at
first. The old function's behavior has been tweaked to work reasonably nicely
with the harmonized logging system implemented for Sopel 7.

Calls to `sopel.logger.get_logger()` will begin emitting deprecation warnings
in Sopel 8.0, to alert stragglers (or users of possibly-abandoned code). We
will remove this function from the API in Sopel 9.0.

### Removal of deprecated attributes

A number of ancient attributes that were considered deprecated many releases
ago _finally_ have been removed, mostly from the `Bot` object. Among them:

  - `bot.ops`
  - `bot.halfplus`
  - `bot.voices`
  - `bot.stats`

This list is not comprehensive, but honestly: If your code breaks because a
removed attribute no longer exists, it's actually been broken for a *long* time
on account of the attributes themselves always being empty. They were just
placeholders to avoid anything raising `AttributeError`. (Sopel's current
maintenance team wouldn't have done it that way, but we can't undo the past.)

### Improvements to testing tools

An absolute *ton* of work went into refactoring how the bot works internally
for Sopel 7, and one side benefit (nay, a goal) of the changes was to make
things more directly testable, without needing to create special "mock"
objects. If you have used `sopel.test_tools` to write unit tests for your
plugin code, you're probably familiar with at least one of `MockConfig`,
`MockSopel`, and `MockSopelWrapper`—three classes Sopel itself used in its own
tests until this release.

With the rewrites for Sopel 7, though, Sopel's tests don't need these mock
classes any more. We test directly on the "real" objects, only switching out
the IRC connection for a fake one that just logs its input and output instead
of actually opening a socket to a remote server. This means that our `Mock*`
classes can be marked as **deprecated**; we will remove them in Sopel 8.

Fortunately, there's also another bit of good news: Sopel now comes with a
`pytest` plugin and a whole set of fixtures, factories, and mocks in
`sopel.tests`. We encourage you to [explore them][docs-test-tools] and update
your plugins' tests (or write them, if you haven't already, you naughty
developer!) to use the new goodies. Trust us—they're *much* nicer to write
tests with than the old tools were!

  [docs-test-tools]: https://sopel.chat/docs/tests.html


## Sopel 7 plugin changes

### Reminder DB migration

Sopel 7 refers to specific instances by config file name whenever possible,
instead of using other pieces of config data such as `nick` or `host` that are
more likely to change. The `remind` and `tell` plugins have been updated with
this in mind, and will attempt to automatically convert their respective data
files to the new name format if the old filename exists.

You probably will not need to do anything. However, if the automatic migration
does fail, it will output (and log) information about what it was trying to
do, and link to this section of the Sopel 7 upgrade guide for convenience.

**If a migration failure brought you here:** Above the link you should find
the old and new filenames the plugin was attempting to use.

**Important:** Ensure the associated instance of Sopel is NOT running
before doing anything. Tampering with the reminder files while Sopel is
running can result in data loss.

In most error cases, migration will fail because the new filename already
exists. The simplest fix is to move or rename the conflicting file, and run
Sopel again so the migration can complete.

If both the old and new files are non-empty, you might want to peek inside
them with a text editor to see what's there before deciding which to keep. The
current format for both plugins' data files is essentially <abbr
title="Tab-Separated Values">TSV</abbr>, and they can be merged by hand
(again, _with Sopel stopped_) if both the old _and_ new files somehow contain
meaningful, unique entries.

And of course, if you need more in-depth assistance with fixing a failed
migration, [our IRC channel][sopel-channel] always welcomes questions.

### Core plugin removals

### `spellcheck`

As of February 2018, the Python bindings for `enchant` [became
unmaintained][pyenchant-unmaintained]. This made installing the `spellcheck`
plugin's dependencies increasingly difficult, and often
[caused][windows-enchant] [problems][linux-enchant] with new installations.

Because of this, the `spellcheck` plugin was rewritten to use `aspell` instead,
and extracted into a [standalone PyPI package][pypi-spellcheck] to eliminate
those non-Python dependencies for installing Sopel itself.

The rewrite also added new commands to manage a custom word list:

  - `.scadd` - stages a word for adding to the bot's word list
  - `.scpending` - lists words pending addition to the bot's custom list
  - `.scdel` - removes a word from the pending list
  - `.scsave` - commits pending words to the bot's word list
  - `.scclear` - clears the list of pending words without saving

Unfortunately, the `aspell` API only supports *adding* words to the custom
dictionary. To *remove* a custom word, a user must manually edit the dictionary
file, so we decided to go with a two-step process. Hopefully it will help Sopel
admins around the world avoid adding typos to their bots' custom dictionaries!

  [pyenchant-unmaintained]: https://github.com/rfk/pyenchant/commit/4df35b7
  [windows-enchant]: https://github.com/sopel-irc/sopel/issues/1142
  [linux-enchant]: https://github.com/sopel-irc/sopel/pull/1454
  [pypi-spellcheck]: https://pypi.org/project/sopel-spellcheck/

### `ipython`

We've made the `ipython` plugin into [its own PyPI package][pypi-ipython],
further reducing the packages required to install Sopel itself. Mostly, though,
this decision was based on the limited utility of this plugin for
non-developers. Its main use case is poking around in Sopel's state while
debugging a new plugin or plugin feature, with Sopel running in an interactive
shell. It's unusable when Sopel is run as a service/daemon (which is how most
deployments *should* run Sopel), and we decided it doesn't make sense to
continue bundling this plugin and its requirements with the core bot.

  [pypi-ipython]: https://pypi.org/project/sopel-ipython/


## Planned user-facing changes

A "user" here is someone who *runs* (or is responsible for *maintaining*) one
or more Sopel bots. If you're reading this, that *probably* describes you.
(Unless you're some kind of weirdo who only writes plugins for other people to
use, and never tests them… Seriously, who does that?)

You might need to edit your bot's configuration in the future due to these
plans. Sopel might take care of them on its own, too. But in case human
intervention is required, here are the details.

### Config format change for list values

Since the new config system was introduced, lists of values have been
represented in the config file as strings separated by commas (`,`). Sopel 7
adds support for storing these lists as multi-line strings instead, with
values separated by newlines.

Here's what that means in practice:

```conf
# Current format
[core]
enable = admin,reddit,wikipedia,wiktionary

# New format
[core]
enable =
    admin
    reddit
    wikipedia
    wiktionary
```

Note that Sopel 7 will upgrade the stored format of any comma-separated list
value if the config is saved (with `.save`) after editing the value via IRC
(with `.set section.option_name list,of,values`).

**Important: The new format requires any value beginning with `#` to be
quoted.** Automatic conversion will handle this, but be aware of it if you're
tweaking your config file(s) by hand during the upgrade process. You will need
to be careful with the `core.channels` list, in particular. Most updated
`core.channels` values should look like this:

```conf
[core]
channels =
    "#spam"
    "#eggs"
    "#bacon"
    "#snakes"
```

Unquoted values beginning with `#` *might* work properly on certain Python
versions *if indented*, but you should quote them anyway to be safe.

In Sopel 7, this is just a convenience change. It means that lists stored in
the new format can support commas within the values, without any annoying
escape-character shenanigans (which was our first plan). Old-style values (all
on a single line) will continue to be split on commas when reading config
files, as before, and will be updated to the new format as described above.

**Eventually, in Sopel 9, we plan to remove this fallback behavior.** Sopel 8
may emit a warning to the console and/or log file at startup if old-style list
values are present in the loaded config file, but we encourage updating your
config files *long* before then.

### Modules vs. plugins

Sopel has long used the term "module" in reference to Python files (or folders
thereof) that add functionality to the bot. This is reflected even in the
layout of Sopel's data folder: `~/.sopel/modules` is so named because that's
where the modules go! We'd like to change that, though.

The simplified explanation is that Python already has "modules". Add-ons for
Sopel are also Python modules, but not all Python modules are Sopel add-ons.
This overlap gets confusing for developers sometimes.

Many similar projects call this kind of add-on a "plugin". We've already
started using the term "plugin" in our documentation, in place of "module",
when referring to these add-ons.

As a user, you'll notice only small changes related to this shift. The
restructured CLI [described above](#cli-restructuring), for example, uses
`sopel configure --plugins` to replace `sopel --configure-modules`. Our
documentation will continue to move toward consistently referring to "plugins"
and "modules" as distinct concepts. We'll also change the default search
location for plugins from `~/.sopel/modules` to `~/.sopel/plugins` in a future
release (probably Sopel 8), but it will be easy to re-add the old folder if
you like via the `core.extra` setting.


## Planned future API changes

This section is all about stuff that won't cause problems *now*, but *will*
break in a future release if not updated. Most of these are planned removals of,
or changes to, API features deprecated long ago.

We suggest reviewing these upcoming changes, and updating your own plugins if
they still use anything listed here, as soon as possible. Updating plugins
published to PyPI should take priority, especially plugins written for Sopel 6
that are not future-proofed by capping Sopel's version in their requirements.

If you use third-party plugins that have not been updated, we encourage you to
inform the author(s) politely that they need to update. Or better yet, submit a
pull request or patch yourself!

### Removal of `bot.privileges`

Sopel 7.x will be the last release series to support the `bot.privileges` data
structure (deprecated in [Sopel 6.2.0][v6.2.0], released January 16, 2016).

Beginning in Sopel 8, `bot.privileges` will be removed and plugins trying to
access it will throw an exception. `bot.channels` will be the _only_ place to
get privilege data going forward.

### Removal of `bot.msg()`

[Back in 6.0][v6.0.0], Sopel's API standardized around a consistent argument
order for messaging functions: `message` first, then an optional `recipient` (or
`destination`, if you like). Part of the old API, `bot.msg()`, has stuck around
since then because it was, [quote][msg-hard-comment], "way too much of a pain to
remove". In fact, it turned out to be quite easy to remove.

None of Sopel's own code uses this old method any more, and we will remove it
entirely in 8.0. Uses of `bot.msg()` in 7.0 will emit a deprecation warning, so
any remaining third-party code that still uses it can be found and patched.

  [msg-hard-comment]: https://git.io/sopel-msg-pain

### Rename/cleanup of `sopel.web`

While the whole `sopel.web` module was marked as deprecated in [Sopel
6.2.0][v6.2.0], because it largely serves as a wrapper around the `requests`
library, parts of it seem to be useful enough that they should be kept around.

For Sopel 8, we intend to move `sopel.web` to `sopel.tools.web`. The new
location is available in Sopel 7 to provide a transitional period. Similar to
how importing from both `willie` and `sopel` worked in the run-up to Sopel 6.0,
it is possible to do any of the following during Sopel 7's life cycle:

  - `import sopel.web`
  - `from sopel import web`
  - `import sopel.tools.web`
  - `from sopel.tools import web`

In Sopel 8, we will remove the pointers from `sopel.web` to the new location.
These explicitly deprecated functions will also be removed at the same time:

  - `sopel.web.get()` — use `requests.get()` directly instead
  - `sopel.web.head()` — use `requests.head()` directly instead
  - `sopel.web.post()` — use `requests.post()` directly instead
  - `sopel.web.get_urllib_object()` — really, just use [`requests`][requests]

We will also tweak the module constants:

  - `sopel.web.default_headers`: renamed to `sopel.tools.web.DEFAULT_HEADERS`
  - `sopel.web.ca_certs`: removed in `sopel.tools.web` — it no longer has any
    function (and was probably not useful for Sopel plugins to import, anyway)

New additions to Sopel's web tools made during the life of 7.x will be
available only in `sopel.tools.web`. Functions and constants that we plan to
remove in Sopel 8 (as listed above) will be available only from the old
`sopel.web` module.

  [requests]: https://pypi.org/project/requests/
  [v6.0.0]: {% link _changelogs/6.0.0.md %}
  [v6.2.0]: {% link _changelogs/6.2.0.md %}

### Finalizing the "plugin" transition

As described [above](#modules-vs-plugins), we're trying to get away from the
ambiguity of calling Sopel add-ons "modules", because that term is already
used by Python itself. All Sopel add-ons are Python modules, but not all
Python modules are Sopel add-ons. Many people in the Sopel community have
tripped over this blurred line, and we have plans to change more than just
documentation eventually.

Already, in Sopel 7, the internal mechanisms for handling add-on code are all
about "plugins" now. The module responsible is named `sopel.plugins`, and its
submodules & members all agree on this nomenclature. But you, a _plugin_
developer, must—confusingly—still import the "_module_" module from `sopel` to
use any of the decorators that make Sopel plugins work. Annoying, isn't it?

Rest assured that we're not done yet. Future releases of Sopel will support
(even encourage) importing `sopel.plugin` instead, which will be the new home
of all plugin-related decorators & constants. `sopel.module` won't go away any
time soon, though. It predates even the name "Willie", after all—we can't just
yank it out without a _long_ deprecation cycle. We might just leave it as a
permanent alias to `sopel.plugin`. (The plan is still [up for
discussion][#1738], if you're interested.)

But, once available, you should _definitely_ use `sopel.plugin` in your new
code. It's the future!

  [#1738]: https://github.com/sopel-irc/sopel/issues/1738
