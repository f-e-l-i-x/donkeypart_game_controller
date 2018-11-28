# donkeypart_ps3_game_controller ベースで実装された Donkey Car 用ジョイスティックコントローラ

以下の2つのジョイスティックを動かすために、donkeypart_ps3_game_controller パッケージ上のクラスを継承して作成しました。

* [Logicool Wireress GamePad F710](https://amzn.to/2R85kAK)
   F710の上面に"D"か"X"のどちらかに設定できるスイッチが用意されています。
   これは、従来の DirectInput に準拠したドライバを使って接続するかどうかを選択できます。
   残念ながらこのディレクトリ上にあるコードは、Xinput設定の場合のみ対応しています。

> DirectInputモードで Donkey Car を操作したい場合は、 [donkeypart_ps3_game_controller パッケージ準拠のコントローラクラス](../README.md) を使用してください。

* [ELECOM Wireress GamePad JC-U3912T](https://amzn.to/2SddDvo)

ともにUSBドングルが同梱された製品であるため、Raspberry Pi上でのbluetooth設定が不要です。

> ドングルを刺さずに、Raspberry Pi本体上に搭載されたBluetoothデバイスを使用することもできますが、本サイトでは紹介しません。

## 1 インストール

Raspberry Pi上にdonkeycarパッケージがインストールされ、`donkey createcar`コマンドで独自アプリ用ディレクトリ`~/mycar`が作成されている状態とします。

1. Raspberry Pi にターミナル接続します。
2. 以下のコマンドを実行して、前提パッケージ`donkeypart_ps3_game_controller`をインストールします。
    ```bash
    cd
    git clone https://github.com/autorope/donkeypart_ps3_controller.git
    cd donkeypart_ps3_controller
    pip install -e .
    ```
3. 以下のコマンドを実行して、本リポジトリをcloneします。
    ```bash
    cd
    git clone https://github.com/coolerking/donkeypart_game_controller.git
    ```
4. 以下のコマンドを実行して、必要なファイルを`~/mycar/part`へコピーします。
    ```bash
    mkdir ~/mycar/part
    cp ~/donkeypart_game_controller/part/*.py ~/mycar/part/
    ```
5. `~/mycar/manage.py` を、以下のように編集します(ELECOM JC-U3912Tの場合)。
    ```python
        # manage.py デフォルトのジョイスティックpart生成
        #if use_joystick or cfg.USE_JOYSTICK_AS_DEFAULT:
        #    ctr = JoystickController(max_throttle=cfg.JOYSTICK_MAX_THROTTLE,
        #                             steering_scale=cfg.JOYSTICK_STEERING_SCALE,
        #                             throttle_axis=cfg.JOYSTICK_THROTTLE_AXIS,
        #                             auto_record_on_throttle=cfg.AUTO_RECORD_ON_THROTTLE)
        # ジョイスティック part の生成
        if use_joystick or cfg.USE_JOYSTICK_AS_DEFAULT:
            # F710用ジョイスティックコントローラを使用
            #from part.logicool import F710_JoystickController
            #ctr = F710_JoystickController(
            # PS4 Dualshock4 ジョイスティックコントローラを使用
            #from donkeypart_ps3_controller.part import PS4JoystickController
            #ctr = PS4JoystickController(
            # ELECOM JC-U3912T ジョイスティックコントローラを使用
            from part.elecom import JC_U3912T_JoystickController
            ctr = JC_U3912T_JoystickController(

                                 throttle_scale=cfg.JOYSTICK_MAX_THROTTLE,
                                 steering_scale=cfg.JOYSTICK_STEERING_SCALE,
                                # throttle_axis=cfg.JOYSTICK_THROTTLE_AXIS,
                                 auto_record_on_throttle=cfg.AUTO_RECORD_ON_THROTTLE)
    ```

6. Raspberry Piwをシャットダウンします。
7. コントローラ同梱のUSBドングルをRaspberry Piに刺します。
8. コントローラ上面の D/Xスイッチを"X"側にします(DirectInputモードには対応していません)。
9. JC-U3912T の場合は AUTO ボタン、F710 の場合はLogicool ロゴボタンを押します。
10. Raspberry Piを再起動します。

## 2 キー割り当ての変更

Logicool製品の場合は `part/logicool.py`、Elecom製品の場合は `part/elecom.py`上の各 `JoystickController` クラス上のinit_trigger_mapsメソッドを編集することで、ジョイスティックのキーの割当機能を変更できます。

以下の関数は、`part/logicool.py`より該当メソッドのみ切り出した例です。

```python
    def init_trigger_maps(self):
        '''
        F710 上の各ボタンに機能を割り当てるマッピング情報を
        初期化する。

        引数
            なし
        戻り値
            なし
        '''
        self.button_down_trigger_map = {
            'RB': self.toggle_mode,                     # 運転モード変更(user, local_angle, local)
            'LB': self.toggle_manual_recording,         # 手動記録設定変更
            'Y': self.erase_last_N_records,             # 最後のN件を削除(動作していない?)
            'A': self.emergency_stop,                   # 緊急ストップ
            'LB': self.toggle_constant_throttle,        # 一定速度維持
            'LT_pressure': self.chaos_monkey_on_left,   # カオスモード左
            'RT_pressure': self.chaos_monkey_on_right,  # カオスモード右
            'X': self.increase_max_throttle,            # 最大スロットル＋＋
            'B': self.decrease_max_throttle,            # 最大スロットル－－
        }

        self.button_up_trigger_map = {
            "LT_pressure": self.chaos_monkey_off,       # カオスモードオフ
            "RT_pressure": self.chaos_monkey_off,       # カオスモードオフ
        }

        self.axis_trigger_map = {
            'left_stick_horz': self.set_steering,       # ステアリング操作
            'right_stick_vert': self.set_throttle,      # スロットル操作

        }
```

## 3 PS3/PS4 コントローラの充電

PS3/PS4 コントローラは、ドライバがインストール済みであるノードでなくては充電できません。
これは携帯電話などのACアダプタからUSBケーブルでコントローラに接続しても充電できないことを意味しています。

Windows10 は、接続時自動でドライバをインストールします。
充電したい場合は、Windows10 PCとPS4コントローラをケーブルで接続します。


# 4 実行(手動運転)

* 以下のコマンドを実行して、ジョイスティックを使った手動運転を開始します。
   ```bash
   cd ~/mycar
   python manage.py drive --js
   ```



# A ライセンス

本リポジトリの上記OSSで生成、コピーしたコード以外のすべてのコードは[MITライセンス](../LICENSE) 準拠とします。