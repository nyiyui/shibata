+++
title = "Qrystal"
summary = """
集中管理型WireGuard設定ソフト
"""
+++

Qrystal (クリスタル)はWireGuard tunnelを一つの設定ファイルと使用する端末毎のトークンで設定出来ます。
私はこれを使い、サーバ間のWireGuard tunnelを設定してます。（それぞれ別のクラウドにあるので。）

```nix
{
  qrystal.services.cs = {
    enable = true;
    config.central.networks.example = {
      listenPort = 12345; # wg-quickのListenPortと同様
      ips = [ "10.123.0.0/16" ];
    };
    config.tokens = [
      {
        name = "server0";
        hash = "hash-of-token";
        networks.example = "server0"; # 「example」ネットワーク内のサーバ名
        canPull = true; # 「server0」サーバはこのネットワークを設定を閲覧できるか?
      }
    ];
  };
}
```

[Website](nyiyui.ca/qrystal)
