### Put this in your /boot/firmware/usercfg.txt

    # [GPU]

    # underclock gpu
    core_freq=50

    # [CPU]

    # unlock cpu
    force_turbo=1

    # avoid gpu/cpu scale sharing
    avoid_pwm_pll=1

    # overclock cpu
    arm_freq=1400

    # overvolt for cpu
    over_voltage=6

    # [SD]

    # unlock sd rate
    sdram_schmoo=0x02000020

    # overclock sdram
    sdram_freq=600

    # overvolt sd
    over_voltage_sdram_p=6
    over_voltage_sdram_i=6
    over_voltage_sdram_c=6

    # overclock sdcard interface
    dtoverlay=sdhost,overclock_50=100

    # [Misc]

    # audio off
    dtparam=audio=off

    # delay overclock during boot to prevent sd card corruption
    boot_delay=1

    # disable splash on boot
    disable_splash=1

    # temperature safeguard
    temp_limit=85


### as well as putting this line into the end of /boot/firmware/config.txt

    gpu_mem=16

### reboot and enjoy faster rpi

    sudo shutdown -r now