Create soure:
flux create source helm tailscale   --url=https://pkgs.tailscale.com/helmcharts   --interval=1h   --export > tailscale-helmrepository.yaml

Create helm release:

flux create helmrelease tailscale-operator \
  --source=HelmRepository/tailscale \
  --chart=tailscale \
  --release-name=tailscale-operator \
  --values=values.yaml \
  --namespace=tailscale \
  --export > tailscale-helmRelease.yaml
