https://github.com/hakimel/reveal.js/wiki/Linux-udev-keys-binding

As root, install program `evtest`

    sudo apt-get install evtest

Run it

    sudo evtest

Find the smart pointer entry, for example

    /dev/input/event6:	Gaon-Int Wireless Presenter

Give the device event number (here 6).
Or interrupt and restart `evtest` giving the device path

    sudo evtest /dev/input/event6

First, read the line `Input device ID:`

    Input device ID: bus 0x3 vendor 0xef5 product 0x3014 version 0x100

Memorize bus, vendor and product values using 4 capitalized digits without 0x: `0003` `0EF5` `3014`. They will be used later.

Now click on device buttons and read scancode after `(MSC_SCAN), value`

    Event: time 1455575367.579903, type 4 (EV_MSC), code 4 (MSC_SCAN), value 7004b

The scancode is here `7004b`.

As root, create a file `/etc/udev/hwdb.d/61-smart-pointer.hwdb` and put in it something similar to

    ### Smart pointer UP4-SP-800 (aka Satechi SP600) binding example ###
    evdev:input:b0003v0EF5p3014*
      KEYBOARD_KEY_7004b=pageup   # key marked PAGE UP    -> previous
      KEYBOARD_KEY_70005=left     # key marked PAUSE      -> left
      KEYBOARD_KEY_70029=right    # key marked START #1   -> right
      KEYBOARD_KEY_7003e=right    # key marked START #2   -> right
      KEYBOARD_KEY_7004e=pagedown # key marked PAGE DOWN  -> next
      KEYBOARD_KEY_70052=b        # key marked PREV       -> toggle blank mode
      KEYBOARD_KEY_70051=esc      # key marked NEXT       -> toggle overview mode
      KEYBOARD_KEY_700e2=unknown  # key marked ALT-TAB #1 -> nothing 
      KEYBOARD_KEY_7002b=f11      # key marked ALT-TAB #2 -> full screen

The formula `b0003v0EF5p3014*` is constructed by chaining character `b`, bus value `0003`, character `v`, vendor value `0EF5`, character `p`, product value `3014`, a star `*`. Very important, hex numbers should be have **4 capitalized digits**.

To assign the value `pageup` to the top key which emits scancode 0x7004b, write

    KEYBOARD_KEY_7004b=pageup

The word `pageup` on the right of `=` sign comes from https://hal.freedesktop.org/quirk/quirk-keymap-list.txt.

For UP4-SP-800 smart pointer (aka Satechi SP600), the keys START and ALT-TAB are special. START generates alternatively scancodes 
`0x70029` and `0x7003e`. ALT-TAB generates scancode `0x700e2` followed by scancode `0x7002b`. Here `f11` is used as full screen command on Chrome, Firefox, etc.

To take into account the content of the file, call

    sudo udevadm hwdb --update
    sudo udevadm trigger

Tested on Ubuntu 15.10.


 