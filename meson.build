project('phosphor-user-manager',
        'cpp',
        version: '0.1', meson_version: '>=0.57.0',
        default_options: [
          'warning_level=3',
          'werror=true',
          'cpp_std=c++20',
          'buildtype=debugoptimized',
        ]
)

conf_data = configuration_data()

conf_data.set_quoted('USER_MANAGER_BUSNAME', 'xyz.openbmc_project.User.Manager',
                      description : 'The DBus busname to own.')

conf_data.set_quoted('LDAP_MAPPER_MANAGER_BUSNAME', 'xyz.openbmc_project.LDAP.PrivilegeMapper',
                      description : 'The Dbus busname LDAP privilege mapper application.')

conf_data.set_quoted('LDAP_MAPPER_PERSIST_PATH', '/var/lib/phosphor-ldap-mapper/groups',
                      description : 'Path of directory containing LDAP groups privilege mapping.')

conf_data.set_quoted('DEFAULT_CRYPT_ALGO', '1',
                      description : 'The default crypt algorithm if one not found in shadow.')

conf_data.set('CLASS_VERSION', 1,
                      description : 'Class version to register with Cereal.')

conf_data.set_quoted('LDAP_CONFIG_FILE', '/etc/nslcd.conf',
                      description : 'Path of LDAP configuration file.')

conf_data.set_quoted('TLS_CACERT_PATH', '/etc/ssl/certs/authority',
                      description : 'Path of LDAP server CA certificate.')

conf_data.set_quoted('TLS_CERT_FILE', '/etc/nslcd/certs/cert.pem',
                      description : 'Path of LDAP client certificate.')

conf_data.set_quoted('LDAP_CONFIG_ROOT', '/xyz/openbmc_project/user/ldap',
                      description : 'LDAP configuration root.')

conf_data.set_quoted('LDAP_CONFIG_DBUS_OBJ_PATH', '/xyz/openbmc_project/user/ldap/config',
                      description : 'D-Bus path of LDAP config object.')

conf_data.set_quoted('LDAP_CONFIG_BUSNAME', 'xyz.openbmc_project.Ldap.Config',
                      description : 'D-Bus busname of LDAP config service.')

conf_data.set_quoted('LDAP_CONF_PERSIST_PATH', '/var/lib/phosphor-ldap-conf',
                      description : 'path of directory having persisted LDAP configuration enabled property.')

conf_data.set_quoted('SYSTEMD_BUSNAME', 'org.freedesktop.systemd1',
                      description : 'systemd busname.')

conf_data.set_quoted('SYSTEMD_PATH', '/org/freedesktop/systemd1',
                      decsription :  'systemd path')

conf_data.set_quoted('SYSTEMD_INTERFACE', 'org.freedesktop.systemd1.Manager',
                      description : 'systemd interface.')

conf_header = configure_file(output: 'config.h',
    configuration: conf_data)

phosphor_dbus_interfaces_dep = dependency(
  'phosphor-dbus-interfaces',
  fallback: ['phosphor-dbus-interfaces', 'phosphor_dbus_interfaces_dep'],
)

sdbusplus_dep = dependency(
  'sdbusplus',
  fallback: ['sdbusplus', 'sdbusplus_dep'],
)

phosphor_logging_dep = dependency(
  'phosphor-logging',
   fallback: ['phosphor-logging', 'phosphor_logging_dep'],
)

systemd_dep = dependency('systemd')

cpp = meson.get_compiler('cpp')

user_manager_src = [
    'mainapp.cpp',
    'user_mgr.cpp',
    'users.cpp'
]

user_manager_deps = [
     sdbusplus_dep,
     phosphor_logging_dep,
     phosphor_dbus_interfaces_dep
]

user_manager_lib = static_library(
    'phosphor-user-manager',
    [
        'user_mgr.cpp',
        'users.cpp',
    ],
    dependencies: user_manager_deps,
)

user_manager_dep = declare_dependency(
    link_with: user_manager_lib,
    dependencies: user_manager_deps
)

executable(
    'phosphor-user-manager',
    'mainapp.cpp',
    dependencies: user_manager_dep,
    link_args: ['-lcrypt', '-lstdc++fs'],
    cpp_args: ['-DBOOST_ALL_NO_LIB', '-DBOOST_SYSTEM_NO_DEPRECATED', '-DBOOST_ERROR_CODE_HEADER_ONLY'],
    install: true,
)


systemd_system_unit_dir = systemd_dep.get_variable(
    pkgconfig: 'systemdsystemunitdir'
)

busconfig_dir = get_option('datadir') / 'dbus-1' / 'system.d'

cert_manager_dir = get_option('datadir') / 'phosphor-certificate-manager'

certs = []

busconfig = []

systemd_alias = []

certs += 'nslcd'

busconfig += 'phosphor-nslcd-cert-config.conf'

systemd_alias += [[
        '../phosphor-certificate-manager@.service',
        'multi-user.target.wants/phosphor-certificate-manager@nslcd.service'
]]


install_data(
    busconfig,
    install_dir: busconfig_dir,
)

install_data(
    certs,
    install_dir: cert_manager_dir,
)

foreach service: systemd_alias
    # Meson 0.61 will support this:
    #install_symlink(
    #      service,
    #      install_dir: systemd_system_unit_dir,
    #      pointing_to: link,
    #  )
    meson.add_install_script(
        'sh', '-c',
        'mkdir -p $(dirname $DESTDIR/@0@/@1@)'.format(systemd_system_unit_dir,
            service[1]),
    )
    meson.add_install_script(
        'sh', '-c',
        'ln -s @0@ $DESTDIR/@1@/@2@'.format(service[0], systemd_system_unit_dir,
            service[1]),
    )
endforeach

subdir('phosphor-ldap-mapper')
subdir('phosphor-ldap-config')

if get_option('tests').enabled()
  subdir('test')
endif
