# Perl5 Module Meta - Best Practices

This is a collection of things that help making your CPAN module "portable"
with regard to metadata.

CPAN/PAUSE itself is not very strict about these things. But turning a module
into an RPM or Debian package for example can be hard if it is missing
meta data.

People maintaining `rpm` or `deb` repositories for perl often have to do a lot
of manual work, although theoretically most of it can be automated.

## Table of Contents

### Author

* [Versions](#Versions) - Use a consistent versioning scheme
* [Description](#Description) - Create a useful description
* [Archive filename](#Archive-filename) - Create a correct archive
* [`.` in `@INC`](#INC) - The current directory `.` is not in `@INC`
* [Changelog](#Changelog) - Use a common filename and format
* [License](#License) - Correctly specify license
* [Dependencies](#Dependencies) - Specify all dependencies
* [Shebang](#Shebang) - Use a portable shebang for scripts
* [MANIFEST](#MANIFEST) - Exclude certain files from your archive
* [Bugtracker](#Bugtracker) - Specify URL to Bugtracker
* [Tests](#Tests) - Make sure tests are passing everywhere
* [Installer](#Installer)
* [Author Tests](#Author-Tests) - Useful Author tests
* [Other Resources](#Other-Resources)

### Vendors

* [Versions](#Versions)
* [License](#License)

## Versions

### Use a consistent versioning scheme

The short story: Always use the same amount of digits after the major version.

The long story:

Perl versions are different from common versioning schemes.

In Perl, the version `1.1901` is semantically equal to `v1.190.100`,
and `1.20` to `v1.200`.

So it can happen that a CPAN module was released with version `1.1901`, and
a year later a new version `1.20` is released. For perl, the new version is
bigger.

If vendors want to turn the module into an `rpm` or `deb`, they would have to
turn the version into `1.190.100`/`1.200` to be correct. The only disadvantage
is that the version may not look the same as the original CPAN version anymore.

But if they take the version as it is, then `1.190` is bigger than the following
`1.20`.

Make it easier for vendors and always use the same amounts of digits. If you want
to change to a different versioning scheme, then at least change the major
version before.

Use the same distribution version everywhere (`META.*` files, `Makefile.PL`,
archive name, Changelog).

### Do not reuse versions

Always increase the version for a new upload and do not reuse the same version.

Also have a look at Grinnz' [Guide to Versions in
Perl](http://blogs.perl.org/users/grinnz/2018/04/a-guide-to-versions-in-perl.html).

## Description

Every module should have a `DESCRIPTION` section in the pod documentation.

    =head1 NAME

    Module::Name - Abstract

    =head1 SYNOPSIS

        use Module::Name;
        # example code

    =head1 DESCRIPTION

    A few paragraphs with a description of your module.

This description is usually what ends up in the vendor package description.
Don't make it too long and create extra headers for more specific parts.

Also check for spelling mistakes. See [Author Tests](#Author-Tests).

## Archive filename

The filename you upload should match the distribution name and version, e.g.

    Your-Module-1.23.tar.gz
    Your-Module-1.23.zip
    Your-Module-v1.23.tar.gz
    Your-Module-v1.23.zip

`tgz` and `tar.bz2` might also work.

The archive should unpack to `Your-Module-version`. Do not omit the version.
If people unpack different versions of your module, they should not end up
in the same directory.

The contents of the module should be in the top folder of the archive, e.g.

    Your-Module-1.23/
        lib/Your/Module.pm
        t/...
        LICENSE
        Makefile.PL

## `@INC`

In perl 5.26, the current directory `.` was removed from `@INC`. Make sure
you don't rely on `.`, for example when using `Module::Install`, or when
using modules in your test directory.

Don't use

    use t::lib::Some::Module;

Instead do:

    use lib 't/lib';
    use Some::Module;

Or:

    use FindBin '$Bin';
    use lib "$Bin/lib";
    use Some::Module;

## Changelog

Use a common format and filename for your changelog file. The most common filename
is `Changes`.

For vendors to be able to parse the file, use a format like this:

    1.23 2019-11-02 12:34:56+02:00

        - Bug fix: ...
        - New Feature: ...

The version should exactly match the version of the distribution.

Newer versions should be above older versions.

If you fixed a bug that was reported in an issue, mention the issue.
Vendors often create local patches for modules and, if they have time, create
an issue.
If you fixed the issue and mention it in the changelog, vendors can more
easily remove outdated patches.


## License

The license should appear ideally in four places:

* META.json
* META.yml
* LICENSE file
* The pod documentation of your main module

## Dependencies

List *all* dependencies. The list of modules contained in core may change over
time. It's best to include all dependencies including the ones that are
currently in core.

If possible, add the minimum required version of the dependency. For example,
when using `Test::More::done_testing()`, require `Test::More` version `0.88`,
as it was added in this version.

Also don't forget to specify the minimum perl version.

Also have a look at Neil's [Specifying dependencies for your CPAN
distribution](http://blogs.perl.org/users/neilb/2017/05/specifying-dependencies-for-your-cpan-distribution.html).

### Non-perl dependencies

If your module needs an external library, there is currently no way to specify
this in `META.json`/`META.yml`. But you can help vendors by documenting
the external libaries your module needs in pod.

## Shebang

The first line of any perl scripts included in your distribution, especially
the ones that will be installed, should look like that:

    #!/usr/bin/perl

When the module is installed, this line will be automatically replaced with
the path to perl that was used for building.

Alternatively `#!perl` should also work.

Note that if you use

    #!/usr/bin/env perl

the line is currently not rewritten. See
[Perl-Toolchain-Gang/ExtUtils-MakeMaker#58](https://github.com/Perl-Toolchain-Gang/ExtUtils-MakeMaker/issues/58)
for a discussion on this.  If the line is not rewritten and your installed
script is executed with a different perl, this can lead to problems, for example
dependencies will not be found.

## MANIFEST

Make sure to not include files in your archive that don't belong there,
like editor backup files (`.swp`), the `.git` folder, forgotten tarballs.

Usually build tools will skip files listed in `MANIFEST.SKIP`.

## Bugtracker

If your repository is hosted at a site like GitHub or BitBucket, and you are
using the bugtracker it provides, add the URL to the bugtracker in the
`META.json/META.yml`. That makes it easier for people to find existing
and file new bugs.

## Tests

Regularly look at the CPAN Testers site (linked from your module page on MetaCPAN)
to see if there are any test failures.

If possible, don't rely on a working network and mock network operations.

Don't rely on a fixed order of hash keys.

Tests should not create or alter files in the distribution.
If you have to create files for tests, delete them at the end.

This can be done in an `END` block, so the files will be deleted also when
a test fails and exits unexpectedly.

    use File::Path 'rmtree';
    ...
    END {
        rmtree "/path/to/temporary/testfiles";
    }

## Installer

Don't prompt for input in the install process, for example in `Makefile.PL` or
`Build.PL`. This will break automatic installations.
If you need a prompt, use `ExtUtils::MakeMaker::prompt()`, as it will detect
if it's running interactively or not and use a default.

## Author Tests

There are a number of Test modules that help you avoiding some of the mentioned
issues.

* [Test::Pod](https://metacpan.org/pod/Test::Pod) - Check for correct POD
* [Test::Spelling](https://metacpan.org/pod/Test::Spelling) - Check for spelling mistakes in POD
* [Test::CPAN::Meta](https://metacpan.org/pod/Test::CPAN::Meta) - Validate `META.yml`
* [Test::Kwalitee](https://metacpan.org/pod/Test::Kwalitee) - Includes a number of different checks
* [Test::CheckChanges](https://metacpan.org/pod/Test::CheckChanges) - Check that the Changes file matches the distribution
* [Test::CPAN::Changes](https://metacpan.org/pod/Test::CPAN::Changes) - Check format of Changes file

Place them in the `xt/` directory so they are not run by default. They
should be run by the author before releasing.

## Other resources

### openSUSE

You can find a lot of CPAN modules in the [devel:languages:perl](https://build.opensuse.org/project/show/devel:languages:perl)
repository of openSUSE Build Service (OBS).

Look if you can find your module there. Maybe you'll find out that it contains
local patches.

The most used tool to create `.spec` files for CPAN modules is
[cpanspec](https://github.com/openSUSE/cpanspec).  It can use a `cpanspec.yml`
file for any changes necessary to make the module build successfully.

You will also see a lot of modules not using `cpanspec.yml` yet. The spec was
probably created manually.

The `cpanspec.yml` can be used for adding forgotten dependencies, but also for
non-perl dependencies, like you can see in the
[Gtk3](https://build.opensuse.org/package/show/devel:languages:perl/perl-Gtk3)
distribution.

`cpanspec` is not perfect, and OBS maintainers will be glad for help in making
it handle more distributions correctly.

### Debian

See the [Debian Perl Group](https://perl-team.pages.debian.net/)

* List of [perl modules maintained by the Debian Perl
  Group](https://salsa.debian.org/perl-team/modules/packages)
* [Debian Package Tracker](https://tracker.debian.org/teams/pkg-perl/)
* [How to request a package](https://perl-team.pages.debian.net/howto/RFP.html)
* Presentation [From Debian, with ♥](https://perl-team.pages.debian.net/docs/yapc-europe-2016/from_debian_with_love.pdf)

The tool to create debian packages from Perl modules is
[dh-make-perl](https://salsa.debian.org/perl-team/modules/packages/dh-make-perl).

### Fedora

[List of perl packages](https://apps.fedoraproject.org/packages/s/perl)


# Vendors

## Versions

Versions in Perl come in two forms, decimal numbers and dotted decimal tuples.

In Perl, the version `1.1901` is semantically equal to `v1.190.100`,
and `1.20` to `v1.200`.

Basically, if a module has a version without a `v` in front and with only one
dot, it's a a decimal version.

If you are using dotted tuples, you can convert all versions with the
[version.pm](https://metacpan.org/pod/version) module:

    my $tuple = version->parse($input_string)->normal;
    # 1.2      -> v1.200.0
    # 1.2345   -> v1.234.500
    # v1.2345  -> v1.2345.0
    # 1.2345.6 -> v1.2345.6

You can find more details in Grinnz' [Guide to Versions in
Perl](http://blogs.perl.org/users/grinnz/2018/04/a-guide-to-versions-in-perl.html).


## License

https://metacpan.org/pod/CPAN::Meta::Spec#SERIALIZATION

> In the past, the distribution metadata structure had been packed with
> distributions as META.yml, a file in the YAML Tiny format (for which, see
> YAML::Tiny). Tools that consume distribution metadata from disk should be
> capable of loading META.yml, but should prefer META.json if both are found.

https://metacpan.org/pod/CPAN::Meta::History::Meta_1_4#license
