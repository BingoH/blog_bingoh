---
title: "karabiner settings"
date: 2021-08-23T13:16:50Z
draft: false
---
最近从海鲜市场上收了一把Anne Pro2键盘，也是头一次使用61键配列键盘（没有了方向键和`~`键)。因此借机扩展了键盘配置神器[https://karabiner-elements.pqrs.org/https://karabiner-elements.pqrs.org](karabiner-elemements)的使用。

<!--more-->
61键配列的键盘实际上是标准键盘为了便携性去掉了数字区和F区（包括方向键）剩下的按键。对我来说，macbook自带的蝶式键盘犹如一块钢板，敲一天的代码手指酸痛无比，而大部分60%配列的键盘正好可以放在笔记本的键盘区上，替换自带的键盘。于是寻思许久，在咸鱼淘了一把诸多推荐的Anne Pro2。经过几天的体验后，发现61键使用也足够舒服，特别是对于vim重度使用者，当然需要karabiner-elements的配合。

## 基本配置
- 换键
简单的按键交换可以在`Simple Modifications`标签下配置，诸如capslock改为left_control之类设置既可以由操作系统或者键盘自定义按键配置，也可以利用karabiner-elements进行修改。
- 关闭笔记本键盘
karabiner配置下`Devices->Advanced`可以在指定外接键盘连接后，自动禁用笔记本内置键盘。
- fn-功能键
  60%配列的键盘通常没有独立的fn-功能键，而是将fn键合并到数字键上。因此osx键盘功能键提供的多媒体控制等等需要在karabiner-elements中配置映射，具体在`Function keys`下选择All Devices或者指定外接键盘，做如下映射配置
  ```
  f1 -> display_brightness_decrement
  f2 -> display_brightness_increment
  f3 -> mission_control
  f4 -> launchpad
  f5 -> illumination_down
  f6 -> illumination_up
  f7 -> rewind
  f8 -> play_or_pause
  f9 -> fast_forward
  f10 -> mute
  f11 -> volume_decrement
  f12 -> volume_increment
  ```
  但是做此配置之后，其他app中fn绑定的快捷键则无法生效了。fn映射下方可选择选项`Use all F1, F2, etc. keys as standard function keys`，则fn功能键仅输入`fx`而无法提供osx默认的快捷键功能。所谓鱼与熊掌无法兼得。
  
  妥协的方法是利用karabiner elements另行绑定这些快捷键，对于anne pro2而言，则更为简单可以直接定义至fn1或者fn2层按键。

## 复杂规则
### complex modification定义
karabiner-elements强大之处在于其提供的复杂映射规则。在其配置界面中找到config目录，自定义的复杂规则需要放置在assets子目录下然后回到配置界面gui激活。所有激活的复杂规则会被集中添加到软件的全局配置文件config/karabiner.json中。自定义规则同样以json表示，格式如下
```json
{
    "title": "Title for the list of rules/complex modifications."
    "rules": [
        {
            "description": "This description is shown in Preferences.",
            "manipulators": [
                {
                    "type": "basic",

                    "from": from event definition,

                    "to": [
                        to event definition,
                        to event definition,
                        ...
                    ],

                    "to_if_alone": [
                        to event definition,
                        to event definition,
                        ...
                    ],

                    "to_if_held_down": [
                        to event definition,
                        to event definition,
                        ...
                    ],

                    "to_after_key_up": [
                        to event definition,
                        to event definition,
                        ...
                    ],

                    "to_delayed_action": {
                        "to_if_invoked": [
                            to event definition,
                            to event definition,
                            ...
                        ],
                        "to_if_canceled": [
                            to event definition,
                            to event definition,
                            ...
                        ]
                    },

                    "conditions": [
                        condition definition,
                        condition definition,
                        ...
                    ],

                    "parameters": {
                        ...
                    },

                    "description": "Optional description for human"
                },
                {
                    "type": "basic",
                    ...
                },
                ...
            ]
        },
        {
            "description": "...",
            "manipulators": [
                ...
            ]
        },
        ...
    ]
}
```
rules列表中每一条rule表示一个映射规则集，其中包含的manipulatros列表为具体的映射规则。

### 针对60%配列配置
- 交换`;`和`:`
  对于vim特别方便，见[文档示例](https://karabiner-elements.pqrs.org/docs/json/typical-complex-modifications-examples)，这里也给出了几个manipulator定义的例子。
- esc和`~`
  linux下需要频繁输入`~`，而Anne Pro2（60%配列）合并了`Esc`和`~`键（markdown下输出反点号难倒了我），所以为了输入后者需要同时按下`Fn+Shift+Esc`。此时可以通过复杂规则映射`Shift+Esc`为`Shift+grave_accent_and_tilde`。同时mac常用的同应用不同窗口切换快捷键`Cmd+~`可以绑定为`Cmd+Esc`。
  ```json
  {
    "type": "basic",
    "from": {
      "key_code": "escape",
      "modifiers": {
        "mandatory": ["shift"],
        "optional": ["caps_lock"]
      }
    },
    "to": [
      {
        "key_code": "grave_accent_and_tilde",
        "modifiers": ["shift"]
      }
    ]
  },
  {
    "type": "basic",
    "from": {
      "key_code": "escape",
      "modifiers": {
        "mandatory": ["command"],
        "optional": ["shift"]
      }
    },
    "to": [
      {
        "key_code": "grave_accent_and_tilde",
        "modifiers": ["command"]
      }
    ]
  }
  ```
  其中mandatory为必须的修饰键，在映射后会被去掉，即如果需要保留修饰键的话需要在`to`中列出，而optional修饰键则会被自动保留。
- 键盘控制光标
  在karabiner-elements提供的complex modifications中可以找到诸多mouse mode方案，采用其中的Mu Mouse Mode配置可以实现键盘控制鼠标移动和实现点击动作。具体方法为映射application+d(或者任意其他键)为[virtual modifier](https://karabiner-elements.pqrs.org/docs/json/extra/virtual-modifier/)，利用conditions条件设置在这些virtual modifier激活状态下，映射hjkl等键为鼠标移动事件。
  ```json
  {
    "type": "basic",
    "from": {
      "key_code": "d",
      "modifiers": {
        "optional": ["any"]
      }
    },
    "to": [
      {
        "set_variable": {
          "name": "osx60_mouse_mode",
          "value": 1
        }
      },
      {
        "set_variable": {
          "name": "osx60_mouse_move_mode",
          "value": 1
        }
      },
      {
        "set_variable": {
          "name": "osx60_mouse_move_fast_mode",
          "value": 0
        }
      },
      {
        "set_variable": {
          "name": "osx60_mouse_scroll_mode",
          "value": 0
        }
      }
    ]
  },
  {
    "type": "basic",
    "from": {
      "key_code": "h",
      "modifiers": {
        "optional": ["any"]
      }
    },
    "conditions": [
      {
        "type": "variable_if",
        "name": "osx60_mouse_move_mode",
        "value": 1
      },
      {
        "type": "variable_if",
        "name": "osx60_scroll_mode",
        "value": 0
      }
    ],
    "to": [
      "mouse_key": {
        "x": -700
      }
    ]
  }
  //...
  ```
- 特定应用下改键
  可以定义特定应用下生效的按键映射，例如在preview中映射`j/k`为方向下/上实现vim式移动，只需要对于按键映射定义中添加conditions如下
  ```json
  "conditions": [
    {
      "type": "frontmost_application_if",
      "bundle_identifiers": [
        "^com.apple.Preview$",
        "^com.pdfguru.PDFGurus$"
      ]
    }
  ]
  ```
  
### debug
至于如何测试所编写的规则，可以利用karabiner附带的karabiner-eventviewer应用，依次测试所定义的按键发出预想之中的信号。上面提到的frontmost_application_if，键盘设备的vendor_id，自定义变量(virtual modifier)都可以在其中观察。
