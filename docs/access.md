# Getting access
describe how to generate+add SSH keys + provide a default SSH config file for all servers

connecting
  - SSH
  - x2go (only axomamma)
  - transferring files
  develop in fx VSCode

provide a full SSH config file with all servers?

SSH config proxy jump through sshgw.aau.dk:
Host sshgw.aau.dk
  HostName sshgw.aau.dk
  User ksa@bio.aau.dk
  Port 22
  ForwardAgent yes

Host test
  HostName bio-oscloud03.srv.aau.dk
  User ksa@bio.aau.dk
  Port 22
  ProxyJump sshgw.aau.dk