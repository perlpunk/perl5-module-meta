# Perl5 Module Meta - Best Practices

This is a collection of things that help making your CPAN module "portable"
with regard to metadata.

CPAN/PAUSE itself is not very strict about these things. But turning a module
into an RPM or Debian package for example can be hard if it is missing
meta data.

People maintaining `rpm` or `deb` repositories for perl often have to do a lot
of manual work, although theoretically most of it can be automated.

## Use a consistent versioning scheme

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

Always increase the version for a new upload and do not reuse the same version.

Also have a look at Grinnz' [Guide to Versions in
Perl](http://blogs.perl.org/users/grinnz/2018/04/a-guide-to-versions-in-perl.html).

## Create a useful description

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

## `.` in `@INC`

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

If you fixed a bug that was reported in a ticket/issue, mention the issue.
Vendors often create local patches for modules and, if they have time, create
a ticket.
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

Also don't forget to specify the minimum perl version.

Also have a look at Neil's [Specifying dependencies for your CPAN
distribution](http://blogs.perl.org/users/neilb/2017/05/specifying-dependencies-for-your-cpan-distribution.html).

## Shebang

The first line of any perl scripts included in your distribution, especially
the ones that will be installed, should look like that:

    #!/usr/bin/perl

When the module is installed, this line will be automatically replaced with
the path to perl that was used for building.

## MANIFEST

Make sure to not include files in your archive that don't belong there,
like editor backup files (`.swp`), the `.git` folder, forgotten tarballs.

Usually build tools will skip files listed in `MANIFEST.SKIP`.

## Bugtracker URL

If your repository is hosted at a site like GitHub or BitBucket, and you are
using the bugtracker it provides, add the URL to the bugtracker in the
`META.json/META.yml`. That makes it easier for people to find existing
and file new bugs.

## Non-perl dependencies

If your module needs an external library, there is currently no way to specify
this in `META.json`/`META.yml`. But you can help vendors by documenting
the external libaries your module needs in pod.

## CPAN Testers

Regularly look at the CPAN Testers site (linked from your module page on MetaCPAN)
to see if there are any test failures.

## Other resources

### openSUSE Build Service (OBS)

You can find a lot of CPAN modules in the [devel:languages:perl](https://build.opensuse.org/project/show/devel:languages:perl)
repository of OBS.

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

### Fedora

[List of perl packages](https://apps.fedoraproject.org/packages/s/perl)


