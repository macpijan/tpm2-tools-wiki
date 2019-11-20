# Release Lifecycle

General information can be found in the [RELEASE.md](https://github.com/tpm2-software/tpm2-tools/blob/master/RELEASE.md) file. However, it doesn't really describe the overall life cycle of releases. The tpm2-tools project has a bit of a BC/AD like split. Anything before 4.0 is considered legacy and is or will be reaching end of life. Releases greater than 4.0 will always **will be backwards compatible**. Thus, based on the semver.org rules outlined, pretty much dictates we will never be off of a 4.X version number. Because of this, master will always be the *next* release, and bugfix only releases can be branched off of *master* as needed. These patch level branches will be supported on an as needed bases, since we don't have dedicated stable maintainers. The majority of development will occur on *master* with tagged release numbers following semver.org recommendations. This page explicitly does not formalize an LTS support timeline, and that is intentional. The release schedules and required features are driven by community involvement and needs. However, milestones will be created to outline the goals, bugs, issues and timelines of the next release.

# End Of Life versions
- [1.X](https://github.com/tpm2-software/tpm2-tools/tree/1.X)
- [2.X](https://github.com/tpm2-software/tpm2-tools/tree/2.X)
- [3.0.X](https://github.com/tpm2-software/tpm2-tools/tree/3.0.X)

# Near End of Life
- [3.0.X](https://github.com/tpm2-software/tpm2-tools/tree/3.0.X): EOL after 3.2.1 release.

## OpenSSL

tpm2-tools relies heavily on OpenSSL. OpenSSL will be EOL'ing 1.0.2 at the end of 2019, see: https://www.openssl.org/blog/blog/2018/05/18/new-lts/. When this occurs, we will remove OSSL 1.0.2 support from the tpm2-tools repository as supporting an EOL crypto library is not a good idea.