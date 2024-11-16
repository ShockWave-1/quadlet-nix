# quadlet-nix

Manages Podman containers, networks, pods, etc. on NixOS via [Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html).

## Why

Compared to alternatives like [`virtualisation.oci-containers`](https://github.com/NixOS/nixpkgs/blob/master/nixos/modules/virtualisation/oci-containers.nix) or [`arion`](https://github.com/hercules-ci/arion), `quadlet-nix` is special in that:

|                                                          | `quadlet-nix` | `oci-containers` | `arion` |
| -------------------------------------------------------- | ------------- | ---------------- | ------- |
| **Supports networks / pods**                             | ✅             | ❌              | ✅      |
| **Updates / deletes networks on change**                 | ✅             | /               | ❌      |
| **Supports [podman-auto-update][podman-auto-update]**    | ✅             | ✅              | ❌      |

[podman-auto-update]: https://docs.podman.io/en/latest/markdown/podman-auto-update.1.html

## How

### `flake.nix`

```nix
{
    inputs = {
        nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
        quadlet-nix.url = "github:SEIAROTg/quadlet-nix";
        quadlet-nix.inputs.nixpkgs.follows = "nixpkgs";
    };
    outputs = { nixpkgs, quadlet-nix, ... }@attrs: {
        nixosConfigurations.machine = nixpkgs.lib.nixosSystem {
            system = "x86_64-linux";
            modules = [
                ./configuration.nix
                quadlet-nix.nixosModules.quadlet
            ];
        };
    };
}
```

### `configuration.nix`

```nix
{ config, ... }: {
    # ...
    virtualisation.quadlet = let
        inherit (config.virtualisation.quadlet) networks pods;
    in {
        containers = {
            nginx.containerConfig.image = "docker.io/library/nginx:latest";
            nginx.containerConfig.networks = [ "podman" networks.internal.ref ];
            nginx.containerConfig.pod = pods.foo.ref;
            nginx.serviceConfig.TimeoutStartSec = "60";
        };
        networks = {
            internal.networkConfig.subnets = [ "10.0.123.1/24" ];
        };
        pods = {
            foo = { };
        };
    };
}
```

See [`container.nix`](./container.nix), [`network.nix`](./network.nix), and [`pod.nix`](./pod.nix) for all options.
