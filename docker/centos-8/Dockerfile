# This stage builds an rpm from the source
FROM centos:centos8 as centos-8-builder

RUN dnf install --enablerepo=powertools -y rpm-build git autoconf pcre-devel \
        automake libtool make readline-devel texinfo net-snmp-devel pkgconfig \
        groff pkgconfig json-c-devel pam-devel bison flex python3-pytest \
        c-ares-devel python3-devel python3-sphinx systemd-devel libcap-devel platform-python-devel \
        https://ci1.netdef.org/artifact/LIBYANG-LY1REL/shared/build-4/CentOS-8-x86_64-Packages/libyang1-1.0.184-0.x86_64.rpm \
        https://ci1.netdef.org/artifact/LIBYANG-LY1REL/shared/build-4/CentOS-8-x86_64-Packages/libyang-devel-1.0.184-0.x86_64.rpm \
        https://ci1.netdef.org/artifact/RPKI-RTRLIB/shared/build-110/CentOS-7-x86_64-Packages/librtr-0.7.0-1.el7.centos.x86_64.rpm \
        https://ci1.netdef.org/artifact/RPKI-RTRLIB/shared/build-110/CentOS-7-x86_64-Packages/librtr-devel-0.7.0-1.el7.centos.x86_64.rpm

COPY . /src

ARG PKGVER

RUN echo '%_smp_mflags %( echo "-j$(/usr/bin/getconf _NPROCESSORS_ONLN)"; )' >> /root/.rpmmacros \
    && cd /src \
    && ./bootstrap.sh \
    && ./configure \
        --enable-rpki \
        --enable-numeric-version \
        --with-pkg-extra-version="_git$PKGVER" \
    && make dist \
    && cd / \
    && mkdir -p /rpmbuild/{SOURCES,SPECS} \
    && cp /src/frr*.tar.gz /rpmbuild/SOURCES \
    && cp /src/redhat/frr.spec /rpmbuild/SPECS \
    && rpmbuild \
        --define "_topdir /rpmbuild" \
        -ba /rpmbuild/SPECS/frr.spec

# This stage installs frr from the rpm
FROM centos:centos8
RUN mkdir -p /pkgs/rpm \
    && yum install -y https://ci1.netdef.org/artifact/LIBYANG-LY1REL/shared/build-4/CentOS-8-x86_64-Packages/libyang1-1.0.184-0.x86_64.rpm \
        https://ci1.netdef.org/artifact/RPKI-RTRLIB/shared/build-110/CentOS-7-x86_64-Packages/librtr-0.7.0-1.el7.centos.x86_64.rpm

COPY --from=centos-8-builder /rpmbuild/RPMS/ /pkgs/rpm/

RUN yum install -y /pkgs/rpm/*/*.rpm \
    && rm -rf /pkgs
COPY docker/centos-8/docker-start /usr/lib/frr/docker-start
ENTRYPOINT [ "/usr/lib/frr/docker-start" ]
