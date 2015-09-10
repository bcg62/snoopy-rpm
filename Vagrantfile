# -*- mode: ruby -*-
# # vi: set ft=ruby ts=2 sw=2 et :
# ###
# # Description: rpmbuild environment for snooy https://github.com/a2o/snoopy
# # Instructions: vagrant up
# ###

# confirm all required plugins are installed
unless ARGV.first == 'plugin'
  plugins = %w(
    vagrant-proxyconf
  )
  if plugins.detect { |p| !Vagrant.has_plugin?(p) }
    warn "Installing required plugins: #{plugins.join(' ')}"
    system "vagrant plugin install #{plugins.join(' ')}"
    exec 'vagrant', *ARGV
  end
end

buildessential = %w(
  automake
  bison
  flex
  gettext
  gcc
  gcc-c++
  make
  m4
  patch
  libtool
  socat
  git
  rpm-build
  curl
  spectool
  buildsys-macros
)

Vagrant.configure(2) do |config|
  # vagrant-proxyconf - will update vm to pass through proxy settings
  config.proxy.http     = ENV['http_proxy']
  config.proxy.https    = ENV['https_proxy']
  config.proxy.no_proxy = ENV['no_proxy']

  # system base image centos 6.7 x86_64
  config.vm.box = 'bento/centos-6.7'

  # system base image centos 6.7 i386
  # config.vm.box = 'bento/centos-6.7-i386'

  # system base image centos 5.10 x86_64
  # config.vm.box = 'bento/centos-5.10'
  # config.vm.box_url = 'http://opscode-vm-bento.s3.amazonaws.com/vagrant/virtualbox/opscode_centos-5.10_chef-provisionerless.box'

  # system base image centos 5.10 i386
  # config.vm.box = 'bento/centos-5.10-i386'
  # config.vm.box_url = 'http://opscode-vm-bento.s3.amazonaws.com/vagrant/virtualbox/opscode_centos-5.10-i386_chef-provisionerless.box'

  # some global provisioning steps
  config.vm.provision 'shell', inline: [

    # die on errors
    'set -e',

    # update packages

    'yum install -y epel-release',

    # epel breaks on 6
    '
      [[ $(cat /etc/redhat-release) =~ "release 6" ]] &&
        sed -i "s/mirrorlist=https/mirrorlist=http/" /etc/yum.repos.d/epel.repo
    ',

    'yum update -y',

    # install build essentials
    "yum install -y #{buildessential.join(' ')}"

  ].join("\n\n")

  config.vm.define 'snoopy-rpm' do |c|
    name = 'snoopy-rpm'

    c.vm.hostname = name

    c.vm.provision 'shell', inline: [

      # die on errors
      'set -e',

      # get compiled build tools into path
      '
        export PATH="/usr/local/bin:$PATH"
      ',

      # el5 old wget version cuts off file extenstions on redirects
      # http://superuser.com/questions/301044/how-to-wget-a-file-with-correct-name-when-redirected
      "
        [[ $(cat /etc/redhat-release) =~ 'release 5' ]] &&
        {
          grep -q content_disposition /etc/wgetrc ||
            echo 'content_disposition = on' >> /etc/wgetrc
        }
      ",

      # el5 autoconf version is too old
      # require autoconf 2.63, but have 2.59
      "
        [[ $(autoconf --version) =~ 2.6  ]] ||
        {
          cd /tmp
          curl -O http://ftp.gnu.org/gnu/autoconf/autoconf-latest.tar.gz

          tar -zxvf autoconf-latest.tar.gz
          cd /tmp/autoconf-*
          ./configure
          make
          make install
        }
      ",

      # el5 automake is too old
      # require Automake 1.11, but have 1.9.6
      "
        [[ $(automake --version) =~ 1.1 ]] ||
        {
          cd /tmp
          curl -O http://ftp.gnu.org/gnu/automake/automake-1.11.6.tar.gz

          tar -zxvf automake-1.11.6.tar.gz

          cd /tmp/automake-1.11.6
          ./configure
           make
           make install

          cd /usr/local/share
          ln -sf aclocal-1.11 aclocal
        }
      ",

      # libtool is also too old on el5
      "
      [[ $(cat /etc/redhat-release) =~ 'release 5' ]] &&
      {
        [[ -f /tmp/libtool-2.2.6b.tar.gz ]] ||
        {
          cd /tmp
          curl -O http://gnu.mirror.iweb.com/libtool/libtool-2.2.6b.tar.gz

          tar -zxvf libtool-2.2.6b.tar.gz

          cd libtool-2.2.6b
          ./configure
          make
          make install
        }
      }
      ",

      # build!
      "
        topdir=$(rpm --eval '%{_topdir}')
        mkdir -p $topdir/{BUILD,RPMS,SOURCES,SPECS,SRPMS}

        curl -o $topdir/SPECS/snoopy.spec \
          https://raw.githubusercontent.com/tkimball83/rpmbuild/master/SPECS/snoopy.spec

        cd $topdir/SPECS
        spectool -g -R snoopy.spec
        rpmbuild -bb snoopy.spec

        [[ -d /vagrant/artifacts ]] || mkdir /vagrant/artifacts
        cp -R $topdir/RPMS/ /vagrant/artifacts
      "

    ].join("\n\n")
  end
end
