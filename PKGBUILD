# jedit:mode=shellscript:
# Maintainer: Dan Fuhry <dan@fuhry.com>
pkgbase=beats
pkgname=(filebeat auditbeat heartbeat metricbeat packetbeat)
pkgdesc="The Elastic Beats framework. Includes filebeat, auditbeat, heartbeat, metricbeat and packetbeat."
license=('Apache')
arch=('x86_64')
url="https://github.com/elastic/${pkgbase}"

pkgver=6.2.1
pkgrel=1

depends=(go)
makedepends=(go python-virtualenv)

source=("git+${url}.git#tag=v${pkgver}"
		beat-wrapper
		beats.service)
sha256sums=('SKIP'
			'SKIP'
			'SKIP')

build() {
	cd "${srcdir}"
	
	# go packages are very picky about directory structure
	# we have to put everything into a tree that looks like "src/github.com/..."
	test -d "./src" && rm -rf "./src"
	mkdir -p "src/github.com/elastic"
	mv -v "beats" "src/github.com/elastic/"

	# FIXME: Kind of a hacky way to get thread count, and this will definitely
	# completely peg the system while we're building. Ideally, siphon the "-j"
	# flag from MAKEFLAGS in /etc/makepkg.conf and use that for -p at "go build"
	# below.
	ncpu=$(cat /proc/cpuinfo | grep '^cpu MHz' | wc -l)

	# the big compile loop
	for prog in ${pkgname[@]}; do
		cd "${srcdir}/src/github.com/elastic/${pkgbase}/${prog}"
		# set GOPATH before building.
		# TODO: get dynamic linking working, currently we are compiling all
		# static binaries and they are laaaaaaarge.
		env GOPATH=$srcdir go build -x -p=$[ncpu+2]
		# Ignore errors for this because not all of the beats programs include
		# a "configs" target in their makefiles.
		make configs || :
	done
}

# Shared function to package all beats packages.
_package() {
	local _pkgname="$1"
	local _srcpath="${srcdir}/src/github.com/elastic/${pkgbase}/${_pkgname}"

	cd "${_srcpath}"

	# Install the wrapper script, which executes the main executable 
	install -d -m755 "${pkgdir}/usr/bin"
	sed -re "s;BEATNAME;${_pkgname};g" "${srcdir}/beat-wrapper" > "${pkgdir}/usr/bin/${_pkgname}"
	chmod 0755 "${pkgdir}/usr/bin/${_pkgname}"

	# Install the systemd service unit file. The text "BEATNAME" (case
	# sensitive) in the source file is replaced with the name of the executable.
	install -d -m755 "${pkgdir}/usr/lib/systemd/system"
	sed -re "s;BEATNAME;${_pkgname};g" "${srcdir}/beats.service" > "${pkgdir}/usr/lib/systemd/system/${_pkgname}.service"
	chmod 0644 "${pkgdir}/usr/lib/systemd/system/${_pkgname}.service"

	# Install the main binary
	install -d -m755 "${pkgdir}/usr/lib/${_pkgname}"
	install -m755 "${_pkgname}" "${pkgdir}/usr/lib/${_pkgname}/${_pkgname}"

	# Install modules and kibana dashbaord files
	install -d -m755 "${pkgdir}/usr/share/${_pkgname}/module"
	install -d -m755 "${pkgdir}/usr/share/${_pkgname}/kibana"

	# More or less usurped from filebeat's scripts and cleaned up a bit. Also
	# designed in such a way that this is safe to run on beats that 
	modules=($(find "${_srcpath}/module" -mindepth 1 -maxdepth 1 -type d | xargs -n1 --no-run-if-empty basename))
	metadirs=(kibana)
	for m in ${modules[@]}; do
		cp -av "${_srcpath}/module/$m" "${pkgdir}/usr/share/${_pkgname}/module/$m"
		for d in ${metadirs[@]}; do
			metadir="${pkgdir}/usr/share/${_pkgname}/module/$m/_meta/$d"
			[ -d "$metadir" ] || continue
			mkdir -p "${pkgdir}/usr/share/${_pkgname}/$d"
			(cd $metadir && tar cvf - .) | tar xvCf "${pkgdir}/usr/share/${_pkgname}/$d" -
		done
		
		for d in _meta test; do
			find "${pkgdir}/usr/share/${_pkgname}/module/$m" -type d -name "$d" -print0 | xargs -0 rm -rf
		done
	done

	# Install configuration files.
	mkdir -p "${pkgdir}/etc/${_pkgname}"
	cp -av "${_srcpath}/${_pkgname}.yml" "${pkgdir}/etc/${_pkgname}/"
	if [ -d "${_srcpath}/modules.d" ]; then
		cp -av "${_srcpath}/modules.d" "${pkgdir}/etc/${_pkgname}/"
	fi
	if [ -f "${_srcpath}/_meta/beat.reference.yml" ]; then
		cp -av "${_srcpath}/_meta/beat.reference.yml" "${pkgdir}/etc/${_pkgname}/${_pkgname}.full.yml"
	fi
	if [ -f "${_srcpath}/_meta/fields.common.yml" ]; then
		cp -av "${_srcpath}/_meta/fields.common.yml" "${pkgdir}/etc/${_pkgname}/fields.yml"
	fi


}

package_auditbeat() {
	_package "${pkgname}"
}

package_filebeat() {
	_package "${pkgname}"
}

package_heartbeat() {
	_package "${pkgname}"
}

package_metricbeat() {
	_package "${pkgname}"
}

package_packetbeat() {
	_package "${pkgname}"
}
