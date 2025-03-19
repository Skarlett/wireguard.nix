# wireguard.nix

*define your wireguard network once, and use it across multiple `nixosConfiguration` fields.*

---

`wireguard.nix` is a convenience wrapper for `networking.wireguard.interfaces`, and relevant options related to it. 

Many themes in `wireguard.nix` can be done manually, but when `wireguard.nix` is deployed properly, adding most of its mechanisms work

# Example
---

### network
``` nix
# testnet.nix
let
  cfg = config.wireguard.build.networks.testnet;
  inherit (builtins) toString;
in
wireguard.networks.testnet = {
  listenPort = 4000;
  
  peers.by-name = {
    alice = {
      allowedIPs = ["10.100.0.1/32"];
      publicKey = "0JvggtAs57Jv1EkWoEGKvOebEvqvgIei2gh9qJU+/Hg=";
      endpoint = "192.168.100.1:${toString cfg.listenPort}"
    };
    
    bob = {
      allowedIPs = ["10.100.0.2/32"];
      publicKey = "92lkejSAniQXvMiaGXXvF6NvI7YeEnlCG+6jaOsEQTg=";
      persistentKeepalive = 30;
    };
  };
}
```

``` nix
# alice/configuration.nix

imports = [ 
  inputs.sops.modules.nixos.default
  inputs.wireguard.modules.nixos.default
  ./testnet.nix
];

wireguard.enable = true;
networking.hostname = "alice";

sops.defaultSopsFile = ./secrets.json;
sops.defaultSopsFormat = "json";

sops.secrets.testnet = { };
```


``` nix
# bob/configuration.nix

imports = [ 
  inputs.sops.modules.nixos.default
  inputs.wireguard.modules.nixos.default
  ./testnet.nix
];

wireguard.enable = true;
networking.hostname = "bob";

sops.defaultSopsFile = ./secrets.json;
sops.defaultSopsFormat = "json";

sops.secrets.testnet = { };
```


