---
title: My Services on Oracle Cloud Free Tier
date: 2025-08-11 10:10:00
categories: [linux]
tags: [linux]     # TAG names should always be lowercase
---
Oracle cloud gives a pretty generous free tier. 4 vCPU, 24gb ram, and 200gb of block storage which can be broken down and used however you like.
I chose to split this up into 3 instances:
1-2 vCPU with 12gb of ram for my VPN
2-1 vCPU with 6gb ram for my emails
3-1 vCPU with 6gm ram for some services like my uptime monitor, stack monitoring (grafana), and personal file backup

The free tier forces you to use an Oracle-modified image, of which I had no interest in running, therefore I ran their image and "infected it" with NixOS, meaning I was able to install NixOS on top of their running linux without having to have access to pre-boot installation method. If you are interested in learning about this, it is the "kexec" method, and there are a few examples of tools that try and automate this process and althouh I did it manually, searching for "nixos-anywhere" or "nix-infect" might get you going in the right direction if you are interested in trying this.
The main thing about doing this on Oracle, is to pay attention to the networking settings as your instance is attributed a fixed internal IP address and there is no DHCP so without that, your NixOS installation will not get network connection and you will not be able to SSH into your instance once installed, so it might take some figuring out on your end to get the right IP information in your nix config to get it going.

vpn with AmneziaWG:
```nix
{ config, pkgs, lib, ... }:
let 
  NM_DOMAIN = "ymrtech.com";
in
{
  imports =
    [ # Include the results of the hardware scan.
      ./hardware-configuration.nix
    ];

  boot.loader.systemd-boot.enable = true;
  boot.loader.efi.canTouchEfiVariables = true;
  boot.kernel.sysctl."net.ipv4.ip_forward" = 1;
  fileSystems = {
    "/".options = [ "compress=zstd" "noatime" "nodiratime" "discard" ];
    "/home".options = [ "compress=zstd" ];
    "/nix".options = [ "compress=zstd" ];
  };
  time.timeZone = "America/Toronto";
  nixpkgs.config.allowUnfree = true;
  networking = {
    hostName = "vpn";
    defaultGateway = {
      interface = "eth0";
      address = "10.0.0.1";
    };
    useDHCP = false;
    networkmanager = {
      enable = true;
    };
    enableIPv6 = false;
    nat = {
      enable = true;
      enableIPv6 = false;
      externalInterface = "eth0";
      internalInterfaces = [ "wg0" "wg1" ];
#      internalIPs = [ "11.0.0.1/24" ];
      externalIP = "10.0.0.175";
    };
    interfaces.eth0.ipv4.addresses = [
    {
      address = "10.0.0.175";
      prefixLength = 24;
    }];

    interfaces.eth1.ipv4.addresses = [
    {
      address = "10.0.0.169";
      prefixLength = 24;
    }];

    # Open ports in the firewall
    firewall = {
      allowedUDPPorts = [ 41020 41021 ];
      extraCommands = ''
        iptables -A nixos-fw -p tcp --source 11.0.0.0/24  -j nixos-fw-accept
        iptables -A nixos-fw -p udp --source 11.0.0.0/24  -j nixos-fw-accept
      '';
    };
    wg-quick.interfaces = {
    #personal VPN
      wg0 = {
        type = "amneziawg";
        extraOptions = {
	    Jc = 6;
	    Jmin = 48;
	    Jmax = 144;
  	    S1 = 16;
  	    S2 = 32;
        H1 = 1;
        H2 = 2;
        H3 = 3;
	    H4 = 4;
	  };
        address = [ "11.0.0.1/24" ];
        listenPort = 41020;
        privateKeyFile = "/home/truva/wireguard-keys/private";
        postUp = ''
          ${pkgs.iptables}/bin/iptables -A FORWARD -i wg0 -j ACCEPT
          ${pkgs.iptables}/bin/iptables -t nat -A POSTROUTING -s 11.0.0.1/24 -o eth0 -j MASQUERADE
        '';
        preDown = ''
          ${pkgs.iptables}/bin/iptables -D FORWARD -i wg0 -j ACCEPT
          ${pkgs.iptables}/bin/iptables -t nat -D POSTROUTING -s 11.0.0.1/24 -o eth0 -j MASQUERADE
        '';

        peers = [
          { # captain
            publicKey = "sSp6r/JEFjXkYVOzaoECbvpZFPOQNmMbT3EYSv5ykl8=";
            presharedKeyFile = "/home/truva/wireguard-keys/captain.psk";
            allowedIPs = [ "11.0.0.2/32" ];
          }
          { # command
            publicKey = "Gz81fsh4ExB7nYbD80a6GZByxWLV8ywItVBjaf07Myg=";
            presharedKeyFile = "/home/truva/wireguard-keys/command.psk";
            allowedIPs = [ "11.0.0.3/32" ];
          }
          { # pixel
            publicKey = "M2VIiUXg42Idennf+wd+h4DIX4bwFVP1+jSjK+Kdr14=";
            allowedIPs = [ "11.0.0.4/32" ];
          }
          { # giga
            publicKey = "xP82Jap3FYyD9wO6m0gDhT+kLibsbt158ABbhNuIDH8=";
            presharedKeyFile = "/home/truva/wireguard-keys/giga.psk";
            allowedIPs = [ "11.0.0.5/32" ];
          }
          { # public
            publicKey = "5BKRJEGUMC7ZTZ8Gnd/UH8EPWICC6kbClICA1m+zExw=";
            presharedKeyFile = "/home/truva/wireguard-keys/public.psk";
            allowedIPs = [ "11.0.0.7/32" ];
          }
          # More peers can be added here.
        ];
      };
      #parents
      wg1 = {
        address = [ "12.0.0.1/24" ];
        listenPort = 41021;
        privateKeyFile = "/home/truva/wireguard-keys/private";
        postUp = ''
          ${pkgs.iptables}/bin/iptables -A FORWARD -i wg1 -j ACCEPT
          ${pkgs.iptables}/bin/iptables -t nat -A POSTROUTING -s 12.0.0.1/24 -o eth0 -j MASQUERADE
        '';
        preDown = ''
          ${pkgs.iptables}/bin/iptables -D FORWARD -i wg1 -j ACCEPT
          ${pkgs.iptables}/bin/iptables -t nat -D POSTROUTING -s 12.0.0.1/24 -o eth0 -j MASQUERADE
        '';

        peers = [
          { # mom    
            publicKey = "ro8MqMAo/lV2Q+Z1HgMP/ADCJDilcNSS/bRRHuLnRxQ=";
            allowedIPs = [ "12.0.0.2/32" ];
          }
          { # dad 
            publicKey = "lnR8OUlxTksqIXCKImiML3GlYC9Yb8ddRYwOK+MeoXE=";
            allowedIPs = [ "12.0.0.3/32" ];
          }
          # More peers can be added here.
        ];
      };
    # More wireguard interfaces can be added here.
    };
  };
  
  i18n.defaultLocale = "en_US.UTF-8";
  programs.wireshark.enable = true;  
  users.users.truva = {
    isNormalUser = true;
    extraGroups = [ "wheel" "wireshark" ]; # Enable ‘sudo’ for the user.
    openssh.authorizedKeys.keyFiles = [
      ./key.pub
    ];
  };
  environment.systemPackages = with pkgs; [
    dig
    fish 
    dnsutils
    wireshark
  ];

  services = {
    adguardhome = {
      enable = true;
      settings = {
        http = {
          address = "127.0.0.1:3000";
        };
        dns = {
          upstream_dns = [ "127.0.0.1:5353" ];
        };
        filtering = {
          protection_enabled = true;
          filtering_enabled = true;
          parental_enabled = false;
          safe_search = {
            enabled = false;
          };
        };
	filters = map(url: { enabled = true; url = url; }) [
          "https://raw.githubusercontent.com/ppfeufer/adguard-filter-list/refs/heads/master/blocklist"
        ];
      };
    };
    btrfs.autoScrub = {
      enable = true;
      interval = "weekly";
      fileSystems = [ "/" ];
    };
    openssh = {
      enable = true;
      settings = {
        PasswordAuthentication = false;
        PermitRootLogin = "no";
      };
    };
    unbound = {
      enable = true;
      settings = {
        server = {
          access-control = "11.0.0.0/24 allow";
          do-ip6 = "no";
          aggressive-nsec = "yes";
          hide-identity = "yes";
          hide-version = "yes";
          prefetch = "yes";
          interface = [ "0.0.0.0" ];
          port =  "5353";
          rrset-roundrobin = "yes";
          so-reuseport = "yes";
#          do-tcp = "no";
          use-caps-for-id = "yes";
          private-address = "11.0.0.0/24";
          num-threads = "4";
          msg-cache-slabs = "8";
          rrset-cache-slabs = "8";
          infra-cache-slabs = "8";
          key-cache-slabs = "8";
          msg-cache-size = "512M";
          rrset-cache-size = "1024M";
          outgoing-range = "8192";
#          so-rcvbuf = "4m";
          num-queries-per-thread = "4096";
          private-domain = "ymr";
          local-zone = ''"ymr." static'';
          local-data = [
            ''"vpn.ymr.	IN A 11.0.0.1"''
            ''"captain.ymr.	IN A 11.0.0.2"''
            ''"command.ymr.	IN A 11.0.0.3"''
            ''"public.ymr.	IN A 11.0.0.7"''
            ''"mail.ymr.      IN A 168.138.71.187"''
          ];
        };
        forward-zone = [
          {
            name = ".";
            forward-tls-upstream = "yes";
            forward-addr = [
              "149.112.112.112@853#dns.quad9.net"
              "1.0.0.1@853#one.one.one.one"
            ];
          }
        ];
      };
    };
  };
 
  system.stateVersion = "23.05"; # Did you read the comment?
}
```
Email server:
```nix
{ config, pkgs, ... }: {
  imports = [
    ./hardware-configuration.nix
    (builtins.fetchTarball {
      # Pick a release version you are interested in and set its hash, e.g.
      url = "https://gitlab.com/simple-nixos-mailserver/nixos-mailserver/-/archive/master/nixos-mailserver-master.tar.gz";
      # To get the sha256 of the nixos-mailserver tarball, we can use the nix-prefetch-url command:
      # release="nixos-24.11"; nix-prefetch-url "https://gitlab.com/simple-nixos-mailserver/nixos-mailserver/-/archive/${release}/nixos-mailserver-${release}.tar.gz" --unpack
      sha256 = "1b9aw2sn7imcziqdaqf6q3zq7c3pji0h3dgf6dgjwz8gn6zrhdbm";
    })
  ];

  # Use the systemd-boot EFI boot loader.
  boot.loader.systemd-boot.enable = true;
  boot.loader.efi.canTouchEfiVariables = true;
  nix.settings.auto-optimise-store = true;
  fileSystems = {
    "/".options = [ "compress=zstd" "noatime" "nodiratime" "discard" ];
    "/home".options = [ "compress=zstd" ];
    "/nix".options = [ "compress=zstd"  ];
    "/srv/emails" = {
      device = "/dev/disk/by-uuid/57cab892-dfa1-497a-9580-6fb13c5fcd9f";
      fsType = "btrfs"; 
      options = [ "subvol=emails" "compress=zstd" "noatime" "nodiratime" "discard" ];
    };
  };
  time.timeZone = "America/Toronto";
  nixpkgs.config.allowUnfree = true;
  networking = {
    hostName = "mail";
    defaultGateway = {
      interface = "eth0";
      address = "10.0.0.1";
    };
    useDHCP = false;
    nameservers = [ "1.1.1.1" "9.9.9.9" ];
    networkmanager = {
      enable = true;
      dns = "systemd-resolved";
    };
    enableIPv6 = false;
    interfaces.eth0.ipv4.addresses = [
    {
      address = "10.0.0.184";
      prefixLength = 24;
    }];
    # Open ports in the firewall
    firewall = {
      allowedTCPPorts = [ 80 443 ];
    };
  };
  i18n.defaultLocale = "en_US.UTF-8";
  environment.systemPackages = with pkgs; [
    dig
    ncdu
    fish 
    dnsutils
  ];
  users.users.truva = { 
    isNormalUser = true;
    extraGroups = [ "wheel" ]; # Enable ‘sudo’ for the user.
    openssh.authorizedKeys.keyFiles = [
      ./key.pub
    ];
  };
  security.acme = {
    defaults.email = "info@ymrtech.com";
    acceptTerms = true;
  };
  services = {
    btrfs.autoScrub = {
      enable = true;
      interval = "weekly";
      fileSystems = [ "/" ];
    };
    openssh = {
      enable = true;
      settings = {
        PasswordAuthentication = false;
        PermitRootLogin = "no";
      };
    };
   nginx = {
      enable = true;
      recommendedProxySettings = true;
      recommendedTlsSettings = true;
      recommendedGzipSettings = true;
      recommendedOptimisation = true;
      sslCiphers = "AES256+EECDH:AES256+EDH:!aNULL";
      sslProtocols = "TLSv1.3";
      virtualHosts."mail.ymrtech.com" =  {
        enableACME = true;
        forceSSL = true;
        locations."/" = {
          return  = "301 http://www.ymrtech.com/";
        };
      };
    };
    vsftpd = {
      enable = false;
      userlistDeny = false;
      localUsers = true;
      userlist = ["truva"];
    };
    fail2ban = {
      enable = false;
      ignoreIP = [
        "168.138.92.78"
        "168.138.71.187"
        "168.138.75.142"
        "localhost"
      ];
      maxretry = 2;
      bantime-increment = {
        enable = true;
        maxtime = "7d"; 
      };
      jails = {
        dovecot = ''
          enabled = true
          filter = dovecot[mode=aggressive]
        '';
        postfix = ''
          enabled = true
          port = smtp,465,submission
          filter   = postfix[mode=aggressive]
        '';
      };
    };
  };
  mailserver = {
    enable = true;
    localDnsResolver = false;
    fqdn = "mail.ymrtech.com";
    domains = [ "ymrtech.com" ];
    mailDirectory = "/srv/emails";
    fullTextSearch = {
      enable = true;      
      autoIndex = true; #index new email as they arrive
      autoIndexExclude = [ "Junk" ];
      enforced = "body";
    };
    # A list of all login accounts. To create the password hashes, use
    # nix-shell -p mkpasswd --run 'mkpasswd -sm bcrypt'
    loginAccounts = {
        "yannick@ymrtech.com" = {
            hashedPasswordFile = "/etc/nixos/ymrtechemail";
            aliases = ["info@ymrtech.com"];
        };
        "nextcloud@ymrtech.com" = {
            hashedPasswordFile = "/etc/nixos/nextcloudemail";
            #aliases = ["info@ymrtech.com"];
        };
    };
    certificateScheme = "acme";    
    virusScanning = false;
    enableSubmission = false;
    enableSubmissionSsl = false;
    rebootAfterKernelUpgrade.enable = true;
    monitoring.alertAddress = "yannick@ymrtech.com";
    monitoring.enable = true;
    dmarcReporting.domain = "ymrtech.com";
    dmarcReporting.enable = true;
    dmarcReporting.organizationName = "YMR Technologies";    
    stateVersion = 3;

  };
  system.stateVersion = "24.05"; # Did you read the comment?

}

```
Public services & backup:
```nix
{ config, lib, pkgs, ... }:

{
  imports = [
    ./hardware-configuration.nix
  ];
 
  boot.loader.efi.canTouchEfiVariables = true;
  boot.loader.systemd-boot.enable = true;
  nix.settings.auto-optimise-store = true;
  fileSystems = {
    "/".options = [ "compress=zstd" "noatime" "nodiratime" "discard" ];
    "/home".options = [ "compress=zstd" ];
    "/nix".options = [ "compress=zstd" ];
    };
  time.timeZone = "America/Toronto";
  nixpkgs.config.allowUnfree = true;
  networking = {
    hostName = "public";
    defaultGateway = {
      interface = "eth0";
      address = "10.0.0.1";
    };
    useDHCP = false;
    nameservers = [ "1.1.1.1" "9.9.9.9" ];
    networkmanager = {
      enable = true;
#      dns = "default"
#      dns = "systemd-resolved";
    };
    enableIPv6 = false;
    interfaces.eth0.ipv4.addresses = [
    {
      address = "10.0.0.220";
      prefixLength = 24;
    }];
    # Open ports in the firewall
    firewall = {
      allowedTCPPorts = [ 80 443 ];
##      allowedUDPPorts = [ 41020 ];
      extraCommands = '' 
        iptables -A nixos-fw -p tcp --source 11.0.0.0/24  -j nixos-fw-accept
        iptables -A nixos-fw -p udp --source 11.0.0.0/24  -j nixos-fw-accept
      '';
    };
    wg-quick.interfaces = {
      wg0 = {
        type = "amneziawg";
        extraOptions = {
          Jc = 5;
          Jmax = 48;
          Jmin = 16;
          S1 = 16;
          S2 = 32;
          H1 = 1;
          H2 = 2;
          H3 = 3;
          H4 = 4;
        };
        address = [ "11.0.0.7/32" ];
        dns = [ "11.0.0.1" ];
        privateKeyFile = "/home/truva/wireguard-keys/private";
        peers = [
          {
            publicKey = "j3fM602CdZVuGJBACyR8nB7SKmB7T4zZGkA9SHjnjHg=";
            presharedKeyFile = "/home/truva/wireguard-keys/public.psk";
            allowedIPs = [ "11.0.0.0/16" ];
            endpoint = "168.138.92.78:41020";
          }
        ];
      };
    };
  };

  i18n.defaultLocale = "en_US.UTF-8";
  environment.systemPackages = with pkgs; [
    dig 
    dnsutils
    fish
    cryptsetup
  ];
  users.users.truva = { 
    isNormalUser = true;
    extraGroups = [ "wheel" ]; # Enable ‘sudo’ for the user.
    openssh.authorizedKeys.keyFiles = [
      ./key.pub
    ];
  };
  security.acme = {
    defaults.email = "info@ymrtech.com";
    acceptTerms = true;
    certs = {
      "ghost.ymrtech.com".email = "yannick@ymrtech.com";
      "uptime.ymrtech.com".email = "yannick@ymrtech.com";
      "nextcloud.ymrtech.com".email = "yannick@ymrtech.com";
    };
  };
  services = {
    btrfs.autoScrub = {
      enable = true;
      interval = "weekly";
      fileSystems = [ "/" ];
    };
    openssh = {
      enable = true;  
      settings = {  
        PasswordAuthentication = false;
        PermitRootLogin = "no"; 
      };
    };
    syncthing = {
      enable = true;
      user = "truva";
      dataDir = "/home/truva/Documents";
      guiAddress = "0.0.0.0:8384";
      configDir = "/home/truva/.config/syncthing";
      overrideDevices = true;     # overrides any devices added or deleted through the WebUI
      overrideFolders = true;     # overrides any folders added or deleted through the WebUI
      settings = {
        devices = {
          "Pixel8" = {
            id = "***********";
          };
          "captain" = {
            id = "***********";
          };
          "giga" = {
            id = "***********";
          };
        };
        folders = {
          "pixel_8_pro_y9u9-photos" = {         # Name of folder in Syncthing, also the folder ID
            path = "/mnt/Camera";    # Which folder to add to Syncthing
            devices = [ "Pixel8" ];      # Which devices to share the folder with
          };
          "backup" = {
            path = "/mnt/backup";
            devices = [ "captain" "giga" ];
           # ignorePerms = false;  # By default, Syncthing doesn't sync file permissions. This line enables it for this folder.
          };
        };
      };
    };
    nginx = {
      enable = true;
      recommendedProxySettings = true;
      recommendedTlsSettings = true;
      recommendedGzipSettings = true;
      recommendedOptimisation = true;
      sslCiphers = "AES256+EECDH:AES256+EDH:!aNULL";
      sslProtocols = "TLSv1.3";
      virtualHosts."nextcloud.ymrtech.com" =  {
        enableACME = true;
        forceSSL = true;
        locations."/" = {
          proxyPass = "http://11.0.0.2/";
        };
      };
      virtualHosts."ghost.ymrtech.com" =  {
        enableACME = true;
        forceSSL = true;
        locations."/" = {
          proxyPass = "http://11.0.0.2:2368";
        }; 
      };
      virtualHosts."uptime.ymrtech.com" =  {
        enableACME = true;
        forceSSL = true;
        locations."/" = {
          proxyPass = "http://localhost:3001/";
          proxyWebsockets = true;
        };
      };
    };
    uptime-kuma = {
      enable = true;
      settings.port = "3001";
    };
  };
  system.stateVersion = "24.05"; # Did you read the comment?
```
}

```
