# External Adapter Support Environment

The External Adapter Support Environment (EASE) allows [WiFi Explorer Pro](https://www.adriangranados.com/apps/wifi-explorer) to use certain external USB adapters for Wi-Fi scanning in macOS. Adapters must be Linux-compatible and support monitor mode.

EASE is basically a lightweight Debian VM that's been customized to leverage the [remote sensor](https://github.com/adriangranados/wifiexplorer-sensor) functionality to automatically configure an external Wi-Fi adapter as a pseudo-local sensor. These pseudo-local sensors are listed in WiFi Explorer Pro separately from remote sensors, but they work in the same manner. 

EASE supports multiple adapters and the installation is fairly straightforward using Vagrant. Once installed, you only need to _attach_ the adapter to the EASE VM and it will automatically show up in WiFi Explorer Pro where you can choose it for scanning.

## Supported External Adapters

EASE can work with Linux-compatible external USB adapters that support monitor mode, however, only a few adapters have been tested:

* [ASUS USB-N53](https://www.amazon.com/Asus-Wireless-N-Graphical-Interface-USB-N53/dp/B005SAKW9G/ref=sr_1_1?ie=UTF8&qid=1515551234&sr=8-1&keywords=asus+usb+n53)
* [ALFA AWUS051NH 802.11a/b/g/n](https://www.amazon.com/Alfa-AWUS051NH-Wireless-Network-9dBi/dp/B003YH1X48/ref=sr_1_1?ie=UTF8&qid=1515526895&sr=8-1&keywords=AWUS051NH)
* [ALFA AWUS036NHA 802.11b/g/n](https://www.amazon.com/Alfa-AWUS036NHA-Wireless-USB-Adaptor/dp/B004Y6MIXS) - Atheros AR9271
* [COMFAST CF-912AC](https://www.amazon.com/Comfast-CF-912AC-1200Mbps-802-11ac-Wireless/dp/B00W37XPPK) - Realtek RTL8812AU
* [Edimax EW-7822UAC AC1200](https://www.amazon.com/gp/product/B00BXAXO7C/ref=ppx_yo_dt_b_asin_title_o01__o00_s00?ie=UTF8&psc=1) - Realtek RTL8812AU 
* [Edimax EW-7833UAC AC1750](https://www.amazon.com/gp/product/B01G51FBF6/ref=oh_aui_detailpage_o01_s00?ie=UTF8&psc=1) - Realtek RTL8814AU
* [Odroid Wi-Fi Module 4](https://ameridroid.com/products/wifi-module-4) - MediaTek (Ralink) RT5572N
* [Odroid Wi-Fi Module 5](https://ameridroid.com/products/wifi-module-5) - Realtek RTL8812AU
* [TP-Link 802.11b/g/n TL-WN272N](https://www.amazon.com/TP-Link-Wireless-Adapter-150Mbps-TL-WN727N/dp/B001WU2N1G/ref=sr_1_1?ie=UTF8&qid=1515706464&sr=8-1&keywords=tp-link+tl-wn727n)

Other adapters using the same driver/chipset should work fine. If your device works in Linux, supports monitor mode but cannot be used with EASE, [contact me](mailto:support@adriangranados.com).

## Installation

1. Install [Vagrant](https://www.vagrantup.com/downloads.html), [VirtualBox](https://www.virtualbox.org/wiki/Downloads) and the [VirtualBox Extension Pack](https://www.virtualbox.org/wiki/Downloads) (required to enable USB support in VirtualBox). If you have any of these components already installed, please make sure you're running the latest version.
2. If __git__ is not installed, type:
```bash
xcode-select --install
```
3. Install the support environment:
```bash
git clone https://github.com/adriangranados/ease.git
cd ease
vagrant up
```

The installation of the environment will take a few minutes as we need to download a clean Debian image and provision it. Once done, EASE is ready to use and you can proceed to attach your external USB Wi-Fi adapter.

## Attaching External USB Adapters to EASE

External USB adapters need to be _attached_ to the EASE VM. Using _VirtualBox's USB Device Filters_ we can configure the environment so that every time you plug in the adapter to the USB port in your computer, the adapter is automatically connected to the EASE VM.

1. Plug in the adapter to any of the USB ports in your Mac.
1. Open VirtualBox and select the EASE VM.
1. Choose Settings > Ports > USB.
1. Add a new USB filter by clicking the icon with the _+_ sign and choosing the adapter you just plugged in.
1. Click OK, then **unplug and plug back in the adapter.**

![USB Device Filters](../master/images/usb-device-filters.png "USB Device Filters")

The adapter will be automatically connected to the EASE VM and made available for Wi-Fi scanning in WiFi Explorer Pro. Repeat the steps above for every adapter you want to use with EASE.

## Using External USB Adapters with WiFi Explorer Pro

Once you have installed EASE and configured the USB device filters to automatically connect the external adapters to the EASE VM, you can choose the adapter from the _Scan Mode_ menu in the WiFi Explorer Pro's toolbar.

![WiFi Explorer Pro's Toolbar](../master/images/wifiexplorerpro-toolbar.png "WiFi Explorer Pro's Toolbar")

## Known Issues

### Noise measurements
It appears none of the adapters tested with EASE report noise values due to limitations of the driver. In order to produce other metrics, such as SNR, WiFi Explorer Pro will use a default noise floor of -96 dBm.

### Atheros AR9271 drivers not compatible with VirtualBox USB 2.0/3.0 controllers
Adapters that require the ath9k_htc driver for the Atheros AR9271 chip (e.g. ALFA AWUS036NHA) don't work with the VirtualBox USB 2.0/3.0 controllers. If you need to use one of these adapters, you must disable USB 2.0/3.0 on the EASE VM as follows:
    
1. Edit the Vagrantfile and change the following line:
    ```bash
    vb.customize ["modifyvm", :id, "--usbxhci", "on"]
    ```

    to:
    
    ```bash
    vb.customize ["modifyvm", :id, "--usbehci", "off"]
    vb.customize ["modifyvm", :id, "--usbxhci", "off"]
    ```

    Note the additional line to disable USB 2.0 (eHCI).

2. Then, restart the EASE VM:
    ```bash
    vagrant halt
    vagrant up
    ```
    
    You may need to re-add the USB filters for the adapter as specified in the instructions for attaching USB adapters to the EASE VM.

## Troubleshooting

If the adapter doesn't appear in WiFi Explorer Pro:
* Make sure you have unplugged and plugged back in the adapter after adding the USB device filter in VirtualBox.
* Make sure the EASE VM is running. If the computer went to sleep, VirtualBox will pause the VM and save the VM state. You can check the status of the VM in VirtualBox or by using the Vagrant CLI.
    ```bash
    cd <directory containing the EASE Vagrantfile>
    vagrant status
    ```
    If the EASE VM is not running, simple type _vagrant up_ from the same directory where the EASE's Vagrantfile is located. 
    ```bash
    cd <directory containing the EASE Vagrantfile>
    vagrant up
    ```
* If nothing helps, you can reboot the EASE VM by typing _vagrant reload_.
    ```bash
    cd <directory containing the EASE Vagrantfile>
    vagrant reload
    ```

If the adapter still doesn't appear in WiFi Explorer Pro, [contact me](mailto:support@adriangranados.com).
