puppet-vyatta
=============

Generates and installs Vyatta configuration.

Tested on Vyatta Core 6.6 R1, patches are welcome.

## Installation

Clone this repo to your Puppet modules directory

    git clone git://github.com/ahaitoute/puppet-vyatta.git vyatta

## Requirements

* [File concatenation system for Puppet] (https://github.com/puppetlabs/puppetlabs-concat)
* [Puppet Labs Standard Library module] (https://github.com/puppetlabs/puppetlabs-stdlib)

## Usage

Define the server.

    class { 'vyatta':
      configuration_file => '/home/vyatta/configuration',
    }

### vyatta::interfaces

Define the interfaces.

    vyatta::interfaces::ethernet { 'eth0':
      configuration => {
        address => 'dhcp',
        duplex => 'auto',
        hw-id => $macaddress_eth0,
        smp_affinity => 'auto',
        speed => 'auto'
      }
    }
    vyatta::interfaces::ethernet { 'eth1':
      configuration => {
        'address 192.168.1.1/24' => '',
        'address 192.168.2.1/24' => '',
        hw-id => $macaddress_eth1,
        speed => 'auto'
      }
    }
    vyatta::interfaces::loopback { 'lo':
      configuration => {
        address => '10.0.0.10/32'
      }
    }
    vyatta::interfaces::openvpn { 'vtun0':
      configuration => {
        local-address => '10.4.0.1',
        local-port => '5000',
        mode => 'site-to-site',
        remote-address => '10.4.0.2',
        remote-host => '192.168.1.2',
        remote-port => '5001',
        tls => {
          ca-cert-file => '/config/auth/ca.crt',
          cert-file => '/config/auth/vyatta.crt',
          key-file => '/config/auth/vyatta.key',
          role => 'active'
        }
      }
    }

### vyatta::system

Define the system.

    vyatta::system::system { 'vyatta':
      configuration => {
        gateway-address => '10.0.2.2',
        host-name => 'vyatta',
        time-zone => 'Europe/Amsterdam'
      }
    }

    vyatta::system::config-management { 'config-management':
      configuration => {
        commit-archive => {
          location => "tftp://<ip-address tfp-server>/$hostname"
        },
        commit-revisions => '20'
      }
    }

    vyatta::system::login { 'login':
      configuration => {
        'user vyatta' => {
          authentication => {
            encrypted-password => '$6$GUyv4c3u7RZwjhRx$44.RQbxRI.nMEeV.ZJx61K7xMYQpAmOR8VjdWd3Wkz7TuG44eeygBoG2u9B3Jv8Cbfr0i.JTTwnrC5MDUkclI/', #Password: vyatta
            'public-keys user@host' => {
              key => 'Your public key',
              type => 'ssh-rsa'
            }
          },
          level => 'admin'
        },
        'user operator' => {
          authentication => {
            encrypted-password => '$6$GUyv4c3u7RZwjhRx$44.RQbxRI.nMEeV.ZJx61K7xMYQpAmOR8VjdWd3Wkz7TuG44eeygBoG2u9B3Jv8Cbfr0i.JTTwnrC5MDUkclI/', #Password: vyatta
          },
          level => 'operator'
        }
      }
    }
    vyatta::system::ntp { 'ntp':
      configuration => {
        'server 0.vyatta.pool.ntp.org' => {
          prefer => ''
        },
        'server 1.vyatta.pool.ntp.org' => {
        },
        'server 2.vyatta.pool.ntp.org' => {
        },
      }
    }
    vyatta::system::package { 'package':
      configuration => {
        'repository community' => {
          components => 'main',
          distribution => 'stable',
          url => 'http://packages.vyatta.com/vyatta'
        },
        'repository puppet' => {
          components => '"main dependencies"',
       	  distribution => 'stable',
          url => 'http://apt.puppetlabs.com'
        },
        'repository squeeze' => {
          components => 'main',
          distribution => 'stable',
          url => 'http://ftp.nl.debian.org/debian'
        }
      }
    }
    vyatta::system::syslog { 'syslog':
      configuration => {
        'file kernel-log' => {
          archive => {
            files => '10',
            size => '10485760'
          },
          'facility kern' => {
            level => 'info'
          }
        },
        global => {
          'facility all' => {
            level => 'notice'
          },
          'facility protocols' => {
            level => 'debug'
          }
        }
      }
    }

### vyatta::service

Define the service.

    vyatta::service::https { 'https':
    }
    vyatta::service::ssh { 'ssh':
      configuration => {
        port => 22
      }
    }

### vyatta::policy

Define the policy.

#### vyatta::policy::access_list

    vyatta::policy::access_list { '110':
      configuration => {
        description => '"Access list description"',
        'rule 10' => {
          action => 'permit',
          description => '"Rule 10 description."',
          destination => {
            any => '',
          },
          source => {
            inverse-mask => '0.0.0.63',
            network => '145.21.240.0'
          }
        }
      }
    }

#### vyatta::policy::prefix_list

    vyatta::policy::prefix_list { 'PREFIX-LIST':
      configuration => {
        'rule 1' => {
          action => 'permit',
       	  prefix => '192.168.0.0/16'
        },
        'rule 2' => {
          action => 'permit',
          description => '"Rule 2 description."',
          le => '15',
          prefix => '172.16.0.0/16'
        },
        'rule 3' => {
          action => 'permit',
          description => '"Rule 3 description."',
          le => '17',
          prefix => '10.10.0.0/16'
        },
      }
    }

#### vyatta::policy::route_map

    vyatta::policy::route_map { 'ROUTE-MAP':
      configuration => {
        description => '"Route-map description."',
        'rule 1' => {
          action => 'permit',
          description => '"Rule 1 description."',
          match => {
            ip => {
              address => {
                prefix-list => 'PREFIX-LIST'
              }
            }
          }
        }
      }
    }

### vyatta::protocols

Define the protocols.

#### vyatta::protocols::bgp

    vyatta::protocols::bgp { '65000':
      configuration => {
        'neighbor 192.168.1.10' => {
          'remote-as' => '65001',
          'update-source' => '192.168.1.1'
        },
        'neighbor 192.168.1.20' => {
          'remote-as' => '65002',
          'update-source' => '192.168.1.1'
        },
        'network 192.168.1.0/24' => {
          backdoor => ''
        },
        'network 192.168.2.0/24' => {
          route-map => 'ROUTE-MAP'
        },
        'network 192.168.3.0/24' => {
        },
        parameters => {
          router-id => '192.168.1.1'
        },
        redistribute => {
          connected => {
            metric => '1',
          },
          ospf => {
          },
          rip => {
            route-map => 'ROUTE-MAP'
          },
          static => {
            metric => '1',
            route-map => 'ROUTE-MAP'
          }
        }
      }
    }

#### vyatta::protocols::ospf

    vyatta::protocols::ospf { 'ospf':
      configuration => {
        'area 0.0.0.0' => {
          'network 192.168.1.0/24' => '',
          'network 192.168.2.0/24' => ''
        },
        default-information => {
          originate => {
            metric-type => '2'
          }
        },
        parameters => {
          abr-type => 'cisco',
          router-id => '192.168.1.1'
        },
        'passive-interface default' => '',
        'passive-interface-exclude eth0' => '',
        'passive-interface-exclude eth1' => '',
        redistribute => {
          bgp => {
            metric-type => '2',
          },
          connected => {
            metric => '1',
            metric-type => '2'
          },
          rip => {
            route-map => 'ROUTE-MAP'
          },
          static => {
            metric => '1',
            metric-type => '2',
            route-map => 'ROUTE-MAP'
          }
        }
      }
    }

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Added some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
