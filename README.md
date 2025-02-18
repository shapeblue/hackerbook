# CloudStack HackerBook

Hackerbook is a rapid learning framework for onboarding and training new
CloudStack developers. This learn by doing yourself course is aimed at anybody
who wants to learn how to develop a feature for [Apache
CloudStack](http://cloudstack.apache.org/). The basic course can be completed in
4-5 weeks and overall it can be completed in about 6-8 weeks.

In this course, each chapter has some short videos and suggested exercises which
the new developer can work on to learn by doing them, hence the name
`hackerbook`. The course starts with chapter 1 on general guideline on getting
started, and then encourages the developer to learn CloudStack as a user
in chapter 2 where they are asked to install, use and work with CloudStack
using the API, UI and have `cmk` (CLI) and ansible-based automation exercises.
Next, in chapter 3 the developer is asked to work on a fictious feature which
helps them learn about various aspect of building parts of a feature such as
creating an API, handling API via a service layer manager, DB handling, UI etc.
Rest of the remaining chapters encourage self learning and exploration with
recommended reading and references around advanced CloudStack topics.

ShapeBlue started `hackerbook` course material internally in late 2018 to onboard
and train new engineers to work on Apache CloudStack. After successfully
onboarding and training a bunch of new engineers and improving hackerbook,
ShapeBlue opensourced `hackerbook` for the Apache CloudStack community in 2021.

## Contents

| Chapter | Topic | Est. Effort |
| ------- | ----- | ----------- |
| #1 | [Getting Started](0-init.md) | 10 hours |
| #2 | [Test Drive CloudStack](1-user.md) | 40 hours |
| #3 | [Basic CloudStack Development](2-dev.md) | 150 hours |
| #4 | [Advanced CloudStack Development](3-adv.md) | 40 hours |
| #5 | [Hypervisor and Storage](4-compute-storage.md) | 40 hours |
| #6 | [Networking](5-network.md) | 40 hours |
|    | [Appendix: Primers](primer/index.md) | |
| | | **320 hours** (6-8 weeks) |

## Bookmarks

- [CloudStack Awesome List](https://github.com/resmo/awesome-cloudstack)
- [CloudStack Docs](http://docs.cloudstack.apache.org/en/latest/)

## Contribution and Getting Help

Raise a pull request to contribute changes to the course documentation. We may not
be able to work on any reported issue and offer individual help to the reader.
We encourage readers to join and ask questions on the Apache CloudStack dev/user
mailing lists: http://cloudstack.apache.org/mailing-lists.html

## Changelog

- 18 Feb 2025 - updated for Ubuntu 24.04 & MacOS as dev platforms
- 1 Jan 2023 - updated for Ubuntu 22.04 as dev platform
- 26 Feb 2021 - hackerbook opensourced
- 22 Feb 2021 - repository updated against Ubuntu 20.04 as dev platform
- 15 Oct 2018 - hackerbook started by Rohit Yadav to train new engineers at ShapeBlue

## License

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img
alt="Creative Commons License" style="border-width:0"
src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a><br />This work
is licensed under a <a rel="license"
href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons
Attribution-ShareAlike 4.0 International License</a>.
