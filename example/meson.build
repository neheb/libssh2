examples_c = [
  'direct_tcpip',
  'scp',
  'scp_nonblock',
  'scp_write',
  'scp_write_nonblock',
  'sftp',
  'sftp_RW_nonblock',
  'sftp_append',
  'sftp_mkdir',
  'sftp_mkdir_nonblock',
  'sftp_nonblock',
  'sftp_write',
  'sftp_write_nonblock',
  'sftp_write_sliding',
  'sftpdir',
  'sftpdir_nonblock',
  'ssh2',
  'ssh2_agent',
  'ssh2_echo',
  'ssh2_exec',
  'subsystem_netconf',
  'tcpip-forward',
]

foreach e : examples_c
  executable(e, '@0@.c'.format(e), dependencies: examples_dep)
endforeach
