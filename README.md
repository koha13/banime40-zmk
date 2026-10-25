# banime40 — ZMK config

Firmware ZMK cho keyboard **banime40** (ortho 4x10), port từ cấu hình QMK gốc.

> ⚠️ **Phần cứng:** ZMK không chạy trên ATmega32U4 (Pro Micro AVR gốc trong QMK).
> Config này build cho controller **nRF52840** footprint Pro Micro (SuperMini nRF52840),
> flash bằng board target `nice_nano_v2` (tương thích bootloader UF2 + pin map).

## Cấu trúc

```
build.yaml                                  # ma trận build cho GitHub Actions
config/
  west.yml                                  # manifest kéo ZMK về
  banime40.conf                             # tuỳ chọn Kconfig (sleep, BLE…)
  banime40.keymap                           # keymap (4 layer)
  boards/shields/banime40/
    banime40.overlay                        # ma trận GPIO + matrix-transform
    banime40.zmk.yml                        # metadata shield
    Kconfig.shield
    Kconfig.defconfig
```

## Build (khuyến nghị: GitHub Actions)

1. Push repo này lên GitHub.
2. Vào tab **Actions** → workflow build ZMK chạy tự động, artifact `.uf2` nằm ở phần Artifacts.
   (Nếu chưa có workflow, thêm file `.github/workflows/build.yml` theo hướng dẫn ZMK:
   https://zmk.dev/docs/user-setup)

## Build cục bộ (west)

```bash
west init -l config
west update
west zephyr-export
west build -s zmk/app -b nice_nano_v2 -- -DSHIELD=banime40 -DZMK_EXTRA_MODULES="$(pwd)"
# kết quả: build/zephyr/zmk.uf2
```

## Flash

1. Cắm keyboard, double-tap nút reset → hiện ổ đĩa USB `NICENANO`.
2. Copy `zmk.uf2` vào ổ đó → board tự reboot.

## Pin mapping (QMK → ZMK pro_micro)

| QMK (AVR) | Vai trò | `&pro_micro` |
|-----------|---------|--------------|
| E6 D7 C6 D4 | rows | 7 6 5 4 |
| B4 B5 B6 B2 B3 B1 F7 F6 F5 F4 | cols | 8 9 10 16 14 15 18 19 20 21 |

Diode direction: `COL2ROW` (rows = input pull-down, cols = output).

## Trackpad Azoteq TPS43

Driver: bản vendor của [AYM1607/zmk-driver-azoteq-iqs5xx](https://github.com/AYM1607/zmk-driver-azoteq-iqs5xx)
@`27321f0` trong `modules/zmk-driver-azoteq-iqs5xx/`, nạp qua `zephyr/module.yml` của repo.
Thay đổi so với bản gốc: thuộc tính `scroll-divisor` (tốc độ cuộn) và `multi-finger-lockout-ms`
(không di con trỏ khi đang đặt 2 ngón hoặc ngay sau khi cuộn), `two-finger-nav`
(vuốt ngang 2 ngón = nút chuột Back/Forward). TPS43 dùng chip IQS572, giao tiếp I2C địa chỉ `0x74`.

| TPS43 | `&pro_micro` | nRF52840 |
|-------|--------------|----------|
| SDA   | 2            | P0.17    |
| SCL   | 3            | P0.20    |
| RDY   | 1            | P0.06    |
| NRST  | 0            | P0.08    |
| VDD   | 3V3 (VCC)    | —        |
| GND   | GND          | —        |

Gesture mặc định (sửa trong node `tps43` của `banime40.overlay`): chạm 1 ngón = click trái,
chạm 2 ngón = click phải, chạm-giữ = kéo thả, vuốt dọc 2 ngón = cuộn,
vuốt ngang 2 ngón = Back (sang phải) / Forward (sang trái).
Nếu hướng con trỏ bị ngược/xoay, thêm `switch-xy;`, `flip-x;`, `flip-y;` vào node.

## Lưu ý về keymap

`keyboard.json` của QMK **không chứa keymap** (nằm ở `keymap.c` không được cung cấp),
nên `banime40.keymap` là bố cục QWERTY 4x10 mặc định hợp lý + layer LOWER/RAISE/ADJUST
(có sẵn phím điều khiển Bluetooth và bootloader). Chỉnh lại theo ý bạn.
