project(
  'libssh2',
  'c',
  version: '1.10.0',
  license: 'BSD-3',
  meson_version: '>=0.52.0',
  default_options: 'c_std=gnu99',
)

cc = meson.get_compiler('c')
pkgconfig = import('pkgconfig')
libssh2_config = configuration_data()

cargs = ['-DHAVE_CONFIG_H']

if host_machine.system() == 'windows'
  if cc.get_argument_syntax() == 'msvc'
    add_project_arguments('-D_CRT_SECURE_NO_WARNINGS', cc.get_supported_arguments('-Wno-deprecated-declarations'), language: 'c')
  endif

  if get_option('default_library') != 'static' and cc.has_function_attribute('dllimport')
    libssh2_config.set('LIBSSH2_API', '__declspec(dllimport)')
  endif
else
  if cc.has_function_attribute('visibility:default')
    libssh2_config.set('LIBSSH2_API', '__attribute__ ((__visibility__ ("default")))')
  endif
endif

check_headers = [
  'unistd.h',
  'inttypes.h',
  'sys/select.h',
  'sys/uio.h',
  'sys/socket.h',
  'sys/param.h',
  'netinet/in.h',
  'arpa/inet.h',
  'sys/ioctl.h',
  'sys/time.h',
  'sys/un.h',
]

foreach h : check_headers
  libssh2_config.set('HAVE_@0@'.format(h.underscorify().to_upper()), cc.has_header(h))
endforeach

libssh2_config.set('HAVE_O_NONBLOCK', cc.has_header_symbol('fcntl.h', 'O_NONBLOCK'))
libssh2_config.set('HAVE_FIONBIO', cc.has_header_symbol('sys/ioctl.h', 'FIONBIO'))
libssh2_config.set('HAVE_IOCTLSOCKET_CASE', cc.has_header_symbol('winsock2.h', 'ioctlsocket'))
libssh2_config.set('HAVE_SNPRINTF', cc.has_header_symbol('stdio.h', 'snprintf'))

functions = [
  'explicit_bzero',
  'explicit_memset',
  'gettimeofday',
  'socket',
]

if host_machine.system() == 'windows'
  functions += ['select', 'memset_s', 'strtoi64']
else
  functions += ['poll', 'strtoll']
endif

foreach f : functions
  libssh2_config.set('HAVE_@0@'.format(f.underscorify().to_upper()), cc.has_function(f))
endforeach

libssh2_config.set('LIBSSH2DEBUG', get_option('buildtype') == 'debug')

libssh2_config_h = configure_file(
  input: 'src/libssh2_config_cmake.h.in',
  output: 'libssh2_config.h',
  configuration: libssh2_config,
  format: 'cmake',
)

libssh2_sources = files(
  'src/agent.c',
  'src/agent_win.c',
  'src/bcrypt_pbkdf.c',
  'src/blowfish.c',
  'src/channel.c',
  'src/comp.c',
  'src/crypt.c',
  'src/crypto.c',
  'src/global.c',
  'src/hostkey.c',
  'src/keepalive.c',
  'src/kex.c',
  'src/knownhost.c',
  'src/mac.c',
  'src/misc.c',
  'src/packet.c',
  'src/pem.c',
  'src/publickey.c',
  'src/scp.c',
  'src/session.c',
  'src/sftp.c',
  'src/transport.c',
  'src/userauth.c',
  'src/userauth_kbd_packet.c',
  'src/version.c',
)

libssh2_deps = []

if host_machine.system() == 'windows'
  win = import('windows')
  libssh2_sources += win.compile_resources('win32/libssh2.rc')
  add_project_arguments('-DLIBSSH2_WIN32', language: 'c')
elif host_machine.system() == 'darwin'
  add_project_arguments('-DLIBSSH2_DARWIN', language: 'c')
endif

crypto = get_option('crypto_backend')
if crypto == 'auto'
  if host_machine.system() == 'windows'
    crypto = 'wincng'
  else
    crypto = 'openssl'
  endif
endif

if crypto == 'openssl'
  if meson.version().version_compare('>= 0.62.0')
    openssl_dep = dependency('openssl', version: '>= 1.0.0', required: get_option('crypto_backend') == 'openssl')
  else
    openssl_dep = cc.find_library('libcrypto', required: get_option('crypto_backend') == 'openssl')
  endif
  if openssl_dep.found()
    libssh2_sources += 'src/openssl.c'
    libssh2_deps += openssl_dep
    cargs += '-DLIBSSH2_OPENSSL'
  else
    crypto = 'gcrypt'
  endif
endif

if crypto == 'gcrypt'
  gcrypt_dep = dependency('libgcrypt', required: get_option('crypto_backend') == 'gcrypt')
  if gcrypt_dep.found()
    libssh2_sources += 'src/libgcrypt.c'
    libssh2_deps += gcrypt_dep
    cargs += '-DLIBSSH2_LIBGCRYPT'
  else
    crypto = 'mbedtls'
  endif
endif

if crypto == 'mbedtls'
  libmbedtls = cc.find_library('mbedtls', required: get_option('crypto_backend') == 'mbedtls')
  libmbedx509 = cc.find_library('mbedx509', required: get_option('crypto_backend') == 'mbedtls')
  libmbedcrypto = cc.find_library('mbedcrypto', required: get_option('crypto_backend') == 'mbedtls')
  if libmbedtls.found() and libmbedx509.found() and libmbedcrypto.found()
    libssh2_sources += 'src/mbedtls.c'
    libssh2_deps += [libmbedtls, libmbedx509, libmbedcrypto]
    cargs += '-DLIBSSH2_MBEDTLS'
  else
    error('No crypto backends available')
  endif
endif

if crypto == 'wincng'
  if host_machine.system() != 'windows'
    error('WinCNG is a Windows only backend')
  endif
  libssh2_sources += 'src/wincng.c'
  libssh2_deps += [cc.find_library('bcrypt'), cc.find_library('crypt32')]
  cargs += '-DLIBSSH2_WINCNG'
endif

ws_dep = cc.find_library('ws2_32', required: host_machine.system() == 'windows')
libssh2_deps += ws_dep

inc_dir = include_directories('include')
abi_version = '1.0.1'
libssh2_lib = library(
  'ssh2',
  libssh2_sources,
  c_args: cargs,
  include_directories: inc_dir,
  dependencies: libssh2_deps,
  install: true,
  version: abi_version,
  soversion: host_machine.system() == 'windows' ? '' : '1',
)

libssh2_dep = declare_dependency(
  include_directories: inc_dir,
  link_with: libssh2_lib,
)

install_headers(
  libssh2_config_h,
  'include/libssh2.h',
  'include/libssh2_publickey.h',
  'include/libssh2_sftp.h',
)

pkgconfig.generate(
  libssh2_lib,
  name: 'libssh2',
  description: 'Library for SSH-based communication',
)

if get_option('examples')
  examples_inc = include_directories('.', 'src')
  examples_dep = declare_dependency(
    compile_args: cargs,
    dependencies: libssh2_dep,
    include_directories: examples_inc,
  )

  subdir('example')
endif
