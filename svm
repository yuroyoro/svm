#!/usr/bin/env bash

# Copyright (c) 2011 Tomohito Ozaki(yuroyoro).
# All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions
# are met:
#
#  1. Redistributions of source code must retain the above copyright
#     notice, this list of conditions and the following disclaimer.
#  2. Redistributions in binary form must reproduce the above copyright
#     notice, this list of conditions and the following disclaimer in
#     the documentation and/or other materials provided with the
#     distribution.
#  3. The name of the author may not be used to endorse or promote
#     products derived from this software without specific prior
#     written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE AUTHOR ``AS IS'' AND ANY EXPRESS
# OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
# WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
# ARE DISCLAIMED. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR ANY
# DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
# DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE
# GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
# INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER
# IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR
# OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN
# IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

NEXT_VERSION="2.10.0"
AVAILABLE_VERSIONS=`cat <<EOS
2.5.0.final
2.5.1.final
2.6.0.final
2.7.0.final
2.7.1.final
2.7.2.final
2.7.3.final
2.7.4.final
2.7.5.final
2.7.6.final
2.7.7.RC1
2.7.7.RC2
2.7.7.final
2.8.0.Beta1-RC1
2.8.0.Beta1-RC2
2.8.0.Beta1-prerelease
2.8.0.RC1
2.8.0.RC2
2.8.0.RC3
2.8.0.RC4
2.8.0.RC5
2.8.0.RC6
2.8.0.RC7
2.8.0.final
2.8.1.RC1
2.8.1.RC2
2.8.1.RC3
2.8.1.RC4
2.8.1.final
2.8.2.final
2.9.0
2.9.0.1
2.9.1.RC1
2.9.1.RC2
2.9.1.RC3
2.9.1.RC4
2.9.1.final
2.9.1-1-RC1
2.9.1-1
2.9.2-RC1
2.9.2-RC2
2.9.2-RC3
2.9.2
2.9.3-RC1
2.9.3
2.10.0-M1
2.10.0-M2
2.10.0-M3
2.10.0-M4
2.10.0-M5
2.10.0-M6
2.10.0-M7
2.10.0-RC1
2.10.0-RC2
2.10.0-RC3
2.10.0-RC4
2.10.0-RC5
2.10.0
2.10.1
2.10.2-RC2
2.10.2
2.10.3-RC2
2.10.3-RC3
2.10.3
2.11.0-M2
2.11.0-M3
2.11.0-M4
2.11.0-M5
EOS`

BASE_DIR="${SCALA_INSTALLED_DIR}"
if [ -z "${BASE_DIR}" ] ; then
  BASE_DIR="${HOME}/.svm"
fi

if [ ! -d "${BASE_DIR}" ] ; then
  mkdir -p "${BASE_DIR}"
fi

path_included=0

path=$PATH:
while [ $path ]
do
  p=${path%%:*}
  if [ "${BASE_DIR}/current/rt/bin" = "${p}" ] ; then
    path_included=1
  fi
  path=${path#*:}
done

if [ $path_included -eq 0 ] ; then
  echo "warning: You don't have ${BASE_DIR}/current/rt/bin in your PATH."
fi

usage() {
  cat << EOS

Usage :
  svm [Action] [Scala-Version] [Options]

Action :
  -h|help             - show this usage information
  -c|current          - show the currently use scala version
  -l|list             - show the scala version installed in svm_path(default is $HOME/.svm)
  -v|versions         - show the abalabe scala version not installed
  -i|install          - install specific scala version
  -r|remove|uninstall - uninstall specific scala version and remove their sources
  -s|switch|-u|use    - setup to use a specific scala version
  update-latest       - install or update nightly build scala version
  latest              - setup to use nightly build scala version
  stable              - setup to use stable(x.x.x.final) scala version
  (any strings)       - setup to use specific scala version(shortcut of svm switch)


Options :
  --docs    - with install, update-latest. to download scala-devel-docs.
  --sources - with install, update-latest. to download scala-sources.

Info :
  ゆろよろ Tomohito Ozaki
  blog   : http://yuroyoro.hatenablog.com/
  twitter: http://twitter.com/yuroyoro/

EOS
}

yes_no() {
  MSG=$1
  while :
  do
    echo -n "${MSG} y/N: "
    read ans
    case $ans in
      [yY]|[yY][eE][sS]) return 0 ;;
      [nN]|[nN][oO]) return 1 ;;
    esac
  done
}

list(){
  case "`uname`" in
    Darwin*)   find ${BASE_DIR} -maxdepth 1 -d -print0 | xargs -0 basename | grep 'scala' | sed 's/scala-//' ;;
    *)         find ${BASE_DIR} -maxdepth 1 -depth -printf "%f\n" | grep 'scala' | sed 's/scala-//' ;;
  esac
}

versions(){
  case "`uname`" in
    Darwin*)   installed_versions=`find ${BASE_DIR} -maxdepth 1 -d -print0 | xargs -0 basename | grep 'scala' | sed 's/scala-//'` ;;
    *)         installed_versions=`find ${BASE_DIR} -maxdepth 1 -depth -printf "%f\n" | grep 'scala' | sed 's/scala-//'` ;;
  esac

  for v in ${AVAILABLE_VERSIONS}
  do
    for i in ${installed_versions}
    do
      if [ "${v}" = "${i}" ] ; then
        installed=1
      fi
    done
    if [ -z "${installed}" ] ;then
      echo "${v}"
    fi
    unset installed
  done
  cat << EOS
more previous releases
see http://www.scala-lang.org/node/165

EOS
}

download(){
  url="$1"
  echo "Downloading $url"
  if which wget > /dev/null 2>&1; then
    wget "$url"
  elif which curl > /dev/null 2>&1; then
    curl -O -L "$url"
  else
    echo >&2 "wget or curl is required"
    return 1
  fi
  if [ $? -ne 0 ]; then
    echo >&2 "Failed to download $url"
    return 1
  fi
}

install(){
  version=$1
  echo "Trying to installing version of ${version}."
  dist_url="http://www.scala-lang.org/files/archive"
  version_dir="${BASE_DIR}/scala-${version}"
  if [ -d "${version_dir}" ]
  then
    echo "${version_dir} is already exisits."
    yes_no "Do you want to replace this version?"
    if [ $? -ne 0 ]; then
      rm -rf "${version_dir}"
    else
      return 0;
    fi
  fi

  mkdir -p "${version_dir}"
  cd "${version_dir}"
  echo "in ${version_dir}"

  download "${dist_url}/scala-${version}.tgz"
  if [ $? -ne 0 ]; then
    cd "${PWD}"
    rm -rf "${version_dir}"
    return 1;
  fi

  tar -xf scala-${version}.tgz
  mv scala-${version} rt
  rm scala-${version}.tgz

  if [ ${docs_on} -eq 1 ] ; then
    download "${dist_url}/scala-${version}-devel-docs.tgz"
    if [ $? -ne 0 ]; then
      cd "${PWD}"
      rm -rf "${version_dir}"
      return 1;
    fi

    tar -xf scala-${version}-devel-docs.tgz
    mv scala-${version}-devel-docs devel-docs
    rm scala-${version}-devel-docs.tgz
  fi


  if [ ${source_on} -eq 1 ] ; then
    download "${dist_url}/scala-${version}-sources.tgz"
    if [ $? -ne 0 ]; then
      cd "${PWD}"
      rm -rf "${version_dir}"
      return 1;
    fi

    tar -xf scala-${version}-sources.tgz
    mv scala-${version}-sources sources
    rm scala-${version}-sources.tgz
  fi

  cd "${PWD}"
  echo "Successfully installed version of ${version}."
  echo "installed directory is ${version_dir}"

  return 0;
}

uninstall(){
  version=$1
  version_dir="${BASE_DIR}/scala-${version}"
  get_current current_ver

  if [ "${version}" = "${current_ver}" ] ; then
    yes_no "Do you want to remove currently version [${version}] ?"
    if [ $? -ne 0 ]; then
      return 0;
    else
      read_next_version next_version
    fi
  fi


  rm -rf "${version_dir}"
  echo -n "removed version ${version}"

  if [ ! -z "${next_version}" ];then
    switch "${next_version}"
  fi
  echo "Successfully uninstalled version of ${version}"

}

update_latest(){
  latest_dir="${BASE_DIR}/scala-latest"

  if [ -d "${latest_dir}" ] ; then
    mv "${latest_dir}" "${latest_dir}_bak"
  fi
  mkdir -p "${latest_dir}"
  cd "${latest_dir}"
  echo "in ${latest_dir}"

  ESCAPED=${NEXT_VERSION//\./\\\.}
  dist_url="http://www.scala-lang.org/files/archive/nightly/distributions"

  download "${dist_url}/scala-${NEXT_VERSION}-latest.tgz"
  if [ $? -ne 0 ]; then
    cd "${PWD}"
    rm -rf "${latest_dir}"
    if [ -d "${latest_dir}_bak" ] ;then
      mv "${latest_dir}_bak" "${latest_dir}"
    fi
    return 1;
  fi

  tar -xf scala-${NEXT_VERSION}-latest.tgz
  EXTRACTED_DIR=`find . -maxdepth 1 -depth -regex ".*scala-${ESCAPED}\.r.*"`
  rev="${EXTRACTED_DIR//scala-/}"
  mv "${EXTRACTED_DIR}" rt
  rm scala-${NEXT_VERSION}-latest.tgz

  if [ ${docs_on} -eq 1 ] ; then
    download "${dist_url}/scala-${NEXT_VERSION}-latest-devel-docs.tgz"
    if [ $? -ne 0 ]; then
      cd "${PWD}"
      rm -rf "${latest_dir}"
      if [ -d "${latest_dir}_bak" ] ;then
        mv "${latest_dir}_bak" "${latest_dir}"
      fi
      return 1;
    fi

    tar -xf scala-${NEXT_VERSION}-latest-devel-docs.tgz
    EXTRACTED_DIR=`find . -maxdepth 1 -depth -regex ".*scala-${ESCAPED}\.r.*"`
    mv "${EXTRACTED_DIR}" devel-docs
    rm scala-${NEXT_VERSION}-latest-devel-docs.tgz
  fi

  if [ ${source_on} -eq 1 ] ; then
    download "${dist_url}/scala-${NEXT_VERSION}-latest-sources.tgz"
    if [ $? -ne 0 ]; then
      cd "${PWD}"
      rm -rf "${latest_dir}"
      if [ -d "${latest_dir}_bak" ] ;then
        mv "${latest_dir}_bak" "${latest_dir}"
      fi
      return 1;
    fi

    tar -xf scala-${NEXT_VERSION}-latest-sources.tgz
    EXTRACTED_DIR=`find . -maxdepth 1 -depth -regex ".*scala-${ESCAPED}\.r.*"`
    mv "${EXTRACTED_DIR}" sources
    rm scala-${NEXT_VERSION}-latest-sources.tgz
  fi

  rm -rf "${latest_dir}_bak"
  cd "${PWD}"
  echo "Successfully updated scala-latest to revision of ${rev}"
  return 0;

}

read_next_version(){
  list
  get_stable stable
  echo ""
  echo -n "Please input currently set version.[${stable}]"
  read next_version
  if [ -z "${next_version}" ];then
    next_version="${stable}"
  fi
  eval $1="${next_version}"
  return 0;
}

confirm_switch(){
  version=$1
  yes_no "Do you want to change current runtime version to ${version}?"
  if [ $? -eq 0 ]; then
    switch "${version}"
    return 0;
  fi
  return 1;
}

switch(){
  version=$1
  if [ -z "${version}" ] ; then
    read_next_version version
  fi

  get_current current_ver

  if [ "${version}" = "${current_ver}" ] ; then
    echo "Nothing to do..."
    return 0;
  fi

  version_dir="${BASE_DIR}/scala-${version}"
  if [ ! -d "${version_dir}" ] ; then
    echo "Version ${version} is not installed."
    yes_no "Do you want to try to install version of ${version}?"
    if [ $? -eq 0 ]; then
      install "${version}"
      if [ $? -ne 0 ]; then
        return 1;
      fi
    else
      return 0;
    fi
  fi

  if [ -h "${BASE_DIR}/current" ] ; then
    rm "${BASE_DIR}/current"
  fi
  ln -fs "scala-${version}" "${BASE_DIR}/current"
  echo "currently version is $1"
}

current(){
  get_current currnet_version
  echo "currently version is ${currnet_version}"
}

get_current(){
  currnet_version=`readlink "${BASE_DIR}/current" | sed 's/scala-//'`
  eval $1="${currnet_version}"
  return 0;
}

get_stable(){
  case "`uname`" in
    Darwin*)   version=`find "${BASE_DIR}" -maxdepth 1 -d -print0 | xargs -0 basename | grep 'scala' | sed 's/scala-//' | grep "final" | tail -1` ;;
    *)         version=`find "${BASE_DIR}" -maxdepth 1 -depth -printf "%f\n" | grep 'scala' | sed 's/scala-//' | grep "final" | tail -1` ;;
  esac


  if [ -z "${version}" ] ; then
    case "`uname`" in
      Darwin*)   version=`find "${BASE_DIR}" -maxdepth 1 -d -print0 | xargs -0 basename | grep 'scala' | sed 's/scala-//' | grep -e "[0-9].*" | tail -1`;;
      *)         version=`find "${BASE_DIR}" -maxdepth 1 -depth -printf "%f\n" | grep 'scala' | sed 's/scala-//' | grep -e "[0-9].*" | tail -1` ;;
    esac
  fi

  eval $1="${version}"
  return 0;
}



cmd="$1"

# download scala-sources if source_on=1
source_on=0
# download scala-devel-docs if docs_on=1
docs_on=0

for o in $*
do
  case "${o}" in
    --sources)   source_on=1  ;;
    --docs)      docs_on=1    ;;
    * )             ;;
  esac
done

if [ -z "${cmd}" ] ; then
  current
else
  case "$cmd" in
    -h|help)             usage ;;
    -c|current)          current ;;
    -l|list)             list
                         current
                         ;;
    -v|versions)         versions ;;
    -i|install)          install $2
                         if [ $? -eq 0 ]; then
                           confirm_switch $2
                         fi
                         ;;
    -r|remove|uninstall) uninstall $2 ;;
    -s|-u|switch|use)    switch $2 ;;
    update-latest)       update_latest
                         if [ $? -eq 0 ]; then
                           confirm_switch "latest"
                         fi
                         ;;
    latest)              latest_dir="${BASE_DIR}/scala-latest"
                         if [ ! -d "${latest_dir}" ] ; then
                           update_latest
                         fi
                         switch "latest" ;;
    stable)              get_stable stable
                         switch "${stable}"
                         ;;
    * )                  switch "${cmd}" ;;
  esac
fi
