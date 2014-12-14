# encoding: utf-8
# vim: ft=ruby expandtab shiftwidth=2 tabstop=2

require 'yaml'

Vagrant.require_version '>= 1.5'

_conf = YAML.load(
  File.open(
    File.join(File.dirname(__FILE__), 'config/default.yml'),
    File::RDONLY
  ).read
)

if File.exists?(File.join(File.dirname(__FILE__), 'config/local.yml'))
  _local = YAML.load(
    File.open(
      File.join(File.dirname(__FILE__), 'config/local.yml'),
      File::RDONLY
    ).read
  )
  _conf.merge!(_local) if _local.is_a?(Hash)
end

if File.exists?('./site.yml')
  _site = YAML.load(
    File.open(
      File.join(ENV['VAGRANT_DOTFILE_PATH'], 'site.yml'),
      File::RDONLY
    ).read
  )
  _conf.merge!(_site) if _site.is_a?(Hash)
end

# path to the cookbooks (e.g. ~/vagrants/vccw)
if File.exists?(_conf['chef_cookbook_path'])
  chef_cookbooks_path = _conf['chef_cookbook_path']
elsif File.exists?(File.join(File.dirname(__FILE__), _conf['chef_cookbook_path']))
  chef_cookbooks_path = File.join(File.dirname(__FILE__), _conf['chef_cookbook_path'])
else
  puts "Can't find "+_conf['chef_cookbook_path']+'. Please check chef_cookbooks_path in the config.'
  exit 1
end


Vagrant.configure(2) do |config|

  config.vm.box = ENV['wp_box'] || _conf['wp_box']
  config.ssh.forward_agent = true

  config.vm.box_check_update = true

  config.vm.hostname = _conf['hostname']
  config.vm.network :private_network, ip: _conf['ip']

  config.vm.synced_folder 'www/wordpress/', '/var/www/wordpress', :create => 'true'

  if Vagrant.has_plugin?('vagrant-hostsupdater')
    config.hostsupdater.remove_on_suspend = true
  end

  config.vm.provider :virtualbox do |vb|
    vb.customize [
      'modifyvm', :id,
      '--natdnsproxy1', 'on',
      '--natdnshostresolver1', 'on'
    ]
  end

  if 'miya0001/vccw' != config.vm.box && 'provision' != ARGV[0]
    config.vm.provision 'shell',
        inline: 'curl -L https://www.opscode.com/chef/install.sh | sudo bash -s -- -v 11'
  end

  if File.exists?(File.join(File.expand_path(File.dirname(__FILE__)), 'provision', 'provision-pre.sh')) then
    config.vm.provision :shell, :path => File.join( 'provision', 'provision-pre.sh' )
  end

  config.vm.provision :chef_solo do |chef|

    chef.cookbooks_path = [
      File.join(chef_cookbooks_path, 'cookbooks'),
      File.join(chef_cookbooks_path, 'site-cookbooks')
    ]

    chef.json = {
      :apache => {
        :docroot_dir  => '/var/www/wordpress',
        :user         => 'vagrant',
        :group        => 'vagrant',
        :listen_ports => ['80', '443']
      },
      :php => {
        :packages => %w(php php-cli php-devel php-mbstring php-gd php-xml php-mysql),
        :directives => {
            'default_charset'            => 'UTF-8',
            'mbstring.language'          => 'neutral',
            'mbstring.internal_encoding' => 'UTF-8',
            'date.timezone'              => 'UTC',
            'short_open_tag'             => 'Off',
            'session.save_path'          => '/tmp'
        }
      },
      :mysql => {
        :bind_address           => '0.0.0.0',
        :server_debian_password => 'wordpress',
        :server_root_password   => 'wordpress',
        :server_repl_password   => 'wordpress'
      },
      'wpcli' => {
        :wp_version        => ENV['wp_version'] || _conf['version'],
        :wp_host           => _conf['hostname'],
        :wp_home           => _conf['wp_home'],
        :wp_siteurl        => _conf['wp_siteurl'],
        :locale            => ENV['wp_lang'] || _conf['lang'],
        :admin_user        => _conf['admin_user'],
        :admin_password    => _conf['admin_pass'],
        :default_plugins   => _conf['plugins'],
        :default_theme     => _conf['theme'],
        :title             => _conf['title'],
        :is_multisite      => _conf['multisite'],
        :force_ssl_admin   => _conf['force_ssl_admin'],
        :debug_mode        => _conf['wp_debug'],
        :savequeries       => _conf['savequeries'],
        :theme_unit_test   => _conf['theme_unit_test'],
        :theme_unit_test_data_url => _conf['theme_unit_test_uri'],
        :always_reset      => _conf['reset_db'],
        :dbhost            => _conf['db_host'],
        :dbprefix          => _conf['db_prefix'],
        :options           => _conf['options'],
        :rewrite_structure => _conf['rewrite_structure']
      },
      :vccw => {
        :wordmove => {
          :movefile        => File.join('/vagrant', 'Movefile'),
          :url             => 'http://' << File.join(_conf['hostname'], _conf['wp_home']),
          :wpdir           => File.join('www/wordpress', _conf['wp_siteurl']),
          :dbhost          => _conf['db_host']
        }
      },
      :rbenv => {
        'rubies'  => ['2.1.2'],
        'global'  => '2.1.2',
        'gems'    => {
          '2.1.2' => [
            {
              name: 'bundler',
              options: '--no-ri --no-rdoc'
            },
            {
              name: 'sass',
              options: '--no-ri --no-rdoc'
            },
            {
              name: 'wordmove',
              options: '--no-ri --no-rdoc'
            }
          ]
        }
      }
    }

    chef.add_recipe 'yum::remi'
    chef.add_recipe 'iptables'
    chef.add_recipe 'apache2'
    chef.add_recipe 'apache2::mod_php5'
    chef.add_recipe 'apache2::mod_ssl'
    chef.add_recipe 'mysql::server'
    chef.add_recipe 'mysql::ruby'
    chef.add_recipe 'php::package'
    chef.add_recipe 'wpcli'
    chef.add_recipe 'wpcli::install'
    chef.add_recipe 'vccw'

  end

  if File.exists?(File.join(File.expand_path(File.dirname(__FILE__)), 'provision', 'provision-post.sh')) then
    config.vm.provision :shell, :path => File.join( 'provision', 'provision-post.sh' )
  end

end
