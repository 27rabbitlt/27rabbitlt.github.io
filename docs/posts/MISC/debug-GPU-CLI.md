# Arch + niri 常用排查命令

## [GPU / 显示器]

看显卡与驱动
```bash
lspci -nnk | grep -A3 -E 'VGA|3D|Display'
```

看 DRM 设备
```bash
ls -1 /sys/class/drm/
```

fish: 看每个输出接口属于哪张卡
```bash
for x in /sys/class/drm/card*-*
    echo "=== "(basename $x)" ==="
    echo -n "GPU path: "
    readlink -f "$x/device"
    if test -f "$x/status"
        echo -n "status:   "
        cat "$x/status"
    end
    echo
end
```

fish: 看每张 card 对应哪个 PCI 设备
```bash
for c in /sys/class/drm/card[0-9]
    echo "== $c =="
    readlink -f "$c/device"
end
```

fish: 看每个 render node 对应哪个 PCI 设备
```bash
for r in /sys/class/drm/renderD*
    echo "== $r =="
    readlink -f "$r/device"
end
```

看 niri 识别到的输出
```bash
niri msg outputs
```

看 niri 是否手动指定 render device
```bash
grep -n "render-drm-device" ~/.config/niri/config.kdl
```

看 niri 日志里的 render node
```bash
journalctl --user -b -u niri | grep -i "render node"
```

更宽一点看 niri 的 DRM / 输出相关日志
```bash
journalctl --user -b -u niri | grep -iE "render node|primary node|adding device|new connector"
```


[音频]

看音频设备和正在使用的内核驱动
```bash
lspci -k | grep -A3 -iE 'audio|multimedia'
```

看已加载的声音模块
```bash
lsmod | grep '^snd'
```

看 ALSA 识别到的声卡
```bash
cat /proc/asound/cards
```

看音频相关内核日志
```bash
dmesg | grep -iE 'audio|snd|sof|hda|firmware'
```

看 PipeWire / WirePlumber 状态
```bash
systemctl --user status pipewire wireplumber
```

查模块文件路径
```bash
modinfo snd_hda_intel | grep filename
```


[媒体 / MPRIS]

看有哪些 MPRIS 播放器
playerctl -l

看当前媒体元数据
playerctl metadata

看当前播放状态
playerctl status

直接看 D-Bus 上的 MPRIS 服务
busctl --user list | grep mpris


[服务 / 日志]

```bash
systemctl --user status niri
systemctl --user status pipewire
systemctl --user status wireplumber
```

```bash
journalctl --user -b -u niri
# reverse log
journalctl --user -b -u 服务名 -r
# 实时跟踪日志
journalctl --user -u 服务名 -f
```

[最小工具箱]

lspci -nnk
lsmod
niri msg outputs
ls -1 /sys/class/drm/
systemctl --user status niri
journalctl --user -b -u niri
