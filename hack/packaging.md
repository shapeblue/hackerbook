# Packaging

RPM packaging references:
- https://fedoraproject.org/wiki/How_to_create_a_GNU_Hello_RPM_package
- https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html-single/rpm_packaging_guide/index
- https://rpm-guide.readthedocs.io/en/latest/rpm-guide.html

DEB packaging references:
- https://www.debian.org/doc/devel-manuals#packaging-tutorial
- https://www.debian.org/doc/manuals/maint-guide/index.en.html
- https://wiki.debian.org/Packaging/Intro
- https://wiki.debian.org/BuildingTutorial
- http://packaging.ubuntu.com/html/packaging-new-software.html

Reference implementation of a CloudStack feature/plugin packages outside of
the CloudStack tree: https://github.com/shapeblue/ccs/tree/master/packaging

## RPM Package Development

The rpm `spec` files for `centos{6,7}` are at `packaging` directory where you
can define the package.

As an guided example, here's how you can define a new `rpm` package
`cloudstack-hackerbook-coffee` in `packaging/centos7/cloud.spec`:

Define the package name and metadata such as requirements, descriptions etc:

```
%package hackerbook-coffee
Summary: Apache CloudStack Hackerbook Coffee feature
Requires: %{name}-management = %{_ver}
Group: System Environmnet/Libraries
%description hackerbook-coffee
Apache CloudStack Hackerbook Coffee feature
```

In the spec's `%install` section define how you want the feature artifacts to be
installed:

```
# hackerbook-coffee feature
mkdir -p ${RPM_BUILD_ROOT}%{_datadir}/%{name}-hackerbook-coffee/lib
cp -r plugins/hackerbook/feature/target/cloud-plugin-hackerbook-feature-%{_maventag}.jar ${RPM_BUILD_ROOT}%{_datadir}/%{name}-management/lib
```

You may add additional `%preun`, `%pre`, `%post` sections for the package to
define how it gets installed/upgraded/uninstalled etc.

Finally define the `%files` section:

```
%files hackerbook-feature
%defattr(0644,cloud,cloud,0755)
%attr(0644,root,root) %{_datadir}/%{name}-management/lib/*hackerbook-feature*jar
```

## DEB Package Development

Debian packaging rules and configuration files are in the `debian` directory.

As an guided example, here's how you can define a new deb package
`cloudstack-hackerbook-coffee` in `debian`:

Define your new package in `debian/control`:

```
Package: cloudstack-hackerbook-coffee
Architecture: all
Depends: ${misc:Depends}, cloudstack-management (= ${source:Version})
Description: The CloudStack Hackerbook feature
```

In the `override_dh_auto_install` section of `debian/rules` define how your
feature's artifacts are copied, installed:

```
# cloudstack-hackerbook-coffee
mkdir -p $(DESTDIR)/usr/share/$(PACKAGE)-management/lib
cp -r plugins/hackerbook/feature/target/cloud-plugin-hackerbook-feature-*jar $(DESTDIR)/usr/share/$(PACKAGE)-management/lib/
```

Define what artifacts your package will install, for example create a file
`debian/cloudstack-hackerbook-coffee.install` that lists files that will be
installed by your package:

```
/usr/share/cloudstack-management/lib/*hackerbook-coffee*
```

In addition, you can define custom script to execute for before/after package
install/uninstall/upgrade etc by defining the
`debian/cloudstack-hackerbook-coffee.{preinst,postinst}` files as needed.

## Exercises

Package the feature and any of its plugins (jars and UI plugin) as a separately
installed rpm/deb package.
