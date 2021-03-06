# NAME

Linux::APT - Interface with APT for Debian distributions

# DESCRIPTION

Perl interface to `apt-get` and `apt-cache`.
If Debian's `aptpkg` modules were on CPAN, this module (probably) wouldn't be necessary.
This is just a wrapper around the `apt` tools along with some regular expression magic
to capture interesting pieces of information/warnings/errors in the process.
It doesn't do _everything_ that is possible, but it should fill the most typical needs.
Features will be added on request or my own need.
Please file a wishlist bug report on the CPAN bug tracker with your feature requests.

All (or almost all?) features require root privileges.
If you can use `sudo` to provide this functionality, see `new` to see how to do this.

It's not ready for production use, but you're welcome to give it a try.
_Please_ file bug reports if you come across any problems/bugs/etc.
Patches are always welcomed, of course.

# EXAMPLE

    my $apt = Linux::APT->new;
    my $update = $apt->update;
    my $toupgrade = $apt->toupgrade;
    my $upgraded = $apt->install(keys(%{$toupgrade->{packages}}));

# METHODS

## new

    my $apt = Linux::APT->new;

    # only if you _really_ want to see what's going on...
    my $apt = Linux::APT->new(debug => 1);

    # if you want to use an alternate apt-get/apt-cache binary
    my $apt = Linux::APT->new(
      aptget => '/some/path/to/apt-get',
      aptcache => '/some/path/to/apt-cache',
    );

    # if you have special needs (like sudo, etc)
    my $apt = Linux::APT->new(
      aptget => '/usr/bin/sudo /some/path/to/apt-get -s', # sudo and no-act
      aptcache => '/usr/bin/sudo /some/path/to/apt-cache', # sudo
    );

Creates an instance of Linux::APT, just like you would expect.

If you have special needs for only one function (install maybe?), make a separate instance
with your special needs (flags, sudo, etc) and use that instance for your special need.

If your special need can't be accommodated via the `aptget` option above, let me know and
I'll attempt to implement whatever it is that you need within the module or make your special
need a bit more "accessible" to you.
File a bug report on the CPAN bug tracker.
Patches welcome, of course.

Arguments available:

- debug

    Set to `1` to enable, defaults to `0`.

- aptget

    Specify the `apt-get` binary to use along with any special flags or command line tricks (sudo, chroot, fakeroot, etc).
    Defaults to `` `which apt-get` ``.

- aptcache

    Specify the `apt-cache` binary to use along with any special flags or command line tricks (sudo, chroot, fakeroot, etc).
    Defaults to `` `which apt-cache` ``.

## update

    my $update = $apt->update;

    warn "There were errors...\n" if $update->{error};
    warn "There were warnings...\n" if $update->{warning};

Update apt cache.
Basically equivalent to `apt-get update`.

Returns hashref containing these items:

- error

    Arrayref of errors.

- warning

    Arrayref of warnings.

- speed

    Network transfer speed of update.

- time

    Wallclock time it took to update.

- size

    Amount of received transferred during update.

## toupgrade

    my $toupgrade = $apt->toupgrade;

Returns hashref of packages, errors, and warnings:

- warning

    Warnings, if any.

- error

    Errors, if any.

- packages

    Contains a hashref of updateable packages.
    Keys are package names.
    Each update is a hashref containing these items:

    - current

        Currently installed version.

    - new

        Version to be installed.

## search

    my $search = $apt->search('^t\w+d$', 'perl');

    my $search = $apt->search({in => ['all']}, '^t\w+d$', 'perl'); # 'all' is default

    my $search = $apt->search({in=>['name', 'description']},
      'linux[\s-]image', 'linux[\s-]source', 'linux kernel image');

    my $search = $apt->search({in => ['description']}, 'linux kernel source');

Requires one or more search arguments in regex format.  Optional options as first
argument in hashref format.

Return a hashref of packages that match the regex search.

- packages

    Multiple searches can be specified.  Each search is a hash key then broken
    down by each matching package name and it's summary.

## install

    # install or upgrade the specified packages (and all deps)
    my $install = $apt->install('nautilus', 'libcups2', 'rhythmbox');

    # just a dry run
    my $install = $apt->install('-test', 'nautilus', 'libcups2', 'rhythmbox');

    # upgrade all upgradable packages with a name containing "pulseaudio" (and all deps)
    my $toupgrade = $apt->toupgrade;
    my $install = $apt->install(grep(m/pulseaudio/i, keys(%{$toupgrade->{packages}})));

Install a list of packages.
If the packages are already installed, they will be upgraded if an upgrade is available.

Pass in these optional options:

- -force

    If you wish to force an update (eg: `WARNING: The following packages cannot be authenticated!`),
    pass `-force` as one of your arguments (same effect as `apt-get --force-yes install $packages`).

- -test

    If you just want to know what packages would be installed/upgraded/removed, pass `-test`
    as one of your arguments.
    No actions will actually take place, only the actions that would have been performed will be captured.
    This is useful when you want to ensure some bad thing doesn't happen on accident (like removing
    `apache2-mpm-worker` when you install `php5`) or to allow you to present the proposed changes to the
    user via a user interface (GUI, webapp, etc).

Returns hashref of packages, errors, and warnings:

- warning

    Warnings, if any.

- error

    Errors, if any.

- packages

    Contains a hashref of installed/upgraded packages.
    Keys are package names.
    Each item is a hashref containing these items:

    - current

        Currently installed version (after install/upgrade attempt).
        This version is found via an experimental technique and might fail (though it has yet to fail for me).
        Let me know if you find a bug or have a problem with this value.

    - new

        Version to be installed.
        If `new == current`, the action seems to have succeeded.

    - old

        Version that was installed before the upgrade was performed.
        If `old == current`, the action seems to have failed.

## remove

    my $removed = $apt->remove('php5', 'php5-common');

    # just a dry run
    my $removed = $apt->remove('-test', 'php5', 'php5-common');

Remove a list of packages.
Arguments are the exact same as `install`.
Returns the exact same as `install`.

## purge

    my $removed = $apt->purge('php5', 'php5-common');

    # just a dry run
    my $removed = $apt->purge('-test', 'php5', 'php5-common');

Purge a list of packages.
Arguments are the exact same as `install`.
Returns the exact same as `install`.

# TODO

- (update this todo list...)
- Add functions to modify the `sources.list`.
- Add `dist-upgrade` functionality.
- Add function to show version(s) of currently installed specified package(s).
- Determine other necessary features. (please use the CPAN bug tracker to request features)

# BUGS/WISHLIST

**REPORT BUGS!**
Report any bugs to the CPAN bug tracker.  Bug reports are adored.

To wishlist something, use the CPAN bug tracker (set as wishlist).
I'd be happy to implement useful functionality in this module on request.

# PARTICIPATION

I'd be very, very happy to accept patches in diff format.  Or...

If you wish to hack on this code, please fork the git repository found at:
[http://github.com/emmaly/Linux--APT/](http://github.com/emmaly/Linux--APT/)

If you have some goodness to push back to that repository, just use the
"pull request" button on the github site or let me know where to pull from.

# THANKS

- Nicholas DeClario
    - Patch to provide initial search function for version 0.02.

# COPYRIGHT/LICENSE

Copyright 2009 Emmaly.  You can use any one of these licenses: Perl Artistic, GPL (version >= 2), BSD.

## Perl Artistic License

Read it at [http://dev.perl.org/licenses/artistic.html](http://dev.perl.org/licenses/artistic.html).
This is the license we prefer.

## GNU General Public License (GPL) Version 2

    This program is free software; you can redistribute it and/or
    modify it under the terms of the GNU General Public License
    as published by the Free Software Foundation; either version 2
    of the License, or (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with this program.  If not, see http://www.gnu.org/licenses/

See the full license at [http://www.gnu.org/licenses/](http://www.gnu.org/licenses/).

## GNU General Public License (GPL) Version 3

    This program is free software: you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with this program.  If not, see http://www.gnu.org/licenses/

See the full license at [http://www.gnu.org/licenses/](http://www.gnu.org/licenses/).

## BSD License

    Copyright (c) 2009 Emmaly.
    All rights reserved.

    Redistribution and use in source and binary forms, with or without modification, are permitted
    provided that the following conditions are met:

        * Redistributions of source code must retain the above copyright notice, this list of conditions
        and the following disclaimer.
        * Redistributions in binary form must reproduce the above copyright notice, this list of conditions
        and the following disclaimer in the documentation and/or other materials provided with the
        distribution.
        * Neither the name of Emmaly nor the names of its contributors may be used to endorse
        or promote products derived from this software without specific prior written permission.

    THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED
    WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A
    PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT OWNER OR CONTRIBUTORS BE LIABLE FOR
    ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
    LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
    INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
    OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN
    IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
